# cluster-aws

`cluster-aws` is an app that helps create a CRs for a Cluster API AWS cluster for Giant Swarm platform.

## Custom Subnet Layouts

As of v0.21.0 it possible to specify more complex subnet layouts that allow using different sets of subnets for different grouping of machines.

Subnet groupings can be defined by setting `.network.subnets`. For example, to have different subnets for control plane, worker and bastion nodes you might have something similar to the following:

```yaml
connectivity:
  availabilityZoneUsageLimit: 3
  network:
    vpcCidr: 10.0.0.0/16
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
  # Ingress load balancer subnets
  - cidrBlocks:
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

bastion:
  subnetTags:
    - subnet.giantswarm.io/role: bastion

controlPlane:
  subnetTags:
    - subnet.giantswarm.io/role: control-plane

machinePools:
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

## Upgrade Migrations

### Upgrading to `v0.21.0`

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

### Upgrading to `v0.24.0`

You will need to change the definition of your machine pools from using a list to an object.
For example, instead of the following:

```yaml
machinePools:
- name: def00  # Name of node pool.
  availabilityZones: []
  instanceType: m5.xlarge
  minSize: 3  # Number of replicas in node pool.
  maxSize: 3
  rootVolumeSizeGB: 300
  customNodeLabels:
  - label=default
  customNodeTaints: []
```

You should have:

```yaml
machinePools:
  def00:  # Name of node pool.
    availabilityZones: []
    instanceType: m5.xlarge
    minSize: 3  # Number of replicas in node pool.
    maxSize: 3
    rootVolumeSizeGB: 300
    customNodeLabels:
    - label=default
    customNodeTaints: []
```


## Maintaining `values.schema.json` and `values.yaml`

**tldr**:  
We only maintain `values.schema.json` and automatically generate `values.yaml` from it.
```
make normalize-schema
make validate-schema
make generate-values
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

## Full values documentation

<!-- DOCS_START -->

### AWS settings
Properties within the `.providerSpecific` top-level object

| **Property** | **Description** | **More Details** |
| :----------- | :-------------- | :--------------- |
| `providerSpecific.ami` | **Amazon machine image (AMI)** - If specified, this image will be used to provision EC2 instances.|**Type:** `string`<br/>|
| `providerSpecific.awsClusterRoleIdentityName` | **Cluster role identity name** - Name of an AWSClusterRoleIdentity object. This in turn refers to the IAM role used to create all AWS cloud resources when creating the cluster. The role can be in another AWS account in order to create all resources in that account. Note: This name does not refer directly to an IAM role name/ARN.|**Type:** `string`<br/>**Value pattern:** `^[-a-zA-Z0-9_\.]{1,63}$`<br/>**Default:** `"default"`|
| `providerSpecific.flatcarAwsAccount` | **AWS account owning Flatcar image** - AWS account ID owning the Flatcar Container Linux AMI.|**Type:** `string`<br/>**Default:** `"075585003325"`|
| `providerSpecific.region` | **Region**|**Type:** `string`<br/>|

### Connectivity
Properties within the `.connectivity` top-level object

| **Property** | **Description** | **More Details** |
| :----------- | :-------------- | :--------------- |
| `connectivity.availabilityZoneUsageLimit` | **Availability zones** - Maximum number of availability zones (AZ) that should be used in a region. If a region has more than this number of AZs then this number of AZs will be picked randomly when creating subnets.|**Type:** `integer`<br/>**Default:** `3`|
| `connectivity.bastion` | **Bastion host**|**Type:** `object`<br/>|
| `connectivity.bastion.enabled` | **Enable**|**Type:** `boolean`<br/>**Default:** `true`|
| `connectivity.bastion.instanceType` | **EC2 instance type**|**Type:** `string`<br/>**Default:** `"t3.small"`|
| `connectivity.bastion.replicas` | **Number of hosts**|**Type:** `integer`<br/>**Default:** `1`|
| `connectivity.bastion.subnetTags` | **Subnet tags** - Tags to filter which AWS subnets will be used for the bastion hosts.|**Type:** `array`<br/>|
| `connectivity.bastion.subnetTags[*]` | **Subnet tag**|**Type:** `object`<br/>|
| `connectivity.bastion.subnetTags[*].*` | **Tag value**|**Type:** `string`<br/>**Value pattern:** `^[ a-zA-Z0-9\._:/=+-@]+$`<br/>|
| `connectivity.containerRegistries` | **Container registries** - Endpoints and credentials configuration for container registries.|**Type:** `object`<br/>**Default:** `{"docker.io":[{"endpoint":"registry-1.docker.io"},{"endpoint":"giantswarm.azurecr.io"}]}`|
| `connectivity.containerRegistries.*` | **Registries** - Container registries and mirrors|**Type:** `array`<br/>|
| `connectivity.containerRegistries.*[*]` | **Registry**|**Type:** `object`<br/>|
| `connectivity.containerRegistries.*[*].credentials` | **Credentials**|**Type:** `object`<br/>|
| `connectivity.containerRegistries.*[*].credentials.auth` | **Auth** - Base64-encoded string from the concatenation of the username, a colon, and the password.|**Type:** `string`<br/>|
| `connectivity.containerRegistries.*[*].credentials.identitytoken` | **Identity token** - Used to authenticate the user and obtain an access token for the registry.|**Type:** `string`<br/>|
| `connectivity.containerRegistries.*[*].credentials.password` | **Password** - Used to authenticate for the registry with username/password.|**Type:** `string`<br/>|
| `connectivity.containerRegistries.*[*].credentials.username` | **Username** - Used to authenticate for the registry with username/password.|**Type:** `string`<br/>|
| `connectivity.containerRegistries.*[*].endpoint` | **Endpoint** - Endpoint for the container registry.|**Type:** `string`<br/>|
| `connectivity.dns` | **DNS**|**Type:** `object`<br/>|
| `connectivity.dns.additionalVpc` | **Additional VPCs** - If DNS mode is 'private', the VPCs specified here will be assigned to the private hosted zone.|**Type:** `array`<br/>|
| `connectivity.dns.additionalVpc[*]` | **VPC identifier**|**Type:** `string`<br/>**Example:** `"vpc-x2aeasd1d"`<br/>**Value pattern:** `^vpc-[0-0a-zA-Z]+$`<br/>|
| `connectivity.dns.mode` | **Mode** - Whether the Route53 hosted zone of this cluster should be public or private.|**Type:** `string`<br/>**Default:** `"public"`|
| `connectivity.dns.resolverRulesOwnerAccount` | **Resolver rules owner** - ID of the AWS account that created the resolver rules to be associated with the workload cluster VPC.|**Type:** `string`<br/>|
| `connectivity.network` | **Network**|**Type:** `object`<br/>|
| `connectivity.network.podCidr` | **Pod subnet** - IPv4 address range for pods, in CIDR notation.|**Type:** `string`<br/>**Default:** `"100.64.0.0/12"`|
| `connectivity.network.serviceCidr` | **Service subnet** - IPv4 address range for services, in CIDR notation.|**Type:** `string`<br/>**Default:** `"172.31.0.0/16"`|
| `connectivity.network.vpcCidr` | **VPC subnet** - IPv4 address range to assign to this cluster's VPC, in CIDR notation.|**Type:** `string`<br/>**Default:** `"10.0.0.0/16"`|
| `connectivity.proxy` | **Proxy** - Whether/how outgoing traffic is routed through proxy servers.|**Type:** `object`<br/>|
| `connectivity.proxy.enabled` | **Enable**|**Type:** `boolean`<br/>|
| `connectivity.proxy.httpProxy` | **HTTP proxy** - To be passed to the HTTP_PROXY environment variable in all hosts.|**Type:** `string`<br/>|
| `connectivity.proxy.httpsProxy` | **HTTPS proxy** - To be passed to the HTTPS_PROXY environment variable in all hosts.|**Type:** `string`<br/>|
| `connectivity.proxy.noProxy` | **No proxy** - To be passed to the NO_PROXY environment variable in all hosts.|**Type:** `string`<br/>|
| `connectivity.sshSsoPublicKey` | **SSH public key for single sign-on**|**Type:** `string`<br/>**Default:** `"ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIM4cvZ01fLmO9cJbWUj7sfF+NhECgy+Cl0bazSrZX7sU vault-ca@vault.operations.giantswarm.io"`|
| `connectivity.subnets` | **Subnets** - Subnets are created and tagged based on this definition.|**Type:** `array`<br/>**Default:** `[{"cidrBlocks":[{"availabilityZone":"a","cidr":"10.0.0.0/20"},{"availabilityZone":"b","cidr":"10.0.16.0/20"},{"availabilityZone":"c","cidr":"10.0.32.0/20"}],"isPublic":true},{"cidrBlocks":[{"availabilityZone":"a","cidr":"10.0.64.0/18"},{"availabilityZone":"b","cidr":"10.0.128.0/18"},{"availabilityZone":"c","cidr":"10.0.192.0/18"}],"isPublic":false}]`|
| `connectivity.subnets[*]` | **Subnet**|**Type:** `object`<br/>|
| `connectivity.subnets[*].cidrBlocks` | **Network**|**Type:** `array`<br/>|
| `connectivity.subnets[*].cidrBlocks[*]` |**None**|**Type:** `object`<br/>|
| `connectivity.subnets[*].cidrBlocks[*].availabilityZone` | **Availability zone**|**Type:** `string`<br/>**Example:** `"a"`<br/>|
| `connectivity.subnets[*].cidrBlocks[*].cidr` | **Address range** - IPv4 address range, in CIDR notation.|**Type:** `string`<br/>|
| `connectivity.subnets[*].cidrBlocks[*].tags` | **Tags** - AWS resource tags to assign to this subnet.|**Type:** `object`<br/>|
| `connectivity.subnets[*].cidrBlocks[*].tags.*` | **Tag value**|**Type:** `string`<br/>**Value pattern:** `^[ a-zA-Z0-9\._:/=+-@]+$`<br/>|
| `connectivity.subnets[*].isPublic` | **Public**|**Type:** `boolean`<br/>|
| `connectivity.subnets[*].tags` | **Tags** - AWS resource tags to assign to this CIDR block.|**Type:** `object`<br/>|
| `connectivity.subnets[*].tags.*` | **Tag value**|**Type:** `string`<br/>**Value pattern:** `^[ a-zA-Z0-9\._:/=+-@]+$`<br/>|
| `connectivity.topology` | **Topology** - Networking architecture between management cluster and workload cluster.|**Type:** `object`<br/>|
| `connectivity.topology.mode` | **Mode** - Valid values: GiantSwarmManaged, UserManaged, None.|**Type:** `string`<br/>**Default:** `"None"`|
| `connectivity.topology.prefixListId` | **Prefix list ID** - ID of the managed prefix list to use when the topology mode is set to 'UserManaged'.|**Type:** `string`<br/>|
| `connectivity.topology.transitGatewayId` | **Transit gateway ID** - If the topology mode is set to 'UserManaged', this can be used to specify the transit gateway to use.|**Type:** `string`<br/>|
| `connectivity.vpcEndpointMode` | **VPC endpoint mode** - Who is reponsible for creation and management of VPC endpoints.|**Type:** `string`<br/>**Default:** `"GiantSwarmManaged"`|
| `connectivity.vpcMode` | **VPC mode** - Whether the cluser's VPC is created with public, internet facing resources (public subnets, NAT gateway) or not (private).|**Type:** `string`<br/>**Default:** `"public"`|

### Control plane
Properties within the `.controlPlane` top-level object

| **Property** | **Description** | **More Details** |
| :----------- | :-------------- | :--------------- |
| `controlPlane.apiMode` | **API mode** - Whether the Kubernetes API server load balancer should be reachable from the internet (public) or internal only (private).|**Type:** `string`<br/>**Default:** `"public"`|
| `controlPlane.containerdVolumeSizeGB` | **Containerd volume size (GB)**|**Type:** `integer`<br/>**Default:** `100`|
| `controlPlane.etcdVolumeSizeGB` | **Etcd volume size (GB)**|**Type:** `integer`<br/>**Default:** `100`|
| `controlPlane.instanceType` | **EC2 instance type**|**Type:** `string`<br/>**Default:** `"m5.xlarge"`|
| `controlPlane.kubeletVolumeSizeGB` | **Kubelet volume size (GB)**|**Type:** `integer`<br/>**Default:** `100`|
| `controlPlane.machineHealthCheck` | **Machine health check**|**Type:** `object`<br/>|
| `controlPlane.machineHealthCheck.enabled` | **Enable**|**Type:** `boolean`<br/>**Default:** `true`|
| `controlPlane.machineHealthCheck.maxUnhealthy` | **Maximum unhealthy nodes**|**Type:** `string`<br/>**Example:** `"40%"`<br/>**Default:** `"40%"`|
| `controlPlane.machineHealthCheck.nodeStartupTimeout` | **Node startup timeout** - Determines how long a machine health check should wait for a node to join the cluster, before considering a machine unhealthy.|**Type:** `string`<br/>**Examples:** `"10m", "100s"`<br/>**Default:** `"8m0s"`|
| `controlPlane.machineHealthCheck.unhealthyNotReadyTimeout` | **Timeout for ready** - If a node is not in condition 'Ready' after this timeout, it will be considered unhealthy.|**Type:** `string`<br/>**Example:** `"300s"`<br/>**Default:** `"10m0s"`|
| `controlPlane.machineHealthCheck.unhealthyUnknownTimeout` | **Timeout for unknown condition** - If a node is in 'Unknown' condition after this timeout, it will be considered unhealthy.|**Type:** `string`<br/>**Example:** `"300s"`<br/>**Default:** `"10m0s"`|
| `controlPlane.oidc` | **OIDC authentication**|**Type:** `object`<br/>|
| `controlPlane.oidc.caPem` | **Certificate authority** - Identity provider's CA certificate in PEM format.|**Type:** `string`<br/>|
| `controlPlane.oidc.clientId` | **Client ID**|**Type:** `string`<br/>|
| `controlPlane.oidc.groupsClaim` | **Groups claim**|**Type:** `string`<br/>|
| `controlPlane.oidc.issuerUrl` | **Issuer URL** - Exact issuer URL that will be included in identity tokens.|**Type:** `string`<br/>|
| `controlPlane.oidc.usernameClaim` | **Username claim**|**Type:** `string`<br/>|
| `controlPlane.rootVolumeSizeGB` | **Root volume size (GB)**|**Type:** `integer`<br/>**Default:** `120`|
| `controlPlane.subnetTags` | **Subnet tags** - Tags to select AWS resources for the control plane by.|**Type:** `array`<br/>|
| `controlPlane.subnetTags[*]` | **Subnet tag**|**Type:** `object`<br/>|
| `controlPlane.subnetTags[*].*` | **Tag value**|**Type:** `string`<br/>**Value pattern:** `^[ a-zA-Z0-9\._:/=+-@]+$`<br/>|

### Internal
Properties within the `.internal` top-level object
For Giant Swarm internal use only, not stable, or not supported by UIs.

| **Property** | **Description** | **More Details** |
| :----------- | :-------------- | :--------------- |
| `internal.hashSalt` | **Hash salt** - If specified, this token is used as a salt to the hash suffix of some resource names. Can be used to force-recreate some resources.|**Type:** `string`<br/>|
| `internal.kubernetesVersion` | **Kubernetes version**|**Type:** `string`<br/>**Example:** `"1.24.7"`<br/>**Default:** `"1.24.10"`|
| `internal.nodePools` | **Default node pool**|**Type:** `object`<br/>**Default:** `{"def00":{"customNodeLabels":["label=default"],"instanceType":"m5.xlarge","minSize":3}}`|
| `internal.nodePools.PATTERN` | **Node pool**|**Type:** `object`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `internal.nodePools.PATTERN.availabilityZones` | **Availability zones**|**Type:** `array`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `internal.nodePools.PATTERN.availabilityZones[*]` | **Availability zone**|**Type:** `string`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `internal.nodePools.PATTERN.customNodeLabels` | **Custom node labels**|**Type:** `array`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `internal.nodePools.PATTERN.customNodeLabels[*]` | **Label**|**Type:** `string`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `internal.nodePools.PATTERN.customNodeTaints` | **Custom node taints**|**Type:** `array`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `internal.nodePools.PATTERN.customNodeTaints[*]` |**None**|**Type:** `object`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `internal.nodePools.PATTERN.customNodeTaints[*].effect` | **Effect**|**Type:** `string`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `internal.nodePools.PATTERN.customNodeTaints[*].key` | **Key**|**Type:** `string`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `internal.nodePools.PATTERN.customNodeTaints[*].value` | **Value**|**Type:** `string`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `internal.nodePools.PATTERN.instanceType` | **EC2 instance type**|**Type:** `string`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `internal.nodePools.PATTERN.maxSize` | **Maximum number of nodes**|**Type:** `integer`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `internal.nodePools.PATTERN.minSize` | **Minimum number of nodes**|**Type:** `integer`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `internal.nodePools.PATTERN.rootVolumeSizeGB` | **Root volume size (GB)**|**Type:** `integer`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `internal.nodePools.PATTERN.subnetTags` | **Subnet tags** - Tags to filter which AWS subnets will be used for this node pool.|**Type:** `array`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `internal.nodePools.PATTERN.subnetTags[*]` | **Subnet tag**|**Type:** `object`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `internal.nodePools.PATTERN.subnetTags[*].*` | **Tag value**|**Type:** `string`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>**Value pattern:** `^[ a-zA-Z0-9\._:/=+-@]+$`<br/>|

### Kubectl image
Properties within the `.kubectlImage` top-level object

| **Property** | **Description** | **More Details** |
| :----------- | :-------------- | :--------------- |
| `kubectlImage.name` | **Repository**|**Type:** `string`<br/>**Default:** `"giantswarm/kubectl"`|
| `kubectlImage.registry` | **Registry**|**Type:** `string`<br/>**Default:** `"quay.io"`|
| `kubectlImage.tag` | **Tag**|**Type:** `string`<br/>**Default:** `"1.23.5"`|

### Metadata
Properties within the `.metadata` top-level object

| **Property** | **Description** | **More Details** |
| :----------- | :-------------- | :--------------- |
| `metadata.description` | **Cluster description** - User-friendly description of the cluster's purpose.|**Type:** `string`<br/>|
| `metadata.name` | **Cluster name** - Unique identifier, cannot be changed after creation.|**Type:** `string`<br/>|
| `metadata.organization` | **Organization**|**Type:** `string`<br/>|

### Node pools
Properties within the `.nodePools` top-level object
Node pools of the cluster. If not specified, this defaults to the value of `internal.nodePools`.

| **Property** | **Description** | **More Details** |
| :----------- | :-------------- | :--------------- |
| `nodePools.PATTERN` | **Node pool**|**Type:** `object`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `nodePools.PATTERN.availabilityZones` | **Availability zones**|**Type:** `array`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `nodePools.PATTERN.availabilityZones[*]` | **Availability zone**|**Type:** `string`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `nodePools.PATTERN.customNodeLabels` | **Custom node labels**|**Type:** `array`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `nodePools.PATTERN.customNodeLabels[*]` | **Label**|**Type:** `string`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `nodePools.PATTERN.customNodeTaints` | **Custom node taints**|**Type:** `array`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `nodePools.PATTERN.customNodeTaints[*]` |**None**|**Type:** `object`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `nodePools.PATTERN.customNodeTaints[*].effect` | **Effect**|**Type:** `string`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `nodePools.PATTERN.customNodeTaints[*].key` | **Key**|**Type:** `string`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `nodePools.PATTERN.customNodeTaints[*].value` | **Value**|**Type:** `string`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `nodePools.PATTERN.instanceType` | **EC2 instance type**|**Type:** `string`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `nodePools.PATTERN.maxSize` | **Maximum number of nodes**|**Type:** `integer`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `nodePools.PATTERN.minSize` | **Minimum number of nodes**|**Type:** `integer`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `nodePools.PATTERN.rootVolumeSizeGB` | **Root volume size (GB)**|**Type:** `integer`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `nodePools.PATTERN.subnetTags` | **Subnet tags** - Tags to filter which AWS subnets will be used for this node pool.|**Type:** `array`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `nodePools.PATTERN.subnetTags[*]` | **Subnet tag**|**Type:** `object`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>|
| `nodePools.PATTERN.subnetTags[*].*` | **Tag value**|**Type:** `string`<br/>**Key pattern:**<br/>`PATTERN`=`^[a-z0-9]{5,10}$`<br/>**Value pattern:** `^[ a-zA-Z0-9\._:/=+-@]+$`<br/>|

### Other

| **Property** | **Description** | **More Details** |
| :----------- | :-------------- | :--------------- |
| `baseDomain` | **Base DNS domain**|**Type:** `string`<br/>|
| `cluster-shared` | **Library chart**|**Type:** `object`<br/>|
| `managementCluster` | **Management cluster** - Name of the Cluster API cluster managing this workload cluster.|**Type:** `string`<br/>|
| `provider` | **Cluster API provider name**|**Type:** `string`<br/>|



<!-- DOCS_END -->
