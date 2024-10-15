# cluster-vsphere

This repository contains the Helm chart used for deploying CAPI clusters via CAPV. It deploys:

- CAPI resources
- `cilium` as CNI in `kube-proxy` replacement mode (see [Limitations](#Limitations) section below)
- CPI and CSI for vSphere

## Limitations

With this chart we deploy `cilium` CNI to the cluster in `kube-proxy` replacement mode. This requires us to specify `k8sServiceHost` value in the `cilium` chart which we chose to be a templated domain name:

```yaml
k8sServiceHost: api.{{ include "resource.default.name" $ }}.{{ .Values.baseDomain }}
```

You can see it in [cilium-helmrelease.yaml](helm/cluster-vsphere/templates/helmreleases/cilium-helmrelease.yaml).

This means cluster nodes won't come up Ready before this domain is set to the IP of the Kubernetes API server (it's defined in the `Cluster` CR under `.spec.controlPlaneEndpoint.host`). In Giant Swarm clusters we use [dns-operator-route53](https://github.com/giantswarm/dns-operator-route53) to create the records (public DNS resolution is then required).

## IP address management (IPAM) for workload clusters

When creating a new vsphere cluster, a user can put an empty string to `.connectivity.network.controlPlaneEndpoint.host` and at the same time specify the `.connectivity.network.controlPlaneEndpoint.ipPoolName`. In this case, the cluster will be created in the `paused: true` state and post-install job will be spawned.
The goal of this job is to acquire the new IP address and assign it to `.spec.controlPlaneEndpoint.host` of newly created clusters and other places where it's needed (kubevip static pod definition and `certSANs`). Only then the cluster is unpaused.

The abovementioned mechanism relies on `IpAddressClaim` and `IpAddress` CRDs. These are part of the Cluster API spec and can be reconciled for instance by [`cluster-api-ipam-provider-in-cluster`](https://github.com/kubernetes-sigs/cluster-api-ipam-provider-in-cluster). So if you want to use it, make sure this [app](https://github.com/giantswarm/cluster-api-ipam-provider-in-cluster-app) is also installed in the management cluster.

DELETE ME