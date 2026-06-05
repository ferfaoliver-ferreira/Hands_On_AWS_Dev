# Arquitetura Resiliente e Desacoplada com Amazon SNS e Amazon SQS

Este laboratório prático foca em mensageria assíncrona, desacoplamento de sistemas (Microsserviços) e tratamento de falhas utilizando **Amazon SNS** (Simple Notification Service) e **Amazon SQS** (Simple Queue Service) com suporte a **DLQ** (Dead-Letter Queue).

---

## 🏗️ Passo a Passo Técnico: Implementação Prática

Abaixo está o detalhamento prático de todas as etapas executadas no Console da AWS para a construção e validação desta arquitetura.

---

### 🛠️ Fase 1: Configuração do Buffer Durável e Isolamento de Erros (Amazon SQS)

O primeiro passo consiste em preparar o ambiente de filas. Criamos a fila de falhas antes da principal, pois a fila principal precisa referenciar o ARN da DLQ em sua política de redrive.

#### 1. Criação da Dead-Letter Queue (DLQ)
1. No console do **Amazon SQS**, clique em **Criar fila**.
2. Mantenha o tipo como **Padrão**.
3. No campo **Nome**, defina como `minha-dlq-lab`.
4. Deixe as demais configurações como padrão e clique em **Criar fila**.

![Configuração Inicial da Fila DLQ](./images/Captura%20de%20tela%202026-05-13%20213252.png)
*Definição do tipo de fila padrão no painel do SQS.*

![Confirmação de Criação da DLQ](./images/Captura%20de%20tela%202026-05-13%20213514.png)
*Fila de mensagens mortas criada com sucesso.*

---

#### 2. Criação da Fila Principal e Vínculo de Redrive
1. No painel do SQS, clique em **Criar fila** (Tipo: **Padrão**).
2. Defina o **Nome** como `minha-fila-principal-lab`.
3. Configure o campo **Tempo limite de visibilidade** (*Visibility Timeout*) para **30 segundos**.

![Configurando Detalhes da Fila Principal](./images/Captura%20de%20tela%202026-05-13%20213537.png)
*Definição de nomenclatura e tempos base para a fila principal.*

4. Role até a seção **Fila de mensagens mortas (Dead-letter queue)** e marque a opção **Habilitada**.
5. Em **Escolher fila**, selecione **Inserir ARN da fila SQS** e cole o ARN da `minha-dlq-lab`.
6. No campo **Contagem máxima de recebimentos** (*maxReceiveCount*), defina o valor como **3**.
7. Clique em **Criar fila**.

![Ativação da Política de Redrive Policy](./images/Captura%20de%20tela%202026-05-13%20213635.png)
*Vinculação da fila principal com a DLQ com limite de 3 tentativas.*

![Filas Criadas e Prontas](./images/Captura%20de%20tela%202026-05-13%20213914.png)
*Listagem das duas filas prontas no painel do SQS.*

---

### 📢 Fase 2: Configuração do Barramento de Eventos (Amazon SNS)

Componente responsável por receber os eventos de um serviço publicador e distribuí-los para as filas assinadas.

#### 1. Criação do Tópico SNS
1. Acesse o console do **Amazon SNS** e clique em **Tópicos** -> **Criar tópico**.
2. Selecione o tipo **Padrão** e defina o nome como `meu-topico-sns-lab`.

![Criação do Tópico SNS](./images/Captura%20de%20tela%202026-05-13%20214128.png)
*Parametrização do tópico Standard no console do Amazon SNS.*

![Tópico Criado com Sucesso](./images/Captura%20de%20tela%202026-05-13%20214222.png)
*Painel exibindo as configurações gerais e o ARN do novo tópico.*

---

#### 2. Criação da Assinatura (Subscription)
1. Dentro da página do tópico criado, acesse a aba **Assinaturas** e clique em **Criar assinatura**.
2. No campo **Protocolo**, escolha **Amazon SQS**.
3. No campo **Endpoint**, insira o ARN exato da sua fila principal (`minha-fila-principal-lab`).
4. Clique em **Criar assinatura**.

![Configurando a Assinatura SQS](./images/Captura%20de%20tela%202026-05-13%20214231.png)
*Mapeamento e vinculação da fila consumidora principal como endpoint.*

![Assinatura Ativada no Tópico](./images/Captura%20de%20tela%202026-05-13%20214550.png)
*Visualização da assinatura ativa vinculando o barramento ao buffer SQS.*

---

### 🔒 Fase 3: Restrição de Acesso e Segurança (IAM Resource Policy)

Para aplicar o princípio do privilégio mínimo, editamos a política da fila para permitir apenas o tráfego vindo do tópico SNS específico.

1. No console do SQS, selecione a fila principal e vá até a aba **Política de acesso** -> **Editar**.
2. Altere o JSON para permitir estritamente que a ação `sqs:SendMessage` seja executada pelo serviço `sns.amazonaws.com` apenas quando a origem for o ARN do seu tópico.

![Acessando Configurações de Segurança da Fila](./images/Captura%20de%20tela%202026-05-13%20215437.png)
*Painel de visualização da política padrão do SQS.*

![Estruturação da Política JSON Customizada](./images/Captura%20de%20tela%202026-05-13%20215547.png)
*Edição manual do JSON aplicando as restrições de segurança.*

---

### 🧪 Fase 4: Validação da Arquitetura e Teste de Redrive (DLQ)

#### 1. Publicação do Evento via SNS
1. No console do Amazon SNS, entre no tópico e clique em **Publicar mensagem**.
2. Envie um payload estruturado para simular o comportamento de uma aplicação real.

![Publicando Evento de Teste no SNS](./images/Captura%20de%20tela%202026-05-13%20220154.png)
*Disparo de mensagem de teste através do console do SNS.*

---

#### 2. Simulação de Falhas Consecutivas e Transbordo para DLQ
1. Retorne ao console do SQS na fila principal. Note que há **1 mensagem disponível**.
2. Clique em **Enviar e receber mensagens** e depois em **Sondar mensagens** (*Poll for messages*).

![Sondagem Inicial de Mensagens Disponíveis](./images/Captura%20de%20tela%202026-05-13%20220849.png)
*Identificação da mensagem retida no buffer pronta para leitura.*

3. Abra os detalhes da mensagem sem excluí-la. Note que o contador **Contagem de recebimentos** (*Receive count*) estará em **1**.

![Primeira Tentativa de Recebimento](./images/Captura%20de%20tela%202026-05-13%20220931.png)
*Início do primeiro ciclo de processamento da mensagem.*

4. Aguarde os 30 segundos do *Visibility Timeout* expirarem e clique em **Sondar mensagens** novamente. O contador mudará para **2**.

![Segunda Tentativa de Recebimento](./images/Captura%20de%20tela%202026-05-13%20220947.png)
*Incremento automático do contador após expiração do tempo de visibilidade.*

5. Repita o processo até que a contagem atinja o limite máximo de 3 tentativas sem sucesso. Na tentativa seguinte, a mensagem expirará da fila principal.
6. Acesse a fila `minha-dlq-lab` e realize a sondagem: a mensagem estará armazenada nela de forma isolada e segura.

![Atingindo o Limite Máximo de Erros](./images/Captura%20de%20tela%202026-05-13%20222515.png)
*Confirmação de que a mensagem estourou o limite configurado e foi movida para a DLQ.*
