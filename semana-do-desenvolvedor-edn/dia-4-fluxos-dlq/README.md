# Dia 4 - Persistencia, Fluxos Adicionais de Pedidos, DLQs e Lambda Layers

Este laboratorio documenta a etapa final da arquitetura, expandindo o sistema para lidar com **cancelamento de pedidos**, **alteracao de pedidos** e **teste pratico de DLQs**, usando **Amazon EventBridge**, **Amazon SQS Standard**, **AWS Lambda** e a **tabela principal no DynamoDB** criada no Dia 3. A aula tambem consolida a visao completa da arquitetura integrada ao longo da semana.

## Por que este conhecimento e importante

Sistemas reais nao tratam apenas criacao de pedidos. Em algum momento, pedidos precisam ser alterados, cancelados ou reprocessados apos falhas. O Dia 4 traz exatamente esse realismo: operacoes adicionais no ciclo de vida do pedido, novas regras de roteamento e validacao do comportamento das DLQs em um fluxo orientado a eventos.

Os ganhos praticos dessa etapa sao:

- **expansao funcional do ciclo de vida do pedido**
- **reaproveitamento da arquitetura existente** com novas operacoes
- **tratamento isolado por tipo de evento**
- **uso pratico de DLQs** para resiliencia e diagnostico
- **visao consolidada da arquitetura completa**

## Objetivo do laboratorio

Construir um fluxo em que:

- eventos `CancelarPedido` e `AlterarPedido` sejam publicados no EventBridge
- regras dedicadas encaminhem cada tipo de evento para sua fila SQS
- Lambdas especificas atualizem o pedido existente no DynamoDB
- as DLQs capturem falhas repetidas no processamento
- a arquitetura final demonstre um sistema orientado a eventos mais completo

## Servicos utilizados

- Amazon EventBridge
- Amazon SQS Standard
- Amazon SQS Dead-Letter Queue
- AWS Lambda
- Amazon DynamoDB
- AWS IAM
- Amazon CloudWatch Logs

## Arquitetura implementada

![Arquitetura do laboratorio](./images/arquitetura-dia-4.png)

```mermaid
flowchart LR
    A["EventBridge"] --> B["Rule CancelarPedido"]
    A --> C["Rule AlterarPedido"]
    B --> D["SQS Cancelar Pedido"]
    C --> E["SQS Alterar Pedido"]
    D --> F["Lambda Cancela Pedido"]
    E --> G["Lambda Altera Pedido"]
    D --> H["DLQ Cancelamento"]
    E --> I["DLQ Alteracao"]
    F --> J["DynamoDB Pedidos"]
    G --> J
```

## Recursos criados no laboratorio

- `lambda-altera-cancela-role-seu-nome`
- `cancela-pedido-dlq-seu-nome`
- `cancela-pedido-queue-seu-nome`
- `cancela-pedido-lambda-seu-nome`
- `altera-pedido-dlq-seu-nome`
- `altera-pedido-queue-seu-nome`
- `altera-pedido-lambda-seu-nome`
- `cancela-pedido-rule-seu-nome`
- `altera-pedido-rule-seu-nome`

## Fluxo funcional do Dia 4

1. Um evento `CancelarPedido` ou `AlterarPedido` e publicado no EventBridge.
2. A regra correspondente direciona o evento para sua fila SQS dedicada.
3. A Lambda especifica consome a fila.
4. No cancelamento, o pedido recebe `statusPedido = CANCELADO`.
5. Na alteracao, o pedido recebe novos itens e `statusPedido = ALTERADO`.
6. Se houver falhas repetidas, a mensagem vai para a DLQ correspondente.

## Passo a passo tecnico

### 1. Criar a role IAM compartilhada das novas Lambdas

Criar a role:

- `lambda-altera-cancela-role-seu-nome`

Com a politica gerenciada:

- `AWSLambdaBasicExecutionRole`

Depois, complementar com permissoes de:

- leitura das filas SQS de cancelamento e alteracao
- `dynamodb:UpdateItem`
- `dynamodb:GetItem`

### 2. Criar o fluxo de cancelamento

Recursos:

- `cancela-pedido-dlq-seu-nome`
- `cancela-pedido-queue-seu-nome`
- `cancela-pedido-lambda-seu-nome`
- `cancela-pedido-rule-seu-nome`

Configuracoes importantes da fila principal:

- tipo `Standard`
- `Visibility timeout = 70 segundos`
- DLQ habilitada
- `Maximum receives = 3`

Permissao inicial para o trigger da fila:

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
      "Resource": [
        "COLE_AQUI_O_ARN_DA_SUA_FILA_cancela-pedido-queue-seu-nome"
      ]
    }
  ]
}
```

Codigo da Lambda de cancelamento:

```python
import json
import os
import boto3
from datetime import datetime

DYNAMODB_TABLE_NAME = os.environ["DYNAMODB_TABLE_NAME"]
dynamodb = boto3.resource("dynamodb")
table = dynamodb.Table(DYNAMODB_TABLE_NAME)

def lambda_handler(event, context):
    print(f"Evento SQS (cancelamento) recebido: {event}")

    for record in event["Records"]:
        pedido_id = None
        try:
            eventbridge_event = json.loads(record["body"])
            print(f"Evento EventBridge recebido via SQS: {eventbridge_event}")

            if "detail" not in eventbridge_event:
                print(f"Erro: Campo 'detail' nao encontrado no evento: {eventbridge_event}")
                continue

            pedido_data = eventbridge_event["detail"]
            pedido_id = pedido_data.get("pedidoId")

            if not pedido_id:
                print(f"Erro: pedidoId nao encontrado nos detalhes do evento: {pedido_data}")
                continue

            print(f"Processando cancelamento para pedido: {pedido_id}")

            response = table.update_item(
                Key={"pedidoId": str(pedido_id)},
                UpdateExpression="SET statusPedido = :status, timestampAtualizacao = :ts",
                ExpressionAttributeValues={
                    ":status": "CANCELADO",
                    ":ts": datetime.utcnow().isoformat() + "Z"
                },
                ReturnValues="UPDATED_NEW"
            )

            print(f"Pedido {pedido_id} atualizado para CANCELADO. Resposta DynamoDB: {response}")

        except json.JSONDecodeError as je:
            print(f"Erro de JSON ao processar registro {record['messageId']}: {str(je)}")
            raise je
        except Exception as e:
            print(f"Erro geral ao processar cancelamento {record['messageId']} (pedidoId: {pedido_id}): {str(e)}")
            raise e

    return {"statusCode": 200, "body": "Processamento de cancelamentos concluido"}
```

Regra do EventBridge:

```json
{
  "source": ["lab.aula4.operacoes"],
  "detail-type": ["CancelarPedido"]
}
```

### 3. Criar o fluxo de alteracao

Recursos:

- `altera-pedido-dlq-seu-nome`
- `altera-pedido-queue-seu-nome`
- `altera-pedido-lambda-seu-nome`
- `altera-pedido-rule-seu-nome`

Permissao inicial para o trigger da fila:

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
      "Resource": [
        "COLE_AQUI_O_ARN_DA_SUA_FILA_altera-pedido-queue-seu-nome"
      ]
    }
  ]
}
```

Codigo da Lambda de alteracao:

```python
import json
import os
import boto3
from datetime import datetime

DYNAMODB_TABLE_NAME = os.environ["DYNAMODB_TABLE_NAME"]
dynamodb = boto3.resource("dynamodb")
table = dynamodb.Table(DYNAMODB_TABLE_NAME)

def lambda_handler(event, context):
    print(f"Evento SQS (alteracao) recebido: {event}")

    for record in event["Records"]:
        pedido_id = None
        try:
            eventbridge_event = json.loads(record["body"])
            print(f"Evento EventBridge recebido via SQS: {eventbridge_event}")

            if "detail" not in eventbridge_event:
                print(f"Erro: Campo 'detail' nao encontrado no evento: {eventbridge_event}")
                continue

            pedido_data = eventbridge_event["detail"]
            pedido_id = pedido_data.get("pedidoId")
            novos_itens = pedido_data.get("novosItens")

            if not pedido_id or novos_itens is None:
                print(f"Erro: pedidoId ou novosItens nao encontrados nos detalhes do evento: {pedido_data}")
                continue

            print(f"Processando alteracao para pedido: {pedido_id} com novos itens: {novos_itens}")

            response = table.update_item(
                Key={"pedidoId": str(pedido_id)},
                UpdateExpression="SET itens = :i, statusPedido = :s, timestampAtualizacao = :ts",
                ExpressionAttributeValues={
                    ":i": novos_itens,
                    ":s": "ALTERADO",
                    ":ts": datetime.utcnow().isoformat() + "Z"
                },
                ReturnValues="UPDATED_NEW"
            )

            print(f"Pedido {pedido_id} atualizado para ALTERADO. Resposta DynamoDB: {response}")

        except json.JSONDecodeError as je:
            print(f"Erro de JSON ao processar registro {record['messageId']}: {str(je)}")
            raise je
        except Exception as e:
            print(f"Erro geral ao processar alteracao {record['messageId']} (pedidoId: {pedido_id}): {str(e)}")
            raise e

    return {"statusCode": 200, "body": "Processamento de alteracoes concluido"}
```

Regra do EventBridge:

```json
{
  "source": ["lab.aula4.operacoes"],
  "detail-type": ["AlterarPedido"]
}
```

### 4. Atualizar permissoes da role para o DynamoDB

Adicionar policy com:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:UpdateItem",
        "dynamodb:GetItem"
      ],
      "Resource": "COLE_AQUI_O_ARN_DA_SUA_TABELA_pedidos-db-seu-nome"
    }
  ]
}
```

### 5. Testar cancelamento e alteracao

Evento de cancelamento:

```json
{
  "pedidoId": "<ID_DO_SEU_PEDIDO>"
}
```

Com:

- `Event source = lab.aula4.operacoes`
- `Detail type = CancelarPedido`

Evento de alteracao:

```json
{
  "pedidoId": "<ID_DO_SEU_PEDIDO>",
  "novosItens": [
    { "sku": "PROD-ALTERADO-1", "qtd": 1 },
    { "sku": "PROD-ALTERADO-2", "qtd": 99 }
  ]
}
```

Com:

- `Event source = lab.aula4.operacoes`
- `Detail type = AlterarPedido`

Resultados esperados:

- pedido passa para `CANCELADO` no fluxo de cancelamento
- pedido passa para `ALTERADO` e recebe novos itens no fluxo de alteracao

### 6. Testar uma DLQ na pratica

Para validar a DLQ do fluxo principal:

1. editar a `processa-pedidos-lambda-seu-nome`
2. adicionar uma falha forcada no inicio do `lambda_handler`
3. reenviar um pedido pelo fluxo da API
4. observar a falha repetida da Lambda
5. confirmar a mensagem na `pedidos-pendentes-dlq-seu-nome`
6. remover a falha forcada e fazer novo deploy

Esse teste mostra na pratica como a mensagem deixa a fila principal apos exceder o numero maximo de tentativas.

## Aprendizados consolidados

Este laboratorio fecha a semana adicionando operacoes reais de ciclo de vida do pedido e reforcando a resiliência do desenho. A arquitetura passa a lidar nao apenas com criacao e processamento inicial, mas tambem com manutencao do estado do pedido e diagnostico de falhas.

Os principais conceitos reforcados foram:

- **novos tipos de eventos no EventBridge**
- **filas e Lambdas especializadas por operacao**
- **atualizacao incremental do estado no DynamoDB**
- **DLQs como mecanismo de seguranca operacional**
- **visao integrada da arquitetura completa**

## Limpeza dos recursos

Ao final do laboratorio, remover:

- regras `cancela-pedido-rule-seu-nome` e `altera-pedido-rule-seu-nome`
- filas principais e DLQs de cancelamento e alteracao
- Lambdas `cancela-pedido-lambda-seu-nome` e `altera-pedido-lambda-seu-nome`
- policies inline adicionadas a role
- restaurar qualquer Lambda alterada no teste de DLQ

## Evidencias visuais em ordem das orientacoes

### Fase 1 - Role compartilhada das Lambdas
Criacao da role `lambda-altera-cancela-role-seu-nome`.
![Evidencia 01](./images/step-01.png)
Selecao do caso de uso Lambda na IAM Role.
![Evidencia 02](./images/step-02.png)
Adicao da policy basica de execucao.
![Evidencia 03](./images/step-03.png)

### Fase 2 - Filas e Lambda de cancelamento
Criacao da DLQ `cancela-pedido-dlq-seu-nome`.
![Evidencia 04](./images/step-04.png)
Criacao da fila principal `cancela-pedido-queue-seu-nome`.
![Evidencia 05](./images/step-05.png)
Configuracao da DLQ e do visibility timeout.
![Evidencia 06](./images/step-06.png)
Criacao da policy de leitura da fila de cancelamento.
![Evidencia 07](./images/step-07.png)
Criacao da Lambda `cancela-pedido-lambda-seu-nome`.
![Evidencia 08](./images/step-08.png)
Edicao do codigo da Lambda de cancelamento.
![Evidencia 09](./images/step-09.png)
Configuracao de variavel de ambiente e timeout.
![Evidencia 10](./images/step-10.png)
Adicao do trigger SQS da fila de cancelamento.
![Evidencia 11](./images/step-11.png)
Criacao da regra `cancela-pedido-rule-seu-nome` no EventBridge.
![Evidencia 12](./images/step-12.png)

### Fase 3 - Filas e Lambda de alteracao
Criacao da DLQ `altera-pedido-dlq-seu-nome`.
![Evidencia 13](./images/step-13.png)
Criacao da fila principal `altera-pedido-queue-seu-nome`.
![Evidencia 14](./images/step-14.png)
Configuracao da policy de leitura da fila de alteracao.
![Evidencia 15](./images/step-15.png)
Criacao da Lambda `altera-pedido-lambda-seu-nome`.
![Evidencia 16](./images/step-16.png)
Edicao do codigo da Lambda de alteracao.
![Evidencia 17](./images/step-17.png)
Configuracao de variavel de ambiente e timeout.
![Evidencia 18](./images/step-18.png)
Adicao do trigger SQS da fila de alteracao.
![Evidencia 19](./images/step-19.png)
Criacao da regra `altera-pedido-rule-seu-nome` no EventBridge.
![Evidencia 20](./images/step-20.png)

### Fase 4 - Permissoes finais e testes funcionais
Atualizacao da role com permissoes de `UpdateItem` e `GetItem`.
![Evidencia 21](./images/step-21.png)
Teste de envio do evento `CancelarPedido`.
![Evidencia 22](./images/step-22.png)
Validacao do pedido cancelado no DynamoDB.
![Evidencia 23](./images/step-23.png)
Teste de envio do evento `AlterarPedido`.
![Evidencia 24](./images/step-24.png)
Validacao do pedido alterado no DynamoDB.
![Evidencia 25](./images/step-25.png)

### Fase 5 - Teste de DLQ
Insercao de erro forcado na Lambda de processamento.
![Evidencia 26](./images/step-26.png)
Reenvio de pedido para disparar falhas repetidas.
![Evidencia 27](./images/step-27.png)
Observacao da fila principal durante as tentativas.
![Evidencia 28](./images/step-28.png)
Confirmacao da mensagem na DLQ.
![Evidencia 29](./images/step-29.png)
Restauracao da Lambda apos o teste.
![Evidencia 30](./images/step-30.png)
Fechamento da arquitetura e validacao final do Dia 4.
![Evidencia 31](./images/step-31.png)
