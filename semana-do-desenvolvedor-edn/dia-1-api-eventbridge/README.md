# Dia 1 - Ingestao de Pedidos via API e EventBridge na AWS

Este laboratorio documenta a construcao de um fluxo inicial de ingestao de pedidos usando **Amazon API Gateway**, **AWS Lambda**, **Amazon SQS FIFO**, **Dead-Letter Queue (DLQ)**, **IAM** e **Amazon EventBridge**. A proposta foi simular a entrada de pedidos em uma API REST, realizar uma pre-validacao, enfileirar os dados com garantia de ordem e, por fim, validar e publicar um evento de dominio em um barramento customizado.

## Por que este conhecimento e importante

Em cenarios reais, o recebimento de pedidos costuma ser a porta de entrada para varios fluxos internos. Quando esse processo e construido de forma assicrona e desacoplada, a aplicacao ganha mais resiliencia, controle operacional e capacidade de escalar sem depender de processamento sincrono ponta a ponta.

Os principais ganhos praticos deste laboratorio sao:

- **Desacoplamento entre entrada e processamento** com API Gateway, Lambda e SQS.
- **Garantia de ordenacao** com uso de fila FIFO.
- **Tratamento de falhas** com DLQ para mensagens que excedem o numero de tentativas.
- **Publicacao de eventos de negocio** com EventBridge para integracoes futuras.
- **Observabilidade** via CloudWatch Logs em cada etapa do fluxo.

## Objetivo do laboratorio

Construir um pipeline de ingestao de pedidos em que:

- uma **API REST** recebe os dados do pedido;
- uma **Lambda de pre-validacao** valida os campos basicos e publica a mensagem na fila;
- uma **fila SQS FIFO** desacopla a entrada do processamento principal;
- uma **Lambda de validacao** consome a fila, valida o payload e publica um evento no EventBridge;
- uma **DLQ FIFO** recebe mensagens com falha recorrente no consumo.

## Servicos utilizados

- Amazon API Gateway
- AWS Lambda
- Amazon SQS FIFO
- Amazon SQS Dead-Letter Queue
- AWS IAM
- Amazon EventBridge
- Amazon CloudWatch Logs

## Arquitetura implementada

![Arquitetura do laboratorio](./images/arquitetura-dia-1.png)

```mermaid
flowchart LR
    A["Cliente / Requisicao HTTP"] --> B["API Gateway (REST)"]
    B --> C["Lambda de Pre-Validacao"]
    C --> D["SQS FIFO Pedidos"]
    D --> E["Lambda de Validacao de Pedidos"]
    D --> F["SQS FIFO DLQ"]
    E --> G["Custom Event Bus"]
    G --> H["Regras futuras / novos consumidores"]
```

## Recursos criados no laboratorio

- `lambda-prevalidacao-role-seu-nome`
- `lambda-validacao-pedidos-role-seu-nome`
- `pedidos-fifo-dlq-seu-nome.fifo`
- `pedidos-fifo-queue-seu-nome.fifo`
- `pre-validacao-lambda-seu-nome`
- `pedidos-api-seu-nome`
- `pedidos-event-bus-seu-nome`
- `validacao-pedidos-lambda-seu-nome`

## Fluxo funcional do Dia 1

1. O cliente envia um `POST /pedidos` para o API Gateway.
2. A Lambda `pre-validacao` recebe o payload e valida campos minimos como `pedidoId` e `clienteId`.
3. Se o payload estiver valido, a mensagem e enviada para a fila `pedidos-fifo-queue-seu-nome.fifo`.
4. A Lambda `validacao-pedidos` e acionada pelo trigger da fila SQS.
5. A funcao realiza validacoes adicionais e publica o evento `NovoPedidoValidado` no EventBridge.
6. Se houver falhas recorrentes na etapa de consumo, a mensagem segue para a DLQ configurada.

## Passo a passo tecnico

### 1. Criar as roles IAM das Lambdas

Criar duas roles para separar as permissoes de cada etapa do fluxo:

- `lambda-prevalidacao-role-seu-nome`
- `lambda-validacao-pedidos-role-seu-nome`

Ambas com a politica gerenciada:

- `AWSLambdaBasicExecutionRole`

Depois, complementar com inline policies especificas:

- `sqs:SendMessage` para a role da pre-validacao
- `sqs:ReceiveMessage`, `sqs:DeleteMessage`, `sqs:GetQueueAttributes` e `events:PutEvents` para a role da validacao

### 2. Criar a DLQ FIFO e a fila principal

Criar primeiro a fila de mensagens mortas:

- `pedidos-fifo-dlq-seu-nome.fifo`

Depois criar a fila principal:

- `pedidos-fifo-queue-seu-nome.fifo`

Configuracoes relevantes:

- tipo `FIFO`
- redrive policy habilitada
- DLQ apontando para a fila morta criada anteriormente
- `Maximum receives = 3`

### 3. Atualizar a role da Lambda de pre-validacao

Adicionar uma inline policy para permitir envio de mensagens para a fila principal:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sqs:SendMessage",
      "Resource": "arn:aws:sqs:REGION:ACCOUNT_ID:pedidos-fifo-queue-seu-nome.fifo"
    }
  ]
}
```

### 4. Criar a Lambda de pre-validacao

Criar a funcao:

- `pre-validacao-lambda-seu-nome`

Runtime sugerido:

- `Python 3.12`

Variavel de ambiente:

- `SQS_QUEUE_URL`

Codigo utilizado:

```python
import json
import os
import boto3

SQS_QUEUE_URL = os.environ["SQS_QUEUE_URL"]
sqs = boto3.client("sqs")

def lambda_handler(event, context):
    print(f"Evento recebido do API Gateway: {event}")
    try:
        body_str = event.get("body", "{}")
        if not body_str:
            body_str = "{}"

        body = json.loads(body_str)
        pedido_id = body.get("pedidoId")
        cliente_id = body.get("clienteId")

        if not pedido_id or not cliente_id:
            print("Erro: pedidoId ou clienteId ausente.")
            return {
                "statusCode": 400,
                "headers": {"Content-Type": "application/json"},
                "body": json.dumps({"message": "pedidoId e clienteId sao obrigatorios"})
            }

        response = sqs.send_message(
            QueueUrl=SQS_QUEUE_URL,
            MessageBody=json.dumps(body),
            MessageGroupId=str(pedido_id),
            MessageDeduplicationId=str(context.aws_request_id)
        )

        print(f"Mensagem enviada para SQS: {response['MessageId']}")
        return {
            "statusCode": 200,
            "headers": {"Content-Type": "application/json"},
            "body": json.dumps({
                "message": "Pedido recebido e enfileirado",
                "sqsMessageId": response["MessageId"]
            })
        }
    except json.JSONDecodeError:
        print("Erro: corpo da requisicao nao e um JSON valido.")
        return {
            "statusCode": 400,
            "headers": {"Content-Type": "application/json"},
            "body": json.dumps({"message": "Corpo da requisicao invalido"})
        }
    except Exception as e:
        print(f"Erro inesperado: {str(e)}")
        return {
            "statusCode": 500,
            "headers": {"Content-Type": "application/json"},
            "body": json.dumps({"message": f"Erro interno do servidor: {str(e)}"})
        }
```

### 5. Criar a API REST no API Gateway

Criar uma API REST chamada:

- `pedidos-api-seu-nome`

Depois:

1. Criar o recurso `/pedidos`
2. Criar o metodo `POST`
3. Usar integracao com `Lambda Function`
4. Habilitar `Use Lambda Proxy integration`
5. Apontar para `pre-validacao-lambda-seu-nome`
6. Fazer deploy no stage `dev`

Ao final, anotar a `Invoke URL`.

### 6. Testar a ingestao inicial com curl

Exemplo de teste:

```bash
curl -X POST https://abcdef123.execute-api.us-east-1.amazonaws.com/dev/pedidos \
  -H "Content-Type: application/json" \
  -d '{
    "pedidoId": "lab001-seu-nome",
    "clienteId": "clienteXYZ-seu-nome",
    "itens": [
      { "produto": "Caneta Azul", "quantidade": 10 },
      { "produto": "Caderno Universitario", "quantidade": 2 }
    ]
  }'
```

Resposta esperada:

```json
{
  "message": "Pedido recebido e enfileirado",
  "sqsMessageId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

### 7. Criar o custom event bus

Criar no EventBridge:

- `pedidos-event-bus-seu-nome`

Esse barramento sera o destino dos eventos publicados pela Lambda de validacao.

### 8. Atualizar a role da Lambda de validacao

Adicionar uma inline policy com permissoes de leitura na fila e publicacao no EventBridge:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "sqs:ReceiveMessage",
        "sqs:DeleteMessage",
        "sqs:GetQueueAttributes"
      ],
      "Resource": "COLE_AQUI_O_ARN_DA_SUA_FILA_PRINCIPAL_FIFO"
    },
    {
      "Effect": "Allow",
      "Action": "events:PutEvents",
      "Resource": "COLE_AQUI_O_ARN_DO_SEU_EVENT_BUS"
    }
  ]
}
```

### 9. Criar a Lambda de validacao de pedidos

Criar a funcao:

- `validacao-pedidos-lambda-seu-nome`

Runtime sugerido:

- `Python 3.12`

Variavel de ambiente:

- `EVENT_BUS_NAME`

Codigo utilizado:

```python
import json
import os
import boto3
from datetime import datetime

EVENT_BUS_NAME = os.environ["EVENT_BUS_NAME"]
event_bridge = boto3.client("events")

def lambda_handler(event, context):
    print(f"Evento SQS recebido: {event}")

    for record in event["Records"]:
        try:
            pedido_data_str = record["body"]
            pedido_data = json.loads(pedido_data_str)
            print(f"Processando pedido do SQS: {pedido_data}")

            if not pedido_data.get("itens") or not isinstance(pedido_data.get("itens"), list) or len(pedido_data.get("itens")) == 0:
                print(f"Erro de validacao: Pedido {pedido_data.get('pedidoId')} nao possui itens validos.")
                continue

            print(f"Pedido {pedido_data.get('pedidoId')} validado com sucesso.")

            if "timestamp" not in pedido_data:
                pedido_data["timestamp"] = datetime.utcnow().isoformat() + "Z"

            response = event_bridge.put_events(
                Entries=[{
                    "Source": "lab.aula1.pedidos.validacao",
                    "DetailType": "NovoPedidoValidado",
                    "Detail": json.dumps(pedido_data),
                    "EventBusName": EVENT_BUS_NAME
                }]
            )

            print(f"Evento publicado no EventBridge: {response}")

        except json.JSONDecodeError as je:
            print(f"Erro de JSON ao processar registro {record['messageId']}: {str(je)}")
            raise je
        except Exception as e:
            print(f"Erro geral ao processar registro {record['messageId']}: {str(e)}")
            raise e

    return {"statusCode": 200, "body": "Processamento de mensagens SQS concluido"}
```

### 10. Configurar o trigger SQS da Lambda de validacao

Na Lambda `validacao-pedidos-lambda-seu-nome`, adicionar um trigger:

- fonte `SQS`
- fila `pedidos-fifo-queue-seu-nome.fifo`
- `Batch size = 1`

Esse ajuste e importante para respeitar o comportamento FIFO e facilitar a leitura dos logs.

### 11. Testar o fluxo completo do Dia 1

Enviar um novo pedido para a API:

```bash
curl -X POST <INVOKE_URL>/pedidos \
  -H "Content-Type: application/json" \
  -d '{
    "pedidoId": "lab001-seu-nome",
    "clienteId": "clienteABC-seu-nome",
    "itens": [
      { "produto": "Borracha Branca", "quantidade": 5 }
    ]
  }'
```

Validacoes esperadas:

- a Lambda de pre-validacao registra o recebimento da requisicao
- a mensagem aparece e e consumida da fila SQS
- a Lambda de validacao registra o processamento do pedido
- o EventBridge recebe um evento `NovoPedidoValidado`

## Exemplo do evento publicado

```json
{
  "Source": "lab.aula1.pedidos.validacao",
  "DetailType": "NovoPedidoValidado",
  "Detail": {
    "pedidoId": "lab001-seu-nome",
    "clienteId": "clienteABC-seu-nome",
    "itens": [
      {
        "produto": "Borracha Branca",
        "quantidade": 5
      }
    ],
    "timestamp": "2026-06-21T00:00:00Z"
  }
}
```

## Aprendizados consolidados

Este laboratorio reforca um padrao muito comum em arquiteturas orientadas a eventos. Em vez de concentrar toda a logica em uma unica API sincrona, o pedido entra por um endpoint simples, e o processamento segue por etapas desacopladas. Isso reduz acoplamento, melhora resiliencia operacional e prepara o sistema para novas integracoes sem alterar o ponto de entrada.

Os principais conceitos praticados foram:

- **API Gateway como porta de entrada** para ingestao de pedidos
- **Lambda para validacao em duas etapas**
- **SQS FIFO para ordenacao e desacoplamento**
- **DLQ para tratamento de falhas**
- **EventBridge para propagacao de eventos de negocio**
- **CloudWatch Logs para acompanhamento do fluxo**

## Limpeza dos recursos

Ao final do laboratorio, remover:

- API Gateway criada para o endpoint `/pedidos`
- Lambdas `pre-validacao` e `validacao-pedidos`
- fila principal FIFO e sua DLQ
- custom event bus
- roles IAM criadas para o laboratorio

## Evidencias visuais em ordem das orientacoes

### Fase 1 - IAM e preparacao inicial
Tela inicial do laboratorio com foco na area de funcoes e permissao.
![Evidencia 01](./images/step-01.png)
Criacao da role para a Lambda de pre-validacao.
![Evidencia 02](./images/step-02.png)
Selecao do tipo de trusted entity para Lambda.
![Evidencia 03](./images/step-03.png)
Associacao da politica AWSLambdaBasicExecutionRole.
![Evidencia 04](./images/step-04.png)
Revisao da primeira role criada para o fluxo.
![Evidencia 05](./images/step-05.png)
Criacao da role para a Lambda de validacao de pedidos.
![Evidencia 06](./images/step-06.png)
Conferencia das roles IAM disponiveis para o laboratorio.
![Evidencia 07](./images/step-07.png)

### Fase 2 - SQS FIFO e DLQ
Inicio da criacao da dead-letter queue FIFO.
![Evidencia 08](./images/step-08.png)
Configuracao da DLQ `pedidos-fifo-dlq-seu-nome.fifo`.
![Evidencia 09](./images/step-09.png)
DLQ criada com sucesso no console do Amazon SQS.
![Evidencia 10](./images/step-10.png)
Criacao da fila principal FIFO de pedidos.
![Evidencia 11](./images/step-11.png)
Associacao da redrive policy com a fila morta.
![Evidencia 12](./images/step-12.png)
Definicao de `Maximum receives = 3`.
![Evidencia 13](./images/step-13.png)
Revisao final da fila principal de pedidos.
![Evidencia 14](./images/step-14.png)

### Fase 3 - Permissao de envio para a fila
Retorno ao IAM para criar a policy inline da pre-validacao.
![Evidencia 15](./images/step-15.png)
Edicao da policy em formato JSON com `sqs:SendMessage`.
![Evidencia 16](./images/step-16.png)
Substituicao do ARN da fila principal no documento da policy.
![Evidencia 17](./images/step-17.png)
Revisao da policy antes da criacao.
![Evidencia 18](./images/step-18.png)
Policy criada e associada a role da Lambda de pre-validacao.
![Evidencia 19](./images/step-19.png)

### Fase 4 - Lambda de pre-validacao
Criacao da funcao `pre-validacao-lambda-seu-nome`.
![Evidencia 20](./images/step-20.png)
Selecao do runtime Python para a primeira Lambda.
![Evidencia 21](./images/step-21.png)
Escolha da role de execucao da pre-validacao.
![Evidencia 22](./images/step-22.png)
Funcao criada e pronta para edicao do codigo.
![Evidencia 23](./images/step-23.png)
Edicao do arquivo `lambda_function.py` com a logica de enfileiramento.
![Evidencia 24](./images/step-24.png)
Deploy do codigo da Lambda.
![Evidencia 25](./images/step-25.png)
Configuracao da variavel de ambiente `SQS_QUEUE_URL`.
![Evidencia 26](./images/step-26.png)
Conferencia final da Lambda de pre-validacao.
![Evidencia 27](./images/step-27.png)

### Fase 5 - API Gateway REST
Criacao da API REST `pedidos-api-seu-nome`.
![Evidencia 28](./images/step-28.png)
Definicao dos detalhes iniciais da API.
![Evidencia 29](./images/step-29.png)
Criacao do recurso `/pedidos`.
![Evidencia 30](./images/step-30.png)
Configuracao do metodo `POST`.
![Evidencia 31](./images/step-31.png)
Integracao do metodo com a Lambda de pre-validacao.
![Evidencia 32](./images/step-32.png)
Deploy da API no stage `dev`.
![Evidencia 33](./images/step-33.png)
Visualizacao da `Invoke URL` para testes.
![Evidencia 34](./images/step-34.png)

### Fase 6 - Teste inicial API -> Lambda -> SQS
Execucao do primeiro teste com `curl`.
![Evidencia 35](./images/step-35.png)
Payload JSON enviado para o endpoint `/pedidos`.
![Evidencia 36](./images/step-36.png)
Resposta de sucesso indicando o enfileiramento do pedido.
![Evidencia 37](./images/step-37.png)
Consulta aos logs da Lambda de pre-validacao no CloudWatch.
![Evidencia 38](./images/step-38.png)
Validacao da mensagem publicada na fila SQS.
![Evidencia 39](./images/step-39.png)
Detalhes da mensagem presente na fila principal.
![Evidencia 40](./images/step-40.png)

### Fase 7 - EventBridge custom event bus
Criacao do barramento customizado no Amazon EventBridge.
![Evidencia 41](./images/step-41.png)
Definicao do nome `pedidos-event-bus-seu-nome`.
![Evidencia 42](./images/step-42.png)
Event bus criado e pronto para receber eventos.
![Evidencia 43](./images/step-43.png)

### Fase 8 - Permissoes da Lambda de validacao
Retorno ao IAM para editar a role de validacao.
![Evidencia 44](./images/step-44.png)
Criacao da policy inline com leitura da SQS e `events:PutEvents`.
![Evidencia 45](./images/step-45.png)
Insercao do ARN da fila principal FIFO na policy.
![Evidencia 46](./images/step-46.png)
Insercao do ARN do custom event bus.
![Evidencia 47](./images/step-47.png)
Revisao final da policy da Lambda de validacao.
![Evidencia 48](./images/step-48.png)

### Fase 9 - Lambda de validacao e trigger SQS
Criacao da funcao `validacao-pedidos-lambda-seu-nome`.
![Evidencia 49](./images/step-49.png)
Selecao do runtime Python e da role da segunda Lambda.
![Evidencia 50](./images/step-50.png)
Edicao do codigo que consome a SQS e publica no EventBridge.
![Evidencia 51](./images/step-51.png)
Deploy do codigo da Lambda de validacao.
![Evidencia 52](./images/step-52.png)
Configuracao da variavel de ambiente `EVENT_BUS_NAME`.
![Evidencia 53](./images/step-53.png)
Adicao do trigger SQS na funcao.
![Evidencia 54](./images/step-54.png)
Selecao da fila FIFO principal como origem do trigger.
![Evidencia 55](./images/step-55.png)
Definicao de `Batch size = 1`.
![Evidencia 56](./images/step-56.png)
Trigger configurado e pronto para processar mensagens.
![Evidencia 57](./images/step-57.png)

### Fase 10 - Teste completo do fluxo
Novo envio de pedido para validar o pipeline completo.
![Evidencia 58](./images/step-58.png)
Resposta da API com o pedido aceito e enfileirado.
![Evidencia 59](./images/step-59.png)
Logs da pre-validacao confirmando o envio para a fila.
![Evidencia 60](./images/step-60.png)
Logs da Lambda de validacao consumindo a mensagem do SQS.
![Evidencia 61](./images/step-61.png)
Confirmacao da validacao bem-sucedida do pedido.
![Evidencia 62](./images/step-62.png)
Publicacao do evento no EventBridge registrada nos logs.
![Evidencia 63](./images/step-63.png)
Monitoramento do barramento mostrando atividade apos o teste.
![Evidencia 64](./images/step-64.png)
Conferencia final do laboratorio e das metricas do fluxo.
![Evidencia 65](./images/step-65.png)
Validacao de encerramento da etapa do Dia 1.
![Evidencia 66](./images/step-66.png)
