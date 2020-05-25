# Appendex. NCコマンドあれこれ

参考：https://qiita.com/hana_shin/items/97e6c03ac5e5ed67ce38

- NCコマンドとは、通信確認をする際に便利なコマンドのことで、「Netcat」というアプリケーションが本家本元。
- Netcatにはいくつかの派生・互換ツールがあり、オプションなど使い勝手も微妙に異なる。
    - http://www.intellilink.co.jp/article/column/security-net01.html
- Dockerのalpineイメージにデフォルトでインストールされているncと、CentOS7でインストールするncとは異なる。
    - alpineとは、セキュアで軽量なLinuxディストリビューションで、Dockerに最適とよく言われている。
    - alpineでの検証 [Netcat]
        ```sh
        $ docker container run -it alpine /bin/sh

        / # hostname -i
        172.17.0.2

        # NG
        / # nc -lnv 172.17.0.3 54321
        listening on 0.0.0.0:34955 ...　# 引数に指定したポート番号は無視された

        # OK
        / # nc -lnv -s 172.17.0.3 -p 54321
        listening on 172.17.0.3:54321 ...
        ```
    - CentOS7での検証 [Ncat]
        ```sh
        $ docker container run -it centos:centos7 /bin/sh

        sh-4.2# yum install nmap-ncat -y

        sh-4.2# hostname -i
        172.17.0.2

        sh-4.2# nc -lnv 172.17.0.2 54321
        Ncat: Version 7.50 ( https://nmap.org/ncat )
        Ncat: Listening on 172.17.0.2:54321
        ```
- [Ncat] 通信してみる
    
    VM2 -> VM1に接続し、VM2で送信した文字がVM1に表示されることを確認する。

    VM1
    ```sh
    $ yum install nmap-ncat -y

    # ポート11111で待ち受ける
    $ nc -lnv 192.168.33.11 11111
    Ncat: Version 7.50 ( https://nmap.org/ncat )
    Ncat: Listening on 192.168.33.11:11111
    # 以下はVM2の操作が終わると出力される
    Ncat: Connection from 192.168.33.11:50152.
    Hello World! # VM2で入力した内容
    ```
    VM2
    ```sh
    $ yum install nmap-ncat -y

    $ nc 192.168.33.11 11111
    Hello World! # <- 入力してEnter押下
    ^C # Ctrl + Cで中断
    ```

- [Ncat] ポートスキャン(-z)の結果に「Succeed」が出力されないが、ステータスコードで何とか判別はできる
    ```sh
    # ポート80が立ち上がっているVMで確認した（81は立ち上げていない）
    $ nc -z 192.168.33.11 80
    $ echo $?
    0
    $ nc -z 192.168.33.11 81
    $ echo $?
    1
    ```
    - > ncコマンドのポートスキャン機能 (zオプション) は REHL7系からなくなってしまいましたので、
        - https://wisteriasec.wordpress.com/2017/12/18/nc-netcat-%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%9Fnw%E7%96%8E%E9%80%9A%E7%A2%BA%E8%AA%8D/
        - https://keyamb.hatenablog.com/entry/2016/03/06/105831

    - -z指定でエラーにならないのは不明

- nmapでも分析できるがが面倒なのでncのステータスコード($?)を確認した方が扱いやすそう。
    ```sh
    $ nmap 192.168.33.11 -p 80

    Starting Nmap 6.40 ( http://nmap.org ) at 2020-04-22 18:41 JST
    Nmap scan report for 192.168.33.11
    Host is up (0.000059s latency).
    PORT   STATE SERVICE
    80/tcp open  http # ★
    ```

    ```
    $ nmap -h
    PORT SPECIFICATION AND SCAN ORDER:
    -p <port ranges>: Only scan specified ports
        Ex: -p22; -p1-65535; -p U:53,111,137,T:21-25,80,139,8080,S:9
    -F: Fast mode - Scan fewer ports than the default scan
    -r: Scan ports consecutively - don't randomize
    --top-ports <number>: Scan <number> most common ports
    --port-ratio <ratio>: Scan ports more common than <ratio>
    ```