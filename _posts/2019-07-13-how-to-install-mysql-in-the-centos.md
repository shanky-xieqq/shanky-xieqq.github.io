---
layout: post
title:  "如何在Linux[Centos]中安装并配置mysql"
categories: Linux
tags: Linux ssh
---

* content
{:toc}

最近一直想写点自己的东西，在学习Koa2+mysql的api开发流程。    

然后就免不了配置环境，但是在服务器上安装mysql遇到了一些小坑，记录下来。				          
		   					    
				




### 安装

* 首先一定要选择比较新的centos版本，这里用最新的centos7.6(之前用老版本6.0出现了问题)             

* 其次安装的教程最好是按照mysql[官网](https://dev.mysql.com/doc/mysql-repo-excerpt/5.6/en/linux-installation-yum-repo.html)的步骤走       

* 从官网链接[这里](https://dev.mysql.com/downloads/repo/yum/)找到你需要的版本（一般都是最新的）      

* 点击download之后，右键点击右下角的[No thanks, just start my download.]，复制链接地址，如下图

![](/img/img20190819.png)    

* `wget '拷贝的链接'`下载当前文件到centos服务器，这里选择的是mysql8.x版（这里有个坑后面会说到）    

* 之后按照官网的教程      

```shell

sudo yum localinstall platform-and-version-specific-package-name.rpm

```

* `localinstall platform-and-version-specific-package-name` 就是你当前包的名字

```shell

sudo yum install mysql-community-server

```

* 等待它安装完成就好啦      


### 启动      

```shell

sudo service mysqld start

```

#### 坑1   

* 这里因为我所购买的是盖中盖版服务器，1核0.5G的（没办法，穷），所以导致每次在start的时候都是      

```shell

Job for mysqld.service failed because the control process exited with error code. See "systemctl status mysqld.service" and "journalctl -xe" for details.

```       

* 再看`systemctl status mysqld.service`就会有个大大的`Active: failed`摆在那里

* google来百度去发现是内存的问题，因为mysql大概的占用都将近500了，所以我0.5的内存根本不够用，最经济的办法就是新增 SWAP 分区（当然，如果不差钱，最省心的办法还是增加内存）            

* 下面是最快速的创建 Swap 空间几个命令：      

```shell

sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

```

* 创建完成后就能完整的start了    

### 登录

* 首先需要在mysql的日志文件里找到密码      

```shell

grep "password" /var/log/mysqld.log

```

* 用以下命令进入数据库    

```

mysql -uroot -p

```

* 修改用户密码    

```

ALTER USER 'root'@'localhost' IDENTIFIED BY 'new password';

```

* 这里会有密码强度的问题，一般可以把安全系数降低就可以设置123456之类的了，但是我这是服务器所以还是大小写+字符串搞定了


### 开启远程访问

#### 坑2

 * 在这里遇到了个小坑，在开启远程访问的时候查询了许多资料，大多数解释开启远程Mysql远程服务的命令如下：    

 ```shell

grant all privileges on *.* to 'root'@'%' identified by '你的密码' with grant option

 ```

 * 但是这个就会报'你的密码'那里语法错误，结果有资料説这种方法并不适用于Mysql 8.0以后的版本（前面说到的坑）           

 * 8.0之后的版本需要用以下三条命令开启远程访问      

 ```shell

 CREATE USER 'root'@'%' IDENTIFIED BY '你的密码'; 
 GRANT ALL ON *.* TO 'root'@'%'; 
 ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '你的密码';

 ```

 * 三条命令按顺序执行完成后刷新权限      

 ```shell

FLUSH PRIVILEGES;

 ```

 * 之后在就可以远程访问服务器的数据库了      

 * 最后一点，重中之重，一定要在服务器的管理界面开启3306的mysql端口安全规则，切记切记（鬼知道我链接了多少次才想起来）      

 * 附上我DateGrip的截图     

 ![](/img/img20190819_2.png)

   













