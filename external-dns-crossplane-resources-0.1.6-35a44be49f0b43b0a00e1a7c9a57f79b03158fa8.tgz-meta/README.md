[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/external-dns-crossplane-resources/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/external-dns-crossplane-resources/tree/main)

# external-dns-crossplane-resources chart

This chart is used as a dependency in [cluster-aws](https://github.com/giantswarm/cluster-aws) to create AWS IAM roles for external-dns Route53 access on CAPA clusters via Crossplane.

It replaces the IAM role previously managed by `capa-iam-operator`.
