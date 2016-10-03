---
title: Deployment Basics
menu:
  main:
    Name: Deployment Basics
    identifier: bosh/deployment-basics
    parent: bosh
---

A [deployment](deployment.html) is a collection of VMs and persistent disks. A deployment is split into smaller logical units called deployment jobs.

The sections below show how to describe the desired deployment in a deployment manifest. An [example full deployment manifest](#example) is below.

## <a id='deployment-jobs'></a> Deployment Jobs

A _deployment job_ is a logical unit, defined by the cluster operator, that represents a long running service or a short running task (an errand). Deployment jobs are defined in the deployment manifest. A job can be configured to run multiple copies of itself: i.e., a job has multiple instances. The operator usually decides which jobs he or she needs to make their deployment do something useful. For example, when creating a Redis deployment, the operator will add a Redis server job, and might also decide to add a Redis slave job.

Each job needs to be backed by an actual software -- releases. An operator, when defining a job in the manifest, has to specify which release or even multiple releases make up this job. Continuing with our example, the Redis server job would use a `redis-release` that contains packaged up Redis server software.

```yaml
# Associates redis release with this deployment
releases:
- {name: redis, version: 12}

jobs:
- name: redis-master
  instances: 1

  # Associates redis-master with redis release
  templates:
  - {name: redis-server, release: redis}

  resource_pool: redis-servers
  networks:
  - name: default
```

---
## <a id='resource-pools'></a> Resource Pools

Since each job instance will run on a single machine (a VM or a container), it needs to reference a stemcell. Hence each deployment job belongs to exactly one resource pool. A _resource pool_ is a collection of machines (VMs) that are created from a specific stemcell. Resource pools are also defined by the operator in the deployment manifest. In addition to specifying a stemcell, the resource pool definition allows the operator to specify VM size, instance type, and other IaaS-specific characteristics that are used when creating VMs. For example, the `redis-servers` resource pool for Redis servers, referenced by the `redis-master` job in the above example, might look something like this on AWS:

```yaml
resource_pools:
- name: redis-servers
  network: default

  # Association with Ubuntu Trusty stemcell for AWS
  stemcell:
    name: bosh-aws-xen-ubuntu-trusty-go_agent
    version: 2765

  # IaaS specific properties
  cloud_properties:
    instance_type: m1.medium
```

### Compilation

Most VMs in the deployment belong to a specific resource pool; however, the Director also creates compilation worker VMs for release compilation. The Director will compile each release on every necessary stemcell defined in the resource pools section. A compilation definition allows an operator to specify VM size, instance type, and other IaaS-specific characteristics that are used when creating VMs.

```yaml
compilation:
  workers: 2
  network: default

  # IaaS-specific properties
  cloud_properties:
    instance_type: m1.medium
```

---
## <a id='persistent-disks'></a> Persistent Disks

Some deployment jobs may need to store persistent data -- data that lives as long as the job instance is defined, independent of whether the job instance VM is deleted or recreated. For example, an operator can configure each instance of the `redis-slave` job to have a 10GB persistent disk:

```yaml
jobs:
- name: redis-slave
  instances: 2
  templates:
  - {name: redis-server, release: redis}

  # Associated 10GB disk
  persistent_disk: 10_240

  resource_pool: redis-servers
  networks:
  - name: default
```

[Learn more about persistent disks](persistent-disks.html).

---
## <a id='networks'></a> Networks

A deployment job must also reference at least one network. A _BOSH network_ is an IaaS-agnostic representation of the networking layer. Each job instance gets an IP from its associated networks. Based on the type of the network, BOSH decides how to find and select an IP. There are three network types: `manual`, `dynamic`, and `vip`. Manual networks require that users specify one or more subnets and BOSH determines how to assign IPs to each job instance. For dynamic networks, BOSH defers IP selection to the IaaS. For vip networks, BOSH allows users to assign IPs to VMs. In the deployment job example shown above, the `redis-master` job references a `default` network. The example below shows how a user might specify a `default` network definition for AWS:

```yaml
networks:
- name: default
  type: manual

  subnets:
  - range:    10.10.0.0/24
    gateway:  10.10.0.1
    dns:      [10.10.0.2]
    reserved: [10.10.0.2-10.10.0.10]

    # IaaS specific properties
    cloud_properties:
      subnet: subnet-9be6c3f7
```

[Learn more about networks](networks.html).

---
## <a id='example'></a> Example Full Deployment Manifest

With all pieces combined together most deployment manifests look something like this:

<p class="note"><strong>Note</strong>: Certain sections (compilation and update) of the deployment manifest were not described. Refer to the <a href="./deployment-manifest.html">Deployment Manifest Schema</a> topic for details.</p>

```yaml
name: my-redis-deployment

director_uuid: cf8dc1fc-9c42-4ffc-96f1-fbad983a6ce6

releases:
- {name: redis, version: 12}

networks:
- name: default
  type: manual
  subnets:
  - range:    10.10.0.0/24
    gateway:  10.10.0.1
    dns:      [10.10.0.2]
    reserved: [10.10.0.2-10.10.0.10]
    cloud_properties: {subnet: subnet-9be6c3f7}

resource_pools:
- name: redis-servers
  network: default
  stemcell:
    name: bosh-aws-xen-ubuntu-trusty-go_agent
    version: 2708
  cloud_properties:
    instance_type: m1.small
    availability_zone: us-east-1c

compilation:
  workers: 2
  network: default
  cloud_properties:
    instance_type: m1.small
    availability_zone: us-east-1c

update:
  canaries: 1
  max_in_flight: 10
  update_watch_time: 1000-30000
  canary_watch_time: 1000-30000

jobs:
- name: redis-master
  instances: 1
  templates:
  - {name: redis-server, release: redis}
  persistent_disk: 10_240
  resource_pool: redis-servers
  networks:
  - name: default

- name: redis-slave
  instances: 2
  templates:
  - {name: redis-server, release: redis}
  persistent_disk: 10_240
  resource_pool: redis-servers
  networks:
  - name: default
```

Once an operator has made a decision as to which deployment jobs, releases, and stemcells will be used, the operator can create a deployment.

---
Next: [Uploading Stemcells](uploading-stemcells.html) or see [Deployment Manifest Schema](deployment-manifest.html) for deeper dive into deployment manifest.

Previous: [Basic Workflow](basic-workflow.html)
