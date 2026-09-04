[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/network-policies-app/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/network-policies-app/tree/main)

[Read me after cloning this template (GS staff only)](https://handbook.giantswarm.io/docs/dev-and-releng/app-developer-processes/adding_app_to_appcatalog/)

# network-policies-app chart

Giant Swarm offers a network-policies-app App which can be installed in workload clusters.
Here we define the network-policies-app chart with its templates and default configuration.

**What is this app?**

**Why did we add it?**

**Who can use it?**

## Installing

There are several ways to install this app onto a workload cluster.

- [Using GitOps to instantiate the App](https://docs.giantswarm.io/advanced/gitops/apps/)
- [Using our web interface](https://docs.giantswarm.io/platform-overview/web-interface/app-platform/#installing-an-app).
- By creating an [App resource](https://docs.giantswarm.io/use-the-api/management-api/crd/apps.application.giantswarm.io/) in the management cluster as explained in [Getting started with App Platform](https://docs.giantswarm.io/getting-started/app-platform/).

## Configuring

### values.yaml

**This is an example of a values file you could upload using our web interface.**

```yaml
# values.yaml

```

### Sample App CR and ConfigMap for the management cluster

If you have access to the Kubernetes API on the management cluster, you could create
the App CR and ConfigMap directly.

Here is an example that would install the app to
workload cluster `abc12`:

```yaml
# appCR.yaml

```

```yaml
# user-values-configmap.yaml

```

See our [full reference on how to configure apps](https://docs.giantswarm.io/getting-started/app-platform/app-configuration/) for more details.

## Policies

### denyEgressToIMDS

Blocks pod egress to the cloud provider instance metadata service (IMDS,
`169.254.169.254`), so a compromised pod cannot use IMDS to assume the node's IAM role.
Disabled by default, opt in per cluster:

```yaml
denyEgressToIMDS:
  enabled: true
```

This renders a single cluster-scoped `CiliumClusterwideNetworkPolicy` using an
`egressDeny` rule. Deny rules take precedence over every allow rule in every other
policy, so a workload cannot re-open IMDS by shipping its own permissive egress policy.
The policy sets `enableDefaultDeny.egress: false`, so it only ever drops IMDS traffic:
it does not put any endpoint into egress default-deny mode, and no other traffic is
affected.

Only the `169.254.169.254/32` address is denied, never the wider `169.254.0.0/16`
link-local range, which also carries VPC DNS, NTP and Cilium's own addresses. `cidrs`
is a list, so dual-stack clusters can add the IPv6 address `fd00:ec2::254/128`.

#### Exempting workloads that need IMDS

`kube-system` and `giantswarm` are always excluded and cannot be removed through the
values. `kube-system` must stay excluded: `ebs-csi-node` and `aws-pod-identity-webhook`
both authenticate through IMDS using the node role, and the latter needs it to resolve
short-form `role-arn` annotations for every app relying on IRSA.
`aws-cloud-controller-manager` needs no exemption because it runs with
`hostNetwork: true`.

Because a deny cannot be undone by an allow, further exemptions are expressed as
exclusions in the policy's own endpoint selector, at namespace granularity. Additional
namespaces are added to the two above, never replacing them:

```yaml
denyEgressToIMDS:
  enabled: true
  excludedNamespaces:
  - my-namespace
```

Or by namespace label, which lets a namespace be exempted without changing the app
configuration afterwards:

```yaml
denyEgressToIMDS:
  enabled: true
  excludedNamespaceLabels:
  - key: policy.giantswarm.io/allow-imds
    values:
    - "true"
```

Every namespace labelled `policy.giantswarm.io/allow-imds: "true"` is then exempt. Note
that anyone who can label a namespace can grant that exemption, and that relabelling a
namespace recalculates the Cilium identity of every endpoint in it.

The same policy works on any provider whose metadata service answers on
`169.254.169.254`, which includes AWS, Azure and GCP. Only the set of platform
namespaces or workloads needing an exemption differs.

## Compatibility

This app has been tested to work with the following workload cluster release versions:

- _add release version_

## Limitations

Some apps have restrictions on how they can be deployed.
Not following these limitations will most likely result in a broken deployment.

- `denyEgressToIMDS` does not affect pods running with `hostNetwork: true`. Those pods
  share the node's network namespace and carry Cilium's `host` identity, so they are
  never selected by a pod endpoint selector and keep IMDS access. Restricting them is a
  job for the Pod Security Standards and Kyverno policies, not for this app.
- `denyEgressToIMDS` exemptions are per namespace, not per workload. Every pod in an
  excluded namespace keeps IMDS access.
- `denyEgressToIMDS` defaults to IPv4 only. On a dual-stack cluster, add the IPv6
  metadata address to `cidrs`.
- Enabling `denyEgressToIMDS` breaks workloads that legitimately use node role
  credentials through IMDS. That is the point of the policy; those namespaces have to be
  excluded explicitly.

## Credit

- {APP HELM REPOSITORY}
