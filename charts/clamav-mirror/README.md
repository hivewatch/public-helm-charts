# Kellnr Helm-Chart

This repository provides a [Helm](https://helm.sh/) Chart for [Kellnr](https://kellnr.io), the Rust registry for self-hosting crates.

## Installation

Add the _Kellr_ helm repository which contains _Kellnr_:

```bash
# Add  repository
helm repo add kellnr https://kellnr.github.io/helm

# Update to latest version
helm repo update
```

For a list of possible configuration flags see below: [Configuration](#configuration)

To install a minimal _Kellnr_ installation with in-memory storage run the command below. The in-memory storage variant is useful for testing but not recommended for production as all data (crates, users, ...) will be lost if the container restarts.

```bash
helm install kellnr kellnr/kellnr --set kellnr.origin.hostname="kellnr.example.com"
```

For a persistent _Kellnr_ instance, a _PersistentVolumeClaim_ (PVC) is needed. The helm chart can create a _PersistentVolumeClaim_ and _PersistentVolume_ (PV), if you don't have one already.

```bash
# Use an existing PersistentStorage (storage_name) to get a PersistentStorageClaim.
# The storage class can be overwritten with "pvc.storageClassName" and defaults to "manual".
helm install kellnr kellnr/kellnr \
    --set pvc.enabled=true --set pvc.name="storage_name" \
    --set kellnr.origin.hostname="kellnr.example.com"
```

```bash
# Create a new PersistentVolume and a corresponding claim. The host path e.g. "/mnt/kellnr" has to exists before the PV is created.
helm install kellnr kellnr/kellnr \
    --set pv.enabled=true --set pv.path="/mnt/kellnr" \
    --set pvc.enabled=true \
    --set kellnr.origin.hostname="kellnr.example.com"
```

For information about the _Cargo_ setup and default values, see the official [Kellnr documentation](https://kellnr.io/documentation).

## Upgrade

Upgrade _Kellnr_ by updating the Helm repository and applying the latest version.

```bash
# Update helm repositories
helm repo update

# Upgrade Kellnr
helm upgrade kellnr kellnr/kellnr
```

## Configuration

All settings can be set with the `--set name=value` flag on `helm install`. Some settings are required and have to be provided other are recommended for security reasons.

### Kellnr

Check the [documentation](https://kellnr.io/documentation) and the [values.yaml](./charts/kellnr/values.yaml) for possible configuration values. 

### Service

Settings to configure the web-ui/API endpoint service and the crate index service.

| Setting                | Required | Description                                                   | Default   |
| ---------------------- | -------- | ------------------------------------------------------------- | --------- |
| service.api.type       | No       | Type of the service that exports the API and web-ui endpoint. | ClusterIP |
| service.api.port       | No       | Port of the service that exports the API and web-ui endpoint. | 80        |

### DNS

You can set DNS servers for _Kellnr_ which should be used instead of the default one.

> Kellnr needs to be able to resolve it's own domain name, to be able to generate Rustdocs automatically.

| Setting | Required | Description | Default |
| ---------------------- | -------- | ------------------------------------------------------------- | --------- |
| dns.enabled | No | Enable an additional _dnsPolicy_ | false |
| dns.dnsPolicy | No | Set the _dnsPolicy_ | "None" |
| dns.dnsConfig.nameservers | No | List of nameservers to use. | - "" |
| dns.dnsConfig.searches | No | List of searches to use. | - "" |

### Persistence

A _PersistentVolume_ can be created for _Kellnr_ to hold all stored data.

| Setting             | Required         | Description                                                        | Default |
| ------------------- | ---------------- | ------------------------------------------------------------------ | ------- |
| pv.enabled          | No               | Enable a _PersistentVolume_ to store the data from _Kellnr_        | false   |
| pv.name             | No               | Name of the _PersistentVolume_                                     | kellnr  |
| pv.storageClassName | No               | storageClassName of the _PersistentVolume_                         | manual  |
| pv.storage          | No               | Size of the storage.                                               | 5Gi   |
| pv.path             | Yes (if enabled) | Host path to the storage. Needs to exists before the PV is created | ""      |

A _PersistentVolumeClaim_ can be used by _Kellnr_ to hold all stored data.

| Setting              | Required | Description                                                      | Default |
| -------------------- | -------- | ---------------------------------------------------------------- | ------- |
| pvc.enabled          | No       | Enable a _PersistentVolumeClaim_ to store the data from _Kellnr_ | false   |
| pvc.name             | No       | Name of the _PersistentVolumeClaim_                              | kellnr  |
| pvc.storageClassName | No       | storageClassName of the _PersistentVolumeClaim_                  | manual  |
| pvc.storage          | No       | Size of the storage.                                             | 5Gi   |

For a full set of possible variables see: [values.yaml](./charts/kellnr/values.yaml)

## Feedback

If the Helm Chart does not work for you for any reason or you need a feature, feel free to create a Github issue.

## Release a new version

Run the following steps to release a new version of the chart.

1. Change the `appVersion` in [Chart.yaml](./charts/kellnr/Chart.yaml) to the current image version.
2. Increase the `version` in [Chart.yaml](./charts/kellnr/Chart.yaml).
3. Push or create a Pull-Request to the `main`/`master` branch.
