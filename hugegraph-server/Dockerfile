#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

# Dockerfile for HugeGraph Server
# 1st stage: build source code
FROM maven:3.9.0-eclipse-temurin-11 AS build

COPY . /pkg
WORKDIR /pkg
RUN mvn package -e -B -ntp -DskipTests -Dmaven.javadoc.skip=true && pwd && ls -l

# 2nd stage: runtime env
FROM openjdk:11-slim
# TODO: get the version from the pom.xml
ENV version=1.5.0
COPY --from=build /pkg/hugegraph-server/apache-hugegraph-incubating-$version/ /hugegraph-server
LABEL maintainer="HugeGraph Docker Maintainers <dev@hugegraph.apache.org>"

# TODO: use g1gc or zgc as default
ENV JAVA_OPTS="-XX:+UnlockExperimentalVMOptions -XX:+UseContainerSupport -XX:MaxRAMPercentage=50 -XshowSettings:vm" \
    HUGEGRAPH_HOME="hugegraph"

#COPY . /hugegraph/hugegraph-server
WORKDIR /hugegraph-server/

# 1. Install environment
RUN set -x \
    && apt-get -q update \
    && apt-get -q install -y --no-install-recommends --no-install-suggests \
       dumb-init \
       procps \
       curl \
       lsof \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# 2. Init HugeGraph Sever
RUN set -e \
    && pwd && cd /hugegraph-server/ \
    && sed -i "s/^restserver.url.*$/restserver.url=http:\/\/0.0.0.0:8080/g" ./conf/rest-server.properties

# 3. Init docker script
COPY hugegraph-server/hugegraph-dist/docker/scripts/remote-connect.groovy ./scripts
COPY hugegraph-server/hugegraph-dist/docker/scripts/detect-storage.groovy ./scripts
COPY hugegraph-server/hugegraph-dist/docker/docker-entrypoint.sh .
RUN chmod 755 ./docker-entrypoint.sh 

EXPOSE 8080
VOLUME /hugegraph-server

ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["./docker-entrypoint.sh"]