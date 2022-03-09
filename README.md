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

## Accessing the public url
```bash
heroku open
```

## Viewing the microservice log app-name1
Before running the command change the  `<app-name>` 
 ```bash
 heroku logs --tail --app <app-name>
 ```

## Creating a new payment microservice'
The process below speeds up the creation and already runs the 'dp init' 
```bash
dp new dp-payment --state mongodb --stream kafka --marketplace payment --init
``` 
 At the end enter the dp-payment folder

## Change the settings and credentials
- In the project folder open the configuration file 
```bash
code .\src\App\appsettings.json
```
- Change the ports in the DevPrime_Web item to ['https://localhost:5002;http://localhost:5003'](https://localhost:5002;http://localhost:5003)
- In the State item add the MongoDB credentials
- In the Stream item add the Kafka credentials
- Include in subscribe the queues 'orderevents'.


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


## Receiving events in the Stream adapter
- Implementing an event in the Stream 
```bash
dp add event OrderCreated -as PaymentService
```
-  Change the DTO 'OrderCreatedEventDTO' 
```bash
code .\src\Core\Application\Services\Payment\Model\OrderCreatedEventDTO.cs
``` 


```csharp
public class OrderCreatedEventDTO                     
  {                                                     
    public Guid OrderID { get; set; }
    public string CustomerName { get; set; }
    public double Value { get; set; }  
  }
```


## Configuring Subscribe in Stream
- Open the Event Stream configuration 
```bash 
code .\src\Adapters\Stream\EventStream.cs
``` 
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



## Modify the configuration in the project dockerfile
- Find and remove the line below 
```bash
ENTRYPOINT ["dotnet", "App.dll"]
```
- Add the line below 
```bash
CMD ASPNETCORE_URLS=http://*:$PORT dotnet App.dll
``` 


## Export the settings
Run to export settings 
```bash
dp export heroku
``` 

## Publishing the settings to Heroku
- Locate and open the created file 
```bash 
code .dp.devprimeherokuinstructions.txt
``` 
- Locate the  `<app-name>` tag and replace it with your app-name2  
- Locate the  `<token>` tag and replace it with the Heroku access token   
- Now we will create the 'Config Vars' environment variables on Heroku 
- Copy the curl command and run it on the command line. 
- Note the difference of curl in Windows Command, Powershell, Linux.
 
```bash
curl -X PATCH https://api.heroku.com/apps/<app-name>/config-vars -H "Content-Type: application/json" -H "Accept: application/vnd.heroku+json; version=3" -H "Authorization: Bearer <token>" -d @.\.devprime\heroku\heroku.json
```



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
![Heroku Apps](/images/heroku-04-run-dp-payment.png)

## Optional steps to stop services or view processes
a) Before running the command change the `<app-name>` 

```bash
heroku ps --app <app-name> 
heroku ps:stop web.1 --app <app-name>
heroku ps:start web.1 --app <app-name>
```

## Final considerations## 
- During this Heroku journey we have developed two microservices.
- To automate in your devops strategy use GitHub Actions.
