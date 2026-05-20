# Cluster chart

Cluster chart is a provider-independent Helm chart that renders Cluster API resources and HelmReleases of provider-independent apps.

Cluster chart contains these resources:
- Cluster,
- KubeadmControlPlane for the control plane,
- MachinePool and KubeadmConfig for the node pools,
- (deprecated) MachineDeployment and KubeadmConfigTemplate for bastions,
- HelmRepository resources for the following catalogs:
  - default-catalog,
  - default-test-catalog,
  - cluster-catalog,
- HelmRelease resources for the following provider-independent apps:
  - vertical-pod-autoscaler-crd,
  - coredns,
  - cilium, and
  - network-policies.

## Provider integration

This section describes how cluster chart can be used in a cluster-\<provider\> app.

### Few notes before we dig into it

The top level cluster chart Helm value that is used for configuring provider integration is `.Values.providerIntegration`.
When the cluster-\<provider\> app is setting this value, it is setting `.Values.cluster.providerIntegration` value in its
Helm values, because all Helm values that cluster-\<provider\> app (parent chart) is passing to cluster chart (subchart)
are set under  `.Values.cluster` object (which is just a regular Helm feature). This document is describing how a cluster-\<provider\>
app can integrate the cluster chart, so it will be using cluster-\<provider\> app’s `.Values.cluster.providerIntegration`
Helm value in explanations and examples.

One important detail to mention is that `.Values.cluster.providerIntegration` is supposed to be an immutable Helm value,
as it is an entry point that cluster-\<provider\> apps use to adapt cluster chart functionality to a specific-provider.

Therefore, when creating a cluster, values under `.Values.cluster.providerIntegration` should not be overridden in cluster-\<provider\>
app Helm values (this is currently not enforced in any technical way, but we will try to find a way to enforce it). Custom
cluster-specific values can be set under `.Values.global` and `.Values.internal.advancedConfiguration` (docs TBA for these
values).

### Configuring if and how the resources are rendered

While some resources rendered by cluster chart are fairly simple and not very much configurable, some are quite complex
and have an abundance of configuration options, so integrating cluster chart into a cluster-\<provider\> app can should
be done in stages, by starting to use cluster chart resources one by one  (more or less). The integration of cluster chart
should be done incrementally also because cluster chart is rendering CAPI resources slightly differently compared to the
existing cluster-\<provider\> apps (simply because the existing cluster-\<provider\> apps are doing it slightly differently,
when compared to each other), so you will want to make sure that the resources rendered by the cluster chart are working
correctly for your cluster-\<provider\> app.

Helm values which are controlling if the cluster chart will render some resource or not are all under `.Values.cluster.providerIntegration.resourcesApi`.
Some resources, when enabled, will also require you to set some other `.Values.cluster.providerIntegration` values for them
to be correctly rendered. The following sections will describe how to configure cluster chart resources with required and
optional Helm values.

> Note: for the sake of bevity the following text will use the term “resource flag” for referring to a Helm value under
> `.Values.cluster.providerIntegration.resourcesApi` that controls if some resource is rendered or not.

#### Cluster resource

Integrating Cluster resource is like warming up before the training, it should be easy (and it's not really telling you
how the whole training will look like 😜).

You can enable Cluster resource rendering by setting the `clusterResourceEnabled` resource flag.

Additionally, you also have to specify the group, version and kind of provider-specific cluster resource (e.g. AWSCluster,
AzureCluster, etc.) by specifying `.Values.cluster.providerIntegration.resourcesApi.infrastructureCluster` value.

When we put all of this in YAML, we get this:

```yaml
# cluster-<provider> values.yaml
cluster:
  providerIntegration:
    resourcesApi:
      clusterResourceEnabled: true
      infrastructureCluster:
        group: infrastructure.cluster.x-k8s.io
        kind: AWSCluster
        version: v1beta1
```

#### Control plane resources

There are two control plane resources, KubeadmControlPlane and MachineHealthCheck, and each has its own resource flag which
are `controlPlaneResourceEnabled` and `machineHealthCheckResourceEnabled`, respectively.

Besides the above two resource flags, there are few other values that have to be specified:
- Group, version and kind of the provider-specific machine template resources (e.g. AWSMachineTemplate, AzureMachineTemplate,
  etc.) which is done by setting `.Values.cluster.providerIntegration.controlPlane.resources.infrastructureMachineTemplate`,
- Name of the Helm template that renders the spec of the provider-specific machine template, which is done by setting
  `.Values.cluster.providerIntegration.controlPlane.resources.infrastructureMachineTemplateSpecTemplateName`. E.g. in case
  of AWS, this template renders `AWSMachineTemplate.spec.template.spec` value.

Here is the YAML:

```yaml
# cluster-<provider> values.yaml
cluster:
  providerIntegration:
    controlPlane:
      resources:
        infrastructureMachineTemplate:
          group: infrastructure.cluster.x-k8s.io
          kind: AWSMachineTemplate
          version: v1beta1
        infrastructureMachineTemplateSpecTemplateName: controlplane-awsmachinetemplate-spec
    resourcesApi:
      controlPlaneResourceEnabled: true
      machineHealthCheckResourceEnabled: true
```

There are more than few optional Helm values that you can set (optional from the point of view of cluster chart, but those
can be required for some cluster-\<provider\> app in order to correctly configure KubeadmControlPlane from the cluster
chart and successfully deploy a workload cluster).

You can customize some of the API server extra args that are set in `KubeadmControlPlane.spec.kubeadmConfigSpec.clusterConfiguration.apiServer`
by setting some of the following Helm values:

```yaml
# cluster-<provider> values.yaml
cluster:
  providerIntegration:
    controlPlane:
      kubeadmConfig:
        clusterConfiguration:
          apiServer:
            additionalAdmissionPlugins:
            - AlwaysPullImages
            apiAudiences:
              templateName: awsApiServerApiAudiences # name of the Helm template that renders api-audiences value
            featureGates:
            - name: StatefulSetAutoDeletePVC
              enabled: true
            serviceAccountIssuer:
              clusterDomainPrefix: irsa # which sets service-account-issuer to irsa.<cluster name>.<base domain>
```

You can specify provider-specific Ignition configuration (systemd units and storage). Here is an example from cluster-aws
which specifies filesystems and mount units for etcd, kubelet and containerd disks:

```yaml
# cluster-<provider> values.yaml
cluster:
  providerIntegration:
    controlPlane:
      kubeadmConfig:
        ignition:
          containerLinuxConfig:
            additionalConfig:
              storage:
                filesystems:
                  - mount:
                      device: /dev/xvdc
                      format: xfs
                      label: etcd
                      wipeFilesystem: true
                    name: etcd
                  - mount:
                      device: /dev/xvdd
                      format: xfs
                      label: containerd
                      wipeFilesystem: true
                    name: containerd
                  - mount:
                      device: /dev/xvde
                      format: xfs
                      label: kubelet
                      wipeFilesystem: true
                    name: kubelet
              systemd:
                units:
                  - contents:
                      install:
                        wantedBy:
                          - local-fs-pre.target
                      mount:
                        type: xfs
                        what: /dev/disk/by-label/etcd
                        where: /var/lib/etcd
                      unit:
                        defaultDependencies: false
                        description: etcd volume
                    enabled: true
                    name: var-lib-etcd.mount
                  - contents:
                      install:
                        wantedBy:
                          - local-fs-pre.target
                      mount:
                        type: xfs
                        what: /dev/disk/by-label/kubelet
                        where: /var/lib/kubelet
                      unit:
                        defaultDependencies: false
                        description: kubelet volume
                    enabled: true
                    name: var-lib-kubelet.mount
                  - contents:
                      install:
                        wantedBy:
                          - local-fs-pre.target
                      mount:
                        type: xfs
                        what: /dev/disk/by-label/containerd
                        where: /var/lib/containerd
                      unit:
                        defaultDependencies: false
                        description: containerd volume
                    enabled: true
                    name: var-lib-containerd.mount
```

#### Node pool resources

There are two node pool resources, MachinePool and KubeadmConfig, and both are rendered unders a single resource flag which
is `machinePoolResourcesEnabled` (which should have been called `nodePoolPoolResourcesEnabled` 🙈).

Besides the above two resource flags, there are few other values that have to be specified:
- Type of node pools that the cluster-\<provider\> app is using, which is set under `.Values.cluster.providerIntegration.resourcesApi.nodePoolKind `.
  Currently only `MachinePool` is supported, and `MachineDeployment` will be added later.
- Group, version and kind of the provider-specific machine pool resource (e.g. AWSMachinePool, AzureMachinePool, etc.) which
  is done by setting `.Values.cluster.providerIntegration.resourcesApi.infrastructureMachinePool`,
- Default node pools Helm values, which is used if no node pools are specified when creating a cluster. This is set by specifying
  `.Values.cluster.providerIntegration.workers.defaultNodePools`.

Here is the YAML that shows all of the above Helm values:

```yaml
# cluster-<provider> values.yaml
cluster:
  providerIntegration:
    resourcesApi:
      machinePoolResourcesEnabled: true
      nodePoolKind: MachinePool
      infrastructureMachinePool:
        group: infrastructure.cluster.x-k8s.io
        kind: AWSMachinePool
        version: v1beta1
    workers:
      defaultNodePools:
        def00:
          customNodeLabels:
            - label=default
          instanceType: r6i.xlarge
          maxSize: 3
          minSize: 3
```

#### HelmRelease and HelmRepository resources

Resource flags used for HelmRelease and HelmRepository resources are:
- `helmRepositoryResourcesEnabled` for default-catalog and default-test-catalog HelmRepositories.
- `ciliumHelmReleaseResourceEnabled` for cilium HelmRelease,
- `cleanupHelmReleaseResourcesEnabled` for cleanup HelmRelease,
- `coreDnsHelmReleaseResourceEnabled` for coredns HelmRelease,
- `verticalPodAutoscalerCrdHelmReleaseResourceEnabled` for vertical-pod-autoscaler-crd HelmRelease, and
- `networkPoliciesHelmReleaseResourceEnabled` for network-policies HelmRelease and cluster-catalog HelmRepository.

Written in YAML we get the following:
```yaml
# cluster-<provider> values.yaml
cluster:
  providerIntegration:
    resourcesApi:
      helmRepositoryResourcesEnabled: true
      ciliumHelmReleaseResourceEnabled: true
      cleanupHelmReleaseResourcesEnabled: true
      coreDnsHelmReleaseResourceEnabled: true
      verticalPodAutoscalerCrdHelmReleaseResourceEnabled: true
      networkPoliciesHelmReleaseResourceEnabled: true
```

Optionally, you can also specify the name of the Helm template that renders default provider-specific cilium Helm values
(these values will overwrite cilium’s provider-independent default Helm values that are in the cluster chart):

```yaml
# cluster-<provider> values.yaml
cluster:
  providerIntegration:
    apps:
      cilium:
        configTemplateName: awsCiliumHelmValues
```

At the moment only provider-specific default Helm values can be specified only for cilium (it can be easily added for other
apps as well if and when that is needed).

### Other kubeadm Helm values

You can configure multiple kubeadm-related Helm values:
- files that get deployed to nodes,
- ignition configuration (systemd units and storage config)
- preKubeadmCommands, and
- postKubeadmCommands.

These can be set for all nodes, or specifically for control plane or for workers.

Config for all nodes is set in `.Values.cluster.providerIntegration.kubeadmConfig`.

Config for control plane nodes only is set in `.Values.cluster.providerIntegration.controlPlane.kubeadmConfig`.

Config for worker nodes only is set in `.Values.cluster.providerIntegration.workers.kubeadmConfig`.

Here is an example for specifying files:

```yaml
# cluster-<provider> values.yaml
cluster:
  providerIntegration:
    kubeadmConfig:
      # files for both control plane and worker nodes
      files:
      - path: /etc/all/nodes/file.yaml
        permissions: "0644"
        contentFrom:
          secret:
            name: cluster-super-secret
            key: node-stuff
    controlPlane:
      kubeadmConfig:
        # files for control plane nodes
        files:
        - path: /etc/control-plane/node/file.yaml
          permissions: "0644"
          contentFrom:
            secret:
              name: cluster-super-secret-control-plane
              key: node-stuff
    workers:
      kubeadmConfig:
        # files for worker nodes
        files:
        - path: /etc/control-plane/node/file.yaml
          permissions: "0644"
          contentFrom:
            secret:
              name: cluster-super-secret-control-plane
              key: node-stuff
```

Here is an example for ignition config:

```yaml
# cluster-<provider> values.yaml
cluster:
  providerIntegration:
    kubeadmConfig:
      # ignition for both control plane and worker nodes
      ignition:
        containerLinuxConfig:
          additionalConfig:
            systemd:
              units:
              - name: var-lib-kubelet.mount
                enabled: true
                mask: false
                contents:
                  unit:
                    description: kubelet volume
                    defaultDependencies: false
                  install:
                    wantedBy:
                    - local-fs-pre.target
                  mount:
                    what: /dev/disk/by-label/kubelet
                    where: /var/lib/kubelet
                    type: xfs
            storage:
              directories:
              - path: /var/lib/stuff
                overwrite: true
                filesystem: stuff
                mode: 750
                user:
                  id: 12345
                  name: giantswarm
                group:
                  id: 23456
                  name: giantswarm
    controlPlane:
      kubeadmConfig:
        # ignition for control plane nodes
        ignition:
          containerLinuxConfig:
            additionalConfig:
              systemd:
                units:
                # same type of object as in the example above
              storage:
                # same type of object as in the example above
    workers:
      kubeadmConfig:
        # ignition for worker nodes
        ignition:
          containerLinuxConfig:
            additionalConfig:
              systemd:
                units:
                # same type of object as in the example above
              storage:
                # same type of object as in the example above
```

Here is an example of preKubeadmCommands and postKubeadmCommands:

```yaml
# cluster-<provider> values.yaml
cluster:
  providerIntegration:
    kubeadmConfig:
      # commands for both control plane and worker nodes
      preKubeadmCommands:
      - echo "command before kubeadm"
      postKubeadmCommands:
      - echo "command after kubeadm"
    controlPlane:
      kubeadmConfig:
        # commands for control plane nodes
        preKubeadmCommands:
        - echo "control plane command before kubeadm"
        postKubeadmCommands:
        - echo "control plane command after kubeadm"
    workers:
      kubeadmConfig:
        # commands for worker nodes
        preKubeadmCommands:
        - echo "workers command before kubeadm"
        postKubeadmCommands:
        - echo "workers command after kubeadm"
```

### Systemd unit templating

You can pass Helm templating syntax through from cluster-\<provider\> charts which will be rendered by the cluster chart. This is
written as plain text within the cluster-\<provider\> chart values under the `additionalFields` key. Consider the following:

```
global:
  connectivity:
    network:
      staticRoutes:
        - destination: 10.2.3.0/24
          via: 10.9.8.7
        - destination: 10.20.30.0/24
          via: 10.9.8.7
cluster:
  providerIntegration:
    kubeadmConfig:
      # ignition for both control plane and worker nodes
      ignition:
        containerLinuxConfig:
          additionalConfig:
            systemd:
              units:
                - contents:
                    install:
                      wantedBy:
                        - multi-user.target
                    service:
                      additionalFields: |-
                        {{- if $.global.connectivity.network.staticRoutes }}
                        {{- range $.global.connectivity.network.staticRoutes }}
                        ExecStart=/usr/bin/bash -cv 'ip route add {{ .destination }} via {{ .via }}'
                        {{- end }}
                        {{- end }}
                    unit:
                      requires:
                        - coreos-metadata.service
```

This results in the following unit:

```
[Unit]
Requires=coreos-metadata.service
[Service]
ExecStart=/usr/bin/bash -cv 'ip route add 10.2.3.0/24 via 10.9.8.7'
ExecStart=/usr/bin/bash -cv 'ip route add 10.20.30.0/24 via 10.9.8.7'
[Install]
WantedBy=multi-user.target
```

The Helm templating syntax is treated as plain text by the provider chart. The cluster chart's templating function has
access to the values under the provider chart's `.global` key so any values referenced in the template must exist
under `.global`.

Note that variable scoping is important here - the templating function does not have access to the root `$.Values` object,
so any variables under `.global` must be referenced as `$.global.some.var` (not `$.Values.global.some.var`).

## Workload cluster configuration

Workload clusters can be configured by setting Helm values in two top-level objects:
- `.Values.global`, which represents a public API for customizing workload clusters, and
- `.Values.cluster.internal.advancedConfiguration`, which should be used only by Giant Swarm staff for advanced cluster configuration.

P.S. `.Values.cluster.internal.advancedConfiguration` is the Helm value from the point of view of cluster-\<provider\> app,
while that value inside of cluster chart is `.Values.internal.advancedConfiguration`.

More details about customizing workload cluster Helm values TBA.


## Maintaining `values.schema.json` and `values.yaml`

**tldr**:
We only maintain `values.schema.json` and automatically generate `values.yaml` from it.
```
make normalize-schema
make validate-schema
make generate-values
make generate-docs
```

**Details**:

In order to provide a better UX we validate user values against `values.schema.json`.
In addition we also use the JSON schema in our frontend to dynamically generate a UI for cluster creation from it.
To succesfully do this, we have some requirements on the `values.schema.json`, which are defined in [this RFC](https://github.com/giantswarm/rfc/pull/55).
These requirements can be checked with [schemalint](https://github.com/giantswarm/schemalint).
`schemalint` does a couple of things:

- Normalize JSON schema (indentation, white space, sorting)
- Validate whether your schema is valid JSON schema
- Validate whether the requirements for cluster app schemas are met
- Check whether schema is normalized

The first point can be achieved with:
```
make normalize-schema
```
The second to fourth point can be achieved with:
```
make validate-schema
```

The JSON schema in `values.schema.json` should contain defaults defined with the `default` keyword.
These defaults should be same as those defined in `values.yaml`.
This allows us to generate `values.yaml` from `values.schema.json` with:

```
make generate-values
```
