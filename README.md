# TDP build proxy whitelist

This repo is used to build TDP components with a whitelist domain proxy.

## Get git submodules

This repository work with git submodules to build TDP components.

```bash
git submodule update --init
```

## Build the builder

Build the docker image used to build TDP components.

```bash
docker build ./TDP/build-env -t tdp-builder
```
