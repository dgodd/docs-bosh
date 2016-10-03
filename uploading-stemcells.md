---
title: Uploading Stemcells
menu:
  main:
    Name: Uploading Stemcells
    identifier: bosh/uploading-stemcells
    parent: bosh
---

(See [What is a Stemcell?](stemcell.html) for an introduction to stemcells.)

As described earlier, each resource pool references a specific stemcell on the Director. Before the Director can form a deployment, all referenced stemcells must be uploaded to the Director.

## <a id='find'></a> Finding Stemcells

The [stemcells section of bosh.io](http://bosh.io/stemcells) lists official stemcells.

---
## <a id='upload'></a> Uploading to the Director

Assuming the CLI is already targeted at the Director, the CLI provides a single command to upload a stemcell.

- If you have a URL to a stemcell tarball (for example URL provided by bosh.io):

    <pre class="terminal">
    $ bosh upload stemcell [URL] --skip-if-exists
    </pre>

- If you have a stemcell tarball on your local machine:

    <pre class="terminal">
    $ bosh upload stemcell ~/Downloads/bosh-stemcell-2751-aws-xen-hvm-ubuntu-trusty-go_agent.tgz --skip-if-exists
    </pre>

Once the command succeeds you can view all uploaded stemcells in the Director:

<pre class="terminal">
$ bosh stemcells

+-----------------------------------------+---------------+---------+--------------+
| Name                                    | OS            | Version | CID          |
+-----------------------------------------+---------------+---------+--------------+
| bosh-aws-xen-hvm-ubuntu-trusty-go_agent | ubuntu-trusty | 2751    | ami-cc27a1a4 |
| bosh-aws-xen-centos-go_agent            | centos-6      | 2710    | ami-a0a674c8 |
+-----------------------------------------+---------------+---------+--------------+

(*) Currently in-use

Stemcells total: 2
</pre>

---
## <a id='using'></a> Deployment Manifest Usage

To use uploaded stemcell in your deployment, stemcell needs to be referenced by one of the resource pools:

```yaml
resource_pools:
- name: redis-servers
  network: default

  # Association with Ubuntu Trusty stemcell for AWS
  stemcell:
    name: bosh-aws-xen-hvm-ubuntu-trusty-go_agent
    version: 2751

  cloud_properties: {instance_type: m1.medium}
```

---
Next: [Uploading Releases](uploading-releases.html)

Previous: [Deployment Basics](deployment-basics.html)
