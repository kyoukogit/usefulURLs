http://my-web-site.iobb.net/~yuki/2018-12/soft-tool/wsl-ubuntu1804-ssh-pwauth/
# sshサーバ設定
WSL UbuntuにSSH通信ができるように/etc/ssh/sshd_configを設定します。最初に設定ファイルをバックアップしてから編集します。

```
$ sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.org
$ sudo vi /etc/ssh/sshd_config

   （2カ所修正します）

$ diff /etc/ssh/sshd_config /etc/ssh/sshd_config.org
56,57c56,57
＜ PasswordAuthentication yes
＜ PermitEmptyPasswords no
---
＞ PasswordAuthentication no
＞ #PermitEmptyPasswords no
```
PasswordAuthentication ： パスワード認証を有効（yes）
PermitEmptyPasswords ： パスワードなしでのログイン禁止（no）･･･コメント解除

# SSHサービス開始
SSHサービスを開始します。
```
$ sudo service ssh start
[sudo] pi のパスワード:
 * Starting OpenBSD Secure Shell server sshd   [ OK ]
```
