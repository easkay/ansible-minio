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
The variable validation also checks a couple of Minio requirements, e.g. the number of volumes for sanity.

If you are struggling to get past the validation, check the output for suggestions.

The following variables apply to all Minio deployments using this role:

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


minio_download_url: https://dl.minio.io/server/minio/release/linux-amd64/minio
minio_download_dir: /tmp
minio_working_dir: /usr/local
minio_binary_dir: /usr/local/bin
minio_log_dir: /var/log/minio
minio_always_update_binary: false
minio_setup_logging: true
minio_setup_logrotate: true
minio_setup_firewall: "{{ ansible_distribution != 'Debian' }}"
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

## Dependencies

None.

## Example Playbook

    - hosts: webservers
      roles:
        - { role: username.acme }

## License

MIT
