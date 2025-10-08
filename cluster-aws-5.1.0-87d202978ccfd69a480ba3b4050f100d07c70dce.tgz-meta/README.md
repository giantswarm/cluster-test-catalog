# cluster-aws

`cluster-aws` is an app that helps create a CRs for a Cluster API AWS cluster for Giant Swarm platform.

foo

## Configuration

See our [full list of configuration options](helm/cluster-aws/README.md).

## Custom Subnet Layouts

As of v0.21.0 it possible to specify more complex subnet layouts that allow using different sets of subnets for different grouping of machines.

Subnet groupings can be defined by setting `.connectivity.subnets`. For example, to have different subnets for control plane, worker and bastion nodes you might have something similar to the following:

```yaml
connectivity:
  availabilityZoneUsageLimit: 3
  network:
    vpcCidr: 10.0.0.0/16
  subnets:
  # Control plane nodes subnets
  - id: control-plane-nodes
    cidrBlocks:
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
  - id: worker-nodes
    cidrBlocks:
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
  - id: bastion-nodes
    cidrBlocks:
    - cidr: 10.0.0.0/24
      availabilityZone: a
    - cidr: 10.0.1.0/24
      availabilityZone: b
    - cidr: 10.0.2.0/24
      availabilityZone: c
    isPublic: true
    tags:
      subnet.giantswarm.io/role: bastion
  # Ingress load balancer subnets
  - id: load-balancer
    cidrBlocks:
    - cidr: 10.0.3.0/24
      availabilityZone: a
      tags:
        Name: cluster-ingress-lb-a
    - cidr: 10.0.4.0/24
      availabilityZone: b
      tags:
        Name: cluster-ingress-lb-b
    - cidr: 10.0.5.0/24
      availabilityZone: c
      tags:
        Name: cluster-ingress-lb-c
    isPublic: true
    tags:
      subnet.giantswarm.io/role: ingress
```

The desired subnet can then be targetted by using the `subnetTags` value to set the AWS tags to match on. For example:

```yaml
global:
  connectivity:
    bastion:
      subnetTags:
        - subnet.giantswarm.io/role: bastion

  controlPlane:
    subnetTags:
      - subnet.giantswarm.io/role: control-plane

  nodePools:
    def00:
      subnetTags:
        - subnet.giantswarm.io/role: workers
```

### API-server ELB subnets

Currently it's not possible to target the subnets used for the api-server ELB using the `subnetTags` approach like other resources. The ELB will be associated with the first grouping of subnets defined in the `network.subnets` list.

If you want the ELB to use its own subnets then you can add a new grouping **first** in the list specifically for the api-server ELB.

### VPC Endpoints subnets

You can have VPC Endpoints target specific subnets by using the `subnet.giantswarm.io/endpoints: "true"` on those subnets. E.g.

```yaml
# VPC endpoints
- cidrBlocks:
  - cidr: 10.233.19.0/24
    availabilityZone: a
  - cidr: 10.233.20.0/24
    availabilityZone: b
  - cidr: 10.233.21.0/24
    availabilityZone: c
  isPublic: true
  tags:
    subnet.giantswarm.io/role: attachments
    subnet.giantswarm.io/endpoints: "true"
```

If the `subnet.giantswarm.io/endpoints: "true"` tag isn't found on any subnets then it will default to using the first grouping of subnets.

### Transit Gateway Attachment subnets

> **Warning**
> Currently not possible, see [giantswarm/roadmap#1865](https://github.com/giantswarm/roadmap/issues/1865)

You can have Transit Gateway Attachments target specific subnets by using the `subnet.giantswarm.io/tgw-attachments: "true"` on those subnets. E.g.

```yaml
# TGW attachments
- cidrBlocks:
  - cidr: 10.233.19.0/24
    availabilityZone: a
  - cidr: 10.233.20.0/24
    availabilityZone: b
  - cidr: 10.233.21.0/24
    availabilityZone: c
  isPublic: true
  tags:
    subnet.giantswarm.io/role: attachments
    subnet.giantswarm.io/tgw-attachments: "true"
```

If the `subnet.giantswarm.io/tgw-attachments: "true"` tag isn't found on any subnets then it will default to using the first grouping of subnets.

### Ingress subnets

> **Warning**
> Currently not possible, see [giantswarm/roadmap#1866](https://github.com/giantswarm/roadmap/issues/1866)

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
