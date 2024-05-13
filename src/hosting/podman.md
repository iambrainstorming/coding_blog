# Podman Tutorial


<iframe width="560" height="315" src="https://www.youtube.com/embed/YXfA5O5Mr18?si=coQxUI7YT-hMU8xB" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Installation command

```bash
sudo apt install podman	
podman machine init
podman machine start
podman info
podman version
```

## Custom container and Registry Configuration

Registries file

/etc/containers/registries.conf

Every time you try to connect the registries, this file is used.

You can also create a separate file in your user directory

$HOME/.config/containers/registries.conf


```
mkdir .config/containers
hx .config/containers/registries.conf


unqualified-search-registries = ['docker.io', 'ghrc.io', 'quay.io']

```

## Podman search registries

```
podman search <image_name>

podman search ubuntu
```

## Pulling a container image

```
podman search <image_name>
podman pull <image_name>
podman images
```

```
podman search ubuntu
podman pull docker.io/library/ubuntu	
```

## Running a container iamge

```
podman run -it <image_name>
podman ps
podman ps -a
```

-it flag tells podman to allocate a virtual terminal session within the container.

To remove the container after execution

```
podman run -it -rm <image_name>
```

podman ps shows only the container that are currently running.

To see all containers, including those that are not running use -a flag.

```
podman run -it ubuntu	
```

ls : to view the folders

exit: to exit from the container

You can also stop the container by pressing Ctrl+C


