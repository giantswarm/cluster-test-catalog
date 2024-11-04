# cluster-vsphere

This repository contains the Helm chart used for deploying CAPV clusters using CAPI. The chart only contains the CAPV-specific components; the core CAPI compnents are deployed via the shared [giantswarm/cluster](https://github.com/giantswarm/cluster) chart.

## IP address management (IPAM) for workload clusters

When creating a new vSphere cluster, a user can put an empty string to `.connectivity.network.controlPlaneEndpoint.host` and at the same time specify the `.connectivity.network.controlPlaneEndpoint.ipPoolName`. In this case, the cluster will be created in the `paused: true` state and a post-install job will be spawned.
The goal of this job is to acquire the new IP address and assign it to `.spec.controlPlaneEndpoint.host` of newly created clusters and other places where it's needed (`kube-vip` static pod definition and `certSANs`). Only then the cluster is unpaused.

The abovementioned mechanism relies on `IpAddressClaim` and `IpAddress` CRDs. These are part of the Cluster API spec and can be reconciled for instance by [`cluster-api-ipam-provider-in-cluster`](https://github.com/kubernetes-sigs/cluster-api-ipam-provider-in-cluster). So if you want to use it, make sure this [app](https://github.com/giantswarm/cluster-api-ipam-provider-in-cluster-app) is also installed in the management cluster.
