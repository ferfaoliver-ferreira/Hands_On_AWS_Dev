# Laboratório - Criando um Orçamento na AWS com Budget

## Objetivo

Este laboratório tem como objetivo demonstrar a criação de um orçamento utilizando o serviço AWS Budgets, permitindo monitorar os gastos da conta AWS e receber notificações automáticas quando determinados limites financeiros forem atingidos.

## Arquitetura

O AWS Budgets monitora os custos da conta AWS e envia notificações por e-mail quando os limites configurados são alcançados.

## Pré-requisitos

- Conta AWS fornecida pela Escola da Nuvem
- Acesso ao Console de Gerenciamento da AWS
- Endereço de e-mail válido
- Permissões para acessar o serviço AWS Budgets

---

# Parte 1 - Criação do Orçamento

## Passo 1 - Acessar o serviço AWS Budgets

Após realizar login no Console de Gerenciamento da AWS, foi acessado o serviço **AWS Budgets**. Em seguida, foi selecionada a opção **Criar orçamento** e escolhida a modalidade **Personalizar (Avançado)** para criação de um orçamento customizado.

<img width="1897" height="928" alt="Captura de tela 2026-05-06 204547" src="https://github.com/user-attachments/assets/00673b4c-7cd1-4126-aa3e-d875fa170515" />


---

## Passo 2 - Definir o nome do orçamento

No campo **Nome do orçamento**, foi informado um identificador seguindo o padrão solicitado pelo laboratório.

Exemplo:

```text
meu-budget-FernandaOliveira
```

Após preencher o nome, foi iniciada a configuração do orçamento.

<img width="1898" height="894" alt="Captura de tela 2026-05-06 204615" src="https://github.com/user-attachments/assets/7643da47-b821-4a5d-975f-f4c2b3f47cd8" />


---

# Parte 2 - Configuração do Budget

## Passo 3 - Definir o valor do orçamento

Na etapa de configuração financeira foi informado o valor de **US$ 10**, que será utilizado como limite máximo para monitoramento dos custos da conta AWS.

Após preencher o valor, foi selecionada a opção para avançar para a próxima etapa.

<img width="1894" height="931" alt="Captura de tela 2026-05-06 205015" src="https://github.com/user-attachments/assets/78985948-1646-4530-9f27-4e38cdee8e35" />

---

## Passo 4 - Configurar alerta de orçamento

Foi criado um alerta para monitorar o consumo do orçamento.

As configurações utilizadas foram:

- Limite: 10
- Tipo: % do valor orçado
- Acionador: Real
- Destinatário: endereço de e-mail cadastrado

Após preencher todas as informações, foi concluída a criação do orçamento.

<img width="1895" height="941" alt="Captura de tela 2026-05-06 205121" src="https://github.com/user-attachments/assets/09bd5c5a-2337-4d2e-a3be-7db59fa49b25" />


---

# Parte 3 - Validação do Orçamento

## Passo 5 - Validar o orçamento criado

Após a criação do orçamento, foi realizada a validação na tela principal do AWS Budgets.

Foram verificadas as seguintes informações:

- Nome do orçamento;
- Valor orçado;
- Valor utilizado;
- Percentual consumido;
- Status do orçamento.

A validação confirmou que o orçamento foi criado corretamente e encontra-se ativo para monitoramento dos custos da conta.

<img width="1912" height="336" alt="Captura de tela 2026-05-06 205648" src="https://github.com/user-attachments/assets/5e91052a-f050-48a6-8ba0-038783b918e5" />


---

# Importância do AWS Budgets

A configuração de um orçamento é uma prática essencial para gerenciamento financeiro na nuvem.

O AWS Budgets permite:

- Monitorar gastos em tempo real;
- Receber alertas automáticos por e-mail;
- Identificar recursos que estejam gerando custos;
- Evitar cobranças inesperadas;
- Adotar ações corretivas antes de ultrapassar limites financeiros.

Essa abordagem transforma o gerenciamento de custos de um processo reativo para um processo proativo.

---

# Conclusão

Neste laboratório foram explorados os principais recursos do AWS Budgets, incluindo:

- Criação de orçamentos personalizados;
- Definição de limites financeiros;
- Configuração de alertas automáticos;
- Configuração de notificações por e-mail;
- Monitoramento de custos da conta AWS;
- Validação das configurações realizadas.

Ao final da atividade foi possível compreender como o AWS Budgets auxilia no controle financeiro dos recursos em nuvem, permitindo acompanhar os gastos em tempo real e receber notificações preventivas antes que os custos ultrapassem os limites definidos.
