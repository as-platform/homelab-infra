# homelab-infra

🇬🇧 [English version](README.md)

Ansible-код для развёртывания домашнего Kubernetes-кластера платформы [as-platform](https://github.com/as-platform) на [K3s](https://k3s.io).

## Структура

```text
ansible/
├── ansible.cfg
├── site.yml              # основной плейбук
├── requirements.yml      # коллекции Ansible
├── inventory/
│   ├── hosts.yml         # группы k3s_server / k3s_agent
│   └── group_vars/all.yml
└── roles/
    ├── common/           # базовая подготовка узла
    ├── k3s/              # установка и настройка K3s
    ├── helm/             # установка Helm
    └── registries/       # доступ узла к реестру Zot
```

## Роли

- **common** — пакеты, часовой пояс и NTP, модули ядра (`overlay`, `br_netfilter`), sysctl, отключение swap.
- **k3s** — установка K3s официальным скриптом (версия задаётся в `k3s_version`), конфиг `/etc/rancher/k3s/config.yaml`, отключены `traefik` и `servicelb`. После установки ждёт готовности узла и сохраняет kubeconfig в `~/.kube/config-k3s` и вливает его в `~/.kube/config` (остальные контексты не затрагиваются, `current-context` не меняется, перед записью создаётся бэкап).
- **helm** — ставит Helm в `/usr/local/bin` на узле control plane (версия задаётся в `helm_version`, архив проверяется по опубликованному sha256). Если эта версия уже установлена, ничего не происходит.
- **registries** — даёт узлу доступ к образам в Zot, на server и на agent: добавляет `zot.homelab` в `/etc/hosts`, читает CA из секрета `cert-manager/homelab-ca` кластера и сохраняет его в `/etc/rancher/k3s/zot-ca.crt`, пишет `/etc/rancher/k3s/registries.yaml` с учётными данными Zot и перезапускает K3s при изменениях. Учётные данные — переменные `registries_zot_user` и `registries_zot_password` в зашифрованном `ansible/inventory/group_vars/k3s_cluster/vault.yml`.

## Использование

```bash
cd ansible
ansible-galaxy collection install -r requirements.yml
ansible-playbook site.yml
```

Затем:

```bash
kubectx <k3s_context_name>
kubectl get nodes
```

Адреса узлов задаются в `ansible/inventory/hosts.yml`, параметры ролей — в `roles/*/defaults/main.yml`.

## Связанные репозитории

- [`k3s`](https://github.com/as-platform/k3s) — GitOps-репозиторий, который работает на кластере, созданном этим кодом

## Лицензия

MIT — см. [`LICENSE`](LICENSE).
