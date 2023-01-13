# cluster-aws

`cluster-aws` is an app that helps create a CRs for a Cluster API AWS cluster for Giant Swarm platform.

## Custom Subnet Layouts

As of v0.21.0 it possible to specify more complex subnet layouts that allow using different sets of subnets for different grouping of machines.

Subnet groupings can be defined by setting `.network.subnets`. For example, to have different subnets for control plane, worker and bastion nodes you might have something similar to the following:

```yaml

network:
  availabilityZoneUsageLimit: 3
  vpcCIDR: 10.0.0.0/16
  subnets:
  # Control plane nodes subnets
  - cidrBlocks:
    - cidr: 10.0.32.0/19
      availabilityZone: a
    - cidr: 10.0.64.0/19
      availabilityZone: b
    - cidr: 10.0.96.0/19
      availabilityZone: c
    isPublic: false
    tags:
      subnet.giantswarm.io/role: control-plane
  # Worker nodes subnets
  - cidrBlocks:
    - cidr: 10.0.128.0/19
      availabilityZone: a
    - cidr: 10.0.160.0/19
      availabilityZone: b
    - cidr: 10.0.192.0/19
      availabilityZone: c
    isPublic: false
    tags:
      subnet.giantswarm.io/role: workers
  # Bastion nodes subnets
  - cidrBlocks:
    - cidr: 10.0.0.0/24
      availabilityZone: a
    - cidr: 10.0.1.0/24
      availabilityZone: b
    - cidr: 10.0.2.0/24
      availabilityZone: c
    isPublic: true
    tags:
      subnet.giantswarm.io/role: bastion
```

The desired subnet can then be targetted by using the `subnetTags` value to set the AWS tags to match on. For example:

```yaml

bastion:
  subnetTags:
    - subnet.giantswarm.io/role: bastion

controlPlane:
  subnetTags:
    - subnet.giantswarm.io/role: control-plane

machinePools:
- name: def00
  subnetTags:
    - subnet.giantswarm.io/role: workers
```

## Upgrading to `v0.21.0`

If your cluster previously has `network.vpcMode` set to private you will need to make a small change to your values when upgrading to this version.

If using the default list of subnets you will need to set the following in your values:

```yaml
network:
  subnets:
  - cidrBlocks:
    - cidr: 10.0.0.0/18
      availabilityZone: a
    - cidr: 10.0.64.0/18
      availabilityZone: b
    - cidr: 10.0.128.0/18
      availabilityZone: c
    isPublic: false
```

If you've specified your own CIDR blocks previous you'll need to convert those strings to the block structure like above. Be aware to make sure the correct availability zone is specified for each CIDR block.
