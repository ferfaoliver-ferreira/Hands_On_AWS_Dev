# 🎮 Serverless Guessing Game (Jogo de Adivinhação) na AWS

Este diretório contém a documentação técnica e o guia de implementação de uma aplicação web inteiramente serverless construída na Amazon Web Services (AWS). O objetivo do projeto é demonstrar na prática como construir, integrar, testar e desestruturar uma arquitetura desacoplada, escalável e de alta disponibilidade, eliminando completamente a sobrecarga de gerenciamento de servidores.

---

## 🏗️ Arquitetura da Solução

A aplicação foi desenhada seguindo o modelo de microsserviços e computação orientada a eventos (*Event-Driven Architecture*), dividida em três camadas independentes:

1. **Camada de Apresentação (Frontend):** Uma interface estática (HTML5, CSS3 e JavaScript) hospedada e distribuída de forma pública e global através do **Amazon S3**.
2. **Camada de Integração e Roteamento (API):** O **Amazon API Gateway (HTTP API)** atua como a porta de entrada única, gerenciando as requisições assíncronas do cliente, tratando políticas de segurança de CORS (*Cross-Origin Resource Sharing*) e encaminhando o tráfego de forma otimizada.
3. **Camada de Computação (Backend):** Uma função serverless no **AWS Lambda** desenvolvida em **Python 3.13**, responsável por executar a lógica de negócio (validação dos palpites com base em um número secreto aleatório).

---

## 🛠️ Tecnologias e Serviços AWS Utilizados

* **AWS Lambda:** Computação serverless executada sob demanda.
* **Amazon API Gateway:** Exposição e gerenciamento de endpoints RESTful com baixo acoplamento.
* **Amazon S3 (Simple Storage Service):** Armazenamento de objetos e hospedagem estável de sites estáticos.
* **Python 3.13:** Engine do backend para processamento lógico rápido.
* **JavaScript (Fetch API):** Consumo assíncrono de APIs no lado do cliente.

---

## 🚀 Guia de Implementação Passo a Passo

### Etapa 1: Criação da Função AWS Lambda
O ponto de partida do backend consistiu em instanciar um ambiente de computação isolado.
1. No painel do AWS Lambda, criamos uma função configurada do zero.
2. Definiu-se o nome do recurso como `LambdaGame-FernandaOliveira`.
3. O ambiente de execução selecionado foi o **Python 3.13**, mantendo a arquitetura padrão baseada em `x86_64`.

![Instanciação inicial da função Lambda](./assets/211034.png)
*Legenda: Instanciação inicial da função Lambda com o ambiente de runtime Python 3.13 definido.*

### Etapa 2: Implementação da Lógica do Jogo em Python
Substituiu-se o esqueleto de código padrão da AWS pelo script real da aplicação. A lógica foi estruturada da seguinte forma:
* Importação da biblioteca nativa `random` para computar um número randômico oculto de 1 a 10.
* Extração do parâmetro `palpite` enviado pela query string do evento de requisição HTTP (`queryStringParameters`).
* Validação condicional encadeada (`if`, `elif`, `else`) para checar se o número inserido é maior, menor ou idêntico ao número secreto.
* Construção do dicionário de resposta com o cabeçalho JSON (`Content-Type`) e o `statusCode: 200` exigidos pelo protocolo HTTP.

![Editor de código integrado do Lambda](./assets/211343.png)
*Legenda: Editor de código integrado do Lambda exibindo o script Python implementado e pronto para o deploy.*

### Etapa 3: Criação e Configuração do Amazon API Gateway
Para que o navegador do usuário conseguisse se comunicar com o nosso código Python, expusemos um endpoint público utilizando uma **API HTTP**:
1. No console do **Amazon API Gateway**, criamos uma nova API chamada `API-FernandaOliveira`.
2. Mapeamos uma rota específica utilizando o método HTTP **`GET`** no recurso `/jogo`.
3. Definimos a integração dessa rota apontando diretamente para o destino da nossa função Lambda (`LambdaGame-FernandaOliveira`).

![Criação da API HTTP](./assets/211516.png)
*Legenda: Criação da API HTTP no console do API Gateway especificando a integração nativa com o AWS Lambda.*

![Definição da rota GET /jogo](./assets/211550.png)
*Legenda: Definição da rota GET /jogo vinculada ao microsserviço de backend.*

4. Configuramos as etapas de implantação (*Stages*). O ambiente foi publicado no estágio padrão **`prod`** com a propriedade de *Auto-deploy* (implantação automática de alterações) ativada.

![Configuração do estágio](./assets/211612.png)
*Legenda: Configuração do estágio de implantação 'prod' da API.*

!https://daemons.com.br/ritual-invocacao-evocacao/(./assets/211913.png)
*Legenda: Visualização do estágio publicado e captura da URL de Invocação gerada pela AWS.*

### Etapa 4: Mitigação de Restrições de CORS
Como o nosso frontend e o nosso backend rodam em domínios diferentes da AWS, os navegadores bloqueariam as requisições por padrão. Para resolver isso, configurou-se as políticas de **CORS (Cross-Origin Resource Sharing)** diretamente na API:
* **Access-Control-Allow-Origin:** `*` (Permitindo requisições vindas de qualquer origem, essencial para ambientes de teste).
* **Access-Control-Allow-Headers:** `content-type`
* **Access-Control-Allow-Methods:** `GET`

![Painel de gerenciamento de CORS](./assets/212457.png)
*Legenda: Painel de gerenciamento de CORS no API Gateway garantindo a liberação de requisições de origens cruzadas.*

### Etapa 5: Provisionamento e Configuração do Amazon S3
Com a infraestrutura de backend operacional, configuramos o armazenamento e a distribuição estável do frontend.
1. Criamos um bucket S3 com o nome exclusivo `s3-website-fernandaoliveira`.

![Menu de criação de buckets](./assets/213233.png)
*Legenda: Menu de criação de buckets do Amazon S3 na região us-east-1.*

2. Atualizamos o script JavaScript local contido no arquivo `index.html`, inserindo a URL de Invocação do API Gateway obtida no passo 3 dentro da função `fetch()`. Em seguida, efetuamos o upload do arquivo para o bucket.

![Console de upload do Amazon S3](./assets/213432.png)
*Legenda: Console de upload do Amazon S3 confirmando o envio do arquivo index.html.*

3. Navegamos até a aba de propriedades do bucket e ativamos o recurso **Hospedagem de site estático (Static website hosting)**, definindo o arquivo `index.html` como o documento de índice principal.

![Ativação do Static website hosting](./assets/213753.png)
*Legenda: Ativação e configuração das propriedades de hospedagem web no S3.*

!https://www.instagram.com/p/DWuOuzzkYLr/?img_index=3&hl=ro(./assets/213813.png)
*Legenda: URL pública gerada pelo S3 para acesso direto à aplicação web.*

4. Por padrão, a AWS bloqueia acessos públicos ao S3. Para expor o jogo na internet, desativamos a caixa global de **"Bloquear todo o acesso público" (Block all public access)**.

![Alteração das configurações de visibilidade](./assets/214539.png)
*Legenda: Alteração das configurações de visibilidade do bucket para permissão pública de leitura.*

5. Para finalizar a segurança da camada de apresentação, aplicamos uma **S3 Bucket Policy** em formato JSON, concedendo explicitamente o efeito `Allow` para a ação `s3:GetObject` em todos os recursos internos daquele bucket.

![Aplicação da Bucket Policy](./assets/214959.png)
*Legenda: Aplicação e salvamento da Bucket Policy estruturada em formato JSON.*

---

## 🎯 Testes e Evidências do Resultado Final

Acessando a URL pública gerada pelo endpoint do Amazon S3, validamos com sucesso a integração de ponta a ponta em tempo real.

* **Cenário de Teste 1:** Quando o usuário submete um palpite menor do que o número randômico armazenado em memória na execução do Lambda, o backend computa o valor e envia a dica instantaneamente de volta para a tela:

![Execução do jogo - Palpite menor](./assets/215348.jpg)
*Legenda: Execução do jogo no navegador exibindo a tratativa para palpites inferiores.*

* **Cenário de Teste 2:** Ao persistir e inserir o número exato, a função Lambda valida a igualdade e retorna a flag de sucesso, atualizando a interface gráfica com a tela azul de vitória:

![Estado final de sucesso](./assets/215402.png)
*Legenda: Sucesso na adivinhação do número secreto, confirmando a perfeita comunicação S3 -> API Gateway -> Lambda.*

---

## 🧼 Ciclo de Vida e Desestruturação de Recursos (FinOps)

Seguindo as melhores práticas recomendadas pela **Escola da Nuvem**, uma vez concluída e validada a homologação do software, iniciou-se o processo completo de *Clean-up* (limpeza). Essa rotina é vital em ambientes corporativos de engenharia de nuvem para evitar desperdício financeiro e recursos órfãos (*zombie resources*).

### 1. Limpeza de Dados e Exclusão do Amazon S3
A console da AWS impede por segurança a deleção de buckets que contenham dados. O processo seguiu o fluxo correto de purga:
* Selecionou-se o bucket e executou-se a ação de esvaziamento, sendo exigido digitar a frase de segurança `"excluir permanentemente"`.

![Confirmação de esvaziamento](./assets/215707.png)
*Legenda: Confirmação de segurança exigida para purgar de forma permanente os objetos do S3.*

![Bucket esvaziado](./assets/215816.png)
*Legenda: Confirmação da console validando que o bucket foi esvaziado com sucesso.*

Com o bucket completamente limpo, realizou-se a sua exclusão total inserindo o nome exato do recurso (`s3-website-fernandaoliveira`) na caixa de texto.

![Exclusão do bucket S3](./assets/215824.png)
*Legenda: Janela de encerramento definitivo e remoção do bucket S3.*

### 2. Deleção da Camada de APIs (Amazon API Gateway)
Acessou-se o menu de gerenciamento de APIs, selecionando o recurso criado no laboratório e disparando a remoção imediata do endpoint do ecossistema.

![Exclusão da API HTTP](./assets/215903.png)
*Legenda: Desativação e exclusão das rotas e do barramento HTTP no API Gateway.*

### 3. Desprovimento do Computo (AWS Lambda)
Por fim, acessamos a lista de funções do console AWS Lambda para eliminar o backend serverless.
* Localizou-se o recurso `LambdaGame-FernandaOliveira` na tabela ativa.

![Filtro e seleção da função Lambda](./assets/220253.png)
*Legenda: Filtro e seleção da função Lambda ativa antes do encerramento de ciclo de vida.*

* Através do botão superior "Ações", confirmou-se a remoção final do microsserviço da conta AWS.

![Exclusão da função Lambda](./assets/220707.png)
*Legenda: Pop-up de encerramento definitivo e exclusão lógica da função Lambda.*

---
🔬 *Projeto técnico prático elaborado com a infraestrutura fornecida e orientada pela Escola da Nuvem.*
