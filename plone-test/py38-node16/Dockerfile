# syntax=docker/dockerfile:1
FROM debian:12-slim

ENV LANG C.UTF-8
ENV NODE_MAJOR 16
ENV PYTHON_VERSION 3.8.10
ENV PYTHON_GET_PIP_URL https://github.com/pypa/get-pip/raw/9af82b715db434abb94a0a6f3569f43e72157346/public/get-pip.py
ENV PIP_VERSION 23.0.1
ENV PYTHON_SETUPTOOLS_VERSION 57.5.0

RUN <<EOT
    set -e
    apt update
    buildDeps="build-essential ca-certificates curl default-jdk dirmngr dpkg-dev gcc git gnupg libatomic1 libbluetooth-dev libbz2-dev libc6-dev libdb-dev libexpat1-dev libffi-dev libgdbm-dev libjpeg62-turbo-dev libldap2-dev liblzma-dev libmagic1 libncursesw5-dev libopenjp2-7-dev libpcre3-dev libpq-dev libreadline-dev libsasl2-dev libsqlite3-dev libssl-dev libtiff5-dev libxml2-dev libxslt1-dev lsof make netbase openssh-client procps python3-dev tk-dev tzdata uuid-dev wget xvfb xz-utils zlib1g-dev"
    apt install -y --no-install-recommends $buildDeps
    cypressDeps="libgtk2.0-0 libgtk-3-0 libgbm-dev libnotify-dev libgconf-2-4 libnss3 libxss1 libasound2 libxtst6 xauth xvfb"
    apt install -y --no-install-recommends $cypressDeps
EOT

# Install Node
RUN <<EOT
    set -e
    printf "Package: *\nPin: origin deb.nodesource.com\nPin-Priority: 1001" > /etc/apt/preferences.d/preferences
    mkdir -p /etc/apt/keyrings
    curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
    echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" > /etc/apt/sources.list.d/nodesource.list
    apt update
    apt install -y nodejs
    npm install -g yarn
EOT

# Install Python
RUN <<EOT
    set -e
    curl -o python.tar.xz -L "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz"
    curl -o get-pip.py -L "$PYTHON_GET_PIP_URL"
    mkdir -p /usr/src/python
    tar --extract --directory /usr/src/python --strip-components=1 --file python.tar.xz
    cd /usr/src/python
    export gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)"
    export nproc="$(nproc)"
	export EXTRA_CFLAGS="$(dpkg-buildflags --get CFLAGS)"
	export LDFLAGS="$(dpkg-buildflags --get LDFLAGS)"
	export LDFLAGS="${LDFLAGS:--Wl},--strip-all"
    ./configure  --build="$gnuArch" --enable-loadable-sqlite-extensions --enable-optimizations --enable-option-checking=fatal --enable-shared --with-system-expat --without-ensurepip
    make -j "$nproc" "EXTRA_CFLAGS=${EXTRA_CFLAGS:-}" "LDFLAGS=${LDFLAGS:-}" "PROFILE_TASK=${PROFILE_TASK:-}"
    make install
    cd /
    rm -rf /usr/src/python
	find /usr/local -depth \
		\( \
			\( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
			-o \( -type f -a \( -name '*.pyc' -o -name '*.pyo' -o -name 'libpython*.a' \) \) \
			-o \( -type f -a -name 'wininst-*.exe' \) \
		\) -exec rm -rf '{}' +
    ldconfig
    rm -Rf /usr/bin/python /usr/bin/python3-config
    ln -s /usr/local/bin/python3.8-config /usr/bin/python3-config
    ln -s /usr/local/bin/python3.8 /usr/bin/python
    python get-pip.py --disable-pip-version-check --no-cache-dir --no-compile "pip==$PIP_VERSION" "setuptools==$PYTHON_SETUPTOOLS_VERSION"
    ln -s /usr/bin/pip3 /usr/bin/pip
    rm python.tar.xz
    rm -Rf get-pip.py
EOT

RUN <<EOT
    mkdir -p /home/ci
    useradd --system -m -d /home/ci -U -u 500 ci
    chown -R 500 /home/ci
EOT


LABEL maintainer="kitconcept GmbH <server@kitconcept.com>" \
      org.label-schema.name="ghcr.io/kitconcept/plone-test-py38-node16" \
      org.label-schema.description="Image with Python 3.8 and Node 16" \
      org.label-schema.vendor="kitconcept GmbH"

USER ci

CMD ["bash"]
