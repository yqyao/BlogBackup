---
title: rsync
date: 2017-02-07 13:23:50
tags: [sersync同步配置]
categories: Linux配置
---
### 为何要用这个？
* sersync是基于Inotify开发的，类似于Inotify-tools的工具
* sersync可以记录下被监听目录中发生变化的（包括增加、删除、修改）具体某一个文件或某一个目录的名字，然后使用rsync同步的时候，只同步发生变化的这个文件或者这个目录。
* rsync在同步的时候，只同步发生变化的这个文件或者这个目录（每次发生变化的数据相对整个同步目录数据来说是很小的，rsync在遍历查找比对文件时，速度很快），因此，效率很高。

### 测试机器
* 机器 ip
10.0.2.123（内网地址）
10.0.2.124
* 操作系统
   centos7
* 两台机器既为客户端也为服务端
<!-- more -->

### 配置rsync
1. 关闭SELINUX
```bash
    vi /etc/selinux/config #编辑防火墙配置文件
    SELINUX=enforcing #注释掉
    SELINUXTYPE=targeted #注释掉
    SELINUX=disabled #增加
    :wq! #保存，退出
    setenforce 0 #立即生效
```
2. 创建用户认证文件
    * 设置本机用户名与密码
        vi /etc/rsync.pas yyq:88888888
    * 设置远程服务器的密码
        vi /etc/rsync_server.pas 88888888
    * 修改文件属性
        chmod 600 /etc/rsync.pas
        chmod 600 /etc/rsync_server.pas
3. 配置rsync
一般的centos 系统会 自带rsync，但是没有配置文件，我们修改配置文件。
vi /etc/rsyncd.conf #创建配置文件，添加以下代码
log file = /var/log/rsyncd.log #日志文件位置，启动rsync后自动产生这个文件，无需提前创建
pidfile = /var/run/rsyncd.pid  #pid文件的存放位置
lock file = /var/run/rsync.lock  #支持max connections参数的锁文件
secrets file = /etc/rsync.pass  #用户认证配置文件，里面保存用户名称和密码，后面会创建这个文件
uid = root #设置rsync运行权限为root
gid = root #设置rsync运行权限为root
use chroot = no #默认为true，修改为no，增加对目录文件软连接的备份
read only = no  #设置rsync服务端文件为读写权限
max connections = 200 #最大连接数
timeout = 600  #设置超时时间
[search] #自定义名称
path = /home/yyq/ #rsync服务端数据目录路径
auth users = yyq
hosts allow = 10.0.2.123 #允许进行数据同步的客户端IP地址，可以设置多个，用英文状态下逗号隔开
hosts deny = 10.0.2.125 # 禁止数据同步的客户端IP地址， 也可以设置多个
4. 启动rsync
     rsync --daemon --config=/etc/rsyncd.conf

### 安装sersync
* 目的： 实时触发同步
* 下载sersync
    sersync 不需要编译，可直接使用其可执行文件。
    sersync下载地址：https://sersync.googlecode.com/files/sersync2.5.4_64bit_binary_stable_final.tar.gz
* 配置 confxml.xml
``` xml
<sersync>
        <localpath watch="/opt/tongbu">
            <remote ip="10.0.2.123" name="search"/>
            <!--<remote ip="192.168.8.39" name="tongbu"/>-->
            <!--<remote ip="192.168.8.40" name="tongbu"/>-->
        </localpath>
        <rsync>
            <commonParams params="-artuz"/>
            <auth start="true" users="yyq" passwordfile="/etc/rsync_server.pas"/>
            <userDefinedPort start="false" port="874"/><!-- port=874 -->
            <timeout start="false" time="100"/><!-- timeout=100 -->
            <ssh start="false"/>
        </rsync>
        <failLog path="/tmp/rsync_fail_log.sh" timeToExecute="60"/><!--default every 60mins execute once-->
        <crontab start="false" schedule="600"><!--600mins-->
            <crontabfilter start="false">
                <exclude expression="*.php"></exclude>
                <exclude expression="info/*"></exclude>
            </crontabfilter>
        </crontab>
        <plugin start="false" name="command"/>
    </sersync>
    
 ```
* 启动sersync
     sersync2 -d -r -o /home/yyq/sersync/confxml.xml
* 检查同步是否完成
    查看同步日志tail -f 100 /var/log/rsync.log
* 设置sersync监控开机自动执行
    vi /etc/rc.d/rc.local  #编辑，在最后添加一行
    sersync2 -d -r -o /home/yyq/sersync/confxml.xml ＃设置开机自动运行脚本

### 注意事项
* 用户认证文件是属于root，因此在运行sersync注意也需要用root去运行，否则会出现同步错误
* 用户认证文件的文件属性是600
* 本台机器是在10.0.2.124机器上配置的，在10.0.2.123上配置同这个一样，只需要把ip换成10.0.2.124即可

### rsync 用法
#### 常用格式
1.  rsync [OPTION]... SRC [SRC]... [USER@]HOST:DEST 
    将本地内容拷贝到远程机器上
2.  rsync [OPTION]... [USER@]HOST::SRC [DEST]
    将远程机器的内容拷贝到本地
3.  rsync [OPTION]... SRC [SRC]... DEST
    本地拷贝
4.  rsync [OPTION]... [USER@]HOST::SRC [DEST] 
    从远程rsync服务器中拷贝文件到本地机
5.  rsync [OPTION]... SRC [SRC]... [USER@]HOST::DEST
    从本地机器拷贝文件到远程rsync服务器中
6. rsync [OPTION]... rsync://[USER@]HOST[:PORT]/SRC [DEST]
    列远程机的文件列表

#### 常用参数
* -r 表示递归 ；-l 是链接文件，意思是拷贝链接文件；-p 表示保持文件原有权限；
* -t 保持文件原有时间；-g 保持文件原有用户组；-o 保持文件原有属主；
* -z 传输时压缩；-P 传输进度；-e ssh的参数建立起加密的连接
* -u只进行更新，防止本地新文件被重写，注意两者机器的时钟的同时
* -v 传输时的进度等信息；
* --progress是指显示出详细的进度情况
* --delete是指如果服务器端删除了这一文件，那么客户端也相应把文件删除，保持真正的一致
* --password-file=/password/path/file来指定密码文件
* --exclude=PATTERN 指定排除不需要传输的文件模式
* --include=PATTERN 指定不排除而需要传输的文件模式
* --exclude-from=FILE 排除FILE中指定模式的文件
* --include-from=FILE 不排除FILE指定模式匹配的文件 
* --config=FILE 指定其他的配置文件，不使用默认的rsyncd.conf文件
* --ignore-errors 即使出现IO错误也进行删除 
* --existing 仅仅更新那些已经存在于DST的文件，而不备份那些新创建的文件
* --delete-excluded 同样删除接收端那些被该选项指定排除的文件
* --delete-after 传输结束以后再删除

    
 


    







