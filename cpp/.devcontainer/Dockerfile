FROM ubuntu:24.04 AS base

ENV DEBIAN_FRONTEND=noninteractive

RUN <<EOT

apt-get update && \
    apt-get install -y \
    build-essential \
    curl \
    git \
    vim \
    ca-certificates \
    net-tools \
    procps \
    sudo

uid_exists=$(getent passwd 1000 | cut -d: -f1)
if [ -n "$uid_exists" ]; then 
    usermod -l dev "$uid_exists"
    usermod -d /home/dev -m dev
    groupmod -n dev "$uid_exists"
else
    groupadd -g 1000 dev
    useradd -m -u 1000 -g 1000 dev
fi

echo 'dev:dev' | chpasswd

echo "dev ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/dev
chmod 0440 /etc/sudoers.d/dev

update-ca-certificates

EOT

FROM base AS cpp


RUN <<EOT


EOT


FROM cpp as run

USER dev

SHELL ["/bin/bash", "-c"]

RUN <<EOT

EOT


WORKDIR /workspace
