# cluster-vsphere

This repository contains the Helm chart used for deploying CAPV clusters using CAPI. The chart only contains the CAPV-specific components; the core CAPI compnents are deployed via the shared [giantswarm/cluster](https://github.com/giantswarm/cluster) chart.

## IP address management (IPAM) for workload clusters

When creating a new vSphere cluster, a user can put an empty string to `.connectivity.network.controlPlaneEndpoint.host` and at the same time specify the `.connectivity.network.controlPlaneEndpoint.ipPoolName`. In this case, the cluster will be created in the `paused: true` state and a post-install job will be spawned.
The goal of this job is to acquire the new IP address and assign it to `.spec.controlPlaneEndpoint.host` of newly created clusters and other places where it's needed (`kube-vip` static pod definition and `certSANs`). Only then the cluster is unpaused.

The abovementioned mechanism relies on `IpAddressClaim` and `IpAddress` CRDs. These are part of the Cluster API spec and can be reconciled for instance by [`cluster-api-ipam-provider-in-cluster`](https://github.com/kubernetes-sigs/cluster-api-ipam-provider-in-cluster). So if you want to use it, make sure this [app](https://github.com/giantswarm/cluster-api-ipam-provider-in-cluster-app) is also installed in the management cluster.

## Tagging VMs

It is possible to apply vSphere tags to the virtual machines deployed by this chart.

* First find the ID of the tags to apply.

```
Powershell:
get-tag -name <tag name> -category <tag category> | select -ExpandProperty id

govc:
govc tags.info -c <tag category> -json | jq -r ".[] | select(.name == \"<tag name>") | .id"
```

The ID should look like this: `urn:vmomi:InventoryServiceTag:a5242a12-87e4-4954-b357-d711c99c91e5:GLOBAL`

* Set the values where you want to apply the tag.

```yaml
    global:
      controlPlane:
        machineTemplate:
          tagIDs:
          - "urn:vmomi:InventoryServiceTag:827c9b5a-e6e5-4c3f-8349-29c083395a7f:GLOBAL"
      nodePools:
        <nodepool name>:
          tagIDs:
          - "urn:vmomi:InventoryServiceTag:827c9b5a-e6e5-4c3f-8349-29c083395a7f:GLOBAL"
```

## Using a VM template per location

When there are clusters in different geographical locations under the same `datacenter` entity, it is recommended to use a VM template stored in that cluster to avoid fetching cloning from a template over a WAN connection.

In this case:

- Set e.g. `global.providerSpecific.templateSuffix=paris`
- Upload the VM image template to the Paris cluster and rename it to add the `-paris` suffix.
