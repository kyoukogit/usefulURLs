https://qiita.com/ryuichi1208/items/4151110327584c971f11

## 概要

サーバにSSHで入ろうとして失敗するときの原因特定までの流れを書いた記事。
失敗の原因はデバッグモードで起動することでなんとなく見えてくる

``` bash
$ ssh -vvv {hostname}
```

後はログを見て解決しましょう。

``` bash
$ less /var/log/auth.log
```

## OpenSSHとは

OpenSSH (Open Secure Shell) は、SSHプロトコルを利用するためのソフトウェア
以下は主要なコマンド。原因調査というよりは問題解決に使うコマンドですね。

* ssh-keygen
* ssh-copy-id

日本語のマニュアルは下記
https://euske.github.io/openssh-jman/

## 失敗したときにとりあえず試すこと

失敗したらログ見る前にとりあえず以下を試してます。
サーバ構築直後だったりすると設定ミスとかを疑います。

* Pingが通ること
* ユーザ名・パスワード
* sshdサービスが起動していること
* SSHで利用するポートが開いていること
* ログインの認証方式が正しいこと

## エラーメッセージが出力される場合

TeraTermとかで直接入るわけではなくサーバを経由して入る場合はエラーメッセージが見れます。
それをヒントに調査していきます。
以下はよくあるメッセージと対処法です。

#### Temporary failure in name resolution

名前解決に失敗しているときに出るメッセージ
DNSや「/etc/hosts」の設定を疑ってみましょう。

#### No route to host

サーバー側の SSH ポートと合っていない場合に出力される
以下2点を疑いましょう。

* サーバー側の SSH サービスのポート設定
* サーバー側のファイアーウォールの許可設定で SSH のポートが上記と同じポート番号であるか

``` /etc/ssh/sshd_config
#Port 22 
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

# 書き換えたらリスタート
$ sudo systemctl restart ssh
```

設定ファイルではPORTでSSHのポートが設定できる。
これが22以外になっている場合はSSHクライアント側でポートの指定が必要
「-p PORT番号」で指定できます。

ファイアウォールの場合はSSHを許可する
閉じた環境ならそもそもfirewalldを停止させてもいい気がする()

``` bash
# sshのデフォルトポートを許可
$ sudo ufw allow ssh

# 指定したポートを許可。ついでにデフォルトポートを閉じる
$ sudo ufw deny ssh
$ sudo ufw allow 2222
$ sudo ufw enable

# 書き換えたらリスタート
$ sudo systemctl restart ssh
```

#### REMOTE HOST IDENTIFICATION HAS CHANGED!

``` bash
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that the RSA host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
29:24:c2:69:a3:b0:dc:4d:23:fc:9d:85:9f:ea:01:9b.
Please contact your system administrator.
Add correct host key in /home/grgrjnjn/.ssh/known_hosts to get rid of this message.
Offending key in /home/grgrjnjn/.ssh/known_hosts:3
RSA host key for remote_host has changed and you have requested strict checking.
Host key verification failed.
```

SSHでは、安全な接続を行うために接続先サーバのRSA公開鍵のフィンガープリントを、クライアントは保存する。SSH接続時には、以前保存したこの情報と、いままさに接続しようとしているサーバの情報が一致しているかを確認する。この時にサーバ情報が一致しないと上みたいなメッセージが出る。
OS入れ替えとか鍵の再生成とかするとIPは同じでも出力されるメッセージ
下記で対応できる

```
$ ssh-keygen -R hostname
```

やってることは~/.ssh/known_hostsから該当するサーバの情報を削除している。
これによりサーバ情報をいったん削除し新たにSSHを行う。

#### Permission denied

ユーザ名/パスワードの確認は必須としてあとは鍵関連のパーミッションだったりを疑う
前に一回あったのがユーザ名/パスワードはあってるのにだめで原因を探るとユーザがロックされているパターン


■サーバー側

* ホームディレクトリのパーミッションが755かそれより厳しくなってる
* .sshディレクトリのパーミッションが0700になっている
* .ssh/authorized_keysのパーミッションが0600になっている

■クライアント側

* ホームディレクトリのパーミッションが755かそれより厳しくなってる
* .sshディレクトリのパーミッションが0700になっている
* 秘密鍵のパーミッションが0600になっている

#### Warning: Identity file /* not accessible

-iなどで鍵ファイルを指定した際に読込みに失敗したか、その指定ディレクトリに該当の鍵がないという警告です。

``` bash
$ ssh -i 鍵のパス remote_host
Warning: Identity file /* not accessible: No such file or directory.
```

#### Received disconnect from * ~

ユーザー名での認証をした回数が制限を超えてしまったとき出る
サーバ側でユーザロック解除が必要。

``` bash
Received disconnect from [*IPアドレス]: 2: Too many authentication failures for [*username]
```

#### SSH protocol v.1 is no longer supported

古いルータや機器だとsshのバージョンが古いときに出るメッセージ
sshクライアントがmacOSだと出るらしいです。
どのバージョンからサポートが切れたのかは不明ですがEl Capitan時代はv1も使えたとのこと。自前で昔のopensshのインストールする必要があるらしい(未検証)

``` bash
$ ssh -V
OpenSSH_7.8p1, LibreSSL 2.7.3
```

## sshのちょっとした小技

#### パスなしログインの設定を簡略化

鍵認証による ssh を行おうとすると ssh する元のサーバで公開鍵を生成してそれを ssh 先のサーバに持って行って.ssh/authorized_keysに追記しないといけない。
これを毎回手動でやってたがssh-copy-idというコマンドで一発でできる

``` bash
$ ssh-copy-id ${USER}@${remote_host}

# -iオプションで任意の公開鍵を指定することもできる
$ ssh-copy-id -i ${identity_file} ${USER}@${remote_host}
```

#### ユーザ指定がめんどくさいとき

複数ホストへ特定のユーザで入る必要があるときユーザ名をsshで毎回打つのがめんどくさいときは~/.ssh/configに設定を行えばユーザ指定なしで特定のユーザで接続を行ってくれる。

``` ~/.ssh/config
Host 任意の接続名(hoge)
# Host HOST_??のように書けばHOST_01,HOST02などを一括で認識する
HostName ホスト名
User ユーザー名
Port ポート番号
```

上記のように設定しておけばあとはssh remoteホストで指定ポート/ユーザで接続してくれる。

| 項目 | 説明 |
|:--|:--|
| Host | SSH 接続する際に利用する名前 |
| Hostname | 接続先ホストのアドレス、または FQDN を指定 |
| Port | 接続先ホストで sshd が Listen しているポートを指定 |
| IdentityFile | 秘密鍵のパスを指定 |
| User | SSH 接続する際に利用するユーザ名を指定 |
| ProxyCommand | 接続先ホストで実行するコマンドを指定 |

ProxyCommandを設定しておけば多段接続が必要なマシンへのログインもとても楽になる
[多段ssh設定のまとめ](https://rcmdnk.com/blog/2014/06/08/comptuer-linux-windows-putty/)

## まとめ

以上がSSHできないときの個人的な調査観点でした。
とりあえず-vvvオプションとauth.log見とけば解決することが多いです。

## 参考リンク

[SSHサーバへ接続出来ない・遅い時の原因と対処法](https://orebibou.com/2014/12/ssh%E3%82%B5%E3%83%BC%E3%83%90%E3%81%B8%E6%8E%A5%E7%B6%9A%E5%87%BA%E6%9D%A5%E3%81%AA%E3%81%84%E3%83%BB%E9%81%85%E3%81%84%E6%99%82%E3%81%AE%E5%8E%9F%E5%9B%A0%E3%81%A8%E5%AF%BE%E5%87%A6%E6%B3%95/)
[インフラエンジニアじゃなくても押さえておきたいSSHの基礎知識](https://qiita.com/tag1216/items/5d06bad7468f731f590e)
