# Laboratório - Automatizando o Fim das Instâncias na AWS

## Objetivo

Este laboratório tem como objetivo demonstrar a criação de uma solução automatizada para encerramento de instâncias Amazon EC2 utilizando AWS Lambda, IAM e Amazon EventBridge.

Ao final da atividade, será possível criar uma política de permissões, configurar uma função Lambda em Python para encerrar instâncias EC2 automaticamente e agendar sua execução utilizando o EventBridge.

## Arquitetura

A solução utiliza os seguintes serviços AWS:

- AWS IAM para gerenciamento de permissões;
- AWS Lambda para execução do código de automação;
- Amazon EC2 como recurso a ser gerenciado;
- Amazon EventBridge para agendamento da execução automática;
- Amazon CloudWatch Logs para registro dos logs de execução.

Fluxo da arquitetura:

```text
EventBridge
     │
     ▼
AWS Lambda
     │
     ▼
Amazon EC2
```

## Pré-requisitos

- Conta AWS fornecida pela Escola da Nuvem;
- Permissões para utilização dos serviços IAM, Lambda e EventBridge;
- Arquivo Terminator.zip fornecido junto ao laboratório;
- Conhecimentos básicos sobre AWS Console.

---

# Parte 1 - Criação da Política IAM

## Passo 1 - Criar política de permissões para encerramento de instâncias

Foi acessado o serviço IAM e selecionada a opção **Políticas**. Em seguida, foi iniciado o processo de criação de uma nova política.

<img width="1896" height="929" alt="Captura de tela 2026-05-08 200937" src="https://github.com/user-attachments/assets/4697b565-7808-4cb1-8b68-59690df1f1ed" />


---

## Passo 2 - Configurar permissões em JSON

Foi selecionada a opção **JSON** e inserida a política responsável por conceder permissões para:

- Encerrar instâncias EC2;
- Consultar instâncias;
- Consultar regiões;
- Criar e registrar logs no CloudWatch.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogStream",
        "logs:CreateLogGroup",
        "logs:PutLogEvents",
        "ec2:DescribeInstances",
        "ec2:TerminateInstances",
        "ec2:DescribeRegions"
      ],
      "Resource": "*"
    }
  ]
}
```

<img width="1912" height="937" alt="Captura de tela 2026-05-08 201142" src="https://github.com/user-attachments/assets/5d53358c-cdb4-4ab6-9ed7-9be08002f0ff" />


---

## Passo 3 - Definir o nome da política

Foi informado um nome seguindo o padrão solicitado pelo laboratório:

```text
PoliticaTerminarEC2-NomeSobrenome
```

Após a validação das configurações, a política foi criada.

<img width="1891" height="923" alt="Captura de tela 2026-05-08 201506" src="https://github.com/user-attachments/assets/40fdb640-9ae2-4f39-80ba-f2e4893faef6" />


---

# Parte 2 - Criação da Role IAM

## Passo 4 - Criar uma função (Role) para o AWS Lambda

No serviço IAM foi acessada a seção **Funções** e selecionada a opção **Criar perfil**.

<img width="1909" height="940" alt="Captura de tela 2026-05-08 201707" src="https://github.com/user-attachments/assets/5510cdd5-d8c2-4e46-a713-a5821953d3ab" />


---

## Passo 5 - Configurar entidade confiável

Foi selecionado:

- Serviço da AWS;
- Lambda.

Essa configuração permite que a função Lambda utilize as permissões atribuídas à role.

<img width="1912" height="911" alt="Captura de tela 2026-05-08 201801" src="https://github.com/user-attachments/assets/78dcbb0c-ce81-4be5-a98c-ca59fc9b6f4b" />


---

## Passo 6 - Associar a política criada

Foi pesquisada e selecionada a política criada anteriormente para conceder permissões à função Lambda.

<img width="1885" height="918" alt="Captura de tela 2026-05-08 202135" src="https://github.com/user-attachments/assets/ad60d353-ce8d-4aa6-9f2b-a236d29a739b" />


---

## Passo 7 - Finalizar criação da Role

Foi definido um nome seguindo o padrão:

```text
RoleTerminarEC2-NomeSobrenome
```

Após a configuração, a role foi criada.

<img width="1911" height="900" alt="Captura de tela 2026-05-08 202212" src="https://github.com/user-attachments/assets/5230aad4-1477-40f8-8d9c-c8c954ab2640" />


---

# Parte 3 - Criação da Função Lambda

## Passo 8 - Criar a função Lambda

Foi acessado o serviço AWS Lambda e selecionada a opção **Criar função**.

Foram utilizadas as seguintes configurações:

- Criar do zero;
- Runtime Python 3.13;
- Nome da função:
  `LambdaTerminarEC2-NomeSobrenome`;
- Role de execução:
  `RoleTerminarEC2-NomeSobrenome`.

<img width="1899" height="928" alt="Captura de tela 2026-05-08 202513" src="https://github.com/user-attachments/assets/ad6dcff4-84d7-4512-be49-a9a452084f86" />


---

## Passo 9 - Validar a criação da função

Após a criação, foi realizada a validação das configurações iniciais da função Lambda.

<img width="1894" height="915" alt="Captura de tela 2026-05-08 204149" src="https://github.com/user-attachments/assets/29c37630-b6c2-4a88-9fa8-54efd6506475" />

---

## Passo 10 - Fazer upload do código da automação

Na seção **Origem do código**, foi utilizada a opção **Fazer upload de → Arquivo ZIP** para carregar o script Terminator.

<img width="1895" height="930" alt="Captura de tela 2026-05-08 204513" src="https://github.com/user-attachments/assets/0f75267e-3382-44d8-800c-925ad73d5d84" />

---

## Passo 11 - Selecionar o arquivo Terminator.zip

Foi selecionado o arquivo compactado contendo o código responsável pela automação do encerramento das instâncias EC2.

<img width="1912" height="981" alt="Captura de tela 2026-05-08 205102" src="https://github.com/user-attachments/assets/7f644b04-2b3e-459c-937a-9d971bd6b6e3" />


---

## Passo 12 - Validar o código carregado e realizar Deploy

Após o upload, foi realizada a validação do arquivo Terminator.py e executado o deploy da função.

<img width="1903" height="947" alt="Captura de tela 2026-05-08 205622" src="https://github.com/user-attachments/assets/8f3bfaed-e15f-4e4f-b6af-79c3703d4cba" />


<img width="1908" height="948" alt="Captura de tela 2026-05-08 210215" src="https://github.com/user-attachments/assets/240b8cc7-c8bb-49c9-b1c7-128bd41755d7" />


---

# Parte 4 - Configuração da Função Lambda

## Passo 13 - Acessar as configurações da função

Após o deploy do código, foi acessada a aba **Configuração** da função Lambda para realizar ajustes adicionais necessários para a execução correta da automação.

<img width="1911" height="942" alt="Captura de tela 2026-05-08 210501" src="https://github.com/user-attachments/assets/b1485d87-ef7d-48f7-88da-b8f0e7652f90" />


---

## Passo 14 - Alterar o tempo limite de execução

Na seção **Configuração geral**, foi alterado o campo **Tempo limite (Timeout)** de 3 segundos para 10 segundos.

Essa alteração garante que a função tenha tempo suficiente para consultar e encerrar instâncias EC2 sem ser interrompida prematuramente.

<img width="1895" height="539" alt="Captura de tela 2026-05-08 210518" src="https://github.com/user-attachments/assets/676c8bef-3f32-4ca9-94ff-948673be13f1" />

---

## Passo 15 - Configurar o Handler da função

Na seção **Configurações de tempo de execução**, foi configurado o campo **Manipulador (Handler)** com o valor:

```text
Terminator.lambda_handler
```

Essa configuração informa ao AWS Lambda qual arquivo e qual função devem ser executados quando a função for acionada.

<img width="1905" height="950" alt="Captura de tela 2026-05-08 210557" src="https://github.com/user-attachments/assets/998b0600-0ba8-4c37-810e-887a232e3b7b" />


---

# Parte 5 - Configuração do Agendamento com EventBridge

## Passo 16 - Adicionar um gatilho à função Lambda

Foi utilizada a opção **Adicionar gatilho** para permitir que a função seja executada automaticamente através de eventos programados.

<img width="1906" height="361" alt="Captura de tela 2026-05-08 210608" src="https://github.com/user-attachments/assets/f9959383-b519-4ffc-8d72-5e7b6b2889c4" />


---

## Passo 17 - Configurar o Amazon EventBridge

Na configuração do gatilho, foi selecionado o serviço **Amazon EventBridge**.

Foi criada uma nova regra para controlar quando a função Lambda será executada automaticamente.

As configurações utilizadas foram:

```text
Nome da regra:
GatilhoTerminarEC2-NomeSobrenome
```

Tipo de regra:

```text
Expressão de agendamento
```

Exemplo de execução a cada 12 horas:

```text
rate(12 hours)
```

Exemplo de execução a cada 5 minutos:

```text
rate(5 minutes)
```

<img width="1897" height="945" alt="Captura de tela 2026-05-08 210701" src="https://github.com/user-attachments/assets/0b54abc4-3701-47a9-92e3-d0f535c7d04e" />


---

## Passo 18 - Finalizar a criação da regra

Após configurar a expressão de agendamento, foi concluída a criação da regra do EventBridge.

A partir desse momento, a função Lambda passa a ser executada automaticamente de acordo com o intervalo definido.

<img width="1900" height="942" alt="Captura de tela 2026-05-08 210804" src="https://github.com/user-attachments/assets/42019b9d-d2db-4bd9-850e-ee5f6dfacc37" />


---

# Parte 6 - Teste da Automação

## Passo 19 - Validar a execução automática

Com a regra criada, a função Lambda passa a ser acionada automaticamente pelo EventBridge.

Quando uma instância EC2 estiver em execução e o agendamento for atingido, a função executará a ação de encerramento utilizando as permissões concedidas pela política IAM criada anteriormente.

A execução pode ser acompanhada através dos logs gerados automaticamente no Amazon CloudWatch.

---

# Parte 7 - Limpeza dos Recursos

## Passo 20 - Excluir a função Lambda

Após a conclusão dos testes, foi acessada a lista de funções Lambda.

A função criada durante o laboratório foi selecionada e removida através da opção **Ações → Excluir**.

<img width="1905" height="934" alt="Captura de tela 2026-05-08 210919" src="https://github.com/user-attachments/assets/f79fdb16-d275-4249-a746-d302f9c77acb" />

---

## Passo 21 - Excluir a Role IAM

Foi acessado o serviço IAM e localizada a role criada para a função Lambda.

Após a seleção da role, foi realizada sua exclusão.

<img width="1905" height="934" alt="Captura de tela 2026-05-08 210932" src="https://github.com/user-attachments/assets/6ddda3a9-ad9e-4dde-9aac-50d203ae6e56" />


---

## Passo 22 - Excluir a política IAM

Foi localizada a política criada para permitir o encerramento das instâncias EC2.

Após a confirmação, a política foi removida da conta AWS.

<img width="1908" height="936" alt="Captura de tela 2026-05-08 211015" src="https://github.com/user-attachments/assets/30749c68-2bd9-4c5e-b198-7e6bc8f8a38d" />


---

## Passo 23 - Excluir a regra do EventBridge

Por fim, foi acessado o serviço Amazon EventBridge para localizar e excluir a regra de agendamento criada durante o laboratório.

Dessa forma, todos os recursos relacionados à automação foram removidos.

<img width="1907" height="552" alt="Captura de tela 2026-05-09 132533" src="https://github.com/user-attachments/assets/afe96318-5cd8-4e99-b161-238e2e5dae1e" />


---

# Arquitetura da Solução

A arquitetura implementada durante o laboratório segue o fluxo abaixo:

```text
Amazon EventBridge
        │
        ▼
AWS Lambda
        │
        ▼
Amazon EC2
```

Fluxo de execução:

1. O EventBridge executa a regra agendada.
2. A regra dispara a função Lambda.
3. A função Lambda consulta as instâncias EC2.
4. As instâncias encontradas são encerradas automaticamente.
5. Os logs da execução são registrados no Amazon CloudWatch.

---

# Benefícios da Solução

A utilização dessa arquitetura proporciona diversas vantagens:

- Automação de tarefas operacionais;
- Redução de custos com recursos esquecidos em execução;
- Utilização de serviços serverless;
- Centralização dos logs de execução;
- Aplicação de boas práticas de IAM;
- Integração entre múltiplos serviços AWS;
- Escalabilidade automática da solução.

---

# Conclusão

Neste laboratório foram explorados conceitos fundamentais de automação na AWS utilizando serviços serverless.

Durante a atividade foram realizados os seguintes procedimentos:

- Criação de uma política IAM personalizada;
- Criação de uma Role IAM para o AWS Lambda;
- Configuração de permissões para gerenciamento de instâncias EC2;
- Criação de uma função Lambda utilizando Python;
- Upload e implantação de código através de arquivo ZIP;
- Configuração de Timeout e Handler da função;
- Criação de um gatilho utilizando Amazon EventBridge;
- Agendamento automático da execução da função;
- Encerramento automatizado de instâncias EC2;
- Monitoramento através do Amazon CloudWatch;
- Remoção de todos os recursos criados ao final do laboratório.

Ao concluir este laboratório foi possível compreender como integrar IAM, Lambda, EventBridge e EC2 para construir soluções automatizadas capazes de reduzir custos operacionais e simplificar o gerenciamento de recursos em ambientes AWS.
