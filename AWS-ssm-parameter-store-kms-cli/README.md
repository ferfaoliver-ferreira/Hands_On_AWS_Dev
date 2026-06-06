# AWS Systems Manager Parameter Store com KMS e AWS CLI

## Objetivo

Este laboratório tem como objetivo demonstrar a utilização do **AWS Systems Manager Parameter Store** para armazenamento centralizado de configurações de aplicações, incluindo parâmetros do tipo **String** e **SecureString**.

Além disso, será realizada a integração com o **AWS Key Management Service (KMS)** para criptografia de informações sensíveis e o acesso aos parâmetros através da **AWS CLI** utilizando o **AWS CloudShell**.

Ao final do laboratório será possível criar parâmetros seguros, armazenar informações criptografadas e recuperar seus valores utilizando comandos da AWS CLI.

## Arquitetura

A arquitetura implementada utiliza:

- AWS Systems Manager Parameter Store;
- AWS Key Management Service (KMS);
- AWS CloudShell;
- AWS CLI;
- Parâmetros String;
- Parâmetros SecureString.

<img width="530" height="453" alt="image" src="https://github.com/user-attachments/assets/7ec46a37-8679-4f3a-9d55-7ceab9cdb095" />


### Fluxo da Solução

1. Criação de parâmetros String para armazenar URLs de banco de dados;
2. Criação de uma chave KMS para criptografia;
3. Criação de parâmetros SecureString utilizando a chave KMS;
4. Consulta dos parâmetros através da AWS CLI;
5. Recuperação de valores criptografados e descriptografados;
6. Remoção dos recursos ao final do laboratório.

## Pré-requisitos

- Conta AWS fornecida pela Escola da Nuvem;
- Acesso ao Console AWS;
- Permissões para AWS Systems Manager;
- Permissões para AWS KMS;
- Permissões para AWS CloudShell;
- Navegador Web atualizado.

---

# Parte 1 - Criação dos Parâmetros String no Parameter Store

## Passo 1 - Acessar o AWS Systems Manager

Após realizar login no Console AWS, foi acessado o serviço **AWS Systems Manager**.

<img width="1899" height="935" alt="Captura de tela 2026-05-26 204525" src="https://github.com/user-attachments/assets/293a5d62-5204-4c7b-9fd3-e5220e606f43" />


---

## Passo 2 - Acessar o Parameter Store

No menu lateral do Systems Manager, foi localizada a seção **Application Tools** e selecionada a opção **Parameter Store**.

![Captura de tela 2026-05-26 204649](./Captura%20de%20tela%202026-05-26%20204649.png)

---

## Passo 3 - Iniciar a criação de um parâmetro

Foi selecionada a opção **Create parameter** para iniciar a criação dos parâmetros da aplicação.

<img width="1908" height="951" alt="Captura de tela 2026-05-26 204649" src="https://github.com/user-attachments/assets/7301d398-daa1-45b0-930c-b2b5a7ee9bfe" />

---

## Passo 4 - Criar parâmetro String do ambiente de desenvolvimento

Foi criado o primeiro parâmetro do tipo **String**, responsável por armazenar a URL do banco de dados do ambiente de desenvolvimento.

Configurações utilizadas:

- **Name:** `/meu-app-<NomeSobrenome>/dev/db-url`
- **Description:** `Database URL for my app in dev`
- **Type:** `String`
- **Value:** `dev.database.<NomeSobrenome>.com:3306`

<img width="1910" height="941" alt="Captura de tela 2026-05-26 204930" src="https://github.com/user-attachments/assets/a70dcf41-2534-4cad-b1fb-039257c55248" />


---

## Passo 5 - Confirmar criação do parâmetro de desenvolvimento

Após preencher as informações, foi confirmada a criação do parâmetro de desenvolvimento.

![Captura de tela 2026-05-26 205009](./Captura%20de%20tela%202026-05-26%20205009.png)

---

## Passo 6 - Criar parâmetro String do ambiente de produção

Foi iniciado o processo de criação do segundo parâmetro, agora referente à URL do banco de dados do ambiente de produção.

Configurações utilizadas:

- **Name:** `/meu-app-<NomeSobrenome>/prod/db-url`
- **Description:** `URL for my app in prod`
- **Type:** `String`
- **Value:** `prod.database.<NomeSobrenome>.com:3306`

<img width="1900" height="888" alt="Captura de tela 2026-05-26 205107" src="https://github.com/user-attachments/assets/d23cab3f-5283-4148-b7f5-b2dd2d0dba54" />


---

## Passo 7 - Confirmar criação do parâmetro de produção

Após preencher as informações, foi confirmada a criação do parâmetro de produção.

<img width="1903" height="888" alt="Captura de tela 2026-05-26 205113" src="https://github.com/user-attachments/assets/08ab5655-36ac-4eb1-9b9a-a96d66f51693" />


---

# Parte 2 - Criação da Chave KMS

## Passo 8 - Acessar o AWS Key Management Service

Após criar os dois parâmetros do tipo String, foi acessado o serviço **AWS Key Management Service (KMS)** para criação da chave de criptografia.

<img width="1897" height="620" alt="Captura de tela 2026-05-26 205233" src="https://github.com/user-attachments/assets/d9221d26-8502-42df-9825-2559e1d7c633" />


---

## Passo 9 - Iniciar a criação da chave KMS

Foi selecionada a opção **Create a key** para iniciar a criação de uma chave gerenciada pelo cliente.

![Captura de tela 2026-05-26 205303](./Captura%20de%20tela%202026-05-26%20205303.png)<img width="1912" height="945" alt="Captura de tela 2026-05-26 205303" src="https://github.com/user-attachments/assets/403d4951-34ff-4e6a-a558-53356a000bee" />

---

## Passo 10 - Manter as configurações iniciais da chave

Na primeira etapa da criação da chave KMS, as configurações padrão foram mantidas e foi selecionada a opção para avançar.

<img width="1909" height="940" alt="Captura de tela 2026-05-26 205317" src="https://github.com/user-attachments/assets/685e45e6-daf6-44d9-ae05-02f9620a3793" />


---

## Passo 11 - Configurar Alias e Descrição da chave

Foram preenchidos os campos de identificação da chave KMS.

Configurações utilizadas:

- **Alias:** `<NomeSobrenome>-key`
- **Description:** `<NomeSobrenome>-key`

<img width="1899" height="862" alt="Captura de tela 2026-05-26 205410" src="https://github.com/user-attachments/assets/efec216d-28c2-4d10-8a8d-639b93633dc4" />

---

## Passo 12 - Avançar para revisão da chave

Após informar o Alias e a Description, foi utilizada a opção **Skip to review** para avançar diretamente para a revisão da chave.

<img width="1906" height="943" alt="Captura de tela 2026-05-26 205445" src="https://github.com/user-attachments/assets/d88cfb79-e4ca-48d9-907f-5263d4feda11" />


---

## Passo 13 - Finalizar criação da chave KMS

Após revisar as configurações, foi selecionada a opção **Finish** para concluir a criação da chave KMS.

<img width="1897" height="940" alt="Captura de tela 2026-05-26 205502" src="https://github.com/user-attachments/assets/d2848bd7-3775-43ee-9b42-d05059ad3868" />


---

## Passo 14 - Validar chave KMS criada

A chave KMS criada foi validada na listagem de chaves gerenciadas pelo cliente.

<img width="1896" height="949" alt="Captura de tela 2026-05-26 205517" src="https://github.com/user-attachments/assets/7d95ad1e-1633-431b-931d-f0dc2b1b338c" />


---

# Parte 3 - Criação dos Parâmetros SecureString

## Passo 15 - Retornar ao Parameter Store

Após criar a chave KMS, foi acessado novamente o **AWS Systems Manager Parameter Store** para criação dos parâmetros seguros.

<img width="1902" height="938" alt="Captura de tela 2026-05-26 205657" src="https://github.com/user-attachments/assets/04717197-3c49-4e5c-9504-e11618dd85fc" />


---

## Passo 16 - Criar parâmetro SecureString de desenvolvimento

Foi iniciado o cadastro do parâmetro responsável por armazenar a senha do banco de dados do ambiente de desenvolvimento.

Configurações utilizadas:

- **Name:** `/meu-app-<NomeSobrenome>/dev/db-password`
- **Description:** `Database password for my app in dev`
- **Type:** `SecureString`
- **Value:** `aqui é a senha do dev`

<img width="1904" height="887" alt="Captura de tela 2026-05-26 205935" src="https://github.com/user-attachments/assets/ee530381-a0e2-43aa-ab1d-556714041549" />


---

## Passo 17 - Selecionar a chave KMS para o parâmetro SecureString

No campo **KMS key id**, foi selecionada a chave KMS criada anteriormente para criptografar o parâmetro SecureString.

![Captura de tela 2026-05-26 205948](./Captura%20de%20tela%202026-05-26%20205948.png)

---

## Passo 18 - Confirmar criação do parâmetro SecureString de desenvolvimento

Após configurar o tipo SecureString, selecionar a chave KMS e informar o valor, foi criado o parâmetro de desenvolvimento.

<img width="1906" height="938" alt="Captura de tela 2026-05-26 210009" src="https://github.com/user-attachments/assets/39e6efe7-5e7f-4b35-999c-9712c600739b" />


---

## Passo 19 - Criar parâmetro SecureString de produção

Foi criado o parâmetro responsável por armazenar a senha do banco de dados do ambiente de produção.

Configurações utilizadas:

- **Name:** `/meu-app-<NomeSobrenome>/prod/db-password`
- **Description:** `Database password for my app in prod`
- **Type:** `SecureString`
- **Value:** `aqui é a senha do prod`

<img width="1907" height="938" alt="Captura de tela 2026-05-26 210308" src="https://github.com/user-attachments/assets/a2024b9d-3116-4297-86eb-36a3651f1d06" />


---

## Passo 20 - Selecionar a chave KMS para o parâmetro de produção

Foi selecionada a mesma chave KMS criada anteriormente para criptografar o parâmetro SecureString de produção.

<img width="1908" height="946" alt="Captura de tela 2026-05-26 210353" src="https://github.com/user-attachments/assets/1cdab439-9b4a-46fe-958d-270011e128a0" />


---

## Passo 21 - Confirmar criação do parâmetro SecureString de produção

Após preencher todas as informações, foi confirmada a criação do parâmetro SecureString do ambiente de produção.

<img width="1897" height="936" alt="Captura de tela 2026-05-26 210420" src="https://github.com/user-attachments/assets/c692da92-7471-436e-8267-7677ae8f01e8" />


---

## Passo 22 - Validar os quatro parâmetros criados

Ao final da etapa de criação, foram validados os quatro parâmetros no Parameter Store:

- `/meu-app-<NomeSobrenome>/dev/db-url`
- `/meu-app-<NomeSobrenome>/prod/db-url`
- `/meu-app-<NomeSobrenome>/dev/db-password`
- `/meu-app-<NomeSobrenome>/prod/db-password`

Foram criados dois parâmetros do tipo **String** e dois parâmetros do tipo **SecureString**.

<img width="1908" height="943" alt="Captura de tela 2026-05-26 210520" src="https://github.com/user-attachments/assets/3de11aa5-5eab-48d5-a61a-ffea33701e4c" />

---

# Parte 4 - Consulta dos Parâmetros pelo AWS CloudShell

## Passo 23 - Acessar o AWS CloudShell

Foi acessado o serviço **AWS CloudShell** para execução de comandos AWS CLI diretamente pelo navegador.

<img width="1908" height="949" alt="Captura de tela 2026-05-26 210746" src="https://github.com/user-attachments/assets/77fa1faa-e936-44f1-9255-6736579c3155" />


---

## Passo 24 - Abrir o terminal do CloudShell

Após acessar o CloudShell, foi aguardado o carregamento do terminal para iniciar a execução dos comandos.


---

## Passo 25 - Consultar parâmetros sem descriptografia

Foi executado o comando abaixo para recuperar os parâmetros do ambiente de desenvolvimento:

```bash
aws ssm get-parameters \
--names /meu-app-<NomeSobrenome>/dev/db-url /meu-app-<NomeSobrenome>/dev/db-password
```

Nesta consulta, o parâmetro do tipo **String** retorna o valor em texto claro, enquanto o parâmetro do tipo **SecureString** retorna o valor criptografado.

<img width="1900" height="935" alt="Captura de tela 2026-05-26 211609" src="https://github.com/user-attachments/assets/7978d2b8-4b9a-4b90-a9c5-b896e565fc54" />


---

## Passo 26 - Validar retorno criptografado

Foi validado no retorno do comando que a URL do banco foi exibida em texto claro, enquanto a senha foi exibida criptografada.

<img width="1907" height="941" alt="Captura de tela 2026-05-26 211644" src="https://github.com/user-attachments/assets/ec6ae319-f564-429e-9704-d7f8a7bccba8" />


---

## Passo 27 - Consultar parâmetros com descriptografia

Foi executado o mesmo comando anterior, adicionando a opção `--with-decryption`.

```bash
aws ssm get-parameters \
--names /meu-app-<NomeSobrenome>/dev/db-url /meu-app-<NomeSobrenome>/dev/db-password \
--with-decryption
```

<img width="1904" height="942" alt="Captura de tela 2026-05-26 211653" src="https://github.com/user-attachments/assets/35e65c13-aed6-4ace-8a30-b8d47f7d6aa6" />

---

## Passo 28 - Validar retorno descriptografado

Após executar o comando com `--with-decryption`, foi possível visualizar o valor real armazenado no parâmetro SecureString.

Essa etapa demonstra a diferença entre recuperar um parâmetro seguro com e sem descriptografia.

---

# Parte 5 - Exclusão dos Parâmetros

## Passo 29 - Selecionar os parâmetros criados

Após a validação via CLI, foi acessado novamente o Parameter Store para remover os parâmetros criados durante o laboratório.

---

## Passo 30 - Excluir os parâmetros do Parameter Store

Os parâmetros criados foram selecionados e foi utilizada a opção **Excluir** para removê-los.

---

## Passo 31 - Confirmar exclusão dos parâmetros

Foi confirmada a exclusão dos parâmetros para garantir que os recursos não permanecessem ativos após o laboratório.

---

# Parte 6 - Exclusão da Chave KMS

## Passo 32 - Acessar chaves gerenciadas pelo cliente

Foi acessado o serviço **AWS KMS** e selecionada a opção **Chaves gerenciadas pelo cliente**.

---

## Passo 33 - Selecionar a chave KMS criada

Foi localizada e selecionada a chave KMS criada durante o laboratório.

---

## Passo 34 - Programar exclusão da chave KMS

No menu **Ações da chave**, foi selecionada a opção **Programar exclusão da chave**.

O período de espera foi configurado para **7 dias**.

---

## Passo 35 - Confirmar exclusão programada da chave

Foi marcada a confirmação exigida pela AWS para programar a exclusão da chave KMS após o período de espera de 7 dias.

---

# Importância do Parameter Store com KMS

O AWS Systems Manager Parameter Store permite armazenar configurações e segredos de aplicações de forma centralizada.

Quando integrado ao AWS KMS, o Parameter Store oferece maior segurança para dados sensíveis, como senhas, tokens, strings de conexão e credenciais de aplicações.

Principais benefícios:

- Armazenamento centralizado de configurações;
- Separação entre parâmetros comuns e sensíveis;
- Criptografia de informações sigilosas;
- Integração com AWS CLI;
- Possibilidade de automação em aplicações;
- Controle de acesso via IAM;
- Redução do risco de exposição de credenciais no código-fonte.

---

# Limpeza dos Recursos

Ao final do laboratório foram removidos os recursos criados:

- Parâmetros String;
- Parâmetros SecureString;
- Chave KMS programada para exclusão;
- Arquivos e sessões temporárias utilizadas no CloudShell.

A remoção dos recursos evita consumo indevido e mantém o ambiente organizado.

---

# Conclusão

Neste laboratório foram explorados os principais recursos do **AWS Systems Manager Parameter Store** em conjunto com o **AWS Key Management Service** e a **AWS CLI**.

Durante a atividade foram realizados os seguintes procedimentos:

- Criação de parâmetros do tipo String;
- Criação de uma chave KMS;
- Criação de parâmetros SecureString;
- Associação de parâmetros seguros a uma chave KMS;
- Consulta de parâmetros utilizando AWS CLI;
- Recuperação de valores criptografados;
- Recuperação de valores descriptografados com `--with-decryption`;
- Utilização do AWS CloudShell;
- Exclusão dos parâmetros criados;
- Programação da exclusão da chave KMS.

Ao final da atividade foi possível compreender como armazenar e recuperar configurações de aplicações de forma segura, utilizando o Parameter Store como repositório centralizado e o KMS para proteger informações sensíveis.
