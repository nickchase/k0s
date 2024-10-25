# Backup/Restore overview

k0s has integrated support for backing up cluster state and configuration. The k0s backup utility backs up and restores k0s managed parts of the cluster.

The backups created by the `k0s backup` command include the following components:

- certificates (the content of the `<data-dir>/pki` directory)
- an etcd or Kine/SQLite snapshot, depending on which is in use
- k0s.yaml
- any custom defined manifests under `<data-dir>/manifests`
- any image bundles located under `<data-dir>/images`
- any helm configurations

Components **NOT** covered by the backup utility:

- `PersistentVolume`s of any running application
- the Kubernetes datastore if it runs on something other then etcd or Kine/SQLite
- any manual configuration changes to the cluster (In other words, changes that not created using manifests and stored in `<data-dir>/manifests`.)


## Backup/restore a k0s node locally

Any backup/restore related operations MUST be performed on a controller node.

### Backup (local)

To create backup, run the following command on the controller node:

```shell
k0s backup --save-path=<directory>
```

The directory used for the `save-path` value must exist and be writable. The default value is the current working directory.
The command creates a backup archive using the following naming convention: `k0s_backup_<ISODatetimeString>.tar.gz`

Because of the DateTime usage, it is guaranteed that none of the previously created archives would be overwritten.

To output the backup archive to stdout, use `-` as the save path.

### Restore (local)

To restore cluster state from the archive use the following command on the controller node:

```shell
k0s restore /tmp/k0s_backup_2021-04-26T19_51_57_000Z.tar.gz
```

The command will fail if the data directory for the current controller has overlapping data with the backup archive content. In other words, if objects in the backup already exist in the cluster, the `restore` command will fail.

The `restore` command uses the archived `k0s.yaml` as the cluster configuration description.

If your cluster uses HA, after restoring a single controller node, join the rest of the controller nodes to the cluster.
For example, the steps for an N node cluster would be:

- Restore backup on a fresh machine
- Run the controller there
- Join N-1 new machines to the cluster the same way as for the first setup.

To read the backup archive from stdin, use `-` as the file path.

### Encrypting backups (local)

By using `-` as the save or restore path, it is possible to pipe the backup archive through an encryption utility such as [GnuPG](https://gnupg.org/) or [OpenSSL](https://www.openssl.org/).

Note that unencrypted data will still briefly exist as temporary files on the local file system during the backup archvive generation.

#### Encrypting backups using GnuPG

Follow the instructions for your operating system to install the `gpg` command if it is not already installed.

This tutorial only covers the bare minimum for example purposes. For secure key management practices and advanced usage refer to the GnuPG user manual.

To generate a new key-pair, use:

```shell
gpg --gen-key
```

The key will be stored in your key ring.

```shell
gpg --list-keys
```

This will output a list of keys:

```shell
/home/user/.gnupg/pubring.gpg
------------------------------
pub   4096R/BD33228F 2022-01-13
uid                  Example User <user@example.com>
sub   4096R/2F78C251 2022-01-13
```

To export the private key for decrypting the backup on another host, note the key ID ("BD33228F" in this example) in the list and use:

```shell
gpg --export-secret-keys --armor BD33228F > k0s.key
```

To create an encrypted k0s backup:

```shell
k0s backup --save-path - | gpg --encrypt --recipient user@example.com > backup.tar.gz.gpg
```

#### Restoring encrypted backups using GnuPG

You must have the private key in your gpg keychain. To import the key that was exported in the previous example, use:

```shell
gpg --import k0s.key
```

To restore the encrypted backup, use:

```shell
gpg --decrypt backup.tar.gz.gpg | k0s restore -
```

## Backup/restore a k0s cluster using k0sctl

With k0sctl you can perform cluster level backup and restore remotely with one command.

### Backup (remote)

To create backup run the following command:

```shell
k0sctl backup
```

k0sctl connects to the cluster nodes to create a backup. The backup file is stored in the current working directory.

### Restore (remote)

To restore cluster state from the archive use the following command:

```shell
k0sctl apply --restore-from /path/to/backup_file.tar.gz
```

To use k0sctl, the control plane load balancer address (`externalAddress`) needs to remain the same between backup and restore. (All worker node components connect to this address and cannot currently be re-configured.)
