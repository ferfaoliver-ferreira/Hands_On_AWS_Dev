# DynamoDB com LSI, GSI e Consultas Otimizadas

## Objetivo

Este laboratório tem como objetivo demonstrar a criação e utilização de tabelas Amazon DynamoDB com índices secundários locais (LSI) e índices secundários globais (GSI), além da realização de consultas otimizadas utilizando diferentes estratégias de acesso aos dados.

Ao final da atividade será possível compreender as diferenças entre operações Scan e Query, além dos benefícios do uso de índices secundários para melhorar a eficiência das consultas.

## Arquitetura

A arquitetura utilizada neste laboratório é composta por:

- Amazon DynamoDB;
- Local Secondary Index (LSI);
- Global Secondary Index (GSI);
- AWS CloudShell;
- AWS CLI;
- Operações Scan e Query.

<img width="564" height="484" alt="Captura de tela 2026-05-22 234205" src="https://github.com/user-attachments/assets/414555cd-d3e8-4645-85b7-af4a7ea28143" />


### Fluxo da Solução

1. Os dados são armazenados na tabela principal do DynamoDB.
2. O Local Secondary Index (LSI) permite consultas alternativas utilizando a mesma chave de partição da tabela.
3. O Global Secondary Index (GSI) possibilita novos padrões de acesso utilizando chaves diferentes da tabela principal.
4. As consultas podem ser realizadas diretamente na tabela ou através dos índices, reduzindo a necessidade de operações Scan.
5. O AWS CloudShell é utilizado para inserção de dados e execução dos comandos AWS CLI.

## Pré-requisitos

- Conta AWS fornecida pela Escola da Nuvem;
- Acesso ao Console AWS;
- Permissões para utilização do DynamoDB;
- Permissões para utilização do CloudShell;
- Conhecimentos básicos sobre bancos NoSQL.

---

# Parte 1 - Criação da Tabela DynamoDB

## Passo 1 - Acessar o serviço DynamoDB

Após realizar login no Console AWS, foi acessado o serviço **Amazon DynamoDB**.

<img width="1902" height="943" alt="Captura de tela 2026-05-22 212020" src="https://github.com/user-attachments/assets/22de2f9c-e766-4c6d-8569-ae2ccebe83d4" />


---

## Passo 2 - Criar uma nova tabela

Foi selecionada a opção **Criar tabela** para iniciar a configuração da base de dados.

Foram definidos:

- Nome da tabela: Pedido-<NomeSobrenome>
- Chave de partição: ID do Usuário
- Tipo: String
- Chave de classificação: Data do Pedido
- Tipo: String

<img width="1902" height="944" alt="Captura de tela 2026-05-22 212143" src="https://github.com/user-attachments/assets/e6394c90-0adc-412a-9539-1bd02041aaad" />


---

## Passo 3 - Configurar capacidade provisionada

Foram personalizadas as configurações da tabela utilizando:

- Classe DynamoDB Standard;
- Capacidade Provisionada;
- 5 unidades de leitura;
- 5 unidades de gravação;
- Auto Scaling desabilitado.

<img width="1900" height="889" alt="Captura de tela 2026-05-22 212414" src="https://github.com/user-attachments/assets/9004f751-30a5-4aa1-a57e-981d25e567e0" />


---

## Passo 4 - Criar o índice secundário local (LSI)

Foi criado um índice secundário local com as seguintes configurações:

- Chave de classificação: Status
- Tipo: String
- Nome do índice: LSI-Pedido-Status
- Projeção de atributos: All

<img width="1900" height="944" alt="Captura de tela 2026-05-22 213025" src="https://github.com/user-attachments/assets/4c159469-8748-49a8-85f5-2716fb3f68eb" />


---

## Passo 5 - Validar a criação da tabela

Após concluir as configurações, a tabela foi criada e validada com sucesso.

<img width="1907" height="890" alt="Captura de tela 2026-05-22 214008" src="https://github.com/user-attachments/assets/cc8ea549-f231-4e62-80e5-b3e34c2f8779" />


---

# Parte 2 - Inserção de Dados Manualmente

## Passo 6 - Acessar a tabela criada

Foi selecionada a tabela recém-criada para iniciar o processo de inserção de itens.

<img width="1908" height="890" alt="Captura de tela 2026-05-22 214031" src="https://github.com/user-attachments/assets/87987d7e-69ab-4c6c-b3b8-d345f185e031" />


---

## Passo 7 - Abrir a opção Explorar Itens

Foi acessada a funcionalidade **Explorar itens da tabela**.

<img width="1909" height="890" alt="Captura de tela 2026-05-22 214136" src="https://github.com/user-attachments/assets/1b9f9bd9-b96d-4de6-82bc-1ab1fa4afe5d" />


---

## Passo 8 - Criar um novo item

Foi iniciado o cadastro manual de um pedido contendo os atributos definidos no laboratório.

<img width="1914" height="890" alt="Captura de tela 2026-05-22 214235" src="https://github.com/user-attachments/assets/3e01ff0a-07d4-4f7d-aa2a-810813cfb67c" />


---

## Passo 9 - Configurar atributos compostos

Foram adicionados os atributos:

### Endereco (Mapa)

- bairro
- cidade
- estado
- numero
- rua

### tags (Lista)

- Entrega zona rural
- Equipe Rocket
- Recife

Além dos atributos:

- IDPedido
- Status
- ValorTotal

<img width="1902" height="883" alt="Captura de tela 2026-05-22 214558" src="https://github.com/user-attachments/assets/c500d6d3-aa2e-4fd3-bbed-2157b738d69f" />


---

## Passo 10 - Validar o item criado

Foi realizada a validação do item inserido na tabela.

<img width="1905" height="895" alt="Captura de tela 2026-05-22 214657" src="https://github.com/user-attachments/assets/82758137-a242-4cb0-916b-233e18b40218" />


---

# Parte 3 - Importação de Dados via CloudShell

## Passo 11 - Abrir o CloudShell

Foi acessado o ambiente AWS CloudShell para execução de comandos via terminal.

<img width="1906" height="891" alt="Captura de tela 2026-05-22 214915" src="https://github.com/user-attachments/assets/613a3869-afb3-4a17-95c6-8f336da7bb70" />


---

## Passo 12 - Preparar arquivo de importação

Foi realizado o download e edição do arquivo `pedidos_import.json`, alterando o nome da tabela para o recurso criado durante o laboratório.

<img width="1909" height="936" alt="Captura de tela 2026-05-22 215404" src="https://github.com/user-attachments/assets/1b30e2c8-0b6a-4d84-a7b5-3e48a80e78c5" />


---

## Passo 13 - Realizar upload do arquivo para o CloudShell

Foi utilizado o menu **Ações → Carregar Arquivo** para enviar o arquivo JSON para o ambiente CloudShell.

<img width="1905" height="891" alt="Captura de tela 2026-05-22 215525" src="https://github.com/user-attachments/assets/d0e03b0c-46d8-4fd7-839b-c7a57757308c" />


---

## Passo 14 - Confirmar upload do arquivo

Foi utilizado o comando:

```bash
ls
```

para validar que o arquivo foi enviado corretamente para o ambiente.

<img width="1905" height="891" alt="Captura de tela 2026-05-22 215525" src="https://github.com/user-attachments/assets/1e6b5a36-7e73-4db4-98c3-556b2b4b8ecc" />


---

## Passo 15 - Importar os dados para o DynamoDB

Foi executado o comando:

```bash
aws dynamodb batch-write-item --request-items file://pedidos_import.json
```

para importar os registros para a tabela DynamoDB.

<img width="1908" height="900" alt="Captura de tela 2026-05-22 220114" src="https://github.com/user-attachments/assets/db0af51c-36ec-4378-87b4-4059ef321659" />


---

## Passo 16 - Validar os dados importados

Após a importação, a tabela foi atualizada e os novos registros foram exibidos.

<img width="1905" height="888" alt="Captura de tela 2026-05-22 220557" src="https://github.com/user-attachments/assets/5fff42ae-1558-4f60-be3e-d548f6709660" />


---

# Parte 4 - Consulta utilizando Scan

## Passo 17 - Configurar operação Verificar (Scan)

Foi utilizada a aba **Verificar**, configurando um filtro para buscar registros do usuário U001.

<img width="1903" height="885" alt="Captura de tela 2026-05-22 220637" src="https://github.com/user-attachments/assets/ad526596-e814-4063-b635-4faeb33f1d63" />

---

## Passo 18 - Executar Scan e analisar eficiência

A operação percorreu toda a tabela para localizar os registros correspondentes.

Foi possível observar uma eficiência reduzida, característica das operações Scan.

<img width="1909" height="891" alt="Captura de tela 2026-05-22 220713" src="https://github.com/user-attachments/assets/7b4dd00a-0193-47bf-8e59-fbd68757184a" />


---

# Parte 5 - Consulta utilizando Query

## Passo 19 - Configurar operação Consulta (Query)

Foi utilizada a chave de partição para realizar uma consulta direta aos registros.

<img width="1909" height="888" alt="Captura de tela 2026-05-22 220844" src="https://github.com/user-attachments/assets/84508a16-1099-40c6-af72-310776f2acc2" />


---

## Passo 20 - Comparar eficiência da Query

A operação Query retornou apenas os registros correspondentes ao critério informado, apresentando eficiência significativamente superior.

<img width="1905" height="940" alt="Captura de tela 2026-05-22 222607" src="https://github.com/user-attachments/assets/2cee8d4b-349d-4be5-91d4-5a9fe40c6b20" />


---

# Parte 6 - Consultas utilizando LSI

## Passo 21 - Consultar dados utilizando o índice local

Foi selecionado o índice secundário local criado anteriormente para filtrar pedidos por Status.

<img width="1905" height="892" alt="Captura de tela 2026-05-22 222753" src="https://github.com/user-attachments/assets/74b99f3a-3def-4580-87af-c12c5d4a0830" />


---

## Passo 22 - Aplicar filtros adicionais

Foi realizada uma consulta utilizando filtros adicionais para ValorTotal.

<img width="1911" height="893" alt="Captura de tela 2026-05-22 222851" src="https://github.com/user-attachments/assets/64f3409a-ab45-4ede-b811-2f21700dfe13" />


---

# Parte 7 - Criação e Utilização do GSI

## Passo 23 - Acessar a aba de índices

Foi acessada a aba **Índices** da tabela para criação de um índice secundário global.

<img width="1870" height="854" alt="Captura de tela 2026-05-22 223703" src="https://github.com/user-attachments/assets/257bcaf3-a601-44a7-bb2b-e445e6be73a1" />


---

## Passo 24 - Criar o índice global (GSI)

Foi criado o índice:

- Chave de partição: Status
- Chave de classificação: ValorTotal
- Nome: GSI-Status-ValorTotal

![Captura de tela 2026-05-22 234205](./Captura%20de%20tela%202026-05-22%20234205.png)

---

## Passo 25 - Consultar dados utilizando o GSI

Após a criação do índice global, foram realizadas consultas utilizando:

- Status = Entregue
- ValorTotal ≥ 60

obtendo resultados otimizados diretamente pelo índice.

<img width="1910" height="937" alt="Captura de tela 2026-05-22 235918" src="https://github.com/user-attachments/assets/4e8aeaeb-d991-4eb6-8bee-88ccec6db27a" />


---

## Passo 26 - Validar o retorno da consulta

Foi confirmada a exibição dos registros utilizando o índice secundário global.

<img width="1909" height="944" alt="Captura de tela 2026-05-22 235942" src="https://github.com/user-attachments/assets/37637489-06ec-43ac-8bdb-71a6e8aadadc" />


---

# Importância dos Índices Secundários

Os índices secundários do DynamoDB permitem criar diferentes padrões de acesso aos dados sem alterar a estrutura principal da tabela.

### Benefícios do LSI

- Utiliza a mesma chave de partição da tabela;
- Permite diferentes critérios de ordenação;
- Mantém alta eficiência de leitura;
- Ideal para consultas relacionadas ao mesmo conjunto de partições.

### Benefícios do GSI

- Permite criar novas chaves de partição;
- Possibilita novos padrões de consulta;
- Reduz operações Scan;
- Melhora significativamente a performance da aplicação.

---

# Limpeza dos Recursos

Ao final do laboratório recomenda-se:

1. Excluir o índice global criado;
2. Excluir a tabela DynamoDB;
3. Encerrar sessões do CloudShell;
4. Remover arquivos temporários utilizados na importação.

---

# Conclusão

Neste laboratório foram explorados os principais recursos do Amazon DynamoDB relacionados à modelagem e otimização de consultas.

Foram realizadas as seguintes atividades:

- Criação de tabela DynamoDB;
- Configuração de chave de partição e classificação;
- Criação de índice secundário local (LSI);
- Inserção manual de dados;
- Importação de dados via CloudShell;
- Consultas utilizando Scan;
- Consultas utilizando Query;
- Utilização de filtros avançados;
- Criação de índice secundário global (GSI);
- Consultas otimizadas utilizando índices;
- Comparação de eficiência entre Scan e Query.

Ao final da atividade foi possível compreender como os índices secundários locais e globais contribuem para a construção de aplicações escaláveis, eficientes e com melhor desempenho de leitura no Amazon DynamoDB.
