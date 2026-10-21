# homelab-infra

Ansible-код для развёртывания домашнего Kubernetes-кластера на [K3s](https://k3s.io).

## Структура

```
ansible/
├── ansible.cfg
├── site.yml              # основной плейбук
├── requirements.yml      # коллекции Ansible
├── inventory/
│   ├── hosts.yml         # группы k3s_server / k3s_agent
│   └── group_vars/all.yml
└── roles/
    ├── common/           # базовая подготовка узла
    └── k3s/              # установка и настройка K3s
```

## Роли

- **common** — пакеты, часовой пояс и NTP, модули ядра (`overlay`, `br_netfilter`), sysctl, отключение swap.
- **k3s** — установка K3s официальным скриптом (версия задаётся в `k3s_version`), конфиг `/etc/rancher/k3s/config.yaml`, отключены `traefik` и `servicelb`. После установки ждёт готовности узла и сохраняет kubeconfig локально в `~/.kube/config-k3s`.

## Использование

```bash
cd ansible
ansible-galaxy collection install -r requirements.yml
ansible-playbook site.yml
```

Затем:

```bash
export KUBECONFIG=~/.kube/config-k3s
kubectl get nodes
```

Адреса узлов задаются в `ansible/inventory/hosts.yml`, параметры ролей — в `roles/*/defaults/main.yml`.
