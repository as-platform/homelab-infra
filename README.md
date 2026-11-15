# homelab-infra

🇷🇺 [Русская версия](README_RUS.md)

Ansible code for deploying the home Kubernetes cluster of the [as-platform](https://github.com/as-platform) homelab platform on [K3s](https://k3s.io).

## Structure

```text
ansible/
├── ansible.cfg
├── site.yml              # main playbook
├── requirements.yml      # Ansible collections
├── inventory/
│   ├── hosts.yml         # k3s_server / k3s_agent groups
│   └── group_vars/all.yml
└── roles/
    ├── common/           # basic node preparation
    ├── k3s/              # K3s installation and configuration
    ├── helm/             # Helm installation
    └── registries/       # access of the node to the Zot registry
```

## Roles

- **common** — packages, time zone and NTP, kernel modules (`overlay`, `br_netfilter`), sysctl, swap disabled.
- **k3s** — installs K3s with the official script (the version is set in `k3s_version`), writes the config `/etc/rancher/k3s/config.yaml`, disables `traefik` and `servicelb`. After installation it waits for the node to become ready, saves the kubeconfig to `~/.kube/config-k3s` and merges it into `~/.kube/config` (other contexts are untouched, `current-context` does not change, and a backup is created before writing).
- **helm** — installs Helm into `/usr/local/bin` on the control plane node (the version is set in `helm_version`, the archive is checked against its published sha256). Nothing happens when that version is already installed.
- **registries** — lets the node pull images from Zot, on the server and on the agent: adds `zot.homelab` to `/etc/hosts`, reads the CA from the secret `cert-manager/homelab-ca` of the cluster and saves it to `/etc/rancher/k3s/zot-ca.crt`, writes `/etc/rancher/k3s/registries.yaml` with the Zot credentials and restarts K3s when something changes. The credentials are the variables `registries_zot_user` and `registries_zot_password`, kept in the encrypted `ansible/inventory/group_vars/k3s_cluster/vault.yml`.

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

## Related repositories

- [`k3s`](https://github.com/as-platform/k3s) — the GitOps repository that runs on the cluster this code creates

## License

MIT — see [`LICENSE`](LICENSE).
