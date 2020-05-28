# Redis

- [1. はじめに](#a1)
- [2. インストール](#a2)
- [3. 基本](#a3)
- [4. 冗長化](#a4)
   - [4-1. 種類](#a4-1)
   - [4-2. Cluster](#a4-2)


RedisのインストールからClusterの構築までを行う。

<span id="a1">

## 1. はじめに
- > Redis は、Key-Value型 の NoSQL データベースです。

- > Redis は Client-Server モデル を採用した インメモリデータベースです。

   https://qiita.com/wind-up-bird/items/f2d41d08e86789322c71

<span id="a2">

## 2. インストール
```sh
# CentOS7で確認
$ cat /etc/redhat-release
CentOS Linux release 7.8.2003 (Core)

# epel
$ sudo yum -y install epel-release

# remi
$ sudo yum install http://rpms.famillecollet.com/enterprise/remi-release-7.rpm

# リポジトリ設定
$ cat /etc/yum.repos.d/epel.repo | sed -n '/\[epel\]/,/^$/p' | egrep '\[.*\]|enabled'
[epel]
enabled=1

$ cat /etc/yum.repos.d/remi.repo | sed -n '/\[remi\]/,/^$/p' | egrep '\[.*\]|enabled'
[remi]
enabled=0 # デフォルトは無効

# version検索（epel）
$ sudo yum info redis | egrep 'Name|Version'
Name        : redis
Version     : 3.2.12

# version検索（remi）
$ sudo yum info redis --enablerepo=remi | egrep 'Name|Version'
Name        : redis
Version     : 6.0.4

# Redis-6.0.4をインストール
$ sudo yum -y install redis --enablerepo=remi

# 自動起動on
#$ sudo systemctl enable redis
```
参考
- https://weblabo.oscasierra.net/redis-centos7-install-yum/
- https://colabmix.co.jp/tech-blog/centos7-redis-install/

<span id="a3">

## 3. 基本
### Redisサーバー開始
```sh
$ sudo systemctl start redis

$ sudo systemctl status redis
● redis.service - Redis persistent key-value database
   Loaded: loaded (/usr/lib/systemd/system/redis.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/redis.service.d
           └─limit.conf
   Active: active (running) since Wed 2020-05-27 13:24:11 JST; 1min 12s ago
 Main PID: 6113 (redis-server)
   Status: "Ready to accept connections"
   CGroup: /system.slice/redis.service
           └─6113 /usr/bin/redis-server 127.0.0.1:6379

May 27 13:24:10 cheftest systemd[1]: Starting Redis persistent key-value database...
May 27 13:24:11 cheftest systemd[1]: Started Redis persistent key-value database.

# バージョン確認
$ redis-server --version
Redis server v=6.0.4 sha=00000000:0 malloc=jemalloc-5.1.0 bits=64 build=736d3ac587f8650f

$ redis-cli -v
redis-cli 6.0.4
```

### Redisクライアントで接続

Redisクライアントには様々な種類が存在する。
- https://redis.io/clients

今回は、ターミナルからサーバーに直接アクセスできるコマンドラインインターフェースである`redis-cli`を使用してみる。
- https://redis.io/topics/rediscli

Redisサーバーにアクセス
```sh
$ redis-cli
127.0.0.1:6379> ping
PONG

127.0.0.1:6379> set test "Hello World!"
OK

127.0.0.1:6379> get test
"Hello World!"

127.0.0.1:6379> info
# Server
redis_version:6.0.4
redis_git_sha1:00000000
redis_git_dirty:0
・・・
127.0.0.1:6379> exit

# 停止
$ sudo systemctl stop redis
```
redis-cliから打てるコマンドは https://redis.io/commands を参照のこと。

例）
- [info](https://redis.io/commands/info)
- [flushall](https://redis.io/commands/flushall)

redis-cliでログイン -> infoコマンドというような上記のやり方以外に、`redis-cli info`というように1コマンドでも実行できる。

<span id="a4">

## 4. 冗長化

<span id="a4-1">

### 4-1. 種類

以下の３種類ある。
- > Replication: マスター・スレーブ型のレプリケーション
- > Sentinel: 死活監視と自動フェイルオーバーを行うサービス
- > Cluster: マルチマスター構成。データを複数サーバに分散できる(シャーディング)

<span id="a4-2">

### 4-2. Cluster
今回はClusterを試す。

参考：https://www.sraoss.co.jp/tech-blog/redis/redis-cluster/

イメージ

<img src=https://www.sraoss.co.jp/tech-blog/wp-content/uploads/2019/06/redis-cluster-768x886.png style="width:30%">

- 疑似マルチマスタ構成で保存先を分散（＝シャーディング）
   - 保存先は、ノードに割り当てられたhash slot値をもとに決まる。
- Master,Slave構成も可能
   - Masterが落ちたらSlaveにフェイルオーバー

#### a. クラスタ設定
上図の通り1ホストに6プロセス（master=3,slave=3）立てるため、6プロセス分の設定ファイルを用意する。
```sh
$ mkdir cluster-test
$ cd cluster-test/
$ mkdir 7000 7001 7002 7003 7004 7005
$ sudo cp /etc/redis.conf 7000
$ sudo vi 7000/redis.conf
# 以下のみ編集
port 7000
pidfile "/var/run/redis_7000.pid"
logfile "/root/cluster-test/redis_7000.log"
dbfilename "dump-7000.rdb"
appendonly yes
appendfilename "appendonly-7000.aof"
cluster-enabled yes
cluster-config-file nodes-7000.conf

$ sudo cp 7000/redis.conf 7001
$ sudo vi 7001/redis.conf
# かぶらないように7000の部分を7001に変更
...
$ sudo vi 7005/redis.conf
```

#### b. プロセス起動
バックグラウンド起動する。
```sh
$ sudo redis-server 7000/redis.conf &
$ sudo redis-server 7001/redis.conf &
$ sudo redis-server 7002/redis.conf &
$ sudo redis-server 7003/redis.conf &
$ sudo redis-server 7004/redis.conf &
$ sudo redis-server 7005/redis.conf &

$ ps aux | grep redis | egrep -v 'grep|sudo'
root      5534  0.4  1.1 219604  5580 pts/1    Sl   13:26   0:00 redis-server 127.0.0.1:7000 [cluster]
root      5543  0.3  1.1 219604  5580 pts/1    Sl   13:26   0:00 redis-server 127.0.0.1:7001 [cluster]
root      5556  0.2  1.1 219604  5864 pts/1    Sl   13:27   0:00 redis-server 127.0.0.1:7002 [cluster]
root      5563  0.2  1.1 219604  5772 pts/1    Sl   13:27   0:00 redis-server 127.0.0.1:7003 [cluster]
root      5572  0.2  1.1 219604  5796 pts/1    Sl   13:27   0:00 redis-server 127.0.0.1:7004 [cluster]
root      5581  0.1  1.1 219604  5820 pts/1    Sl   13:27   0:00 redis-server 127.0.0.1:7005 [cluster]
```
-
   <details><summary>生成されたファイル</summary>

   ```sh
   $ sudo ls -l /var/lib/redis/
   total 48
   -rw-r--r--. 1 root  root    0 May 27 14:49 appendonly-7000.aof
   -rw-r--r--. 1 root  root    0 May 27 14:50 appendonly-7001.aof
   -rw-r--r--. 1 root  root    0 May 27 14:50 appendonly-7002.aof
   -rw-r--r--. 1 root  root    0 May 27 14:50 appendonly-7003.aof
   -rw-r--r--. 1 root  root    0 May 27 14:50 appendonly-7004.aof
   -rw-r--r--. 1 root  root    0 May 27 14:50 appendonly-7005.aof
   -rw-r--r--. 1 root  root   92 May 27 14:50 dump-7000.rdb
   -rw-r--r--. 1 root  root   92 May 27 14:50 dump-7002.rdb
   -rw-r--r--. 1 root  root   92 May 27 14:50 dump-7003.rdb
   -rw-r--r--. 1 root  root   92 May 27 14:50 dump-7004.rdb
   -rw-r--r--. 1 root  root   92 May 27 14:50 dump-7005.rdb
   -rw-r--r--. 1 redis redis 116 May 27 13:55 dump.rdb
   -rw-r--r--. 1 root  root  114 May 27 14:49 nodes-7000.conf
   -rw-r--r--. 1 root  root  114 May 27 14:50 nodes-7001.conf
   -rw-r--r--. 1 root  root  114 May 27 14:50 nodes-7002.conf
   -rw-r--r--. 1 root  root  114 May 27 14:50 nodes-7003.conf
   -rw-r--r--. 1 root  root  114 May 27 14:50 nodes-7004.conf
   -rw-r--r--. 1 root  root  114 May 27 14:50 nodes-7005.conf
   ```
   </details>

#### c. クラスタ作成
レプリカ1組の構成で6プロセス（master=3,slave=3）立ち上げる。

master,slaveの割り当てプロセスは自動。手動で割り当てる方法は後述。
```sh
$ redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1
```
-
   <details><summary>コマンド実行ログ</summary>

   ```sh
   $  redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1
   >>> Performing hash slots allocation on 6 nodes...
   Master[0] -> Slots 0 - 5460
   Master[1] -> Slots 5461 - 10922
   Master[2] -> Slots 10923 - 16383
   Adding replica 127.0.0.1:7004 to 127.0.0.1:7000
   Adding replica 127.0.0.1:7005 to 127.0.0.1:7001
   Adding replica 127.0.0.1:7003 to 127.0.0.1:7002
   >>> Trying to optimize slaves allocation for anti-affinity
   [WARNING] Some slaves are in the same host as their master
   M: ae539bb2d7ac8f9b0214f3c909dd6c5f0adc3d67 127.0.0.1:7000
      slots:[0-5460] (5461 slots) master
   M: 90903135538d1f3f5853d6c4ea7ed864faf9e14c 127.0.0.1:7001
      slots:[5461-10922] (5462 slots) master
   M: 2800d5117205b44caa1e136278412a58e79d6c35 127.0.0.1:7002
      slots:[10923-16383] (5461 slots) master
   S: ddc81464297ba7a3d52bc9f581e80214758e118b 127.0.0.1:7003
      replicates 2800d5117205b44caa1e136278412a58e79d6c35
   S: 64414e3890d289ba134032fa5100bbb04ce9fe15 127.0.0.1:7004
      replicates ae539bb2d7ac8f9b0214f3c909dd6c5f0adc3d67
   S: 6c293309f118df2c6321de5aca6525da263128c4 127.0.0.1:7005
      replicates 90903135538d1f3f5853d6c4ea7ed864faf9e14c
   Can I set the above configuration? (type 'yes' to accept): yes
   >>> Nodes configuration updated
   >>> Assign a different config epoch to each node
   >>> Sending CLUSTER MEET messages to join the cluster
   Waiting for the cluster to join
   ...
   >>> Performing Cluster Check (using node 127.0.0.1:7000)
   M: ae539bb2d7ac8f9b0214f3c909dd6c5f0adc3d67 127.0.0.1:7000
      slots:[0-5460] (5461 slots) master
      1 additional replica(s)
   M: 2800d5117205b44caa1e136278412a58e79d6c35 127.0.0.1:7002
      slots:[10923-16383] (5461 slots) master
      1 additional replica(s)
   S: ddc81464297ba7a3d52bc9f581e80214758e118b 127.0.0.1:7003
      slots: (0 slots) slave
      replicates 2800d5117205b44caa1e136278412a58e79d6c35
   S: 6c293309f118df2c6321de5aca6525da263128c4 127.0.0.1:7005
      slots: (0 slots) slave
      replicates 90903135538d1f3f5853d6c4ea7ed864faf9e14c
   S: 64414e3890d289ba134032fa5100bbb04ce9fe15 127.0.0.1:7004
      slots: (0 slots) slave
      replicates ae539bb2d7ac8f9b0214f3c909dd6c5f0adc3d67
   M: 90903135538d1f3f5853d6c4ea7ed864faf9e14c 127.0.0.1:7001
      slots:[5461-10922] (5462 slots) master
      1 additional replica(s)
   [OK] All nodes agree about slots configuration.
   >>> Check for open slots...
   >>> Check slots coverage...
   [OK] All 16384 slots covered.
   ```

   </details>

- > ところで、以前はこのコマンドは「redis-trib.rb」というRubyスクリプトで行っていました。  
   > Redis 5.0では、これがredis-cliで実行することが可能になっています。  
   https://kazuhira-r.hatenablog.com/entry/2018/11/25/234444

master,slaveに割り当てられたポートは以下の通り。（出力ログ内容を参照のこと）

|ポート|区分|対応するmaster|slot|
|--|--|--|--|
|7000|master|-|0-5460|
|7001|〃|-|5461-10922|
|7002|〃|-|10923-16383|
|7003|slave|7002|-|
|7004|〃|7000|-|
|7005|〃|7001|-|


#### d. 確認
```sh
# 指定ポートはどれでもok
$ redis-cli -p 7000 cluster nodes
2800d5117205b44caa1e136278412a58e79d6c35 127.0.0.1:7002@17002 master - 0 1590642262811 3 connected 10923-16383
ddc81464297ba7a3d52bc9f581e80214758e118b 127.0.0.1:7003@17003 slave 2800d5117205b44caa1e136278412a58e79d6c35 0 1590642264833 4 connected
6c293309f118df2c6321de5aca6525da263128c4 127.0.0.1:7005@17005 slave 90903135538d1f3f5853d6c4ea7ed864faf9e14c 0 1590642263821 8 connected
64414e3890d289ba134032fa5100bbb04ce9fe15 127.0.0.1:7004@17004 slave ae539bb2d7ac8f9b0214f3c909dd6c5f0adc3d67 0 1590642263000 1 connected
90903135538d1f3f5853d6c4ea7ed864faf9e14c 127.0.0.1:7001@17001 master - 0 1590642265845 2 connected 5461-10922
ae539bb2d7ac8f9b0214f3c909dd6c5f0adc3d67 127.0.0.1:7000@17000 myself,master - 0 1590642263000 1 connected 0-5460
```

#### e. データ保存
```sh
$ redis-cli -c -p 7000
127.0.0.1:7000> set foo 1
-> Redirected to slot [12182] located at 127.0.0.1:7002
OK
127.0.0.1:7002> get foo
"1"
127.0.0.1:7002> set bar 2
-> Redirected to slot [5061] located at 127.0.0.1:7000
OK
# ポイントは、上のようにRedirected ...と出たらログイン先まで自動で切り替わること。
# これはプロンプトで確認できる
127.0.0.1:7000> get bar
"2"
127.0.0.1:7000> get foo
-> Redirected to slot [12182] located at 127.0.0.1:7002
"1"
127.0.0.1:7002> get hoge
-> Redirected to slot [1525] located at 127.0.0.1:7000
(nil)
127.0.0.1:7000> incr foo
-> Redirected to slot [12182] located at 127.0.0.1:7002
(integer) 2
127.0.0.1:7002> get foo
"2"
127.0.0.1:7002> exit

# key保存の状態を確認（どのポートを指定しても同じ結果。--cluster モードなので）
$ redis-cli --cluster info 127.0.0.1:7000 | sort
0.00 keys per slot on average.
127.0.0.1:7000 (ae539bb2...) -> 1 keys | 5461 slots | 1 slaves.
127.0.0.1:7001 (90903135...) -> 0 keys | 5462 slots | 1 slaves.
127.0.0.1:7002 (2800d511...) -> 1 keys | 5461 slots | 1 slaves.
[OK] 2 keys in 3 masters.
```

#### f. フェイルオーバーの確認
```sh
# 7000のPIDを確認
$ ps aux | grep redis | egrep -v 'sudo|grep'
root      5534  0.2  1.1 219604  5832 pts/1    Sl   13:26   0:08 redis-server 127.0.0.1:7000 [cluster] # ★
root      5543  0.2  1.1 219604  5680 pts/1    Sl   13:26   0:08 redis-server 127.0.0.1:7001 [cluster]
root      5556  0.2  1.1 219604  5692 pts/1    Sl   13:27   0:08 redis-server 127.0.0.1:7002 [cluster]
root      5563  0.1  1.1 219604  5812 pts/1    Sl   13:27   0:08 redis-server 127.0.0.1:7003 [cluster]
root      5572  0.1  1.1 219604  5868 pts/1    Sl   13:27   0:08 redis-server 127.0.0.1:7004 [cluster]
root      5581  0.1  1.1 219604  5744 pts/1    Sl   13:27   0:08 redis-server 127.0.0.1:7005 [cluster]

# 現状のclusterの状態を確認
$ redis-cli -p 7000 cluster nodes
2800d5117205b44caa1e136278412a58e79d6c35 127.0.0.1:7002@17002 master - 0 1590644513000 3 connected 10923-16383
ddc81464297ba7a3d52bc9f581e80214758e118b 127.0.0.1:7003@17003 slave 2800d5117205b44caa1e136278412a58e79d6c35 0 1590644514153 4 connected
6c293309f118df2c6321de5aca6525da263128c4 127.0.0.1:7005@17005 slave 90903135538d1f3f5853d6c4ea7ed864faf9e14c 0 1590644513138 8 connected
64414e3890d289ba134032fa5100bbb04ce9fe15 127.0.0.1:7004@17004 slave ae539bb2d7ac8f9b0214f3c909dd6c5f0adc3d67 0 1590644515166 1 connected
90903135538d1f3f5853d6c4ea7ed864faf9e14c 127.0.0.1:7001@17001 master - 0 1590644514000 2 connected 5461-10922
ae539bb2d7ac8f9b0214f3c909dd6c5f0adc3d67 127.0.0.1:7000@17000 myself,master - 0 1590644512000 1 connected 0-5460 # ★

# 7000を止める
$ sudo kill -kill 5534

# 7000が消えた
$ ps aux | grep redis | egrep -v 'sudo|grep'
root      5543  0.2  1.1 219604  5700 pts/1    Sl   13:26   0:10 redis-server 127.0.0.1:7001 [cluster]
root      5556  0.2  1.1 219604  5812 pts/1    Sl   13:27   0:10 redis-server 127.0.0.1:7002 [cluster]
root      5563  0.1  1.1 219604  5748 pts/1    Sl   13:27   0:10 redis-server 127.0.0.1:7003 [cluster]
root      5572  0.1  1.1 219604  5880 pts/1    Sl   13:27   0:09 redis-server 127.0.0.1:7004 [cluster]
root      5581  0.1  1.1 219604  5712 pts/1    Sl   13:27   0:09 redis-server 127.0.0.1:7005 [cluster]

# 7000を落としたので7001にアクセス（-p 70001）
$ redis-cli -p 7001 cluster nodes
ddc81464297ba7a3d52bc9f581e80214758e118b 127.0.0.1:7003@17003 slave 2800d5117205b44caa1e136278412a58e79d6c35 0 1590645161715 4 connected
6c293309f118df2c6321de5aca6525da263128c4 127.0.0.1:7005@17005 slave 90903135538d1f3f5853d6c4ea7ed864faf9e14c 0 1590645163747 8 connected
ae539bb2d7ac8f9b0214f3c909dd6c5f0adc3d67 127.0.0.1:7000@17000 master,fail - 1590645144457 1590645140399 1 disconnected # ★
90903135538d1f3f5853d6c4ea7ed864faf9e14c 127.0.0.1:7001@17001 myself,master - 0 1590645162000 2 connected 5461-10922
64414e3890d289ba134032fa5100bbb04ce9fe15 127.0.0.1:7004@17004 master - 0 1590645163000 9 connected 0-5460 # ★
2800d5117205b44caa1e136278412a58e79d6c35 127.0.0.1:7002@17002 master - 0 1590645161000 3 connected 10923-16383
# -> 7000の代わりに7004がmasterになった
```
- プロセスの停止は
   ```sh
   $ jobs
   [3]   Running                 sudo redis-server 7002/redis.conf &
   [4]   Running                 sudo redis-server 7003/redis.conf &
   [5]   Running                 sudo redis-server 7004/redis.conf &
   [6]-  Running                 sudo redis-server 7005/redis.conf &
   [7]+  Running                 sudo redis-server 7000/redis.conf &
   $ fg %3
   Ctrl + C 
   ```
   で行くか、普通にkillするか。
   ```
   $ sudo kill <PID>
   ```

#### g. プロセス復帰
```sh
$ sudo redis-server 7000/redis.conf &

$ ps a | grep redis | egrep -v 'sudo|grep'
 5543 pts/1    Sl     0:12 redis-server 127.0.0.1:7001 [cluster]
 5556 pts/1    Sl     0:12 redis-server 127.0.0.1:7002 [cluster]
 5563 pts/1    Sl     0:12 redis-server 127.0.0.1:7003 [cluster]
 5572 pts/1    Sl     0:12 redis-server 127.0.0.1:7004 [cluster]
 5581 pts/1    Sl     0:11 redis-server 127.0.0.1:7005 [cluster]
 5766 pts/0    Sl     0:00 redis-server 127.0.0.1:7000 [cluster] # ★

# 7000がslaveとして復活
$ redis-cli -p 7000 cluster nodes | awk '{ print $2,$3 }'
127.0.0.1:7000@17000 myself,slave # ★
127.0.0.1:7003@17003 slave
127.0.0.1:7001@17001 master
127.0.0.1:7004@17004 master
127.0.0.1:7005@17005 slave
127.0.0.1:7002@17002 master
```
#### h. クラスタ削除
各ポートごとに`cluster reset`する。  
データが残っている場合はエラーになるので、`flushall`でデータを削除してからリセットしなおす。
```sh
$ redis-cli -p 7000 cluster reset
OK

$ redis-cli -p 7001 cluster reset
OK

$ redis-cli -p 7002 cluster reset
(error) ERR cluster reset can't be called with master nodes containing keys
$ redis-cli -p 7002 flushall
OK

$ redis-cli -p 7003 cluster reset
OK

$ redis-cli -p 7004 cluster reset
(error) ERR cluster reset can't be called with master nodes containing keys
$ redis-cli -p 7004 flushall
OK

$ redis-cli -p 7005 cluster reset
OK
```
参考：https://qiita.com/ono-soic/items/d064f4db0e66249f7c85

#### i. クラスタ作成（スレーブ指定）

スレーブを個別に指定してみる
```sh
# Master作成
$ redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002

$ redis-cli -p 7000 cluster nodes | awk '{print $1,$2,$3}'
90903135538d1f3f5853d6c4ea7ed864faf9e14c 127.0.0.1:7001@17001 master
ae539bb2d7ac8f9b0214f3c909dd6c5f0adc3d67 127.0.0.1:7000@17000 myself,master
2800d5117205b44caa1e136278412a58e79d6c35 127.0.0.1:7002@17002 master

# Slave１台目作成
$ redis-cli --cluster add-node 127.0.0.1:7003 127.0.0.1:7000 --cluster-slave --cluster-master-id ae539bb2d7ac8f9b0214f3c909dd6c5f0adc3d67

# Slave２台目作成
$ redis-cli --cluster add-node 127.0.0.1:7004 127.0.0.1:7001 --cluster-slave --cluster-master-id 90903135538d1f3f5853d6c4ea7ed864faf9e14c

# Slave３台目作成
$ redis-cli --cluster add-node 127.0.0.1:7005 127.0.0.1:7002 --cluster-slave --cluster-master-id 2800d5117205b44caa1e136278412a58e79d6c35

$ redis-cli -p 7000 cluster nodes | awk '{print $2,$3}'
127.0.0.1:7004@17004 slave
127.0.0.1:7003@17003 slave
127.0.0.1:7001@17001 master
127.0.0.1:7000@17000 myself,master
127.0.0.1:7005@17005 slave
127.0.0.1:7002@17002 master
```
#### j フェイルオーバー（redis-cli）
```sh
$ redis-cli -p 7000 debug segfault
Error: Server closed the connection
[1]   Segmentation fault      sudo redis-server 7000/redis.conf

$ redis-cli -p 7001 cluster nodes | awk '{print $2,$3}'
127.0.0.1:7003@17003 master
127.0.0.1:7005@17005 slave
127.0.0.1:7000@17000 master,fail # ★
127.0.0.1:7001@17001 myself,master
127.0.0.1:7004@17004 slave
127.0.0.1:7002@17002 master

$ redis-cli -p 7003 cluster failover
(error) ERR You should send CLUSTER FAILOVER to a replica

# slaveしか指定できない模様
$ redis-cli -p 7005 cluster failover
OK

$ redis-cli -p 7001 cluster nodes | awk '{print $2,$3}'
127.0.0.1:7003@17003 master
127.0.0.1:7005@17005 master # ★ slave->master
127.0.0.1:7000@17000 master,fail
127.0.0.1:7001@17001 myself,master
127.0.0.1:7004@17004 slave
127.0.0.1:7002@17002 slave # ★ master->slave
```
参考：https://fisproject.jp/2016/12/create-redis-cluster-on-centos6/

## 5. 参考
- https://qiita.com/wind-up-bird/items/f2d41d08e86789322c71
- https://www.sraoss.co.jp/tech-blog/redis/redis-cluster/
- https://kazuhira-r.hatenablog.com/entry/2018/11/25/234444
