---
title: "Getting Started"
linkTitle: "Getting Started"
weight: 2
description: >
  What do I need to get started with Natlas?
---

Natlas is a distributed platform with multiple components. As such, the setup process is a little more complicated than simply downloading and running a bash script. This page should help you determine if you have the right resources on hand to setup the platform.

## Prerequisites

* Docker
* At least one elasticsearch node
* A set of ip networks to scan

## Installation

The Natlas server and the Natlas agents are all deployed using Docker. This makes it easy for us to provide a consistent, repeatable environment to work from without polluting the underlying system with dependencies. The instructions that follow are in the order that they should be performed.

### Elasticsearch

If you don't already have an elasticsearch node to use, this step must be completed first. If you've never used Elasticsearch before, follow [Elastic's instructions] for setting up a single node cluster with docker. Make sure to pay attention to the section on persisting the elastic data.

An example to run Elastic and expose port 9200 to the host is below. It assumes that `/data/elastic` is a directory that already exists, and it's where the Elastic persistent data will be stored.

```bash
docker run --name natlas_elastic -d -p 127.0.0.1:9200:9200 -p 127.0.0.1:9300:9300 -v /data/elastic:/usr/share/elasticsearch/data -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.8.0
```

**Security Note:** Docker may punch holes in your iptables firewall automatically and without notice via the DOCKER chain. It is very important that you secure your Elasticsearch cluster so that only your application can talk to it. Elasticsearch believes that fancy features like "authentication" and concepts like "secure by default" should be proprietary expansions, rather than baked into their open source product. If you fail to secure your Elasticsearch node, you may find yourself the victim of a [Meow attack]. This is me speaking from first-hand experience. So be sure to nmap yourself from another host on 9200,9300 when you've finished setting up.

### (Optional) Mysql

If you have an existing mysql service that you'd like to make use of, Natlas supports that via the `SQLALCHEMY_DATABASE_URI` configuration option. If you don't, Natlas will use a sqlite database on disk automatically.

If you do plan to use Mysql, use this format for the database URI:

```bash
SQLALCHEMY_DATABASE_URI=mysql://natlas_user:natlas_password@mysql_server/natlas
```

This assumes:

* `natlas_user` is a user that can write to the database
* `natlas_password` is a unique password for the `natlas_user` account
* `mysql_server` is the address of your mysql server
* `natlas` is the name of the database that `natlas_user` will be interacting with

### Natlas Server

Once your Elasticsearch node is up and running, you can start to setup the server. Natlas server builds are available on [Dockerhub](https://hub.docker.com/r/natlas/server). For this guide, we're going to go ahead and grab the latest build. But before we do, there are a couple house keeping tasks that need to be done. We need:

1. A place to store persistent data
1. An environment file that we can pass to the Natlas application for configuration

For this guide, we'll assume that persistent data is stored at `/data/natlas-server`. This will contain the sqlite database (if applicable) and the screenshot images that have been acquired by the platform.

All that's left before we launch is the `.env` file. For this example, we're keeping it at `/data/natlas-server/server-env`, that way it's next to the rest of the natlas data. This is a minimal env file to get started, which uses a local sqlite database and doesn't configure a mail server.

```bash
mkdir -p /data/natlas-server
echo "SECRET_KEY=im-a-really-long-secret-you-should-set-one-and-never-share-it" >> /data/natlas-server/server-env
echo "ELASTICSEARCH_URL=http://<the address of your elasticsearch node>" >> /data/natlas-server/server-env
```

Now that we've got our persistent storage setup and our environment file ready, launch the server like so:

```bash
docker run -d -p 127.0.0.1:5000:5000 --restart=always -v /data/natlas-server:/data:rw -v /data/natlas-server/server-env:/opt/natlas/natlas-server/.env:ro --name natlas_server natlas/server
```

**Security Note:** This step only installs the application, which does not support things like TLS, security headers, or asset caching. You should use consider using [nginx as a reverse proxy] to adjust for this so that your agents and your users can have a safe way to access your Natlas server.

Before you continue to the Natlas Agent section, we need to configure the server to prepare it for it's first agent.

#### Configuring Natlas Server

You'll need to create an admin user. This requires knowing the server address that you'll be accessing natlas from (such as natlas.example.com). Assuming you've named your container `natlas_server`, you can create your admin user using the following command.

```bash
$ docker exec -e SERVER_NAME=natlas.example.com -it natlas_server flask user new --admin
Accept invitation: http://natlas.example.com/auth/invite?token=this-is-an-invalid-example-token
```

You'll need to get agent credentials so that you can deploy your first agent. Once you've accepted your invite, you can get credentials by visiting the `/user` page and clicking the `New Agent` button.

Finally, you'll need to configure some scanning scope. This can be done via the `/admin/scope` webpage.

### Natlas Agent

Hopefully everything has gone swimmingly this far, we're almost ready to start getting our scan data. We just need to deploy the agents that actually do the work now. This process has a couple more steps than the previous.

1. We'll need an env file, like the server, which tells the agent how to communicate with the server.
1. We'll need a seccomp profile so that chrome can run safely inside the container
1. We'll need an agent id and and agent token from the server so that we can authenticate to it. (Make sure you've completed the [Configuring Natlas Server] steps)

We're going to assume that we're centralizing the natlas data in the `/data` directory from the previous step, so that's where we'll create our agent env file.

```bash
mkdir -p /data/natlas-agent
echo "NATLAS_SERVER_ADDRESS=https://natlas.example.com" >> /data/natlas-agent/agent-env
echo "NATLAS_MAX_THREADS=25" >> /data/natlas-agent/agent-env
echo "NATLAS_AGENT_ID=this-is-an-example-id" >> /data/natlas-agent/agent-env
echo "NATLAS_AGENT_TOKEN=this-is-an-example-token" >> /data/natlas-agent/agent-env
```

Now that we've got our agent config file setup and we've added our credentials, let's grab the seccomp profile and get ready to start scanning.

```bash
wget https://raw.githubusercontent.com/natlas/natlas/main/natlas-agent/chrome.json -O /data/natlas-agent/chrome.json
```

Finally, it's time to grab the agent and launch it. Be sure to include the `--security-opt seccomp` option and the `--cap-add=NET_ADMIN`, otherwise your agent will not behave as expected. The former allows chrome to take screenshots, and the latter allows nmap to actually do port scans.

```bash
docker run -d --name natlas_agent --restart=always --security-opt seccomp=/data/natlas-agent/chrome.json --cap-add=NET_ADMIN -v /data/natlas-agent/agent-env:/opt/natlas/natlas-agent/.env natlas/agent
```

## Conclusion

Congratulations, if you followed all of the instructions this far, you should start getting scan data rolling in. You can confirm this by viewing `/browse` on your server installation.

[Elastic's instructions]: https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html
[Meow attack]: https://www.bleepingcomputer.com/news/security/new-meow-attack-has-deleted-almost-4-000-unsecured-databases/
[Configuring Natlas Server]: #configuring-natlas-server
[nginx as a reverse proxy]: /docs/getting-started/nginx-as-a-reverse-proxy/
