# Semana do Desenvolvedor EDN

Esta pasta reune os laboratorios da **Semana do Desenvolvedor AWS** da **Escola da Nuvem**, organizados dentro do repositorio central [Hands_On_AWS_Dev](../). A proposta da semana foi construir, na pratica, uma **arquitetura orientada a eventos** para processamento de pedidos e arquivos na AWS.

## O que e a Semana do Desenvolvedor AWS

Trata-se de um evento intensivo com:

- **4 dias focados 100% na pratica**
- Conteudo voltado para alunos do curso **AWS Developer Associate**
- Uma proposta de construir um fluxo completo usando servicos AWS integrados

A promessa da semana foi clara: desenvolver uma arquitetura capaz de **receber, validar, processar e encaminhar pedidos e arquivos** com desacoplamento, resiliencia e escalabilidade.

## O desafio da semana

O projeto parte de um problema bem comum em sistemas modernos:

- Uma empresa precisa gerenciar pedidos de forma eficiente e escalavel
- esses pedidos podem chegar em **tempo real via API**
- ou em **lotes de arquivos via Amazon S3**

Para resolver isso, a semana foi estruturada em torno de uma arquitetura orientada a eventos, preparada para:

- Receber dados de multiplas fontes
- Validar e processar pedidos
- Desacoplar etapas com filas
- Publicar eventos para novos consumidores
- Permitir evolucao gradual do fluxo de negocio

## A arquitetura que vamos construir

Ao longo dos 4 dias, a trilha monta um sistema com estas caracteristicas centrais:

- **Arquitetura orientada a eventos** como base do processamento
- **Recebe, valida e processa** pedidos de multiplas fontes
- **Desacoplamento, resiliencia e escalabilidade** com servicos AWS integrados

No fim da semana, a arquitetura combina:

- Entrada via **API REST**
- Ingestao de arquivos via **Amazon S3**
- Processamento assicrono com **Amazon SQS**
- Validacoes e regras de negocio com **AWS Lambda**
- Roteamento e barramento de eventos com **Amazon EventBridge**
- Notificacoes com **Amazon SNS**
- Persistencia e rastreabilidade com **Amazon DynamoDB**

### Arquitetura completa da semana

![Arquitetura completa da Semana do Desenvolvedor EDN](./arquitetura-completa.png)

## Resultado esperado ao final da semana

Ao concluir a trilha, a ideia e ter uma arquitetura completa com:

- **Ingestao multipla**
  API REST para pedidos em tempo real e S3 para processamento em lote
- **Processamento assicrono**
  SQS para desacoplamento e EventBridge como barramento de eventos
- **Pipeline de validacao**
  Lambdas especializadas para cada etapa do fluxo
- **Rastreabilidade**
  acompanhamento do estado dos pedidos e dos arquivos processados

## Estrutura da semana

| Dia | Pasta | Tema principal | Foco tecnico |
| :--- | :--- | :--- | :--- |
| Dia 1 | [dia-1-api-eventbridge](./dia-1-api-eventbridge) | Ingestao de pedidos via API e EventBridge | API Gateway, AWS Lambda, Amazon SQS FIFO, DLQ e Amazon EventBridge |
| Dia 2 | [dia-2-s3-integracao](./dia-2-s3-integracao) | Ingestao de arquivos via S3 e rastreamento | Amazon S3, Amazon SQS Standard, AWS Lambda, Amazon DynamoDB e Amazon SNS |
| Dia 3 | [dia-3-processamento-pedidos](./dia-3-processamento-pedidos) | Processamento central de pedidos e persistencia | Amazon EventBridge, Amazon SQS, AWS Lambda e Amazon DynamoDB |
| Dia 4 | [dia-4-fluxos-dlq](./dia-4-fluxos-dlq) | Fluxos adicionais de pedidos e DLQs | Regras de negocio, EventBridge, AWS Lambda, Amazon SQS e dead-letter queues |

## Principais funcionalidades por dia

### Dia 1

O primeiro laboratorio constroi a base de entrada em tempo real:

- **API Gateway**
  endpoint funcional para receber pedidos
- **AWS Lambda**
  duas funcoes Lambda para validacao
- **Amazon SQS**
  uma fila SQS FIFO para desacoplar o processamento
- **Amazon EventBridge**
  um barramento de eventos customizado para notificar sobre novos pedidos validados

### Dia 2

O segundo laboratorio adiciona a entrada por arquivos:

- **Amazon S3**
  bucket configurado para receber arquivos JSON de pedidos
- **Amazon SQS**
  duas filas SQS Standard para desacoplar o processamento inicial dos arquivos
- **AWS Lambda**
  funcao para extrair pedidos e enviar para o pipeline principal
- **Amazon DynamoDB**
  tabela destinada ao historico de validacao de arquivos
- **Amazon SNS**
  topico para notificacoes de erro na validacao de arquivos

### Dia 3

O terceiro dia concentra o processamento principal do pedido:

- **Amazon EventBridge**
  roteamento de eventos de novos pedidos validados
- **AWS Lambda**
  execucao da logica central de processamento do pedido
- **Amazon SQS**
  desacoplamento do processamento principal
- **Amazon DynamoDB**
  armazenamento do estado e dos detalhes dos pedidos processados

### Dia 4

O quarto dia amplia os fluxos de negocio e reforca o tratamento de falhas:

- **Fluxos funcionais**
  cancelamento e alteracao de pedidos
- **Amazon EventBridge**
  novas regras para roteamento dos eventos
- **AWS Lambda**
  duas novas funcoes Lambda para validacao
- **Amazon SQS**
  aprofundamento pratico no funcionamento e na importancia das DLQs

## Nosso roteiro diario

Esta trilha foi organizada da seguinte forma:

1. **Dia 1**
   Ingestao de pedidos via API e EventBridge
2. **Dia 2**
   Ingestao de arquivos via S3 e rastreamento
3. **Dia 3**
   Processamento central de pedidos e persistencia
4. **Dia 4**
   Fluxos adicionais de pedidos e DLQs

## Como esta pasta esta organizada

Cada subpasta desta trilha representa um dia da semana e deve conter:

- `README.md` com o contexto e o passo a passo do laboratorio
- pasta `images/` com as evidencias visuais
- arquitetura da etapa
- servicos utilizados
- secoes de validacao e limpeza dos recursos

## Status atual

- **Dia 1** documentado com arquitetura, explicacao tecnica e evidencias
- **Dia 2** documentado com arquitetura, explicacao tecnica e evidencias
- **Dia 3** documentado com arquitetura, explicacao tecnica e evidencias
- **Dia 4** documentado com arquitetura, explicacao tecnica e evidencias
