---
layout: post
title:  "Running CoreOS Container Linux on Windows VirtualBox"
date:   2017-03-21 09:42:59 +0800
categories: coreos
---
CoreOS Container Linux是一个轻量级的Linux操作系统，专为集群部署而设计，为您最关键的应用程序提供了自动化，安全和可扩展的能力，它可以运行在大多数云服务提供商，虚拟化平台和裸机服务器上，当然也可以在你的笔记本电脑上通过运行本地VM来构建一个不错的开发环境。CoreOS官网提供了很详细的文档来指导如何构建一个本地的开发环境，但是这些开发环境全部是构建于Linux平台之上的，考虑到那些更习惯于Windows平台的开发者（比如笔者自己），本文简单介绍了如何在Windows平台上通过VirtualBox来创建CoreOS Container Linux虚拟机。

环境
---------------------
笔者的工作环境如下：
* WIndows 10 Enterprise
* Oracle VM VirtualBox 5.1.10
* Ubuntu 16.04， 虚拟机映像
* PuTTY 0.68，SSH 和 telnet 客户端
* WinSCP 5.9.4，用于在Linux虚拟机和Windows之间复制文件
* 科学上网

因为CoreOS只提供了Linux的脚本来创建虚拟机磁盘映像及配置驱动（Config Drive）磁盘映像，因此需要搭建Linux环境生成上述磁盘映像，然后把它们应用于Windows VirtualBox。

在Ubuntu 16.04中安装virtualbox和mkisofs
---------------------------------------
在创建虚拟机磁盘映像及配置驱动磁盘映像的过程中，需要上述两个工具，请运行如下命令来安装它们:
~~~
$ sudo apt-get install virtualbox mkisofs
~~~

构建虚拟机磁盘映像
---------------------
CoreOS提供了一个脚本来简化VDI的构建过程，该脚本保存在[GitHub](https://github.com/coreos/scripts/blob/master/contrib/create-coreos-vdi)上。 它会下载一个裸机映像，然后使用GPG验证它，并将映像转换为VirtualBox格式。

1. 下载VDI构建脚本文件并将其标记为可执行。
~~~
$ wget https://raw.githubusercontent.com/coreos/scripts/master/contrib/create-coreos-vdi
$ chmod +x create-coreos-vdi
~~~
2. 运行脚本，可以通过以下参数指定目标文件位置和CoreOS Container Linux版本。
* -d DEST     指定创建CoreOS VDI映像的路径，如未指定，则创建于当前目录
* -V VERSION  指定要安装的版本，比如是alpha，Beta，或者Stable，如未制定，则缺省为Stable

详情请参阅脚本本身,下述命令将会在当前目录下创建Stable版本的映像。
~~~
$ ./create-coreos-vdi
~~~
脚本执行成功后，可以在当前目录下找到CoreOS Container Linux映像。
~~~
$ ls
coreos_production_1298.6.0.vdi  create-coreos-vdi
~~~

创建SSH Keys
-----------
CoreOS通常通过SSH密钥来认证用户，因此本实验需要创建一对SSH Keys。

首先输入下面的命令检查SSH Keys是否存在。
~~~
$ ls ~/.ssh
~~~

如果有文件id_rsa和id_rsa.pub存在，则说明SSH Keys已经存在，否则需要输入下面的命令来创建它们，请酌情修改命令中的邮箱地址。
~~~
$ ssh-keygen -t rsa -C "edward.yh.lu@gmail.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/edward/.ssh/id_rsa):
Created directory '/home/edward/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/edward/.ssh/id_rsa.
Your public key has been saved in /home/edward/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:dzg3Qma94H0o73C54vzhQ/6m/nKPxybNyOdnd7mVFEw edward.yh.lu@gmail.com
The key's randomart image is:
+---[RSA 2048]----+
|               E |
|           .  o  |
|          = .  o |
|         = + o  .|
|        S B B .. |
|         . B.+. .|
|          .o=. *o|
|         ..=+o*o&|
|         .o+BO+XB|
+----[SHA256]-----+

~~~

创建配置驱动（Config Drive）
---------------------------
配置驱动（Config Drive）最初是OpenStack的一个功能，这也是为什么会在配置驱动映像中看到openstack字符的原因。在OpenStack中，一些元数据可以被写入到一个特殊的配置驱动，当虚拟机启动时，这个配置驱动通过CD-ROM或其他接口被附加到虚拟机中，然后虚拟机可以挂载此驱动并从中读取文件，据此可以获取通常通过元数据服务获得的信息。

使用配置驱动的一个典型用例是当管理员不希望使用DHCP为虚拟机实例分配IP地址时，可以通过配置驱动传递网络配置，例如，IP地址。任何能够安装ISO 9660或VFAT文件系统的现代客户操作系统都可以使用配置驱动。

本实验需要通过配置驱动来配置一个SSH密钥用以访问虚拟机，CoreOS提供了如下脚本文件来创建一个基本的配置驱动。
~~~
$ wget https://raw.github.com/coreos/scripts/master/contrib/create-basic-configdrive
$ chmod +x create-basic-configdrive
$ ./create-basic-configdrive -H CoreOS -S ~/.ssh/id_rsa.pub
I: -input-charset not specified, using utf-8 (detected in locale settings)
Total translation table size: 0
Total rockridge attributes bytes: 677
Total directory bytes: 4096
Path table size(bytes): 40
Max brk space used 1f000
178 extents written (0 MB)


Success! The config-drive image was created on /home/edward/CoreOS.iso
~~~

执行成功后，该脚本将创建一个名为CoreOS.iso的ISO文件，该文件将配置虚拟机以接受SSH密钥并将其名称设置为CoreOS。

克隆虚拟机的映像
----------------
`构建虚拟机磁盘映像`一步生成了虚拟机映像文件coreos_production_1298.6.0.vdi，一个通常的做法是把它作为一个基础映像，然后每次使用时应该克隆一个新虚拟机的映像，并可酌情通过指令修改磁盘的大小。

~~~
$ VBoxManage clonehd coreos_production_1298.6.0.vdi CoreOS.vdi
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
Clone medium created in format 'VDI'. UUID: adca78f8-6c76-45b0-a42d-9d6a94b18584
$ VBoxManage modifyhd CoreOS.vdi --resize 32768
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
~~~


在Windows VirtualBox上部署虚拟机
--------------------------------
用WinSCP从Ubuntu复制下述文件到Windows：
* CoreOS.iso
* CoreOS.vdi
* id_rsa

### 生成PuTTY Private Key Files
在PuTTY安装目录下，找到并运行puttygen。
1. 在PuTTY Key Generator对话框中，选择菜单Conversions/Import Key, 在Load private key对话框中选择上一步复制的id_rsa文件，然后选择Open按钮。
2. 在PuTTY Key Generator对话框中，选择Save private key按钮, 选择Yes按钮忽略警告信息，然后在Save private key as对话框中输入要保存的文件名，保存类型为 PuTTY Private Key Files(*.ppk)，选择Save按钮。

### 创建并运行新虚拟机
1. 运行VirtualBox Manager

2. 选择菜单Machine/New，在Create Virtual Machine对话框中输入以下内容：
* Name：虚拟机名称
* Type：Linux
* Version：Linux 2.6 / 3.x / 4.x（64位）
* Memory Size：酌情设置内存大小，推荐至少1GB
* Use an existing virtual hard disk：找到并选择新复制的映像CoreOS.vdi
* 选择Create按钮创建虚拟机

3. 选择新创建的虚拟机，选择菜单Settings，在Settings对话框中修改以下内容：
* 选择Storage，将上一步复制的配置驱动CoreOS.iso挂载到CD/DVD驱动器
* 选择Networek，选择网络类型为Bridged adapter
* 选择OK按钮，虚拟机将准备启动

4. 选择新创建的虚拟机，选择菜单Start

用PuTTY登录CoreOS虚拟机
----------------------
在VirtualBox虚拟机里，网络可能需要一点时间才能运行起来，稍等片刻，一个IP地址就会出现在控制台上，PuTTY可以通过这个IP地址连接到新的虚拟机。
![IP Address on VM Console]({{ site.url }}/assets/ip-address-on-vm-console.png)

运行PuTTY，在PuTTY Configuration对话框中：
1. 在Category中选择Session，在Host Name (or ip address)中输入上一步控制台上出现的IP地址。
2. 在Category中选择Connectionb/SSH/Auth，在Private key file for authetication中，输入`生成PuTTY Private Key Files`步骤中生成的PuTTY Private Key File。
3. 选择Open按钮，在登录提示下输入core，现在可以登录新建的虚拟机了。

~~~
login as: core
Authenticating with public key "imported-openssh-key"
Last login: Tue Mar 21 01:50:16 UTC 2017 from 192.168.1.148 on pts/0
Container Linux by CoreOS stable (1298.6.0)
core@CoreOS ~ $
~~~
注意，core是CoreOS内建的用户名，必须全部小写。


参考资料
--------
[Running CoreOS Container Linux on VirtualBox](https://coreos.com/os/docs/latest/booting-on-virtualbox.html)