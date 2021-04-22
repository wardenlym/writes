---
title: "使用PGP公钥"
date: 2021-03-18T18:13:52+08:00
draft: true
---


```shell
# ======================
# 基本操作
# ======================

# 生成密钥对
$ gpg --gen-key

# 生成 revoke 证书
$ gpg -a -o <revoke.asc> --gen-revoke <UID>       # UID 或 Email 均可，下同

# 查看本地密钥信息
$ gpg --list-keys

# 查看指纹
$ gpg --fingerprint

# 导出公钥
$ gpg -a -o <xxx.pub> --export <UID>

# 导出私钥【谨慎操作】
$ gpg -a -o <xxx.sec> --export-secret-key <UID>

# 加密文件
$ gpg -a -o <encrypted.asc> -r <UID> -e <plain.txt>

# 加密文件并签名
$ gpg -a -o <encrypted.asc> -r <UID> -s -e <plain.txt>

# 签名文件(签名独立一个文件)
$ gpg -a -o <message.sig> -b <message.txt>

# 签名文件(签名内嵌在文件中)
$ gpg -a -o <message.sig> --clearsign <message.txt>

# 验证签名
$ gpg --verify <message.sig>

# 解密文件
$ gpg -o <plain.txt> -d <encrypted.asc>

# 发布公钥到服务器【谨慎操作】
$ gpg --keyserver <keys.gnupg.net> --send-key <UID>

# 从服务器搜索公钥
$ gpg --keyserver <keys.gnupg.net> --search-key <UID>

# 从服务器导入公钥
$ gpg --keyserver <keys.gnupg.net> --recv-key <UID>

# 从文件导入公钥（revoke 证书、密钥等也可导入）
$ gpg --import <xxx.pub>

# 签名并验收公钥
$ gpg --sign-key <UID>

# 删除公钥
$ gpg --delete-keys <UID>

# 删除私钥【谨慎操作】
$ gpg --delete-secret-keys <UID>

# ======================
# 修改模式：
# ======================

# 进入修改模式
$ gpg --edit-key <UID>

# 选定要修改的密钥
gpg> key <id>

# 修改私钥密码
gpg> passwd

# 修改过期时间
gpg> expire

# revoke【谨慎操作】
gpg> revuid

# 保存修改
gpg> save

# 直接退出
gpg> quit
```
