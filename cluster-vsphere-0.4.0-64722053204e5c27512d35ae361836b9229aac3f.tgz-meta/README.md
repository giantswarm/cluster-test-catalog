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

You can see it in [cilium-helmrelease.yaml](helm/cluster-vsphere/templates/cilium-helmrelease.yaml).

This means cluster nodes won't come up Ready before this domain is set to the IP of the Kubernetes API server (it's defined in the `Cluster` CR under `.spec.controlPlaneEndpoint.host`). In Giant Swarm clusters we use [dns-operator-route53](https://github.com/giantswarm/dns-operator-route53) to create the records (public DNS resolution is then required).
