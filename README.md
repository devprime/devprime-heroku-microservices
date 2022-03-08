# Microservices on Heroku using DevPrime, MongoDB Atlas and Confluent Cloud Kafka

Heroku provides a cloud platform for publishing applications and we will be developing two microservices using the DevPrime platform, MongoDB and Kafka and [asynchronous communication](../../how-to/asynchronous-microservices-communication/).

**Items needed in your environment**

- Install the .NET SDK 6 or higher
- Visual Studio Code
- A live [Heroku](https://heroku.com)account
- An active [DevPrime](https:/devprime.io) account and Developer or Enterprise license
- A [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)account
- One account on [Confluent Cloud](https://www.confluent.io)
- [DevPrime CLI](../../../getting-started/) installed and active`(dp auth`)
- Active local Docker + GIT
- Powershell or Bash

**Creating access and getting credentials**

1) Log into [Heroku](http://heroku.com)

a) Create a new application and save the name .

b) Create a second application and save the name .

![Heroku Apps](/images/heroku-01-app.png)

c) Get the access token from["API KEY](https://dashboard.heroku.com/account)`enter code here`

2) Access [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) 
a) Create a free mongodb database  
b) Get the access credentials

3) Go to [Confluent Cloud](https://www.confluent.io) and create a Kafka service

- Create a free Kafka stream service
-  Add a topic named 'orderevents'
-  Add a topic named 'paymentevents'


![Confluent Kafka](/images/heroku-02-kafka.png)

-  Get access credentials to Confluent Cloud

4) Install the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) and login `heroku container:login`

**Creating Microservices 'Order' using DevPrime CLI** 
We will use the [DevPrime CLI](../../../getting-started/creating-the-first-microservice/) for creating the microservices.

- Creating the microservice
`dp new dp-order --stream kafka --state mongodb`Enter the dp-order folder to view the microservice

- Adding business rules
`dp marketplace order`

- Speeding up implementations
`dp init`

**Change the configurations by adding the MongoDB / Kafka credentials**
a) In the project folder open the configuration file
`code .\src\App\appsettings.json`
b) In the State item add the MongoDB credentials
c) In the Stream item add the Kafka credentials

**Run Microservices locally.**
`./run.ps1 or ./run.sh (Linux, MacOS)`

**Make a test post**
a) Open the web browser at http://localhost:5000 or https://localhost:5001
b) Click post and then 'Try it out'
c) Put in the data and submit

If everything has gone well so far then you can proceed with the rest of the setup and publishing in the Heroku environment.

**Fitting the project dockerfile for Heroku support**

a) Find and remove the line below 
`ENTRYPOINT ["dotnet", "App.dll"]`
b) Add the line below to the end 
`CMD ASPNETCORE_URLS=http://*:$PORT dotnet App.dll`

**Export the settings**
dp export heroku

![Microservices devprime heroku](/images/cli/devprime-cli-dp-export-heroku.png)

**Publishing the settings to Heroku**
a) Locate and open the created file `code .dp.devprimeherokuinstructions.txt` 
b) Locate the "<app-name>" tag and replace it with your app-name1 
c) Locate the "<app-name>" tag and replace it with the Heroku access token 
d) Now we will create the 'Config Vars' environment variables on Heroku

- Copy the curl command in the one changed in the previous steps
- Run it on the command line.
- Note the difference of curl in Windows Command, Powershell, Linux.

`curl -X PATCH https://api.heroku.com/apps/<app-name>/config-vars -H "Content-Type: application/json" -H "Accept: application/vnd.heroku+json; version=3" -H "Authorization: Bearer <token>" -d @.\.devprime\heroku\heroku.json`

**Config Vars' settings of your app-name1 in the Heroku**
[Portal](https://heroku.com) -> App -> Settings -> Config Vars

![Config Vars](/images/heroku-03-configvars.png)

**Compiling and publishing the docker image**

a) Before running the command change the '<app-name>'

- Compiling and sending
`heroku container:push web --app <app-name>`

- Change to release
`heroku container:release web --app <app-name>`

At this point you can already view the microservice in the portal![Heroku Apps](/images/heroku-04-run-dp-order.png)

**Accessing the public url**
`heroku open`

**Viewing the microservice log app-name1**
 a) Before running the command change the `<app-name>`
 `heroku logs --tail --app <app-name>`

**Creating a new payment microservice'**
The process below speeds up the creation and already runs the 'dp init'
`dp new dp-payment --state mongodb --stream kafka --marketplace payment --init`
 At the end enter the dp-payment folder

Change**the settings and credentials**
a) In the project folder open the configuration file`code .esrc\App\appsettings.json`
b) Change the ports in the DevPrime_Web item to['https://localhost:5002;http://localhost:5003'](https://localhost:5002;http://localhost:5003)
c) In the State item add the MongoDB credentials d) In the Stream item add the Kafka credentials e) Include in subscribe the queues 'orderevents'.

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

***Receiving events in the Stream adapter***
- Implementing an event in the Stream`dp add event OrderCreated -as PaymentService`
-  Change the DTO 'OrderCreatedEventDTO'
`code .\src\CoreApplication\Services\Payment\Model\OrderCreatedEventDTO.cs`

```csharp
public class OrderCreatedEventDTO                     
  {                                                     
    public Guid OrderID { get; set; }
    public string CustomerName { get; set; }
    public double Value { get; set; }  
  }
```

***Configuring Subscribe in Stream***
- Open the Event Stream configuration`code .\src\Adapters\StreamEventStream.cs`
- Change the implementation in Subscribe
 
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

**Modify the configuration in the project dockerfile**
- Find and remove the line below `ENTRYPOINT ["dotnet", "App.dll"]`
- Add the line below `CMD ASPNETCORE_URLS=http://*:$PORT dotnet App.dll`

**Export the settings**
`dp export heroku`

**Publishing the settings to Heroku**
- Locate and open the created file`code .dp.devprimeherokuinstructions.txt`
- Locate the ``tag and replace it with your app-name2
- Locate the ``tag and replace it with the Heroku access token
- Now we will create the 'Config Vars' environment variables on Heroku

- Copy the curl command in the one changed in the previous steps and run it on the command line.
- Note the difference of curl in Windows Command, Powershell, Linux.
 
`curl -X PATCH https://api.heroku.com/apps/<app-name>/config-vars -H "Content-Type: application/json" -H "Accept: application/vnd.heroku+json; version=3" -H "Authorization: Bearer <token>" -d @.\.devprime\heroku\heroku.json`

**Compiling and publishing the docker image**
a) Before running the command change the `<app-name>`

- Compiling and sending
`heroku container:push web --app <app-name>`

- Change to release
`heroku container:release web --app <app-name>`

![Heroku Apps](/images/heroku-04-run-dp-payment.png)

**Optional steps to stop services or view processes**
a) Before running the command change the `<app-name>`

`heroku ps --app <app-name>`
`heroku ps:stop web.1 --app <app-name>`
`heroku ps:start web.1 --app <app-name>`

**Final considerations**

- During this Heroku journey we have developed two microservices.
- To automate in your devops strategy use GitHub Actions.
