---
title: CLI v2 vs v1
---

<p class="note">Note: Applies to CLI v2 alpha 1.</p>

| Before                      | After
|-----------------------------|-----------------------------
| bosh-init deploy <manifest> | bosh create-env <manifest>
| bosh-init delete <manifest> | bosh delete-env <manifest> [1]
| bosh target <ip>            | bosh env <ip>
| bosh status                 | bosh env [2]
| bosh -t dir ...             | bosh -e dir ...
| bosh -d manifest-path ...   | bosh -d deployment-name ... [3]
| bosh deployment <manifest>  | bosh deployment <name>
| bosh deploy                 | bosh deploy <manifest>
| bosh delete deployment      | bosh delete-deployment (uses currently set deployment)
| bosh tasks recent 1000      | bosh tasks -r=1000
| bosh tasks --no-filter      | bosh tasks -a
| bosh download manifest dep  | bosh manifest (uses currently set deployment)
| bosh vms dep                | bosh instances (uses currently set deployment)

[1] bosh-init CLI is now absorbed by the bosh CLI. One binary!
[2] Use 'environment' term consistently. Matches with create-env/delete-env commands.
[3] Most of the time users had to download manifest just to be able to run deployment commands. Instead of downloading manifest just provide a deployment name.

---
## <a id="general"></a> General differences

- All commands respect `-e` (environment) and `-d` (deployment) flags
- `-d` (deployment) flag accepts a deployment name instead of a manifest
- Variety of commands (create-env/delete-env/deploy/recreate/etc.) accept simple interpolation flags (`-v/-l`)
- All commands support friendlier non TTY output, forceful TTY output and `--json` formatting
- All commands now use dashes instead of spaces
- All commands expect 'piece1/piece2' formatting for instances, releases, and stemcells
- `^+C` doesnt ask for task cancellation and just exits CLI command (task continue to run)
- `--no-track` has been removed from the bosh task command
- Sorts tables in a more consistent manner
- Stores configuration file in `~/.bosh/config`
- Most of the output formatting

---
## <a id="commands"></a> Differences per command

- `bosh env`
  - only allows connections to Director configured with verifiable certificates
  - no longer asks to log-in

- `bosh login`
  - doesn't accept positional arguments for username/password

- `bosh deploy`
  - no longer checks or requires `director_uuid`
    - to achieve similar safety make sure to give unique deployment names across environments

- `bosh tasks`
  - sane arguments syntax (`-r` for recent and `-a` for all)

- `bosh ssh`
  - sane arguments syntax
  - support for running commands against multiple machines
  - adds `--ssh-opts` flag to pass through options to ssh command for port forwarding etc.
  - adds `-r` flag to collate results from multiple machines

- `bosh logs`
  - adds -f flag similar to `tail -f` (uses bosh ssh underneath)

- `bosh scp`
  - sane arguments syntax
  - support for running command against multiple machines

- `bosh vms`
  - removed second param for specifying deployment

- `bosh add-blob`
  - requires a path to its release destination
  - no longer uses symlinks to manage blobs but rather places file directly into `blobs/`

---
## <a id="interpolation"></a> Interpolation

`create-env`, `delete-env`, `deploy`, `update-cloud-config` and `update-runtime-config` commands allow to interpolate manifest variables via `--var` (`-v`) and `--var-files` (`-l`) flags. Currently interpolation requires all variables to be specified or command raises an error. Eventually for Director level commands missing variables will be sent to the Director to be looked up against a config server. Variable values could be any valid YAML value (strings, booleans, maps, etc.).

Example manifest:

```yaml
name: ((name))

networks:
- name: private
  cloud_properties: ((private_cloud_properties))
```

Example variables file:

```yaml
name: my-name
private_cloud_properties:
  subnet_id: subnet-2rq4t8s4
```

You can use `build-manifest` command to see what client side interpolation will do.
