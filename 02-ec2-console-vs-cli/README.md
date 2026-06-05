# 🚀 Laboratório AWS EC2 - Explorando o Amazon EC2

## 📋 Descrição

Este laboratório teve como objetivo explorar os principais recursos do Amazon EC2, realizando a criação, configuração e gerenciamento de instâncias por meio do Console AWS e da AWS CLI utilizando o CloudShell.

Durante a atividade foram executadas tarefas de provisionamento de servidores, configuração de grupos de segurança, implantação automática de aplicações web e remoção dos recursos criados.

---

# 🎯 Objetivos

- Criar uma instância EC2 utilizando o Console AWS.
- Configurar Security Groups para acesso HTTP.
- Utilizar User Data para automatizar a instalação de um servidor web.
- Acessar a aplicação hospedada através do endereço IPv4 público.
- Criar recursos utilizando AWS CLI no CloudShell.
- Gerenciar e remover recursos após a conclusão do laboratório.

---

# 🏗️ Arquitetura Utilizada

```text
Internet
    │
    ▼
Security Group (HTTP - Porta 80)
    │
    ▼
Amazon EC2 (Amazon Linux)
    │
    ▼
Apache HTTP Server
    │
    ▼
Página Web
```

---

# 📚 Etapa 1 - Acessando o serviço EC2

Após acessar o Console AWS, foi realizada a navegação até o serviço Amazon EC2.

### Evidência

![Print 01](./images/print01.png)

---

# 📚 Etapa 2 - Criando uma Instância EC2

Selecionou-se a opção **Executar Instância** para iniciar o provisionamento de uma nova máquina virtual.

### Evidência

![Print 02](./images/print02.png)

---

# 📚 Etapa 3 - Configuração da Instância

Nesta etapa foram definidos:

- Nome da instância;
- Sistema operacional (Amazon Linux);
- Tipo de instância (`t2.micro`);
- Par de chaves;
- Configurações de rede.

### Evidência

![Print 03](./images/print03.png)

---

# 📚 Etapa 4 - Configuração do Security Group

Foi criado um grupo de segurança permitindo acesso HTTP pela porta 80.

### Evidência

![Print 04](./images/print04.png)

---

# 📚 Etapa 5 - Configuração do User Data

Foi utilizado o recurso **User Data** para automatizar a instalação e configuração do Apache.

## Script utilizado

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd

echo "<h1>Servidor Web AWS EC2 - Fernanda Oliveira</h1>" > /var/www/html/index.html
```

### Evidência

![Print 05](./images/print05.png)

---

# 📚 Etapa 6 - Inicialização da Instância

Após revisar todas as configurações, a instância foi criada com sucesso.

### Evidência

![Print 06](./images/print06.png)

---

# 📚 Etapa 7 - Verificação da Instância

Foi realizado o acompanhamento do processo de inicialização até que todos os checks fossem aprovados.

### Evidência

![Print 07](./images/print07.png)

---

# 📚 Etapa 8 - Teste da Aplicação Web

Utilizando o endereço IPv4 público da instância, foi possível acessar a página hospedada.

### Evidência

![Print 08](./images/print08.png)

---

# 📚 Etapa 9 - Utilizando o AWS CloudShell

Foi aberto o serviço CloudShell para execução de comandos AWS CLI.

### Evidência

![Print 09](./images/print09.png)

---

# 📚 Etapa 10 - Criação de Recursos via AWS CLI

Foram executados comandos para criação de Security Groups e instâncias através da linha de comando.

### Evidência

![Print 10](./images/print10.png)

---

# 📚 Etapa 11 - Consulta dos Recursos Criados

Foram realizadas consultas utilizando comandos AWS CLI para validação dos recursos provisionados.

### Evidência

![Print 11](./images/print11.png)

---

# 📚 Etapa 12 - Listagem das Instâncias

Foi realizada a verificação das instâncias existentes na conta AWS.

### Evidência

![Print 12](./images/print12.png)

---

# 📚 Etapa 13 - Encerrando a Instância

Após concluir os testes, a instância criada foi selecionada para encerramento.

### Evidência

![Print 13](./images/print13.png)

---

# 📚 Etapa 14 - Confirmação do Encerramento

A AWS solicitou confirmação da operação de encerramento.

### Evidência

![Print 14](./images/print14.png)

---

# 📚 Etapa 15 - Exclusão dos Security Groups

Os grupos de segurança criados para o laboratório foram removidos.

### Evidência

![Print 15](./images/print15.png)

---

# 📚 Etapa 16 - Validação da Exclusão

Após a exclusão, foi confirmada a remoção dos Security Groups.

### Evidência

![Print 16](./images/print16.png)

---

# 📚 Etapa 17 - Exclusão do Par de Chaves

Foi iniciada a remoção do Key Pair utilizado durante o laboratório.

### Evidência

![Print 17](./images/print17.png)

---

# 📚 Etapa 18 - Confirmação da Exclusão do Par de Chaves

A AWS solicitou confirmação digitando a palavra indicada.

### Evidência

![Print 18](./images/print18.png)

---

# 📚 Etapa 19 - Par de Chaves Removido

Após a confirmação, o Key Pair foi excluído com sucesso.

### Evidência

![Print 19](./images/print19.png)

---

# 📚 Etapa 20 - Exclusão do Ambiente CloudShell

Foi iniciada a remoção do ambiente CloudShell criado durante o laboratório.

### Evidência

![Print 20](./images/print20.png)

---

# 📚 Etapa 21 - Confirmação da Exclusão do CloudShell

A AWS solicitou a digitação da palavra **delete** para confirmação.

### Evidência

![Print 21](./images/print21.png)

---

# 📚 Resultados Obtidos

Ao final do laboratório foi possível:

✅ Criar instâncias EC2 pelo Console AWS

✅ Configurar Security Groups

✅ Implantar automaticamente um servidor web

✅ Utilizar User Data

✅ Trabalhar com AWS CLI

✅ Utilizar o CloudShell

✅ Gerenciar recursos EC2

✅ Encerrar e remover recursos da AWS

---

# 🛠️ Tecnologias Utilizadas

- Amazon EC2
- Amazon Linux
- AWS CLI
- AWS CloudShell
- Security Groups
- Key Pairs
- Apache HTTP Server

---

# 📖 Conceitos Aprendidos

- Provisionamento de instâncias EC2
- Configuração de regras de acesso
- Automação utilizando User Data
- Gerenciamento de infraestrutura via CLI
- Ciclo de vida de instâncias EC2
- Boas práticas de limpeza de recursos

---

# 👩‍💻 Autora

**Fernanda Ferreira de Oliveira**

Projeto desenvolvido durante laboratório prático de computação em nuvem utilizando Amazon Web Services (AWS).

---
