## 手工部署Java Web项目
> 参考链接：https://help.aliyun.com/document_detail/51376.html?spm=5176.doc52806.6.738.HGcf8r

#### 基本命令
```
`查看系统版本`
leon@zsl:/lib/java$  lsb_release -a
LSB Version:	core-9.20160110ubuntu0.2-ia32:core-9.20160110ubuntu0.2-noarch:security-9.20160110ubuntu0.2-ia32:security-9.20160110ubuntu0.2-noarch
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.2 LTS
Release:	16.04
Codename:	xenial
```

```
`查看系统位数`
leon@zsl:/lib/java$ getconf LONG_BIT
32
```

#### 更新源
> 在修改source.list前，最好先备份一份

- 执行备份命令
```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.old
```

- 用vim打开
```
sudo vim /etc/apt/source.list
```

- 复制源 复制到source.list中去并覆盖原来的文件内容。
```
# deb cdrom:[Ubuntu 16.04 LTS _Xenial Xerus_ - Release amd64 (20160420.1)]/ xenial main restricted
deb-src http://archive.ubuntu.com/ubuntu xenial main restricted #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb http://mirrors.aliyun.com/ubuntu/ xenial multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse #Added by software-properties
deb http://archive.canonical.com/ubuntu xenial partner
deb-src http://archive.canonical.com/ubuntu xenial partner
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-security multiverse
```

- update命令
```
sudo apt-get update
```

#### 安装jdk8
> 备注：
> jdk-7u7-i586 ：只能在32位系统中生效。
> jdl-7u7-x64：只能在64位系统中生效。

- 下载jdk

```
leon@zsl:/lib/java$ sudo wget http://mirrors.linuxeye.com/jdk/jdk-8u152-linux-i586.tar.gz
--2017-11-30 21:05:18--  http://mirrors.linuxeye.com/jdk/jdk-8u152-linux-i586.tar.gz
Resolving mirrors.linuxeye.com (mirrors.linuxeye.com)... 121.196.203.66
Connecting to mirrors.linuxeye.com (mirrors.linuxeye.com)|121.196.203.66|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 192699649 (184M) [application/octet-stream]
Saving to: ‘jdk-8u152-linux-i586.tar.gz’

jdk-8u152-linux-i58 100%[===================>] 183.77M  3.50MB/s    in 49s     

2017-11-30 21:06:07 (3.77 MB/s) - ‘jdk-8u152-linux-i586.tar.gz’ saved [192699649/192699649]

```

- 解压
```
tar -zxvf jdk-8u152-linux-i586.tar.gz
```

- 配置环境变量
```
sudo vi /etc/profile
```
```
在文件尾追加：
export JAVA_HOME=/lib/jdk1.8.0_45 `oops!注意这里添加是jdk的实际目录路径`
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
```
- 环境变量立即生效
```
leon@zsl:/etc$ source /etc/profile
leon@zsl:/etc$ java -version
java version "1.8.0_152"
Java(TM) SE Runtime Environment (build 1.8.0_152-b16)
Java HotSpot(TM) Client VM (build 25.152-b16, mixed mode)
```

wget http://download.jboss.org/wildfly/10.0.0.Final/wildfly-10.0.0.Final.tar.gz

cp ./wildfly/bin/init.d/wildfly-init-debian.sh /etc/init.d/wildfly


- 配置安全组  端口号 0 - 9999
- wildfly端口号 -> 8080
- 自启动
- 修改wildfly服务器 127.0.0.1
