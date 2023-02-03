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

## Use the builder

```bash
# Start squid and tdp-builder with docker compose
BUILDER_UID=$(id -u) BUILDER_GID=$(id -g) docker compose up -d
# Get an interactive bash inside tdp-builder container
docker exec -i -t -u builder <tdp_builder_container_name> bash

cd /build/hadoop
mvn clean install -Pdist -Dtar -Pnative -DskipTests -Dmaven.javadoc.skip=true -Dhttps.proxyHost=squid -Dhttps.proxyPort=3128 -Dhttp.proxyHost=squid -Dhttp.proxyPort=3128 -Dmaven.repo.local=/build/m2
cd /build/tez
mvn clean install -pl \!tez-ui -Phadoop28 -P\!hadoop27 -DskipTests -Dhttps.proxyHost=squid -Dhttps.proxyPort=3128 -Dhttp.proxyHost=squid -Dhttp.proxyPort=3128 -Dmaven.repo.local=/build/m2
cd /build/hive1
mvn clean install -Phadoop-2 -DskipTests -Dhttps.proxyHost=squid -Dhttps.proxyPort=3128 -Dhttp.proxyHost=squid -Dhttp.proxyPort=3128 -Dmaven.repo.local=/build/m2
cd /build/spark
https_proxy=http://squid:3128 http_proxy=http://squid:3128 ./dev/make-distribution.sh --name tdp --tgz -Phive -Phive-thriftserver -Pyarn -Phadoop-3.1 -Pflume -Dhttps.proxyHost=squid -Dhttps.proxyPort=3128 -Dhttp.proxyHost=squid -Dhttp.proxyPort=3128 -Dmaven.repo.local=/build/m2
mvn install -Phive -Phive-thriftserver -Pyarn -Phadoop-3.1 -Pflume -DskipTests -Dhttps.proxyHost=squid -Dhttps.proxyPort=3128 -Dhttp.proxyHost=squid -Dhttp.proxyPort=3128 -Dmaven.repo.local=/build/m2
cd /build/hive2
mvn clean install -DskipTests -Dhttps.proxyHost=squid -Dhttps.proxyPort=3128 -Dhttp.proxyHost=squid -Dhttp.proxyPort=3128 -Dmaven.repo.local=/build/m2
cd /build/spark3
https_proxy=http://squid:3128 http_proxy=http://squid:3128 ./dev/make-distribution.sh --name tdp --tgz -Phive -Phive-thriftserver -Pyarn -Phadoop-3.1 -Dhttps.proxyHost=squid -Dhttps.proxyPort=3128 -Dhttp.proxyHost=squid -Dhttp.proxyPort=3128 -Dmaven.repo.local=/build/m2
# Doesn't work https://github.com/TOSIT-IO/spark/issues/3
./build/mvn install -Phive -Phive-thriftserver -Pyarn -Phadoop-3.1 -Dscalastyle.skip=true -DskipTests -Dhttps.proxyHost=squid -Dhttps.proxyPort=3128 -Dhttp.proxyHost=squid -Dhttp.proxyPort=3128 -Dmaven.repo.local=/build/m2
cd /build/hive
mvn clean install -Pdist -DskipTests -Dhttps.proxyHost=squid -Dhttps.proxyPort=3128 -Dhttp.proxyHost=squid -Dhttp.proxyPort=3128 -Dmaven.repo.local=/build/m2
cd /build/hbase
mvn clean install assembly:single -DskipTests -Dhadoop.profile=3.0 -Dhttps.proxyHost=squid -Dhttps.proxyPort=3128 -Dhttp.proxyHost=squid -Dhttp.proxyPort=3128 -Dmaven.repo.local=/build/m2
cd /build/ranger
npm config set proxy "http://squid:3128"
npm config set https-proxy "http://squid:3128"
mvn clean install -DskipTests -Drat.numUnapprovedLicenses=1000 -Dhttps.proxyHost=squid -Dhttps.proxyPort=3128 -Dhttp.proxyHost=squid -Dhttp.proxyPort=3128 -Dmaven.repo.local=/build/m2
rm ~/.npmrc
cd /build/phoenix
mvn clean install -DskipTests -Dhbase.profile=2.1 -Dhttps.proxyHost=squid -Dhttps.proxyPort=3128 -Dhttp.proxyHost=squid -Dhttp.proxyPort=3128 -Dmaven.repo.local=/build/m2
cd /build/phoenix-queryserver
mvn clean install -Dpackage.phoenix.client -Dphoenix.version=5.1.3-TDP-0.1.0-SNAPSHOT -Dphoenix.client.artifactid=phoenix-client-hbase-2.1 -pl '!phoenix-queryserver-it' -DskipTests -Dhttps.proxyHost=squid -Dhttps.proxyPort=3128 -Dhttp.proxyHost=squid -Dhttp.proxyPort=3128 -Dmaven.repo.local=/build/m2
cd /build/knox
npm config set proxy "http://squid:3128"
npm config set https-proxy "http://squid:3128"
mvn clean install -Prelease -Ppackage -Drat.numUnapprovedLicenses=1000 -DskipTests -Dmaven.javdoc.skip=true -Dcheckstyle.skip=true -Dfindbugs.skip=true -Dspotbugs.skip=true -Dhttps.proxyHost=squid -Dhttps.proxyPort=3128 -Dhttp.proxyHost=squid -Dhttp.proxyPort=3128 -Dmaven.repo.local=/build/m2
rm ~/.npmrc
cd /build/hbase-spark-connector
mvn clean install -Dspark.version=2.3.5-TDP-0.1.0-SNAPSHOT -Dscala.version=2.11.8 -Dscala.binary.version=2.11 -Dhadoop-three.version=3.1.1-TDP-0.1.0-SNAPSHOT -Dhbase.version=2.1.10-TDP-0.1.0-SNAPSHOT -DskipTests -Dhttps.proxyHost=squid -Dhttps.proxyPort=3128 -Dhttp.proxyHost=squid -Dhttp.proxyPort=3128 -Dmaven.repo.local=/build/m2
```

## How it works

The goal is to list all domain needed to build TDP components. For that we use Squid to allow a whitelist domain, i.e. domain not in this whitelist is denied.
To ensure the builder do not use internet without Squid as HTTP proxy, the tdp-builder is connected to an internal network and only Squid is connected to a network with internet access and connected to the same internal network to communicate with tdp-builder container.

You can check that with curl inside the tdp-builder container.

```bash
# Get an interactive bash inside tdp-builder container
docker exec -i -t <tdp_builder_container_name> bash
# This curl will not work
curl http://google.com
# This curl will not work, use squid as HTTP proxy but domain is denied
curl --proxy squid:3128 http://google.com
# This curl works, use squid as HTTP proxy and domain is allowed
curl --proxy squid:3128 https://repo.maven.apache.org
```

## Clean repository directories

Builds are done directly inside repository directories. If you want to clean a repository you can use git submodule commands.

```bash
# Clean hadoop repository
git submodule deinit -f hadoop
git submodule update --init hadoop
# Clean all repositories
git submodule deinit -f --all
git submodule update --init
```
