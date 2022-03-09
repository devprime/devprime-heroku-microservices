# Microservices on Heroku using DevPrime, MongoDB Atlas and Confluent Cloud Kafka

Heroku provides a cloud platform for publishing applications and we will be developing two microservices using the DevPrime platform, MongoDB and Kafka and [asynchronous communication](https://docs.devprime.tech/how-to/asynchronous-microservices-communication/).

## Items needed in your environment

- Install the .NET SDK 6 or higher / Visual Studio Code
- A live [Heroku](https://heroku.com) account
- Create [DevPrime](https://devprime.io) account and use Developer or Enterprise license
- A [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) account
- One account on [Confluent Cloud](https://www.confluent.io)
- [DevPrime CLI](https://docs.devprime.tech/getting-started/) installed and active`(dp auth`)
- Active local Docker + GIT
- Powershell or Bash


## Creating access and getting credentials

1. Log into [Heroku](http://heroku.com)
- Create a new application and save the name 
- Create a second application and save the name 
- Get the access token from "API KEY" in [Heroku](https://dashboard.heroku.com/account)

![Heroku Apps](/images/heroku-01-app.png)

```bash 
navigate to API Key
```

2. Access [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
- Create a free mongodb database 
- Get the access credentials

3. Go to [Confluent Cloud](https://www.confluent.io) and create a Kafka service 
- Create a free Kafka stream service 
- Add a topic named 'orderevents'
- Add a topic named 'paymentevents'
-  Get access credentials to Confluent Cloud 

![Confluent Kafka](/images/heroku-02-kafka.png)

4. Install the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) and login 
 ```bash
 heroku container:login 
```

### Important information:
This article provides a complete example at the address below. 

```bash
git clone https://github.com/devprime/devprime-heroku-microservices
```

To use after cloning follow the steps:
- Enter the dp-order folder and type
```bash 
dp license
```
- Enter the dp-payment folder and type
```bash
dp license 
```

If you want, follow the steps below that we will do
from beginning to end.

## Creating Microservices 'Order' using DevPrime CLI 
We will use the [DevPrime CLI](https://docs.devprime.tech/getting-started/creating-the-first-microservice/) for creating the microservices.

- Creating the microservice
```bash 
dp new dp-order --stream kafka --state mongodb
```
Enter the dp-order folder to view the microservice 

- Adding business rules
```bash 
dp marketplace order
```
- Speeding up implementations 
```bash 
dp init
```

## Change the configurations by adding the MongoDB / Kafka credentials

- In the project folder open the configuration file
```bash
code .\src\App\appsettings.json
```
- In the State item add the MongoDB credentials
- In the Stream item add the Kafka credentials 


## Run Microservices locally
```bash
./run.ps1 or ./run.sh (Linux, MacOS)
```

## Make a test post
- Open the web browser at http://localhost:5000 or https://localhost:5001 
- Click post and then 'Try it out' 
- Put in the data and submit 

If everything has gone well so far then you can proceed with the rest of the setup and publishing in the Heroku environment.

## Fitting the project dockerfile for Heroku support

- Find and remove the line below 
```bash 
ENTRYPOINT ["dotnet", "App.dll"]
```
- Add the line below to the end 
```bash 
CMD ASPNETCORE_URLS=http://*:$PORT dotnet App.dll
```


## Export the settings
Run to export settings
```bash 
dp export heroku
```

![Microservices devprime heroku](/images/devprime-cli-dp-export-heroku.png)

## Publishing the settings to Heroku
- Locate and open the created file 
```bash
code .\.devprime\heroku\instructions.txt
```
- Locate the  `<app-name>` tag and replace it with your app-name1 
- Locate the  `<token>` tag and replace it with the Heroku access token 
- Now we will create the 'Config Vars' environment variables on Heroku 
- Copy the curl command and run it on the command line. 
- Run it on the command line.

```bash 
curl -X PATCH https://api.heroku.com/apps/<app-name>/config-vars -H "Content-Type: application/json" -H "Accept: application/vnd.heroku+json; version=3" -H "Authorization: Bearer <token>" -d @.\.devprime\heroku\heroku.json
```


## Config Vars' settings of your app-name1 in the Heroku

[Portal](https://heroku.com) -> App -> Settings -> Config Vars

![Config Vars](/images/heroku-03-configvars.png)

## Compiling and publishing the docker image

a. Before running the command change the `<app-name>`
- Compiling and sending
```bash
  heroku container:push web --app <app-name>
```
- Change to release
```bash
heroku container:release web --app <app-name>
```

At this point you can already view the microservice in the portal
![Heroku Apps](/images/heroku-04-run-dp-order.png)

**Accessing the public url** </br>
`heroku open` </br>

**Viewing the microservice log app-name1** </br>
Before running the command change the  `<app-name>` </br>
 `heroku logs --tail --app <app-name>` </br>

**Creating a new payment microservice'** </br>
The process below speeds up the creation and already runs the 'dp init' </br>
`dp new dp-payment --state mongodb --stream kafka --marketplace payment --init` </br>
 At the end enter the dp-payment folder</br>

**Change the settings and credentials** </br>
- In the project folder open the configuration file </br>
`code .\src\App\appsettings.json`</br>
- Change the ports in the DevPrime_Web item to ['https://localhost:5002;http://localhost:5003'](https://localhost:5002;http://localhost:5003)</br>
- In the State item add the MongoDB credentials</br>
- In the Stream item add the Kafka credentials</br>
- Include in subscribe the queues 'orderevents'.</br>
</br>

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
</br>

***Receiving events in the Stream adapter***</br>
- Implementing an event in the Stream </br>
`dp add event OrderCreated -as PaymentService` </br>
-  Change the DTO 'OrderCreatedEventDTO' </br>
`code .\src\Core\Application\Services\Payment\Model\OrderCreatedEventDTO.cs` </br>
</br>

```csharp
public class OrderCreatedEventDTO                     
  {                                                     
    public Guid OrderID { get; set; }
    public string CustomerName { get; set; }
    public double Value { get; set; }  
  }
```
</br>

***Configuring Subscribe in Stream*** </br>
- Open the Event Stream configuration </br>
`code .\src\Adapters\Stream\EventStream.cs` </br>
- Change the implementation in Subscribe </br>
 </br>

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

</br>

**Modify the configuration in the project dockerfile** </br>
- Find and remove the line below </br>
`ENTRYPOINT ["dotnet", "App.dll"]` </br>
- Add the line below </br>
`CMD ASPNETCORE_URLS=http://*:$PORT dotnet App.dll` </br>
</br>

**Export the settings** </br>
Run to export settings </br>
`dp export heroku` </br>

**Publishing the settings to Heroku** </br>
- Locate and open the created file </br>
`code .dp.devprimeherokuinstructions.txt` </br>
- Locate the  `<app-name>` tag and replace it with your app-name2  </br>
- Locate the  `<token>` tag and replace it with the Heroku access token  </br> 
- Now we will create the 'Config Vars' environment variables on Heroku </br>
- Copy the curl command and run it on the command line. </br>
- Note the difference of curl in Windows Command, Powershell, Linux.</br>
 
`curl -X PATCH https://api.heroku.com/apps/<app-name>/config-vars -H "Content-Type: application/json" -H "Accept: application/vnd.heroku+json; version=3" -H "Authorization: Bearer <token>" -d @.\.devprime\heroku\heroku.json`

</br>

**Compiling and publishing the docker image** </br>
a) Before running the command change the `<app-name>` </br>
- Compiling and sending  </br>
`heroku container:push web --app <app-name>` </br>
- Change to release </br>
`heroku container:release web --app <app-name>` </br>
![Heroku Apps](/images/heroku-04-run-dp-payment.png)</br>

**Optional steps to stop services or view processes** </br>
a) Before running the command change the `<app-name>` </br>

`heroku ps --app <app-name>` </br>
`heroku ps:stop web.1 --app <app-name>`</br>
`heroku ps:start web.1 --app <app-name>`</br>

**Final considerations**</br>
- During this Heroku journey we have developed two microservices.</br>
- To automate in your devops strategy use GitHub Actions.</br>
