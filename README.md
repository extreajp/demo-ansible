# kachidoki-ansible

Ansible repository for provisioning and operating the Kachidoki on-prem cluster.

## Scope

This repository manages:

- kernel pinning and base OS tuning
- ROCm installation and user environment setup
- internal network assignment (netplan)
- K3s cluster deployment
- Longhorn SDS deployment (Multus + Whereabouts + NAD)
- AWS VPC static route configuration
- AMD GPU device plugin deployment
- initial Kubernetes namespaces

## Inventory

- Inventory file: `inventories/production/hosts.ini`
- Main host group: `ubuntu_servers`
- Primary control-plane node used by delegated tasks: `{{ k8s_delegate_host }}` (defined in `group_vars/all/common.yml`)

## Entry Point

- Main playbook: `site.yml`
- Included playbooks and tags:
  - `kernel_update.yml` -> `kernel_update`
  - `rocm_install.yml` -> `rocm_install`
  - `network_assign.yml` -> `network_assign`
  - `k3s_deploy.yml` -> `k3s_deploy`
  - `longhorn_sds_deploy.yml` -> `longhorn_sds_deploy`
  - `aws_route_deploy.yml` -> `aws_route_deploy`
  - `common_deploy.yml` -> `common_deploy`
  - `gpu_plugin_deploy.yml` -> `gpu_plugin_deploy`
  - `k3s_initial_deploy.yml` -> `k3s_initial_deploy`

## Prerequisites

- Ansible installed on the control machine
- Required collections/modules available (for example `kubernetes.core`)
- SSH access from control machine to inventory hosts
- Vault secret available for `group_vars/all/k3s_tokens.yml` (contains `k3s_token`)

## Validation

Run syntax validation before execution:

```bash
ansible-playbook -i inventories/production/hosts.ini site.yml --syntax-check
```

Check available tags:

```bash
ansible-playbook -i inventories/production/hosts.ini site.yml --list-tags
```

## Execution Examples

Run all playbooks:

```bash
ansible-playbook -i inventories/production/hosts.ini site.yml
```

Run only K3s deployment:

```bash
ansible-playbook -i inventories/production/hosts.ini site.yml --tags k3s_deploy
```

Run only Longhorn SDS deployment:

```bash
ansible-playbook -i inventories/production/hosts.ini site.yml --tags longhorn_sds_deploy
```

## Notes

- In `longhorn_sds_deploy`, many Kubernetes operations run with `run_once` and are delegated to `{{ k8s_delegate_host }}`.
- `roles/longhorn_sds_setup` includes a task to remove the default flag from `local-path` StorageClass.
