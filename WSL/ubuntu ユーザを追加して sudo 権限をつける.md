よく使うクセにすぐ忘れるのでメモ。

## ユーザ作成

```
# adduser user_name
```

※ `useradd` はホームディレクトリが作成されない

```
Adding user `user_name' ...
Adding new group `user_name' (1002) ...
Adding new user `user_name' (1002) with group `user_name' ...
Creating home directory `/home/user_name' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:  # パスワード入力
Retype new UNIX password: # パスワード入力（確認）
passwd: password updated successfully
Changing the user information for user_name
Enter the new value, or press ENTER for the default
        Full Name []:    # Enter連打
        Room Number []: 
        Work Phone []: 
        Home Phone []: 
        Other []: 
Is the information correct? [Y/n] Y  # Yes
# 終了
```


## sudo グループに追加

```
# gpasswd -a user_name sudo
Adding user user_name to group sudo
```
