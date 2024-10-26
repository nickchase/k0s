# Environment variables

`k0s install` does not support environment variables.

Setting environment variables for components used by k0s depends on the init system in use. The environment variables set in the `k0scontroller` or `k0sworker` service will be inherited by k0s components such as `etcd`, `containerd`, `konnectivity`, and so on.

Component-specific environment variables can also be set in the `k0scontroller` or `k0sworker` service using prefixes. For example, the `CONTAINERD_HTTPS_PROXY` environment variable will be applied to the `containerd` process, and the prefix `CONTAINERD_` will be stripped, leaving the `HTTPS_PROXY` variable to be used by the `containerd` process.

The exception to this rule is for those variables prefixed with `ETCD_`. These variables are handled specially, and the prefix will not be stripped. For example, `ETCD_MAX_WALS` will still be `ETCD_MAX_WALS` in the etcd process.

That said, the proxy envs `HTTP_PROXY`, `HTTPS_PROXY`, and `NO_PROXY` are always overridden by component specific environment variables, so `ETCD_HTTPS_PROXY` will still be converted to `HTTPS_PROXY` in the etcd process.

## SystemD

To add environment variables to SystemD, create a drop-in directory and add a config file with the desired environment variables:

```shell
mkdir -p /etc/systemd/system/k0scontroller.service.d
tee -a /etc/systemd/system/k0scontroller.service.d/http-proxy.conf <<EOT
[Service]
Environment=HTTP_PROXY=192.168.33.10:3128
EOT
```

## OpenRC

To add environment variables to OpenRC, export the desired environment variables overriding the service configuration to the /etc/conf.d directory:

```shell
echo 'export HTTP_PROXY="192.168.33.10:3128"' > /etc/conf.d/k0scontroller
```
