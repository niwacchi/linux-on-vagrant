# Linux on Vagrant
## 概要
- Windows上に手軽にLinuxを学ぶための環境を構築する手順を説明します。
- プロキシに関わる設定はプロキシサーバーを使用環境でのみ考慮してください。
### 使用OSバージョンなど
- Windows8.1 Pro
- VirtualBox
- Vagrant

### 環境構築
- VirtualBoxのインストール [[https://www.virtualbox.org/:https://www.virtualbox.org/]]
- vagrantのインストール [[http://www.vagrantup.com/:http://www.vagrantup.com/]]
- CentOSのインストール [[http://www.vagrantbox.es/:http://www.vagrantbox.es/]] ※boxファイルの一覧

- 環境変数の設定
```sh
HTTP_PROXY=http://{プロキシサーバーのIPアドレス}:{ポート番号}
HTTPS_PROXY=http://{プロキシサーバーのIPアドレス}:{ポート番号}
```

- boxファイルを使ってインストール
```sh
vagrant box add {好きな名前} {boxファイルのurl}
 
# (例)
vagrant box add CentOS6 http://developer.nrel.gov/downloads/vagrant-boxes/CentOS-6.4-i386-v20130731.box
```

## 動作確認
### 初期化
```sh
 (例)
 vagrant init CentOS6
```

### 起動
```sh
 vagrant up
```

### SSHの確認
```sh
 vagrant ssh
```

#### SSHクライアントでの接続
- puttyなどを利用する。
- ユーザはvagrant、パスワード同じ。

### 停止
```sh
 vagrant halt
```

## プロキシ設定
### 基本設定
#### /etc/environment
```sh
http_proxy="http://{プロキシサーバーipアドレス}:{ポート番号}"
https_proxy="http://{プロキシサーバーipアドレス}:{ポート番号}"
ftp_proxy="http://{プロキシサーバーipアドレス}:{ポート番号}"
```

### シェルの設定
#### /etc/profile.d/proxy.sh
```sh
export http_proxy="http://{サーバーipアドレス}:{ポート番号}"
export https_proxy="http://{サーバーipアドレス}:{ポート番号}"
export ftp_proxy="http://{サーバーipアドレス}:{ポート番号}"
```

#### /etc/profile.d/proxy.csh
```sh
setenv http_proxy="http://{プロキシサーバーipアドレス}:{ポート番号}"
setenv https_proxy="http://{プロキシサーバーipアドレス}:{ポート番号}"
setenv ftp_proxy="http://{プロキシサーバーipアドレス}:{ポート番号}"
```

### yumの設定
#### /etc/yum.conf
```sh
proxy=http://{プロキシサーバーipアドレス}:{ポート番号}
```

### プロキシ環境下でのRubyGems
```sh
gem update --system -p http://{プロキシサーバーipアドレス}:{ポート番号}
```

### gitの設定
```sh
git config --global http.proxy http://{プロキシサーバーipアドレス}:{ポート番号}
git config --global https.proxy http://{プロキシサーバーipアドレス}:{ポート番号}
```

## Apache
### apacheのインストール
```sh
# インストール
sudo yum install httpd

# 自動起動設定
sudo chkconfig httpd on

# 起動
sudo /etc/init.d/httpd start
```

### ホストOSからhttpアクセス
- Vagrantfileの変更。以下のコメントアウトを外す。
 ```sh
 config.vm.network :private_network, ip:"192.168.33.10"
 ```
- vagrant halt
- vagrant up ※このときにVirtul-Boxのネットワークアダプタインストールが実行される。
- ブラウザで「http://192.168.33.10」にアクセス
 

## MySQL
```sh
# インストール
sudo yum install mysql-server

# 起動設定
sudo chkconfig mysqld on

# 起動
sudo /etc/init.d/mysqld start

# 初期化
sudo mysql_install_db

```
## Git
### 最新のgitのインストール
```sh
# yumでgitを含む必要なパッケージをインストール
sudo yum install -y make curl-devel gcc openssl-devel expat-devel  cpan gettext  asciidoc xmlto git

# 最新のgitのソースをcloneしてコンパイルしてインストール
git clone git://github.com/git/git.git
cd git/
make configure
./configure --prefix=/usr/local
make all doc
sudo make install install-doc install-html

# 最新のgitを使うように設定
(~/.bashrc)
alias git='/usr/local/bin/git'
```

## PHP
### 最新のPHPのインストール
```sh
# epelのリポジトリ追加
sudo rpm --httpproxy 192.168.0.5 --httpport 3128 -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm

# remiのリポジトリ追加
sudo rpm --httpproxy 192.168.0.5 --httpport 3128 -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm

# phpのインストール ※ここでは最低限のphpのみ
sudo yum install --enablerepo=remi --enablerepo=remi-php55 php
```

## Ruby
```sh
# 事前の準備
sudo yum -y update
sudo yum -y groupinstall "Development Tools"
sudo yum install -y zlib-devel libxml2-devel libxslt-devel openssl-devel readline-devel sqlite-devel mysql-devel libyaml-devel libffi-devel iconv-devel nkf pcre-devel

# rbenv
git clone git://github.com/sstephenson/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
exec $SHELL -l
git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build

# ruby本体のインストール
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
exec $SHELL -l
rbenv install 2.0.0-p195

# 利用方法
rbenv global 2.0.0-p195
rbenv exec gem install bundler
rbenv rehash

```

## Node
- ...作成中

## debパッケージをrpmパッケージに変換
### alienの導入
```sh
#事前準備
sudo yum install -y rpmbuild perl-ExtUtils-MakeMaker

#alienのダウンロード
wget http://ftp.debian.org/debian/pool/main/a/alien/alien_8.90.tar.gz

#alienのrpmをビルド
sudo rpmbuild -ta alien_8.90.tar.gz

#alienのrpmをインストール
rpm -Uivh /root/rpmbuild/RPMS/noarch/alien-8.90-1.noarch.rpm
```
