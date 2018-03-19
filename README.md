# ansible-minio - WIP
Ansible role to set up Minio

Hi folks, this is a work in progress. If you'd like to help, great! I'd welcome PRs, questions or suggestions.

## Aim

Create a role for Ansible which can handle a large majority of minio setups including:
* Single host, single volume
* Single host, multiple volume (erasure coding)
* Multiple host, single volumes (distributed)
* Multiple host, multiple volumes (distributed)
* Multiple host, multiple instance (multi-tenanting)

Make it as easy as possible to use (sorry I suck at this).

## TODO
* Multi-tenancy
* Distribution testing and customisations (ufw, syslog etc)
* Automated tests
* Separated variables for distribution dependencies
* Access key and secret access key generation
