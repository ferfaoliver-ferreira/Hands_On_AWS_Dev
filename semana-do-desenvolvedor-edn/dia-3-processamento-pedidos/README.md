# Dia 3 - Processamento Central de Pedidos e Persistencia

Este laboratorio documenta a etapa de processamento central dos pedidos, conectando o fluxo de validacao dos Dias 1 e 2 a uma camada principal de persistencia com **Amazon EventBridge**, **Amazon SQS Standard**, **AWS Lambda** e **Amazon DynamoDB**. A proposta foi consumir os eventos `NovoPedidoValidado`, desacoplar o processamento via fila e salvar o estado final do pedido em uma tabela principal.

## Por que este conhecimento e importante

Nos dois primeiros dias, o sistema foi preparado para receber pedidos de multiplas origens e validar esses dados antes de publicar um evento. O Dia 3 mostra como transformar esse evento em um fluxo central de negocio, no qual pedidos validados passam por processamento e sao persistidos de forma estruturada.

Os principais ganhos praticos deste desenho sao:

- **Padronizacao do processamento** apos a validacao dos pedidos
- **Desacoplamento entre roteamento e persistencia**
- **Reaproveitamento dos eventos** publicados por API e S3
- **Buffer resiliente com SQS** antes da regra principal de negocio
- **Persistencia final em DynamoDB** com status e dados do pedido

## Objetivo do laboratorio

Construir um fluxo em que:

- o **EventBridge** capture eventos `NovoPedidoValidado`
- uma **regra** encaminhe esses eventos para uma fila SQS Standard
- uma **Lambda de processamento** consuma a fila
- a funcao simule a logica central do pedido
- o pedido processado seja persistido em uma **tabela DynamoDB principal**

## Servicos utilizados

- Amazon EventBridge
- Amazon SQS Standard
- Amazon SQS Dead-Letter Queue
- AWS Lambda
- Amazon DynamoDB
- AWS IAM
- Amazon CloudWatch Logs

## Arquitetura implementada

![Arquitetura do laboratorio](./images/arquitetura-dia-3.png)

```mermaid
flowchart LR
    A["Custom Event Bus"] --> B["EventBridge Rule"]
    B --> C["SQS Pedidos Pendentes"]
    C --> D["Lambda de Processamento de Pedidos"]
    C --> E["SQS DLQ"]
    D --> F["DynamoDB Pedidos"]
```

## Recursos criados no laboratorio

- `lambda-processa-pedidos-role-seu-nome`
- `pedidos-pendentes-dlq-seu-nome`
- `pedidos-pendentes-queue-seu-nome`
- `processa-pedidos-lambda-seu-nome`
- `pedidos-db-seu-nome`
- `novo-pedido-validado-rule-seu-nome`

## Fluxo funcional do Dia 3

1. Um pedido validado e publicado no custom event bus.
2. A regra `novo-pedido-validado-rule-seu-nome` captura o evento.
3. O evento segue para a fila `pedidos-pendentes-queue-seu-nome`.
4. A Lambda `processa-pedidos-lambda-seu-nome` consome a fila.
5. A funcao extrai o campo `detail` do evento EventBridge.
6. O pedido recebe o status `PEDIDO_PROCESSADO`.
7. Os dados finais sao persistidos em `pedidos-db-seu-nome`.

## Passo a passo tecnico

### 1. Criar a role IAM da Lambda de processamento

Criar a role:

- `lambda-processa-pedidos-role-seu-nome`

Com a politica gerenciada:

- `AWSLambdaBasicExecutionRole`

Depois, complementar com permissoes para:

- ler mensagens da fila SQS principal
- gravar e consultar itens na tabela DynamoDB principal

### 2. Criar a DLQ e a fila principal de pedidos pendentes

Criar primeiro a DLQ:

- `pedidos-pendentes-dlq-seu-nome`

Depois criar a fila principal:

- `pedidos-pendentes-queue-seu-nome`

Configuracoes importantes:

- tipo `Standard`
- `Visibility timeout = 70 seconds`
- DLQ habilitada
- `Maximum receives = 3`

### 3. Criar a tabela DynamoDB principal

Criar a tabela:

- `pedidos-db-seu-nome`

Chave primaria:

- `pedidoId` do tipo `String`

Essa tabela sera o armazenamento principal do estado final dos pedidos processados.

### 4. Atualizar as permissoes da role

Adicionar uma inline policy com acesso a SQS e DynamoDB:

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
      "Resource": "COLE_AQUI_O_ARN_DA_SUA_FILA_pedidos-pendentes-queue-seu-nome"
    },
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:GetItem"
      ],
      "Resource": "COLE_AQUI_O_ARN_DA_SUA_TABELA_pedidos-db-seu-nome"
    }
  ]
}
```

### 5. Criar a Lambda de processamento de pedidos

Criar a funcao:

- `processa-pedidos-lambda-seu-nome`

Runtime sugerido:

- `Python 3.12`

Variavel de ambiente:

- `DYNAMODB_TABLE_NAME`

Codigo utilizado:

```python
import json
import os
import boto3
from datetime import datetime

DYNAMODB_TABLE_NAME = os.environ["DYNAMODB_TABLE_NAME"]
dynamodb = boto3.resource("dynamodb")
table = dynamodb.Table(DYNAMODB_TABLE_NAME)

def lambda_handler(event, context):
    print(f"Evento SQS (pedidos pendentes) recebido: {event}")

    for record in event["Records"]:
        try:
            eventbridge_event = json.loads(record["body"])
            print(f"Evento EventBridge recebido via SQS: {eventbridge_event}")

            if "detail" not in eventbridge_event:
                print(f"Erro: Campo 'detail' nao encontrado no evento: {eventbridge_event}")
                continue

            pedido_data = eventbridge_event["detail"]
            print(f"Processando detalhes do pedido: {pedido_data}")

            pedido_id = pedido_data.get("pedidoId")
            if not pedido_id:
                print(f"Erro: pedidoId nao encontrado nos detalhes do evento: {pedido_data}")
                continue

            status_processamento = "PEDIDO_PROCESSADO"
            print(f"Logica de processamento para pedido {pedido_id} concluida (simulada).")

            item_to_put = {
                "pedidoId": str(pedido_id),
                "clienteId": str(pedido_data.get("clienteId")),
                "itens": pedido_data.get("itens", []),
                "statusPedido": status_processamento,
                "origem": pedido_data.get("origem", "API"),
                "nomeArquivoOriginal": pedido_data.get("nomeArquivoOriginal"),
                "timestampCriacaoEvento": eventbridge_event.get(
                    "time",
                    pedido_data.get("timestamp", datetime.utcnow().isoformat() + "Z")
                ),
                "timestampProcessamento": datetime.utcnow().isoformat() + "Z"
            }

            item_final = {k: v for k, v in item_to_put.items() if v is not None}
            table.put_item(Item=item_final)

            print(f"Pedido {pedido_id} salvo/atualizado no DynamoDB com status {status_processamento}.")

        except json.JSONDecodeError as je:
            print(f"Erro de JSON ao processar registro {record['messageId']}: {str(je)}")
            raise je
        except Exception as e:
            pedido_id_para_log = "N/A"
            if "pedido_data" in locals() and isinstance(pedido_data, dict):
                pedido_id_para_log = pedido_data.get("pedidoId", "N/A")
            print(f"Erro geral ao processar registro {record['messageId']} (pedidoId: {pedido_id_para_log}): {str(e)}")
            raise e

    return {"statusCode": 200, "body": "Processamento de pedidos pendentes concluido"}
```

### 6. Configurar a Lambda

Depois do deploy:

- adicionar `DYNAMODB_TABLE_NAME = pedidos-db-seu-nome`
- ajustar `Timeout = 60 segundos`
- adicionar trigger SQS

Trigger:

- fonte `SQS`
- fila `pedidos-pendentes-queue-seu-nome`
- `Batch size = 1`

### 7. Criar a regra no EventBridge

Criar a regra:

- `novo-pedido-validado-rule-seu-nome`

No custom event bus `pedidos-event-bus-seu-nome`, com o seguinte padrao:

```json
{
  "source": ["lab.aula1.pedidos.validacao"],
  "detail-type": ["NovoPedidoValidado"]
}
```

Destino:

- `SQS queue`
- fila `pedidos-pendentes-queue-seu-nome`

### 8. Testar o fluxo completo

Enviar um pedido via API:

```bash
curl -X POST <INVOKE_URL>/pedidos \
  -H "Content-Type: application/json" \
  -d '{
    "pedidoId": "apiP001-seu-nome",
    "clienteId": "clienteAPI-XYZ-seu-nome",
    "itens": [
      {
        "item": "Produto X API",
        "qtd": 1
      }
    ]
  }'
```

Depois, testar tambem com um arquivo do fluxo do Dia 2 contendo:

- `s3P003-seu-nome`

Validacoes esperadas:

- a Lambda de validacao publica os eventos no EventBridge
- a regra encaminha os eventos para `pedidos-pendentes-queue-seu-nome`
- a Lambda de processamento grava `apiP001-seu-nome` e `s3P003-seu-nome` em `pedidos-db-seu-nome`
- os itens persistidos possuem `statusPedido = PEDIDO_PROCESSADO`

## Aprendizados consolidados

Este laboratorio fecha o elo entre a validacao e a persistencia final do pedido. A partir daqui, tanto pedidos vindos da API quanto pedidos extraidos do S3 podem passar por uma mesma etapa de processamento central e chegar a um armazenamento unico com status consolidado.

Os principais conceitos reforcados foram:

- **EventBridge para roteamento baseado em padrao**
- **SQS como buffer entre evento e regra de negocio**
- **Lambda para processamento central**
- **DynamoDB como base principal dos pedidos**
- **Integracao de multiplas origens em um fluxo comum**

## Limpeza dos recursos

Ao final do laboratorio, remover:

- regra do EventBridge criada para `NovoPedidoValidado`
- fila principal de pedidos pendentes e sua DLQ
- Lambda de processamento de pedidos
- tabela DynamoDB principal
- role IAM criada para o laboratorio

## Evidencias visuais em ordem das orientacoes

### Fase 1 - IAM e preparo do fluxo central
Criacao da role da Lambda de processamento de pedidos.
![Evidencia 01](./images/step-01.png)
Selecao da trusted entity para Lambda.
![Evidencia 02](./images/step-02.png)
Associacao da policy basica da funcao.
![Evidencia 03](./images/step-03.png)
Revisao final da role criada.
![Evidencia 04](./images/step-04.png)

### Fase 2 - SQS pendentes e DLQ
Criacao da DLQ `pedidos-pendentes-dlq-seu-nome`.
![Evidencia 05](./images/step-05.png)
Confirmacao da fila DLQ.
![Evidencia 06](./images/step-06.png)
Criacao da fila `pedidos-pendentes-queue-seu-nome`.
![Evidencia 07](./images/step-07.png)
Configuracao de `Visibility timeout = 70`.
![Evidencia 08](./images/step-08.png)
Associacao da DLQ e ajuste de `Maximum receives = 3`.
![Evidencia 09](./images/step-09.png)
Revisao da fila principal criada.
![Evidencia 10](./images/step-10.png)

### Fase 3 - DynamoDB principal
Criacao da tabela `pedidos-db-seu-nome`.
![Evidencia 11](./images/step-11.png)
Definicao da chave `pedidoId`.
![Evidencia 12](./images/step-12.png)
Tabela criada e pronta para uso.
![Evidencia 13](./images/step-13.png)
Conferencia do ARN da tabela.
![Evidencia 14](./images/step-14.png)

### Fase 4 - Permissoes da role
Criacao da inline policy com acesso a SQS e DynamoDB.
![Evidencia 15](./images/step-15.png)
JSON da policy com `ReceiveMessage`, `DeleteMessage` e `PutItem`.
![Evidencia 16](./images/step-16.png)
Policy revisada antes da criacao.
![Evidencia 17](./images/step-17.png)
Permissoes associadas a role.
![Evidencia 18](./images/step-18.png)

### Fase 5 - Lambda de processamento
Criacao da funcao `processa-pedidos-lambda-seu-nome`.
![Evidencia 19](./images/step-19.png)
Selecao do runtime Python 3.12.
![Evidencia 20](./images/step-20.png)
Escolha da role da Lambda.
![Evidencia 21](./images/step-21.png)
Edicao do codigo da funcao.
![Evidencia 22](./images/step-22.png)
Deploy da Lambda de processamento.
![Evidencia 23](./images/step-23.png)
Configuracao da variavel `DYNAMODB_TABLE_NAME`.
![Evidencia 24](./images/step-24.png)
Ajuste do timeout para 60 segundos.
![Evidencia 25](./images/step-25.png)
Adicao do trigger SQS.
![Evidencia 26](./images/step-26.png)
Selecao da fila `pedidos-pendentes-queue-seu-nome`.
![Evidencia 27](./images/step-27.png)
Trigger configurado com `Batch size = 1`.
![Evidencia 28](./images/step-28.png)

### Fase 6 - Regra do EventBridge
Criacao da regra `novo-pedido-validado-rule-seu-nome`.
![Evidencia 29](./images/step-29.png)
Definicao do event bus customizado.
![Evidencia 30](./images/step-30.png)
Configuracao do padrao `source` e `detail-type`.
![Evidencia 31](./images/step-31.png)
Selecao da fila SQS como destino.
![Evidencia 32](./images/step-32.png)
Revisao e criacao final da regra.
![Evidencia 33](./images/step-33.png)

### Fase 7 - Testes do fluxo completo
Teste com pedido enviado via API.
![Evidencia 34](./images/step-34.png)
Teste com pedido originado de arquivo S3.
![Evidencia 35](./images/step-35.png)
Conferencia da regra no EventBridge.
![Evidencia 36](./images/step-36.png)
Logs da Lambda de processamento central.
![Evidencia 37](./images/step-37.png)
Consulta aos itens persistidos no DynamoDB.
![Evidencia 38](./images/step-38.png)
Validacao final do fluxo central do Dia 3.
![Evidencia 39](./images/step-39.png)
