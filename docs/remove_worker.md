# Remove or replace a worker node

You can manually remove or replace a worker from a multi-node k0s cluster (>=2 workers) without downtime.

## Remove a worker

To remove a worker, you will first need to remove any pods it's running, then delete it:

```shell
# Remove the containers from the node and cordon it
k0s kubectl drain --ignore-daemonsets --delete-emptydir-data <worker>
# Delete the node from the cluster
k0s kubectl delete node <worker>
```

The worker is now removed from the cluster.
To [reset k0s on the machine](reset.md), run the following commands:

```shell
k0s stop
k0s reset
reboot
```

## Replace a worker

To replace a worker, you first remove the old worker (as described above) then follow the [manual installation procedure](k0s-multi-node.md) to add the new one.
