---
title: Bootstrapping an environment
---

MicroBOSH is a single-VM BOSH environment that includes all necessary BOSH components. It is the primary way to deploy the [BOSH Director](bosh-components.html#director) and its supporting components to an IaaS to start managing deployments.

Special BOSH CLI command (<code>bosh micro deploy</code>) can be used to create, update or migrate existing MicroBOSH VM. In addition to managing the VM, it is also responsible for managing the persistent disk attached to the MicroBOSH VM. Persistent disk is used by the Director to store information about the deployments it manages.

<p class="note">Note: Micro CLI plugin is deprecated, but is still supported. To set up a new Director VM or upgrade your MicroBOSH, follow <a href="./init.html">Bootstrapping an environment</a> document.</p>

Cloud Foundry recommends that you update your BOSH environment frequently to stay on the latest version of BOSH.

Depending on your IaaS, choose your next step:

- [Deploying MicroBOSH to AWS](deploy-microbosh-to-aws.html)
- [Deploying MicroBOSH to vSphere](deploy-microbosh-to-vsphere.html)
- [Deploying MicroBOSH to OpenStack](deploy-microbosh-to-openstack.html)
- Deploying MicroBOSH to vCloud

Alternatively, if you want a local BOSH environment, you can try [BOSH Lite](https://github.com/cloudfoundry/bosh-lite). It comes as a pre-built [Vagrant](https://www.vagrantup.com/) box. The Director that comes with BOSH Lite uses Warden CPI which uses containers to emulate VMs. The usage of containers makes it an excellent choice for local development, testing, and general BOSH exploration.

---
Previous: [BOSH Components](bosh-components.html)
