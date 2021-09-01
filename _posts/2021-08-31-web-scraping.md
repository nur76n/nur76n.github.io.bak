---
layout: post
title: "Hubot with telegram adapter"
subtitle: "Deploying hubot as service on systemd"
background: '/img/posts/hubot.png'
---
 


```
useradd -s /bin/bash -m nodejs
su - nodejs 

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
/bin/bash
nvm install v14.17.5
nvm use v14.17.5
npm install -g yo generator-hubot 

mkdir myhubot && cd $_
yo hubot

nano external-scripts.json ## Delete line hubot-redis and node-heroku-keepalive

export TELEGRAM_TOKEN=bot:token

```


nano `scripts/test.coffee`
```
module.exports = (robot) ->

  robot.hear /badger/i, (res) ->
    res.send "Badgers? BADGERS? WE DON'T NEED NO STINKIN BADGERS"

  robot.respond /multiply (.*) and (.*)/i, (res) ->
    val1 = res.match[1]
    val2 = res.match[2]

    if val1 >  50
      res.reply "Too hard"
    else
      res.reply "result is " + String(val1 * val2)

  robot.respond /time in (.*)/i, (res) ->
    today = new Date
    hour = today.getHours()
    minute = today.getMinutes()

    city = res.match[1]
    ctime = switch city
      when "Almaty","almaty","alm" then  hour + ":" + minute
      when "Moscow","moscow","msk" then (hour - 3) + ":" + minute
      when "Kiev","kiev" then (hour - 3) + ":" + minute
      when "London","london" then (hour - 5) + ":" + minute
      else "I don't know"
    res.reply "Time in " + city + " now " + ctime
```

```
bin/hubot -a telegram
```

nano `/etc/systemd/system/hubot.service`
```
[Unit]
Description=Hubot
Requires=network.target
After=network.target

[Service]
Type=simple
WorkingDirectory=/home/nodejs/myhubot
User=nodejs

Restart=always
RestartSec=10

; Configure Hubot environment variables, use quotes around vars with whitespace as shown below.
Environment=TELEGRAM_TOKEN=bot:token
Environment=PATH="/home/nodejs/.nvm/versions/node/v14.17.5/bin"
; Alternatively multiple environment variables can loaded from an external file
;EnvironmentFile=/etc/hubot-environment

ExecStart=/home/nodejs/myhubot/node_modules/.bin/coffee node_modules/hubot/bin/hubot -a telegram

[Install]
WantedBy=multi-user.target
```

```
systemctl daemon-reload 
systemctl start hubot.service
systemctl status hubot.service
```


![Скриншот](https://i.ibb.co/4KNxKGV/rb-hubot.png)