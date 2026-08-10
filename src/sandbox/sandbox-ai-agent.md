# Sandbox Open Code AI agent for dioxus app

## Install Podman

On Ubuntu:

```bash
sudo apt update
sudo apt install podman
```

Check:

```bash
podman --version
```

You don't need Docker Desktop.

---

## Create an OpenCode container image

Create a directory:

```bash
mkdir -p ~/opencode-sandbox
cd ~/opencode-sandbox
```

Create `Containerfile`:

```dockerfile
FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    curl \
    git \
    build-essential \
    pkg-config \
    libssl-dev \
    ca-certificates \
    unzip \
    wget \
    sudo

# Rust
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs \
    | sh -s -- -y

ENV PATH="/root/.cargo/bin:${PATH}"

RUN rustup default stable

# WebAssembly target
RUN rustup target add wasm32-unknown-unknown

# OpenCode
RUN curl -fsSL https://opencode.ai/install | bash

ENV PATH="/root/.opencode/bin:${PATH}"

WORKDIR /workspace

CMD ["bash"]
```

Build it:

```bash
podman build -t opencode-sandbox .
```

This gives you:

* Ubuntu
* Git
* Rust
* Cargo
* OpenCode
---

# Run it against one project

Suppose your project is:

```text
~/Documents/workspace/smartpushti-nostr
```

Run:

```bash
podman run --rm -it \
    --name opencode-smartpushti \
    --network=host \
    -v "$HOME/Documents/workspace/smartpushti-nostr:/workspace:rw" \
    opencode-sandbox
```


## Install on container

```bash
root@amiya:/workspace#
```

```bash
cargo install cargo-binstall

cargo binstall dioxus-cli --force
```


## Don't expose your SSH keys

This is particularly important for an AI coding agent.

**Do not do this:**

```bash
-v ~/.ssh:/root/.ssh
```

The whole point of this setup is that OpenCode shouldn't have access to your host credentials.

The container therefore won't have:

```text
~/.ssh
~/.gnupg
~/.aws
~/.config
```

unless you explicitly mount them.

---

## Add a non-root user

For an even better setup, I would **not run the agent as root**.

Add this to the Containerfile:

```dockerfile
RUN useradd -m -u 1000 developer

RUN chown -R developer:developer /workspace

USER developer

WORKDIR /workspace

ENV PATH="/home/developer/.cargo/bin:/home/developer/.foundry/bin:/home/developer/.opencode/bin:${PATH}"

CMD ["bash"]
```

But there is a practical issue: the host project's files need to have a compatible UID/GID.

Podman makes this easier with:

```bash
--userns=keep-id
```

So your final command can be:

```bash
podman run --rm -it \
    --userns=keep-id \
    -v "$HOME/projects/smartpushti:/workspace:rw" \
    opencode-sandbox
```

This is a much nicer setup for a development environment.

---
