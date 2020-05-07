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

- 基本  
送信するには以下の４つのアドレスが必要。
   - 送信元MACアドレス（L2） // NICに初期登録されている
   - 宛先MACアドレス（L2） // ARPテーブルから取得（なければARP要求）
   - 送信元IPアドレス（L3） // 静的：手動、動的：DHCPより取得
   - 宛先IPアドレス（L3） // アプリケーションで指定

   ![network-カプセル化.jpg](img/network-カプセル化.jpg)

- ホストAがホストBにPingを送る。
   |#|概要|プロトコル|レイヤ|説明|
   |--|--|--|--|--|
   |①|送信元IPアドレスの取得|DHCP|7~1|<ul><li>ホストAから、L2スイッチ配下にブロードキャスト。（ポート67）</li><li>（詳細は省略）</li><li>Aのipアドレスが確定</li></ul>|
   |②|宛先IPアドレスの取得|DNS|7~1|<ul><li>/etc/resolve.confのnameserverに指定してあるスタブリゾルバ（DNSサーバー）に送信先ドメイン名のIPアドレスを要求</li><li>スタブリゾルバのキャッシュに対象のドメイン名が残っていれば、そのドメイン名に該当するIPアドレスを返す。</li></ul>|
   |③|宛先MACアドレスの取得|ARP|3~1|<ul><li>ARPテーブルにホストBのMACアドレスが登録済みか確認</li><li>なければ、ホストAからL2スイッチ配下にARP要求をブロードキャストし、BのMACアドレスを取得しARPテーブルに登録</li></ul>|
   |④|Ping|ICMP|3~1|<ul><li>A->BにPingを送信</li></ul>|

- ホストAがホストCにHTTPアクセスする。
   

# 実験
https://qiita.com/jyokyoku/items/85b030537a5728c9d771
```
$ cat Vagrantfile
config.vm.network "private_network", type: "dhcp"
# publicはなし

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
       valid_lft 85709sec preferred_lft 85709sec
    inet6 fe80::5054:ff:fe26:1060/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:cb:3e:22 brd ff:ff:ff:ff:ff:ff
    inet 172.28.128.3/24 brd 172.28.128.255 scope global noprefixroute dynamic eth1
       valid_lft 1031sec preferred_lft 1031sec
    inet6 fe80::a00:27ff:fecb:3e22/64 scope link
       valid_lft forever preferred_lft forever

$ ip neigh
10.0.2.2 dev eth0 lladdr 52:54:00:12:35:02 REACHABLE
10.0.2.3 dev eth0 lladdr 52:54:00:12:35:03 STALE

```



|ネットワーク|ホスト側IP|ゲスト側IP|ゲスト側DNS|参考|
|--|--|--|--|--|
|NAT（デフォルト）|10.0.2.2|10.0.2.15|10.0.2.3|https://www.virtualbox.org/manual/ch09.html#nat-address-config|
|PrivateNetwork(HostOnly)DHCP|172.28.128.1|172.28.128.3|172.28.128.2??|https://www.virtualbox.org/manual/ch06.html#network_hostonly<br>(※1)|
|PrivateNetwork(HostOnly)FIX|192.168.x.1|192.168.x.x|192.168.x.2??|
|PublicNetwork DHCP|192.168.56.1|192.168.56.x|192.168.56.2??|

★全体
http://zorinos.seesaa.net/article/450304938.html
https://qiita.com/tmiki/items/d786edf221feb4cb1af5
https://qiita.com/satoki-shiro/items/9ba6a7b7118b9eab8b8e
https://www.virtualbox.org/manual/ch09.html#changenat

(※1)
DHCP
https://github.com/hashicorp/vagrant/issues/11403
```sh
C:\Program Files\Oracle\VirtualBox>VBoxManage list dhcpservers
NetworkName:    HostInterfaceNetworking-VirtualBox Host-Only Ethernet Adapter
IP:             0.0.0.0
NetworkMask:    0.0.0.0
lowerIPAddress: 0.0.0.0
upperIPAddress: 0.0.0.0
Enabled:        No
Global options:
   1:0.0.0.0

NetworkName:    HostInterfaceNetworking-VirtualBox Host-Only Ethernet Adapter #2
IP:             0.0.0.0
NetworkMask:    0.0.0.0
lowerIPAddress: 0.0.0.0
upperIPAddress: 0.0.0.0
Enabled:        No
Global options:
   1:0.0.0.0

NetworkName:    HostInterfaceNetworking-VirtualBox Host-Only Ethernet Adapter #3
IP:             192.168.99.16
NetworkMask:    255.255.255.0
lowerIPAddress: 192.168.99.100
upperIPAddress: 192.168.99.254
Enabled:        Yes
Global options:
   1:255.255.255.0

NetworkName:    HostInterfaceNetworking-VirtualBox Host-Only Ethernet Adapter #4
IP:             172.28.128.2
NetworkMask:    255.255.255.0
lowerIPAddress: 172.28.128.3
upperIPAddress: 172.28.128.254
Enabled:        Yes
Global options:
   1:255.255.255.0
```