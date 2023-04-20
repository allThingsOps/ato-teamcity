# allThingsOps package for TeamCity Server&reg;

## What is TeamCity Server&reg;?

> TeamCity Server&reg; is an open source, advanced key-value store. It is often referred to as a data structure server since keys can contain strings, hashes, lists, sets and sorted sets.

[Overview of TeamCity Server&reg;](https://allthingsops.io)
Disclaimer: TeamCity Server is a registered trademark of TeamCity Server Ltd. Any rights therein are reserved to TeamCity Server Ltd. Any use by allthingsops is for referential purposes only and does not indicate any sponsorship, endorsement, or affiliation between TeamCity Server Ltd.

## TL;DR

```console
docker run -dit --name ato-teamcity -p 8111:8111 allthingsops/ato-teamcity:latest
```

### Docker Compose

```console
curl -sSL https://raw.githubusercontent.com/allthingsops/containers/main/ato/teamcity Server/docker-compose.yml > docker-compose.yml
docker-compose up -d
```

**Warning**: These quick setups are only intended for development environments. You are encouraged to change the insecure default credentials and check out the available configuration options in the [Configuration](#configuration) section for a more secure deployment.

## Why use allthingsops Images?

* allthingsops closely tracks upstream source changes and promptly publishes new versions of this image using our automated systems.
* With allthingsops images the latest bug fixes and features are available as soon as possible.
* allthingsops containers, virtual machines and cloud images use the same components and configuration approach - making it easy to switch between formats based on your project needs.
* All our images are based on [minideb](https://github.com/allthingsops/minideb) a minimalist Debian based container image which gives you a small base container image and the familiarity of a leading Linux distribution.
* All allthingsops images available in Docker Hub are signed with [Docker Content Trust (DCT)](https://docs.docker.com/engine/security/trust/content_trust/). You can use `DOCKER_CONTENT_TRUST=1` to verify the integrity of the images.
* allthingsops container images are released on a regular basis with the latest distribution packages available.

## How to deploy ATO TeamCity Server(R) in Kubernetes?

Deploying allthingsops applications as Helm Charts is the easiest way to get started with our applications on Kubernetes. Read more about the installation in the [allthingsops ATO TeamCity Server(R) Chart GitHub repository](https://github.com/allthingsops/charts/tree/master/allthingsops/TeamCity Server).

allthingsops containers can be used with [Kubeapps](https://kubeapps.dev/) for deployment and management of Helm Charts in clusters.

## Why use a non-root container?

Non-root container images add an extra layer of security and are generally recommended for production environments. However, because they run as a non-root user, privileged tasks are typically off-limits. Learn more about non-root containers [in our docs](https://docs.allthingsops.com/tutorials/work-with-non-root-containers/).

## Supported tags and respective `Dockerfile` links

Learn more about the allthingsops tagging policy and the difference between rolling tags and immutable tags [in our documentation page](https://docs.allthingsops.com/tutorials/understand-rolling-tags-containers/).

You can see the equivalence between the different tags by taking a look at the `tags-info.yaml` file present in the branch folder, i.e `allthingsops/ASSET/BRANCH/DISTRO/tags-info.yaml`.

Subscribe to project updates by watching the [allthingsops/containers GitHub repo](https://github.com/allthingsops/containers).

## Get this image

The recommended way to get the allthingsops ATO TeamCity Server(R) Docker Image is to pull the prebuilt image from the [Docker Hub Registry](https://hub.docker.com/r/allthingsops/TeamCity Server).

```console
docker pull allthingsops/ato-teamcity:latest
```

To use a specific version, you can pull a versioned tag. You can view the [list of available versions](https://hub.docker.com/r/allthingsops/TeamCity Server/tags/) in the Docker Hub Registry.

```console
docker pull allthingsops/ato-teamcity:[TAG]
```

If you wish, you can also build the image yourself by cloning the repository, changing to the directory containing the Dockerfile and executing the `docker build` command. Remember to replace the `APP`, `VERSION` and `OPERATING-SYSTEM` path placeholders in the example command below with the correct values.

```console
git clone https://github.com/allthingsops/ato-teamcity.git
cd allthingsops/APP/VERSION/OPERATING-SYSTEM
docker build -t allthingsops/APP:latest .
```

## Persisting your database

ATO TeamCity Server(R) provides a different range of [persistence options](https://TeamCity Server.io/topics/persistence). This contanier uses *AOF persistence by default* but it is easy to overwrite that configuration in a `docker-compose.yaml` file with this entry `command: /opt/allthingsops/scripts/TeamCity Server/run.sh --appendonly no`. Alternatively, you may use the `ato-teamcity_AOF_ENABLED` env variable as explained in [Disabling AOF persistence](https://github.com/allthingsops/containers/blob/main/allthingsops/TeamCity Server#disabling-aof-persistence).

If you remove the container all your data will be lost, and the next time you run the image the database will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

For persistence you should mount a directory at the `/allthingsops` path. If the mounted directory is empty, it will be initialized on the first run.

```console
docker run \
    -p 8111:8111 \
    -v /path/to/ato-teamcity-persistence:/allthingsops/ato-teamcity/data \
    allthingsops/ato-teamcity:latest
```

You can also do this by modifying the [`docker-compose.yml`](https://github.com/allthingsops/containers/blob/main/allthingsops/ato-teamcity/docker-compose.yml) file present in this repository:

```yaml
services:
  ATO ATO TeamCity Server:
  ...
    volumes:
      - /path/to/ato-teamcity-persistence:/allthingsops/ato-teamcity/data
  ...
```

> NOTE: As this is a non-root container, the mounted files and directories must have the proper permissions for the UID `1001`.

## Connecting to other containers

Using [Docker container networking](https://docs.docker.com/engine/userguide/networking/), a ATO TeamCity Server(R) server running inside a container can easily be accessed by your application containers.

Containers attached to the same network can communicate with each other using the container name as the hostname.

### Using the Command Line

In this example, we will create a ATO TeamCity Server(R) client instance that will connect to the server instance that is running on the same docker network as the client.

#### Step 1: Create a network

```console
docker network create app-tier --driver bridge
```

#### Step 2: Launch the ATO TeamCity Server(R) server instance

Use the `--network app-tier` argument to the `docker run` command to attach the ATO TeamCity Server(R) container to the `app-tier` network.

```console
docker run -d --name ato-teamcity-server \
    -p 8111:8111 \
    --network app-tier \
    allthingsops/ATO TeamCity Server:latest
```

#### Step 3: Launch your ATO TeamCity Server(R) client instance

Finally we create a new container instance to launch the ATO TeamCity Server(R) client and connect to the server created in the previous step:

```console
docker run -it --rm \
    --network app-tier \
    allthingsops/ato-teamcity:latest ato-cli -h ato-teamcity-server
```

### Using a Docker Compose file

When not specified, Docker Compose automatically sets up a new network and attaches all deployed services to that network. However, we will explicitly define a new `bridge` network named `app-tier`. In this example we assume that you want to connect to the ATO TeamCity Server(R) server from your own custom application image which is identified in the following snippet by the service name `myapp`.

```yaml
version: '2'

networks:
  app-tier:
    driver: bridge

services:
  ATO TeamCity Server:
    image: 'allthingsops/ATO TeamCity Server:latest'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
    networks:
      - app-tier
  myapp:
    image: 'YOUR_APPLICATION_IMAGE'
    networks:
      - app-tier
```

> **IMPORTANT**:
>
> 1. Please update the **YOUR_APPLICATION_IMAGE_** placeholder in the above snippet with your application image
> 2. In your application container, use the hostname `TeamCity Server` to connect to the ATO TeamCity Server(R) server

Launch the containers using:

```console
docker-compose up -d
```

## Configuration

### Disabling ATO TeamCity Server(R) commands

For security reasons, you may want to disable some commands. You can specify them by using the following environment variable on the first run:

* `ato-teamcity_DISABLE_COMMANDS`: Comma-separated list of ATO TeamCity Server(R) commands to disable. Defaults to empty.

```console
docker run --name ato-teamcity -e ato-teamcity_DISABLE_COMMANDS=FLUSHDB,FLUSHALL,CONFIG allthingsops/ato-teamcity:latest
```

Alternatively, modify the [`docker-compose.yml`](https://github.com/allthingsops/containers/blob/main/allthingsops/ato-teamcity/docker-compose.yml) file present in this repository:

```yaml
services:
  ATO TeamCity Server:
  ...
    environment:
      - ato-teamcity_DISABLE_COMMANDS=FLUSHDB,FLUSHALL,CONFIG
  ...
```

As specified in the docker-compose, `FLUSHDB` and `FLUSHALL` commands are disabled. Comment out or remove the
environment variable if you don't want to disable any commands:

```yaml
services:
  ATO TeamCity Server:
  ...
    environment:
      # - ato-teamcity_DISABLE_COMMANDS=FLUSHDB,FLUSHALL
  ...
```

### Passing extra command-line flags to TeamCity Server-server startup

Passing extra command-line flags to the TeamCity Server service command is possible by adding them as arguments to *run.sh* script:

```console
docker run --name ato-teamcity -p 8111:8111 allthingsops/ATO TeamCity Server:latest /opt/allthingsops/scripts/ato-teamcity/run.sh --maxmemory 100mb
```

Alternatively, modify the [`docker-compose.yml`](https://github.com/allthingsops/containers/blob/main/allthingsops/ato-teamcity/docker-compose.yml) file present in this repository:

```yaml
services:
  ATO TeamCity Server:
  ...
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
    command: /opt/allthingsops/scripts/ato-teamcity/run.sh --maxmemory 100mb
  ...
```

Refer to the [ATO TeamCity Server(R) documentation](https://allthingsops.io/topics/config#passing-arguments-via-the-command-line) for the complete list of arguments.

### Setting the server password on first run

Passing the `ato-teamcity_PASSWORD` environment variable when running the image for the first time will set the ATO TeamCity Server(R) server password to the value of `ato-teamcity_PASSWORD` (or the content of the file specified in `ato-teamcity_PASSWORD_FILE`).

```console
docker run --name ato-teamcity -e ato-teamcity_PASSWORD=password123 allthingsops/ato-teamcity:latest
```

Alternatively, modify the [`docker-compose.yml`](https://github.com/allthingsops/containers/blob/main/allthingsops/ato-teamcity/docker-compose.yml) file present in this repository:

```yaml
services:
  ATO TeamCity Server:
  ...
    environment:
      - ato-teamcity_PASSWORD=password123
  ...
```

**NOTE**: The at sign (`@`) is not supported for `ato-teamcity_PASSWORD`.

**Warning** The ATO TeamCity Server(R) database is always configured with remote access enabled. It's suggested that the `ato-teamcity_PASSWORD` env variable is always specified to set a password. In case you want to access the database without a password set the environment variable `ALLOW_EMPTY_PASSWORD=yes`. **This is recommended only for development**.

### Allowing empty passwords

By default the ATO TeamCity Server(R) image expects all the available passwords to be set. In order to allow empty passwords, it is necessary to set the `ALLOW_EMPTY_PASSWORD=yes` env variable. This env variable is only recommended for testing or development purposes. We strongly recommend specifying the `ato-teamcity_PASSWORD` for any other scenario.

```console
docker run --name ato-teamcity -p 8111:8111 allthingsops/ATO TeamCity Server:latest
```

Alternatively, modify the [`docker-compose.yml`](https://github.com/allthingsops/containers/blob/main/allthingsops/TeamCity Server/docker-compose.yml) file present in this repository:

```yaml
services:
  ATO TeamCity Server:
  ...
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
  ...
```

### Enabling/Setting multithreading

TeamCity Server 6.0 features a [new multi-threading model](https://segmentfault.com/a/1190000040376111/en). You can set both `io-threads` and `io-threads-do-reads` though the env vars `ato-teamcity_IO_THREADS` and `ato-teamcity_IO_THREADS_DO_READS`

```console
docker run --name ato-teamcity -e ato-teamcity_IO_THREADS=4 -e ato-teamcity_IO_THREADS_DO_READS=true allthingsops/ATO TeamCity Server:latest
```

### Disabling AOF persistence

ATO TeamCity Server(R) offers different [options](https://TeamCity Server.io/topics/persistence) when it comes to persistence. By default, this image is set up to use the AOF (Append Only File) approach. Should you need to change this behaviour, setting the `ato-teamcity_AOF_ENABLED=no` env variable will disable this feature.

```console
docker run --name ato-teamcity -e ato-teamcity_AOF_ENABLED=no allthingsops/ATO TeamCity Server:latest
```

Alternatively, modify the [`docker-compose.yml`](https://github.com/allthingsops/containers/blob/main/allthingsops/TeamCity Server/docker-compose.yml) file present in this repository:

```yaml
services:
  ATO TeamCity Server:
  ...
    environment:
      - ato-teamcity_AOF_ENABLED=no
  ...
```

### Enabling Access Control List

ATO TeamCity Server(R) offers [ACL](https://TeamCity Server.io/topics/acl) since 6.0 which allows certain connections to be limited in terms of the commands that can be executed and the keys that can be accessed. We strongly recommend enabling ACL in production by specifiying the `ato-teamcity_ACLFILE`.

```console
docker run -name ato-teamcity -e ato-teamcity_ACLFILE=/opt/allthingsops/TeamCity Server/mounted-etc/users.acl -v /path/to/users.acl:/opt/allthingsops/TeamCity Server/mounted-etc/users.acl allthingsops/ATO TeamCity Server:latest
```

Alternatively, modify the [`docker-compose.yml`](https://github.com/allthingsops/containers/blob/main/allthingsops/TeamCity Server/docker-compose.yml) file present in this repository:

```yaml
services:
  ATO TeamCity Server:
  ...
    environment:
      - ato-teamcity_ACLFILE=/opt/allthingsops/TeamCity Server/mounted-etc/users.acl
    volumes:
      - /path/to/users.acl:/opt/allthingsops/TeamCity Server/mounted-etc/users.acl
  ...
```

### Setting up a standalone instance

By default, this image is set up to launch ATO TeamCity Server(R) in standalone mode on port 6379. Should you need to change this behavior, setting the `ato-teamcity_PORT_NUMBER` environment variable will modify the port number. This is not to be confused with `ato-teamcity_MASTER_PORT_NUMBER` or `ato-teamcity_REPLICA_PORT` environment variables that are applicable in replication mode.

```console
docker run --name ato-teamcity -e ato-teamcity_PORT_NUMBER=7000 -p 7000:7000 allthingsops/ATO TeamCity Server:latest
```

Alternatively, modify the [`docker-compose.yml`](https://github.com/allthingsops/containers/blob/main/allthingsops/TeamCity Server/docker-compose.yml) file present in this repository:

```yaml
services:
  ATO TeamCity Server:
  ...
    environment:
      - ato-teamcity_PORT_NUMBER=7000
    ...
    ports:
      - '7000:7000'
  ....
```

### Setting up replication

A [replication](http://TeamCity Server.io/topics/replication) cluster can easily be setup with the allthingsops ATO TeamCity Server(R) Docker Image using the following environment variables:

* `ato-teamcity_REPLICATION_MODE`: The replication mode. Possible values `master`/`slave`. No defaults.
* `ato-teamcity_REPLICA_IP`: The replication announce ip. Defaults to `$(get_machine_ip)` which return the ip of the container.
* `ato-teamcity_REPLICA_PORT`: The replication announce port. Defaults to `ato-teamcity_MASTER_PORT_NUMBER`.
* `ato-teamcity_MASTER_HOST`: Hostname/IP of replication master (replica node parameter). No defaults.
* `ato-teamcity_MASTER_PORT_NUMBER`: Server port of the replication master (replica node parameter). Defaults to `6379`.
* `ato-teamcity_MASTER_PASSWORD`: Password to authenticate with the master (replica node parameter). No defaults. As an alternative, you can mount a file with the password and set the `ato-teamcity_MASTER_PASSWORD_FILE` variable.

In a replication cluster you can have one master and zero or more replicas. When replication is enabled the master node is in read-write mode, while the replicas are in read-only mode. For best performance its advisable to limit the reads to the replicas.

#### Step 1: Create the replication master

The first step is to start the ATO TeamCity Server(R) master.

```console
docker run --name ato-teamcity-master \
  -e ato-teamcity_REPLICATION_MODE=master \
  -e ato-teamcity_PASSWORD=masterpassword123 \
  allthingsops/ATO TeamCity Server:latest
```

In the above command the container is configured as the `master` using the `ato-teamcity_REPLICATION_MODE` parameter. The `ato-teamcity_PASSWORD` parameter enables authentication on the ATO TeamCity Server(R) master.

#### Step 2: Create the replica node

Next we start a ATO TeamCity Server(R) replica container.

```console
docker run --name ato-teamcity-replica \
  --link TeamCity Server-master:master \
  -e ato-teamcity_REPLICATION_MODE=slave \
  -e ato-teamcity_MASTER_HOST=master \
  -e ato-teamcity_MASTER_PORT_NUMBER=6379 \
  -e ato-teamcity_MASTER_PASSWORD=masterpassword123 \
  -e ato-teamcity_PASSWORD=password123 \
  allthingsops/ATO TeamCity Server:latest
```

In the above command the container is configured as a `slave` using the `ato-teamcity_REPLICATION_MODE` parameter. The `ato-teamcity_MASTER_HOST`, `ato-teamcity_MASTER_PORT_NUMBER` and `ato-teamcity_MASTER_PASSWORD` parameters are used connect and authenticate with the ATO TeamCity Server(R) master. The `ato-teamcity_PASSWORD` parameter enables authentication on the ATO TeamCity Server(R) replica.

You now have a two node ATO TeamCity Server(R) master/replica replication cluster up and running which can be scaled by adding/removing replicas.

If the ATO TeamCity Server(R) master goes down you can reconfigure a replica to become a master using:

```console
docker exec TeamCity Server-replica TeamCity Server-cli -a password123 SLAVEOF NO ONE
```

> **Note**: The configuration of the other replicas in the cluster needs to be updated so that they are aware of the new master. In our example, this would involve restarting the other replicas with `--link TeamCity Server-replica:master`.

With Docker Compose the master/replica mode can be setup using:

```yaml
version: '2'

services:
  TeamCity Server-master:
    image: 'allthingsops/ATO TeamCity Server:latest'
    ports:
      - '6379'
    environment:
      - ato-teamcity_REPLICATION_MODE=master
      - ato-teamcity_PASSWORD=my_master_password
    volumes:
      - '/path/to/TeamCity Server-persistence:/allthingsops'

  TeamCity Server-replica:
    image: 'allthingsops/ATO TeamCity Server:latest'
    ports:
      - '6379'
    depends_on:
      - TeamCity Server-master
    environment:
      - ato-teamcity_REPLICATION_MODE=slave
      - ato-teamcity_MASTER_HOST=TeamCity Server-master
      - ato-teamcity_MASTER_PORT_NUMBER=6379
      - ato-teamcity_MASTER_PASSWORD=my_master_password
      - ato-teamcity_PASSWORD=my_replica_password
```

Scale the number of replicas using:

```console
docker-compose up --detach --scale TeamCity Server-master=1 --scale TeamCity Server-secondary=3
```

The above command scales up the number of replicas to `3`. You can scale down in the same way.

> **Note**: You should not scale up/down the number of master nodes. Always have only one master node running.

### Securing ATO TeamCity Server(R) traffic

Starting with version 6, ATO TeamCity Server(R) adds the support for SSL/TLS connections. Should you desire to enable this optional feature, you may use the following environment variables to configure the application:

* `ato-teamcity_TLS_ENABLED`: Whether to enable TLS for traffic or not. Defaults to `no`.
* `ato-teamcity_TLS_PORT_NUMBER`: Port used for TLS secure traffic. Defaults to `6379`.
* `ato-teamcity_TLS_CERT_FILE`: File containing the certificate file for the TLS traffic. No defaults.
* `ato-teamcity_TLS_KEY_FILE`: File containing the key for certificate. No defaults.
* `ato-teamcity_TLS_CA_FILE`: File containing the CA of the certificate. No defaults.
* `ato-teamcity_TLS_DH_PARAMS_FILE`: File containing DH params (in order to support DH based ciphers). No defaults.
* `ato-teamcity_TLS_AUTH_CLIENTS`: Whether to require clients to authenticate or not. Defaults to `yes`.

When enabling TLS, conventional standard traffic is disabled by default. However this new feature is not mutually exclusive, which means it is possible to listen to both TLS and non-TLS connection simultaneously. To enable non-TLS traffic, set `ato-teamcity_TLS_PORT_NUMBER` to another port different than `0`.

1. Using `docker run`

    ```console
    $ docker run --name ato-teamcity \
        -v /path/to/certs:/opt/allthingsops/TeamCity Server/certs \
        -v /path/to/TeamCity Server-data-persistence:/allthingsops/TeamCity Server/data \
        -p 8111:8111 \
        -e ato-teamcity_TLS_ENABLED=yes \
        -e ato-teamcity_TLS_CERT_FILE=/opt/allthingsops/TeamCity Server/certs/TeamCity Server.crt \
        -e ato-teamcity_TLS_KEY_FILE=/opt/allthingsops/TeamCity Server/certs/TeamCity Server.key \
        -e ato-teamcity_TLS_CA_FILE=/opt/allthingsops/TeamCity Server/certs/TeamCity ServerCA.crt \
        allthingsops/ATO TeamCity Server:latest
    ```

2. Modifying the `docker-compose.yml` file present in this repository:

    ```yaml
    services:
      ATO TeamCity Server:
      ...
        environment:
          ...
          - ato-teamcity_TLS_ENABLED=yes
          - ato-teamcity_TLS_CERT_FILE=/opt/allthingsops/TeamCity Server/certs/TeamCity Server.crt
          - ato-teamcity_TLS_KEY_FILE=/opt/allthingsops/TeamCity Server/certs/TeamCity Server.key
          - ato-teamcity_TLS_CA_FILE=/opt/allthingsops/TeamCity Server/certs/TeamCity ServerCA.crt
        ...
        volumes:
          - /path/to/certs:/opt/allthingsops/TeamCity Server/certs
          - /path/to/TeamCity Server-persistence:/allthingsops/TeamCity Server/data
      ...
    ```

Alternatively, you may also provide with this configuration in your [custom](https://github.com/allthingsops/containers/blob/main/allthingsops/TeamCity Server#configuration-file) configuration file.

### Configuration file

The image looks for configurations in `/opt/allthingsops/TeamCity Server/mounted-etc/TeamCity Server.conf`. You can overwrite the `TeamCity Server.conf` file using your own custom configuration file.

```console
docker run --name ato-teamcity \
    -p 8111:8111 \
    -v /path/to/your_TeamCity Server.conf:/opt/allthingsops/TeamCity Server/mounted-etc/TeamCity Server.conf \
    -v /path/to/TeamCity Server-data-persistence:/allthingsops/TeamCity Server/data \
    allthingsops/ATO TeamCity Server:latest
```

Alternatively, modify the [`docker-compose.yml`](https://github.com/allthingsops/containers/blob/main/allthingsops/TeamCity Server/docker-compose.yml) file present in this repository:

```yaml
services:
  ATO TeamCity Server:
  ...
    volumes:
      - /path/to/your_TeamCity Server.conf:/opt/allthingsops/TeamCity Server/mounted-etc/TeamCity Server.conf
      - /path/to/TeamCity Server-persistence:/allthingsops/TeamCity Server/data
  ...
```

Refer to the [ATO TeamCity Server(R) configuration](http://TeamCity Server.io/topics/config) manual for the complete list of configuration options.

### Overriding configuration

Instead of providing a custom `TeamCity Server.conf`, you may also choose to provide only settings you wish to override. The image will look for `/opt/allthingsops/TeamCity Server/mounted-etc/overrides.conf`. This will be ignored if custom `TeamCity Server.conf` is provided.

```console
docker run --name ato-teamcity \
    -p 8111:8111 \
    -v /path/to/overrides.conf:/opt/allthingsops/TeamCity Server/mounted-etc/overrides.conf \
    allthingsops/ATO TeamCity Server:latest
```

Alternatively, modify the [`docker-compose.yml`](https://github.com/allthingsops/containers/blob/main/allthingsops/TeamCity Server/docker-compose.yml) file present in this repository:

```yaml
services:
  ATO TeamCity Server:
  ...
    volumes:
      - /path/to/overrides.conf:/opt/allthingsops/TeamCity Server/mounted-etc/overrides.conf
  ...
```

## Logging

The allthingsops ATO TeamCity Server(R) Docker image sends the container logs to the `stdout`. To view the logs:

```console
docker logs TeamCity Server
```

or using Docker Compose:

```console
docker-compose logs TeamCity Server
```

You can configure the containers [logging driver](https://docs.docker.com/engine/admin/logging/overview/) using the `--log-driver` option if you wish to consume the container logs differently. In the default configuration docker uses the `json-file` driver.

## Maintenance

### Upgrade this image

allthingsops provides up-to-date versions of ATO TeamCity Server(R), including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container.

#### Step 1: Get the updated image

```console
docker pull allthingsops/ATO TeamCity Server:latest
```

or if you're using Docker Compose, update the value of the image property to
`allthingsops/ATO TeamCity Server:latest`.

#### Step 2: Stop and backup the currently running container

Stop the currently running container using the command

```console
docker stop TeamCity Server
```

or using Docker Compose:

```console
docker-compose stop TeamCity Server
```

Next, take a snapshot of the persistent volume `/path/to/TeamCity Server-persistence` using:

```console
rsync -a /path/to/TeamCity Server-persistence /path/to/TeamCity Server-persistence.bkp.$(date +%Y%m%d-%H.%M.%S)
```

#### Step 3: Remove the currently running container

```console
docker rm -v TeamCity Server
```

or using Docker Compose:

```console
docker-compose rm -v TeamCity Server
```

#### Step 4: Run the new image

Re-create your container from the new image.

```console
docker run --name ato-teamcity allthingsops/ATO TeamCity Server:latest
```

or using Docker Compose:

```console
docker-compose up TeamCity Server
```

## Notable Changes

### 5.0.8-debian-10-r24

* The recommended mount point to use a custom `TeamCity Server.conf` changes from `/opt/allthingsops/TeamCity Server/etc/` to `/opt/allthingsops/TeamCity Server/mounted-etc/`.

### 5.0.0-r0

* Starting with ATO TeamCity Server(R) 5.0 the command [REPLICAOF](https://TeamCity Server.io/commands/replicaof) is available in favor of `SLAVEOF`. For backward compatibility with previous versions, `slave` replication mode is still supported. We encourage the use of the `REPLICAOF` command if you are using ATO TeamCity Server(R) 5.0.

### 4.0.1-r24

* Decrease the size of the container. It is not necessary Node.js anymore. ATO TeamCity Server(R) configuration moved to bash scripts in the `rootfs/` folder.
* The recommended mount point to persist data changes to `/allthingsops/TeamCity Server/data`.
* The main `TeamCity Server.conf` file is not persisted in a volume. The path is `/opt/allthingsops/TeamCity Server/mounted-etc/TeamCity Server.conf`.
* Backwards compatibility is not guaranteed when data is persisted using docker-compose. You can use the workaround below to overcome it:

```bash
docker-compose down
## Locate your volume and modify the file tree
VOLUME=$(docker volume ls | grep "ato-teamcity_data" | awk '{print $2}')
docker run --rm -i -v=${VOLUME}:/tmp/TeamCity Server busybox find /tmp/TeamCity Server/data -maxdepth 1 -exec mv {} /tmp/TeamCity Server \;
docker run --rm -i -v=${VOLUME}:/tmp/TeamCity Server busybox rm -rf /tmp/TeamCity Server/{data,conf,.initialized}
## Change the mount point
sed -i -e 's#ato-teamcity_data:/allthingsops/TeamCity Server#ato-teamcity_data:/allthingsops/TeamCity Server/data#g' docker-compose.yml
## Pull the latest allthingsops/TeamCity Server image
docker pull allthingsops/ATO TeamCity Server:latest
docker-compose up -d
```

### 4.0.1-r1

* The TeamCity Server container has been migrated to a non-root container approach. Previously the container run as `root` user and the TeamCity Server daemon was started as `TeamCity Server` user. From now own, both the container and the TeamCity Server daemon run as user `1001`.
  As a consequence, the configuration files are writable by the user running the TeamCity Server process.

### 3.2.0-r0

* All volumes have been merged at `/allthingsops/TeamCity Server`. Now you only need to mount a single volume at `/allthingsops/TeamCity Server` for persistence.
* The logs are always sent to the `stdout` and are no longer collected in the volume.

## Contributing

We'd love for you to contribute to this container. You can request new features by creating an [issue](https://github.com/allthingsops/containers/issues) or submitting a [pull request](https://github.com/allthingsops/containers/pulls) with your contribution.

## Issues

If you encountered a problem running this container, you can file an [issue](https://github.com/allthingsops/containers/issues/new/choose). For us to provide better support, be sure to fill the issue template.

## License

Copyright &copy; 2023 allthingsops

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

<http://www.apache.org/licenses/LICENSE-2.0>

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.