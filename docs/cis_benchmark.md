# Kube-bench Security Benchmark

[Kube-bench](https://github.com/aquasecurity/kube-bench) is an open source tool which can be used to verify security best practices as defined in the CIS Kubernetes Benchmark. It provides a number of tests to help harden your k0s clusters. By default, k0s will pass Kube-bench benchmarks with some exceptions, which are shown below.

## Run

Follow the Kube-bench [quick start instructions](https://github.com/aquasecurity/kube-bench/#quick-start).

After installing Kube-bench on the host that is running the `k0s` cluster run the following command:

```shell
kube-bench run --config-dir docs/kube-bench/cfg/ --benchmark k0s-1.0
```

## Summary of disabled checks

### Master Node Security Configuration

The current configuration has, in total, 8 master checks disabled:

1. **id: 1.2.10** - `EventRateLimit` requires external yaml config. It is left for the users to configure it

    ```yaml
    type: skip
    text: "Ensure that the admission control plugin EventRateLimit is set (Manual)"
    ```

2. **id: 1.2.12** - By default this isn't passed to the apiserver for air-gap functionality

    ```yaml
    type: skip
    text: "Ensure that the admission control plugin AlwaysPullImages is set (Manual)"
    ```

3. **id: 1.2.22** - For the sake of simplicity all audit configurations are skipped. It is left for users to configure 

    ```yaml
    type: skip
    text: "Ensure that the --audit-log-path argument is set (Automated)"
    ```

4. **id: 1.2.23** - For the sake of simplicity all audit configurations are skipped. It is left for the users to configure 

    ```yaml
    type: skip
    text: "Ensure that the --audit-log-maxage argument is set to 30 or as appropriate (Automated)"
    ```

5. **id: 1.2.24** - For the sake of simplicity all audit configurations are skipped. It is left for the user to configure 

    ```yaml
    type: skip
    text: "Ensure that the --audit-log-maxbackup argument is set to 10 or as appropriate (Automated)"
    ```

6. **id: 1.2.25** - For the sake of simplicity all audit configurations are skipped. It is left for the user to configure 

    ```yaml
    type: skip
    text: "Ensure that the --audit-log-maxsize argument is set to 100 or as appropriate (Automated)"
    ```

7. **id: 1.2.33** - By default it is not enabled. Left for the user to decide

    ```yaml
    type: skip
    text: "Ensure that the --encryption-provider-config argument is set as appropriate (Manual)"
    ```

8. **id: 1.2.34** - By default it is not enabled. Left for the user to decide

    ```yaml
    type: skip
    text: "Ensure that encryption providers are appropriately configured (Manual)"
    ```

### Worker Node Security Configuration

Four node checks are disabled:

1. **id: 4.1.1** - Not applicable because k0s does not use the kubelet service file

    ```yaml
    type: skip
    text: "Ensure that the kubelet service file permissions are set to 644 or more restrictive (Automated)"
    ```

2. **id: 4.1.2** - Not applicable because k0s does not use the kubelet service file

    ```yaml
    type: skip
    text: "Ensure that the kubelet service file ownership is set to root:root (Automated)"
    ```

3. **id: 4.2.6** - k0s does not set this. See https://github.com/kubernetes/kubernetes/issues/66693

    ```yaml
    type: skip
    text: "Ensure that the --protect-kernel-defaults argument is set to true (Automated)"
    ```

4. **id: 4.2.10** - k0s doesn't set this up because certs get auto rotated

    ```yaml
    type: skip
    text: "Ensure that the --tls-cert-file and --tls-private-key-file arguments are set as appropriate (Manual)"
    ```

### Control Plane Configuration

Three checks for the control plane are disabled:

1. **id: 3.1.1** - For the purpose of being fully automated, k0s skips this check

    ```yaml
    type: skip
    text: "Client certificate authentication should not be used for users (Manual)"
    ```

2. **id: 3.2.1** - Out-of-the box configuration does not have any audit policy configuration, but the user can customize it in the `spec.api.extraArgs` section of the config

    ```yaml
    type: skip
    text: "Ensure that a minimal audit policy is created (Manual)"
    ```

3. **id: 3.2.2** - Same as previous

    ```yaml
    type: skip
    text: "Ensure that the audit policy covers key security concerns (Manual)"
    ```

### Kubernetes Policies

Policy checks are also disabled. The checks are manual and are up to the end user to decide on them.
