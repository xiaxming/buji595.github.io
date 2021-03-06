---
title: Ansible  Module 
date: 2019-05-27 23:25:22
tags: Ansible
categories: Ansible
copyright: true
---
## Ansible Ad-hoc模式常用模块
ansible-doc 常用命令
```
# ansible-doc -h
Usage: ansible-doc [-l|-F|-s] [options] [-t <plugin type> ] [plugin]
-j  以json格式显示所有模块信息
-l  列出所有的模块
-s  查看模块常用参数
# 直接跟模块名，显示模块所有信息

[root@ansible ~]# ansible-doc -j
[root@ansible ~]# ansible-doc -l
[root@ansible ~]# ansible-doc -l |wc -l   #统计所有模块个数，ansible2.8共计2834个模块
2834
```
### 命令相关的模块
#### command
>`ansible`默认的模块,执行命令，注意：shell中的`"<"`, `">"`, `"|"`, `";"`, `"&"`,`"$"`等特殊字符不能在`command`模块中使用，如果需要使用，则用`shell`模块


```
# 查看模块参数
[root@ansible ~]# ansible-doc -s command

# 在192.168.1.31服务器上面执行ls命令，默认是在当前用户的家目录/root
[root@ansible ~]# ansible 192.168.1.31 -a 'ls'

# chdir  先切换工作目录，再执行后面的命令，一般情况下在编译时候使用
[root@ansible ~]# ansible 192.168.1.31 -a 'chdir=/tmp pwd'
192.168.1.31 | CHANGED | rc=0 >>
/tmp

# creates  如果creates的文件存在，则执行后面的操作
[root@ansible ~]# ansible 192.168.1.31 -a 'creates=/tmp ls /etc/passwd'    #tmp目录存在，则不执行后面的ls命令
192.168.1.31 | SUCCESS | rc=0 >>
skipped, since /tmp exists
[root@ansible ~]# ansible 192.168.1.31 -a 'creates=/tmp11 ls /etc/passwd'    # tmp11文件不存在，则执行后面的ls命令
192.168.1.31 | CHANGED | rc=0 >>
/etc/passwd

# removes  和creates相反，如果removes的文件存在，才执行后面的操作
[root@ansible ~]# ansible 192.168.1.31 -a 'removes=/tmp ls /etc/passwd'    #tmp文件存在，则执行了后面的ls命令
192.168.1.31 | CHANGED | rc=0 >>
/etc/passwd
[root@ansible ~]# ansible 192.168.1.31 -a 'removes=/tmp11 ls /etc/passwd'  #tmp11文件不存在，则没有执行后面的ls命令
192.168.1.31 | SUCCESS | rc=0 >>
skipped, since /tmp11 does not exist
```
#### shell
>专门用来执行`shell`命令的模块，和`command`模块一样，参数基本一样，都有`chdir,creates,removes`等参数

```
# 查看模块参数
[root@ansible ~]# ansible-doc -s shell

[root@ansible ~]# ansible 192.168.1.31 -m shell -a 'mkdir /tmp/test'
[root@ansible ~]# ansible 192.168.1.31 -m shell -a 'ls /tmp' 

#执行下面这条命令，每次执行都会更新文件的时间戳
[root@ansible ~]# ansible 192.168.1.31 -m shell -a 'cd /tmp/test && touch 1.txt && ls' 
192.168.1.31 | CHANGED | rc=0 >>
1.txt

# 由于有时候不想更新文件的创建时间戳，则如果存在就不执行creates
[root@ansible ~]# ansible 192.168.1.31 -m shell -a 'creates=/tmp/test/1.txt cd /tmp/test && touch 1.txt && ls'
192.168.1.31 | SUCCESS | rc=0 >>
skipped, since /tmp/test/1.txt exists
```
#### script
>用于在被管理机器上面执行`shell`脚本的模块，脚本无需在被管理机器上面存在

```
# 查看模块参数
[root@ansible ~]# ansible-doc -s script

# 编写shell脚本
[root@ansible ~]# vim ansible_test.sh 
#!/bin/bash
echo `hostname`

# 在所有被管理机器上执行该脚本
[root@ansible ~]# ansible all -m script -a '/root/ansible_test.sh'
192.168.1.32 | CHANGED => {
    "changed": true, 
    "rc": 0, 
    "stderr": "Shared connection to 192.168.1.32 closed.\r\n", 
    "stderr_lines": [
        "Shared connection to 192.168.1.32 closed."
    ], 
    "stdout": "linux.node02.com\r\n", 
    "stdout_lines": [
        "linux.node02.com"
    ]
}
......
```
### 文件相关的模块
#### file
>用于对文件的处理，创建，删除，权限控制等

```
# 查看模块参数
[root@ansible ~]# ansible-doc -s file
path     #要管理的文件路径
recurse  #递归
state：
     directory  #创建目录，如果目标不存在则创建目录及其子目录
     touch      #创建文件，如果文件存在，则修改文件 属性
     
     absent     #删除文件或目录
     mode       #设置文件或目录权限
     owner      #设置文件或目录属主信息
     group      #设置文件或目录属组信息
     link       #创建软连接，需要和src配合使用
     hard       #创建硬连接，需要和src配合使用

# 创建目录
[root@ansible ~]# ansible 192.168.1.31 -m file -a 'path=/tmp/test1 state=directory'

# 创建文件
[root@ansible ~]# ansible 192.168.1.31 -m file -a 'path=/tmp/test2 state=touch'

# 建立软链接（src表示源文件，path表示目标文件）
[root@ansible ~]# ansible 192.168.1.31 -m file -a 'src=/tmp/test1 path=/tmp/test3 state=link'

# 删除文件
[root@ansible ~]# ansible 192.168.1.31 -m file -a 'path=/tmp/test2 state=absent'

# 创建文件时同时设置权限等信息
[root@ansible ~]# ansible 192.168.1.31 -m file -a 'path=/tmp/test4 state=directory mode=775 owner=root group=root'
```
#### copy
> 用于管理端复制文件到远程主机，并可以设置权限，属组，属主等

```
# 查看模块参数
[root@ansible ~]# ansible-doc -s copy
src      #需要copy的文件的源路径
dest     #需要copy的文件的目标路径
backup   #对copy的文件进行备份
content  #直接在远程主机被管理文件中添加内容，会覆盖原文件内容
mode     #对copy到远端的文件设置权限
owner    #对copy到远端的文件设置属主
group    #对copy到远端文件设置属组


# 复制文件到远程主机并改名
[root@ansible ~]# ansible 192.168.1.31 -m copy -a 'dest=/tmp/a.sh src=/root/ansible_test.sh'

# 复制文件到远程主机，并备份远程文件,安装时间信息备份文件（当更新文件内容后，重新copy时用到）
[root@ansible ~]# ansible 192.168.1.31 -m copy -a 'dest=/tmp/a.sh src=/root/ansible_test.sh backup=yes'

# 直接在远程主机a.sh中添加内容
[root@ansible ~]# ansible 192.168.1.31 -m copy -a 'dest=/tmp/a.sh content="#!/bin/bash\n echo `uptime`"'

# 复制文件到远程主机，并设置权限及属主与属组
[root@ansible ~]# ansible 192.168.1.31 -m copy -a 'dest=/tmp/passwd src=/etc/passwd mode=700 owner=root group=root'
```
#### fetch
> 用于从被管理机器上面拉取文件，拉取下来的内容会保留目录结构，一般情况用在收集被管理机器的日志文件等

```
# 查看模块参数
[root@ansible ~]# ansible-doc -s fetch
src      #指定需要从远端机器拉取的文件路径
dest     #指定从远端机器拉取下来的文件存放路径

# 从被管理机器上拉取cron日志文件，默认会已管理节点地址创建一个目录，并存放在内
[root@ansible ~]# ansible 192.168.1.31 -m fetch -a 'dest=/tmp src=/var/log/cron'

[root@ansible ~]# tree /tmp/192.168.1.31/
/tmp/192.168.1.31/
└── var
    └── log
        └── cron

2 directories, 1 file
```
### 用户相关的模块
#### user
>用于对系统用户的管理，用户的创建、删除、家目录、属组等设置

```
# 查看模块参数
[root@ansible ~]# ansible-doc -s user
name        #指定用户的名字
home        #指定用户的家目录
uid         #指定用户的uid
group       #指定用户的用户组
groups      #指定用户的附加组
password    #指定用户的密码
shell       #指定用户的登录shell
create_home #是否创建用户家目录，默认是yes
remove      #删除用户时，指定是否删除家目录
state：
      absent    #删除用户
      

# 创建用户名指定家目录，指定uid及组
[root@ansible ~]# ansible 192.168.1.31 -m user -a 'name=mysql home=/opt/mysql uid=1002 group=root'
[root@ansible ~]# ansible 192.168.1.31 -m shell  -a 'id mysql && ls -l /opt'
192.168.1.31 | CHANGED | rc=0 >>
uid=1002(mysql) gid=0(root) 组=0(root)
总用量 0
drwx------  3 mysql root 78 5月  27 18:13 mysql

# 创建用户，不创建家目录，并且不能登录
[root@ansible ~]# ansible 192.168.1.31 -m user -a 'name=apache shell=/bin/nologin uid=1003 create_home=no'
[root@ansible ~]# ansible 192.168.1.31 -m shell  -a 'id apache && tail -1 /etc/passwd'
192.168.1.31 | CHANGED | rc=0 >>
uid=1003(apache) gid=1003(apache) 组=1003(apache)
apache:x:1003:1003::/home/apache:/bin/nologin

# 删除用户
[root@ansible ~]# ansible 192.168.1.31 -m user -a 'name=apache state=absent'

# 删除用户并删除家目录
[root@ansible ~]# ansible 192.168.1.31 -m user -a 'name=mysql state=absent remove=yes'
```
#### group
>用于创建组，当创建用户时如果需要指定组，组不存在的话就可以通过`group`先创建组

```
# 查看模块参数
[root@ansible ~]# ansible-doc -s group
name     #指定组的名字
gid      #指定组的gid
state：
     absent   #删除组
     present  #创建组（默认的状态）

# 创建组
[root@ansible ~]# ansible 192.168.1.31 -m group -a 'name=www'

# 创建组并指定gid
[root@ansible ~]# ansible 192.168.1.31 -m group -a 'name=www1 gid=1005'

# 删除组
[root@ansible ~]# ansible 192.168.1.31 -m group -a 'name=www1 state=absent'
```
### 软件包相关的模块
#### yum
>用于对软件包的管理，下载、安装、卸载、升级等操作

```
# 查看模块参数
[root@ansible ~]# ansible-doc -s yum
name            #指定要操作的软件包名字
download_dir    #指定下载软件包的存放路径，需要配合download_only一起使用
download_only   #只下载软件包，而不进行安装，和yum --downloadonly一样
list:
    installed   #列出所有已安装的软件包
    updates     #列出所有可以更新的软件包
    repos       #列出所有的yum仓库
state:   
    installed, present   #安装软件包(两者任选其一都可以)
    removed, absent      #卸载软件包
    latest      #安装最新软件包
    
# 列出所有已安装的软件包
[root@ansible ~]# ansible 192.168.1.31 -m yum -a 'list=installed'

# 列出所有可更新的软件包
[root@ansible ~]# ansible 192.168.1.31 -m yum -a 'list=updates'

#列出所有的yum仓库
[root@ansible ~]# ansible 192.168.1.31 -m yum -a 'list=repos'

#只下载软件包并到指定目录下
[root@ansible ~]# ansible 192.168.1.31 -m yum -a 'name=httpd download_only=yes download_dir=/tmp'

#安装软件包
[root@ansible ~]# ansible 192.168.1.31 -m yum -a 'name=httpd state=installed'

#卸载软件包
[root@ansible ~]# ansible 192.168.1.31 -m yum -a 'name=httpd state=removed'

#安装包组，类似yum groupinstall 'Development Tools'
[root@ansible ~]# ansible 192.168.1.31 -m yum -a 'name="@Development Tools" state=installed'
```
#### pip
>用于安装python中的包

```
# 查看模块参数
[root@ansible ~]# ansible-doc -s pip

# 使用pip时，需要保证被管理机器上有python-pip软件包
[root@ansible ~]# ansible 192.168.1.31 -m yum -a 'name=python-pip'

# 安装pip包
[root@ansible ~]# ansible 192.168.1.31 -m pip -a 'name=flask'
```
#### service
>服务模块，用于对服务进行管理，服务的启动、关闭、开机自启等

```
# 查看模块参数
[root@ansible ~]# ansible-doc -s service
name       #指定需要管理的服务名
enabled    #指定是否开机自启动
state:     #指定服务状态
    started    #启动服务
    stopped    #停止服务
    restarted  #重启服务
    reloaded   #重载服务

# 启动服务，并设置开机自启动 
[root@ansible ~]# ansible 192.168.1.31 -m service -a 'name=crond state=started enabled=yes'
```
### 计划任务相关的模块
#### cron
>用于指定计划任务，和`crontab -e`一样

```
# 查看模块参数
[root@ansible ~]# ansible-doc -s cron
job     #指定需要执行的任务
minute   #分钟
hour     #小时
day      #天
month    #月
weekday  #周
name     #对计划任务进行描述
state:
    absetn   #删除计划任务


# 创建一个计划任务，并描述是干嘛用的
[root@ansible ~]# ansible 192.168.1.31 -m cron -a "name='这是一个测试的计划任务' minute=* hour=* day=* month=* weekday=* job='/bin/bash /root/test.sh'"
[root@ansible ~]# ansible 192.168.1.31 -m shell -a 'crontab -l'
192.168.1.31 | CHANGED | rc=0 >>
#Ansible: 这是一个测试的计划任务
* * * * * /bin/bash /root/test.sh

# 创建一个没有带描述的计划任务
[root@ansible ~]# ansible 192.168.1.31 -m cron -a "job='/bin/sh /root/test.sh'"

# 删除计划任务
[root@ansible ~]# ansible 192.168.1.31 -m cron -a "name='None' job='/bin/sh /root/test.sh' state=absent"
```
### 系统信息相关的模块
#### setup
>用于获取系统信息的一个模块

```
# 查看模块参数
[root@ansible ~]# ansible-doc -s setup

# 查看系统所有信息
[root@ansible ~]# ansible 192.168.1.31 -m setup

# filter 对系统信息进行过滤
[root@ansible ~]# ansible 192.168.1.31 -m setup -a 'filter=ansible_all_ipv4_addresses'

# 常用的过滤选项
ansible_all_ipv4_addresses         所有的ipv4地址
ansible_all_ipv6_addresses         所有的ipv6地址
ansible_architecture               系统的架构
ansible_date_time                  系统时间
ansible_default_ipv4               系统的默认ipv4地址
ansible_distribution               系统名称
ansible_distribution_file_variety  系统的家族
ansible_distribution_major_version 系统的版本
ansible_domain                     系统所在的域
ansible_fqdn                       系统的主机名
ansible_hostname                   系统的主机名,简写
ansible_os_family                  系统的家族
ansible_processor_cores            cpu的核数
ansible_processor_count            cpu的颗数
ansible_processor_vcpus            cpu的个数
```