**This is a work in progress. It is very much not complete, the final approach or not even necesserily a good idea yet!**

# Postal

This repository contains everything you need to start using Postal straight away on your own servers. Follow these instructions to get started quickly. The purpose of this guide is to get you up and running as quickly as possible. We *strongly* recommend you use secure passwords wherever they're used.

## Service dependencies

Postal has a few dependencies on external services which you will need to provide for it. How you do this is entirely up to you. In this example, we're simply going to start the services from Docker containers and connect to them. This is not required (nor recommended) for production installations.

### MariaDB

MariaDB is used to store all mail and configuration data for your installation. It's the core to the whole thing working well. We recommend installing a nicely tuned and replicated database for all installations. If you use a small database server with large mail volume, you will not have a nice time.

```
docker run -d \
   --name postal-mariadb \
   -p 127.0.0.1:3306:3306 \
   --restart always \
   -e MARIADB_DATABASE=postal \
   -e MARIADB_ROOT_PASSWORD=postal \
   mariadb
```

This will start a MariaDB installation on port 3306 on your server. You can connect to this using the credentials provided above. Postal needs access to create and delete databases so, in this example, we will use the `root` user.

### RabbitMQ

RabbitMQ is responsible for dispatching messages between different processes - in our case, our workers. As with MariaDB, there are numerous ways for you to install this. For this guide, we're just going to run a single RabbitMQ worker.

```
docker run -d \
   --name postal-rabbitmq \
   -p 127.0.0.1:5672:5672 \
   --restart always \
   -e RABBITMQ_DEFAULT_USER=postal \
   -e RABBITMQ_DEFAULT_PASS=postal \
   -e RABBITMQ_DEFAULT_VHOST=postal \
   rabbitmq:3
```

## Postal

Once you have your services running, you can proceed to start Postal. To begin, clone this repository to somewhere on your computer.

```
git clone https://github.com/postalhq/install /opt/postal/install
```

### Configuration

Before you can start to run any processes, you'll need some configuration. This repository contains a script that will bootstrap a config directory by creating appropriate keys and config files. You can place this wherever you want on your system.

```
cd /opt/postal/install
./bootstrap-config.sh /opt/postal/config
```

Once this has completed, go and open up `/opt/postal/config/postal.yml` and replace any values as you see fit.

### Processes

Postal has a few processes that need to run in order to function. Each of these processes can (and should) be run within containers. To make getting started as easy as possible, this repository contains a `docker-compose.sh` file which does the heavy lifting. However, if you want full control, you can use this as a template for running you own processes using whatever technology you fancy.

#### Initializing the database

Before you can do anything, you need to initialize the database schema and create your initial user. You can do this using the commands below.

```
cd /opt/postal/install
docker-compose run init_db
docker-compose run make_user
```

#### Running processes

There are a few processes that need to be running.

* A web server (`web`)
* A SMTP server (`smtp`)
* A worker (`worker`)
* A cron process (`cron`)
* A message requeueing process (`requeuer`)

The Docker Compose file includes configuration for all of these. For simplicity, we assume you are running your installation on a dedicated server and run all processes using the host's networking. This allows for

```
cd /opt/postal/install
docker-compose up -d
```