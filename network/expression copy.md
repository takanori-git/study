# まとめ

ネットワークには、大きく分けて以下の２つに分けられる。
- LAN  // 建物・フロア内の狭い範囲
- WAN  // LAN同士を接続する。インターネットも含む。

当ドキュメントではLANについて浅く広く簡潔に記載することを目指した。

## プロトコル
ネットワークを使って相互に通信するためにはプロトコル（規格）が必要。  
そのプロトコルをどのように構成していくのかを、標準化団体ISOが「OSI参照モデル」として提示した。  
一方で、現実に利用されているのは「TCP/IPプロトコル群」。  
https://www.sbbit.jp/article/cont1/12099?page=2

以下は全部覚える！

|レイヤ|OSI参照モデル|TCP/IPプロトコル群|実装プロトコルの例|
|--|--|--|--|
|7|アプリケーション層|アプリケーション層|HTTP,FTP,DHCP,TLS(※1)|
|6|プレゼンテーション層|〃|〃|
|5|セッション層|〃|〃|
|4|トランスポート層|トランスポート層|TCP,UDP|
|3|ネットワーク層|インターネット層|IP,ARP,ICMP|
|2|データリンク層|ネットワークインターフェース層|Ethernet|
|1|物理層|〃|〃|

## LANに必要な機器
- ホスト
- NIC（ネットワークインターフェースカード）
- ケーブル（ネットワーキングメディア）
    - 同軸ケーブル // 同軸ケーブル同士をつなげるにはトランシーバが必要
    - ツイストペア（UTP）
    - 光ファイバ
    
    |規格|ケーブル|トポロジー|伝送速度|利用頻度|
    |--|--|--|--|--|
    |10BASE5|同軸|バス|10Mbps|×|
    |10BASE2|同軸|スター|10Mbps|×|
    |10BASE-T|UTP|スター|10Mbps|×|
    |100BASE-TX|UTP|スター|100Mbps|〇|
    |1000BASE-T|UTP|スター|1Gbps|〇|
    |光ファイバは省略|||
    - Windowsならデバイスドライバで使用中の規格を確認できる。GBEなら1000BASE-T。
    https://support.eonet.jp/connect/net/1g/win10.html
- ネットワーキングデバイス
    |レイヤ|デバイス|ポート数|処理機構|利用頻度|コリジョンドメイン分割|ブロードキャストドメイン分割|備考|
    |--|--|--|--|--|--|--|--|
    |3|ルータ|N|ソフト|〇|〇|〇|WAN接続用|
    |〃|L3スイッチ|N|ハード|〇|〇|〇|LAN接続用|
    |2|L2スイッチ|N|ハード|〇|〇|×|〃|
    |〃|ブリッジ|1|ソフト|×|△|×|〃|
    |1|ハブ|N|ハード|×|×|×|〃|
    |〃|リピータ|1|ハード|×|×|×|〃|
    
    - ハブには内部にトランシーバが内臓されている。なので同軸ケーブル＋トランシーバの組み合わせは不要になりUTPのみ複線化できる。（バス -> スター型のトポロジ）

    [図解]　コリジョンドメインとブロードキャストドメイン
    ![networkdevice.jpg](img/networkdevice.jpg)

    https://www.atmarkit.co.jp/ait/articles/1503/12/news011.html

    https://www.cisco.com/c/m/ja_jp/meraki/documentation/ms/layer-3-switching/layer-3-versus-layer-2-switch-for-vlans.html
    https://japan.zdnet.com/article/35137199/

|#|プロトコル|レイヤ|概要|
|--|--|--|--|
|①|DHCP|7|<ul><li>ホストAから、L2スイッチ・ホストB、ルーターに向かってDHCPのブロードキャスト。（ポート67）</li><li>Aのipアドレスが確定</li></ul>|
|②|ARP|3||<ul><li>ホストAから、</ul></li>|



https://qiita.com/7of9/items/c2265191e30a8110c6e0
```
[vagrant@vagranthost ~]$ sudo grep -R "DHCPOFFER" /var/log/* | less -X
/var/log/anaconda/syslog:20:49:26,833 INFO dhclient:DHCPOFFER from 192.168.122.1
/var/log/anaconda/journal.log:Feb 28 20:49:26 localhost dhclient[2004]: DHCPOFFER from 192.168.122.1
/var/log/messages:May  4 13:38:57 vagranthost dhclient[2306]: DHCPOFFER from 10.0.2.2
/var/log/messages:May  4 13:38:59 vagranthost dhclient[2359]: DHCPOFFER from 192.168.0.1
・・・
/var/log/messages-20200502:May  1 18:35:08 vagranthost dhclient[5774]: DHCPOFFER from 192.168.0.1
/var/log/messages-20200502:May  1 18:35:08 vagranthost dhclient[5775]: DHCPOFFER from 10.0.2.2
/var/log/messages-20200502:May  2 13:48:19 vagranthost dhclient[2372]: DHCPOFFER from 192.168.0.1
/var/log/secure:May  4 14:49:20 vagranthost sudo: vagrant : TTY=pts/0 ; PWD=/home/vagrant ; USER=root ; COMMAND=/bin/grep -R DHCPOFFER /var/log/anaconda /var/log/audit /var/log/btmp /var/log/btmp-20200502 /var/log/chrony /var/log/cron /var/log/cron-20200502 /var/log/dmesg /var/log/dmesg.old /var/log/grubby_prune_debug /var/log/httpd /var/log/lastlog /var/log/maillog /var/log/maillog-20200502 /var/log/messages /var/log/messages-20200502 /var/log/qemu-ga /var/log/rhsm /var/log/samba /var/log/secure /var/log/secure-20200502 /var/log/spooler /var/log/spooler-20200502 /var/log/tallylog /var/log/tuned /var/log/vboxadd-install.log /var/log/vboxadd-setup.log /var/log/vboxadd-setup.log.1 /var/log/vboxadd-setup.log.2 /var/log/vboxadd-setup.log.3 /var/log/vboxadd-setup.log.4 /var/log/wtmp /var/log/yum.log

$ ll /var/lib/dhclient/
total 0

$ ll /var/run/dhclient-eth*
-rw-r--r--. 1 root root 5 May  4 14:32 /var/run/dhclient-eth0.pid
-rw-r--r--. 1 root root 5 May  4 14:32 /var/run/dhclient-eth2.pid

$ sudo su
# ll  /etc/dhcp/dhclient.d
total 4
-rwxr-xr-x. 1 root root 409 Apr 11  2018 chrony.sh

# ll  /etc/dhcp/dhclient-exit-hooks.d/
total 4
-rw-r--r--. 1 root root 594 Aug  6  2019 azure-cloud.sh

```

```sh
docker
[vagrant@vagranthost ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:26:10:60 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 85525sec preferred_lft 85525sec
    inet6 fe80::5054:ff:fe26:1060/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:db:96:81:8f brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
4: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:78:60:22 brd ff:ff:ff:ff:ff:ff
    inet 192.168.33.11/24 brd 192.168.33.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe78:6022/64 scope link
       valid_lft forever preferred_lft forever
5: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:6c:89:d3 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.13/24 brd 192.168.0.255 scope global noprefixroute dynamic eth2
       valid_lft 2726sec preferred_lft 2726sec
    inet6 fe80::a00:27ff:fe6c:89d3/64 scope link
       valid_lft forever preferred_lft forever

net
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:26:10:60 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 86340sec preferred_lft 86340sec
    inet6 fe80::5054:ff:fe26:1060/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:cb:3e:22 brd ff:ff:ff:ff:ff:ff
    inet 172.28.128.3/24 brd 172.28.128.255 scope global noprefixroute dynamic eth1
       valid_lft 1140sec preferred_lft 1140sec
    inet6 fe80::a00:27ff:fecb:3e22/64 scope link
       valid_lft forever preferred_lft forever
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:48:30:77 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.17/24 brd 192.168.0.255 scope global noprefixroute dynamic eth2
       valid_lft 3541sec preferred_lft 3541sec
    inet6 fe80::a00:27ff:fe48:3077/64 scope link
       valid_lft forever preferred_lft forever

docker
$ cat /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE="eth0"
BOOTPROTO="dhcp"
ONBOOT="yes"
TYPE="Ethernet"
PERSISTENT_DHCLIENT="yes"

$ cat /etc/sysconfig/network-scripts/ifcfg-eth1
#VAGRANT-BEGIN
# The contents below are automatically generated by Vagrant. Do not modify.
NM_CONTROLLED=yes
BOOTPROTO=none
ONBOOT=yes
IPADDR=192.168.33.11
NETMASK=255.255.255.0
DEVICE=eth1
PEERDNS=no
#VAGRANT-END

$ cat /etc/sysconfig/network-scripts/ifcfg-eth2
#VAGRANT-BEGIN
# The contents below are automatically generated by Vagrant. Do not modify.
BOOTPROTO=dhcp
ONBOOT=yes
DEVICE=eth2
NM_CONTROLLED=yes
#VAGRANT-END

net
$ cat /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE="eth0"
BOOTPROTO="dhcp"
ONBOOT="yes"
TYPE="Ethernet"
PERSISTENT_DHCLIENT="yes"

$ cat /etc/sysconfig/network-scripts/ifcfg-eth1
#VAGRANT-BEGIN
# The contents below are automatically generated by Vagrant. Do not modify.
BOOTPROTO=dhcp
ONBOOT=yes
DEVICE=eth1
NM_CONTROLLED=yes
#VAGRANT-END

$ cat /etc/sysconfig/network-scripts/ifcfg-eth2
#VAGRANT-BEGIN
# The contents below are automatically generated by Vagrant. Do not modify.
BOOTPROTO=dhcp
ONBOOT=yes
DEVICE=eth2
NM_CONTROLLED=yes
#VAGRANT-END

```
vagrant dhcp
https://qiita.com/centipede/items/64e8f7360d2086f4764f
https://qiita.com/jyokyoku/items/85b030537a5728c9d771

Vagrant ip 10.0.2.15/24
https://qiita.com/tmiki/items/d786edf221feb4cb1af5


virtualbox dns
http://zorinos.seesaa.net/article/450304938.html

resulv.conf
http://oretachino.hatenablog.com/entry/2014/12/25/151112


Vagrantでpublic_networkなし
-> resolv.confのnameserver 10.0.2.3
Vagrantでpublic_networkあり
-> resolv.confのnameserver ＝　Windowsの設定と同じ２つのIP


type: "dhcp"
https://qiita.com/jyokyoku/items/85b030537a5728c9d771


https://server.etutsplus.com/centos-7-net-tools-vs-iproute2/
```sh
$ ip neigh
10.0.2.2 dev eth0 lladdr 52:54:00:12:35:02 REACHABLE
10.0.2.3 dev eth0 lladdr 52:54:00:12:35:03 STALE
```