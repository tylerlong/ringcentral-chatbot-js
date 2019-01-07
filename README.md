# ringcentral-chatbot framework for JavaScript

## Philosophy

- Let developers focus on business logic instead of RingCentral/Glip Platform implementation details


## Quick start

- If you want to create a Glip chatbot and deploy it to your own server, please [read this tutorial](https://github.com/tylerlong/glip-ping-chatbot/tree/express) or [watch this video](https://youtu.be/CR66cwHvsOI).
- If you want to create a Glip chatbot and deploy it to AWS Lambda, please [read this tutorial](https://github.com/tylerlong/glip-ping-chatbot/tree/lambda) or [watch this video](https://youtu.be/JoEMXYmtn4Y).


## Advanced Demo bots

- [Glip Crontab Chatbot](https://github.com/tylerlong/glip-crontab-chatbot)
    - Allow Glip users to create cron jobs to send notifications
    - [Try it](https://www.ringcentral.com/apps/glip-crontab-chatbot)
- [Glip RC Assistant Chatbot](https://github.com/tylerlong/rc-assistant)
    - Allow Glip users to query/edit their RingCentral data via Glip.
    - [Try it](https://www.ringcentral.com/apps/glip-rc-assistant-chatbot)


## Create a RingCentral app

Create a new RingCentral app here: https://developer.ringcentral.com with the following information:

- Application Name: your app's name
- Description: your app's description
- Application Type: Public
- Platform Type: Server/Bot
- Carrier: Check all
- Permission Needed: Glip, WebHook Subscriptions, Read Accounts, Edit Extensions
- OAuth Redreict URI: `https://<chatbot-server>/bot/oauth`


## Supported features

- Bot token management
- Bot WebHook management
- Remove & re-add bot
- Deploy to AWS Lambda
- Update bot name and avatar
- Bot join & leave group


## Setup database

The first time you setup the bot, the database is empty, you need to create tables.

There is an easy way:

```
curl -X PUT -u admin:password https://<bot-server>/admin/setup-database
```


## Maintain

"Maintain" is useful in the following cases:

- If for reason bot server changed, you need to re-setup WebHooks
- You bot server was down for quite a while and your WebHooks have been blacklisted
- There is orphan data in database

You can "maintain" to resolve the issues above:

```
curl -X PUT -u admin:password https://<bot-server>/admin/maintain
```

It is recommended that you create a cron job to run this command daily.


## Diagnostic

Sometimes there are issues and you don't know what happened. This framework provides an interface for diagnostic:

```
curl -X GET -u admin:password https://<bot-server>/admin/diagnostic
```


## Hidden commands

The following commands are considered "hidden" or "easter eggs":

- `__rename__ <newName>`: rename bot to `newName`
- A message with text `__setAvatar__` and an attached image file: set bot avatar to the attached image


## Todo

- Create a website to auto generate code for developer to download
    - let developer select what he wants to do, what programming language to use, and finally generate the code for him
- Demo chatbots
    - survey bot
        - create survey and aggregate result with ease.
    - RingCentral messages monitoring bot
        - monitor RingCentral messages
    - Google drive notification
        - monitor Google Drive folder/file changes


## Notes

- AWS Lambda connects AWS RDS issue
    - If you create an RDS manually and let it create a new security group for you.
        - By default, the security group only allows inbound traffic from your current laptop's public IP address
        - AWS Lambda by default cannot access the newly created AWS RDS
        - We need to update security group to allow inbound traffic from `0.0.0.0/0`
- AWS Lambda and `await` issue
    - AWS Lambda is stateless and it won't keep a background runner
    - So if your code is async, DO remember to `await`. Otherwise that code will not be executed before Lambda invocation terminates
    - I forgot to `await`, a weird phenomenon was: whenever I issued a command, I always got reply of previous command
- AWS Lambda global code dead-lock issue
    - Do NOT invoke lambda functions in global code outside of handlers
    - I tried once and it seemed to case dead-lock. Lots of requests which causes 429 error and database connection exhausted
        - In global code I did `axio.put('https://<bot-server>/admin/maintain')` and it caused dead-lock
