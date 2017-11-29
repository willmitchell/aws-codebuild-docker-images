# Copyright 2017-2017 Amazon.com, Inc. or its affiliates. All Rights Reserved.
#
# Licensed under the Amazon Software License (the "License"). You may not use this file except in compliance with the License.
# A copy of the License is located at
#
#    http://aws.amazon.com/asl/
#
# or in the "license" file accompanying this file.
# This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, express or implied.
# See the License for the specific language governing permissions and limitations under the License.
#

FROM ubuntu:14.04.5

# Building git from source code:
#   Ubuntu's default git package is built with broken gnutls. Rebuild git with openssl.
##########################################################################
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
       wget python python2.7-dev fakeroot ca-certificates tar gzip zip \
       autoconf automake bzip2 file g++ gcc imagemagick libbz2-dev libc6-dev libcurl4-openssl-dev \
       libdb-dev libevent-dev libffi-dev libgeoip-dev libglib2.0-dev libjpeg-dev libkrb5-dev \
       liblzma-dev libmagickcore-dev libmagickwand-dev libmysqlclient-dev libncurses-dev libpng-dev \
       libpq-dev libreadline-dev libsqlite3-dev libssl-dev libtool libwebp-dev libxml2-dev libxslt-dev \
       libyaml-dev make patch xz-utils zlib1g-dev unzip curl \
    && apt-get -qy build-dep git \
    && apt-get -qy install libcurl4-openssl-dev git-man liberror-perl \
    && mkdir -p /usr/src/git-openssl \
    && cd /usr/src/git-openssl \
    && apt-get source git \
    && cd $(find -mindepth 1 -maxdepth 1 -type d -name "git-*") \
    && sed -i -- 's/libcurl4-gnutls-dev/libcurl4-openssl-dev/' ./debian/control \
    && sed -i -- '/TEST\s*=\s*test/d' ./debian/rules \
    && dpkg-buildpackage -rfakeroot -b \
    && find .. -type f -name "git_*ubuntu*.deb" -exec dpkg -i \{\} \; \
    && rm -rf /usr/src/git-openssl \
# Install dependencies by all python images equivalent to buildpack-deps:jessie
# on the public repos.
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

RUN wget "https://bootstrap.pypa.io/get-pip.py" -O /tmp/get-pip.py \
    && python /tmp/get-pip.py \
    && pip install awscli==1.11.25 \
    && rm -fr /var/lib/apt/lists/* /tmp/* /var/tmp/*



# Will Mitchell added this stuff in order to have both Docker and JDK8
ENV JAVA_VERSION=8 \
    JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64" \
    JDK_HOME="/usr/lib/jvm/java-8-openjdk-amd64" \
    JRE_HOME="/usr/lib/jvm/java-8-openjdk-amd64/jre" \
    ANT_VERSION=1.9.6 \
    MAVEN_VERSION=3.3.3 \
    MAVEN_HOME="/usr/share/maven" \
    MAVEN_CONFIG="/root/.m2" \
    GRADLE_VERSION=2.7

# Install Java
RUN apt-get update \
    && apt-get install -y software-properties-common \
    && add-apt-repository ppa:openjdk-r/ppa \
    && apt-get update \
    && apt-get -y install python-setuptools \
    && apt-get -y install openjdk-$JAVA_VERSION-jdk \
    && apt-get clean \
    # Ensure Java cacerts symlink points to valid location
    && update-ca-certificates -f \
    && mkdir -p /usr/src/ant \
    && wget "http://archive.apache.org/dist/ant/binaries/apache-ant-$ANT_VERSION-bin.tar.gz" -O /usr/src/ant/apache-ant-$ANT_VERSION-bin.tar.gz \
    && tar -xzf /usr/src/ant/apache-ant-$ANT_VERSION-bin.tar.gz -C /usr/local \
    && ln -s /usr/local/apache-ant-$ANT_VERSION/bin/ant /usr/bin/ant \
    && rm -rf /usr/src/ant \
    && mkdir -p /usr/share/maven /usr/share/maven/ref $MAVEN_CONFIG \
    && curl -fsSL "https://archive.apache.org/dist/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz" \
        | tar -xzC /usr/share/maven --strip-components=1 \
    && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn \
    && mkdir -p /usr/src/gradle \
    && wget "https://services.gradle.org/distributions/gradle-$GRADLE_VERSION-bin.zip" -O /usr/src/gradle/gradle-$GRADLE_VERSION-bin.zip \
    && unzip /usr/src/gradle/gradle-$GRADLE_VERSION-bin.zip -d /usr/local \
    && ln -s /usr/local/gradle-$GRADLE_VERSION/bin/gradle /usr/bin/gradle \
    && rm -rf /usr/src/gradle \
    && rm -fr /var/lib/apt/lists/* /tmp/* /var/tmp/*

 # End Will Mitchell mods

ENV DOCKER_BUCKET="get.docker.com" \
    DOCKER_VERSION="1.12.1" \
    DOCKER_SHA256="05ceec7fd937e1416e5dce12b0b6e1c655907d349d52574319a1e875077ccb79" \
    DIND_COMMIT="3b5fac462d21ca164b3778647420016315289034"

COPY dockerd-entrypoint.sh /usr/local/bin/

# From the docker:1.11
RUN set -x \
    && curl -fSL "https://${DOCKER_BUCKET}/builds/Linux/x86_64/docker-${DOCKER_VERSION}.tgz" -o docker.tgz \
    && echo "${DOCKER_SHA256} *docker.tgz" | sha256sum -c - \
    && tar -xzvf docker.tgz \
    && mv docker/* /usr/local/bin/ \
    && rmdir docker \
    && rm docker.tgz \
    && docker -v \
# From the docker dind 1.11
    && apt-get update && apt-get install -y --no-install-recommends \
       e2fsprogs iptables xfsprogs xz-utils \
# set up subuid/subgid so that "--userns-remap=default" works out-of-the-box
    && addgroup dockremap \
    && useradd -g dockremap dockremap \
    && echo 'dockremap:165536:65536' >> /etc/subuid \
    && echo 'dockremap:165536:65536' >> /etc/subgid \
    && wget "https://raw.githubusercontent.com/docker/docker/${DIND_COMMIT}/hack/dind" -O /usr/local/bin/dind \
    && chmod +x /usr/local/bin/dind \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

VOLUME /var/lib/docker

ENTRYPOINT ["dockerd-entrypoint.sh"]
