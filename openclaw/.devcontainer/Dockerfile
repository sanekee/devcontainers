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
    sudo \
    netcat-traditional \
    python3 \
    python3-pip \
    python3-venv \
    xz-utils \
    openssh-server \
    systemd \
    unzip

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

FROM base AS node

USER dev

SHELL ["/bin/bash", "-c"]

ENV NVM_DIR="/home/dev/.nvm"
ENV NODE_VERSION=22

RUN <<EOT
curl -sSo- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
\. "$NVM_DIR/nvm.sh"

nvm install $NODE_VERSION
nvm use $NODE_VERSION
nvm alias default $NODE_VERSION

npm update -g npm
npm i -g openclaw

corepack enable pnpm

cat <<EOF >> ${HOME}/.bashrc

echo node version $(node -v)
echo pnpm version $(pnpm -v)

EOF

EOT

FROM node AS go

ARG GO_VERSION=1.26.1

RUN <<EOT
cd /tmp
curl -sLO https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz
ls -l /tmp | tee /home/dev/build.log
tar -C /usr/local -xzf go${GO_VERSION}.linux-amd64.tar.gz
ls -l /usr/local/  | tee -a /home/dev/build.log
rm -f go${GO_VERSION}.linux-amd64.tar.gz
EOT

ENV GO_VERSION=${GO_VERSION}

FROM go AS bun

RUN <<EOT
bash -c "$(curl -fsSL https://bun.sh/install)"
EOT

FROM bun AS brew

RUN <<EOT
bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
EOT

FROM brew AS workspace
WORKDIR /workspace


FROM workspace AS run
