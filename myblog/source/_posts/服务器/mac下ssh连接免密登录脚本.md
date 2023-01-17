---
cover: /img/linux.jpeg
---

# mac 使用 ssh 秘钥远程登录服务器

mac 下创建个 xx.sh 文件 放入脚本代码：

```bash
#!/bin/bash
<< /*
  使用公钥私钥的进行免密远程ssh操作
/*

username="root"
read -p "请输入你要创建远程登录的服务器ip：" remote_id

function createRsa(){
  ssh-keygen
}

function transmitRsaToRemote() {
  ssh-copy-id ${username}@${remote_id}
}

[[ -f ~/.ssh/id_rsa && -f ~/.ssh/id_rsa.pub ]] && flag=yes || flag=no
if [[ $flag == "yes" ]]; then
  transmitRsaToRemote;
else
  createRsa;
  transmitRsaToRemote;
fi

```

脚本会将在本地生成公钥私钥，并把公钥上传到远程服务器，期间需要输入一次密码。

```bash
# 赋予权限
 chmod 777 ./xx.sh
 #然后运行
 ./xx.sh
```

如果是第一次生产秘钥，会提示输入私钥密码 passphrase 回车即可不设置

如果不小心 设置了密码，导致每次 ssh 远程登录都需要输入私钥密码 很 麻烦

去网上查了查，发现 ssh-keygen 本身就实现了这个功能。使用方法如下：

1、在终端下输入 ssh-keygen -p

ssh-keygen -p

Enter file in which the key is (/Users/username/.ssh/id_rsa):

2、系统会提示选择需要修改的私钥，可以直接回车，默认是/Users/username/.ssh/id_rsa

3、选好文件后按回车，会提示你输入旧密码：

Enter old passphrase:

4.输入好后会提示输入新密码：

Enter new passphrase (empty for no passphrase):

5.如果直接回车，会提示确认新密码，再直接回车，此时以前设置的私钥密码就被清除了：
