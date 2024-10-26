# User Management

Kubernetes, and thus k0s, does not have any built-in functionality to manage users. Kubernetes relies solely on external sources for user identification and authentication. A client certificate is considered an external source in this case as Kubernetes api-server "just" validates that the certificate is signed by a trusted CA. This means that it is recommended to use, for example, [OpenID Connect](./examples/oidc/oidc-cluster-configuration.md) to configure the API server to trust tokens issued by an external Identity Provider.

k0s comes with some helper commands to create a kubeconfig with client certificates for users, but there are a few caveats to take into consideration when using client certificates:

* Client certificates have long expiration times; they're valid for one year
* Client certificates **cannot be revoked** (This is a general Kubernetes challenge.)

## Adding a Cluster User

The first step in giving a user access is to create the user within Kubernetes itself.

Run the [kubeconfig create](cli/k0s_kubeconfig_create.md) command on the controller to add a user to the cluster. The command outputs a kubeconfig for the user, to use for authentication.

```shell
k0s kubeconfig create [username]
```

## Enabling Access to Cluster Resources

1. Create the user with the `system:masters` group to grant the user access to the cluster:

   ```shell
   k0s kubeconfig create --groups "system:masters" [username] > k0s.config
   ```

   For example:

   ```shell
   k0s kubeconfig create --groups "system:masters" testUser > k0s.config
   ```

2. Create a `roleBinding` to grant the user access to resources. For example, to give the user testUser access to the `admin` role:

   ```shell
   k0s kubectl create clusterrolebinding --kubeconfig k0s.config testUser-admin-binding --clusterrole=admin --user=testUser
   ```
