# Manual Install (Advanced)

You can manually set up k0s nodes to create a multi-node cluster that is locally managed on each node. This involves first installing each node separately, and then connecting the nodes together using access tokens.

**Note**: If you plan to use automatic upgrades, [install with k0sctl](k0sctl-install.md) instead.

## Prerequisites

Before proceeding, make sure to review the [System Requirements](system-requirements.md).

Though this material is written for and has been tested on Debian/Ubuntu, you can use it for any Linux distro that is running either a Systemd or OpenRC init system.

You can also speed up the use of the `k0s` command by enabling [shell completion](shell-completion.md).

## Create the k0s cluster

### 1. Download k0s to a controller node

Run the k0s download script to download the latest stable version of k0s and make it executable from `/usr/bin/k0s`.

```shell
curl --proto '=https' --tlsv1.2 -sSf https://get.k0s.sh | sudo sh
```

The download script accepts the following environment variables:

| Variable                    | Purpose                                                              |
|:----------------------------|:---------------------------------------------------------------------|
| `K0S_VERSION=v{{{ extra.k8s_version }}}+k0s.0` | Select the version of k0s to be installed         |
| `DEBUG=true`                                   | Output commands and their arguments at execution. |

**Note**: If you need environment variables and use sudo, you can do:

```shell
curl --proto '=https' --tlsv1.2 -sSf https://get.k0s.sh | sudo K0S_VERSION=v{{{ extra.k8s_version }}}+k0s.0 sh
```

### 2. Bootstrap a controller node

Create a configuration file:

```shell
mkdir -p /etc/k0s
k0s config create > /etc/k0s/k0s.yaml
```

For information on settings modification, refer to the [configuration](configuration.md) documentation. Once you're statisfied with the configurationn, go ahead and create and start the controller:

```shell
sudo k0s install controller -c /etc/k0s/k0s.yaml
```

```shell
sudo k0s start
```

The k0s process acts as a "supervisor" for all of the control plane components. In moments the control plane will be up and running.

### 3. Create a join token

Now you need to join new nodes to the cluster. To do that, you need a token, which embeds information that enables mutual trust between the worker and controller(s).

To get a token, run the following command on one of the existing controller nodes:

```shell
sudo k0s token create --role=worker
```

The resulting output is a long [token](#about-tokens) string. For enhanced security, you can set an expiration time for the token:

```shell
sudo k0s token create --role=worker --expiry=100h > token-file
```

### 4. Add workers to the cluster

For each worker node, first download k0s as in step 1, then run k0s in worker mode with the join token you created:

```shell
sudo k0s install worker --token-file /path/to/token/file
```

```shell
sudo k0s start
```

#### About tokens

Join tokens are base64-encoded [kubeconfigs](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/). We've done this for several reasons:

- They have a well-defined structure
- They can be used directly as bootstrap auth configs for kubelet
- They have CA info embedded, enabling mutual trust

The bearer token embedded in the kubeconfig is a [bootstrap token](https://kubernetes.io/docs/reference/access-authn-authz/bootstrap-tokens/). For controller join tokens and worker join tokens k0s uses different usage attributes to ensure that k0s can validate the token role on the controller side.

### 5. Add controllers to the cluster

Before you add additional controllers, keep the following in mind:
* You must use either etcd or an external data store (MySQL or Postgres) via [kine](kine-data-store.md). 
* Controllers must meet the requirements noted in the [high availability configuration](high-availability.md). 
* The configuration must be identical for all controller nodes.

To add a new cluster, you'll first need a new controller join token:

```shell
sudo k0s token create --role=controller --expiry=1h > token-file
```

On the new controller, first download k0s as in step 1, then run:

```shell
sudo k0s install controller --token-file /path/to/token/file -c /etc/k0s/k0s.yaml
```

Note that each controller in the cluster must have a `k0s.yaml` file. Otherwise, some cluster nodes will use default config values, which will lead to inconsistent behavior. If your configuration file includes IP addresses (node address, sans, etcd peerAddress), remember to update them accordingly for this specific controller node.

Finally. start k0s on that node:

```shell
sudo k0s start
```

### 6. Check k0s status

To get general information about your k0s instance's status on a particular node:

```shell
 sudo k0s status
```

```shell
Version: v{{{ extra.k8s_version }}}+k0s.0
Process ID: 2769
Parent Process ID: 1
Role: controller
Init System: linux-systemd
Service file: /etc/systemd/system/k0scontroller.service
```

### 7. Access your cluster

Use the Kubernetes 'kubectl' command-line tool that comes with the k0s binary to deploy your application or check your node status:

```shell
sudo k0s kubectl get nodes
```

```shell
NAME   STATUS   ROLES    AGE    VERSION
k0s    Ready    <none>   4m6s   v{{{ extra.k8s_version }}}+k0s
```

You can also access your cluster easily with [Lens](https://k8slens.dev/), simply by copying the kubeconfig and pasting it to Lens:

```shell
sudo cat /var/lib/k0s/pki/admin.conf
```

**Note**: To access the cluster from an external network you must replace `localhost` in the `kubeconfig` with the host ip address for your controller.

## Next Steps

- [Install using k0sctl](k0sctl-install.md): Deploy multi-node clusters using just one command
- [Control plane configuration options](configuration.md): Networking and datastore configuration
- [Worker node configuration options](worker-node-config.md): Node labels and kubelet arguments
- [Support for cloud providers](cloud-providers.md): Load balancer or storage configuration
- [Installing the Traefik Ingress Controller](examples/traefik-ingress.md): Ingress deployment information
