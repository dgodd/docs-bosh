---
title: Compiled Releases
menu:
  main:
    Name: Compiled Releases
    identifier: bosh/compiled-releases
    parent: bosh
---

<p class="note">Note: This feature is available with bosh-release v210+.</p>

Typically release tarballs are distributed with source packages; however, there may be a requirement to use compiled packages in an environment (for example a production environment) where:

- compilation is not permitted for security reasons
- access to source packages is not permitted for legal reasons
- exact existing audited binary assets are expected to be used

Any release can be exported as a compiled release by using the Director and [bosh export release](sysadmin-commands.html#dir-release) command.

---
## <a id="export"></a> Using export release command

To export a release:

1. Create an empty deployment (or use an existing one). This deployment will hold compilation VMs if compilation is necessary.

    ```yaml
    ---
    name: compilation-workspace

    director_uuid: 51f02cfb-0e06-46ac-bab4-bf0da18f28c4

    releases:
    - name: uaa
      version: "6"

    stemcells:
    - alias: default
      os: ubuntu-trusty
      version: latest

    jobs: []

    update:
      canaries: 1
      max_in_flight: 1
      canary_watch_time: 1000-90000
      update_watch_time: 1000-90000
    ```

    <p class="note">Note: This example assumes you are using [cloud config](cloud-config.html), hence no compilation, networks and other sections were defined. If you are not using cloud config you will have to define them.</p>

1. Reference desired release versions you want to export.

1. Deploy. Example manifest above does not allocate any resources when deployed.

1. Run `bosh export release` command. In our example: `bosh export release uaa/6 ubuntu-trusty/3197`. If release is not already compiled it will create necessary compilation VMs and compile all packages.

1. Find exported release tarball in the current directory. Compiled release tarball can be now imported into any other Director via `bosh upload release` command.

1. Optionally use `bosh inspect release` command to view associated compiled packages on the Director. In our example: `bosh inspect release uaa/6`.
