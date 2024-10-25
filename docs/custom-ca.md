# Install using custom CA certificates and a SA key pair

k0s generates all needed certificates automatically in the `<data-dir>/pki` directory (`/var/lib/k0s/pki`, by default).  

Sometimes, however, you need to have the CA certificates and SA key pair in advance. Fortunately, to make this work, you just need to put the appropriate files in the `<data-dir>/pki` and `<data-dir>/pki/etcd` directories:

```shell
export LIFETIME=365
mkdir -p /var/lib/k0s/pki/etcd
cd /var/lib/k0s/pki
openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -key ca.key -sha256 -days $LIFETIME -out ca.crt -subj "/CN=Custom CA"
openssl genrsa -out sa.key 2048
openssl rsa -in sa.key -outform PEM -pubout -out sa.pub
cd ./etcd
openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -key ca.key -sha256 -days $LIFETIME -out ca.crt -subj "/CN=Custom CA"
```

You can then [install k0s as usual](./install.md).

## Pre-generated tokens

It's also possible to get join tokens without having a running cluster:

```shell
k0s token pre-shared --role worker --cert /var/lib/k0s/pki/ca.crt --url https://<controller-ip>:6443/
```

This command generates a join token and a Secret. You can then deploy the Secret to the cluster to authorize the token.
For example, you can put the Secret under the [manifest](manifests.md) directory and it will be deployed automatically.

Please note that if you are generating a join token for a controller, the port number must be 9443 instead of 6443, because controller bootstrapping requires talking to the `k0s-apiserver` instead of the `kube-apiserver`.

For example:

```shell
k0s token pre-shared --role controller --cert /var/lib/k0s/pki/ca.crt --url https://<controller-ip>:9443/
```
