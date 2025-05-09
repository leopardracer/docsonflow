---
title: Setting Up a Flow Node
sidebar_label: Node Setup
description: How to run a Collection, Consensus, Verification and Execution node
sidebar_position: 12
---

This guide is for running a Collection, Consensus, Verification and Execution node.
If you are planning to run an Access node then refer to [access node setup](../access-nodes/access-node-setup.md).

First you'll need to provision a machine or virtual machine to run your node software. Please see follow the [node-provisioning](./node-provisioning.md) guide for it.

## Pull the Flow Images

The `flow-go` binaries are distributed as container images, and need to be pulled down to your host with your image management tool of choice.

Replace `$ROLE` with the node type you are planning to run. Valid options are:

- collection
- consensus
- execution
- verification
- access

```shell
  # Docker
  docker pull gcr.io/flow-container-registry/${ROLE}:alpha-v0.0.1

  # Containerd
  ctr images pull gcr.io/flow-container-registry/${ROLE}:alpha-v0.0.1",
```

## Prepare Your Node to Start

Your nodes will need to boot at startup, and restart if they crash.

If you are running `systemd` you can use the service files provided by `flow-go`.
Find them in the [Flow Go](https://github.com/onflow/flow-go/tree/master/deploy).

If you are using some other system besides Systemd, you need to ensure that the Flow container is started,
the appropriate key directories are mounted into the container, and that the container will
automatically restart following a crash.

The `systemd` files pull runtime settings from `/etc/flow/runtime-config.env` and any `.env`
files under `/etc/flow/conf.d`. Examples of these files are also available in the github repo.
You will need to modify the runtime config file later.

### Systemd

<Callout type="info">
If you are not using Systemd, you can skip this step
</Callout>

1. Ensure that you pulled the latest changes from [flow-go repository](https://github.com/onflow/flow-go) via `git`

```shell
## Clone the repo if you haven't already done so
git clone https://github.com/onflow/flow-go

## Get latest changes
cd flow-go
git pull origin master
```

2. Copy your respective [systemd unit file](https://github.com/onflow/flow-go/tree/master/deploy/systemd-docker) to: `/etc/systemd/system`
3. Create directory `sudo mkdir /etc/flow`
4. Copy the [runtime-conf.env](https://github.com/onflow/flow-go/blob/master/deploy/systemd-docker/runtime-conf.env) file to: `/etc/flow/`
5. Enable your service `sudo systemctl enable flow-$ROLE.service` (replace `$ROLE` with your node role - eg. `collection`)

### Docker Configuration

If you are not using Systemd, sample commands for running each Docker container are below.
Be sure to replace `/path/to/data` and `/path/to/bootstrap` with the appropriate paths you are using.

:::warning

Do not run your node using `docker run` command directly without a mechanism for the node
to automatically restart following a crash.

:::

:::info

The actual Docker image tag can be found [here](./past-upgrades) for appropriate spork.

:::

### System Configuration

Flow nodes create connections to other nodes on the network, which are represented as file descriptors by the OS. Depending on the default
limits for your machine, you may need to increase the soft limit available to the node software.

Make sure the soft limit is at least `8192`.

You can configure the ulimit for the node's docker container. See the [Docker documentation](https://docs.docker.com/engine/networks/commandline/run/#ulimit) for more details.

### Admin Server

Each node can be configured with an admin server, which allows you to control some of the node's configuration, as well as view some of its internal state. You can find
a few of the commands in the Admin Server [README](https://github.com/onflow/flow-go/blob/master/admin/README.md). Two commands to highlight are:

* `list-commands`: which returns a list of all of the available commands for your node
* `set-log-level`: which allows you to change the log level of your node at runtime

You can enable the admin server by passing the `--admin-addr` flag with an interface and port.

> ⚠️ _IMPORANT: The admin server can modify your node's configuration. DO NOT allow access to untrusted clients._

### Access

```shell
docker run --rm \
	-v /path/to/bootstrap:/bootstrap:ro \
	-v /path/to/data:/data:rw \
	--name flow-go \
	--network host \
	--ulimit nofile=8192 \
	gcr.io/flow-container-registry/access:<applicable docker tag> \
	--nodeid=${FLOW_GO_NODE_ID} \
	--bootstrapdir=/bootstrap \
	--datadir=/data/protocol \
	--secretsdir=/data/secrets \
	--execution-data-dir=/data/execution_data \
	--rpc-addr=0.0.0.0:9000 \
	--http-addr=0.0.0.0:8000 \
	--admin-addr=0.0.0.0:9002 \
	--collection-ingress-port=9000 \
	--script-addr=${FLOW_NETWORK_EXECUTION_NODE} \
	--bind 0.0.0.0:3569 \
	--loglevel=error
```

### Collection

```shell
docker run --rm \
	-v /path/to/bootstrap:/bootstrap:ro \
	-v /path/to/data:/data:rw \
	--name flow-go \
	--network host \
	--ulimit nofile=8192 \
	gcr.io/flow-container-registry/collection:<applicable docker tag> \
	--nodeid=${FLOW_GO_NODE_ID} \
	--bootstrapdir=/bootstrap \
	--datadir=/data/protocol \
	--secretsdir=/data/secrets \
	--ingress-addr=0.0.0.0:9000 \
	--admin-addr=0.0.0.0:9002 \
	--bind 0.0.0.0:3569 \
	--access-node-ids=4e17496619df8bb4dcd579c252d9fb026e54995db0dc6825bdcd27bd3288a990,7e3fe64ccc119f578a7795df8b8c512e05409bdc7de4f74259c6f48351fecb26,416c65782048656e74736368656c009530ef3ab4b8bf83b24df54fe5f81853de,416e647265772042757269616e00d219355d62b9adad8ebd3fab223a1cf84c22 \
	--gossipsub-peer-scoring-enabled=false  \
	--gossipsub-peer-gater-enabled=true \
	--loglevel=error
```

### Consensus

```shell
docker run --rm \
	-v /path/to/bootstrap:/bootstrap:ro \
	-v /path/to/data:/data:rw \
	--name flow-go \
	--network host \
	--ulimit nofile=8192 \
	gcr.io/flow-container-registry/consensus:<applicable docker tag> \
	--nodeid=${FLOW_GO_NODE_ID} \
	--bootstrapdir=/bootstrap \
	--datadir=/data/protocol \
	--secretsdir=/data/secrets \
	--admin-addr=0.0.0.0:9002 \
	--bind 0.0.0.0:3569 \
	--access-node-ids=4e17496619df8bb4dcd579c252d9fb026e54995db0dc6825bdcd27bd3288a990,7e3fe64ccc119f578a7795df8b8c512e05409bdc7de4f74259c6f48351fecb26,416c65782048656e74736368656c009530ef3ab4b8bf83b24df54fe5f81853de,416e647265772042757269616e00d219355d62b9adad8ebd3fab223a1cf84c22 \
	--gossipsub-peer-scoring-enabled=false  \
	--gossipsub-peer-gater-enabled=true \
	--loglevel=error
```

### Execution

```shell
docker run --rm \
	-v /path/to/bootstrap:/bootstrap:ro \
	-v /path/to/data:/data:rw \
	--name flow-go \
	--network host \
	--ulimit nofile=500000 \
	gcr.io/flow-container-registry/execution:<applicable docker tag> \
	--nodeid=${FLOW_GO_NODE_ID} \
	--bootstrapdir=/bootstrap \
	--datadir=/data/protocol \
	--secretsdir=/data/secrets \
	--triedir=/data/execution \
	--execution-data-dir=/data/execution_data \
	--rpc-addr=0.0.0.0:9000 \
	--admin-addr=0.0.0.0:9002 \
	--bind 0.0.0.0:3569 \
	--loglevel=error
```

For execution nodes, it is recommend to increase the open files limit in your operating system. To do that, add the following to your `/etc/security/limits.conf` or the equivalent `limits.conf` for your distribution:

```
* hard nofile 500000
* soft nofile 500000
root hard nofile 500000
root soft nofile 500000
```

Restart your machine to apply these changes. To verify that the new limits have been applied, run:

```
ulimit -n
```

### Verification

```shell
docker run --rm \
	-v /path/to/bootstrap:/bootstrap:ro \
	-v /path/to/data:/data:rw \
	--name flow-go \
	--network host \
	--ulimit nofile=8192 \
	gcr.io/flow-container-registry/verification:<applicable docker tag> \
	--nodeid=${FLOW_GO_NODE_ID} \
	--bootstrapdir=/bootstrap \
	--datadir=/data/protocol \
	--secretsdir=/data/secrets \
	--admin-addr=0.0.0.0:9002 \
	--bind 0.0.0.0:3569 \
	--loglevel=error
```

## Start the Node

Now that your node is provisioned and configured, it can be started.

<Callout type="warning">

Before starting your node, ensure it is [registered](./node-bootstrap.md#step-2---stake-your-node) and [authorized](./node-bootstrap.md#confirming-authorization).

</Callout>

Ensure you start your node at the appropriate time.
See [Spork Process](./spork.md) for when to start up a node following a spork.
See [Node Bootstrap](./node-bootstrap.md#timing) for when to start up a newly registered node.

### Systemd

1. Check that your `runtime-conf.env` is at `/etc/flow/runtime-conf.env`
2. Update your environment variables: `source /etc/flow/runtime-conf.env`
3. Start your service: `sudo systemctl start flow`

## Verify your Node is Running


Here are a few handy commands that you can use to check if your Flow node is up and running

### Systemd

- To get Flow logs: `sudo journalctl -u flow-YOUR_ROLE`
- To get the status: `sudo systemctl status flow`

```shell
● flow-verification.service - Flow Access Node running with Docker
Loaded: loaded (/etc/systemd/system/flow-verification.service; enabled; vendor preset: enabled)
Active: active (running) since Wed 2020-05-20 18:18:13 UTC; 1 day 6h ago
Process: 3207 ExecStartPre=/usr/bin/docker pull gcr.io/flow-container-registry/verification:${FLOW_GO_NODE_VERSION} (code=exited, status=0/SUCCESS)
Main PID: 3228 (docker)
Tasks: 10 (limit: 4915)
Memory: 33.0M
CGroup: /system.slice/flow-verification.service
        └─3228 /usr/bin/docker run --rm -v /var/flow/bootstrap:/bootstrap:ro -v /var/flow/data:/data:rw --rm --name flow-go --network host gcr.io/flow-container-registry/verification:candidate8 --nodeid=489f8a4513d5bd8b8b093108fec00327b683db545b37b4ea9153f61b2c0c49dc --bootstrapdir=/bootstrap --datadir=/data/protocol --alpha=1 --bind 0.0.0.0:3569 --loglevel=error
```

### Docker

- To get Flow logs: `sudo docker logs flow-go`
- To get the status: `sudo docker ps`

```shell
$ sudo docker ps
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
1dc5d43385b6 gcr.io/flow-container-registry/verification:candidate8 \"/bin/app --nodeid=4…\" 30 hours ago Up 30 hours flow-go
```

## Monitoring and Metrics

This is intended for operators who would like to see what their Flow nodes are currently doing. Head over to [Monitoring Node Health](./monitoring-nodes.md) to get setup.

### Node Status

The metrics for the node should be able to provide a good overview of the status of the node. If we want to get a quick snapshot of the status of the node, and if it's properly participating in the network, you can check the `consensus_compliance_finalized_height` or `consensus_compliance_sealed_height` metric, and ensure that it is not zero and strictly increasing.

```shell
curl localhost:8080/metrics | grep consensus_compliance_sealed_height

# HELP consensus_compliance_sealed_height the last sealed height
# TYPE consensus_compliance_sealed_height gauge
consensus_compliance_sealed_height 1.132054e+06
```
