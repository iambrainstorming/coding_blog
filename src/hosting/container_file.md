# Podman build

```
podman build . -t=image1
podman run -d --name=container1 image1
```

Then podman build . -t=image1 and podman run -d --name=container1 image1 to start it detached.

## Containerfile

```
FROM docker.io/library/ubuntu:22.04

# show backtraces
ENV RUST_BACKTRACE 1

# install tools and dependencies
RUN apt-get update && \
	DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
		ca-certificates && \
# apt cleanup
	apt-get autoremove -y && \
	apt-get clean && \
	find /var/lib/apt/lists/ -type f -not -name lock -delete; \
# add user and link ~/.local/share/polkadot to /data
	useradd -m -u 1000 -U -s /bin/sh -d /polkadot polkadot && \
	mkdir -p /data /polkadot/.local/share && \
	chown -R polkadot:polkadot /data && \
	ln -s /data /polkadot/.local/share/node-template

USER polkadot

# copy the compiled binary to the container
COPY --chown=polkadot:polkadot --chmod=774 target/release/node-template /usr/bin/node-template

# check if executable works in this container
RUN /usr/bin/node-template --version

# ws_port
EXPOSE 9930 9333 9944 30333 30334

CMD ["/usr/bin/node-template"]
```

## Breakdown of file

Here is a breakdown of the Dockerfile:
1. `FROM docker.io/library/ubuntu:22.04`: This line specifies the base image to use for the container. In this case, it is using the Ubuntu 22.04 image from the official Docker library.
2. `ENV RUST_BACKTRACE 1`: This line sets an environment variable `RUST_BACKTRACE` to `1` inside the container. This variable controls whether Rust programs print a backtrace on panic.
3. `RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends ca-certificates`: This line runs a command inside the container to update the package repository and install the `ca-certificates` package. The `DEBIAN_FRONTEND=noninteractive` option tells `apt-get` to run in non-interactive mode, which means that it will not prompt the user for input during the installation process.
4. `apt-get autoremove -y && apt-get clean && find /var/lib/apt/lists/ -type f -not -name lock -delete`: These lines perform some cleanup tasks after the package installation. The `apt-get autoremove -y` command removes any packages that were automatically installed as dependencies but are no longer needed. The `apt-get clean` command removes the package cache, and the `find` command deletes any leftover package list files.
5. `useradd -m -u 1000 -U -s /bin/sh -d /polkadot polkadot`: This line creates a new user account inside the container with the username `polkadot`. The `-m` option creates a home directory for the user, the `-u 1000` option sets the user ID to 1000, the `-U` option creates a group with the same name as the user, the `-s /bin/sh` option sets the default shell to `/bin/sh`, and the `-d /polkadot` option sets the home directory to `/polkadot`.
6. `mkdir -p /data /polkadot/.local/share && chown -R polkadot:polkadot /data && ln -s /data /polkadot/.local/share/node-template`: These lines create some directories and set up the permissions for the `polkadot` user. The `mkdir -p` command creates the `/data` and `/polkadot/.local/share` directories, the `chown -R` command changes the ownership of the `/data` directory to the `polkadot` user, and the `ln -s` command creates a symbolic link from `/data` to `/polkadot/.local/share/node-template`.
7. `USER polkadot`: This line sets the user inside the container to the `polkadot` user that was created earlier. This means that any subsequent commands in the Dockerfile will be executed as the `polkadot` user.
8. `COPY --chown=polkadot:polkadot --chmod=774 node-template /usr/bin/node-template`: This line copies the `node-template` binary from the host machine into the container and sets the ownership and permissions for the file. The `--chown` option sets the ownership of the file to the `polkadot` user and group, and the `--chmod` option sets the permissions to `774`, which means that the owner and group have read, write, and execute permissions, and others have only read and execute permissions.
9. `RUN /usr/bin/node-template --version`: This line runs the `node-template` binary inside the container to check if it is working correctly. The `--version` option prints the version of the binary.
10. `EXPOSE 9930 9333 9944 30333 30334`: This line exposes several ports on the container. This means that the application running inside the container will be accessible on these ports.
11. `CMD ["/usr/bin/node-template"]`: This line sets the command that will be run when the container starts. In this case, it will run the `node-template` binary.
