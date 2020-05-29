# Chef

- [1. 前提知識](#a1)
- [2. ChefDKのインストール](#a2)
- [3. Chef Repositoryの作成](#a3)
- [4. 実行設定](#a4)
- [5. 実行](#a5)
- [6. 実験](#a6)
- [7. 参考](#a7)

Chefをlocal modeで動かす。

<span id="a1">

## 1. 前提知識
- Chefはサーバー設定管理ツール。
    - 例えばPHPをインストールするといった設定をファイルに記載することで、何度でもその状態にすることができる。これを収束(Converge)するという。
- Chef（公式には`Chef Infra`というらしい）は大まかに以下の３つから構成されている。
    - Chef Workstation // クライアント端末
    - Chef Server // 収束するための設定類を格納するサーバー
        - Chef Serverは必須ではない。デバッグ用途などではChef Serverを使わない`local mode`でChefを実行することもできる。
        - Chef Serverを使用する場合は、自前でサーバーを立てるかChef社のホスティングサーバーを利用することもできる。
    - Client // 収束させたいサーバー

|項目名|説明|アプリケーション名|備考|
|--|--|--|--|
|Chef Workstation|クライアント端末の環境|ChefDK,<br>Chef Workstation|Chef WorkstationはChefDKの後継。<br>local modeでは不要|
|Chef Server|収束するための設定類を格納するサーバー|Chef Manage|local modeでは不要|
|Client|アプリケーションサーバーなどの収束させたいサーバー|Chef Infra Client|-|

参考
- > Chef WorkstationはChefDKの後継製品であり、ChefDKに現在備わっているすべてのツールを含み、すべて無料でご利用いただけます。  
https://www.creationline.com/lab/23515

- > ChefDKとはChefおよびポピュラーな関連ツール一式を簡単にインストールできるようにパッケージングしたソフトウェアになります。  
https://knowledge.sakura.ad.jp/2825/  
https://www.creationline.com/lab/6255

- 公式 Chef Infra: https://docs.chef.io/release_notes_chefdk/

### 補足

`Chef Workstation`は`ChefDK`の後継で`ChefDK`を含み、`ChefDK`は`Chef Infra Client`を含む模様。
- Chef Workstation
    - ChefDK
        - Chef Infra Client

-> 今回はChefDKを入れてみる。（local modeだけならChef Infra Clientだけでもよさそうだが）

<span id="a2">

## 2. ChefDKのインストール
```sh
$ cat /etc/redhat-release
CentOS Linux release 7.8.2003 (Core)

$ sudo rpm -Uvh https://packages.chef.io/files/stable/chefdk/3.13.1/el/7/chefdk-3.13.1-1.el7.x86_64.rpm

$ chef -v
Chef Development Kit Version: 3.13.1
chef-client version: 14.14.29
delivery version: master (4b21ec7e07fdfa82e86aa80e4f2372dde8e368bb)
berks version: 7.0.8
kitchen version: 1.25.0
inspec version: 3.9.3
```
参考：https://downloads.chef.io/chefdk/stable/3.13.1

<span id="a3">

## 3. Chef Repositoryの作成
ひな型を作るコマンドがあるので実行する。
```sh
$ chef generate repo chef-repo && cd chef-repo
$ ll
total 12
-rw-rw-r--. 1 vagrant vagrant 1176 May 28 18:17 chefignore
drwxrwxr-x. 3 vagrant vagrant   38 May 28 18:17 cookbooks
drwxrwxr-x. 3 vagrant vagrant   38 May 28 18:17 data_bags
drwxrwxr-x. 2 vagrant vagrant   43 May 28 18:17 environments
-rw-rw-r--. 1 vagrant vagrant   70 May 28 18:17 LICENSE
-rw-rw-r--. 1 vagrant vagrant 1338 May 28 18:17 README.md
drwxrwxr-x. 2 vagrant vagrant   43 May 28 18:17 roles

```
### ひな型で作成される内容を確認

前提知識
- Chefをlocal modeで実行するためには、最低限、収束対象サーバーに関する以下の4ファイルが必要。
 
    |項目|説明|
    |--|--|--|
    |node|利用するenvironment,role,recipeが記載されているファイル|
    |environment|環境変数などをまとめた設定ファイル|
    |role|メインメソッドのようなものが記載されているファイル(`run_list`という項目で管理)。<br>role自身や直接recipeを呼び出せる。|
    |recipe|メソッドのようなものが記載されているファイル。cookbookというものの中で管理されている。|

```sh
# Environment
$ cat environments/example.json | grep -v '^#'
{
    "name": "example",
    "description": "This is an example environment defined as JSON",
    "chef_type": "environment",
    "json_class": "Chef::Environment",
    "default_attributes": {
    },
    "override_attributes": {
    },
    "cookbook_versions": {
        "example": "= 1.0.0"
    }
}

# Role
$ cat roles/example.json
{
    "name": "example",
    "description": "This is an example role defined as JSON",
    "chef_type": "role",
    "json_class": "Chef::Role",
    "default_attributes": {
    },
    "override_attributes": {
    },
    "run_list": [
        "recipe[example]"
    ]
}

# Recipe
$ cat cookbooks/example/recipes/default.rb | grep -v '^#'

log "Welcome to Chef Infra Client, #{node['example']['name']}!" do
  level :info
end

# Node
ひな型には入っていないので、以下の操作で作成する。

# 空振り実行。「-z」はlocal modeで実行することを示す。
$ sudo chef-client -z
・・・
resolving cookbooks for run list: []
・・・
[2020-05-28T18:31:51+09:00] WARN: Node cheftest has an empty run list.
・・・
Chef Client finished, 0/0 resources updated in 02 seconds # ★

# 空振り実行により、nodes/<hostname>.jsonができる。このファイルは直接いじらない
$ sudo cat nodes/cheftest.json | wc -l
5465 # 5000行以上
```
- local modeのnode/＜hostname＞.jsonは、Chef Serverを使う通常モードの「Chef Serverに登録するNode設定」に対応すると思われる。

<span id="a4">

## 4. 実行設定
```sh
# NodeファイルのRoleに、ひな型で作成されたrole「example」を設定
$ sudo knife node run_list set cheftest role[example] -z
WARNING: No knife configuration file found. See https://docs.chef.io/config_rb_knife.html for details.
cheftest:
  run_list: role[example]

$ sudo grep \"run_list\" -A2 nodes/cheftest.json
  "run_list": [
    "role[example]"
  ]

# Nodeファイルのenvironmentに、ひな型で作成されたenvironment「example」を設定
$ sudo knife node environment set cheftest example -z
WARNING: No knife configuration file found. See https://docs.chef.io/config_rb_knife.html for details.
cheftest:
  chef_environment: _default

# 確認
$ sudo knife node show cheftest -z
WARNING: No knife configuration file found. See https://docs.chef.io/config_rb_knife.html for details.
Node Name:   cheftest
Environment: example
FQDN:        cheftest
IP:          10.0.2.15
Run List:    role[example]
Roles:       example
Recipes:     example, example::default
Platform:    centos 7.8.2003
Tags:
```

<span id="a5">

## 5. 実行
```sh
$ sudo chef-client -z
[2020-05-29T09:50:39+09:00] WARN: No config file found or specified on command line. Using command line options instead.
Starting Chef Client, version 14.14.29
resolving cookbooks for run list: ["example"]
Synchronizing Cookbooks:
  - example (1.0.0)
Installing Cookbook Gems:
Compiling Cookbooks...
Converging 1 resources
Recipe: example::default
  * log[Welcome to Chef Infra Client, Sam Doe!] action write # ★


Running handlers:
Running handlers complete
Chef Client finished, 1/1 resources updated in 01 seconds # ★
```
適用内容
- environment: example
- run_list: recipe[example]

<span id="a6">

## 6. 実験
### 実験１
直接Recipeをたたくようにしてみる
```sh
# recipeを直接呼ぶ（以下は省略記法。フルで記載すると「example:default」）
$ sudo knife node run_list set cheftest recipe[example] -z

$ sudo knife node show cheftest -z
・・・
Node Name:   cheftest
Environment: example
FQDN:        cheftest
IP:          10.0.2.15
Run List:    recipe[example]
Roles:       example
Recipes:     example, example::default
Platform:    centos 7.8.2003
Tags:

$ sudo chef-client -z
・・・
resolving cookbooks for run list: ["example"]
Synchronizing Cookbooks:
  - example (1.0.0)
Recipe: example::default
  * log[Welcome to Chef Infra Client, Sam Doe!] action write
・・・
Chef Client finished, 1/1 resources updated in 01 seconds
```
- Role経由ではなく直接Recipeを実行できた。

### 実験２
新しいRecipeを作成し、そのRecipeを呼び出すように変更する。
```sh
# 新しいRecipe
$ vi cookbooks/example/recipes/test_recipe.rb

log "Hello World!!!!, #{node['example']['name']}!" do
  level :info
end

$ sudo knife node run_list set cheftest recipe[example::test_recipe] -z

$ sudo knife node show cheftest -z
・・・
Node Name:   cheftest
Environment: _default
FQDN:        cheftest
IP:          10.0.2.15
Run List:    recipe[example::test_recipe] # ★
Roles:       example # 呼ばれてないっぽい
Recipes:     example, example::default # 呼ばれてないっぽい
Platform:    centos 7.8.2003
Tags:

$ sudo chef-client -z
・・・
resolving cookbooks for run list: ["example::test_recipe"]
Synchronizing Cookbooks:
  - example (1.0.0)
Recipe: example::test_recipe
  * log[Hello World!!!!, Sam Doe!] action write
```
- おそらく、`Run List:    recipe[example::test_recipe]`の項目しか見ていないと思われ
る。
    ```sh
    $ sudo knife node show cheftest -z
    ・・・
    Node Name:   cheftest
    Environment: _default
    FQDN:        cheftest
    IP:          10.0.2.15
    Run List:    recipe[example::test_recipe] # ★ 見てる
    Roles: # 見てない
    Recipes: # 見てない
    Platform:    centos 7.8.2003
    Tags:
    ```

### 実験３
新しく作ったRoleを適用してみる。
```sh
# 新しいRole
$ vi roles/test.json
{
    "name": "test",
    "description": "This is an example role defined as JSON",
    "chef_type": "role",
    "json_class": "Chef::Role",
    "default_attributes": {
    },
    "override_attributes": {
    },
    "run_list": [
        "recipe[example:test_recipe]"
    ]
}

# ポイントは引用符でくくること
$ sudo knife node run_list set cheftest 'role[test]' -z
・・
cheftest:
  run_list: role[test]

$ sudo chef-client -z
・・・
resolving cookbooks for run list: ["example::test_recipe"]
Synchronizing Cookbooks:
  - example (1.0.0)
・・・
Recipe: example::test_recipe
  * log[Hello World!!!!, Sam Doe!] action write

$ sudo knife node show cheftest -z
・・・
Node Name:   cheftest
Environment: _default
FQDN:        cheftest
IP:          10.0.2.15
Run List:    recipe[roles]
Roles:       test
Recipes:     example::test_recipe
Platform:    centos 7.8.2003
Tags:
```
<details><summary>トラブルシューティング</summary>

knifr node runlistでは、引数を引用符でくくらないとだめなケースがある。  
testというrole名がLinuxのtestコマンドとして実行されてしまったのかと考えたが、test_aaaなどの名前でも駄目だった。  
よくわからない。

```sh
$ sudo knife node run_list set cheftest role[test] -z
・・
cheftest:
  run_list: recipe[roles] # なぜか追加されない

$ sudo knife node show cheftest -z
Node Name:   cheftest
Environment: _default
FQDN:        cheftest
IP:          10.0.2.15
Run List:    recipe[roles]
Roles:       example
Recipes:     example, example::default
Platform:    centos 7.8.2003
Tags:

$ sudo chef-client -z
// もちろんエラー
```
参考：https://docs.chef.io/workstation/knife_node/
</details>

<span id="a7">

## 7. 参考
- Chef Zero: https://umatomakun.hatenablog.com/entry/2016/06/05/122108
- Chef Server: https://qiita.com/jabba/items/d8f623b5a25ea1953b33