# syntax=docker/dockerfile:1
FROM debian:10-slim

ENV LANG C.UTF-8
ENV PIP_VERSION 22.0.4

RUN <<EOT
    set -e
    apt update
    buildDeps="build-essential ca-certificates curl default-jdk dirmngr dpkg-dev gcc git gnupg libatomic1 libbluetooth-dev libbz2-dev libc6-dev libdb-dev libexpat1-dev libffi-dev libgdbm-dev libjpeg62-turbo-dev libldap2-dev liblzma-dev libmagic1 libncursesw5-dev libopenjp2-7-dev libpcre3-dev libpq-dev libreadline-dev libsasl2-dev libsqlite3-dev libssl-dev libtiff5-dev libxml2-dev libxslt1-dev make netbase openssh-client python3-dev tk-dev tzdata uuid-dev wget xvfb xz-utils zlib1g-dev"
    apt install -y --no-install-recommends $buildDeps
    curl -sL https://deb.nodesource.com/setup_14.x | bash -
    apt update
    apt install -y python3.7-dev python3.7-venv python3-pip
    apt install -y nodejs
    npm install -g yarn
    pip3 install -U "pip==${PIP_VERSION}"
    ln -s /usr/bin/pip3 /usr/bin/pip
    mkdir -p /home/ci
    useradd --system -m -d /home/ci -U -u 500 ci
EOT

LABEL maintainer="kitconcept GmbH <server@kitconcept.com>" \
      org.label-schema.name="ghcr.io/kitconcept/plone-test-py37-node14" \
      org.label-schema.description="Image with Python 3.7 and Node 14" \
      org.label-schema.vendor="kitconcept GmbH"

USER ci

CMD ["bash"]
