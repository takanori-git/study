# 2. Docker環境構築

- [2-1. 環境](#a1)
- [2-2. Vagrantの設定](#a2)
- [2-3. Dockerのインストール](#a3)
- [2-4. ユーザーの編集](#a4)
- [2-5. 起動関連の設定](#a5)
- [2-6. Docker Compose](#a6)

<span id="a1">

## 2-1. 環境
- Windows10 Home 1909 // ホストOS
- Oracle VM VirtualBox 6.0.12
- Vagrant 2.2.5
- CentOS7 // ゲストOS

WindowsのVirtualBox上に構築したCentOS7にDockerをインストールすることにする。  
  - Windows10 Homeなので、Windows上に立てるとなるとレガシー扱いのDocker Toolboxを使わないといけないし、Docker Machineの知識も必要になり面倒なため

また、疎通確認のため使用するWSLをインストールしておくこと。
- https://qiita.com/Aruneko/items/c79810b0b015bebf30bb

<span id="a2">

## 2-2. Vagrantの設定
```sh
$ mkdir /c/vagrant/CentOS7_docker/
$ cd /c/vagrant/CentOS7_docker/

$ vi Vagrantfile
Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.hostname = "vagranthost"
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "private_network", ip: "192.168.33.11"
  config.vm.network "public_network"
  config.vm.synced_folder ".", "/vagrant_data"

  config.vm.provider "virtualbox" do |vb|
    vb.gui = true
  end

  config.vm.provision "shell", inline: <<-SHELL
    sudo ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
  SHELL
end

$ vagrant up

$ vagrant ssh

$ uname -a
Linux vagranthost 3.10.0-957.5.1.el7.x86_64 #1 SMP Fri Feb 1 14:54:57 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux

$ cat /etc/redhat-release
CentOS Linux release 7.7.1908 (Core)
```

<span id="a3">

## 2-3. Dockerのインストール
以降は公式ドキュメントに沿ってセットアップしていく。

```sh
# パッケージマネージャ
$ sudo yum install -y yum-utils
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# インストール
$ sudo yum install docker-ce docker-ce-cli containerd.io
#$ yum list docker-ce --showduplicates | sort -r | less

$ docker -v
Docker version 19.03.8, build afacb8b

$ sudo systemctl start docker
$ sudo docker run hello-world
Hello from Docker!
This message shows that your installation appears to be working correctly.
・・・
```
参考：https://docs.docker.com/engine/install/centos/
<span id="a4">

## 2-4. ユーザーの編集
sudoなしでdockerコマンドを使えるようにする。
```sh
$ echo $USER
vagrant

# ユーザーの情報を確認
$ id
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

# dockerグループの追加（既にあるっぽい）
$ sudo groupadd docker
groupadd: group 'docker' already exists

# vagrantユーザのセカンダリグループとして（-G）、dockerグループを追加する（-a）。
# https://www.atmarkit.co.jp/ait/articles/1612/14/news022.html
$ sudo usermod -aG docker $USER

# まだ反映されていないのでpermission denied
$ id
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

# まだ反映されていない
$ docker ps
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.40/containers/json: dial unix /var/run/docker.sock: connect: permission denied

# sudoつければもちろんok
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

$ exit
> vagrant ssh

# 反映された
$ id
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant),992(docker) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

# ok
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```
参考：https://docs.docker.com/engine/install/linux-postinstall/

<span id="a5">

## 2-5. 起動関連の設定
```sh
$ sudo systemctl enable docker
$ sudo chkconfig docker on
```

<span id="a5">

## 2-6. Docker Compose
```sh
# 前提：Docker Engineをインストール済み

$ sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose

$ docker-compose -v
docker-compose version 1.25.5, build 8a1c60f6
```
参考：https://docs.docker.com/compose/install/