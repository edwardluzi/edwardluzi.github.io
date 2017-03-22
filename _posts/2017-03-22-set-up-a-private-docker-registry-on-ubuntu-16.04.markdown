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

或者编辑Docker Compose文件，内容如下：
~~~
#docker-compose.yml
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

运行如下命令来安装并运行Docker Registry。
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

首先将crt文件移动并重命名为/certs/domain.crt，并将密钥文件更改为 /certs/domain.key。

. 重新编辑Docker Compose文件如下：
~~~
#docker-compose.yml
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


如果没有可用的域名，那么也可以使用自签名证书，或者设置自己的证书签发机构，本文假定了一个域名registry.gr.org，并且在hosts文件中把这个域名指向了将要运行Docker Registry的主机。

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

2. 重新编辑Docker Compose文件如下：
~~~
#docker-compose.yml
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

3. 安装并运行Docker Registry。
~~~
$ docker-compose up
~~~

4. 确保每一个Docker客户端信任该证书。复制domain.crt文件到Docker客户端/etc/docker/certs.d/registry.gr.org:5000/ca.crt，然后重启客户端的Docker守护进程。

### 设置自己的证书签发机构

为了追求更好的灵活性，本文接下来将要介绍如何设置自己的证书签发机构。




参考资料

--------
[Deploying a registry server](https://docs.docker.com/registry/deploying/)