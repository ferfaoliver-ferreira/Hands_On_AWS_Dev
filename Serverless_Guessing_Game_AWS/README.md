# 🎮 Serverless Guessing Game (Jogo de Adivinhação) na AWS

Este diretório contém a documentação técnica e o guia de implementação de uma aplicação web inteiramente serverless construída na Amazon Web Services (AWS). O objetivo do projeto é demonstrar na prática como construir, integrar, testar e desestruturar uma arquitetura desacoplada, escalável e de alta disponibilidade, eliminando completamente a sobrecarga de gerenciamento de servidores.

<img width="538" height="476" alt="Captura de tela 2026-05-20 220707" src="https://github.com/user-attachments/assets/33940d18-667b-44d4-ab21-67e0e051e985" />

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

<img width="1906" height="948" alt="Captura de tela 2026-05-20 211034" src="https://github.com/user-attachments/assets/cdba0f8e-6863-471a-b1f4-991013257339" />

*Legenda: Instanciação inicial da função Lambda com o ambiente de runtime Python 3.13 definido.*

### Etapa 2: Implementação da Lógica do Jogo em Python
Substituiu-se o esqueleto de código padrão da AWS pelo script real da aplicação. A lógica foi estruturada da seguinte forma:
* Importação da biblioteca nativa `random` para computar um número randômico oculto de 1 a 10.
* Extração do parâmetro `palpite` enviado pela query string do evento de requisição HTTP (`queryStringParameters`).
* Validação condicional encadeada (`if`, `elif`, `else`) para checar se o número inserido é maior, menor ou idêntico ao número secreto.
* Construção do dicionário de resposta com o cabeçalho JSON (`Content-Type`) e o `statusCode: 200` exigidos pelo protocolo HTTP.

<img width="1904" height="940" alt="Captura de tela 2026-05-20 211343" src="https://github.com/user-attachments/assets/59272167-ec75-41aa-b38d-2cf63d366fff" />

*Legenda: Editor de código integrado do Lambda exibindo o script Python implementado e pronto para o deploy.*

### Etapa 3: Criação e Configuração do Amazon API Gateway
Para que o navegador do usuário conseguisse se comunicar com o nosso código Python, expusemos um endpoint público utilizando uma **API HTTP**:
1. No console do **Amazon API Gateway**, criamos uma nova API chamada `API-FernandaOliveira`.
2. Mapeamos uma rota específica utilizando o método HTTP **`GET`** no recurso `/jogo`.
3. Definimos a integração dessa rota apontando diretamente para o destino da nossa função Lambda (`LambdaGame-FernandaOliveira`).

<img width="1906" height="935" alt="Captura de tela 2026-05-20 211627" src="https://github.com/user-attachments/assets/8c1667c2-e25b-4d6a-9389-caea96dedfbc" />

*Legenda: Criação da API HTTP no console do API Gateway especificando a integração nativa com o AWS Lambda.*

<img width="1909" height="941" alt="Captura de tela 2026-05-20 211717" src="https://github.com/user-attachments/assets/2a9a31da-9c9e-4920-88d0-26ee1436d164" />

*Legenda: Definição da rota GET /jogo vinculada ao microsserviço de backend.*

4. Configuramos as etapas de implantação (*Stages*). O ambiente foi publicado no estágio padrão **`prod`** com a propriedade de *Auto-deploy* (implantação automática de alterações) ativada.

<img width="1904" height="929" alt="Captura de tela 2026-05-20 211806" src="https://github.com/user-attachments/assets/bb7c3916-fe3e-4fa5-a2ce-9e0ff1da9385" />



*Legenda: Configuração do estágio de implantação 'prod' da API.*

<img width="1903" height="941" alt="Captura de tela 2026-05-20 211913" src="https://github.com/user-attachments/assets/a92f414a-af03-442c-995b-434b1a7e87e4" />

*Legenda: Visualização do estágio publicado e captura da URL de Invocação gerada pela AWS.*

### Etapa 4: Mitigação de Restrições de CORS
Como o nosso frontend e o nosso backend rodam em domínios diferentes da AWS, os navegadores bloqueariam as requisições por padrão. Para resolver isso, configurou-se as políticas de **CORS (Cross-Origin Resource Sharing)** diretamente na API:
* **Access-Control-Allow-Origin:** `*` (Permitindo requisições vindas de qualquer origem, essencial para ambientes de teste).
* **Access-Control-Allow-Headers:** `content-type`
* **Access-Control-Allow-Methods:** `GET`

<img width="1905" height="938" alt="Captura de tela 2026-05-20 212056" src="https://github.com/user-attachments/assets/fff5c749-3def-4eaf-b1aa-037184440724" />

*Legenda: Painel de gerenciamento de CORS no API Gateway garantindo a liberação de requisições de origens cruzadas.*

### Etapa 5: Provisionamento e Configuração do Amazon S3
Com a infraestrutura de backend operacional, configuramos o armazenamento e a distribuição estável do frontend.
1. Criamos um bucket S3 com o nome exclusivo `s3-website-fernandaoliveira`.

<img width="1907" height="945" alt="Captura de tela 2026-05-20 213047" src="https://github.com/user-attachments/assets/dd4f3880-f925-4218-b10d-bce70027f81e" />

*Legenda: Menu de criação de buckets do Amazon S3 na região us-east-1.*

2. Atualizamos o script JavaScript local contido no arquivo `index.html`, inserindo a URL de Invocação do API Gateway obtida no passo 3 dentro da função `fetch()`. Em seguida, efetuamos o upload do arquivo para o bucket.

<img width="1899" height="943" alt="Captura de tela 2026-05-20 213432" src="https://github.com/user-attachments/assets/37db3347-ff78-40ff-aa88-919d810bc6f0" />

*Legenda: Console de upload do Amazon S3 confirming o envio do arquivo index.html.*

3. Navegamos até a aba de propriedades do bucket e ativamos o recurso **Hospedagem de site estático (Static website hosting)**, definindo o arquivo `index.html` como o documento de índice principal.

<img width="1911" height="944" alt="Captura de tela 2026-05-20 213631" src="https://github.com/user-attachments/assets/1a1ff4db-1e3e-4d04-8380-2cb1c15b3a44" />

*Legenda: Ativação e configuração das propriedades de hospedagem web no S3.*

<img width="1910" height="946" alt="Captura de tela 2026-05-20 213653" src="https://github.com/user-attachments/assets/7306d8a6-ef74-4bf1-b18a-1ee3a90c1a0d" />

*Legenda: URL pública gerada pelo S3 para acesso direto à aplicação web.*

4. Por padrão, a AWS bloqueia acessos públicos ao S3. Para expor o jogo na internet, desativamos a caixa global de **"Bloquear todo o acesso público" (Block all public access)**.

<img width="1902" height="943" alt="Captura de tela 2026-05-20 213726" src="https://github.com/user-attachments/assets/a38953c6-f0c3-4feb-8691-c4cf369018ac" />

<img width="1912" height="503" alt="Captura de tela 2026-05-20 213801" src="https://github.com/user-attachments/assets/6c59b4d3-fa1e-498f-a951-136f7d42e542" />

*Legenda: Alteração das configurações de visibilidade do bucket para permissão pública de leitura.*

5. Para finalizar a segurança da camada de apresentação, aplicamos uma **S3 Bucket Policy** em formato JSON, concedendo explicitamente o efeito `Allow` para a ação `s3:GetObject` em todos os recursos internos daquele bucket.

<img width="1911" height="939" alt="Captura de tela 2026-05-20 215402" src="https://github.com/user-attachments/assets/64650ad0-27d4-470c-9105-4688721f81ea" />

*Legenda: Aplicação e salvamento da Bucket Policy estruturada em formato JSON.*

---

## 🎯 Testes e Evidências do Resultado Final

Acessando a URL pública gerada pelo endpoint do Amazon S3, validamos com sucesso a integração de ponta a ponta em tempo real.

* **Cenário de Teste 1:** Quando o usuário submete um palpite menor do que o número randômico armazenado em memória na execução do Lambda, o backend computa o valor e envia a dica instantaneamente de volta para a tela:

<img width="1912" height="983" alt="Captura de tela 2026-05-20 215348" src="https://github.com/user-attachments/assets/4a641d9b-9f06-4051-a281-ef2205754e2e" />

*Legenda: Execução do jogo no navegador exibindo a tratativa para palpites inferiores.*

* **Cenário de Teste 2:** Ao persistir e inserir o número exato, a função Lambda valida a igualdade e retorna a flag de sucesso, atualizando a interface gráfica com a tela azul de vitória:

![Estado final de sucesso](./assets/Captura%20de%20tela%202026-05-20%20215402.png)
*Legenda: Sucesso na adivinhação do número secreto, confirmando a perfeita comunicação S3 -> API Gateway -> Lambda.*

---

## 🧼 Ciclo de Vida e Desestruturação de Recursos (FinOps)

Seguindo as melhores práticas recomendadas pela **Escola da Nuvem**, uma vez concluída e validada a homologação do software, iniciou-se o processo completo de *Clean-up* (limpeza). Essa rotina é vital em ambientes corporativos de engenharia de nuvem para evitar desperdício financeiro e recursos órfãos (*zombie resources*).

### 1. Limpeza de Dados e Exclusão do Amazon S3
A console da AWS impede por segurança a deleção de buckets que contenham dados. O processo seguiu o fluxo correto de purga:
* Selecionou-se o bucket e executou-se a ação de esvaziamento, sendo exigido digitar a frase de segurança `"excluir permanentemente"`.

Com o bucket completamente limpo, realizou-se a sua exclusão total inserindo o nome exato do recurso (`s3-website-fernandaoliveira`) na caixa de texto.

### 2. Deleção da Camada de APIs (Amazon API Gateway)
Acessou-se o menu de gerenciamento de APIs, selecionando o recurso criado no laboratório e disparando a remoção imediata do endpoint do ecossistema.


### 3. Desprovimento do Computo (AWS Lambda)
Por fim, acessamos a lista de funções do console AWS Lambda para eliminar o backend serverless.
* Localizou-se o recurso `LambdaGame-FernandaOliveira` na tabela activa.

* Através do botão superior "Ações", confirmou-se a remoção final do microsserviço da conta AWS.


---
🔬 *Projeto técnico prático elaborado com a infraestrutura fornecida e orientada pela Escola da Nuvem.*
