# Laboratório - AWS Lambda com Aliases e API Gateway com Stages

## Objetivo

Este laboratório tem como objetivo demonstrar a utilização de versionamento de funções AWS Lambda por meio de **Versions** e **Aliases**, além da integração com o **Amazon API Gateway** utilizando **Stages** para representar ambientes distintos, como Desenvolvimento e Produção.

Ao final do laboratório, será possível acessar diferentes versões da mesma função Lambda através de endpoints específicos do API Gateway.

## Arquitetura

A arquitetura implementada utiliza:

- AWS Lambda para execução do código serverless;
- Versionamento de funções Lambda;
- Aliases para representar ambientes;
- Amazon API Gateway REST API;
- Stages de implantação para Desenvolvimento e Produção;
- Integração Proxy entre API Gateway e Lambda.

## Pré-requisitos

- Conta AWS fornecida pela Escola da Nuvem;
- Acesso ao Console AWS;
- Permissões para criação de funções Lambda;
- Permissões para criação de APIs no API Gateway;
- Navegador Web atualizado.

---

# Parte 1 - Criação da Função Lambda

## Passo 1 - Acessar o serviço AWS Lambda

Após realizar login no Console AWS, foi acessado o serviço **AWS Lambda**.

<img width="1906" height="935" alt="Captura de tela 2026-05-18 210548" src="https://github.com/user-attachments/assets/5cef0fdd-7e5e-416a-82da-93f862cb1ae0" />


---

## Passo 2 - Criar uma nova função Lambda

Foi selecionada a opção **Criar função**, escolhendo:

- Criar do zero;
- Runtime Python 3.13;
- Nome seguindo o padrão solicitado pelo laboratório;
- Criação automática da Role de execução.

<img width="1901" height="940" alt="Captura de tela 2026-05-18 210833" src="https://github.com/user-attachments/assets/7e6ffe3e-8ebd-4dc8-a944-2c1f76d20edc" />


---

## Passo 3 - Configurar a função Lambda

Foram preenchidas as configurações iniciais da função e criado o recurso.

<img width="1904" height="942" alt="Captura de tela 2026-05-18 210914" src="https://github.com/user-attachments/assets/21551ec4-43dd-4ce8-84ee-ef99ed96b80f" />


---

## Passo 4 - Inserir o código da versão de desenvolvimento

Na seção **Código**, foi substituído o conteúdo padrão pelo código fornecido no roteiro do laboratório para representar o ambiente de desenvolvimento.

<img width="1890" height="939" alt="Captura de tela 2026-05-18 212354" src="https://github.com/user-attachments/assets/8ceb865e-49eb-449d-954d-6c90ea3d7731" />


---

## Passo 5 - Realizar o deploy da função

Após inserir o código, foi executado o deploy da função Lambda.

<img width="1908" height="930" alt="Captura de tela 2026-05-18 212534" src="https://github.com/user-attachments/assets/0905dba7-c526-40a1-9977-2d70f9b7f894" />


---

# Parte 2 - Teste e Versionamento da Função

## Passo 6 - Criar evento de teste

Foi criado um evento de teste utilizando o modelo **API Gateway AWS Proxy**.

<img width="1904" height="941" alt="Captura de tela 2026-05-18 212606" src="https://github.com/user-attachments/assets/8c5283ad-d545-40b3-8e0b-813aa1557ac8" />


---

## Passo 7 - Executar o teste da função

A função foi executada utilizando o evento criado para validar o retorno esperado.

<img width="1909" height="937" alt="Captura de tela 2026-05-18 212659" src="https://github.com/user-attachments/assets/95ddf1ef-ab47-4bce-85e1-ebf7ae5662e6" />


---

## Passo 8 - Publicar a Versão 1

Foi acessada a aba **Versions** e publicada a primeira versão da função Lambda.

<img width="1906" height="941" alt="Captura de tela 2026-05-18 212743" src="https://github.com/user-attachments/assets/ea7198f0-a619-42f1-a25a-4371fd2521e6" />


---

## Passo 9 - Criar o Alias de desenvolvimento

Foi criado o Alias **dev**, apontando para a Versão 1 da função.

<img width="1906" height="942" alt="Captura de tela 2026-05-18 212907" src="https://github.com/user-attachments/assets/e8db2a2f-a918-4564-8762-7f0522005460" />


---

## Passo 10 - Alterar o código para Produção

O código da função foi alterado para representar o ambiente de Produção.

<img width="1904" height="936" alt="Captura de tela 2026-05-18 213047" src="https://github.com/user-attachments/assets/1d6f5d80-05d8-4895-a335-4e1f6596c3e3" />


---

## Passo 11 - Publicar a Versão 2

Após a alteração do código, foi publicada uma nova versão da função.

<img width="1908" height="939" alt="Captura de tela 2026-05-18 213144" src="https://github.com/user-attachments/assets/bb552d2c-c73b-4588-9368-b6424a0432c6" />


---

## Passo 12 - Criar o Alias de produção

Foi criado o Alias **prod**, apontando para a Versão 2 da função Lambda.

<img width="1901" height="943" alt="Captura de tela 2026-05-18 213355" src="https://github.com/user-attachments/assets/c4ace256-6193-4e99-9073-204cb96645e2" />


---

# Parte 3 - Criação do API Gateway

## Passo 13 - Acessar o serviço API Gateway

Foi acessado o serviço **Amazon API Gateway** para criação da API REST.

<img width="1905" height="940" alt="Captura de tela 2026-05-18 213701" src="https://github.com/user-attachments/assets/886a1aaf-0736-4dbf-b38f-167b6413dba4" />


---

## Passo 14 - Criar uma API REST

Foi criada uma API REST regional para integração com a função Lambda.

<img width="1907" height="945" alt="Captura de tela 2026-05-18 214638" src="https://github.com/user-attachments/assets/7d28a016-a513-4f50-b4e3-d54f5bf18276" />


---

## Passo 15 - Criar recurso e método GET

Foi criado o recurso **/hello** e configurado o método GET utilizando integração Proxy com a função Lambda através do Alias **dev**.

<img width="1908" height="947" alt="Captura de tela 2026-05-18 215243" src="https://github.com/user-attachments/assets/2f404b10-a749-4bee-a364-d9f1f5d0b01b" />


---

# Parte 4 - Implantação dos Stages

## Passo 16 - Criar Stage de Desenvolvimento

Foi realizado o deploy da API criando o Stage **Desenvolvimento**.

<img width="1915" height="273" alt="Captura de tela 2026-05-18 215357" src="https://github.com/user-attachments/assets/da67041d-fb88-46c5-8341-4373064194a5" />


---

## Passo 17 - Configurar integração com Alias de Produção

A integração foi alterada para utilizar o Alias **prod**.

<img width="1910" height="229" alt="Captura de tela 2026-05-18 215433" src="https://github.com/user-attachments/assets/a955a289-00f4-44d8-8534-f681c51e30cb" />


---

## Passo 18 - Criar Stage de Produção

Foi realizado o deploy criando o Stage **Producao**.

<img width="1913" height="297" alt="Captura de tela 2026-05-18 215508" src="https://github.com/user-attachments/assets/23b2420f-1c7a-49f9-ab88-ea6285d24beb" />


---

# Parte 5 - Testes dos Endpoints

## Passo 19 - Validar os ambientes Desenvolvimento e Produção

Foram acessadas as URLs geradas pelo API Gateway para validar o comportamento dos dois ambientes.

A validação confirmou que:

- O Stage Desenvolvimento utiliza o Alias dev;
- O Stage Produção utiliza o Alias prod;
- Cada ambiente retorna sua respectiva versão da função Lambda.

<img width="1899" height="509" alt="Captura de tela 2026-05-18 215649" src="https://github.com/user-attachments/assets/9701fbd3-34e4-44f7-84ed-ab404161120c" />


---

# Importância do uso de Aliases e Stages

A utilização conjunta de **Aliases** e **Stages** oferece diversas vantagens:

- Separação entre ambientes de desenvolvimento e produção;
- Controle de versões da aplicação;
- Implantações mais seguras;
- Possibilidade de rollback rápido;
- Endpoints estáveis para os consumidores da API;
- Melhor gerenciamento do ciclo de vida de aplicações serverless.

---

# Limpeza dos Recursos

Ao final do laboratório recomenda-se remover todos os recursos criados:

1. Excluir a API criada no API Gateway;
2. Excluir a função Lambda;
3. Remover os Aliases criados;
4. Excluir as versões publicadas;
5. Remover os logs do CloudWatch;
6. Excluir a Role IAM criada para a função.

---

# Conclusão

Neste laboratório foram explorados os principais conceitos de gerenciamento de versões em aplicações serverless utilizando AWS Lambda e API Gateway.

Foram realizadas as seguintes atividades:

- Criação de função Lambda;
- Testes utilizando eventos simulados;
- Publicação de versões da função;
- Criação de Aliases para Desenvolvimento e Produção;
- Criação de API REST no API Gateway;
- Configuração de integração Proxy;
- Criação de Stages para ambientes distintos;
- Testes dos endpoints publicados;
- Limpeza dos recursos criados.

Ao final da atividade foi possível compreender como utilizar Aliases e Stages para implementar ambientes independentes utilizando uma única função Lambda e uma única API, simplificando o gerenciamento e a implantação de aplicações serverless na AWS.
