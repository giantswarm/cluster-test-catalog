[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/cluster-aks/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/cluster-aks/tree/main)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/giantswarm/cluster-aks/badge)](https://securityscorecards.dev/viewer/?uri=github.com/giantswarm/cluster-aks)

[Guide about how to manage an app on Giant Swarm](https://handbook.giantswarm.io/docs/dev-and-releng/app-developer-processes/adding_app_to_appcatalog/)

# cluster-aks chart

Giant Swarm offers a cluster-aks App which can be installed in workload clusters.
Here, we define the cluster-aks chart with its templates and default configuration.

**What is this app?**

**Why did we add it?**

**Who can use it?**

## Installing

There are several ways to install this app onto a workload cluster.

- [Using GitOps to instantiate the App](https://docs.giantswarm.io/tutorials/continuous-deployment/apps/add-appcr/)
- By creating an [App resource](https://docs.giantswarm.io/reference/platform-api/crd/apps.application.giantswarm.io) using the platform API as explained in [Getting started with App Platform](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/).

## Configuring

### values.yaml

**This is an example of a values file you could upload using our web interface.**

```yaml
# values.yaml

```

### Sample App CR and ConfigMap for the management cluster

If you have access to the Kubernetes API on the management cluster, you could create the App CR and ConfigMap directly.

Here is an example that would install the app to workload cluster `abc12`:

```yaml
# appCR.yaml

```

```yaml
# user-values-configmap.yaml

```

See our [full reference on how to configure apps](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/app-configuration/) for more details.

## BYO CNI (opt-in)

> [!WARNING]
> Not supported yet. Use it on test clusters only until the mode is declared supported.

By default, clusters use AKS-managed networking (Azure CNI powered by Cilium), and Microsoft supports the pod network. A new cluster can instead opt into BYO CNI:

```yaml
global:
  connectivity:
    cni:
      managed: true
    network:
      pods:
        cidrBlocks:
          - 192.168.0.0/16
```

AKS then installs no CNI (`networkPlugin: none`, with `networkDataplane`, `networkMode` and `networkPolicy` ignored), and the Release installs Giant Swarm's Cilium with the AKS BYO CNI values: VXLAN tunnelling, cluster-pool IPAM from the pod CIDR, no kube-proxy replacement and socket LB for the host namespace. Apps that need the pod network wait for Cilium. Cilium is tuned per cluster through `global.apps.cilium.values`, as on other providers.

Constraints:

- New clusters only. AKS can't change the network plugin of a running cluster.
- Pod IPs are on an overlay. Pods aren't reachable from the VNet, peered or on-premises networks except through Services, and their egress is SNATed to the node IP.
- Exactly one pod CIDR block, which must not overlap the VNet, peered or on-premises networks, the service CIDR or the AKS reserved ranges.
- No Windows node pools and no API server VNet integration.
- kube-proxy stays (AKS-managed, iptables). Cilium features that need kube-proxy replacement, such as eBPF host routing, aren't available.
- Webhooks behind a default deny must allow ingress from the `host` and `remote-node` entities, because the API server reaches pods through the hostNetwork Konnectivity agent.
- Microsoft supports the control plane, Konnectivity and the node VMs. Giant Swarm supports the pod network, Cilium and network policies.

## Azure CNI Overlay and ACNS

On AKS-managed networking, a cluster can use Azure CNI Overlay and Advanced Container Networking Services (ACNS):

```yaml
global:
  connectivity:
    network:
      pods:
        cidrBlocks:
          - 100.64.0.0/16
  controlPlane:
    networking:
      networkPluginMode: overlay
      advancedNetworking:
        enabled: true
        observability:
          enabled: true
        security:
          enabled: true
          advancedNetworkPolicies: L7
```

Overlay gives pods IPs from the pod CIDR instead of the VNet. ACNS observability provides Hubble network metrics and flow logs, and ACNS security provides FQDN filtering (the default) or L7 network policies.

Constraints:

- Overlay is a creation-time choice. Existing clusters can't be migrated from Azure CNI node-subnet.
- Exactly one pod CIDR block, which must not overlap the VNet, peered or on-premises networks, or the service CIDR.
- The pod CIDR, the service CIDR and the VNet must not overlap the AKS reserved ranges `169.254.0.0/16`, `172.30.0.0/16`, `172.31.0.0/16`, `192.0.2.0/24` and `224.0.0.0/4`. AKS rejects the cluster otherwise.
- Pod egress is SNATed to the node IP and pod IPs aren't routable outside the cluster, so allowlists must use node or subnet IPs.
- Overlay and ACNS require `networkPlugin: azure`. Overlay doesn't support `networkMode: bridge`, and ACNS security requires `networkDataplane: cilium`.
- ACNS is a paid Azure add-on billed to the cluster's subscription. It can be turned on or off on an existing cluster. With `advancedNetworking.enabled` unset, the chart leaves the AKS setting untouched; set it to `false` to disable ACNS. Observability and security require `enabled: true`.
- Neither can be combined with the BYO CNI opt-in.

## User Defined Routing

To send the cluster's egress through your own firewall or NVA, use `userDefinedRouting` on a BYO subnet:

```yaml
global:
  connectivity:
    network:
      vnet:
        subnetArmId: /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Network/virtualNetworks/<vnet>/subnets/<subnet>
  controlPlane:
    networking:
      outboundType: userDefinedRouting
```

Constraints:

- Requires a BYO subnet. A VNet created by the chart has no route table, so rendering fails without `subnetArmId`.
- The subnet's route table must send `0.0.0.0/0` to your egress device. With per-pool subnets, each of them needs that route table too.
- Changing the outbound type of an existing cluster changes its egress IP and interrupts existing connections. Update firewall rules and authorized IP ranges accordingly.

## Compatibility

This app has been tested to work with the following workload cluster release versions:

- _add release version_

## Limitations

Some apps have restrictions on how they can be deployed.
Not following these limitations will most likely result in a broken deployment.

- _add limitation_

## Credit

- {APP HELM REPOSITORY}
