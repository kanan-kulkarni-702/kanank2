# Docker environment setup before using ScalaarDL

This document shows how to set up a local Docker environment to be used with ScalarDL
 [DockerCompose](https://docs.docker.com/compose/).

## Prerequisites

- [Docker Engine](https://docs.docker.com/engine/) and [Docker Compose](https://docs.docker.com/compose/).

    Follow the instructions on the Docker website according to your platform.

## Docker login

`docker login` is required to start the ScalarDL Docker image. Because the
[`scalar-ledger`](https://github.com/orgs/scalar-labs/packages/container/package/scalar-ledger) repository
on GitHub Container Registry is currently private, your GitHub account needs to be set with permissions to access the container images.
Ask a person in charge to get your account ready. Note also that you need to use a personal access token (PAT) as a password to login `ghcr.io`. Please read [the official document](https://docs.github.com/en/packages/guides/migrating-to-github-container-registry-for-docker-images#authenticating-with-the-container-registry) for more detail.

```
# read:packages scope needs to be selected in a personal access token to login
$ export CR_PAT=YOUR_PERSONAL_ACCESS_TOKEN
$ echo $CR_PAT | docker login ghcr.io -u USERNAME --password-stdin
```
