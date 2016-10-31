---
layout: default
---

# Running Ceph on a Cluster

In this post we will deploy and configure CephFS on a cluster. Then we will run
some metadata workloads to show how CephFS balances metadata. We assume the
image with your Ceph binaries is sitting in the internal registry as per the
[Packaging Ceph/Mantle Binaries](/blog0-packaging) blog.

## Telling Ansible about your Cluster

Ansible reads hosts from a `hosts` file:

```
[osds]
issdm-0
issdm-1
issdm-11
issdm-14
issdm-24
issdm-27
issdm-29
issdm-34
issdm-40

[mons]
issdm-3 

[mdss]
issdm-12

[clients0]
issdm-24
issdm-27
```

Now we can use [Ceph Ansible](https://github.com/ceph/ceph-ansible/wiki) to
configure and start all the Ceph daemons in the cluster. Ceph Ansible is
rapidly developed which means: (1) it will become mainstream soon and (2)
sometimes it does work smoothly. We forked the project and fixed some cluster
specific issues. 

```bash
~/$ mkdir -p  mantle/site
~/$ cd mantle
~/mantle$ git submodule add https://github.com/systemslab/srl-roles.git site/roles/srl
~/mantle$ git submodule add https://github.com/michaelsevilla/ceph-ansible.git site/roles/ceph
```

Now we point Ansible at these new roles in the `.ansible.cfg`:

```bash
[defaults]
action_plugins = roles/ceph/plugins/actions
roles_path = roles/ceph/roles:roles/srl
inventory = hosts
remote_user = issdm
host_key_checking = False
deprecation_warnings = False
retry_files_enabled = False
```

Finally, we add a `ceph.yml` file to specify how to bring up the cluster:

```yaml
---
# Defines deployment design and assigns role to server groups

- include: cleanup.yml

- hosts: mons
  become: True
  roles:
  - ceph-mon
  serial: 1 # MUST be '1' WHEN DEPLOYING MONITORS ON DOCKER CONTAINERS

- hosts: osds
  become: True
  roles:
  - ceph-osd

- hosts: mdss
  become: True
  roles:
  - ceph-mds
  serial: 1 # Please make the MDSs come in a specific order

- name: wait for the cluster to come up
  hosts: mons
  tasks:
  - command:  docker exec {{ ansible_hostname }} ceph -s
    register: result
    until:    result.stdout.find("HEALTH_OK") != -1
    retries:  120
  - debug:    var=result.stdout_lines

- hosts: clients
  become: True
  roles:
    - ceph-client
```

You also need to set the variables for your cluster. 
