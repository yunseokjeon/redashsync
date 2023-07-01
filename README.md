*redashsync* is a tool integrating Redash with Slack. It gets data from Redash and send it to Slack channel.

# Install

```
pip install redashsync
```

# How to get necessary keys

## Redash API Key and Query ID

<img src="https://github.com/yunseokjeon/redashsync/blob/master/doc/images/1.png?raw=true">

Click *Show API Key* and You would get below address.

http://localhost/api/queries/2/results.json?api_key=2CHjsILfC3fD4Q8nJU7IejS2CuRtKG8lKztY1HOk

Your API Key : 2CHjsILfC3fD4Q8nJU7IejS2CuRtKG8lKztY1HOk

Your Query ID : 2

## Slack Bot Token and Setting

https://api.slack.com/apps

<img src="https://github.com/yunseokjeon/redashsync/blob/master/doc/images/2.png?raw=true">

Create New App.

<img src="https://github.com/yunseokjeon/redashsync/blob/master/doc/images/3.png?raw=true">

Get Your Bot Token.

<img src="https://github.com/yunseokjeon/redashsync/blob/master/doc/images/4.png?raw=true">

Add an OAuth Scope. 1) chat:write 2) files:write

<img src="https://github.com/yunseokjeon/redashsync/blob/master/doc/images/5.png?raw=true">

Invite your slack bot.

# Your YAML

Follow the format below to complete your yaml file.

```yaml
# ./resource/sample.yml

slack_configs:
  slack:
    token: <YOUR_SLACK_BOT_TOKEN> # "xoxb-***-***-***"
    username: <YOUR_USER_NAME> # "redash-echo"

redash_configs:
  url: <YOUR_HOST> # "http://localhost"
  user_email: <YOUR_EMAIL> # "myname@example.com"
  query_api_key: <YOUR_QUERY_API_KEY>  # "2CHjsILfC3fD4Q8nJU7IejS2CuRtKG8lKztY1HOk"
```

# Use

## Send messages regularly

```Python
from redashsync.redash.data import Redash
from redashsync.slack.messages import Slack
from redashsync.config.yaml import Config
from datetime import datetime
from cron_converter import Cron

config = Config("./resource/sample.yml")
redashHost = config.getRedashHost()
redashQueryKey = config.getQueryApiKey()
redashQueryId = config.getQueryId()
slackBotToken = config.getSlackBotToken()

redash = Redash(redashHost)
slack = Slack(slackBotToken)

cron = Cron()
cron.from_string("*/1 * * * *") # Send every 1 minute
referenceTime = datetime.now()
schedule = cron.schedule(referenceTime)
expectedTime = schedule.next().isoformat()

while True:
    if(expectedTime == datetime.now().isoformat()[0:19]):
        print(expectedTime)
        expectedTime = schedule.next().isoformat()

        response = redash.getQueriesFromRedash(redashQueryId, redashQueryKey)
        result = redash.getColumnData(response, ['Open', 'Close'])
        # result : [ [123, 456], [789, 987], ... ]
        
        template = "*시가* : %s \n*종가* : %s \n"
        
        # 2 is inside array's length.
        result = redash.getMarkdown(template, result[0:5], 2)

        # "#redash-echo" is channel name.
        slack.sendMessage("#redash-echo", result)
```

## Send a graph image

```Python
from redashsync.redash.data import Redash
from redashsync.slack.messages import Slack
from redashsync.config.yaml import Config
from datetime import datetime
from cron_converter import Cron

config = Config("./resource/sample.yml")
redashHost = config.getRedashHost()
redashQueryKey = config.getQueryApiKey()
redashQueryId = config.getQueryId()
slackBotToken = config.getSlackBotToken()

redash = Redash(redashHost)
slack = Slack(slackBotToken)

# Histogram
response = redash.getQueriesFromRedash(redashQueryId, redashQueryKey)
result = redash.getColumnData(response, ['Open', 'Close'])

# If you want to name image one.png and locate it at ./resource 
redash.visualizeHistplotAndSave(result, "./resource/one.png")
slack.sendFile("./resource/one.png", "#redash-echo", "Title", "comment")
```



