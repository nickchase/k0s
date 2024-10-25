# System requirements

System requirements for k0s are modest, but important.

## Minimum memory and CPU requirements

The minimum requirements for k0s detailed below are approximations, and thus your results may vary based on the types of workload.

| Role                | Memory (RAM) | Virtual CPU (vCPU) |
|---------------------|--------------|--------------------|
| Controller node     | 1   GB       | 1 vCPU             |
| Worker node         | 0.5 GB       | 1 vCPU             |
| Controller + worker | 1   GB       | 1 vCPU             |

## Controller node recommendations

| # of Worker nodes | # of Pods    | Recommended RAM | Recommended vCPU |
|-------------------|--------------|-----------------|------------------|
| up to   10        | up to   1000 | 1-2   GB        | 1-2   vCPU       |
| up to   50        | up to   5000 | 2-4   GB        | 2-4   vCPU       |
| up to  100        | up to  10000 | 4-8   GB        | 2-4   vCPU       |
| up to  500        | up to  50000 | 8-16  GB        | 4-8   vCPU       |
| up to 1000        | up to 100000 | 16-32 GB        | 8-16  vCPU       |
| up to 5000        | up to 150000 | 32-64 GB        | 16-32 vCPU       |

k0s is built to obey the standard Kubernetes limits for the maximum number of nodes, pods, and so on. For example, a cluster must satisfy all of the following constraints:
    - No more than 110 pods per node
    - No more than 5,000 nodes
    - No more than 150,000 total pods
    - No more than 300,000 total containers 
For more details, see [the Kubernetes considerations for large clusters](https://kubernetes.io/docs/setup/best-practices/cluster-large/).

### Controller node measured memory consumption

The actual memory consumption of controllers varies depending on load, but the following table shows the measured memory consumption in the cluster of one controller node as a representative example you can use for reference.

| # of Worker nodes | # of Pods (besides default) | Memory consumption |
|-------------------|-----------------------------|--------------------|
| 1                 | 0                           | 510  MB            |
| 1                 | 100                         | 600  MB            |
| 20                | 0                           | 660  MB            |
| 20                | 2000                        | 1000 MB            |
| 50                | 0                           | 790  MB            |
| 50                | 5000                        | 1400 MB            |
| 100               | 0                           | 1000 MB            |
| 100               | 10000                       | 2300 MB            |
| 200               | 0                           | 1500 MB            |
| 200               | 20000                       | 3300 MB            |

Measurement details:

- k0s v1.22.4+k0s.2 (default configuration with etcd)
- Ubuntu Server 20.04.3 LTS, OS part of the used memory was around 180 MB
- Hardware: AWS t3.xlarge (4 vCPUs, 16 GB RAM)
- Pod image: nginx:1.21.4

## Storage

The specific storage consumption for k0s is as follows:

| Role                 | Usage (k0s part) | Minimum required |
|----------------------|------------------|------------------|
| Controller node      | ~0.5 GB          | ~0.5 GB          |
| Worker node          | ~1.3 GB          | ~1.6 GB          |
| Controller + worker  | ~1.7 GB          | ~2.0 GB          |

For worker nodes you must have at least 15% relative disk space free.
Make sure to consider the operating system and application requirements in addition to k0s.

Use of an SSD for [optimal storage performance](https://etcd.io/docs/current/op-guide/performance/) is recommended, as cluster latency and throughput are sensitive to storage.

## Host operating system

k0s runs on the following operating systems:

- Linux (see [Linux specific requirements] for details)
- Windows Server 2019

Note that a cluster must have all controllers and at least one worker running on Linux; extra workers can run on Windows.

[Linux specific requirements]: external-runtime-deps.md#linux-specific

## Architecture

k0s supports the following architectures:

- x86-64
- ARM64
- ARMv7

## Networking

For information on the required ports and protocols, refer to [networking](networking.md).

## External runtime dependencies

k0s strives to be as independent from the OS as possible. The current and past
external runtime dependencies are documented [here](external-runtime-deps.md).

To run automated compatiblility checks on your system, use
[`k0s sysinfo`](cli/k0s_sysinfo.md).


