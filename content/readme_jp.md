# AI学習環境構築用Ansibleスクリプト説明

<div align=right>
作成日: 2020/12/21 <br>
作成者: 佐久間
</div>

## 前書き
本ドキュメントは、AI学習環境構築用に作成したAnsibleのスクリプトを使用・修正等の用途で用いる者を対象に作成したものです。

## Revision情報
- 2020/12/21 Rev.0: 初版作成

## 既知の問題・制約
- rkeを使用したKubernetes v1.18のデプロイに失敗する（rke upは完了するが、全てのPodが立ち上がっていない）。
- 設定ファイルの生成部分がMaster x1 Worker x1固定（将来Versionで修正予定）。
  - rke
  - prometheus
  - hosts(dnsmasq稼働サーバ)
- 管理ノードにKubernetesのWorkerノードの属性を持たせられない。正確にはrke上で管理ノードにWorkerのRoleを割り当てると、iptablesの設定がHarborと競合する。

## 前提条件
Ansibleはクラスタ外のPC（会社PC）を使用することを前提に作成しています。もし異なる環境であったとしても、以下の前提条件と同様にAnsibleと周辺パッケージをインストールできるのであれば問題ありません。
作成者は、会社PCにWSL(Windows Subsystem for Linux)をインストールし、その上にAnsibleと関連パッケージをインストールして使用しています。

会社のPCにWSLをインストールすること自体には問題はありませんが、Microsoft Storeアプリの仕様で社内のインストールに失敗します。社外のネットワークに接続して、Ubuntu 18.04ないしUbuntu 20.04をインストールしてください。

Ansibleの使用環境が準備出来たら、追加で以下の準備を行ってください。
- 構築対象のノードでのSSHアクセス（今回はパスワードでのログインを前提にしています）
  - Ubuntuの場合は`sudo apt install openssh-server`です（Ubuntu serverの場合はインストールプロセスでSSHログインを有効にするか聞かれるので、そこで有効にしている場合は必要ありません）。
  - CentOSの場合、通常はデフォルトでSSHログインできるので、インストールは必要ありませんが、後から削除されたなどの理由でSSHが使えない場合は以下のコマンドでインストールしてください。
    yum install openssh-server
- Python3環境の準備
  - Ubuntu 18.04、CentOS 7以降であればプリインストールされているはずなので、インストールは不要（のはず）です。
- Ansibleのインストール。上記のWSLでUbuntuを使用していることを前提条件として、以下のコマンドでインストールができます。単にsudo apt install ansibleとすると古いVersionがインストールされるかもしれないので注意。（Versionが古いと一部モジュールが動かない可能性がある）
```
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
```
  また、Ansible自体はPython2を前提にしているので、この点は注意。（現状のUbuntu, CentOSにはPython2が含まれているが将来的に削除される可能性有）

- Ansibleのadd-onパッケージのインストール。今回作成したAnsibleはビルトイン以外のパッケージも使用しているので、以下のコマンドでパッケージを追加する必要がある（Ansibleの実行PCのみにインストールで良い）

```
ansible-galaxy collection install community.general
ansible-galaxy collection install ansible.posix
ansible-galaxy collection install community.grafana
```

※community.grafanaが入っているとcommunity.generalのインストールには失敗（既に入っているので失敗）と出るかもしれない。

**_また、Ansibleの実行PCからは少なくとも一回各ノードにログインする必要がある。これは初回ログイン時のQ&AをAnsibleがさばけないため。_**

実行テストを行ったAnsibleのVersionは以下の通り、これ以降のVersionでの実行を推奨。将来ずっと動作が保証できるわけではない（将来のVersionで削除されるかもしれないので）
```
$ ansible --version
ansible 2.9.15
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/konata/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.17 (default, Sep 30 2020, 13:38:04) [GCC 7.5.0]
```

## クラスタ構築の実行
上記の準備が出来れば基本的に以下のコマンドでクラスタ構築を実行できる。パッケージ内のsite.ymlを設置したディレクトリでコマンド実行すればよい。ただし、デプロイ対象のIP等は別途設定する必要がある（後述する）。
`
ansible-playbook --inventory inventories/build/hosts site.yml
`
## Ansibleのディレクトリ構成
ディレクトリ構成と簡単な説明は以下の通り。

- root: "site.yml" を含むディレクトリ、スクリプトの最上位ディレクトリという意味でrootと書いている。その他に各roleの.ymlファイルを持つ (e.g. dnsmasq.yml)
  - inventories: ホスト情報、変数を格納する。
    - build: クラスタ構築時に使用する情報を格納するディレクトリ。hostsファイルが含まれる。
      - group_var:
        "all.yml"にクラスタ情報・設定項目の全てを含む。
    - maintenance: クラスタのアップデート時に使用する予定
  - roles: 下位ディレクトリにrole名に対応したディレクトリをもつ。それらはさらにtasks, files, templatesの子ディレクトリを持つ。
    - <task name>(e.g. dnsmasq)
      - tasks: 基本的にはmain.ymlのみ持つ。roleに対応した具体的な操作が記述されている。
      - files: このロールの操作時にリモートサーバにコピーするファイル
      - templates: <output filename>.j2という名称のファイルを置く。Ansibleのテンプレート、フィルタ機能を使用して、変数を加工して各サーバに必要なファイルを生成して、リモートサーバに設置する。

## hosts
現在、"inventory/build"に置かれている。ノード、グループの情報を記述している。 [mgmt_node] と [compute_node] はノード情報を格納する特別なグループとして扱っている（Ansible上特別なわけではない）。ホストIPアドレス（SSHでログインする際に使用するアドレス）とエイリアス名が設定されている。例えば[mgmt_node]を見ると、

```
[mgmt_node]
mgmt01 ansible_host=192.168.100.189

[mgmt_node:vars]
ansible_ssh_user=mgmt01
ansible_ssh_pass=Test123
ansible_ssh_port=22
ansible_sudo_pass=Test123
ansible_python_interpreter=/usr/bin/python3
```

となっており、これはSSHログインアドレスが192.168.100.189、hosts上のエイリアス名としてmgmt01を設定している。また、その直後に[mgmt_node:var]としてSSHのログインユーザ名等を設定している。また、ここで指定している変数は、Ansibleのスクリプトからも参照可能である（具体的にkubernetes_node_setupの"Add ssh user to "docker" group"で参照している）。エイリアス名は、各roleの割り当てに使用している。

例えば、
```
[kubernetes_node_setup]
mgmt01
compute01
```
はkubernetes_node_setupにある処理（設定）をmgmt01、compute01に対して行うということになる。これらの設定対象を変更・追加することでサーバ構成を変更できる。

セキュリティ上の理由で、ここの変数の格納方法はAnsible Vaultに変更になるかもしれない。

## all.yml
"inventory/build/group_var"に設置。
前述した通り、構築に関係する設定項目はここにすべて含まれると思ってよい。IPアドレス等の情報は一部hostsと重複するが将来的には統合されるかもしれない。

## スクリプトの実行フロー
Ansibleに慣れていない場合、実行フローがよく分からないと思うので、実行コマンドから順を追って説明する。まず、
“ansible-playbook --inventory inventories/build/hosts site.yml” は2つのファイルを参照している。一つは前述したhostsファイルで、もう一つはsite.ymlである。
hostsには先ほど説明した通り、設定対象のノードと、設定グループごとの設定ノードを記述している。 site.ymlでは実際の設定内容を記述していくが、下記に示すようにsite.yml自体には具体的な設定は記載されておらず、各タスクごとに区分けしたファイルを参照しているのみである。

```
---
- import_playbook: essential_packages.yml
- import_playbook: dnsmasq.yml
- import_playbook: docker.yml
- import_playbook: node_exporter.yml
- import_playbook: gpu_metrics.yml
- import_playbook: prometheus.yml
- import_playbook: grafana.yml
- import_playbook: nfs_server.yml
- import_playbook: nfs_client.yml
- import_playbook: harbor.yml
- import_playbook: hugo.yml
- import_playbook: kubectl_helm.yml
- import_playbook: gitlab.yml
- import_playbook: kubernetes_node_setup.yml
- import_playbook: rke.yml

```
Ansibleはこのファイルに書かれた順番でtaskを実行していく。したがって、site.ymlではタスクの実行順を管理しているといえる。
さらに、これらの参照されているファイル、例えばdnsmasq.ymlを見てみると以下の通り。

```
---
- hosts: dnsmasq
  become: true
  roles:
    - dnsmasq
```

ここには3つの要素がある。
"hosts"は前述のhostsファイルで定義されているグループを指定する（ノードを個別に指定することもできるが、それをするとグループ化の意味がない）。この例では、hosts内での[dnsmasq]グループに指定されているmgmt01が設定の対象になる。"roles"ではrolesの下に設置したディレクトリ名を指定する。ただ、hostsのグループ名とrolesのグループ名同じにしておいた方が分かりやすいと思う。上記の例の場合、roleとして、"dnsmasq/tasks"のmain.ymlを参照することになる。
最後に、"become:"は操作時の権限を指定する。become: trueはrootユーザとして実行することを意味する（sudoとは異なり、実際にrootになっている。Ubuntuの場合はsudo suしていることになる）。これは環境変数やhomeディレクトリがsshユーザと変わることを意味するので注意。例えばwhoamiを実行してもsshユーザ名は返ってこない。ファイルの実行権限等に影響が出るため要注意。
　今回の操作の大半は"become: true"でやったほうが手っ取り早い（というかrootでないと操作できないディレクトリなどを多く含む）ので、site.ymlでインポートしているほとんどの.ymlファイルで"become: true"を使用している。ただし、rke.ymlだけはrootとして操作すると問題が発生するので、become: trueを取り除いている。

次に、"roles/dnsmasq/tasks/main.yml"の中身を見てみよう。各操作は- name: xxxから始まる。その次の行がAnsibleのモジュール名、インデントされた部分はモジュールに与えるパラメータである。最後にwhenやwith_fileとあるのは条件分岐や、入力として与えているファイルである。

この例では、DNSサーバが使用するhostsファイル、DNSサーバ（dnsmasq）のインストール、dnsmasqの設定ファイルの編集（設定の追加）、サービスの再起動を行っている。

```
---
- name: Add host information onto hosts file on target
  blockinfile: 
    path: /etc/hosts 
    create: yes
    insertafter: EOF
    marker: "# {mark} Host information of management server and worker"
    block: "{{item}}"
  with_file:
    - files/hosts

- name: Install dnsmasq on terget (ubuntu)
  apt:
    deb: http://archive.ubuntu.com/ubuntu/pool/universe/d/dnsmasq/dnsmasq_{{ dnsmasq_version_ubuntu }}_all.deb
  when: ansible_distribution == 'Ubuntu'

- name: Install dnsmasq on terget (CentOS)
  yum:
    name: http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/{{ dnsmasq_version_centos}}.el8_2.1.x86_64.rpm
    state: present
  when: ansible_distribution == 'CentOS'

- name: Modify dnsmasq configuration
  blockinfile: 
    path: /etc/dnsmasq.conf
    create: yes
    insertafter: EOF
    marker: "# {mark} Add dnsmasq configuration as a local dns server"
    block: "{{item}}"
  with_file:
    - files/dnsmasq_conf

- name: Restart dnsmasq service
  service:
    name: dnsmasq
    state: restarted
```


## このAnsibleスクリプトで使用しているモジュール
今回は、以下のモジュールを使用した。基本的にはansible <モジュール名>で検索すればRedhatのドキュメントにたどり着くかと思う。
- blockinfile
- apt
- yum
- service
- get_url
- file
- unarchive
- copy
- shell
- template
- community.docker_compose
- posix.firewalld
- community.grafana_dashboard

上記のうち、
- community.docker_compose
- posix.firewalld
- community.grafana
はビルトイン（組み込み済み）のモジュールではない。これはansible-galaxyで事前にインポートしておく必要がある。手順は（前提条件）の項を参照。

## Application (roles)
今回作成したスクリプトでは、以下のroleを定義している。モジュール間の依存関係を維持していれば順番を変えることも可能である。
 - essential_packages: リモートノードのローカルのアプリケーションとして必須に近い（複数のモジュールで前提条件とする）アプリケーションのインストール、設定。現状はcurlとpython3-pipのみインストールしている。
 - dnsmasq: DNSサーバソフトウェア
 - docker: dockerとdocker composeのインストール。ansibleからdocker_composeを実行する際にはpip3のdocker-composeも必要となるため、ここでインストールしている。
 - node_exporter: Prometheusが配布しているnode_exporterのインストールを行う。また、Linuxのサービスとして登録している。筆者が別環境で試した際には単にバイナリをバックグラウンド実行させていたが、かなりの頻度でダウンしていたため、サービスとして運用することにした。監視対象の全てのノードを指定。
 - gpu_metrics: GPU固有の設定を行う（GPUのmetrics生成や/etc/docker/daemon.jsonの変更など）。 
 - prometheus: Prometheusのインストールと、設定ファイル上書き、サービス登録。
 - grafana: grafana-serverのインストールとdatasource(prometheus)およびdashboardの設定
 - nfs_server: nfs server softwareのインストール
 - nfs_client: nfs client softwareのインストールとマウントに関する設定。 "nfs_server"を先に実行する必要がある。
 - harbor: プライベートレジストリのHarborのインストール。"docker"の実行が前提条件。
 - hugo: 静的サイトジェネレータのhugoのインストール
 - kubectl_helm: kubectlとhelmのインストール。bash completionも合わせて実施している。
 - gitlab: gitlab-ceのインストール。"docker"の実行が前提条件
 - kubernetes_node_setup: kubernetes master/workerに関連する設定。現状はrke前提の設定を行っている。
 - rke: rkeのインストールとkubernetesクラスタのデプロイ.単に起動する所までをフォローしているので、さらに追加の設定等が必要になる。"kubernetes_node_setup"の実行が前提。

## Github上のアプリケーションのversion情報の入手方法(Tips)
無論、Webブラウザでちまちま見に行ってもよいが、かなり面倒なので、以下のコンソールアプリ（github-cli）のインストールを推奨する。
github-cliダウンロードサイト：https://github.com/cli/cli
使用に当たってはgithubへのログインが必要になるので、アカウント登録しておくこと。
ログインは以下のコマンドで行う。
`gh auth login`
API keyの出し方などはヘルプやwebサイト参照。
ログイン後、以下のコマンドversion（Release）情報を確認できる。

`gh release list --repo <repository name>`

今回の例では以下のようなアプリケーションのversion情報を確認できる。
- rancher/rancher
- prometheus/node_exporter
- prometheus/prometheus
- helm/helm
- goharbor/harbor
- kubernetes/kubernetes
- gohugoio/hugo
- docker/compose
ただし、
- kubernetes/kubectl
はrelease情報が取得できなかった。ただし、kubectlのversionはkubernetesのversionと揃えることが推奨されており、またその運用で現状は問題なさそうに見えている。

ちなみに実行すると以下のような結果が返ってくる。

```
konata@HD11658B:~$ gh release list --repo rancher/rancher
v2.5.4-rc6       Pre-release  (v2.5.4-rc6)   about 1 day ago
v2.5.4-rc5       Pre-release  (v2.5.4-rc5)   about 2 days ago
v2.5.4-rc4       Pre-release  (v2.5.4-rc4)   about 6 days ago
v2.5.4-rc3       Pre-release  (v2.5.4-rc3)   about 8 days ago
v2.5.4-rc2       Pre-release  (v2.5.4-rc2)   about 9 days ago
v2.5.4-rc1       Pre-release  (v2.5.4-rc1)   about 14 days ago
Release v2.4.11  Latest       (v2.4.11)      about 17 days ago
v2.4.11-rc6      Pre-release  (v2.4.11-rc6)  about 17 days ago
v2.4.11-rc5      Pre-release  (v2.4.11-rc5)  about 17 days ago
v2.4.11-rc4      Pre-release  (v2.4.11-rc4)  about 17 days ago
v2.4.11-rc3      Pre-release  (v2.4.11-rc3)  about 19 days ago
v2.4.11-rc2      Pre-release  (v2.4.11-rc2)  about 24 days ago
Release v2.5.3                (v2.5.3)       about 17 days ago
v2.5.3-rc1       Pre-release  (v2.5.3-rc1)   about 26 days ago
v2.4.11-rc1      Pre-release  (v2.4.11-rc1)  about 28 days ago
Release v2.5.2                (v2.5.2)       about 1 month ago
v2.5.2-rc10      Pre-release  (v2.5.2-rc10)  about 1 month ago
v2.4.10                       (v2.4.10)      about 1 month ago
v2.4.10-rc1      Pre-release  (v2.4.10-rc1)  about 1 month ago
v2.4.9                        (v2.4.9)       about 1 month ago
v2.4.9-rc12      Pre-release  (v2.4.9-rc12)  about 1 month ago
v2.5.2-rc9       Pre-release  (v2.5.2-rc9)   about 1 month ago
v2.5.2-rc8       Pre-release  (v2.5.2-rc8)   about 1 month ago
v2.4.9-rc11      Pre-release  (v2.4.9-rc11)  about 1 month ago
v2.4.9-rc10      Pre-release  (v2.4.9-rc10)  about 1 month ago
v2.5.2-rc7       Pre-release  (v2.5.2-rc7)   about 1 month ago
v2.5.2-rc6       Pre-release  (v2.5.2-rc6)   about 1 month ago
v2.5.2-rc5       Pre-release  (v2.5.2-rc5)   about 1 month ago
v2.5.2-rc4       Pre-release  (v2.5.2-rc4)   about 1 month ago
v2.4.9-rc9       Pre-release  (v2.4.9-rc9)   about 1 month ago


A new release of gh is available: 1.3.0 → v1.4.0
https://github.com/cli/cli/releases/tag/v1.4.0
```