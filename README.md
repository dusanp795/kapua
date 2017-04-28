![Eclipse Kapua™ logo](docs/user-manual/en/images/kapua-logo.png)

# Eclipse Kapua&trade;

[![Build](https://api.travis-ci.org/eclipse/kapua.svg)](https://travis-ci.org/eclipse/kapua/) [![Hudson](https://img.shields.io/jenkins/s/https/hudson.eclipse.org/kapua/job/Develop.svg)](https://hudson.eclipse.org/kapua/)

[Eclipse Kapua&trade;](http://eclipse.org/kapua) is a modular platform providing the services required to manage IoT gateways and smart edge devices. Kapua provides a core integration framework and an initial set of core IoT services including a device registry, device management services, messaging services, data management, and application enablement.


## Quick Start Guide

Running an Eclipse Kapua&trade; instance on your local machine is a really simple task. There's no need to install
IDEs and build tools to start a demo.

Eclipse Kapua&trade; runs as distributed application that exposes three basic services:
* A Messaging Service
* A Web Administration Console
* A RESTful API

Two more backend services are required that implement the data tier:
* A SQL database
* A NoSQL datastore

Eclipse Kapua&trade; can be deployed in a variety of modes. A practical way for running a local demo is through Docker containers.

### Requirements

Before starting, check that your environment has the following prerequisites:

* 64 bit architecture
* Java VM Version 8
* Docker Version 1.2+
* Internet Access (needed to download the artifacts)

### Demo Setup

Follow the steps below to setup and run a Kapua instance.

* Choose the Kapua version you want to test.

The team maintains some docker images in a Docker Hub repository at [Kapua Repository](https://hub.docker.com/r/kapua/). Check the repo to view the list of available images, if you haven't found one fitting your needs you may create your own. Please refer to the paragraph [More deployment info](# More deployment info) to find more about creating your own images and/or alternative demo deployment scenarios.

***
**Note :** the Docker Hub repository mentioned above is not the official project repository from Eclipse Foundation.
***

Suppose the target is the current milestone release 0.2.0-M1.

* Run Docker

* Open an OS shell
* Execute the following command lines

```
docker run -td --name kapua-sql -p 8181:8181 -p 3306:3306 kapua/kapua-sql:0.2.0-M1
docker run -td --name kapua-elasticsearch -p 9200:9200 -p 9300:9300 elasticsearch:2.4 -Dcluster.name=kapua-datastore -Ddiscovery.zen.minimum_master_nodes=1
docker run -td --name kapua-broker --link kapua-sql:db --link kapua-elasticsearch:es -p 1883:1883 -p 61614:61614 kapua/kapua-broker:0.2.0-M1
docker run -td --name kapua-console --link kapua-sql:db --link kapua-broker:broker --link kapua-elasticsearch:es -p 8080:8080 kapua/kapua-console-jetty:0.2.0-M1
docker run -td --name kapua-api --link kapua-sql:db --link kapua-broker:broker --link kapua-elasticsearch:es -p 8081:8080 kapua/kapua-api-jetty:0.2.0-M1
```

The command lines above start a Docker container for each of the services mentioned above.

If the tag is not specified then the image tagged as _latest_ will be used by default.

The images will be downloaded from Docker Hub and all the containers will be started.

You can check if every container is ok by typing the following command:

```
docker ps -as
```

The system will show the containers currently running:

```
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                                                                  NAMES
f5e2b6ccabf3        kapua/kapua-api-jetty       "/home/kapua/run-j..."   8 minutes ago       Up 8 minutes        8778/tcp, 0.0.0.0:8081->8080/tcp                                       kapua-api
47c973b1241e        kapua/kapua-console-jetty   "/home/kapua/run-c..."   19 minutes ago      Up 19 minutes       0.0.0.0:8080->8080/tcp, 8778/tcp                                       kapua-console
915eeb9668d9        kapua/kapua-broker          "/maven/bin/active..."   19 minutes ago      Up 19 minutes       8778/tcp, 0.0.0.0:1883->1883/tcp, 0.0.0.0:61614->61614/tcp, 8883/tcp   kapua-broker
dc682f7b1533        elasticsearch:2.4           "/docker-entrypoin..."   19 minutes ago      Up 19 minutes       0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp                         kapua-elasticsearch
42d408470cf0        kapua/kapua-sql             "/home/kapua/run-h2"     20 minutes ago      Up 20 minutes       0.0.0.0:3306->3306/tcp, 0.0.0.0:8181->8181/tcp, 8778/tcp               kapua-sql
```

**Note:** in subsequent runs, before launching the new containers, ensure that there are no other containers already running.

### Access

Once the containers are running, the Kapua services can be accessed. Kapua is a multi tenant
system. The demo installation comes with one default default tenant, called _kapua-sys_, which is also the root tenant. In Eclipse Kapua a tenant is commonly referred to as an _account_.

#### The console

The administration console is available at http://localhost:8080/. Copy paste the URL above to a Web browser, as the login screen appears, type the following credentials:

* Username: _kapua-sys_
* Password: _kapua-password_

Press _Login_ button and start working with the console.

#### RESTful APIs

The documentation of RESTful API is available at http://localhost:8081/doc/ while the mount points are available at http://localhost:8081/v1/.

The documentation is available through Swagger UI which greatly helps testing and exploring the exposed services.

In order to get access a REST resource through an API, an authentication token is needed. Fortunately the token can be easily obtained by executing the authentication API. There are several ways to invoke API, likely the easiest is by using the Swagger UI:

* Open the URL http://localhost:8081/doc/
* Select item _Authentication_
* Select item _/authentication/user_
* Using the test feature run 'POST /authentication/user' by specifying the following body:

```
{
  "password" : [ "kapua-password" ],
  "username": "kapua-sys"
}
```

**Note:** as an alternative to the previous, if curl is available on your machine, execute the following from the shell:

```
curl -X POST  'http://localhost:8081/v1/authentication/user' --header 'Content-Type: application/json' --header 'Accept: application/json' -d '{
  "password": [
    "kapua-password"
  ],
  "username": "kapua-sys"
}'
```

The system will return a JSON object.

* Copy the value of the field _tokenId_
* Paste the value in the box labelled _api\_key_ at the top of the web page

Swagger will automatically add the authentication token to each subsequent request done using the Swagger API. You're ready to try executing the documented APIs.

#### The Broker

The broker container exposes an [Mqtt](http://mqtt.org/) end point at tcp://localhost:1883/.
The broker can be accessed through either a plain Mqtt client like [Eclipse Paho](http://www.eclipse.org/paho/) for example, or an higher level client like [Eclipse Kura](http://www.eclipse.org/kura/).

In order for a client to establish an Mqtt connection with the broker, a client must provide a valid identity. The _kapua-sys_ account provides the user named _kapua-broker_ which has been pre-seeded and profiled for the purpose.

The credentials for the user kapua-broker are the following:

* Username: _kapua-broker_
* Password: _kapua-password_

**Note:** do not use the user kapua-sys to establish Mqtt connections.

#### More deployment info
Installing and running a demo using Docker is easy but it's not the only way. There are other scenarios that the users may be interested to. We provide advanced setup scenarios in the following guides:

* [Running with Docker](assembly/README.md)
* [Running with Vagrant](dev-tools/src/main/vagrant/README.md#demo-machine-quick-start)
* [Running with OpenShift](docs/developer-guide/en/running.md#openshift)

They will provide more advanged deployment scenarios.

### User & Developer guides

* [User Manual](http://download.eclipse.org/kapua/docs/develop/user-manual/en)
* [Developer Guide](http://download.eclipse.org/kapua/docs/develop/developer-guide/en)

### Contributing

If you're interested to get involved in IoT and Eclipse Kapua&trade; project, join the community and give your contribution to the project, please read [how to contribute to Eclipse Kapua&trade;](https://github.com/eclipse/kapua/blob/develop/CONTRIBUTING.md).
