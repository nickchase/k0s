# Support Insight

In many cases, especially when looking for [commercial support](commercial-support.md) there's a need to share the cluster state with other people.
While one could always give access to the live cluster, that is not always desired or even possible.

For this kind of case we can lean on the work our friends at [troubleshoot.sh](https://troubleshoot.sh) have done.

With the troubleshoot tool you can essentially take a dump of the cluster state and share it with other people. You can even use the [sbctl](https://github.com/replicatedhq/sbctl) tool to make the dump tarball act as the Kubernetes API.

Let's look at how this works with k0s.

## Setting up

To gather all the needed data we need another tool called [`support-bundle`](https://troubleshoot.sh/docs/support-bundle/introduction/).

You can download it from the [releases page](https://github.com/replicatedhq/troubleshoot/releases); make sure that you download the right architecture.

## Creating the support bundle

A Support Bundle tool needs to know what to collect and optionally, what to analyze. This is defined in a YAML file.

While you can customize the data collection and analysis for your specific needs, we've made a good reference for k0s that covers the core k0s components such as:

- collecting info on the host
- collecting system component statuses from `kube-system` namespace
- checking health of Kubernetes API, Etcd etc. components
- collecting k0s logs
- checking status of firewalls, anti-virus etc. services which are known to interfere with Kubernetes

As we need to collect host level information, you should run the commands on the hosts directly, on controllers and/or workers.

To get a support bundle, after setting up the [tooling](#setting-up) simply run:

```shell
support-bundle --kubeconfig /var/lib/k0s/pki/admin.conf https://docs.k0sproject.io/stable/support-bundle-<role>.yaml
```

The above `<role>` refers to either the `controller`or `worker`. For different roles we collect different things. If you are running a controller with `--enable-worker` or `--single`, where it becomes also a worker, you can also get a combined dump:

```shell
support-bundle --kubeconfig /var/lib/k0s/pki/admin.conf https://docs.k0sproject.io/stable/support-bundle-controller.yaml https://docs.k0sproject.io/stable/support-bundle-worker.yaml
```

Once the data collection and analysis finishes you will get a file titled `support-bundle-<timestamp>.tar.gz`. The file contains all the collected information you can share with other people.
