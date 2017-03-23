---
layout: post
title:  "Setup a Private Domain Docker Registry on Ubuntu 16.04"
date:   2017-03-22 09:23:59 +0800
categories: docker
---
Docker Registry是一个无状态，高度可扩展的服务器端应用程序，可以存储和分发Docker映像。Docker官方提供了一个名为[Docker Hub](https://hub.docker.com/)的公共服务来存储Docker映像，开发者可以免费地将自己创作的Docker映像上传到Docker Hub，但是任何上传的内容都是公开的，在某些情况下，对一个项目来说，这可能不是一个最佳选择。

私有的Docker Registry可以：
* 严格控制的映像存储位置
* 完全拥有映像分配渠道
* 将映像存储和分发紧密集成到内部开发工作流程中

本文简单介绍了如何设置私有的Docker Registry，并展示了如何将自定义的Docker映像推送到新设置的私有Docker Registry，以及从不同的主机提取映像。注意，本文并未涉及负载均衡及安全访问限制。

环境
---------------------
笔者的工作环境如下：
* Ubuntu 16.04 x 2，一台用来配置私有Docker Registry，另外一台用来当作Docker客户端
* Docker Engine 17.03.0-ce
* Docker Compose 1.11.2
* 科学上网

Docker Compose不是必需的，但是最好使用Docker Compose，这样，我们可以轻松地在一个容器中运行Docker Registry，并可以运行其他Docker容器，比如Nginx，让Nginx处理与外界的安全和沟通。

安装单机版Docker Registry
-------------------------------
### 安装
通过如下命令可以很容易的安装并运行单机版Docker Registry。
~~~
$ docker run -d -p 5000:5000 --restart=always --name registry registry:2.6
~~~

或者使用Docker Compose命令
1. 编辑docker-compose.yml文件，内容如下：
~~~
version: '3'
services:
    registry:
        image: registry:2.6
        ports:
            - 5000:5000
        volumes:
            - /data:/var/lib/registry
        restart: always
~~~
2. 运行如下命令来安装并运行Docker Registry。
~~~
$ docker-compose up
~~~

### 验证

运行如下命令来验证Docker Registry是否安装成功。
~~~
$ docker ps -a
CONTAINER ID        IMAGE                                  COMMAND                  CREATED             STATUS                   PORTS                    NAMES
7aa1ecb388fc        registry:2.6                           "/entrypoint.sh /etc/"   22 minutes ago      Up 22 minutes            0.0.0.0:5000->5000/tcp   registry
~~~

另外，还可以使用curl直接向Docker Registry发出HTTP请求，如果一切正常，那么Docker Registry会返回一个空的json对象`{}`。
~~~
$ curl http://localhost:5000/v2/
{}
~~~

安装网络版Docker Registry
-------------------------
虽然在localhost上运行Docker Registry也有它的用途，但是多数人都希望他们的Docker Registry可以在更广泛网络环境中使用。为此，Docker引擎需要使用TLS进行安全保护，这在概念上非常类似于使用SSL配置Web服务器。

### 获得一个证书
如果已经拥有某个域名，并且其DNS记录已经指向运行Docker Registry的主机，则首先需要从CA获取证书。

如果没有可用的域名，那么也可以使用自签名证书，或者设置自己的证书签发机构，本文假定了一个域名registry.gr.org，并且在hosts文件中把这个域名指向将要运行Docker Registry的主机。

### 已获得一个证书

1. 首先将crt文件和密钥文件更名为domain.crt和domain.key，然后把它们移动到/certs目录。

2. 编辑docker-compose.yml文件，内容如下：
~~~
version: '3'
services:
    registry:
        image: registry:2.6
        ports:
            - 5000:5000
        environment:
            REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
            REGISTRY_HTTP_TLS_KEY: /certs/domain.key
        volumes:
            - /data:/var/lib/registry
            - /certs:/certs
        restart: always
~~~

3. 运行如下命令来安装并运行Docker Registry。
~~~
$ docker-compose up
~~~

### 使用自签名SSL证书
1. 生成自己的证书，必须确保Common Name是registry.gr.org，其他的参数可以随意输入。
~~~
$ mkdir -p /certs && openssl req \
   -newkey rsa:4096 -nodes -sha256 -keyout /certs/domain.key \
   -x509 -days 365 -out /certs/domain.crt
Generating a 4096 bit RSA private key
.........++
............................................................................++
writing new private key to '/certs/domain.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:registry.gr.org
Email Address []:edward@test.com
~~~

2. 按照上文`已获得一个证书`的步骤安装和运行Docker Registry
3. 为确保每一个Docker客户端信任该自签名SSL证书，复制domain.crt文件到Docker客户端/etc/docker/certs.d/registry.gr.org:5000/ca.crt，然后重启客户端的Docker守护进程。

### 设置自己的证书签发机构

为了追求更好的灵活性，本文接下来将要介绍如何设置自己的证书签发机构。
~~~
数字证书采用信任链验证。数字证书的信任锚（信任的起点）就是根证书颁发机构。
证书颁发机构可以在一个树结构中签发多个证书，根证书就是这个树结构中最顶层的证书，其私钥用于“签名”其他证书。在根证书之后的所有证书都会继承根证书的可信赖性——根证书的签名有点类似“公正”一个现实世界中的身份。
根证书通常采用比普通证书更严格的机制以确保它更可信，例如物理层面上的安全分发。一些广泛使用的根证书是由网页浏览器的制造商分发。
~~~
上文摘录自维基百科，由此可见，如果设置自己的证书签发机构，那么需要创建一个根证书，并把它添加到本地操作系统的受信任根证书列表里，然后就可以用这个根证书签发其他证书了。

1. 为自己的证书签发机构生成私钥，文件名为PersonalCA.key
~~~
$ opensslgenrsa -out PersonalCA.key 2048
Generating RSA private key, 2048 bit long modulus
...................................+++
............+++
e is 65537 (0x10001)
~~~
2. 为自己的证书签发机构生成根证书，参数可随意输入
~~~
$ opensslreq -x509 -new -nodes -key PersonalCA.key -days 10000 -out PersonalCA.crt
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Beijing
Locality Name (eg, city) []:Beijing
Organization Name (eg, company) [Internet Widgits Pty Ltd]:GR
Organizational Unit Name (eg, section) []:CA
Common Name (e.g. server FQDN or YOUR name) []:ca.gr.org
Email Address []:edward.yh.lu@gmail.com
~~~
3. 把根证书添加到本地操作系统的受信任根证书列表
~~~
$ sudomkdir /usr/local/share/ca-certificates/personal-cert
$ sudocp PersonalCA.crt /usr/local/share/ca-certificates/personal-cert
$ sudo update-ca-certificates
~~~
该根证书需要安装在所有需要访问私有Docker Registry的客户端。另外，还需要重新启动Docker服务。
~~~
sudo service docker restart
~~~
4. 为私有Docker Registry服务器生成私钥
~~~
$ opensslgenrsa -out domain.key 2048
Generating RSA private key, 2048 bit long modulus
........+++
.......................+++
e is 65537 (0x10001)
~~~
5. 为私有Docker Registry服务器生成证书签发申请
~~~
$ opensslreq -new -key domain.key -out registry.gr.org.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Beijing
Locality Name (eg, city) []:Beijing
Organization Name (eg, company) [Internet Widgits Pty Ltd]:GR
Organizational Unit Name (eg, section) []:Docker
Common Name (e.g. server FQDN or YOUR name) []:registry.gr.org
Email Address []:edward.yh.lu@gmail.com
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:12345678
An optional company name []:
~~~
键入此命令后，OpenSSL将提示回答几个问题。 写入最初想要的任何东西，但是当OpenSSL提示输入Common Name时，请确保键入私有Docker Registry服务器的域名。

6. 利用自己的证书签发机构签发私有Docker Registry服务器证书
~~~
openssl x509 -req -in registry.gr.org.csr -CA PersonalCA.crt -CAkeyPersonalCA.key -CAcreateserial -out domain.crt -days 10000
~~~
7. 按照上文`已获得一个证书`的步骤安装和运行Docker Registry
### 使用
1. 从Docker Hub获取任何Docker映像，并将其标记为指向上步建立的私有Docker Registry
~~~
docker pull ubuntu&&docker tag ubuntu registry.gr.org:5000/ubuntu
~~~
2. 把ubuntu映像推到私有Docker Registry
~~~
docker push registry.gr.org:5000/zookeeper
~~~

Happy developing！

--------

参考资料
--------
1. [Deploying a registry server](https://docs.docker.com/registry/deploying/)