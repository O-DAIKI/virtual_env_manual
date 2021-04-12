# 環境構築手順書

<br />

## 【バージョン一覧】

<br />

| name    | Version         |
| ------- | --------------- |
| PHP     | 7.3.27          |
| Nginx   | 1.19.9          |
| CentOS  | 7.9.2009 (core) |
| MySQL   | 5.7.33          |
| Laravel | ※6.20.22        |

<br />
※Laravel 6.*系では最新のバージョンがインストールされます。
<br />
<br />

## 【構築手順】

<br />

### □ Vagrantのインストール

<br />

Vagrantをインストールします。
ターミナルを開いて下記のコマンドを実行してください。

```sh
$ brew install --cask vagrant
```

<br />

### □ vagrant boxのインストール

<br />

boxをダウンロードします。
コマンドを実行するディレクトリはどこでも構いません。

```sh
$ vagrant box add centos/7
```

<br />

コマンドを実行すると下記のような選択肢が表示されます。
```
1) hyperv
2) libvirt
3) virtualbox
4) vmware_desktop

Enter your choice: 3
```

<br />

今回使用するソフトはVirtualBoxのため、3を選択してenterを押しましょう。  
下記のように表示されたら完了です。
```
Successfully added box 'centos/7' (v1902.01) for 'virtualbox'!
```

<br />

### □ Vagrantの作業ディレクトリを用意する

<br />

Vagrantの作業用ディレクトリを作成します。

以下のいずれかのディレクトリの下に 任意の名前でディレクトリを作成しましょう。

<br />

* 自分の作業用ディレクトリ
* デスクトップ

<br/>

```
$ mkdir [フォルダ名]
```

<br />

作成したフォルダへ移動し、以下のコマンドを実行します。

```
$ cd [作成したフォルダ名]
# vagrant init box名 先ほどダウンロードしたboxを使用することになります
$ vagrant init centos/7

# 実行後問題なければ以下のような文言が表示されます
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

<br />

### □ Vagrantfileの編集

<br />

作成されたVagrantfileをエディターで開いて編集します。
今回行う編集は、三箇所です。

```
# 変更点①
config.vm.network "forwarded_port", guest: 80, host: 8080

# 変更点②
config.vm.network "private_network", ip: "192.168.33.19"  # 編集
```

上記二箇所の # がついているのを外します。  
また、今回使用するIPアドレスは 192.168.33.19 ですので、変更点②を忘れずに編集しましょう。  

また、以下の箇所はコメントインし変更を加えてください。

```
# 変更点③
config.vm.synced_folder "../data", "/vagrant_data"
# ↓ 以下に編集
config.vm.synced_folder "./", "/vagrant", type:"virtualbox"
```

<br />

### □ Vagrant プラグインのインストール

<br />

Vagrantには様々なプラグイン(拡張機能)が用意されています。  
今回は vagrant-vbguest というプラグインをインストールします。  
vagrant-vbguestは初めに追加したBoxの中にインストールされているGuest Additionsというもののバージョンを、VirtualBoxのバージョンに合わせて最新化してくれるプラグインです。  
それでは下記コマンドを実行しましょう。

```
$ vagrant plugin install vagrant-vbguest
```

<br />

vagrant-vbguestのインストールが完了しているか下記のコマンドを実行して確認しましょう。

```
$ vagrant plugin list
```

<br />

正常にプラグインがインストールされていれば下記のようにプラグインのリストとバージョンが表示されるはずです。

```
vagrant-vbguest (0.29.0, global)
```

<br />

### □ Vagrantを使用してゲストOSの起動

<br />

以上で仮想環境を構築する準備は整いました。  
Vagrantfileがあるディレクトリにて以下のコマンドを実行して、早速起動してみましょう。

```
$ vagrant up
```

<br />

初回の起動には、時間がかかりますので優雅にコーヒーブレイクでもしましょう。

<br />

### □ ゲストOSへのログイン

<br />

作成したvagrantの作業用ディレクトリに移動して下記のコマンドを実行しましょう。

```
$ vagrant ssh
```

<br />

コマンドを実行した後、以下のような表記になっていればゲストOSにログインしていることになります。

```
Welcome to your Vagrant-built virtual machine.
[vagrant@localhost ~]$
```

<br />

### □ パッケージをインストール

<br />

ゲストOSにパッケージをインストールしていきます。  
ゲストOSにログインした状態で下記のコマンドを実行してください。

```
$ sudo yum -y groupinstall "development tools"
```

<br />

上記のコマンドを実行することによりgitなどの開発に必要なパッケージを一括でインストールできます。

<br />

### □ PHPのインストール

<br />

PHPをゲストOSにインストールしていきます。  
下記コマンドを順に実行していってください。

```
$ sudo yum -y install epel-release wget
$ sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
$ sudo rpm -Uvh remi-release-7.rpm
$ sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip
$ php -v
```

<br />

PHPのバージョンが表示されれば正常にインストールが完了しています。

<br />

### □ composerのインストール

<br />

次にPHPのパッケージ管理ツールであるcomposerをインストールしていきます。

```
$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
$ php composer-setup.php
$ php -r "unlink('composer-setup.php');"

# どのディレクトリにいてもcomposerコマンドを使用できるようfileの移動を行います
$ sudo mv composer.phar /usr/local/bin/composer
$ composer -v
```

<br />

composerのバージョンの確認ができれば完了です。

<br />

### □ Laravelアプリケーションの作成

<br />

次にゲストOSで動かすためのLaravelアプリケーションを作成します。  
ゲストOSからログアウトしてからアプリケーション作成コマンドを実行します。

```
$ exit
$ composer create-project laravel/laravel --prefer-dist [アプリ名] 6.0
```

<br />

### □ 認証機能の実装  

<br />

作成したアプリにログイン機能を追加します。
先にマイグレーションを実行しておきましょう。

```
$ php artisan migrate
```

<br />

laravel/uiライブラリをインストールします。

```
$ composer require laravel/ui 1.*
$ php artisan ui vue --auth
```

以上になります。

<br />

### □ Laravelアプリケーションのコピー作成

<br />

先ほど作成したアプリケーションをゲストOS内で稼働させるためにアプリケーションのコピーを作成していきます。
vagrant作業用ディレクトリ下にコピーを作成しますので、下記コマンドを実行してください。

```
# vagrant作業用ディレクトリで実行
$ cp -r laravel_appディレクトリまでの絶対パス ./
```

<br />

### □ データベースのインストール

<br />

MySQLをインストールしていきます。  
versionは5.7を使用します。

rpmに新たにリポジトリを追加し、インストールを行います。
ゲストOSにログインした状態でコマンドを実行してください。

```
$ sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
$ sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm
$ sudo yum install -y mysql-community-server
$ mysql --version
```

<br />

MySQLの初期パスワードは less /var/log/mysqld.log に発行されます。
まずはpasswordを調べ、接続しpassswordの再設定を行っていきましょう。

```
$ sudo cat /var/log/mysqld.log | grep 'temporary password'  # このコマンドを実行したら下記のように表示されたらOKです
2017-01-01T00:00:00.000000Z 1 [Note] A temporary password is generated for root@localhost: hogehoge
```

<br />

**hogehoge**と記載されている箇所に存在するランダムな文字列がパスワードとなります。
下記コマンドを実行しMySQLへ接続しましょう。

```
$ sudo systemctl start mysqld
$ mysql -u root -p
Enter password:
```

<br />

次はpasswordの変更を行います。

```
mysql > set password = "新たなpassword";
```

<br />

passwordの変更が完了したら、  
実際にLaravelのTodoアプリケーションを動かす上で使用するデータベースの作成を行います。

```
mysql > create database [データベース名];
```

<br />

Query OKと表示されたら作成は完了となります。

次にlaravelアプリケーションの **.env** ファイルを開き以下の編集を加えてください。

```
DB_PASSWORD=
# ↓ 以下に編集
DB_PASSWORD=登録したパスワード
```

<br />

では、laravel_appディレクトリに移動して **php artisan migrate** を実行します。

<br />

### □ Nginxのインストール

<br />

Nginxの最新版をインストールしていきます。
viエディタを使用して以下のファイルを作成します。

```
$ sudo vi /etc/yum.repos.d/nginx.repo
```

<br />

書き込む内容は以下になります。

```console
[nginx]
name=nginx repo
baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
gpgcheck=0
enabled=1
```

<br />

書き終えたら保存して、以下のコマンドを実行しNginxのインストールを実行します。

```
$ sudo yum install -y nginx
$ nginx -v
```

<br />

Nginxのバージョンは確認できたでしょうか？
ではNginxの起動をしましょう。

```
$ sudo systemctl start nginx
```

<br />

ブラウザにて http://192.168.33.19 (Vagrantfileでipを書き換えた方はそのipアドレス)と入力し、NginxのWelcomeページが表示されましたでしょうか？  
表示されたら問題なく動いていますので次に進みましょう。

<br />

### □ Laravelを動かす

<br />

Nginxの設定ファイルを編集していきます。
使用しているOSがCentOSの場合、/etc/nginx/conf.d ディレクトリ下の default.conf ファイルが設定ファイルとなります。

```
$ sudo vi /etc/nginx/conf.d/default.conf
```

<br />

```
server {
  listen       80;
  server_name  192.168.33.19; # Vagranfileでコメントを外した箇所のipアドレスを記述してください。
  # ApacheのDocumentRootにあたります
  root /vagrant/[アプリ名]/public; # 追記
  index  index.html index.htm index.php; # 追記

  #charset koi8-r;
  #access_log  /var/log/nginx/host.access.log  main;

  location / {
      #root   /usr/share/nginx/html; # コメントアウト
      #index  index.html index.htm;  # コメントアウト
      try_files $uri $uri/ /index.php$is_args$args;  # 追記
  }

  # 省略

  # 該当箇所のコメントを解除し、必要な箇所には変更を加える
  # 下記は root を除いたlocation { } までのコメントが解除されていることを確認してください。

  location ~ \.php$ {
  #    root           html;
      fastcgi_pass   127.0.0.1:9000;
      fastcgi_index  index.php;
      fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更
      include        fastcgi_params;
  }

  # 省略
```

<br />

Nginxの設定ファイルの変更は、以上です。
次に php-fpm の設定ファイルを編集していきます。

```
$ sudo vi /etc/php-fpm.d/www.conf
```

<br />

変更箇所は以下になります。

```
;24行目近辺
user = apache
# ↓ 以下に編集
user = nginx

group = apache
# ↓ 以下に編集
group = nginx
```

<br />

設定ファイルの変更に関しては、以上となります。
では早速起動しましょう(Nginxは再起動になります)。

```
$ sudo systemctl restart nginx
$ sudo systemctl start php-fpm
```

<br />

先程php-fpmの設定ファイルの user と group を **nginx** に変更したと思いますが、  
現在ファイルとディレクトリの実行 user と group に **nginx** が許可されていません。  
以下のコマンドを実行して **nginx** というユーザーでもログファイルへの書き込みができる権限を付与してあげましょう。

```
$ cd /vagrant/[作成したアプリ名]
$ sudo chmod -R 777 storage
```

<br />

それでは http://192.168.33.19/ へアクセスしましょう。
laravelのウェルカムページが表示されていれば完了です。

<br />
<br />

## 【環境構築の所感】

<br />

### □ vagrant snapshot機能について

<br />

環境構築を行っていくにあたって、コマンド実行する場所の誤りや設定ファイルの編集ミス等に起因するエラーが頻出しました。  
修正するにあたってエラーが起きる前の状態に戻したいと感じることが多々ありました。  
そこで一度saharaの使い方について調べたのですが、現在のvagrantのバージョンだとsnapshotという機能が使えることを知り、早速使ってみました。
使い方は非常にわかりやすく、  
```
$ vagrant snapshot save [セーブポイント名]
```
上記コマンドでセーブポイントを設定し、
```
$ vagrant restore [セーブポイント名]
```
で任意のセーブポイントに戻せます。
```
$ vagrant snapshot list
```
上記コマンドで作成したセーブポイントの一覧が確認でき、任意のタイミングへいつでも戻すことができたので、作業効率が上がりました。  
vagrantでの環境構築の際は必須の機能だと感じました。

<br />

### □ viでの設定ファイル編集

<br />

一番多かったエラーがviでの設定ファイル編集でのミスタイプでした。  
編集の際は念入りに確認し、またエラーログから仮説を立ててエラーの原因となる行の特定をすることも有効だと感じました。  

<br />

### □ laravel6.0 ログイン画面のレイアウトの崩れについて

<br />

laravelバージョン6.0のアプリケーションでログイン機能を実装した時、 **npm run dev** コマンドでエラーが起きてしまい、ログイン画面のcssが適用されていない状態になっていました。  
解決方法として、  
①package.jsonファイルを開く  
②ファイルの中をsass-loaderで検索する  
③バージョンが８になっているので、7に変更する。
```
"sass-loader": "^7.0.0",
```
④node_modulesを消す。
```
rm -rf node_modules
```
⑤npmのinstall
```
npm install
```
以上の手順で解決しました。  
バージョン関係でのエラーは頻出のようなので、優先的にチェックしていこうと感じました。

<br />
<br />

## 【参考サイト】

<br />

### ・ReaDouble
[セキュリティ: 認証](https://readouble.com/laravel/6.x/ja/authentication.html)

<br />

### ・Qiita

[# Vagrant + VirtualBOx で 最新のCentOS7 vbox(centos/7 2004.01)でマウントできない問題](https://qiita.com/mao172/items/f1af5bedd0e9536169ae)

[Laravel6 ログイン機能を実装する](https://qiita.com/ucan-lab/items/bd0d6f6449602072cb87)

[Laravel6でauthを入れるときnpm run devで　ERROR in ./resources/sass/app.scss　が発生したら？](https://qiita.com/econbusi/items/b28996a3024f3632d719)

<br />

### ・WEB ARCH LABO

[VagrantのSnapshot機能で仮想マシンの状態を保存/復元しよう](https://weblabo.oscasierra.net/vagrant-snapshot/)

<br />

### ・Wedding Park CREATERS Blog

[Nginxで403 Forbiddenが表示された時のチェックポイント5選](https://engineers.weddingpark.co.jp/nginx-403-forbidden/)