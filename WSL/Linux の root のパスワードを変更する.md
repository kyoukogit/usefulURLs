ユーザのパスワードの変更には「passwd」コマンドを利用します。 特にオプションを指定しない場合は passwd ユーザ名 となります。 ここでは root のパスワードを変更するので、root 権限が必要です。 root 権限を持っていない場合は「su」コマンドを使って root 権限を取得して下さい。
```
# passwd root
Changing password for user root.
New password: (入力する、見えない)
Retype new password: (再入力する、見えない)
passwd: all ahtentication tokens updated successfully.
```