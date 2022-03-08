# Microservices on Heroku using DevPrime, MongoDB Atlas and Confluent Cloud Kafka
--

O Heroku oferece uma plataforma de cloud para publicação de aplicações e nós estaremos desenvolvendo dois microsserviços utilizando a plataforma DevPrime, MongoDB e Kafka e [comunicação assíncrona](../../how-to/asynchronous-microservices-communication/).

**Itens necessários em seu ambiente**
- Instale o .NET SDK 6 ou superior
- Visual Studio Code
- Uma conta ativa no [Heroku](https://heroku.com)
- Uma conta ativa na plataforma [DevPrime](https:/devprime.io) e licença de uso Developer ou Enterprise.
- Uma conta no [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
- Uma conta no [Confluent Cloud](https://www.confluent.io)
- [DevPrime CLI](../../../getting-started/) instalado e ativo (`dp auth`)
- Docker local ativo + GIT
- Powershell ou Bash

**Criação de acessos e obtenção de credenciais**

**1) Acesse o [Heroku](http://heroku.com)**

a) Crie um novo aplicativo e guarde o nome <app-name1>.

b) Crie um segundo aplicativo e guarde o nome <app-name2>.

![Heroku Apps](/images/heroku-01-app.png)
 
c) Obtenha o token de acesso em "[API KEY](https://dashboard.heroku.com/account)"

**2) Acesse o [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)**
&nbsp;
a) Crie um banco de dados mongodb gratuito
&nbsp;
b) Obtenha as credenciais de acesso

**3) Acesse o [Confluent Cloud](https://www.confluent.io) e crie um serviço Kafka**
a) Crie um serviço de stream Kafka gratuito
b) Adicione o tópico com o nome 'orderevents'
c) Adicione um tópico com o nome 'paymentevents'
![Confluent Kafka](/images/heroku-02-kafka.png)
d) Obtenha as credenciais de acesso ao Confluent Cloud


**4) Instale o [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) e efetue o login**
`heroku container:login`

**Criando o Microservices 'Order' utilizando DevPrime CLI**
Nós utilizaremos o [DevPrime CLI](../../../getting-started/creating-the-first-microservice/) para a criação dos microserviços.

- Criando o microsserviço
`dp new dp-order --stream kafka --state mongodb`
Entre na pasta dp-order para visualizar o microsserviço
- Adicionando regras negócio
`dp marketplace order`
- Acelerando as implementações 
`dp init`

**Aletere as configurações adicionando as credenciais MongoDB / Kafka**
a) Na pasta do projeto abra o arquivo de configuração
`code .\src\App\appsettings.json`
b) No item State adicione as credenciais do MongoDB
c) No item Stream adicione as credenciais do Kafka

**Execute o Microservices localmente.**
`.\run.ps1 ou ./run.sh (Linux, MacOS)`

**Faça um post de teste**
a) Abra o navegador web em http://localhost:5000 ou httsp://localhost:5001
b) Clique em post e depois em 'Try it out'
c) Colocque os dados e envie

Se tudo correu bem até aqui então já pode avançar com restante da configuração e publicação no ambiente da Heroku.

**Adequando o dockerfile do projeto para suporte ao Heroku**
a) Localizar e remover a linha abaixo
`ENTRYPOINT ["dotnet", "App.dll"]`
b) Adicionar a linha abaixo ao final
`CMD ASPNETCORE_URLS=http://*:$PORT dotnet App.dll`

**Exporte as configurações**
dp export heroku
![Microservices devprime heroku](/images/cli/devprime-cli-dp-export-heroku.png)

**Publicando as configurações no Heroku**
a) Localize e abra o arquivo criado
`code .\.devprime\heroku\instructions.txt`
b) Localize a tag `<app-name>` e substitua pelo nome do seu app-name1
c) Localize a tag `<token>` e substitua o token de acesso ao Heroku
d) Agora criaremos as variáveis de ambiente 'Config Vars' no Heroku
- Copie o comando curl no alterado nos passos anteriores
- Execulte na linha de comando. 
- Observe a diferença do curl no Windows Command, Powershell, Linux.
`curl -X PATCH https://api.heroku.com/apps/<app-name>/config-vars -H "Content-Type: application/json" -H "Accept: application/vnd.heroku+json; version=3" -H "Authorization: Bearer <token>" -d @.\.devprime\heroku\heroku.json`

**Configurações 'Config Vars' do seu app-name1 no Heroku**
[Portal](https://heroku.com) -> App -> Settings -> Config Vars

![Config Vars](/images/heroku-03-configvars.png)

**Compilação e publicação da imagem docker**
a) Antes de executar o comando altere o `<app-name>`
- Compilação e envio
`heroku container:push web --app <app-name>`
- Alteração para release
`heroku container:release web --app <app-name>`

Nesse momento você já pode visualizar no portal o microsserviço
![Heroku Apps](/images/heroku-04-run-dp-order.png)

**Acessando a url pública do projeto**
`heroku open`

**Visualizando o Log do microsserviço app-name1**
a) Antes de executar o comando altere o `<app-name>`
`heroku logs --tail --app <app-name>`

**Criando um novo microsserviço de payment'**
O processo abaixo acelera a criação e já executa o 'dp init'
`dp new dp-payment --state mongodb --stream kafka --marketplace payment --init`
 Ao final entre na pasta dp-payment

**Aletere as configurações e credenciais**
a) Na pasta do projeto abra o arquivo de configuração
`code .\src\App\appsettings.json`
b) Altere as portas o item DevPrime_Web para 'https://localhost:5002;http://localhost:5003'
c) No item State adicione as credenciais do MongoDB
d) No item Stream adicione as credenciais do Kafka
e) Inclua no subscribe a queues 'orderevents'.

```json
"DevPrime_Stream": [
    {
      "Alias": "Stream1",
      "Enable": "true",
      "Default": "true",
      "StreamType": "Kafka",
      "HostName": "<chang value>",
      "User": "<chang value>",
      "Password": "<chang value>",
      "Port": "9092",
      "Retry": "3",
      "Fallback": "State1",
      "Subscribe": [ { "Queues": "orderevents" } ]
    }
  ],
```
***Recebimento de eventos no adapter de Stream***
a) Implementando um evento no Stream
`dp add event OrderCreated -as PaymentService`
b) Altere o DTO 'OrderCreatedEventDTO'
`code .\src\Core\Application\Services\Payment\Model\OrderCreatedEventDTO.cs`

```csharp
public class OrderCreatedEventDTO                     
  {                                                     
    public Guid OrderID { get; set; }
    public string CustomerName { get; set; }
    public double Value { get; set; }  
  }
```
***Configurando o Subscribe no Stream***
a) Abra a configuração do Event Stream
`code .\src\Adapters\Stream\EventStream.cs`
b) Altere a implementação no Subscribe
```csharp
    public override void StreamEvents()
    {
        Subscribe<IPaymentService>("Stream1", "OrderCreated", (payload, paymentService, Dp) =>
        {
            var dto = System.Text.Json.JsonSerializer.Deserialize<OrderCreatedEventDTO>(payload);
            var command = new Payment()
            {
                CustomerName = dto.CustomerName,
                OrderID = dto.OrderID,
                Value = dto.Value
            };
            paymentService.Add(command);
        });
    }
```

**Modifique a configuração no dockerfile do projeto**
a) Localizar e remover a linha abaixo
`ENTRYPOINT ["dotnet", "App.dll"]`
b) Adicionar a linha abaixo
`CMD ASPNETCORE_URLS=http://*:$PORT dotnet App.dll`

**Exporte as configurações**
`dp export heroku`

**Publicando as configurações no Heroku**
a) Localize e abra o arquivo criado
`code .\.devprime\heroku\instructions.txt`
b) Localize a tag `<app-name>` e substitua pelo nome do seu app-name2
c) Localize a tag `<token>` e substitua o token de acesso ao Heroku
d) Agora criaremos as variáveis de ambiente 'Config Vars' no Heroku
- Copie o comando curl no alterado nos passos anteriores
- Execulte na linha de comando. 
- Observe a diferença do curl no Windows Command, Powershell, Linux.
`curl -X PATCH https://api.heroku.com/apps/<app-name>/config-vars -H "Content-Type: application/json" -H "Accept: application/vnd.heroku+json; version=3" -H "Authorization: Bearer <token>" -d @.\.devprime\heroku\heroku.json`


**Compilação e publicação da imagem docker**
a) Antes de executar o comando altere o `<app-name>`
- Compilação e envio
`heroku container:push web --app <app-name>`
- Alteração para release
`heroku container:release web --app <app-name>`

![Heroku Apps](/images/heroku-04-run-dp-payment.png)

**Passos opcionais para parar serviços ou visualizar os processos**
a) Antes de executar o comando altere o `<app-name>`

`heroku ps --app <app-name>`
`heroku ps:stop web.1 --app <app-name>`
`heroku ps:start web.1 --app <app-name>`

**Considerações finais**
- Durante essa jornada do Heroku nós desenvolvemos dois microsserviços.
- Para automatizar em sua estratégia de devops utilize o GitHub Actions.
