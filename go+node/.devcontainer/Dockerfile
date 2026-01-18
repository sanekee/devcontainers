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

FROM base AS go

ARG GO_VERSION=1.24.0

RUN <<EOT
cd /tmp
curl -sLO https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz
ls -l /tmp | tee /home/dev/build.log
tar -C /usr/local -xzf go${GO_VERSION}.linux-amd64.tar.gz
ls -l /usr/local/  | tee -a /home/dev/build.log
rm -f go${GO_VERSION}.linux-amd64.tar.gz
EOT

ENV GO_VERSION=${GO_VERSION}

FROM go AS install

USER dev

SHELL ["/bin/bash", "-c"]

RUN <<EOT

curl -sSo- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
\. "$HOME/.nvm/nvm.sh"

nvm install 22
corepack enable pnpm

echo <<EOF >> ${HOME}/.bashrc

echo node version $(node -v)
echo pnpm version $(pnpm -v)

export PATH=\${PATH}:/usr/local/go/bin:\${HOME}/go/bin

go install github.com/go-delve/delve/cmd/dlv@latest
EOF

EOT

WORKDIR /workspace

FROM install AS run
