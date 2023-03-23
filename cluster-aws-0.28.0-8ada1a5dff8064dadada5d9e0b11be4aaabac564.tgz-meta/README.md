# cluster-aws

`cluster-aws` is an app that helps create a CRs for a Cluster API AWS cluster for Giant Swarm platform.

## Configuration

<!-- GENERATED_VALUE_DOCS_START -->

| Property | Description | More details |
| :------- | :---------- | :----------- |
| `.baseDomain` | **Base DNS domain**  | **Type:** `string` |
| `.cluster-shared` | **Library chart**  | **Type:** `object` |
| `.connectivity` | **Connectivity**  | **Type:** `object` |
| `.connectivity.availabilityZoneUsageLimit` | **Availability zones**  – Maximum number of availability zones (AZ) that should be used in a region. If a region has more than this number of AZs then this number of AZs will be picked randomly when creating subnets.  | **Type:** `integer` |
| `.connectivity.bastion` | **Bastion host**  | **Type:** `object` |
| `.connectivity.bastion.enabled` | **Enable**  | **Type:** `boolean` |
| `.connectivity.bastion.instanceType` | **EC2 instance type**  | **Type:** `string` |
| `.connectivity.bastion.replicas` | **Number of hosts**  | **Type:** `integer` |
| `.connectivity.bastion.subnetTags` | **Subnet tags**  – Tags to filter which AWS subnets will be used for the bastion hosts.  | **Type:** `array` |
| `.connectivity.bastion.subnetTags[*]` | **Subnet tag**  | **Type:** `object` |
| `.connectivity.containerRegistries` | **Container registries**  – Endpoints and credentials configuration for container registries.  | **Type:** `object` |
| `.connectivity.dns` | **DNS**  | **Type:** `object` |
| `.connectivity.dns.additionalVpc` | **Additional VPCs**  – If DNS mode is 'private', the VPCs specified here will be assigned to the private hosted zone.  | **Type:** `array` |
| `.connectivity.dns.additionalVpc[*]` | **VPC identifier**  | **Type:** `string`<br>**Example:** `"vpc-x2aeasd1d"`<br>**Pattern:** `^vpc-[0-0a-zA-Z]+$` |
| `.connectivity.dns.mode` | **Mode**  – Whether the Route53 hosted zone of this cluster should be public or private.  | **Type:** `string` |
| `.connectivity.dns.resolverRulesOwnerAccount` | **Resolver rules owner**  – ID of the AWS account that created the resolver rules to be associated with the workload cluster VPC.  | **Type:** `string` |
| `.connectivity.network` | **Network**  | **Type:** `object` |
| `.connectivity.network.podCidr` | **Pod subnet**  – IPv4 address range for pods, in CIDR notation.  | **Type:** `string` |
| `.connectivity.network.serviceCidr` | **Service subnet**  – IPv4 address range for services, in CIDR notation.  | **Type:** `string` |
| `.connectivity.network.vpcCidr` | **VPC subnet**  – IPv4 address range to assign to this cluster's VPC, in CIDR notation.  | **Type:** `string` |
| `.connectivity.proxy` | **Proxy**  – Whether/how outgoing traffic is routed through proxy servers.  | **Type:** `object` |
| `.connectivity.proxy.enabled` | **Enable**  | **Type:** `boolean` |
| `.connectivity.proxy.httpProxy` | **HTTP proxy**  – To be passed to the HTTP_PROXY environment variable in all hosts.  | **Type:** `string` |
| `.connectivity.proxy.httpsProxy` | **HTTPS proxy**  – To be passed to the HTTPS_PROXY environment variable in all hosts.  | **Type:** `string` |
| `.connectivity.proxy.noProxy` | **No proxy**  – To be passed to the NO_PROXY environment variable in all hosts.  | **Type:** `string` |
| `.connectivity.sshSsoPublicKey` | **SSH public key for single sign-on**  | **Type:** `string` |
| `.connectivity.subnets` | **Subnets**  – Subnets are created and tagged based on this definition.  | **Type:** `array` |
| `.connectivity.subnets[*]` | **Subnet**  | **Type:** `object` |
| `.connectivity.subnets[*].cidrBlocks` | **Network**  | **Type:** `array` |
| `.connectivity.subnets[*].cidrBlocks[*]` | **None**  | **Type:** `object` |
| `.connectivity.subnets[*].cidrBlocks[*].availabilityZone` | **Availability zone**  | **Type:** `string`<br>**Example:** `"a"` |
| `.connectivity.subnets[*].cidrBlocks[*].cidr` | **Address range**  – IPv4 address range, in CIDR notation.  | **Type:** `string` |
| `.connectivity.subnets[*].cidrBlocks[*].tags` | **Tags**  – AWS resource tags to assign to this subnet.  | **Type:** `object` |
| `.connectivity.subnets[*].isPublic` | **Public**  | **Type:** `boolean` |
| `.connectivity.subnets[*].tags` | **Tags**  – AWS resource tags to assign to this CIDR block.  | **Type:** `object` |
| `.connectivity.topology` | **Topology**  – Networking architecture between management cluster and workload cluster.  | **Type:** `object` |
| `.connectivity.topology.mode` | **Mode**  – Valid values: GiantSwarmManaged, UserManaged, None.  | **Type:** `string` |
| `.connectivity.topology.prefixListId` | **Prefix list ID**  – ID of the managed prefix list to use when the topology mode is set to 'UserManaged'.  | **Type:** `string` |
| `.connectivity.topology.transitGatewayId` | **Transit gateway ID**  – If the topology mode is set to 'UserManaged', this can be used to specify the transit gateway to use.  | **Type:** `string` |
| `.connectivity.vpcEndpointMode` | **VPC endpoint mode**  – Who is reponsible for creation and management of VPC endpoints.  | **Type:** `string` |
| `.connectivity.vpcMode` | **VPC mode**  – Whether the cluser's VPC is created with public, internet facing resources (public subnets, NAT gateway) or not (private).  | **Type:** `string` |
| `.controlPlane` | **Control plane**  | **Type:** `object` |
| `.controlPlane.apiMode` | **API mode**  – Whether the Kubernetes API server load balancer should be reachable from the internet (public) or internal only (private).  | **Type:** `string` |
| `.controlPlane.containerdVolumeSizeGB` | **Containerd volume size (GB)**  | **Type:** `integer` |
| `.controlPlane.etcdVolumeSizeGB` | **Etcd volume size (GB)**  | **Type:** `integer` |
| `.controlPlane.instanceType` | **EC2 instance type**  | **Type:** `string` |
| `.controlPlane.kubeletVolumeSizeGB` | **Kubelet volume size (GB)**  | **Type:** `integer` |
| `.controlPlane.machineHealthCheck` | **Machine health check**  | **Type:** `object` |
| `.controlPlane.machineHealthCheck.enabled` | **Enable**  | **Type:** `boolean` |
| `.controlPlane.machineHealthCheck.maxUnhealthy` | **Maximum unhealthy nodes**  | **Type:** `string`<br>**Example:** `"40%"` |
| `.controlPlane.machineHealthCheck.nodeStartupTimeout` | **Node startup timeout**  – Determines how long a machine health check should wait for a node to join the cluster, before considering a machine unhealthy.  | **Type:** `string`<br>**Example:** `"10m"`, `"100s"` |
| `.controlPlane.machineHealthCheck.unhealthyNotReadyTimeout` | **Timeout for ready**  – If a node is not in condition 'Ready' after this timeout, it will be considered unhealthy.  | **Type:** `string`<br>**Example:** `"300s"` |
| `.controlPlane.machineHealthCheck.unhealthyUnknownTimeout` | **Timeout for unknown condition**  – If a node is in 'Unknown' condition after this timeout, it will be considered unhealthy.  | **Type:** `string`<br>**Example:** `"300s"` |
| `.controlPlane.oidc` | **OIDC authentication**  | **Type:** `object` |
| `.controlPlane.oidc.caPem` | **Certificate authority**  – Identity provider's CA certificate in PEM format.  | **Type:** `string` |
| `.controlPlane.oidc.clientId` | **Client ID**  | **Type:** `string` |
| `.controlPlane.oidc.groupsClaim` | **Groups claim**  | **Type:** `string` |
| `.controlPlane.oidc.issuerUrl` | **Issuer URL**  – Exact issuer URL that will be included in identity tokens.  | **Type:** `string` |
| `.controlPlane.oidc.usernameClaim` | **Username claim**  | **Type:** `string` |
| `.controlPlane.rootVolumeSizeGB` | **Root volume size (GB)**  | **Type:** `integer` |
| `.controlPlane.subnetTags` | **Subnet tags**  – Tags to select AWS resources for the control plane by.  | **Type:** `array` |
| `.controlPlane.subnetTags[*]` | **Subnet tag**  | **Type:** `object` |
| `.defaultMachinePools` | **Default node pool**  | **Type:** `object` |
| `.internal` | **Internal**  – For Giant Swarm internal use only, not stable, or not supported by UIs.  | **Type:** `object` |
| `.internal.hashSalt` | **Hash salt**  – If specified, this token is used as a salt to the hash suffix of some resource names. Can be used to force-recreate some resources.  | **Type:** `string` |
| `.internal.kubernetesVersion` | **Kubernetes version**  | **Type:** `string`<br>**Example:** `"1.24.7"` |
| `.kubectlImage` | **Kubectl image**  | **Type:** `object` |
| `.kubectlImage.name` | **Repository**  | **Type:** `string` |
| `.kubectlImage.registry` | **Registry**  | **Type:** `string` |
| `.kubectlImage.tag` | **Tag**  | **Type:** `string` |
| `.managementCluster` | **Management cluster**  – Name of the Cluster API cluster managing this workload cluster.  | **Type:** `string` |
| `.metadata` | **Metadata**  | **Type:** `object` |
| `.metadata.description` | **Cluster description**  – User-friendly description of the cluster's purpose.  | **Type:** `string` |
| `.metadata.name` | **Cluster name**  – Unique identifier, cannot be changed after creation.  | **Type:** `string` |
| `.metadata.organization` | **Organization**  | **Type:** `string` |
| `.nodePools` | **Node pools**  | **Type:** `object` |
| `.provider` | **Cluster API provider name**  | **Type:** `string` |
| `.providerSpecific` | **AWS settings**  | **Type:** `object` |
| `.providerSpecific.ami` | **Amazon machine image (AMI)**  – If specified, this image will be used to provision EC2 instances.  | **Type:** `string` |
| `.providerSpecific.awsClusterRoleIdentityName` | **Cluster role identity name**  – Name of an AWSClusterRoleIdentity object. This in turn refers to the IAM role used to create all AWS cloud resources when creating the cluster. The role can be in another AWS account in order to create all resources in that account. Note: This name does not refer directly to an IAM role name/ARN.  | **Type:** `string`<br>**Pattern:** `^[-a-zA-Z0-9_\.]{1,63}$` |
| `.providerSpecific.flatcarAwsAccount` | **AWS account owning Flatcar image**  – AWS account ID owning the Flatcar Container Linux AMI.  | **Type:** `string` |
| `.providerSpecific.region` | **Region**  | **Type:** `string` |

<!-- GENERATED_VALUE_DOCS_END -->


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
