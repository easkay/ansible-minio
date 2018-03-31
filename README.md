# Ansible Role: minio

An Ansible role that installs and configures Minio on CentOS, RedHat, Ubuntu and Debian.

Minio can be configured in a variety of different ways, including across one or more servers, one or more disks as well as both in combination.
Minio can also be multi-tenanted to run several instances on the same server.
This role is designed to cater for all of these scenarios while also supporting Minio's own best practices.

## Requirements

Please give the `defaults/main.yml` file a thorough read to make sure the default settings are suitable for your platform.
There are a few examples and comments in the `defaults/main.yml` file explaining in more detail how the variables work.

This role has only been tested on distributions that use systemd as their service control system.

## Role Variables

This role contains many variables to allow the requisite flexibility when installing minio in its different forms.

### Validation

There is an `asserts.yml` task file which includes various assertions to check the sanity of the variables the role depends on.
In most cases, this is to prevent the role from behaving abnormally and failing due to configuration errors.
The variable validation also checks a couple of Minio requirements, e.g. the number of volumes, for sanity.

If you are struggling to get past the validation, check the output for suggestions.

### System-wide variables

```
minio_name_separator: "-"
```
This variable is used when multi-tenanting multiple Minio instances on the same server.
When multi-tenanting, the various configuration files need to be namespaced to a specific instance of Minio.
The instance name is used along with this name separator string to form a string as follows: `minio<minio_name_separator><name>`, e.g. `minio-instance1`.
If hyphens don't float your boat, you can change this separator here.

```
minio_user: minio
```
This variable defines the user to create (if necessary) and run Minio under.

```
minio_download_url: https://dl.minio.io/server/minio/release/linux-amd64/minio
minio_download_dir: /tmp
minio_working_dir: /usr/local
minio_binary_dir: /usr/local/bin
```
These variables tell the role where to download Minio from, where to temporarily put the downloaded binary and where to install it to.
The `minio_working_dir` is required for the systemd service unit file.
You should not need to change these variables unless you wish to put the binary in a specific location.

```
minio_always_update_binary: false
```
This variable emulates behaviour similar to `package: state=latest` and will replace the minio binary if there is a new version available.
If a new binary is installed, all Minio instances will be restarted.

```
minio_setup_logging: true
minio_log_dir: /var/log/minio
minio_setup_logrotate: true
```
These variables control this role's involvement with logging.
If `minio_setup_logging` is set to false, systemd will send output from Minio to systemd's journal.
If set to true, the logging will be configured through syslog.
The `minio_log_dir` specifies which directory Minio should log to via syslog.
The name of the log file is configurable on a per-instance basis (see below).
If you wish to use logrotate, `minio_setup_logrotate` will copy a basic config `templates/minio-logrotate.conf.j2` into logrotate's config directory.
Logrotate's settings for Minio can be changed in that template.

```
minio_setup_firewall: "{{ ansible_distribution != 'Debian' }}"
```
If you want this role to open Minio's port(s) in the OS firewall, set this to true.
To enable the role to run on stock installations of the various distributions, the logic is done here.
You can of course override the firewall setup here if you wish.
Some assumptions are made as to which firewall is installed on your OS based on the `ansible_os_family`.
CentOS/EL assumes firewalld and Ubuntu/Debian assumes ufw.

```
minio_tune_kernel: false
minio_tune_kernel_values:
  - { name: "net.ipv4.tcp_fin_timeout", value: 30 }
  - { name: "net.ipv4.tcp_keepalive_probes", value: 5 }
  - { name: "net.core.wmem_max", value: 540000 }
  - { name: "net.core.rmem_max", value: 540000 }
  - { name: "vm.swappiness", value: 10 }
  - { name: "vm.dirty_background_ratio", value: 1 }
  - { name: "vm.dirty_ratio", value: 5 }
  - { name: "kernel.sched_min_granularity_ns", value: 10000000 }
  - { name: "kernel.sched_wakeup_granularity_ns", value: 15000000 }
```
Minio has given various suggestions around OS tuning for performance, which have been reproduced here.
By default, this tuning is turned off but can be enabled by setting `minio_tune_kernel` to true.
Enabling tuning will also run a disk performance tuning script written by Minio.
If you wish, you can also tweak or change the tuning that's performed by changing the values in `minio_tune_kernel_values`.
The disk tuning script can be found in `files/minio-disk-tuning.sh`.

### Per-instance variables

Minio can be installed in a single or multi-tenanted fashion, and there are different ways to configure the required settings for both of these cases.
If you are running in a single-tenanted manner, you will need to change the variables prefixed with `minio_`.
These variables are found under the `# Single tenant settings` section of `defaults/main.yml` and are as follows:
```
minio_bind_address: 0.0.0.0
minio_bind_port: 9000
```
These two variables control the listening address and port of a particular Minio instance.

```
minio_cluster_members: []
# minio_cluster_members: "{{ groups['minio-ubuntu1710'] }}"
```
This variable controls which other hosts, if any, will take part in a Minio cluster and provide remote volumes.
If you wish to cluster Minio across remote hosts, supply a list of hosts from the inventory that are part of the same play.
The hosts must be part of the play so that facts can be gathered to supply their IPs as remote volumes.

```
minio_volumes: ["/mnt/minio1"]
```
This variable is an array of paths on disk to one or more mounted disks/devices for use by Minio.

```
minio_log_file_name: minio.log
```
If you want to change the name of the log file produced by Minio, you can do it here.

```
minio_access_key: <redacted>
minio_secret_key: <redacted>
```
These variables set the credentials to be used for an instance of Minio.
When using remote volumes as part of a cluster, these *must* be set so that the volumes can connect to one another.
If no remote volumes are in use, these variables are optional and Minio will generate some credentials for you.

```
minio_region: <optional>
minio_domain: <optional>
minio_browser: <optional>
```
The region, domain and browser variables are optional and are as documented here: https://docs.minio.io/docs/minio-server-configuration-guide

### Multi-tenanting

If you wish to run multiple instances of Minio side-by-side on the same server(s), you should configure the `minio_instances` variable instead of the individual per-instance `minio_` prefixed variables as detailed previously.
All of the same settings can be adjusted and an example setup might look like this:
```
minio_instances:
- name: first
  bind_address: 0.0.0.0
  bind_port: 9000
  cluster_members: []
  volumes: ["/mnt/minio-first01", "/mnt/minio-first02", "/mnt/minio-first03", "/mnt/minio-first04"]
  log_file_name: minio-first.log
- name: second
  bind_address: 0.0.0.0
  bind_port: 9001
  cluster_members: "{{ groups['minio'] }}"
  volumes: ["/mnt/minio-second"]
  access_key: <redacted>
  secret_key: <redacted>
  log_file_name: minio-second.log
```

Notice how the `access_key` and `secret_key` variables are optional when there are no remote volumes (`cluster members: []`).
This variable can be omitted and has been left in for completeness.

## Dependencies

None.

## Example Playbook

    - hosts: minio-cluster-alpha
      vars:
        minio_volumes: ["/mnt/volume_one"]
      roles:
        - { role: easkay.minio }

## License

MIT
