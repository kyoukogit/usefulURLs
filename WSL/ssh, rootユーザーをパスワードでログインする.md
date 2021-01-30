# rootユーザーのパスワード設定
バーチャル環境のイメージファイルなどで、CentOSの初期設定ではrootパスワードが未設定のときもあります。

（一般ユーザーの sudo su で代用している。）

なので、まずrootユーザーにパスワードを設定します。sudo suでroot権限にしておいてください。

```
passwd
```
# sshdの設定変更
今度は、rootユーザーのパスワード認証でリモートログインを許可しましょう。

リモートまわりはsshdサービスが管理しているので、その設定ファイルを変更します。

```
PermitRootLogin yes
PasswordAuthentication yes
```