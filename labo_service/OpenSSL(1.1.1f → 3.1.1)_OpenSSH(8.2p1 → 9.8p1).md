OpenSSL(1.1.1f → 3.1.1)、OpenSSH(8.2p1 → 9.8p1)のアップデート

2024/07/04
名前：山田純平

 
### 1.　概要
本資料は、OpenSSH、OpenSSLのアップデート手順について述べる。それぞれ元のバージョンはOpenSSHが8.2p1、OpenSSLが1.1.1fであり、新バージョンはそれぞれOpenSSHが9.8p1、OpenSSLが3.1.1である。
OpenSSH(9.8p1)、OpenSSL(3.1.1)のソースコードをそれぞれダウンロードし、ビルドを行い作成されたプログラムをインストールした。結論として、OpenSSHは9.8p1、OpenSSLは3.1.1にアップデートし、動作することを確認した。


### 2.　OpenSSLのアップデート手順

　本章は、OpenSSLのアップデートの手順について述べる．  

(1) 現在のバージョン確認
　現在使用しているバージョンを以下のコマンドを実行して確認する。
```
$ ssh -V
OpenSSH_8.2p1 Ubuntu-4ubuntu0.11, OpenSSL 1.1.1f  31 Mar 2020
```
(2) 必要なパッケージのインストール
　以下のコマンドを実行し、ソースコードのビルドに必要なツールのインストールを行う。
```
$ sudo apt install build-essential checkinstall zlib1g-dev
```
(3) ソースコードの取得
　GitHubからOpenSSLのソースコードをダウンロードする。今回目的としているバージョンは3.1.1なので、以下のように指定して取得する。
```
$ sudo su -
$ cd ~
$ git clone https://github.com/openssl/openssl -b openssl-3.1.1
$ cd openssl
```
(4) ビルド
　configファイルで設定を更新した後、makeコマンドを実行し、ビルドを行う。makeコマンドは時間がかかるので注意。
```
$ ./config --prefix=/usr/local/ssl --openssldir=/usr/local/ssl shared zlib
$ make
```
　上記の実行が完了したら、以下のコマンドを実行しテストとインストールを行う。
```
$ make test
$ make install
```
(5) インストール後の設定
　以下のコマンドで設定ファイルに追記する。
```
$ cat <<- __EOF__ | tee -a /etc/ld.so.conf.d/openssl-3.1.1.conf
/usr/local/ssl/lib64
__EOF__
```
　上記で記述した設定ファイルの内容を、以下のコマンドで反映する。
```
$ ldconfig -v
```
　既存のプログラムを別のディレクトリに退避する。なお退避場所のディレクトリは任意のディレクトリでよいが、今回は~/backupディレクトリを作成した後、そこへ移動させた。
```
$ cd ~; mkdir backup;
$ mv /usr/bin/c_rehash ~/backup/c_rehash.$(date +%Y%m%d)
$ mv /usr/bin/openssl ~/backup/openssl.$(date +%Y%m%d)
```
　以下のコマンドを実行し、下記のPATHを通す。
```
$ cat <<- __EOF__ | tee -a /etc/environment
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/usr/local/ssl/bin"
__EOF__
```
　以下のコマンドでパスを反映し、echoコマンドで反映した結果を確認する。
```
$ source /etc/environment
$ echo $PATH
```
　下記と出力されていればよい。
```
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/usr/local/ssl/bin"
```
　バージョンアップできているかを、下記のコマンドで確認し、3.1.1のバージョンになっていればOpenSSLのバージョンアップデート作業は終了である。
```
$ openssl version -a
```

### 3.　OpenSSHのアップデート手順
　本章は、OpenSSHのアップデートの手順について述べる。

(1) 現在のバージョン確認
```
$ ssh -V
```
　今回は先述のOpenSSLの方でも確認したため、出力結果については省略する。

(2) 必要なパッケージのインストール
　以下のコマンドを実行し、ソースコードのビルドに必要なツールのインストールを行う。
```
$ sudo apt install build-essential zlib1g-dev libssl-dev libpam0g-dev libselinux1-dev libkrb5-dev
```
(3) ディレクトリ作成と設定
　以下のコマンドを実行し、/var/lib/sshdディレクトリを作成後、権限設定を行う。
```
$ sudo mkdir /var/lib/sshd && sudo chmod -R 700 /var/lib/sshd/ && sudo chown -R root:sys /var/lib/sshd/
```
(4) ソースのダウンロードと展開
　ミラーサーバーからOpenSSHのソースコードをダウンロードする。今回目的としているバージョンは9.8p1なので、以下のように指定して取得する。ダウンロードしたファイルを展開し、展開後のディレクトリ内に移動する。
```
$ wget -c http://mirror.exonetric.net/pub/OpenBSD/OpenSSH/portable/openssh-9.8p1.tar.gz
$ tar -xzf openssh-9.8p1.tar.gz
$ cd openssh-9.8p1
```
(5) コンフィグ
　以下のコマンドを実行し、OpenSSLのプログラムの位置を確認する。
```
$ which openssl
```
　先述のOpenSSLの設定を行っていた場合は、以下のように出力されるはずである。
```
/usr/local/ssl/bin/openssl
```
　以下のコマンドを実行し、SSHサーバの設定の構成を設定する。
```
$ ./configure --with-kerberos5 --with-md5-passwords --with-pam --with-selinux --with-privsep-path=/var/lib/sshd/ --sysconfdir=/etc/ssh --with-ssl-dir=/usr/local/ssl
```

(6) ビルド
　makeコマンドを実行し、ソースコードをビルドする。
```
$ make
```
　以下のコマンドを実行し、ビルドしてできたプログラムをインストールする。
```
$ sudo make install
```
　バージョンアップできているかを、下記のコマンドで確認し、9.8p1のバージョンになっていればOpenSSHのバージョンアップデート作業は終了である。
```
$ ssh -V
```

4.　SSH接続の確認

　上記までの手順が完了したら、SSH接続ができるかどうかを改めて確認する。SSH接続ができて、OpenSSH、OpenSSLのバージョンが任意のものに変更できていれば完了である。

5.　結論

　結論として、OpenSSHは9.8p1、OpenSSLは3.1.1にアップデートし、動作することを確認した。

6.　参考文献

[1] Ubuntu20.04のOpenSSLを1.1.1から3.1.1にアップデート、＜https://manualmaton.com/2023/06/24/ubuntu20-04%e3%81%aeopenssl%e3%82%921-1-1%e3%81%8b%e3%82%893-1-1%e3%81%ab%e3%82%a2%e3%83%83%e3%83%97%e3%83%87%e3%83%bc%e3%83%88%e3%80%82/＞
[2] Ubuntu20.04のOpenSSHを8.2p1から9.8.1pにアップデート、＜https://manualmaton.com/2023/12/21/ubuntu20-04%e3%81%aeopenssh%e3%82%928-2p1%e3%81%8b%e3%82%899-6-1p%e3%81%ab%e3%82%a2%e3%83%83%e3%83%97%e3%83%87%e3%83%bc%e3%83%88%e3%80%82/＞
[3] OpenSSHの脆弱性 CVE-2024-6387についてまとめてみた、＜https://piyolog.hatenadiary.jp/entry/2024/07/02/032122＞