hashicorp-vault-cluster-role
============================

This role creates for you a hashicorp vault cluster secured/unsealed by another vault.
It is not (yet) idempotent!

Requirements
------------

- You need to have docker installed
  - docker.io
  - python3-docker
  - apt-transport-https
- Four instances
  - One for the cluster unsealer
  - Three for the cluster

Role Variables
--------------

Watch out for the `true` values. They need to be strings, otherwise Jinja2 will template them to `True`,
which is invalid in **HCL**.

| Variable                         | Default value                                                          | Usage                                                    |
|----------------------------------|------------------------------------------------------------------------|----------------------------------------------------------|
| vault_docker_registry            | `"index.docker.io"`                                                    | Registry from where to pull the docker image             |
| vault_docker_tag                 | `"latest"`                                                             | Tag of the docker image version                          |
| vault_docker_image               | `"{{ vault_docker_registry }}/hashicorp/vault:{{ vault_docker_tag }}"` | Docker image to use                                      |
| vault_generic_policy_name        | `"autounseal"`                                                         | Name of policy in unsealer vault                         |
| vault_unsealer_inv_group_name    | `"unsealer"`                                                           | Inventory group name for the unseal vault node           |
| vault_unsealer_storage_path      | `"/home/ubuntu/vault-data"`                                            | Data storage location                                    |
| vault_unsealer_config_path       | `"/home/ubuntu/config.hcl"`                                            | Config file location                                     |
| vault_unsealer_policy_path       | `"/tmp/autounseal.hcl"`                                                | Policy file location                                     |
| vault_unsealer_port              | `8200`                                                                 | Port number of the vault node for external communication |
| vault_unsealer_cluster_port      | `"{{ vault_unsealer_port + 1 }}"`                                      | Port number of the vault node for internal communication |
| vault_unsealer_default_lease_ttl | `"168h"`                                                               | Default TTL for secrets; in this case seven days         |
| vault_unsealer_max_lease_ttl     | `"720h"`                                                               | Maximum TTL for secrets; in this case 30 days            |
| vault_unsealer_tls_disable       | `"true"`                                                               | If the external communication should be secured by TLS   |
| vault_unsealer_enable_ui         | `"true"`                                                               | Enable (or disable) the included UI                      |
| vault_cluster_inv_group_name     | `"cluster"`                                                            | Inventory group name for the cluster vault nodes         |
| vault_cluster_storage_path       | `"/home/ubuntu/vault-data"`                                            | Config file location                                     |
| vault_cluster_config_path        | `"/home/ubuntu/config.hcl"`                                            | Policy file location                                     |
| vault_cluster_port               | `8200`                                                                 | Port number of the vault node for external communication |
| vault_cluster_cluster_port       | `"{{ vault_cluster_port + 1 }}"`                                       | Port number of the vault node for internal communication |
| vault_cluster_default_lease_ttl  | `"168h"`                                                               | Default TTL for secrets; in this case seven days         |
| vault_cluster_max_lease_ttl      | `"720h"`                                                               | Maximum TTL for secrets; in this case 30 days            |
| vault_cluster_tls_disable        | `"true"`                                                               | If the external communication should be secured by TLS   |
| vault_cluster_enable_ui          | `"true"`                                                               | Enable (or disable) the included UI                      |
| vault_cluster_tls_skip_verify    | `"true"`                                                               | TLS verification of the unsealer vault is disabled       |

Dependencies
------------

None.

Example Playbook
----------------

Below you'll find an example playbook. This role requires you to have the unsealer and the cluster in two different inventory groups. Based on the groups the cluster connections will be established. 

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
            vault_unsealer_inv_group_name: unsealer
            vault_cluster_inv_group_name: cluster


License
-------

BSD
