---
layout: post
title:  "Running CoreOS Container Linux on Windows VirtualBox"
date:   2017-03-21 09:42:59 +0800
categories: coreos
---
CoreOS Container Linux是一个轻量级的Linux操作系统，专为集群部署而设计，为您最关键的应用程序提供了自动化，安全和可扩展的能力，它可以运行在大多数云服务提供商，虚拟化平台和裸机服务器上，当然也可以在你的笔记本电脑上通过运行本地VM来构建一个不错的开发环境。CoreOS 官网提供了很详细的文档来指导如何构建一个本地的开发环境，但是这些开发环境全部是构建于Linux平台之上的，考虑到那些更习惯于Windows平台的开发者（比如笔者自己），本文简单介绍了如何在Windows平台上通过VirtualBox来创建CoreOS Container Linux虚拟机。

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

构建虚拟机磁盘文件
---------------------
CoreOS提供了一个脚本来简化VDI的构建过程，该脚本保存在[GitHub](https://github.com/coreos/scripts/blob/master/contrib/create-coreos-vdi)上。 它会下载一个裸机映像，然后使用GPG验证它，并将映像转换为VirtualBox格式。

第一步，下载上述脚本文件并把它标记为可执行执行。
~~~
$ wget https://raw.githubusercontent.com/coreos/scripts/master/contrib/create-coreos-vdi
$ chmod +x create-coreos-vdi
~~~
第二步，运行脚本时可以指定目标位置和CoreOS Container Linux版本。
* -d DEST     指定创建CoreOS VDI映像的路径，如未指定，则创建于当前目录
* -V VERSION  指定要安装的版本，比如是alpha，Beta，或者Stable，如未制定，则缺省为Stable

详情请参阅脚本本身,下述命令将会在当前目录下创建Stable版本的映像
~~~
$ ./create-coreos-vdi
~~~
脚本执行成功后，可以在当前目录下找到CoreOS Container Linux映像。
~~~
$ ls
coreos_production_1298.6.0.vdi  create-coreos-vdi
~~~

生成SSH Keys
-----------
如果您已经了解CoreOS，那么您会知道为了连接到一个CoreOS服务器，它需要配置你的SSH密钥。

首先输入下面的命令检查SSH Keys是否存在。
~~~
$ ls ~/.ssh
~~~

如果有文件id_rsa和id_rsa.pub，则说明SSH Keys已经存在，否则需要输入下面的命令来生成它们，请酌情修改命令中的邮箱地址。
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
配置驱动（Config Drive）标准最初是一个OpenStack的功能，这也是为什么你会在配置驱动映像中看到openstack字符串的原因。在OpenStack中，一些元数据可以被写入到一个特殊的配置驱动，当虚拟机启动时，这个配置驱动通过CD-ROM或其他接口被附加到虚拟机中，然后虚拟机可以挂载此驱动并从中读取文件，据此可以获取通常通过元数据服务获得的信息。

使用配置驱动的一个典型用例是当您不希望使用DHCP为虚拟机实例分配IP地址时，可以通过配置驱动传递网络配置，例如，IP地址。任何能够安装ISO 9660或VFAT文件系统的现代客户操作系统都可以使用配置驱动。

在本实验中，您至少需要配置驱动来配置一个SSH密钥来访问虚拟机。CoreOS提供了如下脚本文件来创建一个基本的配置驱动。
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

脚本执行成功后，将创建一个名为CoreOS.iso的ISO文件，该文件将配置虚拟机以接受您的SSH密钥并将其名称设置为CoreOS。

克隆虚拟机的映像
----------------
在构建虚拟机磁盘文件一步中，我们生成了虚拟机映像文件coreos_production_1298.6.0.vdi，我建议把它作为一个基础映像，每次使用时您应该克隆每个新虚拟机的映像，并将其设置为所需的大小。 

~~~
$ VBoxManage clonehd coreos_production_1298.6.0.vdi CoreOS.vdi
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
Clone medium created in format 'VDI'. UUID: adca78f8-6c76-45b0-a42d-9d6a94b18584
$ VBoxManage modifyhd CoreOS.vdi --resize 32768
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
~~~


在Windows VirtualBox上部署虚拟机
--------------------------------
注意，以下步骤都运行在Windows环境中。

用WinSCP从Ubuntu 16.04复制下述文件到Windows：
* CoreOS.iso
* CoreOS.vdi
* id_rsa
在PuTTY安装目录下，运行puttygen，转到菜单Conversions/Import Key, 选择上一步复制的id_rsa文件，然后选择Save private key, 输入文件名CoreOS，保存类型为 PuTTY Private Key Files(*.ppk) 

打开VirtualBox Manager并转到菜单“机器”>“新建”。 键入所需的机器名称，然后选择Linux类型和Linux 2.6 / 3.x / 4.x（64位）版本。

接下来，选择所需的内存大小， 推荐1GB。

接下来，选择“使用现有的虚拟硬盘驱动器文件”并找到新复制的克隆映像。

单击“创建”按钮创建虚拟机。

接下来，从创建的虚拟机转到设置。 然后单击存储选项卡，将创建的配置驱动加载到CD / DVD驱动器。
网络类型选择 Bridge

单击“确定”按钮，虚拟机将准备启动。

用PuTTY登录CoreOS虚拟机
----------------------
在VirtualBox虚拟机里，网络可能需要一点时间才能工作起来，稍等片刻您就会在控制台上看到一个IP地址，通过这个IP地址就可以连接到它。 

在PuTTY中输入上述IP地址，并在Connectionb/SSH/Auth中把上面生成的PuTTY Private Key File配置在Private key file for authetication中。

选择Open, 在登录提示下输入core，现在您可以登录新建的虚拟机了。
~~~
login as: core
Authenticating with public key "imported-openssh-key"
Last login: Tue Mar 21 01:50:16 UTC 2017 from 192.168.1.148 on pts/0
Container Linux by CoreOS stable (1298.6.0)
core@CoreOS ~ $
~~~
注意，core是CoreOS内建的用户名，必须全部小写。 