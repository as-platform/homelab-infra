# homelab-infra

🇷🇺 [Русская версия](README_RUS.md)

Ansible code for deploying a home Kubernetes cluster on [K3s](https://k3s.io).

## Structure

```
ansible/
├── ansible.cfg
├── site.yml              # main playbook
├── requirements.yml      # Ansible collections
├── inventory/
│   ├── hosts.yml         # k3s_server / k3s_agent groups
│   └── group_vars/all.yml
└── roles/
    ├── common/           # basic node preparation
    └── k3s/              # K3s installation and configuration
```

## Roles

- **common** — packages, time zone and NTP, kernel modules (`overlay`, `br_netfilter`), sysctl, swap disabled.
- **k3s** — installs K3s with the official script (the version is set in `k3s_version`), writes the config `/etc/rancher/k3s/config.yaml`, disables `traefik` and `servicelb`. After installation it waits for the node to become ready, saves the kubeconfig to `~/.kube/config-k3s` and merges it into `~/.kube/config` (other contexts are untouched, `current-context` does not change, and a backup is created before writing).

## Usage

```bash
cd ansible
ansible-galaxy collection install -r requirements.yml
ansible-playbook site.yml
```

Then:

```bash
kubectx <k3s_context_name>
kubectl get nodes
```

Node addresses are set in `ansible/inventory/hosts.yml`, role parameters in `roles/*/defaults/main.yml`.
