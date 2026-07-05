# Ansibleの実行環境

## 環境構築
### 事前にインストールするもの
- uv

### ansibleのインストール
```bash
uv venv
uv pip install ansible ansible-lint
```


## 使い方

### ログインできるのかの確認
```
uv run ansible -i config/inventory.ini all -m raw -a "hostname"
```

### 基本的なノードのセットアップ
```bash
uv run ansible-playbook -i config/inventory.ini -K playbooks/setup-nodes.yml
```

### バージョンアップすべきもの
- `roles/container/vars/main.yaml`
    - containerd
    - nerdctl
    - buildkit
    - cni_plugins

### Kubernetesのインストール
```bash
uv run ansible-playbook -i config/inventory.ini -K playbooks/setup-kubernetes.yml
```

### Kubernetesのアップグレード
デフォルトでは、アップグレード対象は1台だけに制限しています。
`--limit` で対象ノードを1台に絞って実行してください。

```bash
uv run ansible-playbook -i config/inventory.ini -K \
  -e kubernetes_upgrade_mode=true \
  -e kubernetes_version=1.34 \
  --limit prx-ubuntu-02 \
  playbooks/setup-kubernetes.yml
```

ダウンタイムを認めて同じ種類のノードを一度にアップグレードする場合は、
`kubernetes_upgrade_allow_multiple_nodes=true` を指定します。
control-plane と worker を同時に対象にすることは禁止しているため、
必ず `--limit kube_worker_node` のように片方のグループだけを指定してください。

```bash
uv run ansible-playbook -i config/inventory.ini -K \
  -e kubernetes_upgrade_mode=true \
  -e kubernetes_upgrade_allow_multiple_nodes=true \
  -e kubernetes_version=1.34 \
  --limit kube_worker_node \
  playbooks/setup-kubernetes.yml
```

複数 control-plane のアップグレードはまだ未対応です。

### Slurmのインストール
```bash
uv run ansible-playbook -i config/inventory.ini -K -e slurm_install_packages=true playbooks/setup-slurm.yml
```

### ansible-lint (CIと同じ手順)
```bash
uv run ansible-galaxy collection install -r requirements.yml
ANSIBLE_CONFIG=ansible.cfg uv run ansible-lint --strict -c .ansible-lint playbooks roles
```
