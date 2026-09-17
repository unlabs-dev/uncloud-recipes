# PostgreSQL High-Availability Using Stolon

[stolon](https://github.com/sorintlab/stolon) is a cloud native PostgreSQL manager for PostgreSQL high availability.
To learn more about stolon itself, you can read [this post](https://sgotti.dev/post/stolon-introduction/).

This recipe runs an HA Postgres cluster on 3 servers:
 - server1
 - server2
 - server3

You should replace these with your own servers in [compose.yml](compose.yml) before deploying.

## Setup

 - Copy [.env.example](.env.example) to `.env` and edit the values inside.
 - Replace `server1`, `server2` and `server3` with your server names in [compose.yml](compose.yml)
 - Deploy with `uc deploy`
 - Create the initial cluster data by running `uc exec pg-keeper1 stolonctl --cluster-name=stolon-cluster --store-backend=etcdv3 --store-endpoints http://etcd-00:2379 init`

If you would like to separate etcd from the Postgres deployment, move the etcd cluster to another `compose.yml` file and deploy separately, or use an existing etcd cluster.

## Usage

You can access the cluster within other containers via `pg-proxy.internal` (or ideally, `nearest.pg-proxy.internal` to get the closest instance).

Your Postgres URL will look something like this: `postgresql://<USER>:<PASSWORD>@nearest.pg-proxy.internal:5432/<DATABASE>`

If you would like to access the database from hosts where the proxy is running, uncomment the `x-ports` section in [compose.yml](compose.yml) and redeploy.

## Data Persistence

Every PG instance (keeper) has a corresponding volume. The cluster can tolerate losing 2/3 instances without data loss, as replicas are always streaming changes from the
primary.

The etcd cluster that maintains stolon cluster state similarly store its data in corresponding volumes. If all etcd instances are lost, you will need to recreate them
and then recreate the cluster data again according to the steps in [Setup](#setup).

## Customizing Postgres and Stolon

Both Postgres and stolon are built using a [Dockerfile](Dockerfile). You can change the Postgres version inside it as well as change the commit hash stolon is being built
from.
