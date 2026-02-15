# Base Image: Ubuntu because it has the best package availability
FROM ubuntu:22.04

ARG DEBIAN_FRONTEND=noninteractive

# 1. Install Basics & C++ Toolchain
RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    ninja-build \
    gcc-arm-none-eabi \
    libnewlib-arm-none-eabi \
    git \
    wget \
    curl \
    python3 \
    && rm -rf /var/lib/apt/lists/*

# 2. Install Rust
ENV RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH

RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y \
    && chmod -R a+w $RUSTUP_HOME $CARGO_HOME \
    && rustup target add thumbv7em-none-eabihf

# 3. Install Renode (Simulation)
RUN wget https://builds.renode.io/renode-latest.linux-portable.tar.gz \
    && tar -xzf renode-latest.linux-portable.tar.gz -C /opt \
    && rm renode-latest.linux-portable.tar.gz \
    && ln -s /opt/renode*/renode /usr/bin/renode

# 4. Set Workspace
WORKDIR /workspace
