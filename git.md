# Git

## 配置git

进入项目文件夹，开启git bash

```shell
$ git init
```

在本地创建ssh key

```shell
$ ssh-keygen -t rsa -C "xxx@xxx.com"
```

在gtihub网站上添加新的ssh，ssh来源为本地的id_rsa.pub

验证是否成功

```sh
$ ssh -T git@github.com
```

![img](https://images2018.cnblogs.com/blog/1357602/201803/1357602-20180323211106192-1936885626.png)

设置用户名和email

github每次commit都会记录

```sh
$ git config --global user.name "your name"
$ git config --global user.email "your_email@youremail.com"
```

上传项目

```sh
$ git remote add origin git@github.com:yourName/yourRepo.git
$ git add . 
$ git commit -m “修改项目代码”
$ git push origin master
```

查看状态

```sh
$ git status
```
