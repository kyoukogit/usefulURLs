https://qiita.com/tag1216/items/5d06bad7468f731f590e

最近はクラウド上のサーバーを利用する事も多くなってきた。
サーバーの用意やネットワーク周りの設定はインフラ部門がやってくれるけど、アプリのデプロイ／設定は開発者がする事が多いので、開発メインでやってるエンジニアでも最低限SSHの知識は必要になる。

また、Vagrant等でローカル環境にVMを作成する事もあるので、ローカル環境内でSSHを使用するケースも増えてきた。

というわけでインフラエンジニアじゃなくてもSSHクライアントの知識は必須になってきているので、改めてSSHの再学習をしてみることにした。

# SSHとは

暗号や認証の技術を利用して、安全にリモートコンピュータと通信するためのプロトコル。

SSHでは以下の点で従来のTelnetより安全な通信が行える。[^1]

- パスワードやデータを暗号化して通信する。
- クライアントがサーバーに接続する時に、接続先が意図しないサーバーに誘導されていないか厳密にチェックする。

詳しくは[Wikipedia](http://ja.wikipedia.org/wiki/Secure_Shell)で

# SSHの認証方式

SSHの主要な認証方式としてはパスワード認証方式と公開鍵認証方式がある。

<dl>
<dt>パスワード認証方式</dt>
<dd>パスワード認証方式はデフォルトの認証方式で、ユーザー名とパスワードでログインする方式。<br/>
ユーザー名とパスワードは接続先OSのユーザーアカウントの情報が使用される。<br/>
</dd>
<dt>公開鍵認証方式</dt>
<dd>公開鍵認証方式は公開鍵と秘密鍵の２つの鍵(キーペア)を使用した接続方式。<br/>
サーバーに公開鍵、クライアントに秘密鍵を置いて使用する。<br/>
公開鍵認証を使うとパスワード入力なしでログインする事が可能になる。
</dd>
</dl>

パスワード認証方式はサーバー側で明示的に無効にしていない限り使用できるが、セキュリティ的には脆弱なので無効にしている事も多い。[^2]

公開鍵の仕組みというか考え方については以下が分かりやすいと思う。
[妻に公開鍵暗号を教えてみた - 西尾泰和のはてなダイアリー](http://d.hatena.ne.jp/nishiohirokazu/20140809/1407556873)

# OpenSSH

SSHプロトコルを使用する為のソフトで、SSHクライアントとSSHサーバーの両方が含まれている。

Linuxではデファクトスタンダードとなっていて、ほぼデフォルトでインストールされている。
WindowsでもCygwinをインストールしたり、Gitに同梱されているGit Bashで使用できる。

[OpenSSH 日本語マニュアルページ](http://euske.github.io/openssh-jman/)

# OpenSSHの主要コマンド

<dl>
<dt>ssh</dt>
<dd>SSHでリモートホストに接続してコマンドを実行する。</dd>
<dt>scp</dt>
<dd>SSHを使用してリモートホストとの間でファイルを転送を行う。
<dt>ssh-keygen</dt>
<dd>公開鍵認証方式で使用するキーペアを生成する。</dd>
<dt>ssh-copy-id</dt>
<dd>公開鍵をリモートホストに登録するコマンド。環境によってはインストールされていない場合がある。</dd>
</dl>

# sshコマンド

sshコマンドはリモートホストに接続してコマンドを実行する時に使用する。

```bash:
ssh [user@]hostname [command]
```

- ユーザー名を省略するとクライアントの現在のユーザーが使用される。
- コマンドを指定するとリモートホストに接続したあと指定のコマンドだけ実行してログアウトする。
- コマンドを省略した場合はリモートホストにログインした状態でコマンドプロンプトが表示されるので、任意のコマンドを実行できる。ログアウトしたい時は`exit`。

主要なオプション

<dl>
<dt>-i identity_file</dt>
<dd>公開鍵認証で使用する秘密鍵ファイルを指定する。デフォルトでは~/.ssh/id_rsaなどが使用される。</dd>
<dt>-F configfile</dt>
<dd>設定ファイルを指定する。デフォルトでは`~/.ssh/config`が使用される。</dd>
<dt>-p</dt>
<dd>ポート番号を指定する。</dd>
</dl>

上記オプションは他のコマンドでもほぼ共通。

[OpenSSH 日本語マニュアルページ - SSH](http://euske.github.io/openssh-jman/ssh.html)

# scpコマンド

scpコマンドはSSHを使用してリモートホストとの間でファイルを転送を行うコマンド。

```bash:クライアントのfile1とfile2をリモートホストのdir1へコピーする
scp file1 file2 [user@]hostname[:dir1]
```

```bash:リモートホストのfile1とfile2をクライアントのdir1へコピーする
scp [user@]hostname:file1 [user@]hostname:file2 dir1
```

[OpenSSH 日本語マニュアルページ - SCP](http://euske.github.io/openssh-jman/scp.html)

# パスワード認証方式での接続

```shell-session
[vagrant@node1 ~]$ ssh node2 ←node1からSSHでnode2へ接続
The authenticity of host 'node2 (192.168.33.12)' can't be established.
RSA key fingerprint is c4:4d:f9:05:09:31:33:05:cd:99:52:5b:fc:e0:10:b5.
Are you sure you want to continue connecting (yes/no)? yes ←初めて接続するサーバーの場合に確認を求められる。
Warning: Permanently added 'node2,192.168.33.12' (RSA) to the list of known hosts.
vagrant@node2's password: ←パスワードを入力
Last login: Fri Mar  7 16:57:20 2014 from 10.0.2.2
[vagrant@node2 ~]$ ←node2のコマンドプロンプト
[vagrant@node2 ~]$ pwd
/home/vagrant
[vagrant@node2 ~]$ exit
logout
Connection to node2 closed.
[vagrant@node1 ~]$ 
```

# 公開鍵認証方式での接続

## キーペアの作成

公開鍵認証で使用するキーペアは`ssh-keygen`コマンドで生成する。

```shell-session
[vagrant@node1 ~]$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa): ←生成する秘密鍵ファイル名を入力
Enter passphrase (empty for no passphrase): ←パスフレーズを入力
Enter same passphrase again: 
Your identification has been saved in /home/vagrant/.ssh/id_rsa.
Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub.
The key fingerprint is:
c4:bf:ba:b6:e0:40:82:8c:29:ca:ee:ae:96:e4:07:c6 vagrant@node1
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|       .         |
|        o        |
|oo     . .       |
|*.. .   S .      |
|+E o       .     |
|=.o . .   .      |
|.+ . o ...       |
|*+.   ..+o       |
+-----------------+
```

パスフレーズは秘密鍵を暗号化するためのもので、パスワード認証方式のパスワードとは別のもの。
パスフレーズを未設定にすることで、以後の接続でパスワード／パスフレーズの入力なしで接続することができるようになる。

## 公開鍵をサーバーに登録する

ssh-copy-idを使うと、リモートホストの指定ユーザーの`~/.ssh/authorized_keys`に公開鍵が追加される。

```shell-session
[vagrant@node1 ~]$ ssh-copy-id node2
vagrant@node2's password: 
Now try logging into the machine, with "ssh 'node2'", and check in:

  .ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.

```

ssh-copy-idが使えない場合は、sshコマンドでリモートホストに接続して`authorized_keys`に公開鍵を追加する。

```shell-session
[vagrant@node1 ~]$ cat ~/.ssh/id_rsa.pub | ssh node3 'cat >> ~/.ssh/authorized_keys; chmod 600 ~/.ssh/authorized_keys'
The authenticity of host 'node3 (192.168.33.13)' can't be established.
RSA key fingerprint is c4:4d:f9:05:09:31:33:05:cd:99:52:5b:fc:e0:10:b5.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'node3,192.168.33.13' (RSA) to the list of known hosts.
vagrant@node3's password: 
```

**`authorizedkeys`は複数の公開鍵を登録するファイルなので、既存の情報を消してしまわないように追加モードで書き込む事。**

公開鍵をサーバーに登録した後にsshコマンドで接続すると、パスワードを聞かれずにログインできるようになる。(パスフレーズを設定している場合はパスフレーズを聞かれる)

```shell-session
[vagrant@node1 ~]$ ssh node3
Last login: Fri Mar  7 16:57:20 2014 from 10.0.2.2
[vagrant@node3 ~]$ 
```

# ~/.ssh/内のファイル

<dl>
<dt>authorized_keys</dt>
<dd>接続を許可する公開鍵を登録しておくサーバー側のファイル。</dd>
<dt>config</dt>
<dd>SSH接続の情報を書くファイル。</dd>
<dt>id_rsa</dt>
<dd>ssh-keygenで生成した秘密鍵。</dd>
<dt>id_rsa.pub</dt>
<dd>ssh-keygenで生成した公開鍵。</dd>
<dt>known_hosts</dt>
<dd>過去に接続したことがあるサーバー。</dd>
</dl>

OpenSSHでは`~/.ssh/`内のファイルがモード600(ユーザーのみ読み書き可能)でないと使用できない。

# ~/.ssh/config

`~/.ssh/config`にはSSH接続時の情報を定義しておくことができるので、sshコマンドのオプションを省略したり、コマンドラインオプションでは指定できない情報も設定できる。

```:~/.ssh/configの例
Host web01
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /vagrant/.vagrant/machines/default/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL  
```

主要な設定項目

<dl>
<dt>Host</dt>
<dd>sshコマンド等で指定するホスト名。</dd>
<dt>HostName</dt>
<dd>実際に接続ホストのアドレス。IPアドレスや/etc/hostsに指定したホスト名等を指定する。</dd>
<dt>User</dt>
<dd>ユーザー名</dd>
<dt>Port</dt>
<dd>ポート番号</dd>
<dt>IdentityFile</dt>
<dd>秘密鍵ファイル</dd>
<dd>
</dl>

以下のサイトでおそらく全設定項目が確認できる。
[OpenSSH 日本語マニュアルページ - SSH_CONFIG](http://euske.github.io/openssh-jman/ssh_config.html)

# 踏み台サーバー/SSHゲートウェイ/SSHプロキシ経由での多段SSH接続

現在使用しているマシンから作業対象マシンに接続したい時に、あるサーバーを経由しないといけないケースがある。

こういう経由しなければならないサーバーの事を「踏み台サーバー」とか「SSHゲートウェイ」とか「SSHプロキシ」とかいう。(以下、踏み台サーバー[^3]）

踏み台サーバー経由で作業対象サーバーにSSH接続する場合、普通にやると先ず踏み台サーバーにsshコマンドで接続して、そこからさらに作業対象サーバーにsshコマンドで接続することになる。

```shell-session:node2を経由してnode3に接続
[vagrant@node1 ~]$ ssh node2
Last login: Sun Feb 22 15:39:03 2015 from 192.168.33.11
[vagrant@node2 ~]$ ssh node3
Last login: Sun Feb 22 15:39:09 2015 from 192.168.33.12
[vagrant@node3 ~]$ 
```

この接続方法だとコマンドを２回実行するので面倒だし、踏み台サーバー上に秘密鍵を置いたりすることになるため、セキュリティ上あまり好ましくない(らしい)。

こういう時には`~/.ssh/config`に多段SSH接続の情報を設定しておくと、踏み台サーバーに秘密鍵を置く必要もなく、コマンド一発で接続できるようになる。

以下はnode2を経由してnode3に接続する設定例。

```:~/.ssh/config
Host node2

Host node3
  ProxyCommand ssh -W %h:%p node2
```
```shell-session:node3に直接ログインできるようになる
[vagrant@node1 ~]$ ssh node3
Last login: Sun Feb 22 15:39:48 2015 from 192.168.33.12
[vagrant@node3 ~]$ exit
```

node2->node3->node4というように複数の踏み台サーバーを経由する設定もできる。

```:~/.ssh/config
Host node2

Host node3
  ProxyCommand ssh -W %h:%p node2

Host node4
  ProxyCommand ssh -W %h:%p node3
```

# WindowsでSSHを使用する

Windowsは標準ではSSHが使えるソフトはインストールされていない。
WindowsでSSHを使用するにはいくつかの方法がある。

## Linuxコマンドが使える系ソフトを使用する

CygwinやGitをインストールするとOpenSSHが使えるようになる。
ただしLinux版と比較すると動作が若干不安定。

## LinuxのVMを作成する

VMWareやVirtualBoxでLinuxのVMを作成してVMからSSHを使用する。
LinuxのOpenSSHを使用するので動作は安定している。

## GUI系SSHクライアント

[TeraTerm](http://sourceforge.jp/projects/ttssh2/)とか[Putty](https://www.google.co.jp/search?q=putty&oq=putty&aqs=chrome..69i57j0l5.957j0j4&sourceid=chrome&es_sm=119&ie=UTF-8)とか。

画面上で接続設定をして保存できるので便利。
パスワード認証のパスワードや公開鍵認証のパスフレーズも保存できるので接続時に入力不要になる。

踏み台サーバー経由の接続設定もできるが、複数の踏み台を経由する接続には対応していない。

キーペアの作成も行えるが、Puttyではちょっと特殊なものが作成されるので注意が必要。

-----

[^1]: ネットワーク上のサーバーに接続して作業する場合、１０年くらい前まではTelnetを使用することが多かったが、Telnetではパスワードを平文で送信するためセキュリティ上の危険性があった。

[^2]: Linuxではrootユーザーは必ず存在するしその他のユーザーも個人名やアプリ名(postgresやmongo等)を使う事が多いので発覚しやすい。パスワードも人間が入力するものなので覚えやすい文字列(ユーザー名+1234等)を使う事が多い為、パスワード認証方式を許可しているとセキュリティ的に脆弱になってしまう。

[^3]: 踏み台サーバー/SSHゲートウェイ/SSHプロキシは正確にはそれぞれ役割が違うかもしれないけどよくわかんない。
