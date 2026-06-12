# Arquitetura Fan-Out com SNS, SQS e Lambda para um E-commerce na AWS

Este laboratório documenta a construção de uma arquitetura orientada a eventos para um cenário de e-commerce usando **Amazon SNS**, **Amazon SQS**, **AWS Lambda**, **IAM** e **CloudWatch**. A proposta foi simular o processamento paralelo de um pedido, separando responsabilidades como atualização de estoque, pagamento, notificação ao cliente e análise de fraude.

## Por que este conhecimento é importante para e-commerce

Em um e-commerce real, um único pedido dispara várias ações ao mesmo tempo. Se tudo ficar acoplado em uma única aplicação síncrona, qualquer lentidão ou falha em um componente pode travar o fluxo inteiro. A arquitetura fan-out resolve esse problema ao transformar o pedido em um evento distribuído para múltiplos consumidores especializados.

Os ganhos práticos são claros:

- **Escalabilidade**: cada fluxo cresce de forma independente conforme o volume de pedidos aumenta.
- **Resiliência**: o uso de SQS e DLQ evita perda de mensagens e isola falhas operacionais.
- **Baixo acoplamento**: inventário, pagamento, notificação e fraude evoluem sem depender de deploy conjunto.
- **Observabilidade**: com CloudWatch Logs, fica mais simples rastrear o caminho do pedido e depurar gargalos.
- **Eficiência de custos**: filtros no SNS evitam execuções desnecessárias das Lambdas.

## Objetivo do laboratório

Construir um barramento de eventos para pedidos de e-commerce no qual:

- o **SNS** publica o evento central do pedido;
- as **Lambdas** de inventário, pagamento e notificação processam apenas eventos relevantes;
- a **SQS** desacopla a análise de fraude do fluxo principal;
- a **DLQ** captura falhas de processamento na etapa de fraude;
- o **CloudWatch** registra a execução completa para validação operacional.

## Serviços utilizados

- Amazon SNS
- Amazon SQS
- AWS Lambda
- AWS IAM
- Amazon CloudWatch Logs

## Arquitetura implementada

```mermaid
flowchart LR
    A[Pedido publicado no SNS] --> B[Lambda de Inventario]
    A --> C[Lambda de Pagamento]
    A --> D[Lambda de Notificacao]
    A --> E[Fila SQS de Analise de Fraude]
    E --> F[Lambda de Analise de Fraude]
    E --> G[DLQ da Fraude]
```

## Recursos criados no laboratório

- `topico-pedidos-ecommerce-FernandaOliveira`
- `fila-fraude-analise-FernandaOliveira`
- `fila-fraude-analise-dlq-FernandaOliveira`
- `fila-pagamento-FernandaOliveira`
- `fila-notificacao-cliente-FernandaOliveira`
- `lambda-inventario-FernandaOliveira`
- `lambda-pagamento-FernandaOliveira`
- `lambda-notificacao-cliente-FernandaOliveira`
- `lambda-analise-fraude-FernandaOliveira`
- `LambdaRoleSNSGeneric-FernandaOliveira`
- `LambdaRoleSQSFraude-FernandaOliveira`

## Passo a passo técnico

### 1. Criar a fila principal e a DLQ da análise de fraude

1. Criar a DLQ `fila-fraude-analise-dlq-FernandaOliveira` no Amazon SQS.
2. Criar a fila principal `fila-fraude-analise-FernandaOliveira`.
3. Habilitar a redrive policy e apontar para a DLQ.
4. Configurar `Maximum receives = 3`.

### 2. Criar o tópico SNS do e-commerce

1. Criar um tópico **Standard** chamado `topico-pedidos-ecommerce-FernandaOliveira`.
2. Registrar o ARN do tópico para assinaturas e políticas.

### 3. Criar os papéis IAM

1. Criar `LambdaRoleSNSGeneric-FernandaOliveira` com `AWSLambdaBasicExecutionRole`.
2. Criar `LambdaRoleSQSFraude-FernandaOliveira` com `AWSLambdaBasicExecutionRole` e permissões de SQS.

### 4. Criar as Lambdas do fluxo de negócio

As funções seguem uma lógica simples: recebem o evento, leem os atributos da mensagem e registram o processamento no CloudWatch.

#### Lambda de inventário

```python
import json

def lambda_handler(event, context):
    message_attributes = event['Records'][0]['Sns']['MessageAttributes']
    order_id = message_attributes.get('OrderID', {}).get('Value', 'N/A')
    print(f"Inventario atualizado para o pedido: {order_id}")
    return {'statusCode': 200, 'body': json.dumps(order_id)}
```

#### Lambda de pagamento

```python
import json

def lambda_handler(event, context):
    message_attributes = event['Records'][0]['Sns']['MessageAttributes']
    order_id = message_attributes.get('OrderID', {}).get('Value', 'N/A')
    payment_type = message_attributes.get('PaymentType', {}).get('Value', 'N/A')
    print(f"Pagamento {payment_type} processado para o pedido: {order_id}")
    return {'statusCode': 200, 'body': json.dumps(order_id)}
```

#### Lambda de notificação ao cliente

```python
import json

def lambda_handler(event, context):
    message_attributes = event['Records'][0]['Sns']['MessageAttributes']
    order_id = message_attributes.get('OrderID', {}).get('Value', 'N/A')
    customer_email = message_attributes.get('CustomerEmail', {}).get('Value', 'N/A')
    print(f"Notificacao enviada para {customer_email} sobre o pedido {order_id}")
    return {'statusCode': 200, 'body': json.dumps(order_id)}
```

#### Lambda de análise de fraude

```python
import json

def lambda_handler(event, context):
    for record in event['Records']:
        sns_message_body = json.loads(record['body'])
        message_attributes = sns_message_body.get('MessageAttributes', {})
        order_id = message_attributes.get('OrderID', {}).get('Value', 'N/A')
        transaction_value = float(message_attributes.get('TransactionValue', {}).get('Value', 0))
        print(f"Analisando fraude para o pedido {order_id} com valor {transaction_value}")
    return {'statusCode': 200, 'body': json.dumps('Analise concluida')}
```

### 5. Configurar o gatilho SQS da Lambda de fraude

1. Abrir `lambda-analise-fraude-FernandaOliveira`.
2. Em `Configuration > Triggers`, adicionar um gatilho SQS.
3. Selecionar `fila-fraude-analise-FernandaOliveira`.
4. Definir `Batch size = 1` para facilitar a leitura dos logs.
5. Garantir que o gatilho fique `Enabled`.

### 6. Criar assinaturas e filtros do SNS

#### Inventário

```json
{
  "EventType": ["OrderPlaced", "InventoryCheckRequired"]
}
```

#### Pagamento

```json
{
  "EventType": ["OrderPlaced"],
  "PaymentType": ["CreditCard", "Boleto"]
}
```

#### Notificação ao cliente

```json
{
  "EventType": ["OrderConfirmed", "OrderShipped"]
}
```

#### Fraude via SQS

```json
{
  "EventType": ["OrderPlaced"],
  "TransactionValue": [{ "numeric": [">", 500] }]
}
```

### 7. Publicar uma mensagem de teste no SNS

Payload publicado no tópico:

```json
{
  "pedido_id": "PEDIDO-123",
  "cliente_id": "CLIENTE-XYZ",
  "itens": [
    { "produto_id": "PROD-A", "quantidade": 2 },
    { "produto_id": "PROD-B", "quantidade": 1 }
  ]
}
```

Atributos da mensagem usados pelos filtros:

- `EventType = OrderPlaced`
- `OrderID = PEDIDO-123`
- `PaymentType = CreditCard`
- `CustomerEmail = cliente@example.com`
- `TransactionValue = 750`

### 8. Validar o fluxo no CloudWatch

Resultado esperado após publicar a mensagem:

- `lambda-inventario` executa porque recebe `OrderPlaced`.
- `lambda-pagamento` executa porque recebe `OrderPlaced` + `PaymentType=CreditCard`.
- `lambda-notificacao-cliente` não executa nesse teste.
- `lambda-analise-fraude` executa porque a SQS recebeu `OrderPlaced` com `TransactionValue > 500`.

## Aprendizados consolidados

Este laboratório mostra, na prática, um padrão muito usado em arquiteturas modernas de e-commerce. Em operações reais, um mesmo pedido pode acionar estoque, cobrança, antifraude, expedição e comunicação com o cliente sem que um fluxo bloqueie o outro. Dominar esse desenho ajuda a construir sistemas mais confiáveis, escaláveis e preparados para picos de campanha, datas promocionais e integrações com múltiplos serviços.

Em termos de arquitetura cloud, os principais pontos reforçados foram:

- **fan-out com SNS** para distribuição inteligente de eventos;
- **filtragem de assinaturas** para roteamento seletivo;
- **SQS como buffer resiliente** para tarefas assíncronas mais sensíveis;
- **DLQ para tratamento de falhas** e recuperação operacional;
- **CloudWatch Logs** para observabilidade do ciclo completo do pedido.

## Limpeza dos recursos

Ao final do laboratório, remover:

- assinaturas do tópico SNS;
- tópico SNS;
- filas SQS principal e DLQ;
- Lambdas criadas;
- papéis IAM do laboratório.

## Evidências visuais em ordem das orientações

### Fase 1 - preparação do SQS e da DLQ
Visão inicial do console SQS antes da criação das filas.
![Evidência 01](./images/step-01.png)
Criação da dead-letter queue da análise de fraude.
![Evidência 02](./images/step-02.png)
Confirmação da DLQ criada com sucesso.
![Evidência 03](./images/step-03.png)
Configuração da fila principal de fraude com redrive policy.
![Evidência 04](./images/step-04.png)
Lista das filas no SQS após a criação da DLQ.
![Evidência 05](./images/step-05.png)
Ajustes complementares da fila principal no console.
![Evidência 06](./images/step-06.png)
Conferência do ambiente SQS antes de seguir para o SNS.
![Evidência 07](./images/step-07.png)
### Fase 2 - criação do tópico SNS
Tela de criação do tópico principal do e-commerce no Amazon SNS.
![Evidência 08](./images/step-08.png)
Definição do tipo Standard para o barramento de eventos.
![Evidência 09](./images/step-09.png)
Confirmação do tópico criado e pronto para receber assinaturas.
![Evidência 10](./images/step-10.png)
Detalhes do tópico com ARN e configurações principais.
![Evidência 11](./images/step-11.png)
Navegação pelas opções do tópico antes das integrações.
![Evidência 12](./images/step-12.png)
Validação final do tópico antes da etapa de IAM.
![Evidência 13](./images/step-13.png)
### Fase 3 - papéis IAM para as Lambdas
Início da criação da role genérica para funções Lambda.
![Evidência 14](./images/step-14.png)
Seleção da política AWSLambdaBasicExecutionRole.
![Evidência 15](./images/step-15.png)
Revisão do papel IAM para Inventário, Pagamento e Notificação.
![Evidência 16](./images/step-16.png)
Criação da role específica para a Lambda de fraude via SQS.
![Evidência 17](./images/step-17.png)
Adição de permissões extras de SQS para consumo da fila.
![Evidência 18](./images/step-18.png)
Conclusão da role LambdaRoleSQSFraude para a análise de fraude.
![Evidência 19](./images/step-19.png)
Conferência dos papéis IAM criados para o laboratório.
![Evidência 20](./images/step-20.png)
### Fase 4 - Lambda de inventário
Criação da função lambda-inventario no console Lambda.
![Evidência 21](./images/step-21.png)
Seleção do runtime Python para a função de inventário.
![Evidência 22](./images/step-22.png)
Escolha da role de execução da Lambda de inventário.
![Evidência 23](./images/step-23.png)
Função criada e pronta para edição do código.
![Evidência 24](./images/step-24.png)
Editor da função com o código de processamento do tópico SNS.
![Evidência 25](./images/step-25.png)
Deploy do código da Lambda de inventário.
![Evidência 26](./images/step-26.png)
Validação da função publicada no console.
![Evidência 27](./images/step-27.png)
### Fase 5 - Lambda de pagamento
Criação da função lambda-pagamento.
![Evidência 28](./images/step-28.png)
Configuração básica com runtime Python 3.12.
![Evidência 29](./images/step-29.png)
Vinculação da role genérica de Lambda ao fluxo de pagamento.
![Evidência 30](./images/step-30.png)
Editor com a lógica de leitura de atributos PaymentType e OrderID.
![Evidência 31](./images/step-31.png)
Deploy do código da Lambda de pagamento.
![Evidência 32](./images/step-32.png)
Revisão da função implantada com sucesso.
![Evidência 33](./images/step-33.png)
Conferência da função antes de seguir para a próxima Lambda.
![Evidência 34](./images/step-34.png)
### Fase 6 - Lambda de notificação ao cliente
Criação da função lambda-notificacao-cliente.
![Evidência 35](./images/step-35.png)
Definição do runtime e parâmetros básicos da função.
![Evidência 36](./images/step-36.png)
Associação da role de execução para logs no CloudWatch.
![Evidência 37](./images/step-37.png)
Código da Lambda de notificação lendo CustomerEmail e OrderID.
![Evidência 38](./images/step-38.png)
Deploy da função de notificação ao cliente.
![Evidência 39](./images/step-39.png)
Validação da função criada no console Lambda.
![Evidência 40](./images/step-40.png)
### Fase 7 - Lambda de análise de fraude e trigger SQS
Criação da função lambda-analise-fraude.
![Evidência 41](./images/step-41.png)
Seleção da role com permissões de SQS para a análise de fraude.
![Evidência 42](./images/step-42.png)
Código da Lambda de fraude consumindo a mensagem recebida da fila.
![Evidência 43](./images/step-43.png)
Deploy da função de fraude com leitura do TransactionValue.
![Evidência 44](./images/step-44.png)
Tela de configuração do gatilho SQS para a Lambda de fraude.
![Evidência 45](./images/step-45.png)
Vínculo da fila fila-fraude-analise como trigger da função.
![Evidência 46](./images/step-46.png)
Gatilho SQS configurado e pronto para ser habilitado.
![Evidência 47](./images/step-47.png)
### Fase 8 - assinaturas SNS com filtros
Criação da assinatura da Lambda de inventário com protocolo AWS Lambda.
![Evidência 48](./images/step-48.png)
Aplicação da filter policy para OrderPlaced e InventoryCheckRequired.
![Evidência 49](./images/step-49.png)
Configuração da assinatura da Lambda de pagamento.
![Evidência 50](./images/step-50.png)
Filter policy do pagamento filtrando PaymentType para CreditCard e Boleto.
![Evidência 51](./images/step-51.png)
Criação da assinatura da fila de pagamento via protocolo Amazon SQS.
![Evidência 52](./images/step-52.png)
Criação da assinatura da fila de notificação ao cliente.
![Evidência 53](./images/step-53.png)
Configuração da assinatura da fila/função usada na análise de fraude.
![Evidência 54](./images/step-54.png)
Filter policy da fraude com TransactionValue maior que 500.
![Evidência 55](./images/step-55.png)
Confirmação das assinaturas registradas no tópico SNS.
![Evidência 56](./images/step-56.png)
Revisão das assinaturas confirmadas no tópico do e-commerce.
![Evidência 57](./images/step-57.png)
### Fase 9 - publicação do evento e validação
Publicação da mensagem de teste no tópico SNS com atributos do pedido.
![Evidência 58](./images/step-58.png)
Configuração dos atributos OrderPlaced, PaymentType e TransactionValue.
![Evidência 59](./images/step-59.png)
Confirmação da publicação da mensagem no tópico.
![Evidência 60](./images/step-60.png)
Resultado final do fluxo com o tópico e assinaturas consolidadas.
![Evidência 61](./images/step-61.png)
Conferência do comportamento esperado antes da análise de logs.
![Evidência 62](./images/step-62.png)
Validação final do laboratório e do fan-out do e-commerce.
![Evidência 63](./images/step-63.png)


