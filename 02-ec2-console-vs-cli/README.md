# Laboratório AWS - Explorando o Amazon EC2

## Resumo

Neste laboratório foi realizada a criação de instâncias Amazon EC2 utilizando o Console de Gerenciamento AWS e o AWS CloudShell. Também foram configurados grupos de segurança, par de chaves, acesso HTTP para um servidor web Apache e, ao final, a remoção de todos os recursos criados.

---

# Objetivos

- Iniciar um servidor web de teste via Console AWS.
- Configurar acesso HTTP através de Security Groups.
- Inicializar uma segunda instância utilizando AWS CloudShell.
- Encerrar as instâncias e remover os recursos criados.

---

# Parte 1 - Criação da Instância EC2 pelo Console

## Passo 1 - Acessar o serviço EC2

Após efetuar login no Console AWS, foi realizada a pesquisa pelo serviço EC2 utilizando a barra de pesquisa.

<img width="1895" height="813" alt="Captura de tela 2026-05-05 203223" src="https://github.com/user-attachments/assets/d9ec4fed-6a02-42d0-b9f5-1c14c4acd38b" />


---

## Passo 2 - Iniciar a criação da instância

Foi selecionada a opção **Executar instância** para iniciar a criação do servidor virtual.

<img width="1893" height="933" alt="Captura de tela 2026-05-05 204334" src="https://github.com/user-attachments/assets/a41f0d00-f6ef-420d-a034-0322a2679382" />


---

## Passo 3 - Configurar a instância

Foram preenchidas as informações solicitadas:

- Nome da instância
- Amazon Linux 2023
- Tipo t2.micro

Conforme orientado no roteiro. :contentReference[oaicite:1]{index=1}

<img width="1904" height="914" alt="Captura de tela 2026-05-05 204701" src="https://github.com/user-attachments/assets/1763ad15-f059-4f0c-af22-4658af93404f" />


---

## Passo 4 - Criar o par de chaves

Foi criado um novo par de chaves RSA com formato `.pem`, utilizado posteriormente para acesso à instância.

<img width="1906" height="913" alt="Captura de tela 2026-05-05 204848" src="https://github.com/user-attachments/assets/6b4c0455-017c-4285-9f95-8c771f4a4df3" />


---

## Passo 5 - Configurar Security Group e User Data

Foi criado um grupo de segurança permitindo tráfego HTTP e inserido o script de User Data responsável pela instalação automática do Apache HTTP Server. :contentReference[oaicite:2]{index=2}

<img width="1904" height="903" alt="Captura de tela 2026-05-05 204913" src="https://github.com/user-attachments/assets/8802efd4-1d7d-4080-a7cb-616c17c0696e" />


---

## Passo 6 - Executar a instância

Após revisar as configurações, a instância foi criada com sucesso.

<img width="1912" height="758" alt="Captura de tela 2026-05-05 205054" src="https://github.com/user-attachments/assets/fcb52a20-ee81-4028-8105-f8cb024c5952" />


---

## Passo 7 - Acompanhar a inicialização

Foi acessada a tela de Instâncias para acompanhar o provisionamento do recurso.

<img width="1901" height="910" alt="Captura de tela 2026-05-05 205228" src="https://github.com/user-attachments/assets/c1aad2c4-4f56-49fa-a551-e6bb60b86aff" />


---

## Passo 8 - Validar o status da instância

Foi aguardada a conclusão das verificações de status da instância.

<img width="1911" height="1030" alt="Captura de tela 2026-05-05 205322" src="https://github.com/user-attachments/assets/13ed9ba7-bc7a-4c84-aa7e-a436e1cb8527" />


---

## Passo 9 - Obter o IPv4 público

Foram consultados os detalhes da instância para obter o endereço IPv4 público.



---

## Passo 10 - Testar o servidor web

Utilizando o IPv4 público da instância foi realizado o acesso HTTP para validar a página criada automaticamente pelo Apache.

<img width="1911" height="951" alt="Captura de tela 2026-05-05 205513" src="https://github.com/user-attachments/assets/6146c4e2-c30f-4afa-be2e-a773e3e12d9a" />


---

# Parte 2 - Inicialização de Instância pelo AWS CloudShell

## Passo 11 - Abrir o CloudShell

Foi aberto o AWS CloudShell através do Console AWS.

<img width="1910" height="921" alt="Captura de tela 2026-05-05 205554" src="https://github.com/user-attachments/assets/3bd4348d-b1d2-4a54-894b-e14e33490503" />


---

## Passo 12 - Definir variável do grupo de segurança

Foi criada a variável contendo o nome do grupo de segurança:

```bash
GRUPO_SEGURANCA="<NomeSobrenome>-grupo"
````

## Passo 13 - Definir nome da instância e par de chaves

Foram configuradas as variáveis utilizadas durante a criação da instância via AWS CLI:

```bash
NOME_INSTANCIA="instancia-<NomeSobrenome>"
PAR_CHAVE="parchave-<NomeSobrenome>"
```

<img width="1910" height="936" alt="Captura de tela 2026-05-05 210722" src="https://github.com/user-attachments/assets/1b507e0e-b2ad-47bf-9e1f-fa3bae9741d5" />


---

## Passo 14 - Criar Security Group via AWS CLI

Foi executado o comando responsável por criar um novo Security Group que será utilizado pela instância criada através do CloudShell.

<img width="1905" height="947" alt="Captura de tela 2026-05-05 211403" src="https://github.com/user-attachments/assets/ee51b2c8-638f-4470-83d3-55b73565400f" />


---

## Passo 15 - Liberar acesso HTTP

Foi executado o comando abaixo para permitir conexões HTTP na porta 80 do Security Group criado:

```bash
aws ec2 authorize-security-group-ingress \
--group-id $SECURITY_GROUP_ID \
--protocol tcp \
--port 80 \
--cidr 0.0.0.0/0
```

<img width="1910" height="944" alt="Captura de tela 2026-05-05 211542" src="https://github.com/user-attachments/assets/51e29f77-45f8-47a9-a2a3-24657529fde5" />


---

## Passo 16 - Criar instância EC2 via AWS CLI

Foi executado o comando `aws ec2 run-instances`, responsável por criar uma nova instância Amazon EC2 utilizando Amazon Linux, o Security Group recém-criado e o par de chaves configurado anteriormente.

<img width="1904" height="876" alt="Captura de tela 2026-05-05 211756" src="https://github.com/user-attachments/assets/0e5d3882-02f9-47ed-b82c-21b3296365a6" />


---

## Passo 17 - Validar a nova instância

Após a execução do comando, foi realizada a validação dos recursos criados para confirmar que a nova instância EC2 estava em execução corretamente.

<img width="1910" height="944" alt="Captura de tela 2026-05-05 211905" src="https://github.com/user-attachments/assets/79b4a8fb-5818-4a70-adbc-c81c19e423f4" />


---

# Parte 3 - Encerramento das Instâncias

## Passo 18 - Selecionar a instância criada

Foi selecionada a instância criada para iniciar o processo de encerramento.

<img width="1911" height="946" alt="Captura de tela 2026-05-05 211916" src="https://github.com/user-attachments/assets/703de4f1-0a4c-46c0-8f42-0d8b38dee3db" />


---

## Passo 19 - Encerrar a instância

Foi escolhida a opção **Encerrar (excluir) instância** para remover o recurso criado durante o laboratório.

<img width="1901" height="944" alt="Captura de tela 2026-05-05 211956" src="https://github.com/user-attachments/assets/8c99eb34-7877-4583-be74-477ea1f33b40" />


---

## Passo 20 - Confirmar encerramento

Foi realizada a confirmação do encerramento da instância EC2.

<img width="1915" height="903" alt="Captura de tela 2026-05-05 212012" src="https://github.com/user-attachments/assets/0f66050f-9ea5-44bc-9976-c38955247382" />


---

## Passo 21 - Verificar remoção

Foi realizada a validação para garantir que a instância foi encerrada corretamente.

<img width="1904" height="913" alt="Captura de tela 2026-05-05 212024" src="https://github.com/user-attachments/assets/719f8342-6ea3-4ae3-b87b-2b13a9dfddb3" />


---

# Parte 4 - Exclusão dos Security Groups

## Passo 22 - Acessar Security Groups

Foi acessada a seção **Security Groups** no serviço Amazon EC2.

<img width="1911" height="943" alt="Captura de tela 2026-05-05 212147" src="https://github.com/user-attachments/assets/9e7f6885-9d0b-4a6c-9f4c-47e96f0606bb" />


---

## Passo 23 - Selecionar o grupo de segurança criado

Foi localizado e selecionado o Security Group criado durante o laboratório.

<img width="1913" height="936" alt="Captura de tela 2026-05-05 212310" src="https://github.com/user-attachments/assets/dec0960e-f982-4ea7-9896-cd4281751b4a" />


---

## Passo 24 - Confirmar exclusão

Foi realizada a confirmação da exclusão do Security Group.

<img width="1907" height="942" alt="Captura de tela 2026-05-05 212356" src="https://github.com/user-attachments/assets/27de8bf6-4831-4bbc-93d7-274dd8bc2e38" />


---

# Parte 5 - Exclusão do Par de Chaves

## Passo 25 - Localizar o par de chaves

Foi acessada a seção **Pares de Chaves (Key Pairs)** para localizar a chave criada durante o laboratório.

<img width="1909" height="939" alt="Captura de tela 2026-05-05 212507" src="https://github.com/user-attachments/assets/19e3c6d7-eff7-4c32-a7ff-77d0f3371dfa" />


---

## Passo 26 - Confirmar exclusão do par de chaves

Foi digitada a confirmação exigida pela AWS para remover permanentemente o par de chaves.

<img width="1909" height="941" alt="Captura de tela 2026-05-05 212524" src="https://github.com/user-attachments/assets/7bfc67c4-c728-4c52-ad7c-370bb17bdd25" />

---

## Passo 27 - Verificar exclusão

Foi validada a remoção do par de chaves criado para o laboratório.

<img width="1913" height="948" alt="Captura de tela 2026-05-05 212542" src="https://github.com/user-attachments/assets/b2dd71ce-ba1b-490e-a43b-db043d1ad1bf" />


---

# Parte 6 - Encerramento do CloudShell

## Passo 28 - Acessar ações do CloudShell

Foi aberto o menu de ações do AWS CloudShell para iniciar a remoção do ambiente utilizado durante o laboratório.


---

## Passo 29 - Excluir o ambiente CloudShell

Foi selecionada a opção para excluir o ambiente CloudShell criado para execução dos comandos AWS CLI.

---

# Conclusão

Neste laboratório foram explorados os principais conceitos do Amazon EC2, incluindo:

- Criação de instâncias pelo Console AWS;
- Configuração de Security Groups;
- Criação e utilização de pares de chaves;
- Instalação automática de um servidor web Apache utilizando User Data;
- Utilização do AWS CloudShell;
- Gerenciamento de recursos utilizando AWS CLI;
- Criação de instâncias por linha de comando;
- Encerramento e remoção dos recursos criados.

Ao final da atividade, todos os recursos foram removidos com sucesso, garantindo que não permanecessem serviços ativos consumindo recursos da conta AWS.
