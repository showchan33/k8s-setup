# 本ツールの概要

Ansibleとkubeadmを使って、LinuxマシンにKubernetesのcontrol planeとworker nodeを構築します。<br>
構築を行うAnsible Playbookは、下の図のようにKubernetesを構築するマシンとは別のマシンから実行します。

![](./doc/k8s-setup.png)

# 必要条件

## Ansibleを実行するマシン

* OS
    * Linux（動作確認OSはUbuntu 20.04のみ）
* ツール
    * Ansible

## Kubernetesを構築する各ノード

* OS
    * Ubuntu（動作確認OSはUbuntu 22.04のみ）
* ツール
    * Python3

# 事前準備

## SSH接続用の公開鍵／秘密鍵を準備

Kubernetesを構築する各ノードにSSHで接続するための公開鍵と秘密鍵のペアを用意します。<br>
公開鍵は、各ノードの ``/home/[username]/.ssh/authorized_keys``に追加し、例えば以下のコマンドでSSHで接続できる状態にしておきます。

```
ssh -i [秘密鍵ファイル名] [username]@[hostname]
```

## 設定ファイルの準備
以下のように、``*.sample``ファイルから末尾の``.sample``を取り除いたファイルをコピーして作成します。

```
cp files/sshkey.sample files/sshkey
cp group_vars/all.yml.sample group_vars/all.yml
cp hosts.sample hosts
cp host_vars/control-plane1.yml.sample host_vars/control-plane1.yml
cp host_vars/worker-node1.yml.sample host_vars/worker-node1.yml
(作成したいワーカーノードの数だけコピー)
```

各ファイルの内容を環境に応じて編集します。

### files/sshkey

SSH接続に使う秘密鍵を格納します。以下の形式になります。
```
-----BEGIN OPENSSH PRIVATE KEY-----
(秘密鍵がBASE64エンコードされたテキストを格納)
-----END OPENSSH PRIVATE KEY-----
```

### group_vars/all.yml

全てのホスト(ノード)で共通に使う変数を定義します。
```
ansible_user: user                              # リモートホストのユーザ名を記載
ansible_ssh_private_key_file: "./files/sshkey"  # 秘密鍵ファイルのパスを指定

file_kubeadm_join_command: "./kubeadm_join_command.tmp" # 編集不要
```

### hosts

機能別にホストをグループ化して、各ホストの名前を定義します。<br>
各グループに記載されたホストの変数は後述の ``host_vars/[hostname].yml`` で定義します。
```
[control-plane] # control planeのホストグループ
control-plane1

[worker-node]   # worker nodeのホストグループ
worker-node1
(作成したいワーカーノードの数だけ記載)
```
### host_vars/control-plane1.yml

control planeで使う変数を定義します。<br>

```
ansible_host: control-plane1.example.com
# node_ip: 10.0.0.4
# kubeadm:
#   pod_network_cidr: 192.168.0.0/16
#   service_cidr: 10.96.0.0/12
#   apiserver_advertise_address: "{{ node_ip }}"
```

* 必須
    * ansible_host : SSH接続する際のホスト名（IPアドレスも可）を記載
* オプショナル
    * node_ip : Kubernetesのノードとして使う場合のIPアドレスを記載。主に``ansible_host``と異なるIPアドレスを使いたい場合に使用。例えばノードのプライベートIP等を記載
    * kubeadm配下 : PodやServiceのCIDRをデフォルトとは異なる値にしたい場合等に記載

### host_vars/worker-node1.yml

worker nodeで使う変数を定義します。<br>

```
ansible_host: worker-node1.example.com
# node_ip: 10.0.0.5
```

* 必須
    * ansible_host : SSH接続する際のホスト名（IPアドレスも可）を記載
* オプショナル
    * node_ip : Kubernetesのノードとして使う場合のIPアドレスを記載

# 実行コマンド

## control planeの構築

以下のコマンドを実行します。
```
ansible-playbook site_control-plane.yml
```

## worker nodeの構築

control planeの構築完了後に、以下のコマンドを実行します。
```
ansible-playbook site_worker.yml
```

# 注意事項、制約事項

```
$ kubectl version
Client Version: v1.28.2
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.28.5

$ kubelet --version
Kubernetes v1.28.2

$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"28", GitVersion:"v1.28.2", GitCommit:"89a4ea3e1e4ddd7f7572286090359983e0387b2f", GitTreeState:"clean", BuildDate:"2023-09-13T09:34:32Z", GoVersion:"go1.20.8", Compiler:"gc", Platform:"linux/amd64"}
```

# Author
showchan33

# License
"k8s-setup" is under [GPL license](https://www.gnu.org/licenses/licenses.en.html).
 
