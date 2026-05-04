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

### Slurmのインストール
```bash
uv run ansible-playbook -i config/inventory.ini -K -e slurm_install_packages=true playbooks/setup-slurm.yml
```

### ansible-lint (CIと同じ手順)
```bash
uv run ansible-galaxy collection install -r requirements.yml
ANSIBLE_CONFIG=ansible.cfg uv run ansible-lint --strict -c .ansible-lint playbooks roles
```
