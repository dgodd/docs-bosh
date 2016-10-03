---
title: Setting Up vSphere for BOSH
---

This topic explains how to prepare vSphere for deploying BOSH or MicroBOSH.

##<a id="hardware"></a>Step 1: Allocate Hardware Resources ##

Your intended use of BOSH determines what hardware resources you should allocate
in your IaaS.

For example:

* BOSH needs room to store the releases it deploys.
Larger applications or larger numbers of applications require more disk space.
* Larger numbers of active users require more RAM.

vSphere describes hardware in terms of RAM, CPU, virtual disk, and other specs.
Refer to the [hardware requirements](http://docs.cloudfoundry.org/deploying/vsphere/hardware_spec.html)
for Cloud Foundry topic for help deciding what hardware to use.

##<a id="traffic"></a>Step 2: Restrict Network Traffic ##

To restrict incoming network traffic to the BOSH VMs on vSphere:

1. Deploy BOSH and your applications deployed with BOSH to separate private
networks.

1. Route requests to BOSH or your applications from a router or load balancer on
a public network.

One way to accomplish this is to configure Network Address Translation (NAT) on a VM.
Assign two network adapters to the VM.
This results in the VM having two IP addresses:

* A public-facing IP address: This IP address connects to the public network
VLAN through an external gateway IP address.
* An internal VLAN IP address: This IP address connects to BOSH and your
application VMs through an internal gateway IP address.
