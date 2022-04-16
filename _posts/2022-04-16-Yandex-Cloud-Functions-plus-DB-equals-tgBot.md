## Using Yandex Cloud Functions to build serverless Telegram Bot with database storage.

Hi there. I recently chose to try the Yandex Cloud suite to host a chatbot project. Having a solid DBA and .Net development background I was surprised how difficult it was to set everything up. This post is aimed to capture the not-obvious configuration steps.

#### Brief overview of the project

I needed to prototype a simple Telegram bot to manage a rather simple business process. Process steps are known, thus I chose the [State Machine](https://en.wikipedia.org/wiki/Finite-state_machine) pattern. The users were expected to send only known commands and inputs, that is, I didn't need to care about parsing natural language.

I chose to implement the bot using c#/.Net core because of prior experience with this language. I need to note the scarcity of examples and documentation for .Net usage at Yandex Cloud. Python has been given much more love in that respect.

For hosting I chose to use Yandex Cloud *Functions* (analogue of AWS Lambda). The reason being limited exposure to system configuration and wish to leverage the serverless setup.
For database I had to choose Postgres even though my major preference is SQL Server (overall and for this specific project). Thus, the first note.

#### Learning 1. Only Postgres and Clickhouse work with YC Functions without SSL configuration.

I wrote an YC Function with simple code to retrieve rows from a managed database on the same YC account. I expected that I needed no sophiaitcated network and security configuration, such as SSL certificates, because both services: code and db - resided in the same cloud.

It turns out that it only holds true for postgres and clicshare dbs. The documentation article (https://cloud.yandex.ru/docs/functions/operations/database-connection) avoids any explicit claims that other DBs, such as SQL server, are not supported. It just doesn't give sample code for other choices.
However, it becomes apparent in settings:
![image](https://user-images.githubusercontent.com/16839729/163674640-ae7a1de6-597e-43ce-9608-2347c0c1fa2b.png)

There is a knowledge article that explains the (rather simple) steps to configure an YC function to interact with a postgres managed instance.
[https://cloud.yandex.ru/docs/functions/operations/database-connection](https://cloud.yandex.ru/docs/functions/operations/database-connection)

→ learning 1: if you use YC functions, your choice of DBs is limited to postgres an clickhouse, for all other DBs you’ll need to install SSL certificates and such.

#### Learning 2. Connecting to Postgres from a .Net YC function to Postgres.

In order to leverage integrated authentication, you’ll need to pass the *Context.securityToken* from the function to the db. To do this, you’ll first need to implement the YC interface(https://cloud.yandex.ru/docs/functions/lang/csharp/model/yc-function) so that your function actually recieves this *context* information.
Again, it's strange that it doesn't come in the default code, but your function handler should look something like this to receive Context information:
```
     public string FunctionHandler(Request request, Yandex.Cloud.Functions.Context context)
        {
            ////Code goes here;
        }
```        
Then you’ll be able to pass the security token as a password to Postgres:
```
  private NpgsqlConnection getConn(string accessTokenYC)
        {
            string connectionString = "Host='{YOURDBID}.postgresql-proxy.serverless.yandexcloud.net';" +
                "Username='{YOURDBNAME}';" +
                "Password='" + accessTokenYC + "';" +
                "Database='{YOURDBID}';" +
                "port=6432;" +
                "sslmode='Require';" +
                "Trust Server Certificate=true";
            return new NpgsqlConnection(connectionString);
        }
```
Make sure that you've assigned a *service account* to the function:

![image](https://user-images.githubusercontent.com/16839729/163674813-3ad60249-0b43-432a-ad3e-aed6bc723d1c.png)

→ learning 2: implement the YC interface from start to receive security Context information - you'll need it to interact with other elements of the YC ecosystem.

#### Learning 3. NuGet package versions: npgsql and newtonsoft.Json

You’ll most likely need these 2 librararies mentioned above. It turns out that the latest versions may not work together.
Newtonsoft’s latest version is (at the moment of writing) 13.0.1 and it fails with this error code: *"Could not load file or assembly 'Newtonsoft.Json, Version=13.0.0.0..."*
I found that the latest working version is 12.0.3.
Same about npgsql. The latest version 6.0.0 fails with an conflict with System.Runtime.CompilerServices.Unsafe 6.0.0.
A couple of solutions from stackoverflow failed to help so I chose to check the older versions. It turned out that the version 5.0.7 was the one that doesn't have this conflict.

Here is the Dependencies file I've used for my project:
```
  <Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
      <TargetFramework>netcoreapp3.1</TargetFramework>
    </PropertyGroup>
    <ItemGroup>
      <PackageReference Include="Newtonsoft.Json" Version="12.0.3"/>
      <PackageReference Include="Telegram.Bot" Version="17.0.0" />
      <PackageReference Include="Yandex.Cloud.SDK" Version="1.1.0"/>
      <PackageReference Include="Npgsql" Version="5.0.7"/>
    </ItemGroup>
  </Project>
```
