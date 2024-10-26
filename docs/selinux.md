# SELinux Overview

SELinux enforces mandatory access control policies that restrict user programs and system services, as well as access to files and network resources. Limiting privilege to the minimum required to work reduces or eliminates the ability of these programs and daemons to cause harm if faulty or compromised.

Enabling SELinux in the container runtime provides an additional security control to help further enforce isolation among deployed containers and the host.

This guide describes how to enable SELinux in the Kubernetes environment provided by k0s on CentOS and Red Hat Enterprise Linux (RHEL).

## Requirements

The following requirements must be in place for SELinux to protect your k0s cluster.

- SELinux is enabled on host OS of the worker nodes.
- SELinux has the container-selinux policy installed.
- SELinux labels are correctly set for k0s installation files of the worker nodes.
- SELinux is enabled in container runtime such as containerd on the worker nodes.

Here's how to make sure these requirements are satisfied.

### Check whether SELinux is enabled on host OS

SELinux is enabled on CentOS and RHEL by default. This command output indicates SELinux is enabled:

```shell
$ getenforce
Enforcing
```
If the status is not set to `Enforcing` you can enable it by editing the `/etc/selinux/config` file and adding the following:

```shell
SELINUX=enforcing
```
Save the file and reboot.

### Install container-selinux

[container-selinux](https://github.com/containers/container-selinux) must be installed.
In most Fedora based distributions including Fedora 37, Red Hat Enterprise Linux 7, 8 and 8, CentOS
7 and 8 and Rocky Linux 9, you can achieve this by simply installing the package container-selinux.

In RHEL 7 and CentOS 7 this is achieved by running:

```shell
yum install -y container-selinux
```

In the rest of the mentioned distributions run:

```shell
dnf install -y container-selinux
```

For Debian/Ubuntu-based Linux distributions, you can install container-selinux with:

```shell
sudo apt install policycoreutils selinux-basics selinux-utils -y
```

### Set SELinux labels for k0s installation files

Run the commands below on the host OS of the worker nodes.

```shell
DATA_DIR="/var/lib/k0s"
sudo semanage fcontext -a -t container_runtime_exec_t "${DATA_DIR}/bin/containerd.*"
sudo semanage fcontext -a -t container_runtime_exec_t "${DATA_DIR}/bin/runc"
sudo restorecon -R -v ${DATA_DIR}/bin
sudo semanage fcontext -a -t container_var_lib_t "${DATA_DIR}/containerd(/.*)?"
sudo semanage fcontext -a -t container_ro_file_t "${DATA_DIR}/containerd/io.containerd.snapshotter.*/snapshots(/.*)?"
sudo restorecon -R -v ${DATA_DIR}/containerd
```

### Enable SELinux in containerd of k0s

Add the lines below to `/etc/k0s/containerd.toml` on the worker nodes. You need to restart the k0s service on the node to make the change take effect.

```toml
[plugins."io.containerd.grpc.v1.cri"]
  enable_selinux = true
```

## Verify SELinux works in the Kubernetes environment

By following the example [Assign SELinux labels to a Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#assign-selinux-labels-to-a-container), deploy a testing pod using this YAML file:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-selinux
spec:
  containers:
  - image: busybox
    name: test-selinux
    command: ["sleep", "infinity"]
    securityContext:
      seLinuxOptions:
        level: "s0:c123,c456"
```

After the pod starts, ssh to the worker node on which the pod is running and check the pod process. It should display the label `s0:c123,c456` that you sepecified in YAML file:

```shell
$ ps -efZ | grep -F 'sleep infinity'
system_u:system_r:container_t:s0:c123,c456 root 3346 3288  0 16:39 ?       00:00:00 sleep infinity
```
