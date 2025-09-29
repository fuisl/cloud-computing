# Minimal Ubuntu dev box with SSH client + common tools
FROM ubuntu:22.04

ARG DEBIAN_FRONTEND=noninteractive

# Create non-root user "dev" with uid/gid 1000 by default
RUN groupadd -g 1000 dev && useradd -m -u 1000 -g 1000 dev

# Basic tooling: ssh client, curl, git, unzip, zip, jq, build tools, python3, pip
RUN apt-get update && apt-get install -y --no-install-recommends \
    openssh-client \
    ca-certificates \
    curl \
    git \
    unzip \
    zip \
    jq \
    less \
    nano \
    python3 \
    python3-pip \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

ARG TARGETARCH

# Map Docker arch -> AWS archive name
RUN set -eux; \
    case "$TARGETARCH" in \
      amd64)  AWS_ARCH=x86_64 ;; \
      arm64)  AWS_ARCH=aarch64 ;; \
      *) echo "Unsupported TARGETARCH: $TARGETARCH" >&2; exit 1 ;; \
    esac; \
    curl -sSL "https://awscli.amazonaws.com/awscli-exe-linux-${AWS_ARCH}.zip" -o /tmp/awscliv2.zip && \
    unzip /tmp/awscliv2.zip -d /tmp && \
    /tmp/aws/install && \
    rm -rf /tmp/aws /tmp/awscliv2.zip

# Switch to root for admin setup
USER root

# Install sudo and allow the 'dev' user to sudo without a password
RUN apt-get update && apt-get install -y --no-install-recommends sudo \
 && echo "dev ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/90-dev \
 && chmod 0440 /etc/sudoers.d/90-dev \
 && rm -rf /var/lib/apt/lists/*

USER dev

WORKDIR /workspace

# Improve SSH UX: accept new host keys automatically on first connect
RUN mkdir -p /home/dev/.ssh && \
    printf "Host *\n  StrictHostKeyChecking accept-new\n" > /home/dev/.ssh/config
