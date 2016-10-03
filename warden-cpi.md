---
title: Warden/Garden CPI
menu:
  main:
    Name: Warden/Garden CPI
    identifier: bosh/warden-cpi
    parent: bosh
---

<p class="note">Note: Updated for bosh-warden-cpi v28+.</p>

This topic describes cloud properties for different resources created by the Warden/Garden CPI.

## <a id='azs'></a> AZs

Currently the CPI does not support any cloud properties for AZs.

Example:

```yaml
azs:
- name: z1
```

---
## <a id='networks'></a> Networks

Currently the CPI does not support any cloud properties for networks.

Example of a manual network:

```yaml
networks:
- name: default
  type: manual
  subnets:
  - range: 10.244.1.0/24
    gateway: 10.244.1.0
    static: [10.244.1.34]
```

<p class="note">Note: bosh-warden-cpi v24+ makes it possible to use subnets bigger than /30 as exemplified above. bosh-lite v9000.48.0 uses that newer bosh-warden-cpi.</p>

Example of a dynamic network:

```yaml
networks:
- {name: default, type: dynamic}
```

The CPI does not support vip networks.

---
## <a id='resource-pools'></a> Resource Pools

Currently the CPI does not support any cloud properties for resource pools.

Example of a resource pool:

```yaml
resource_pools:
- name: default
  network: default
  stemcell:
    name: bosh-warden-boshlite-ubuntu-trusty-go_agent
    version: latest
```

---
## <a id='disk-pools'></a> Disk Pools

Currently the CPI does not support any cloud properties for disks.

Example of 10GB disk:

```yaml
disk_pools:
- name: default
  disk_size: 10_240
```

---
## <a id='global'></a> Global Configuration

The CPI uses containers to represent VMs and loopback devices to represent disks. Since the CPI can only talk to a single Garden server it can only manage resources on a single machine.

Example of a CPI configuration:

```yaml
properties:
  warden_cpi:
    loopback_range: [100, 130]
    warden:
      connect_network: tcp
      connect_address: 127.0.0.1:7777
    actions:
      stemcells_dir: "/var/vcap/data/cpi/stemcells"
      disks_dir: "/var/vcap/store/cpi/disks"
      host_ephemeral_bind_mounts_dir: "/var/vcap/data/cpi/ephemeral_bind_mounts_dir"
      host_persistent_bind_mounts_dir: "/var/vcap/data/cpi/persistent_bind_mounts_dir"
    agent:
      mbus: nats://nats:nats-password@10.244.8.2:4222
      blobstore:
        provider: dav
        options:
          endpoint: http://10.244.8.2:25251
          user: agent
          password: agent-password
```

---
## <a id='cloud-config'>Example Cloud Config</a>

```yaml
azs:
- name: z1
- name: z2

vm_types:
- name: default

disk_types:
- name: default
  disk_size: 1024

compilation:
  workers: 5
  az: z1
  reuse_compilation_vms: true
  vm_type: default
  network: default

networks:
- name: default
  type: manual
  subnets:
    - azs: [z1, z2]
      range: 10.244.0.0/24
      reserved: [10.244.0.1]
      static: [10.244.0.34]
```

---
## <a id='notes'></a> Notes

* Garden server does not have a UI; however, you can use [gaol CLI](https://github.com/xoebus/gaol) to interact with it directly.

---
[Back to Table of Contents](index.html#cpi-config)
