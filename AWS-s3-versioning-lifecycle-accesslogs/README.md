# Amazon S3 Básico, Avançado e Acesso a Logs

## Objetivo

Este laboratório tem como objetivo demonstrar a utilização dos principais recursos do Amazon S3 voltados à organização, segurança e gerenciamento eficiente de dados.

Durante a atividade serão explorados conceitos de criação de buckets, versionamento de objetos, regras de ciclo de vida, geração de URLs pré-assinadas e configuração de logs de acesso utilizando um bucket dedicado.

Ao final do laboratório será possível implementar práticas recomendadas de armazenamento, controle de versões, otimização de custos e monitoramento de acessos em ambientes AWS.

---

## Arquitetura

A arquitetura implementada utiliza:

- Amazon S3;
- Buckets de armazenamento;
- Versionamento de objetos;
- Lifecycle Rules;
- URLs Pré-assinadas;
- Bucket de Logs de Acesso;
- Registro de atividades de acesso ao bucket.

<img width="567" height="480" alt="Captura de tela 2026-05-29 223906" src="https://github.com/user-attachments/assets/0b276294-9cdd-4cee-8878-ba8bddc36ffb" />


### Fluxo da Solução

1. Criação do bucket principal para armazenamento de arquivos;
2. Upload de objetos para o bucket;
3. Ativação do versionamento;
4. Criação de múltiplas versões de um mesmo arquivo;
5. Configuração de regras de ciclo de vida;
6. Geração de URLs pré-assinadas para compartilhamento seguro;
7. Criação de bucket dedicado para logs;
8. Configuração de logs de acesso;
9. Consulta dos registros gerados.

---

## Pré-requisitos

- Conta AWS fornecida pela Escola da Nuvem;
- Acesso ao Console AWS;
- Permissões para utilização do Amazon S3;
- Navegador Web atualizado;
- Editor de texto local.

---

# Parte 1 - Criação do Bucket

## Passo 1 - Acessar o serviço Amazon S3

Após realizar login no Console AWS, foi acessado o serviço Amazon S3 através da barra de pesquisa do console.

<img width="1904" height="941" alt="Captura de tela 2026-05-29 211229" src="https://github.com/user-attachments/assets/3698375f-2052-4a56-9849-b78aadd1e447" />

---

## Passo 2 - Criar um novo bucket

Na tela principal do Amazon S3 foi selecionada a opção **Criar bucket** para iniciar a configuração do repositório de armazenamento.

<img width="1906" height="949" alt="Captura de tela 2026-05-29 211301" src="https://github.com/user-attachments/assets/6adea630-88ee-4db1-9e8e-cd945f1b1d50" />


---

## Passo 3 - Definir o nome do bucket

Foi informado um nome único globalmente para o bucket seguindo o padrão definido pelo laboratório.

Exemplo:

```text
bucket-edn-FernandaOliveira
```

<img width="1904" height="941" alt="Captura de tela 2026-05-29 211326" src="https://github.com/user-attachments/assets/a0cfa705-5dc6-4caf-a8c8-16c7b1868ac8" />


---

## Passo 4 - Configurar propriedades do bucket

Foram mantidas as configurações recomendadas pela AWS para propriedade dos objetos e ACLs desabilitadas.

<img width="1903" height="948" alt="Captura de tela 2026-05-29 211348" src="https://github.com/user-attachments/assets/3cd068e4-20ce-4467-8e90-f23817e2dd19" />

---

## Passo 5 - Bloquear acesso público

Foi mantida habilitada a opção **Bloquear todo o acesso público**, seguindo as boas práticas de segurança recomendadas pela AWS.

<img width="1902" height="943" alt="Captura de tela 2026-05-29 211437" src="https://github.com/user-attachments/assets/072602f1-4e28-41a5-816b-b309d70b4774" />


---

## Passo 6 - Configurar criptografia padrão

Foi mantida a criptografia padrão utilizando chaves gerenciadas pelo Amazon S3 (SSE-S3).

<img width="1908" height="944" alt="Captura de tela 2026-05-29 212207" src="https://github.com/user-attachments/assets/16bd74c1-4e05-460a-bcee-e910b2ce9f1e" />


---

## Passo 7 - Criar o bucket

Após revisar todas as configurações, foi selecionada a opção **Criar bucket**.

<img width="1910" height="940" alt="Captura de tela 2026-05-29 212255" src="https://github.com/user-attachments/assets/4b80b484-e1e5-4353-ac31-3e09be878e20" />


---

## Passo 8 - Validar a criação do bucket

Foi realizada a validação do bucket recém-criado na lista de buckets do Amazon S3.

<img width="1903" height="941" alt="Captura de tela 2026-05-29 212322" src="https://github.com/user-attachments/assets/cb9a37b4-673f-4f6f-8043-011c8ee2f20f" />


---

# Parte 2 - Upload de Arquivos e Versionamento

## Passo 9 - Acessar o bucket criado

Após a criação do bucket, foi acessado o bucket recém-criado para iniciar o armazenamento dos arquivos utilizados durante o laboratório.

<img width="1905" height="892" alt="Captura de tela 2026-05-29 212346" src="https://github.com/user-attachments/assets/0623afd5-e3b5-4b51-a99a-58b9b3d7f073" />


---

## Passo 10 - Criar o arquivo de teste

Foi criado localmente o arquivo de texto:

```text
Lab9.txt
```

Inicialmente contendo o conteúdo:

```text
Versão 1
```

<img width="1908" height="887" alt="Captura de tela 2026-05-29 212720" src="https://github.com/user-attachments/assets/91fdbc3e-9d06-4237-8d57-98b4c1f15bbf" />


---

## Passo 11 - Iniciar o upload do arquivo

Dentro do bucket foi selecionada a opção **Upload** para enviar o arquivo criado localmente.

<img width="1908" height="899" alt="Captura de tela 2026-05-29 212819" src="https://github.com/user-attachments/assets/7b6aa267-ecdc-436c-9a92-c3f1c764082a" />


---

## Passo 12 - Selecionar o arquivo Lab9.txt

Foi utilizado o botão **Adicionar arquivos** para localizar e selecionar o arquivo Lab9.txt.

<img width="1908" height="990" alt="Captura de tela 2026-05-29 212924" src="https://github.com/user-attachments/assets/044050cd-1169-4302-b995-48355f8038c9" />


---

## Passo 13 - Confirmar o upload

Após selecionar o arquivo, foi realizado o envio mantendo as configurações padrão do Amazon S3.

<img width="1903" height="892" alt="Captura de tela 2026-05-29 213258" src="https://github.com/user-attachments/assets/062d9dde-cbec-4a20-bd95-af0b02335adc" />


---

## Passo 14 - Validar o objeto armazenado

Foi realizada a validação do upload, confirmando que o arquivo Lab9.txt foi armazenado com sucesso no bucket.

<img width="1897" height="901" alt="Captura de tela 2026-05-29 213557" src="https://github.com/user-attachments/assets/0ff1327c-32a8-4a03-afa5-f8bf5850ab66" />


---

## Passo 15 - Acessar as propriedades do bucket

Foi acessada a aba **Properties** para habilitar o versionamento do bucket.

<img width="1906" height="888" alt="Captura de tela 2026-05-29 213846" src="https://github.com/user-attachments/assets/a1840970-3876-4e7b-a4fa-e9137c440caa" />


---

## Passo 16 - Editar as configurações de versionamento

Na seção **Bucket Versioning**, foi selecionada a opção **Edit**.

<img width="647" height="348" alt="Captura de tela 2026-05-29 213859" src="https://github.com/user-attachments/assets/d89db59b-2c71-4832-bc97-372d4957be28" />


---

## Passo 17 - Ativar o versionamento

Foi habilitada a funcionalidade de versionamento para permitir o armazenamento de múltiplas versões do mesmo objeto.

<img width="657" height="315" alt="Captura de tela 2026-05-29 213922" src="https://github.com/user-attachments/assets/c67e9bbd-3fe6-43a4-8778-33058bef1f17" />


---

## Passo 18 - Salvar as alterações

Após habilitar o versionamento, as alterações foram salvas.

<img width="792" height="384" alt="Captura de tela 2026-05-29 214134" src="https://github.com/user-attachments/assets/7e7a4b8e-fb1b-4d5b-b23c-6ebb28204a36" />


---

## Passo 19 - Validar o versionamento ativo

Foi realizada a validação confirmando que o bucket passou a possuir versionamento habilitado.

<img width="1910" height="891" alt="Captura de tela 2026-05-29 214211" src="https://github.com/user-attachments/assets/fd791611-12a5-45c6-adf6-87dec0737216" />


---

## Passo 20 - Alterar o conteúdo do arquivo

O arquivo Lab9.txt foi editado localmente adicionando uma nova linha:

```text
Versão 2
```

<img width="1899" height="884" alt="Captura de tela 2026-05-29 214452" src="https://github.com/user-attachments/assets/4aaf4a96-0d65-4a2b-87a4-466795cf57fb" />


---

## Passo 21 - Realizar o upload da nova versão

O arquivo atualizado foi enviado novamente para o bucket.

<img width="1910" height="893" alt="Captura de tela 2026-05-29 214651" src="https://github.com/user-attachments/assets/41db1537-d566-494e-bf45-799c0e23a1e0" />


---

## Passo 22 - Confirmar o envio da nova versão

Foi concluído o upload da nova versão do objeto.

<img width="1906" height="946" alt="Captura de tela 2026-05-29 214731" src="https://github.com/user-attachments/assets/d70594f7-80a1-4b24-8310-47c9d579b2ed" />


---

## Passo 23 - Validar a nova versão armazenada

Foi realizada a validação da atualização do arquivo dentro do bucket.

<img width="1905" height="935" alt="Captura de tela 2026-05-29 214754" src="https://github.com/user-attachments/assets/3ede1109-7e50-443e-b48d-4b44a5ff3f69" />


---

# Parte 3 - Histórico de Versões e Restauração

## Passo 24 - Exibir as versões dos objetos

Na aba **Objects**, foi habilitada a opção **Show Versions** para exibir todas as versões armazenadas.

<img width="1917" height="121" alt="Captura de tela 2026-05-29 214848" src="https://github.com/user-attachments/assets/c46e6428-19e0-4914-bb3f-7048b26f5b71" />


---

## Passo 25 - Visualizar o histórico de versões

Foi possível visualizar as diferentes versões do arquivo Lab9.txt armazenadas pelo Amazon S3.

<img width="1911" height="333" alt="Captura de tela 2026-05-29 214930" src="https://github.com/user-attachments/assets/12a9b682-23a3-49d2-9221-2e65ecde9325" />


---

## Passo 26 - Selecionar uma versão anterior

Foi selecionada a versão mais antiga do arquivo para análise.

<img width="1895" height="893" alt="Captura de tela 2026-05-29 215142" src="https://github.com/user-attachments/assets/cae045ce-d96b-4d71-93b3-a05886531f7d" />


---

## Passo 27 - Baixar a versão selecionada

Foi realizado o download da versão escolhida para recuperação local.

<img width="1908" height="891" alt="Captura de tela 2026-05-29 215228" src="https://github.com/user-attachments/assets/bb4a5ad0-50ba-4673-8920-93a48ed9c77c" />


---

## Passo 28 - Confirmar o download da versão

Foi validada a recuperação da versão anterior do arquivo.

<img width="1902" height="889" alt="Captura de tela 2026-05-29 215248" src="https://github.com/user-attachments/assets/9c5fd8b5-84a0-48ea-bf23-dc65ad8ac4fd" />

# Parte 4 - Configuração de Regra de Ciclo de Vida

## Passo 29 - Acessar a aba Management

Foi acessada a aba **Management** do bucket para configuração de regras de gerenciamento automático do ciclo de vida dos objetos.

<img width="1904" height="891" alt="Captura de tela 2026-05-29 215257" src="https://github.com/user-attachments/assets/349081d9-12cf-4381-9caa-bfab4b0730a9" />


---

## Passo 30 - Criar uma nova regra de ciclo de vida

Foi selecionada a opção **Create lifecycle rule** para iniciar a criação da política.

<img width="1901" height="933" alt="Captura de tela 2026-05-29 215317" src="https://github.com/user-attachments/assets/1a40574d-b08e-49b1-9584-95622f2bf1e0" />

---

## Passo 31 - Definir nome da regra

Foi informado um nome para a regra de ciclo de vida seguindo o padrão definido no laboratório.

Exemplo:

```text
regra-expiracao-lab9
```

<img width="1903" height="892" alt="Captura de tela 2026-05-29 215451" src="https://github.com/user-attachments/assets/a0ec2c0b-6d1d-48c2-834a-0990a28d3e71" />


---

## Passo 32 - Configurar expiração dos objetos

Foi configurada a regra para expirar automaticamente os objetos após o período determinado pelo laboratório.

Essa configuração demonstra como automatizar a remoção de arquivos antigos, reduzindo custos de armazenamento.

<img width="1908" height="893" alt="Captura de tela 2026-05-29 215803" src="https://github.com/user-attachments/assets/5bcb80dd-739e-49a2-9e1d-64fcc9a9f79a" />


---

## Passo 33 - Revisar e criar a regra

Após revisar as configurações, a regra de ciclo de vida foi criada.

<img width="1900" height="699" alt="Captura de tela 2026-05-29 215828" src="https://github.com/user-attachments/assets/7f902f3d-cb61-49b0-ae1b-26ddc83e509b" />


---

# Parte 5 - Geração de URL Pré-assinada

## Passo 34 - Acessar o AWS CloudShell

Foi aberto o ambiente **AWS CloudShell** para execução dos comandos AWS CLI.

<img width="1896" height="890" alt="Captura de tela 2026-05-29 215915" src="https://github.com/user-attachments/assets/f2219cac-4056-48d2-aa82-e8b9b3d473cc" />


---

## Passo 35 - Gerar URL pré-assinada

Foi executado o comando para gerar uma URL temporária de acesso ao objeto armazenado no bucket.

Exemplo:

```bash
aws s3 presign s3://bucket-lab9/Lab9.txt --expires-in 300
```

A URL gerada permite acesso temporário ao arquivo sem necessidade de autenticação na AWS.

<img width="1909" height="762" alt="Captura de tela 2026-05-29 215950" src="https://github.com/user-attachments/assets/fe8d4b24-2480-48de-b7c7-e4e238ccbf30" />
)

---

## Passo 36 - Copiar a URL gerada

A URL retornada pelo comando foi copiada para validação em um navegador.

<img width="1905" height="896" alt="Captura de tela 2026-05-29 220014" src="https://github.com/user-attachments/assets/88205090-901d-468c-a9c8-d845c944b7e5" />


---

## Passo 37 - Validar acesso ao arquivo

A URL foi acessada em uma nova aba do navegador, confirmando o acesso temporário ao conteúdo armazenado no bucket.

<img width="1900" height="934" alt="Captura de tela 2026-05-29 220244" src="https://github.com/user-attachments/assets/09e5764e-1c26-45c7-a2c2-31af7e269722" />


---

# Parte 6 - Criação do Bucket de Logs

## Passo 38 - Criar bucket para armazenamento dos logs

Foi iniciado o processo de criação de um segundo bucket dedicado ao armazenamento dos logs de acesso.

<img width="1914" height="945" alt="Captura de tela 2026-05-29 220254" src="https://github.com/user-attachments/assets/1748a651-b665-4eba-8b09-81bfb3d852e3" />


---

## Passo 39 - Configurar o bucket de logs

Foi informado o nome do bucket destinado ao recebimento dos logs de acesso do bucket principal.

Exemplo:

```text
logs-lab9-nomesobrenome
```

<img width="1898" height="892" alt="Captura de tela 2026-05-29 220957" src="https://github.com/user-attachments/assets/44d958d3-7251-42dd-97f5-0a6d624cbb3c" />


---

## Passo 40 - Finalizar a criação do bucket de logs

Após revisar as configurações, o bucket foi criado.

<img width="1910" height="938" alt="Captura de tela 2026-05-29 221502" src="https://github.com/user-attachments/assets/5662eaad-e70e-4d32-8ef4-2a90af76c399" />


---

# Parte 7 - Habilitação dos Logs de Acesso

## Passo 41 - Acessar propriedades do bucket principal

Foi acessado novamente o bucket principal para habilitar o registro de logs.

<img width="1903" height="847" alt="Captura de tela 2026-05-29 221930" src="https://github.com/user-attachments/assets/0e9a2518-00c5-4cab-bf34-83faa54e7a0c" />

---

## Passo 42 - Configurar Server Access Logging

Na seção **Server Access Logging**, foi selecionada a opção de edição para ativar o registro de acessos.

<img width="1891" height="544" alt="Captura de tela 2026-05-29 222147" src="https://github.com/user-attachments/assets/9d467925-fe83-48e6-99c3-ada1c1a6a36a" />


---

## Passo 43 - Definir bucket de destino dos logs

Foi selecionado o bucket criado anteriormente para armazenar os arquivos de log gerados pelo Amazon S3.

<img width="1903" height="514" alt="Captura de tela 2026-05-29 222247" src="https://github.com/user-attachments/assets/87174547-cbc1-437f-b77e-e083cb614213" />


---

## Passo 44 - Salvar a configuração dos logs

Após definir o bucket de destino, as configurações foram salvas.

---

# Arquitetura da Solução

A arquitetura implementada neste laboratório contempla:

- Bucket principal para armazenamento dos objetos;
- Versionamento de objetos;
- Regras de ciclo de vida;
- URLs pré-assinadas;
- Bucket dedicado para logs;
- Registro de acessos utilizando Server Access Logging.

---

# Importância dos Recursos Utilizados

Durante o laboratório foram explorados recursos amplamente utilizados em ambientes corporativos:

### Versionamento

Permite recuperar versões anteriores dos arquivos, protegendo contra alterações indevidas e exclusões acidentais.

### Lifecycle Rules

Automatizam o gerenciamento dos objetos, reduzindo custos operacionais e de armazenamento.

### URLs Pré-assinadas

Permitem compartilhamento seguro e temporário de arquivos sem exposição pública do bucket.

### Server Access Logging

Registra eventos de acesso aos objetos armazenados, auxiliando auditorias e investigações de segurança.

---

# Limpeza dos Recursos

Ao final do laboratório recomenda-se remover todos os recursos criados:

1. Excluir os objetos armazenados;
2. Excluir todas as versões dos objetos;
3. Remover as regras de ciclo de vida;
4. Desabilitar os logs de acesso;
5. Excluir o bucket de logs;
6. Excluir o bucket principal;
7. Encerrar o ambiente CloudShell, caso tenha sido utilizado.

---

# Conclusão

Neste laboratório foram explorados recursos avançados do Amazon S3 voltados para governança, segurança e gerenciamento do ciclo de vida dos dados.

Foram realizadas as seguintes atividades:

- Criação de bucket Amazon S3;
- Upload e armazenamento de objetos;
- Habilitação do versionamento;
- Recuperação de versões anteriores;
- Configuração de regras de ciclo de vida;
- Geração de URLs pré-assinadas;
- Criação de bucket para logs;
- Configuração de Server Access Logging;
- Validação dos mecanismos de armazenamento e auditoria.

Ao final da atividade foi possível compreender como o Amazon S3 oferece recursos robustos para proteção, compartilhamento, auditoria e gerenciamento automatizado de dados, sendo uma das soluções mais utilizadas para armazenamento de objetos na nuvem AWS.
---
