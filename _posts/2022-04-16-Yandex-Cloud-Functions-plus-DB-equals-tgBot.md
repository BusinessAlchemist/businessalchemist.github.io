## Using Yandex Cloud Functions to build serverless Telegram Bot with database storage.

Hi there. I recently chose to try the Yandex Cloud suite to host a chatbot project. Having a solid DBA and .Net development background I was surprised how difficult it was to set everything up. This post is aimed to capture the not-obvious configuration steps.

#### Brief overview of the project

I needed to prototype a simple Telegram bot to manage a rather simple business process. Process steps are known, thus I chose the State Machine pattern. The users were expected to send only known commands and inputs, that is, I didn't need to care about parsing natural language.
I chose to implement the bot using c#/.Net core because of prior experience with this language. I need to note the scarcity of examples and documentation for .Net usage at Yandex Cloud. Python has been given much more love in that respect.
For hosting I chose to use Yandex Cloud *Functions* (analogue of AWS Lambda). The reason being limited exposure to system configuration and wish to leverage the serverless setup.
For database I had to choose Postgres even though major preference is SQL Server (overall and for this specific project). Thus, the first note.

#### Learning 1. Only Postgres and Clickhouse work with YC Functions in the same network

I wrote an YC Function with simple code to retrieve rows from a managed database on the same YC account. I expected that I needed no spphiaitcated network and security configuration, such as SSL certificates, because both services: code and db - resided in the same cloud.
It turns out that it only holds true for postgres and clicshare dbs. The documentation article (link) avoids explicit claims that other DBs, such as SQL server, are not supported. It just doesn't give sample code for other choices.
It becomes apparent in settings: postgres managed instances have a connector option, where's MySQL nor SQL server have them.
There is a knowledge article that explains the (rather simple) steps to configure an YC function to interact with a postgres managed instance.
[https://cloud.yandex.ru/docs/functions/operations/database-connection](https://cloud.yandex.ru/docs/functions/operations/database-connection)

→ learning 1: if you use YC functions, your choice of DBs is limited to postgres an clickhouse, for all other DBs you’ll need to install SSL certificates and such.

#### Learning 2. Connecting to Postgres from a .Net YC function to Postgres.

In order to leverage integrated authentication, you’ll need to pass the *Context.securityToken* from the function to the db. To do this, you’ll first need to implement the YC interface so that your function actually recieves this context information.
Again, it's strange that it doesn't come in the default code, but your function handler should look something like this to receive Context information:

<>

Then you’ll be able to pass the security token as a password to Postgres:

<>

Make sure that you've assigned a service account to the function:

→ learning 2: implement the YC interface from start to receive security Context information - you'll need it to interact with other elements of the YC ecosystem.

#### Learning 3. NuGet package versions: npgsql and newtonsoft.Json

You’ll most likely need these 2 librararies mentioned above. It turns out that the latest versions may not work together.
Newtonsoft’s latest version is (at the moment of writing) 13.0.0 and it fails with this error code:
I found that the latest working version is 12.3.
Same about npgsql. The latest version 6.0.0 fails with this conflict:
These solutions from stack failed to help so I chose to check the older versions. It turned out that the version 5.7.3 was the one that doesn't have this conflict.
