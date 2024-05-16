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

## Running a container image

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

Since we didn't specified the name using the flag -name, podman generated a name for us.

Best practice is not assign name, let podman assign the name to avoid conflits
## Working with Containers

```
podman run --name <container_name> -p ext_port:int_porst <container_image>

// This start the container in detached mode, this means it runs in background
podman start <container_name>

podman inspect <container_name> 

// List all ports mapping in the container
podman port <container_name>

// To stop the container
podman stop <container_name>


// Remove the container
podman rm <container_name>


// Remove the container image, specify the image id
podman rmi <container_image>

// Check all the images in the system
podman images

// To get list of all commands

podman --help

```

## Buidling a Container Image

```
hx Dockerfile  

podmand build -t <image_name> .

podman run --name <container_name> -p 8080:8080 <image_name>:<tag>

e.g.
podman run --name pdm-golang -p 8080:8080 pdm-golang:latest
```

## Sharing a container images

```
podman login <registry_name>

podman build -t <username>/<image_name> .

podman push <username>/<container_name>  
```

Use dockerhub

```
podman login docker.io

podman build -t reaudito/pdm-golang . 

podman push reaudito/pdm-golang

podman search pdm-golang

podman pull docker.io/reaudito/pdm-golang
  
```

## Building Pod with Podman

Pod are similar to kubernetes pods. They provide a way for applications to be organized and scaled within a Kubernets cluster.

Kubernetes API objects such as Deployments, ReplicaSets, and StatefulSets are used to manage them.

```
podman pod --help


// Create a new empty pod
podman pod create --name <pod_name>

// List all created pods
podman pod ls


// List all the containers in the pod
podman ps -a --pod  
```


Pods are a group of one or more containers sharing the same network, pid and ipc namespaces.

```
// Don't use podman name to avoid conflit, let podman assign it.
podman pod create --name pdm-golang-pod 

podman pod ls

```

You can see there is one container inside the pod pdm-golang-pod, but you created a empty pod, because podmand automatically added a infra container. Allow you to add container in the pod and also start and stop containers within the Pod by keeping the Pod running. 

Default infra container is based on the image k8s.gcr.io/pause

You can display it

```
podman ps -a --pod  
```

## Working with containers inside a Pod with Podman

```
podman run -dt --pod <pod_name> <container_image_name>

podman pod start <pod_name>

podman pod stop <pod_name>

podman pod rm <pod_name>  
```

```
podman run -dt -pod pdm-golang-pod pdm-golang  
```


Even if container is inside the pod, you can use all the avaiable commands to work with containers. 

```
// List all the pods
podman pod ls

podman pod stop pdm-golang-pod

podman pod start pdm-golang-pod

podman pod stop pdm-golang-pod

podman pod rm pdm-golang-pod  
```

## Working with K8s YAML Manifest
