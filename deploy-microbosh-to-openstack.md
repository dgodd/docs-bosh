---
title: Deploying MicroBOSH to OpenStack
menu:
  main:
    Name: Deploying MicroBOSH to OpenStack
    identifier: bosh/deploy-microbosh-to-openstack
    parent: bosh
---

This topic explains how to deploy MicroBOSH to OpenStack.

<p class="note">Note: Micro CLI plugin is deprecated, but is still supported. To set up a new Director VM or upgrade your MicroBOSH, please follow <a href="./init-openstack.html">Initializing on OpenStack</a> document.</p>

##<a id="create-manifest"></a>Step 1: Create a Deployment Manifest ##

The MicroBOSH deployment manifest is a YAML file that defines the components and
properties of the deployment.
Create a manifest for your deployment as follows:

1. Create a local deployment directory to store your manifest.

    <pre class="terminal">
    $ mkdir ~/my-micro-deployment
    </pre>

1. Create a deployment manifest YAML file.

    The example below is a MicroBOSH deployment manifest template.
	Copy and paste the template into a text editor and save the manifest to your
	deployment directory.

    In the template, you must replace the `NETWORK-UUID`,
	`SUBNET-POOL-IP-ADDRESS`, `FLOATING-IP`, `OPENSTACK-PASSWORD`,
	`IDENTITY-API-ENDPOINT`, `OPENSTACK-TENANT`, and `OPENSTACK-USERNAME`
	properties.
	We describe replacing these properties in [Step 2: Prepare an OpenStack environment](#prepare-openstack).

    <p class="note"><strong>Note</strong>: The example below has the file name
	<code>manifest.yml</code>, which we reference in other examples in this
	topic.</p>

```yaml
---
name: microbosh

network:
  type: manual
  vip: FLOATING-IP # Replace with a floating IP address
  ip: SUBNET-POOL-IP-ADDRESS # Replace with an address from the subnet IP address allocation pool of your OpenStack internal network
  cloud_properties:
    net_id: NETWORK-UUID # Replace with your OpenStack internal network UUID

resources:
  persistent_disk: 20000
  cloud_properties:
    instance_type: m1.xlarge

cloud:
  plugin: openstack
  properties:
    openstack:
      auth_url: IDENTITY-API-ENDPOINT # Replace with your OpenStack Identity API endpoint
      tenant: OPENSTACK-TENANT # Replace with OpenStack tenant name
      username: OPENSTACK-USERNAME # Replace with OpenStack username
      api_key: OPENSTACK-PASSWORD # Replace with your OpenStack password
      default_key_name: microbosh # OpenStack Keypair name
      private_key: microbosh.pem # Path to OpenStack Keypair private key
      default_security_groups: [bosh]

apply_spec:
  properties:
    director: {max_threads: 3}
    hm: {resurrector_enabled: true}
    ntp: [0.north-america.pool.ntp.org, 1.north-america.pool.ntp.org]
```

---
##<a id="prepare-openstack"></a>Step 2: Prepare an OpenStack Environment ##

To prepare your OpenStack project for deploying MicroBOSH, use the OpenStack GUI
to perform the following tasks:

* Verify [Prerequisites](#prerequisites)
* Create a [Keypair](#keypair).
* Create and configure [Security Groups](#security-groups).
* Allocate a [floating IP address](#floating-ip).

### <a id="prerequisites"></a>Prerequisites ###

1. An OpenStack environment running one of the following supported releases:
    * [Havana](http://www.openstack.org/software/havana)
    * [Icehouse](http://www.openstack.org/software/icehouse)
    * [Juno](http://www.openstack.org/software/juno)

1. The following OpenStack services:
    * [Identity](http://www.openstack.org/software/openstack-shared-services/):
        MicroBOSH authenticates credentials and retrieves the endpoint URLs for
		other OpenStack services.
    * [Compute](http://www.openstack.org/software/openstack-compute/):
        MicroBOSH boots new VMs, assigns floating IPs to VMs, and creates and
		attaches volumes to VMs.
    * [Image](http://www.openstack.org/software/openstack-shared-services/):
        MicroBOSH stores stemcells using the Image service.
    * **(Optional)** [OpenStack
		Networking](http://www.openstack.org/software/openstack-networking/):
        Provides network scaling and automated management functions that are
		useful when deploying complex distributed systems.

1. The following OpenStack networks:
    * An external network with a subnet.
    * An internal network with a subnet. The subnet must have an IP address allocation pool.

1. A new OpenStack project.

<p class="note"><strong>Note</strong>: See the <a href="http://docs.openstack.org/">OpenStack documentation</a> for help finding
	more information.</p>

---
### <a id="keypair"></a>Create a Keypair ###

1. Select **Access & Security** from the left navigation panel.

1. Select the **Keypairs** tab.

    <%= image_tag("images/micro-openstack/keypair.png") %>

1. Click **Create Keypair**.

1. Name the Keypair "microbosh" and click **Create Keypair**.

    <%= image_tag("images/micro-openstack/create-keypair.png") %>

1. Save the `microbosh.pem` file.

    <%= image_tag("images/micro-openstack/save-keypair.png") %>

1. Move the `microbosh.pem` file into your local deployment directory.
For example, on UNIX run this command:

    <pre class="terminal">
    mv ~/Downloads/bosh.pem ~/my-micro-deployment/microbosh.pem
    </pre>

---
### <a id="security-groups"></a>Create and Configure Security Groups ###

You must create and configure two Security Groups to restrict incoming network
traffic to the BOSH VMs.

####BOSH Security Group ####

1. Select **Access & Security** from the left navigation panel.

1. Select the **Security Groups** tab.

    <%= image_tag("images/micro-openstack/security-groups.png") %>

1. Click **Create Security Group**.

1. Name the security group "bosh" and add the description "BOSH Security Group".

    <%= image_tag("images/micro-openstack/create-bosh-sg.png") %>

1. Click **Create Security Group**.

1. Select the BOSH Security Group and click **Edit Rules**.

1. Click **Add Rule**.

    <%= image_tag("images/micro-openstack/edit-bosh-sg.png") %>

1. Add the following rules to the BOSH Security Group:

    <table border="1" class="nice" >
      <tr>
        <th><b>Direction</b></th>
        <th><b>Ether Type</b></th>
        <th><b>IP Protocol</b></th>
        <th><b>Port Range</b></th>
        <th><b>Remote</b></th>
      </tr>
      <tr>
	    <td>Ingress</td>
	    <td>IPv4</td>
	    <td>TCP</td>
	    <td>1-65535</td>
	    <td>bosh</td>
	  </tr>
      <tr>
	    <td>Ingress</td>
	    <td>IPv4</td>
	    <td>TCP</td>
	    <td>25777</td>
	    <td>0.0.0.0/0 (CIDR)</td>
	  </tr>
      <tr>
	    <td>Ingress</td>
	    <td>IPv4</td>
	    <td>TCP</td>
	    <td>25555</td>
	    <td>0.0.0.0/0 (CIDR)</td>
	  </tr>
      <tr>
	    <td>Ingress</td>
	    <td>IPv4</td>
	    <td>TCP</td>
	    <td>25250</td>
	    <td>0.0.0.0/0 (CIDR)</td>
	  </tr>
      <tr>
	    <td>Ingress</td>
	    <td>IPv4</td>
	    <td>TCP</td>
	    <td>6868</td>
	    <td>0.0.0.0/0 (CIDR)</td>
	  </tr>
      <tr>
	    <td>Ingress</td>
	    <td>IPv4</td>
	    <td>TCP</td>
	    <td>4222</td>
	    <td>0.0.0.0/0 (CIDR)</td>
	  </tr>
      <tr>
	    <td>Ingress</td>
	    <td>IPv4</td>
	    <td>UDP</td>
	    <td>68</td>
	    <td>0.0.0.0/0 (CIDR)</td>
	  </tr>
      <tr>
	    <td>Ingress</td>
	    <td>IPv4</td>
	    <td>TCP</td>
	    <td>53</td>
	    <td>0.0.0.0/0 (CIDR)</td>
	  </tr>
      <tr>
	    <td>Ingress</td>
	    <td>IPv4</td>
	    <td>UDP</td>
	    <td>53</td>
	    <td>0.0.0.0/0 (CIDR)</td>
	  </tr>
      <tr>
	    <td>Ingress</td>
	    <td>IPv4</td>
	    <td>TCP</td>
	    <td>22</td>
	    <td>0.0.0.0/0 (CIDR)</td>
	  </tr>
      <tr>
	    <td>Egress</td>
	    <td>IPv4</td>
	    <td>Any</td>
	    <td>-</td>
	    <td>0.0.0.0/0 (CIDR)</td>
	  </tr>
      <tr>
	    <td>Egress</td>
	    <td>IPv6</td>
	    <td>Any</td>
	    <td>-</td>
	    <td>::/0 (CIDR)</td>
	  </tr>
    </table>

    <p class="note"><strong>Note</strong>: Production environments should have
	MicroBOSH deployed without a floating IP address on the private subnet. We
	highly recommend against running any production environment with
	<code>0.0.0.0/0</code> source IP addresses.</p>

---
###<a id="floating-ip"></a>Allocate a Floating IP Address ###

1. Select **Access & Security** from the left navigation panel.

1. Select the **Floating IPs** tab.

    <%= image_tag("images/micro-openstack/create-floating-ip.png") %>

1. Click **Allocate IP to Project**.

1. Select **External** from the **Pool** dropdown menu.

1. Click **Allocate IP**.

    <%= image_tag("images/micro-openstack/allocate-ip.png") %>

1. Replace `FLOATING-IP` in your deployment manifest with the allocated
Floating IP Address.

    <%= image_tag("images/micro-openstack/floating-ip.png") %>

---
##<a id="download-stemcell"></a>Step 3: Download a Stemcell ##

1. Open [https://bosh.io/stemcells](https://bosh.io/stemcells) in a web browser
to view a list of publicly available BOSH stemcells.
The list displays the most recent build numbers of BOSH stemcells, organized by operating system, target IaaS, and hypervisor.

1. Choose a BOSH stemcell for OpenStack and click the build number to download.

---
##<a id="deploy-microbosh"></a>Step 4: Deploy MicroBOSH ##

1. In a terminal window, run `bosh micro deployment microbosh.yml` from your
deployment directory to instruct MicroBOSH to use your manifest file.

    <pre class='terminal'>
    $ cd ~/my-micro-deployment
    $ bosh micro deployment manifest.yml
    WARNING! Your target has been changed to https://173.81.16.12:25555!
    Deployment set to ~/my-micro-deployment/manifest.yml
    </pre>

    <p class="note"><strong>Note</strong>: BOSH displays a red
    <code>WARNING!</code> message. This is not an error message.

1. Run `bosh micro deploy STEMCELL-NAME` to deploy MicroBOSH.

    <p class="note"><strong>Note</strong>: BOSH may displays a red
    <code>No bosh-deployments.yml file found</code> message.
    If prompted to allow MicroBOSH to the save state in the current directory,
    type <code>yes</code>.</p>

    <pre class='terminal'>
    $ bosh micro deploy bosh-stemcell-3012-openstack-kvm-ubuntu-trusty-go_agent.tgz

    No 'bosh-deployments.yml' file found in current directory.

    Is ~/my-micro-deployment a directory where you can save state? (type 'yes' to continue): yes

    Deploying new micro BOSH instance ~/my-micro-deployment/manifest.yml to 'https://173.81.16.12:25555' (type 'yes' to continue): yes

      Started deploy micro bosh
      ...
      Done deploy micro bosh

    Deployed '~/my-micro-deployment/manifest.yml' to 'https://173.81.16.12:25555', took 00:04:51 to complete
    </pre>

1. Use `bosh target FLOATING-IP-ADDRESS` to log into your new MicroBOSH server.
The default username and password are `admin` and `admin`.

    <pre class="terminal">
    $ bosh target 173.81.16.12
    Target set to 'microbosh'
    Your username: admin
    Enter password: *****
    Logged in as 'admin'

    $ bosh vms
    No deployments
    </pre>

---
##<a id="troubleshooting"></a>Troubleshooting ##

If the deployment fails, run `bosh micro delete`, then deploy again.

If an API Key error message appears, check the accuracy of the `OPENSTACK-PASSWORD` in the deployment manifest.

If other error messages appear:

* Check your deployment manifest for typographical or formatting errors.
* Review your OpenStack configuration.

---
[Back to Table of Contents](index.html#install)

Previous: [Bootstrapping an environment](bootstrap.html)
