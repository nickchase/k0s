# Run k0s in Docker

When you create a k0s cluster on top of Docker, by default both controller and worker nodes run in the same container to provide an easy local testing "cluster."

## Prerequisites

In keeping with the ethos behind Docker, all you will need to run Kubernetes in Docker is a [Docker environment](https://docs.docker.com/get-docker/) running on a Mac, Windows, or Linux system.

## Container images

The k0s containers are published both on Docker Hub and GitHub. For reasons of simplicity, the examples given here use Docker Hub. (GitHub requires a separate authentication that is not covered). Alternative links include:

- docker.io/k0sproject/k0s:v{{{ extra.k8s_version }}}-k0s.0
- ghcr.io/k0sproject/k0s:v{{{ extra.k8s_version }}}-k0s.0

**Note:** Due to Docker Hub tag validation scheme, these links use `-` as the k0s version separator instead of the usual `+`. So for example k0s version `v{{{ extra.k8s_version }}}+k0s.0` is tagged as `docker.io/k0sproject/k0s:v{{{ extra.k8s_version }}}-k0s.0`.

## Start k0s

To run Kubernetes in Docker, follow these steps:

### 1. Initiate k0s

As you might suspect, the first step is to simply start up the container:

```sh
docker run -d --name k0s --hostname k0s --privileged -v /var/lib/k0s -p 6443:6443 --cgroupns=host docker.io/k0sproject/k0s:v{{{ extra.k8s_version }}}-k0s.0 -- k0s controller --enable-worker
```

In this case, we've included the `--enable-worker` flag, so this command starts k0s with a worker. To run just a controller, running the command without the flag, as in:

```sh
docker run -d --name k0s --hostname k0s --privileged -v /var/lib/k0s -p 6443:6443 --cgroupns=host docker.io/k0sproject/k0s:v{{{ extra.k8s_version }}}-k0s.0 -- k0s controller
```

### 2. (Optional) Create additional workers

You can attach multiple workers nodes to the cluster to distribute your application containers to separate workers.

For each required worker:

1. Create a join token for the worker:

    ```sh
    token=$(docker exec -t -i k0s k0s token create --role=worker)
    ```

2. On the worker node, create the container to create and join the new worker:

    ```sh
    docker run -d --name k0s-worker1 --hostname k0s-worker1 --privileged -v /var/lib/k0s --cgroupns=host  docker.io/k0sproject/k0s:v{{{ extra.k8s_version }}}-k0s.0 k0s worker $token
    ```
    
    **Note:** If the container is running on a separate server, be sure to assign the token to the `token` environment variable.
    
### 3. Access your cluster

Access your cluster using kubectl:

You have two options for accessing the cluster. The first is to access it directly through the k0s-installed `kubectl` instance:

```sh
docker exec k0s k0s kubectl get nodes
```

Alternatively, you can grab the kubeconfig file with `docker exec k0s cat /var/lib/k0s/pki/admin.conf` and paste it into [Lens](https://github.com/lensapp/lens/) or use it with an independent `kubectl` installation.

## Use Docker Compose (alternative)

If you are running Infrastructure as Code, as an alternative you can run k0s using Docker Compose:

```yaml
version: "3.9"
services:
  k0s:
    container_name: k0s
    image: docker.io/k0sproject/k0s:v{{{ extra.k8s_version }}}-k0s.0
    command: k0s controller --config=/etc/k0s/config.yaml --enable-worker
    hostname: k0s
    privileged: true
    cgroup: host
    volumes:
      - "/var/lib/k0s"
    ports:
      - "6443:6443"
    network_mode: "bridge"
    environment:
      K0S_CONFIG: |-
        apiVersion: k0s.k0sproject.io/v1beta1
        kind: ClusterConfig
        metadata:
          name: k0s
        # Any additional configuration goes here ...
```

## Known limitations

### No custom Docker networks

Currently, you cannot run k0s nodes cannot be run if the containers are configured to use custom networks (for example, with `--net my-net`), because Docker sets up a custom DNS service within the network, which creates issues with CoreDNS. No completely reliable workaounds are available, however no issues should arise from running k0s cluster(s) on a bridge network.

## Next Steps

- [Install using k0sctl](k0sctl-install.md): Deploy multi-node clusters using just one command
- [Control plane configuration options](configuration.md): Networking and datastore configuration
- [Worker node configuration options](worker-node-config.md): Node labels and kubelet arguments
- [Support for cloud providers](cloud-providers.md): Load balancer or storage configuration
- [Installing the Traefik Ingress Controller](examples/traefik-ingress.md): Ingress deployment information
