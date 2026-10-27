[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/cert-manager-crossplane-resources/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/cert-manager-crossplane-resources/tree/main)

# cert-manager-crossplane-resources

This chart creates the cloud IAM resources that [cert-manager](https://github.com/giantswarm/cert-manager-app) in a workload cluster needs to solve ACME DNS-01 challenges, using [Crossplane](https://docs.crossplane.io/) managed resources that run in the management cluster.
On AWS it creates an IAM Role, trusted by the cluster's OIDC provider, that may only change `TXT` records matching `_acme-challenge.*` in Route53; on Azure it creates a User Assigned Identity with a custom role limited to `TXT` records in DNS zones, and on AKS a `WorkloadIdentity` that also hands the identity's client ID to cert-manager.
One release is created per workload cluster, named after the cluster.

## Installing

The chart is installed automatically into the management cluster by the [`cluster` chart](https://github.com/giantswarm/cluster) as a Flux `HelmRelease` for every workload cluster, which sets `clusterName` and `provider` from the cluster values.
It is not intended to be installed by customers directly.
Crossplane and the matching provider must already be running in the management cluster, as they are on Giant Swarm installations.

## Configuring

Enable exactly one provider and fill in the account details the IAM resources should be created in.
The provider-specific values are documented in the [chart README](helm/cert-manager-crossplane-resources/README.md).

```yaml
# values.yaml
clusterName: abc12

providers:
  aws:
    enabled: true
    accountID: "123456789012"
    region: eu-west-1
    oidc:
      domain: irsa.abc12.example.com
    # Route53 record names cert-manager may change (IAM StringLike wildcards allowed)
    resourcePatterns: ["_acme-challenge.*"]
```

See our [full reference on how to configure apps](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/app-configuration/) for more details.

## Credit

- [Crossplane](https://github.com/crossplane/crossplane) with the [Upbound AWS](https://github.com/crossplane-contrib/provider-upjet-aws) and [Azure](https://github.com/crossplane-contrib/provider-upjet-azure) providers, whose resource kinds this chart templates.
- [cert-manager](https://cert-manager.io/) and its [Route53](https://cert-manager.io/docs/configuration/acme/dns01/route53/) and [AzureDNS](https://cert-manager.io/docs/configuration/acme/dns01/azuredns/) DNS-01 solver documentation, which define the required permissions.
