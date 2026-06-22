# Hands_On_AWS_Dev

Repositorio central de laboratorios praticos na AWS, com foco em arquitetura serverless, integracao entre servicos, observabilidade, seguranca e boas praticas para quem esta estudando desenvolvimento em nuvem.

## Objetivo do repositorio

Este repositorio reune projetos hands-on voltados para a trilha de **AWS Developer**, organizando cada laboratorio em uma pasta propria com:

- documentacao tecnica
- arquitetura
- evidencias visuais
- servicos utilizados
- aprendizados consolidados

## Trilhas e laboratorios

Cada subpasta representa um laboratorio completo ou uma trilha tematica dentro da jornada de estudos.

| Laboratorio | Descricao | Servicos |
| :--- | :--- | :--- |
| [Serverless Guessing Game](./Serverless_Guessing_Game_AWS) | Jogo serverless com frontend estatico e backend desacoplado para praticar integracao e processamento via Lambda. | AWS Lambda, API Gateway, Amazon S3 |
| [AWS Resilient Architecture](./AWS-resilient-architecture-sns-sqs) | Arquitetura desacoplada e resiliente com publicacao, assinaturas e filas para tolerancia a falhas. | Amazon SNS, Amazon SQS |
| [Fan-Out para E-commerce](./AWS-fanout-sns-sqs-lambda-ecommerce) | Fluxo orientado a eventos para e-commerce com filtros, filas, Lambdas e DLQ. | Amazon SNS, Amazon SQS, AWS Lambda, IAM, CloudWatch Logs |
| [AWS DynamoDB LSI/GSI Query](./AWS-dynamodb-lsi-gsi-query) | Estudos práticos com modelagem e consultas em DynamoDB usando indices locais e globais. | Amazon DynamoDB |
| [AWS S3 Versioning, Lifecycle e Access Logs](./AWS-s3-versioning-lifecycle-accesslogs) | Laboratorio de governanca e gerenciamento de objetos com versionamento, regras de ciclo de vida e logs. | Amazon S3 |
| [AWS Lambda Aliases, API Gateway e Stages](./AWS-lambda-aliases-api-gateway-stages) | Estrategias de versionamento e deploy para APIs serverless. | AWS Lambda, Amazon API Gateway |
| [AWS SSM Parameter Store e KMS](./AWS-ssm-parameter-store-kms-cli) | Protecao e uso de configuracoes sensiveis com Parameter Store e criptografia. | AWS Systems Manager, AWS KMS |
| [AWS FinOps EC2 Automation](./AWS-finops-ec2-automation) | Automacao e boas praticas de custo em workloads com EC2. | Amazon EC2, automacao AWS |
| [EC2 Console vs CLI](./02-ec2-console-vs-cli) | Comparacao pratica entre operacoes via console e linha de comando. | Amazon EC2, AWS CLI |

## Organizacao esperada dos projetos

Cada laboratorio ou trilha deve conter, sempre que possivel:

- `README.md` com contexto, objetivo, servicos e passo a passo
- pasta `images/` com capturas e evidencias
- arquitetura ilustrada
- secao de limpeza de recursos

## Observacao sobre custos

Todos os laboratorios devem ser encerrados com limpeza dos recursos criados. Esse cuidado ajuda a consolidar boas praticas de operacao e evita custos desnecessarios na conta AWS.
