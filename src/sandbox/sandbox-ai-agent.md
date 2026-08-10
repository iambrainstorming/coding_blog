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
    clang \
    llvm \
    pkg-config \
    libssl-dev \
    ca-certificates \
    unzip \
    wget \
    sudo \
    && rm -rf /var/lib/apt/lists/*

# Ubuntu 24.04 ships a built-in "ubuntu" user at UID/GID 1000 already.
# Just use it directly instead of creating/renaming a user.
RUN echo "ubuntu ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/ubuntu \
    && chmod 0440 /etc/sudoers.d/ubuntu

USER ubuntu
WORKDIR /home/ubuntu

# Rust
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs \
    | sh -s -- -y
ENV PATH="/home/ubuntu/.cargo/bin:${PATH}"
RUN rustup default stable \
    && rustup target add wasm32-unknown-unknown
RUN cargo install cargo-binstall
RUN cargo binstall dioxus-cli --force

# OpenCode
RUN curl -fsSL https://opencode.ai/install | bash
ENV PATH="/home/ubuntu/.opencode/bin:${PATH}"

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

## Run it against one project

Suppose your project is:

```text
~/Documents/workspace/symbiosky-nostr
```

Run:

```bash
podman run --rm -it \
    --name opencode-smartpushti \
    --userns=keep-id \
    -p 8080:8080 \
    -v "$HOME/Documents/workspace/symbiosky-nostr:/workspace:rw" \
    opencode-sandbox
```


## Keep container

```bash
podman run -it \
    --name opencode-smartpushti \
    --userns=keep-id \
      -p 8080:8080 \
    -v "$HOME/Documents/workspace/symbiosky-nostr:/workspace:rw" \
    opencode-sandbox
```

```bash
cd packages/web
dx serve --addr 0.0.0.0 --port 8080
```

```bash
podman start -ai opencode-smartpushti
```



```bash
ubuntu@e889c99b0b72:/workspace$
```


## Run opencode

```bash
opencode
```
## To exit from session

Type `exit` or `Ctrl+D`

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
