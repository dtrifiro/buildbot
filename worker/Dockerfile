# buildbot/buildbot-worker

# please follow docker best practices
# https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/

# Provides a base debian testing image with latest buildbot worker installed
# the worker image is not optimized for size

FROM        debian:testing
MAINTAINER  Buildbot maintainers


# Last build date - this can be updated whenever there are security updates so
# that everything is rebuilt
ENV         security_updates_as_of 2018-06-15

# This will make apt-get install without question
ARG         DEBIAN_FRONTEND=noninteractive

# Install security updates and required packages
RUN apt-get update && \
    apt-get -y upgrade && \
    apt-get -y install -q \
    build-essential \
    git \
    subversion \
    python-dev \
    libffi-dev \
    libssl-dev \
    python-setuptools \
    curl \
    clang \
    cmake  \
    python-pip && \
    rm -rf /var/lib/apt/lists/*
# Test runs produce a great quantity of dead grandchild processes.  In a
# non-docker environment, these are automatically reaped by init (process 1),
# so we need to simulate that here.  See https://github.com/Yelp/dumb-init
RUN curl https://github.com/Yelp/dumb-init/releases/download/v1.2.1/dumb-init_1.2.1_amd64.deb -Lo /tmp/init.deb && dpkg -i /tmp/init.deb

# Install required python packages, and twisted
RUN pip --no-cache-dir install 'twisted[tls]' && \
    pip install virtualenv && \
    mkdir /buildbot &&\
    useradd -ms /bin/bash buildbot

COPY . /usr/src/buildbot-worker
COPY docker/buildbot.tac /buildbot/buildbot.tac

RUN pip install /usr/src/buildbot-worker && \
    chown -R buildbot /buildbot

USER buildbot
WORKDIR /buildbot

CMD ["/usr/bin/dumb-init", "twistd", "--pidfile=", "-ny", "buildbot.tac"]
