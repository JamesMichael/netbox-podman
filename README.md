# Podman scripts for netbox

This repo shows my work on converting the netbox-docker's `docker-compose.yml`
into a podman pod, done as a learning exercise for docker, podman, kubernetes,
and netbox.

It has been tested and works against the latest netbox docker image as of
17th October 2020, though I don't plan to keep it up-to-date.

The netbox-docker project launches five containers:

postgres
: Used for data persistance.

redis (x2)
: One instance for caching, one instance for persistance.

netbox (x2)
: One worker instance, one http instance.

nginx
: Reverse proxy to netbox and serves static content.

## Usage

### Setup

First, clone the netbox-docker repository:

```bash
git clone https://github.com/netbox-community/netbox-docker.git
cd netbox-docker
git checkout tags/0.25.0
```

Then, inside the netbox-docker repository, check out this repository:

```bash
git clone https://github.com/jamesmichael/netbox-podman.git
```

Then, configure the pod by running:

```bash
podman/init
```

This script downloads, configures, and starts each container. It will take a
while to download the images and perform the netbox first-run install process.

You can keep an eye on the status by running `podman logs $CONTAINER_NAME`,
where a list of containers can be found by running `podman ps`.

At the end of the process, netbox will be running inside a pod on port 1234.

### Stopping

To stop the pod, run:

```bash
podman/down
```

### Re-starting

If the pod has been stopped then it can be restarted with:

```bash
podman/up
```

### Cleaning up

To destroy the pod and remove any data, run:

```bash
podman/destroy
```

You can then run `podman system prune` to clear any unused images, this will
affect data other pods.

## Configuration

The following environment variables can be set to customise the behaviour:

`NETBOX_NAME`
: The name of the pod created, defaults to _netbox_.

`NETBOX_PORT`
: The host port used to access netbox, defaults to _1234_.

Additional configuration can be made by editing the `.env` files in the
netbox-docker repository.

## Learnings

To share data between containers, create and mount a volume. Unlike bind mounts,
the volume will be populated with the contents of the container, rather than
overwrite the mount location. Mounting the same named volume on another
container allows the other container to access the same filesystem.

Each container in podman shares the same network stack. This means that processes
in two separate containers within the same pod cannot bind to the same port.
Additionaly, each container can communicate with the other containers in the
pod using localhost. This differs from docker-compose.

Key commands for inspecting pod state:

`podman pod ps`
: List all pods

`podman ps [-a]`
: List running (or all) containers

`podman exec -it CONTAINER COMMAND ARGS`
: Run a command in the named container.
