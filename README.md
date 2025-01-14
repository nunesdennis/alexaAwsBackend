This repository contains the backend code for homebridge-alexa

# homebridge-alexa

These are my notes and backlog for the creating of the Skill Based approach for integrating Amazon Alexa with HomeBridge.  Also all code for this version will use this branch of the repository.

# Concept

          -------------------
          | Alexa HomeSkill |
          -------------------
                  |
                 \|/
          -------------------
          | website         |
          -------------------
                 /|\
                  |
          ---------------------
          | Homebridge Plugin |
          ---------------------
          | HAPNodeJS         |
          ---------------------

```
Alexa --> HomeBridge --(webservice)--> WebSite <--(MQTT)--> HomeBridge --(WebService)--> (HAP-NodeJS)
          HomeSkill                                         Plugin
```

# End to End Design

![Diagram](docs/Homebridge-Alexa.001.jpeg)

# Plugin Design

![Diagram](docs/Homebridge-Alexa.002.jpeg)

HomeBridge HomeSkill sends alexa directives to website, website uses endpoint.scope.token to lookup account, and mqtt topic of account.  Website sends alexa directive to HomeBridge plugin via MQTT.  Plugin uses endpoint.endpointid to determine HAP instance, and create HAP request.

HomeBridge plugin has a module that generates events for each directive.  Events name based on directive.header.namespace ( ie Alexa.Discovery ), but with 'Alexa.' removed.

My inspiration for the design is based on the work done to create a Alexa Skill for Node Red by Ben Hardill.  You read the details here: http://www.hardill.me.uk/wordpress/2016/11/05/alexa-home-skill-for-node-red/

# backlog

Moved to https://github.com/NorthernMan54/alexaAwsBackend/issues/6

# Setup Development Toolchain

## Alexa Lambda HomeSkill

* Followed this - https://developer.amazon.com/blogs/post/Tx1UE9W1NQ0GYII/Publishing-Your-Skill-Code-to-Lambda-via-the-Command-Line-Interface

1 - Name - AlexaHome
2 - Runtime - NodeJS 6.10
3 - Role - Choose existing
4 - Existing Role - lamba_basic_execution
Create function
5 - Add trigger - Alexa Smart Home
6 - Application ID - Copy from Alexa config
save
7 - Timeout - 10 Seconds

9 - Add ARN to Alexa Skill config

** Environment Variables

ENDPOINT - https://homebridge.cloudwatch.net/
DEBUG - ( empty or true )




## aws LightSail web website

* Selected Ubuntu OS image, and installed nodejs

```
curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
sudo apt-get install -y nodejs
```

* install mongodb mosquitto mosquitto-auth-plugin

```
sudo apt-get install apache2 mongodb unzip
```

apt-get build-dep mosquitto mosquitto-auth-plugin
sudo apt-get install dpkg-dev
sudo apt-get install libmongoc-developer
sudo apt-get install libbson-dev

Installed from source

mongo-c-driver-1.9.2 - http://mongoc.org/libmongoc/current/installing.html
mosquitto-1.4.14
mosquitto-auth-plug-0.1.2

cd mosquitto-auth-plugin
vi config.mk - enable mongo and files
make
sudo cp auth-plug.so /usr/lib/mosquitto-auth-plugin/auth-plugin.so

## mosquitto Config

cp mosquitto/conf/mosquitto.conf /etc/mosquitto/conf.d/mosquitto.conf

### Growing over 1024 Users

1. Added `ulimit -n 60000` to /etc/init.d/mosquitto

2. Added to /etc/security/limits.conf

```
mosquitto 	hard	nofile 		10000000
mosquitto 	soft	nofile 		10000000
```

## Apache SSL Config

* Registered IP Address at freeDNS - homebridge.cloudwatch.net

1. Followed the instructions here http://freedns.afraid.org/scripts/afraid.aws.sh.txt, but used this script as it worked better.  http://freedns.afraid.org/scripts/update.sh.txt

2. URL/hostname is coded in /etc/cron.d/afraid.aws.sh

* Create SSL at Let's Encrypt

This is wrong

```
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-apache
sudo /opt/bitnami/ctlscript.sh stop apache
sudo certbot certonly
  2
  homebridge.cloudwatch.net

- Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/homebridge.cloudwatch.net/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/homebridge.cloudwatch.net/privkey.pem
   Your cert will expire on 2018-04-28. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"

From https://docs.bitnami.com/google/how-to/generate-install-lets-encrypt-ssl/
```

## Enable storage of more logs ##

```
sudo mkdir /var/log/journal
sudo systemd-tmpfiles --create --prefix /var/log/journal
sudo systemctl restart systemd-journald
```

## Local version of awsWebsite

```
brew install mongo; brew services start mongodb
brew install mosquitto; brew services start mosquitto
```

# Amazon AWS Lambda function

## Create AWS Lambda function

1. Name - AlexaHome
2. Version - Node.js 6.1.0
3. Role - Choose an existing Role
4. Existing Role - lambda_basic_execution
5. Trigger - Alexa Smart Home

# Historical Dates

* Beta test started - Feb 19, 2018
* Production launch - March 14, 2018
* Added Germany and France - March 28, 2018
* Added Italian (IT), English (IN), English (AU), Spanish (ES), Japanese (JP), Spanish (MX), French (CA) - Feb 13th, 2019

# Credits

* Ben Hardill - For the inspiration behind the design.
