hashicorp-vault-cluster-role
============================

This role creates for you a hashicorp vault cluster secured/unsealed by another vault.
It is not (yet) idempotent!

Requirements
------------

- You need to have installed
  - `docker.io`
  - `python3-docker`
  - `apt-transport-https`
- Four instances
  - One for the cluster unsealer
  - Three for the cluster

Role Variables
--------------

Watch out for the `true` values. They need to be strings, otherwise Jinja2 will template them to `True`,
which is invalid in **HCL**.

| Variable                         | Default value                                                          | Usage                                                    |
|----------------------------------|------------------------------------------------------------------------|----------------------------------------------------------|
| unsealer_host_group_name         | `"unsealer"`                                                           | Inventory group name for the unsealer node               |
| cluster_hosts_group_name         | `"cluster"`                                                            | Inventory group name for the cluster nodes               |
| vault_docker_registry            | `"index.docker.io"`                                                    | Registry from where to pull the docker image             |
| vault_docker_tag                 | `"latest"`                                                             | Tag of the docker image version                          |
| vault_docker_image               | `"{{ vault_docker_registry }}/hashicorp/vault:{{ vault_docker_tag }}"` | Docker image to use                                      |
| vault_unsealer_policy_name       | `"autounseal"`                                                         | Name of policy in unsealer vault                         |
| vault_unsealer_policy_path       | `"/tmp/{{ vault_unsealer_policy_name }}.hcl"`                          | Policy file location                                     |
| vault_unsealer_host              | `"{{ ansible_default_ipv4.address }}"`                                 | Unsealer node address                                    |
| vault_unsealer_storage_path      | `"/home/ubuntu/vault-data"`                                            | Data storage location                                    |
| vault_unsealer_config_path       | `"/home/ubuntu/config.hcl"`                                            | Config file location                                     |
| vault_unsealer_port              | `8200`                                                                 | Port number of the vault node for external communication |
| vault_unsealer_cluster_port      | `"{{ vault_unsealer_port + 1 }}"`                                      | Port number of the vault node for internal communication |
| vault_unsealer_default_lease_ttl | `"168h"`                                                               | Default TTL for secrets; in this case seven days         |
| vault_unsealer_max_lease_ttl     | `"720h"`                                                               | Maximum TTL for secrets; in this case 30 days            |
| vault_unsealer_tls_disable       | `"true"`                                                               | If the external communication should be secured by TLS   |
| vault_unsealer_enable_ui         | `"true"`                                                               | Enable (or disable) the included UI                      |
| vault_cluster_host               | `"{{ ansible_default_ipv4.address }}"`                                 | Cluster node address                                     |
| vault_cluster_storage_path       | `"/home/ubuntu/vault-data"`                                            | Config file location                                     |
| vault_cluster_config_path        | `"/home/ubuntu/config.hcl"`                                            | Policy file location                                     |
| vault_cluster_port               | `8200`                                                                 | Port number of the vault node for external communication |
| vault_cluster_cluster_port       | `"{{ vault_cluster_port + 1 }}"`                                       | Port number of the vault node for internal communication |
| vault_cluster_default_lease_ttl  | `"168h"`                                                               | Default TTL for secrets; in this case seven days         |
| vault_cluster_max_lease_ttl      | `"720h"`                                                               | Maximum TTL for secrets; in this case 30 days            |
| vault_cluster_tls_disable        | `"true"`                                                               | If the external communication should be secured by TLS   |
| vault_cluster_enable_ui          | `"true"`                                                               | Enable (or disable) the included UI                      |

Dependencies
------------

None.

Example Playbook
----------------

Below you'll find an example playbook. This role requires you to have the unsealer and the cluster in two different inventory groups. Based on the groups the cluster connections will be established. 

```yaml
- name: Setup vault cluster
  hosts: all
  pre_tasks:
    - name: install requirements
      ansible.builtin.apt:
        name:
          - apt-transport-https
          - docker.io
          - python3-docker 
      become: true
  roles:
    - name: vault cluster setup role
      role: hashicorp-vault-cluster-role
      vars:
        unsealer_host_group_name: "unsealer"
        cluster_hosts_group_name: "cluster"
        vault_unsealer_port: 8202  # trick to only use three VMs
```

Example inventory:

```ini
[unsealer]
10.0.0.10

[cluster]
10.0.0.10
10.0.0.11
10.0.0.12
```

License
-------

BSD
