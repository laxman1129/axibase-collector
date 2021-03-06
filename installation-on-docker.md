# Installation on Docker

## Host Requirements

* [Docker Engine](https://docs.docker.com/engine/installation/) 1.7+

## Image Information

* Image name: axibase/collector:latest
* [Dockerfile](https://github.com/axibase/docker-axibase-collector/blob/master/Dockerfile)
* [Docker Hub](https://hub.docker.com/r/axibase/collector/)

## Importing an Image in Restricted Environments

If the target Docker host has no direct connectivity to [Docker Hub](https://hub.docker.com), execute the following steps to prepare and load the Collector image:

* Login into a Docker host which is connected to Docker Hub.
* Pull the Collector image from Docker Hub and export it into an archive file:

```sh
docker pull axibase/collector:latest
docker save -o docker-axibase-collector.tar axibase/collector:latest
gzip docker-axibase-collector.tar
```

* Copy the `docker-axibase-collector.tar.gz` archive to the target Docker host.
* Import the image from the archive:

```sh
docker load < docker-axibase-collector.tar.gz
```

Alternatively, you can download a pre-built image file from [axibase.com](https://axibase.com/public/docker-axibase-collector.tar.gz).

## Start Container

> Using Collector to monitor Docker? Launch container in privileged mode as described in this [document](./jobs/docker.md#local-collection).

```properties
docker run \
 --detach \
 --publish-all \
 --restart=always \
 --name=axibase-collector \
 axibase/collector:latest
```

To automatically configure a connection to the Axibase Time Series Database, add the `-atsd-url` parameter containing the ATSD hostname and https port (default 8443), as well as [collector account](https://github.com/axibase/atsd/blob/master/administration/collector-account.md) credentials:

```properties
docker run \
 --detach \
 --publish-all \
 --restart=always \
 --name=axibase-collector \
 axibase/collector:latest \
  -atsd-url=https://collector-user:collector-password@atsd_host:atsd_https_port
```

If the user name or password contains a `$`, `&`, `#`, or `!` character, escape it with backslash `\`.

The password must contain at least **six** (6) characters and is subject to the following [requirements](https://github.com/axibase/atsd/blob/master/administration/user-authentication.md#password-requirements).

For example, for user `adm-dev` with the password `my$pwd` sending data to ATSD at https://10.102.0.6:8443, specify:

```properties
docker run \
 --detach \
 --publish-all \
 --restart=always \
 --name=axibase-collector \
 axibase/collector:latest \
  -atsd-url=https://adm-dev:my\$pwd@10.102.0.6:8443
```

## Start Container in Privileged Mode

The launch command is different if the Collector container is used to [monitor statistics](./jobs/docker.md#local-collection) from the local Docker Engine.

## Launch Parameters

**Name** | **Required** | **Description**
----- | ----- | -----
`--detach` | Yes | Run container in background and print container id.
`--publish-all` | No | Publish exposed https port (9443) to a random port.
`--restart` | No | Auto-restart policy. _Not supported in all Docker Engine versions._
`--name` | No | Assign a host-unique name to the container.

To bind the collector to a particular port instead of a random one, replace `--publish-all` with `--publish 10443:9443`, where the first number indicates an available port on the Docker host.

## Check Installation

It may take up to 5 minutes to initialize the application.

```sh
docker exec -it axibase-collector tail -f /opt/axibase-collector/logs/axibase-collector.log
```

Wait until the following message appears:

> _FrameworkServlet 'dispatcher': initialization completed._

This message indicates that the initial configuration is complete.

## Validation

```sh
docker ps | grep axibase-collector
```

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                     NAMES
ee15099d9f88        axibase/collector   "/bin/bash /opt/axiba"   33 seconds ago      Up 32 seconds       0.0.0.0:32769->9443/tcp   axibase-collector
```

Take note of the public https port assigned to axibase-collector container, i.e. **32769** in the example above.

## Login

Open https://docker_hostname:32769 in your browser and create an [administrator account](configure-administrator-account.md). 

`docker_hostname` is the hostname or IP address of the Docker host and **32769** is the external port number assigned to the Collector container in the previous step.

## Setup ATSD Connection

Configure the [ATSD Server connection](atsd-server-connection.md) to send data into an Axibase Time Series Database instance.

## Troubleshooting

Review the following log files for any errors:

```sh
docker exec -it axibase-collector tail -n 100 /opt/axibase-collector/logs/axibase-collector.log
```

```sh
docker exec -it axibase-collector tail -n 100 /opt/axibase-collector/logs/err-collector.log
```

## Known Issues

If the container fails to start, verify that the Docker host runs on a supported kernel level.

```sh
uname -a
```

* 3.13.0-79.123+
* 3.19.0-51.57+
* 4.2.0-30.35+

See "Latest Quick Workarounds" for Docker issue [#18180](https://github.com/docker/docker/issues/18180#issuecomment-193708192).
