# 🌐 Redundância e Automação no Amazon EC2: Deploy via Console e AWS CLI

Este diretório contém a documentação técnica e o guia de implementação de uma infraestrutura computacional baseada em instâncias virtuais na Amazon Web Services (AWS). O objetivo do projeto é demonstrar na prática como construir, configurar, testar e desestruturar servidores web funcionais utilizando duas abordagens distintas: a interface gráfica do **AWS Management Console** e a automação programática via **AWS CLI (CloudShell)**.

---

## 🏗️ Arquitetura da Solução

A infraestrutura foi desenhada para ilustrar os conceitos fundamentais de computação elástica e provisionamento automatizado, dividida em duas frentes de implantação paralelas:

1. **Deploy via Console (Manual):** Criação orientada por assistente de uma instância virtual Linux configurada com um script de inicialização (*User Data*) para provisionamento automático do servidor web Apache.
2. **Deploy via CLI (Automatizado):** Instanciação programática via AWS CloudShell envolvendo a criação dinâmica de variáveis de ambiente, isolamento de um novo **Security Group**, parametrização de regras de firewall de entrada (Porta 80) e execução automatizada da instância utilizando consultas em tempo real ao SSM Parameter Store.

---

## 🛠️ Tecnologias e Serviços AWS Utilizados

* **Amazon EC2 (Elastic Compute Cloud):** Instâncias virtuais elásticas de computação baseadas em arquitetura x86_64.
* **AWS CloudShell:** Terminal baseado em navegador pré-configurado com a AWS CLI para gerenciamento de recursos.
* **AWS CLI:** Interface de linha de comando para execução de comandos estruturados nas APIs da AWS.
* **Security Groups:** Firewall virtual stateful atuando no nível da instância para controle de tráfego de entrada e saída.
* **User Data (Bootstrap Scripts):** Scripts de automação em Shell executados com privilégios de root durante o primeiro boot do servidor.

---

## 🚀 Guia de Implementação Passo a Passo

### Parte 1: Deploy via AWS Management Console
O ponto de partida consistiu em criar um servidor web funcional através da interface gráfica da console.

1. No painel do Amazon EC2, iniciou-se o assistente para a criação da instância batizada como `webserver-FernandaOliveira`.
2. O ambiente selecionado foi o **Amazon Linux 2023** rodando sob o tipo de instância elegível ao nível gratuito `t2.micro`.
3. Na seção de detalhes avançados, inseriu-se o script Bash abaixo no campo *User Data* para automatizar o setup do servidor web Apache (HTTPD) e customizar o arquivo de índice:

```bash
#!/bin/bash
dnf update -y
dnf install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello, estou na Developer AWS</h1>" > /var/www/html/index.html
