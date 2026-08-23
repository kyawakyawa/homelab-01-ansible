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
`chrony` をインストールして複数のNTP sourceを設定し、競合する
`systemd-timesyncd` を停止・無効化する。

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
  -e kubernetes_version=1.35 \
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
  -e kubernetes_version=1.35 \
  --limit kube_worker_node \
  playbooks/setup-kubernetes.yml
```

複数 control-plane のアップグレードはまだ未対応です。

### Raspberry Pi 4 Kubernetes workerのmicroSD書き込み抑制

`raspberrypi4-01` 専用のplaybookで、kubeletのコンテナログローテーション、
journaldの揮発化、rsyslogの停止、Podログ用tmpfsのfstab登録を行う。

```bash
uv run ansible-playbook -i config/inventory.ini -K \
  playbooks/setup-raspberrypi4-low-write.yml
```

tmpfsはその場でマウントせず、次回のOS再起動時に`/var/log/pods`へ
マウントする。既存ファイルの削除や移動は行わない。

この設定ではjournalは`/run`上に置かれ、OS再起動で失われる。rsyslogも停止・
無効化するため、通常の`/var/log/syslog`、`/var/log/auth.log`、
`/var/log/kern.log`は更新されなくなる。既存の永続journalやログファイルは
削除しない。tmpfsが容量またはinode上限に達すると、コンテナログの書き込みや
ログを出力するワークロードへ影響する可能性があるため、再起動後は使用量を
監視すること。`/var/log/containers`は別途マウントせず、通常のsymlink構成を保つ。

### Slurmのインストール
```bash
uv run ansible-playbook -i config/inventory.ini -K -e slurm_install_packages=true playbooks/setup-slurm.yml
```

### ansible-lint (CIと同じ手順)
```bash
uv run ansible-galaxy collection install -r requirements.yml
ANSIBLE_CONFIG=ansible.cfg uv run ansible-lint --strict -c .ansible-lint playbooks roles
```
