Redis + Sentinel with TLS + Auth
================================

Full fledged non-persistent Redis + Sentinel stack (3 instances each) with TLS and authentication

# Overview

In short, Redis is a superfast key store which is best used without persistency. In order to not lose all that in-memory data, we deploy 3 instances of Redis. We then deploy additional 3 instances of Redis Sentinel - high availabilty solution for Redis which monitors instances and performs failover automatically. We need an odd amount of Sentinels due to quorum, which is not a requirement when it comes to Redis itself.

# Stack architecture

## Redis

Each instance is a single replica service due to the need for them to be preconfigured beforehand. Redis instance services are named `redis-instance-N` where `N` is the instance id (e.g. 1, 2, 3). `redis-instance-1` is preconfigured to be the initial master. Instances 2 and 3 have the exact same configuration, but are also configured to replicate `redis-instance-1`. Each instance publishes its own TLS secure port `637N` where `N` is the instance id, which is also the port that is being announced to other instances. Instances are also configured to announce themselves using `$ENVIRONMENT_DNS` hostname.

## Redis Sentinel

Redis Sentinel instance services are named `redis-sentinel-N` where `N` is the instance id (e.g. 1, 2, 3). Instances discover each other via the initial Redis master instance `redis-instance-1`. Each instance publishes its own TLS secure port `2637N` where `N` is the instance id, which is also the port that is being announced to other instances. Instances are also configured to announce themselves using `$ENVIRONMENT_DNS` hostname.

## Configuration

Stack deployment can be configured via a `.env` file which comes with this repository. The following variables are used to the stack:

- `ENVIRONMENT_DNS`: DNS name which will lead to one of your Swarm nodes. This can be achieved by deploying a front-facing load balancer or a `keepalived` global service which will juggle a VIP between nodes. TLS certificate must secure this DNS name.
- `REDIS_IMAGE`: image used by both Redis and Redis Sentinel instances. Later official Redis images come with Sentinel built-in.
- `REDIS_PASSWORD`: authentication password for both Redis and Redis Sentinel. Keep in mind that username is NOT configured to be used.
