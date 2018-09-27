# DOCKER

Maintainer: Docker Inc. (SF)
Documentation at: docs.docker.com

```
docker version             # Shows version and check for server being up
docker info                # Shows current status and config
```

## Images

These are the base of container instances.

Images are incremental layers of [filesystem (union fs, copy-on-write) + metadata].

Layers are shared between images, and also with containers.

```
docker history <image>        # Show list of layers (including metadata changes)
docker inspect <image>        # Shows image metadata
```

Images are stored in Registries, and referenced as `repository[:tag]` where the tag defaults to `latest`.

### Registries

Images are stored remotely in a Registry (can be private) and locally in a cache.

Docker Inc.'s managed public registry: [Docker Hub](hub.docker.com)

```sh
docker image ls                             # Show list of images in cache
docker login/logout <registry-host>         # Logs in/out a registry to perform push/pull from it
docker push/pull <image>                    # Upload or download an image
```

### Repositories

Registries contain Repositories, containing one or more related images, normally of the same software.

They can be different versions, different base OS distros, etc. and are differentiated by Tag.

Naming (TODO: only in Docker Hub?):
  - Official: name, eg. nginx
  - User contributed: username/name, eg. johndoe/nginx

### Tags

Same image (same ID) can have many tags. Tags normally refer to version but are free, so you can have the same image with eg. 1.2.4, 1.2, 1, main and latest.

Can be thinked of like GIT tags, pointing to commits (images, from the layering perspective).

```sh
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]     # Tag an image
```

### Dockerfiles

Used to create new images. Normally named `Dockerfile`, but overrideable.

Every line (step) creates a new layer. When one changes, all subsequent need to be re-executed.

[Online reference]( https://docs.docker.com/engine/reference/builder/)

```Dockerfile
FROM image                                            # Base image
ENV MYSOFT_VER 1.2.3                                  # Set env vars, also then usable in RUNs here
RUN apt update \                                      # Run arbitrary commands
    && apt install my software deps
COPY . /opt/mysoft                                    # Copy files from host. Useful for dev testing: copy project
WORKDIR /opt/mysoft                                   # Set working directory
RUN ./install.sh
RUN ln -sf /dev/stdout /var/log/mysoft/$MYSOFT_VER/mysoft.log \     # Redirect logs so Docker can handle them
    && ln -sf dev/stderr /var/log/mysoft/$MYSOFT_VER/errors.log     # see `docker container logs`
EXPOSE 123 456                                        # Expose ports to virtual network (not host: see `-p`)
CMD ["mysoft", "--some-params", "..."]                # Run this command on container start (overrideable)
```

```sh
docker image build -t repo:tag dockerfile-path    # Builds an image. Note the path doesn't include `Dockerfile`, so can be eg. `.`.
```

## Containers

Containers are runnable instances of Images.

They have an extra filesystem layer on top for current disk state. It's commitable by TODO: `docker container commit`.

```sh
docker container ls        # Just list running container
                    -a     # All created containers
docker container run <image>              # Run an image in a new container
                     --name <name>        # Sets a name instead of random generated from scientists/hackers
                     --publish | -p <80:80>   # Listen in local port and forward to port listening inside container
                     --detach | -d        # Detaches the console
                     --env | -e <VAR=val> # Set env vars inside the container
                     -it                  # Allocate TTY, keep STDIN: run container interactively (shell)
                     --rm                 # Removes container after it's done
                     command              # Run command instead of #TODO in Dockerfile
docker container start <container>   # Starts the container
                       -ai           # Interactively (see run -it)
docker container stop <container>    # Stops the container
docker container exec -it <container> command  # Runs command on running container
docker container rm <container>      # Removes a container
docker container inspect <container> # Shows a JSON with container's config
docker container stats <container>   # Shows running stats
docker container top <container>     # Top inside the container
docker container log <container>     # Show last lines of logs
                     -f              # Follow logs
```

## Networking

Docker manages its own virtual networks and attaches containers to them.

These networks are created using drivers. Some are 3rd party but included are:
- bridge: The default. bridges and NATs to the host network
- host: Attach directly to the host network
- null: Dummy driver that's not connected. Will create unconnected NICs in containers

The default network is `bridge/docker0` and it's bridged to the host and NATted.

Bridge networks (except the default one) comes with Docker DNS enabled by default. It makes all containers in the same bridge network to see the rest by container name, as if it was it's hostname.

Using Docker DNS its better than fixed IP given how are those dinamically assigned. 

```sh
docker network ls                    # List virtual networks
docker network inspect               # 
docker network create <name>         # Create a new virtual network
                      --driver       # Use a particular driver. Defaults to bridge
docker network [dis]connect <container> <network>   # (Dis)Connects a container to a network: Creates a new interface on it.
docker container inspect --format '{{ .NetworkSettings.IPAddress }}'    # Shows IP address of container
docker container port <container>    # Show ports exposed to host
docker container run <image>              # Run an image in a new container
                     --net <network>      # TODO: attach to docker virtual network
                     --net-alias <name>   # Extra name to use as hostname in Docekr DNS. Repeats are OK (DNS RoundRobin)
                     --link <containers>  # TODO: make these containers visible through Docker DNS
                     --publish | -p <80:80>   # Listen in local port and forward to port listening inside container
```

## Persistence

### Volumes

Volumes are mounted in containers at specific locations that can be set in the Dockerfile using `VOLUME <mount-point>`, or in the commandline using `docker run -v [<name>:]<mount-point>`, which conveniently allows you to name the volume.

They're created either manually (`docker volume create`) or when a container is run.

When created manually, a driver can be specified. These include:
  - local: (default) It creates a directory under `/var/lib/docker/volumes/` and shares it to the container.\
  - TODO: others?

```sh
docker volume ls
docker volume inspect <volume>
docker volume create              # TODO: lets you specify drivers
```

### Bind Mounting

Mounts an arbitrary host path on a container's mount point.

```sh
docker run -v <absolute-path-in-host>:<mount-point-in-container>    # Note that it knows it's a path and not a name because it starts with a `/`.
```




