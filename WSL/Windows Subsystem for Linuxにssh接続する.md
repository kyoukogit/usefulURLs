https://qiita.com/ezmscrap/items/30eaf9531e240c992cf1

#はじめに

- Windows Subsystem for Linux(WSL)のターミナルは、コピー＆ペーストがし辛く、Tera-termで操作できないものかと思い立った。

# 前提
- Windows 10 pro バージョン1803
- Windows Subsystem for Linux(WSL)版ubuntu バージョン8.04 LTS (Bionic Beaver)

# 全体の流れ
1. インストール
2. コンフィグ設定
3. サービススタート
4. 対策
5. 補足

# インストール

- まず、openssh-serverをインストールする。

```
sudo apt install openssh-server
```

- 結果は以下の通り。
- 御免なさい。既に入っていました。確認してからコマンド打つのが正しいですよね。

```
Reading package lists... Done
Building dependency tree
Reading state information... Done
openssh-server is already the newest version (1:7.6p1-4).
The following packages were automatically installed and are no longer required:
  gyp javascript-common libc-ares2 libhttp-parser2.7.1 libjs-async libjs-inherits libjs-jquery libjs-node-uuid libjs-underscore libpython-stdlib
  libpython2.7-minimal libpython2.7-stdlib libssl-dev libssl-doc libuv1 libuv1-dev nodejs-doc python python-minimal python-pkg-resources python2.7
  python2.7-minimal
Use 'sudo apt autoremove' to remove them.
0 upgraded, 0 newly installed, 0 to remove and 5 not upgraded.
```

# コンフィグ設定
- パスワードでのログインを有効にしておく。

```
vi /etc/ssh/sshd_config
```

- コンフィグの内容を下記のように変える。

```
PasswordAuthentication yes
```

# サービススタート
- これで動くはず。

```
sudo service ssh restart
```

- 駄目だった。以下のようなメッセージが出る。
- RSAキーがないとな。

```
Could not load host key: /etc/ssh/ssh_host_rsa_key
Could not load host key: /etc/ssh/ssh_host_ecdsa_key
Could not load host key: /etc/ssh/ssh_host_ed25519_key
 * Restarting OpenBSD Secure Shell server sshd
Could not load host key: /etc/ssh/ssh_host_rsa_key
Could not load host key: /etc/ssh/ssh_host_ecdsa_key
Could not load host key: /etc/ssh/ssh_host_ed25519_key
```

# 対策
- よろしい手動作成だ。

```
sudo ssh-keygen -A
```

- 以下のように出力されてキーが出来る。

```
ssh-keygen: generating new host keys: RSA DSA ECDSA ED25519
```

- 再度、sshサービスを起動する。

```
sudo service ssh restart
```

- 以下のように表示されて無事起動。

```
 * Restarting OpenBSD Secure Shell server sshd            [ OK ]
```

- あとは、localhostでも、Windows機のIPアドレス指定でもばっちりteratermで接続できた。

# 補足
- `sudo ssh-keygen -A`は一度打てばよい
- Windows機を再起動した場合、サービスが停止した状態から始まる
- 重要なことに*sudo systemctl enableは効かない*ので、毎回、WIndows機起動時にはサービスを手動でスタートする必要がある
- このとき、sshdも立ち上がってないので、TeraTermは使えないので、簡単にサービスを起動できるシェルを作っておく

``` ~/service_start.sh
#!/bin/bash

service ssh start
service dbus start
service avahi-daemon start
service cron start
```

- 実行権限も忘れずに

```
chmod a+x service_start.sh
```

- これで、準備は完了。使い勝手が好みではない、標準ターミナルでも簡単にサービスを起動できる
- WSLの標準ターミナルを起動したら、以下を打てばよ

```
sudo ~/service_start.sh
```

- Enjoy WSL!
