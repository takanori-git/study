# 1. Docker概要

- Dockerはコンテナ技術を使ったアプリケーションの実行環境を構築・運用するためのプラットフォーム。
- コンテナとは、ホストOS上に論理的に作られた区画で、アプリケーション・ミドルウェアの**ファイル群**をひとまとまりとしたもの。
- コンテナ同士はホストOS上のリソースを論理的に分離する（namespace, cgroups）ので、１つのサーバーとして扱うことができる。
-  サーバー仮想化技術にくらべ軽量。（ゲストOSを保持しないことが大きい）
    ![base.jpg](img/base.jpg)
- コンテナは、Dockerfileから作成したイメージをもとに作成する。
    - Dockerfile
        - ↓
    - Docker image　// DockerHubで公開されているものを利用可
        - ↓
    - Docker Container
- Dockerで指定する「ベースイメージ」内のOSは、パッケージ管理のためのOS
に過ぎない。アプリケーションを動かしているOSは、ホストOSのLinux。
    - https://teratail.com/questions/124191
- コンテナのライフサイクル

    ![docker-status.jpg](img/docker-status.jpg)
- Dockerの各種コンポーネント

    |コンポーネント名|概要|
    |--|--|--|
    |Docker Engine|Dockerのコア機能|
    |Docker Registory|イメージ公開／共有|
    |Docker Compose|複数コンテナ一元管理|
    |Docker Machine|Docker実行環境構築|
    |Docker Swarm|クラスタ管理|

    ![docker-全体像.jpg](img/docker-全体像.jpg)
