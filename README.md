Redis + Sentinel with TLS + Auth
================================

Full fledged non-persistent Redis + Sentinel stack (3 instances each) with TLS and authentication + RedisInsight GUI and TLS+Auth proxy

# Overview

In short, Redis is a superfast key store which is best used without persistency. In order to not lose all that in-memory data, we deploy 3 instances of Redis. We then deploy additional 3 instances of Redis Sentinel - high availabilty solution for Redis which monitors instances and performs failover automatically. We need an odd amount of Sentinels due to quorum, which is not a requirement when it comes to Redis itself.

# Stack architecture

The following describes all of the moving parts for this stack as well as how it is configured out of the box. Understanding this is crucial for proper configuration and deployment. It will also help you understand how to edit the stack file to fit your needs.

## Redis

Each instance is a single replica service due to the need for them to be preconfigured beforehand and accessed individually. Redis instance services are named `redis-instance-N` where `N` is the instance id (e.g. 1, 2, 3). `redis-instance-1` is preconfigured to be the initial master. Instances 2 and 3 have the exact same configuration, but are also configured to replicate `redis-instance-1`. Each instance publishes its own TLS secure port `637N` where `N` is the instance id, which is also the port that is being announced to other instances. Instances are also configured to announce themselves using `$ENVIRONMENT_DNS` hostname.

## Redis Sentinel

Each instance is a single replica service due to the need for them to be preconfigured beforehand and accessed individually. Redis Sentinel instance services are named `redis-sentinel-N` where `N` is the instance id (e.g. 1, 2, 3). Instances discover each other via the initial Redis master instance `redis-instance-1`. Each instance publishes its own TLS secure port `2637N` where `N` is the instance id, which is also the port that is being announced to other instances. Instances are also configured to announce themselves using `$ENVIRONMENT_DNS` hostname.

## RedisInsight

GUI for Redis which supports Redis Sentinel. In a very puzzling way, it does not come with any TLS or authentication capabilities (at least none I could find), meaning all of your data securely stored behind Redis authentication and encrypted using a certificate is accessible in plaintext and very much vulnerable to man-in-the-middle attacks. This is where the proxy comes in.

## RedisInsight Proxy

HAProxy instance in front of RedisInsight which performs TLS termination and user authentication. Credentials for the proxy are `admin` with the password being `REDIS_PASSWORD` environment variable.

## Configuration

Stack deployment can be configured via a `.env` file which comes with this repository. The following variables are used in the stack:

- `ENVIRONMENT_DNS`: DNS name which will lead to one of your Swarm nodes. This can be achieved by deploying a front-facing load balancer or a `keepalived` global service which will juggle a VIP between nodes. TLS certificate must secure this DNS name.
- `REDIS_IMAGE`: image used by both Redis and Redis Sentinel instances. Later official Redis images come with Sentinel built-in.
- `REDIS_PASSWORD`: authentication password for both Redis and Redis Sentinel. Keep in mind that username is NOT configured to be used.
- `REDISINSIGHT_IMAGE`: image for the RedisInsight instance
- `REDISINSIGHT_PROXY_IMAGE`: image for the HAProxy instance

# Deployment

First, make sure you complete all of the prep work:

## Prerequisites

- Docker Swarm up and running
- `.env` file configured for your environment
- Certificate signed for the `$ENVIRONMENT_DNS` DNS name created as secrets in PEM format called `cert.pem` and `key.pem`
- CA certificate / full-chain for the signed certificate created as config in PEM format called `ca.pem`
- all of the relevant docker images accessible from the Swarm

## Procedure

1. Log-in to a Swarm manager and transfer the `docker-compose.yaml` as well as the edited `.env` file to it.

2. Substitute the environment variables and deploy the stack like so:
   ```sh
   # register environment variables
   set -a
   . .env
   set +a

   # substitute variables and deploy the stack
   envsubst < docker-compose.yml | docker stack deploy redis -c -
   ```
