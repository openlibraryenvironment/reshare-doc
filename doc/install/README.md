# ReShare Install, example guide
This document describes an installation of a ReShare tenant from the ground up on a single Linux server (Ubuntu 20.04 in this case). Note that this is not a production install. A production install should likely make use of a container orchestration platform like Kubernetes.

## Introduction
Project ReShare is built on the [Okapi/Stripes](https://wiki.folio.org/display/TC/Definition+of+the+Okapi-Stripes+Platform%2C+FOLIO+LSP+Base+and+FOLIO+LSP+Extended+Apps) platform developed for the FOLIO Library Services Platform project. The main components  to a single ReShare tenant are a storage backend (Postgres), a message queue (Kafka), an application backend, and a web frontend javascript bundle.

To better understand how ReShare is built, it would be useful to review the FOLIO developer's documentation available at https://dev.folio.org/start/. In particular, the [Okapi Guide](https://github.com/folio-org/okapi/blob/master/doc/guide.md) and [Stripes Guide](https://github.com/folio-org/stripes/blob/master/doc/dev-guide.md) are useful for showing how the platform fits together.

### Backend software 
There are two primary backend modules in project ReShare: mod-rs, and mod-directory. Project ReShare also makes use of a a suite of modules shared with the FOLIO project to provide functionality such as user login and permissions management.

* [mod-rs](https://github.com/openlibraryenvironment/mod-rs) handles requests
* [mod-directory]https://github.com/openlibraryenvironment/mod-directory) keeps track of the members of a resource sharing consortium.

Releases of mod-rs and mod-directory are tagged in the github repositories above and published as Docker containers on the [reshareorg](https://hub.docker.com/u/reshareorg) dockerhub organization.

### Frontend software
ReShare's UI is built on the stripes platform and comprises the applications listed in the package.json file of the [platform-rs](https://github.com/openlibraryenvironment/platform-rs/blob/v1.0.0/package.json) repository. Code for the frontend applications can be found on the [Open Library Environment github organization](https://github.com/openlibraryenvironment). Releases for each application are indicated with tags in the git repository. NPM artifacts are published on the following NPM repository: https://repository.dev-us-east-1.indexdata.com/repository/reshare-all/. 

### platform-rs
The [platform-rs](https://github.com/openlibraryenvironment/platform-rs/tree/v1.0.0) repository contains a complete list of all the software that comprises a ReShare system. There is a branch of the platform for each release that gives the specific versions of each piece of software that make up that release.

## Requirements
* Linux OS (this guide uses Ubuntu LTS 20.04, but would work for other distributions with minor adjustments)
* 8GB Ram
* Familiarity with Linux administration

## Bring up a Linux server
For the purpose of this guide we'll use a plain Ubuntu vagrant image, but skip this step if you prefer to provision a server another way. The host machine must have [Vagrant](https://www.vagrantup.com/) installed. Use a text editor to create the following Vagrantfile:

```
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-20.04"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = 8122
    vb.cpus = 2
  end
  config.vm.network "forwarded_port", guest: 9130, host: 9130
  config.vm.network "forwarded_port", guest: 80, host: 3000
end
```
Now run `vagrant up` in the directory with the Vagrant file to start the VM. This configuration, you can get shell access to the VM by running `vagrant ssh`.

## Install requirements
Install Java 11 and Nginx. Java 11 is the runtime for Okapi, and Nginx will be used to proxy the application's Javascript bundle. Note that on a production install, okapi would likely be run in a container so having a runtime environment for it other than docker would not be necessary. This guide uses Okapi for container orchestration for development purposes.
```
sudo apt update
sudo apt upgrade
sudo apt-get -y install openjdk-8-jdk nginx postgresql
sudo update-java-alternatives --jre-headless --jre --set java-1.8.0-openjdk-amd64
```

### Configure Postgresql
Configure PostgreSQL to listen on all interfaces and allow connections from all addresses (to allow Docker connections)
* Edit file /etc/postgresql/12/main/postgresql.conf to add line `listen_addresses = '*'` in the "Connection Settings" section
* Edit file /etc/postgresql/12/main/pg_hba.conf to add line `host all all 0.0.0.0/0 md5`
* Restart PostgreSQL with command `sudo systemctl restart postgresql`

### Install and configure Docker
Follow the [instructions to install Docker CE](https://docs.docker.com/engine/install/ubuntu/) on your distribution.

#### Configure Docker engine to listen on network socket
In order to allow Okapi to deploy docker containers, docker must be listening on a network socket. On a systemd system, create a file `docker-opts` in the `/etc/systemd/system/docker.service.d` directory:
```
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo vi /etc/systemd/system/docker.service.d/docker-opts.conf
```
Add the following contents to the docker-opts.conf file
```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:4243
```
Reload the service and restarte docker
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```
### Install Docker Compose
For this setup, we'll run kafka and Zookeeper with a docker-compose stack for convenience. Follow the [instructions for your OS to install docker-compose](https://docs.docker.com/compose/install/) If you choose to run Kafka another way, skip this step. 

### Add build requirements for frontend
```
sudo apt -y install git curl nodejs npm
```
Install n from npm
```
sudo npm install n -g
```
Import the Yarn signing key, add the Yarn apt repository, install Yarn
```
wget --quiet -O - https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
sudo add-apt-repository "deb https://dl.yarnpkg.com/debian/ stable main"
sudo apt-get update
sudo apt-get -y install yarn
```
### Install Kafka
Install Apache Kafka and Zookeeper. If using docker-compose, create a directory for the docker-compose stack and create a docker compose file:
```
sudo mkdir /opt/kafka-zk
cd /opt/kafka-zk
vi docker-compose.yml
```
Add the following contents to the docker-compose file:
```yaml
version: '2'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    container_name: zookeeper
    restart: always
    ports:
      - "2181:2181"
  kafka:
    image: wurstmeister/kafka
    container_name: kafka
    restart: always
    ports:
      - "9092:9092"
      - "29092:29092"
    environment:
      KAFKA_LISTENERS: INTERNAL://:9092,LOCAL://:29092
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://10.0.2.15:9092,LOCAL://localhost:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: LOCAL:PLAINTEXT,INTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_BROKER_ID: 1
      KAFKA_LOG_RETENTION_BYTES: -1
      KAFKA_LOG_RETENTION_HOURS: -1
```
Start the stack:
```
sudo docker-compose up -d
# optionally, check that kafka and zookeeper look happy
sudo docker-compose logs
cd -
```
### Create databases and roles
Login as the postgres superuser and create databases for Okapi and for the ReShare modules:
```
sudo -u postgres psql
```
At the postgres prompt:
```
CREATE ROLE okapi WITH PASSWORD 'okapi25' LOGIN CREATEDB;
CREATE DATABASE okapi WITH OWNER okapi;
CREATE ROLE reshare WITH PASSWORD 'reshare123' LOGIN SUPERUSER;
CREATE DATABASE reshare WITH OWNER reshare;
```

### Install Okapi
```
wget --quiet -O - https://repository.folio.org/packages/debian/folio-apt-archive-key.asc | sudo apt-key add -
sudo add-apt-repository "deb https://repository.folio.org/packages/ubuntu xenial/"
sudo apt update
sudo apt-get -y install okapi=3.1.2-1
sudo apt-mark hold okapi
```

#### Configure Okapi
Edit `/etc/folio/okapi/okapi.conf` to configure Okapi making the following changes:
* `role="dev"`
* `port_end="9230"`
* `host="10.0.2.15"`
* `storage="postgres"`
* `okapiurl="http://10.0.2.15:9130"`
Optionally, add a docker account to sign requests to the docker registry.

Reload and restart Okapi:
```
sudo systemctl daemon-reload
sudo systemctl restart okapi
```
## Install ReShare

### Pull module descriptors
ReShare uses modules from the FOLIO project and reshare project. Pull module descriptors to your local okapi:
```
# FOLIO module descriptors
curl -w '\n' -D - -X POST -H "Content-type: application/json" \
  -d '{"urls" : ["http://folio-registry.aws.indexdata.com"]}' \
  http://localhost:9130/_/proxy/pull/modules
# ReShare module descriptors
curl -w '\n' -D - -X POST -H "Content-type: application/json" \
  -d '{"urls" : ["https://registry.reshare-dev.indexdata.com"]}' \
  http://localhost:9130/_/proxy/pull/modules
```

### Create a tenant
```
curl -w '\n' -D - -X POST -H "Content-type: application/json" \
  -d '{"id":"reshare","name":"ReShare","description":"ReShare Tenant"}' \
  http://localhost:9130/_/proxy/tenants
```
Enable the Okapi module for the tenant
```
curl -w '\n' -D - -X POST -H "Content-type: application/json" \
  -d '{"id":"okapi"}' \
  http://localhost:9130/_/proxy/tenants/reshare/modules
```

### Build the frontend bundle
switch to node lts
```
sudo n lts
```
Clone the ReShare platform and checkout the v1.0.0 branch
```
git clone https://github.com/openlibraryenvironment/platform-rs.git
cd platform-rs
git checkout v1.0.0
```
Install npm packages
```
yarn install
```
Edit the stripes.config.js file with details about your build. If you're following this guide without modification, no changes are necessary.

Build the frontend bundle:
```
yarn build
```
### Configure a webserver to serve the frontend bundle
```
sudo cp -a output /usr/share/nginx/reshare
```
Create a new file at `/etc/nginx/sites-available/reshare` with the following contents
```
server {
  listen 80;
  server_name localhost;
  charset utf-8;
  # Serve index.html for any request not found
  location / {
    # Set path
    root /usr/share/nginx/reshare;
    include mime.types;
    types {
      text/plain lock;
    }
    try_files $uri /index.html;
  }
}
```
Now, create a link to this file in Nginx's `sites-available` directory and restart nginx:
```
sudo ln -s /etc/nginx/sites-available/reshare /etc/nginx/sites-enabled/reshare
sudo nginx -t
sudo systemctl restart nginx
```
If you're using the Vagrant box, at this point you should be able load the frontend at http://localhost:3000. There is no backend deployed yet however.

### Deploy a backend
The tagged release of platform-core contains an okapi-install.json file which, when posted to Okapi, will download all the necessary backend modules as Docker containers, deploy them to the local system, and enable them for your tenant. There is also a stripes-install.json file that will enable the frontend modules for the tenant and load the necessary permissions.

Set environment variable for deployed modules. Okapi will set these environment variables for all containers it deploys. 

```
curl -w '\n' -D - -X POST -H "Content-Type: application/json" -d "{\"name\":\"DB_HOST\",\"value\":\"10.0.2.15\"}" http://localhost:9130/_/env
curl -w '\n' -D - -X POST -H "Content-Type: application/json" -d "{\"name\":\"DB_PORT\",\"value\":\"5432\"}" http://localhost:9130/_/env
curl -w '\n' -D - -X POST -H "Content-Type: application/json" -d "{\"name\":\"DB_DATABASE\",\"value\":\"reshare\"}" http://localhost:9130/_/env
curl -w '\n' -D - -X POST -H "Content-Type: application/json" -d "{\"name\":\"DB_USERNAME\",\"value\":\"reshare\"}" http://localhost:9130/_/env
curl -w '\n' -D - -X POST -H "Content-Type: application/json" -d "{\"name\":\"DB_PASSWORD\",\"value\":\"reshare123\"}" http://localhost:9130/_/env
curl -w '\n' -D - -X POST -H "Content-Type: application/json" -d "{\"name\":\"EVENTS_PUBLISHER_BOOTSTRAP_SERVERS\",\"value\":\"10.0.2.15:9092\"}" http://localhost:9130/_/env
curl -w '\n' -D - -X POST -H "Content-Type: application/json" -d "{\"name\":\"EVENTS_CONSUMER_BOOTSTRAP_SERVERS\",\"value\":\"10.0.2.15:9092\"}" http://localhost:9130/_/env
curl -w '\n' -D - -X POST -H "Content-Type: application/json" -d "{\"name\":\"EVENTS_PUBLISHER_ZK_CONNECT\",\"value\":\"10.0.2.15:2181\"}" http://localhost:9130/_/env
curl -w '\n' -D - -X POST -H "Content-Type: application/json" -d "{\"name\":\"EVENTS_CONSUMER_ZK_CONNECT\",\"value\":\"10.0.2.15:2181\"}" http://localhost:9130/_/env
```
Enable the modules described in okapi-install.json. With the deploy parameter set to true, Okapi will download and spin up all of these containers:
```
curl -w '\n' -D - -X POST -H "Content-type: application/json"   -d @platform-rs/okapi-install.json   http://localhost:9130/_/proxy/tenants/reshare/install?deploy=true\&preRelease=false\&tenantParameters=loadReference%3Dtrue
```
Note: This will take a long time to return, as all the Docker images must be pulled from Docker Hub. Progress can be followed in the Okapi log at /var/log/folio/okapi/okapi.log and via sudo docker ps | grep -v "^CONTAINER" | wc -l

Now post the stripes frontend module list to enable those modules:
```
curl -w '\n' -D - -X POST -H "Content-type: application/json" \
  -d @platform-rs/stripes-install.json \
  http://localhost:9130/_/proxy/tenants/reshare/install?preRelease=false
```
### Create a superuser for the reshare tenant
In this guide, we'll use a script to create the superuser. This script is derived from the ansible role at: https://github.com/folio-org/folio-ansible/tree/master/roles/create-tenant-admin.

Install perl:
```
sudo apt install -y libjson-perl libwww-perl libuuid-tiny-perl
```
Download the boostrap-superuser.pl script:
```
wget https://raw.githubusercontent.com/folio-org/folio-install/master/runbooks/single-server/scripts/bootstrap-superuser.pl
```
Run the script to add a superuser:
```
perl bootstrap-superuser.pl \
  --tenant reshare --user reshare_admin \
  --password admin --okapi http://localhost:9130
```

At this point, you should be able to log into your frontend with the credentials you provided.
