# 第一章：MySQL 8-安装和升级

在本章中，我们将介绍以下配方：

+   使用 YUM/APT 安装 MySQL

+   使用 RPM 或 DEB 文件安装 MySQL 8.0

+   使用通用二进制文件在 Linux 上安装 MySQL

+   启动或停止 MySQL 8

+   卸载 MySQL 8

+   使用 systemd 管理 MySQL 服务器

+   从 MySQL 8.0 降级

+   升级到 MySQL 8.0

+   安装 MySQL 实用程序

# 介绍

在本章中，您将了解 MySQL 8 安装、升级和降级步骤。有五种不同的安装或升级方式；本章涵盖了三种最常用的安装方法：

+   软件存储库（YUM 或 APT）

+   RPM 或 DEB 文件

+   通用二进制文件

+   Docker（未涵盖）

+   源代码编译（未涵盖）

如果您已经安装了 MySQL 并希望升级，请查看“升级到 MySQL 8”部分中的升级步骤。如果您的安装损坏，请查看“升级到 MySQL 8”部分中的卸载步骤。

安装之前，请记下操作系统和 CPU 架构。遵循的约定如下：

**MySQL Linux RPM 包分发标识符**

| **分发值** | **预期的用途** |
| --- | --- |
| el6, el7 | Red Hat 企业 Linux，Oracle Linux，CentOS 6 或 7 |
| fc23, fc24, fc25 | Fedora 23, 24 或 25 |
| sles12 | SUSE Linux 企业服务器 12 |

**MySQL Linux RPM 包 CPU 标识符**

| **CPU 值** | **预期的处理器类型或系列** |
| --- | --- |
| i386, i586, i686 | 奔腾处理器或更高，32 位 |
| x86_64 | 64 位 x86 处理器 |
| ia64 | Itanium（IA-64）处理器 |

**MySQL Debian 和 Ubuntu 7 和 8 安装包 CPU 标识符**

| **CPU 值** | **预期的处理器类型或系列** |
| --- | --- |
| i386 | 奔腾处理器或更高，32 位 |
| amd64 | 64 位 x86 处理器 |

**MySQL Debian 6 安装包 CPU 标识符**

| **CPU 值** | **预期的处理器类型或系列** |
| --- | --- |
| i686 | 奔腾处理器或更高，32 位 |
| x86_64 | 64 位 x86 处理器 |

# 使用 YUM/APT 安装 MySQL

最常见和最简单的安装方式是通过软件存储库，您可以将官方的 Oracle MySQL 存储库添加到列表中，并通过软件包管理软件安装 MySQL。

主要有两种类型的存储库软件：

+   YUM（Centos，Red Hat，Fedora 和 Oracle Linux）

+   APT（Debian，Ubuntu）

# 如何做...

让我们看看以下安装 MySQL 8 的步骤：

# 使用 YUM 存储库

1.  查找 Red Hat 或 CentOS 版本：

```sql
shell> cat /etc/redhat-release
CentOS Linux release 7.3.1611 (Core)
```

1.  将 MySQL Yum 存储库添加到系统的存储库列表中。这是一个一次性操作，可以通过安装 MySQL 提供的 RPM 来执行。

您可以从[`dev.mysql.com/downloads/repo/yum/`](http://dev.mysql.com/downloads/repo/yum/)下载 MySQL YUM 存储库，并根据您的操作系统选择文件。

使用以下命令安装下载的发布包，将名称替换为下载的 RPM 包的特定于平台和版本的包名称：

```sql
shell> sudo yum localinstall -y mysql57-community-release-el7-11.noarch.rpm
Loaded plugins: fastestmirror
Examining mysql57-community-release-el7-11.noarch.rpm: mysql57-community-release-el7-11.noarch
Marking mysql57-community-release-el7-11.noarch.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package mysql57-community-release.noarch 0:el7-11 will be installed
--> Finished Dependency Resolution
~
  Verifying  : mysql57-community-release-el7-11.noarch 1/1 

Installed:
  mysql57-community-release.noarch 0:el7-11
Complete!
```

1.  或者您可以复制链接位置并直接使用 RPM 进行安装（安装后可以跳过下一步）：

```sql
shell> sudo rpm -Uvh "https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm"
Retrieving https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
Preparing...                          ################################# [100%]
Updating / installing...
   1:mysql57-community-release-el7-11 ################################# [100%]
```

1.  验证安装：

```sql
shell> yum repolist enabled | grep 'mysql.*-community.*'
mysql-connectors-community/x86_64 MySQL Connectors Community                  42
mysql-tools-community/x86_64      MySQL Tools Community                       53
mysql57-community/x86_64          MySQL 5.7 Community Server                 227
```

1.  设置发布系列。在撰写本书时，MySQL 8 不是**一般可用性**（**GA**）版本。因此，MySQL 5.7 将被选为默认发布系列。要安装 MySQL 8，您必须将发布系列设置为 8：

```sql
shell> sudo yum repolist all | grep mysql
mysql-cluster-7.5-community/x86_64   MySQL Cluster 7.5 Community disabled
mysql-cluster-7.5-community-source   MySQL Cluster 7.5 Community disabled
mysql-cluster-7.6-community/x86_64   MySQL Cluster 7.6 Community disabled
mysql-cluster-7.6-community-source   MySQL Cluster 7.6 Community disabled
mysql-connectors-community/x86_64    MySQL Connectors Community  enabled:     42
mysql-connectors-community-source    MySQL Connectors Community  disabled
mysql-tools-community/x86_64         MySQL Tools Community       enabled:     53
mysql-tools-community-source         MySQL Tools Community - Sou disabled
mysql-tools-preview/x86_64           MySQL Tools Preview         disabled
mysql-tools-preview-source           MySQL Tools Preview - Sourc disabled
mysql55-community/x86_64             MySQL 5.5 Community Server  disabled
mysql55-community-source             MySQL 5.5 Community Server  disabled
mysql56-community/x86_64             MySQL 5.6 Community Server  disabled
mysql56-community-source             MySQL 5.6 Community Server  disabled
mysql57-community/x86_64             MySQL 5.7 Community Server  enabled:    227
mysql57-community-source             MySQL 5.7 Community Server  disabled
mysql80-community/x86_64             MySQL 8.0 Community Server  disabled
mysql80-community-source             MySQL 8.0 Community Server  disabled
```

1.  禁用`mysql57-community`并启用`mysql80-community`：

```sql
shell> sudo yum install yum-utils.noarch -y
shell> sudo yum-config-manager --disable mysql57-community
shell> sudo yum-config-manager --enable mysql80-community
```

1.  验证`mysql80-community`是否已启用：

```sql
shell> sudo yum repolist all | grep mysql8
mysql80-community/x86_64             MySQL 8.0 Community Server  enabled:     16
mysql80-community-source             MySQL 8.0 Community Server  disabled
```

1.  安装 MySQL 8：

```sql
shell> sudo yum install -y mysql-community-server
Loaded plugins: fastestmirror
mysql-connectors-community | 2.5 kB  00:00:00     
mysql-tools-community      | 2.5 kB  00:00:00     
mysql80-community          | 2.5 kB  00:00:00     
Loading mirror speeds from cached hostfile
 * base: mirror.web-ster.com
 * epel: mirrors.cat.pdx.edu
 * extras: mirrors.oit.uci.edu
 * updates: repos.lax.quadranet.com
Resolving Dependencies
~
Transaction test succeeded
Running transaction
  Installing : mysql-community-common-8.0.3-0.1.rc.el7.x86_64   1/4 
  Installing : mysql-community-libs-8.0.3-0.1.rc.el7.x86_64     2/4 
  Installing : mysql-community-client-8.0.3-0.1.rc.el7.x86_64   3/4 
  Installing : mysql-community-server-8.0.3-0.1.rc.el7.x86_64   4/4 
  Verifying  : mysql-community-libs-8.0.3-0.1.rc.el7.x86_64     1/4 
  Verifying  : mysql-community-common-8.0.3-0.1.rc.el7.x86_64   2/4 
  Verifying  : mysql-community-client-8.0.3-0.1.rc.el7.x86_64   3/4 
  Verifying  : mysql-community-server-8.0.3-0.1.rc.el7.x86_64   4/4 

Installed:
  mysql-community-server.x86_64 0:8.0.3-0.1.rc.el7
Dependency Installed:
  mysql-community-client.x86_64 0:8.0.3-0.1.rc.el7
  mysql-community-common.x86_64 0:8.0.3-0.1.rc.el7  
  mysql-community-libs.x86_64 0:8.0.3-0.1.rc.el7                              

Complete!
```

1.  您可以使用以下命令检查已安装的软件包：

```sql
shell> rpm -qa | grep -i 'mysql.*8.*'
perl-DBD-MySQL-4.023-5.el7.x86_64
mysql-community-libs-8.0.3-0.1.rc.el7.x86_64
mysql-community-common-8.0.3-0.1.rc.el7.x86_64
mysql-community-client-8.0.3-0.1.rc.el7.x86_64
mysql-community-server-8.0.3-0.1.rc.el7.x86_64
```

# 使用 APT 存储库

1.  将 MySQL APT 存储库添加到系统的存储库列表中。这是一个一次性操作，可以通过安装 MySQL 提供的`.deb`文件来执行

您可以从[`dev.mysql.com/downloads/repo/apt/`](http://dev.mysql.com/downloads/repo/apt/)下载 MySQL APT 存储库。

或者您可以复制链接位置并使用`wget`直接在服务器上下载。您可能需要安装`wget`（`sudo apt-get install wget`）：

```sql
shell> wget "https://repo.mysql.com//mysql-apt-config_0.8.9-1_all.deb"
```

1.  使用以下命令安装下载的发布软件包，替换为下载的 APT 软件包的特定平台和版本的软件包名称：

```sql
shell> sudo dpkg -i mysql-apt-config_0.8.9-1_all.deb 
(Reading database ... 131133 files and directories currently installed.)
Preparing to unpack mysql-apt-config_0.8.9-1_all.deb ...
Unpacking mysql-apt-config (0.8.9-1) over (0.8.9-1) ...
Setting up mysql-apt-config (0.8.9-1) ...
Warning: apt-key should not be used in scripts (called from postinst maintainerscript of the package mysql-apt-config)
OK
```

1.  在安装软件包时，将要求您选择 MySQL 服务器和其他组件的版本。按*Enter*进行选择，使用上下键进行导航。

选择 MySQL 服务器和集群（当前选择：mysql-5.7）。

选择 mysql-8.0 预览（在撰写本文时，MySQL 8.0 尚未 GA）。您可能会收到警告，例如 MySQL 8.0-RC 请注意，MySQL 8.0 目前是一个 RC。它应该只安装以预览 MySQL 即将推出的功能，并不建议在生产环境中使用。 （**RC**是**发布候选**的缩写）。

如果要更改发布版本，请执行以下操作：

```sql
shell> sudo dpkg-reconfigure mysql-apt-config
```

1.  使用以下命令从 MySQL APT 存储库更新软件包信息（此步骤是强制性的）：

```sql
shell> sudo apt-get update
```

1.  安装 MySQL。在安装过程中，您需要为 MySQL 安装的 root 用户提供密码。记住密码；如果忘记了，您将不得不重置 root 密码（参考*重置 root 密码*部分）。这将安装 MySQL 服务器的软件包，以及客户端和数据库公共文件的软件包：

```sql
shell> sudo apt-get install -y mysql-community-server
~
Processing triggers for ureadahead (0.100.0-19) ...
Setting up mysql-common (8.0.3-rc-1ubuntu14.04) ...
update-alternatives: using /etc/mysql/my.cnf.fallback to provide /etc/mysql/my.cnf (my.cnf) in auto mode
Setting up mysql-community-client-core (8.0.3-rc-1ubuntu14.04) ...
Setting up mysql-community-server-core (8.0.3-rc-1ubuntu14.04) ...
~
```

1.  验证软件包。`ii`表示软件包已安装：

```sql
shell> dpkg -l | grep -i mysql
ii  mysql-apt-config            0.8.9-1               all   Auto configuration for MySQL APT Repo.
ii  mysql-client                8.0.3-rc-1ubuntu14.04 amd64 MySQL Client meta package depending on latest version
ii  mysql-common                8.0.3-rc-1ubuntu14.04 amd64 MySQL Common
ii  mysql-community-client      8.0.3-rc-1ubuntu14.04 amd64 MySQL Client
ii  mysql-community-client-core 8.0.3-rc-1ubuntu14.04 amd64 MySQL Client Core Binaries
ii  mysql-community-server      8.0.3-rc-1ubuntu14.04 amd64 MySQL Server
ii  mysql-community-server-core 8.0.3-rc-1ubuntu14.04 amd64 MySQL Server Core Binaires
```

# 使用 RPM 包安装 MySQL 8.0

使用存储库安装 MySQL 需要访问公共互联网。出于安全考虑，大多数生产机器不连接到互联网。在这种情况下，您可以在系统管理上下载 RPM 或 DEB 文件，并将其复制到生产机器。

主要有两种类型的安装文件：

+   RPM（CentOS，Red Hat，Fedora 和 Oracle Linux）

+   DEB（Debian，Ubuntu）

有多个软件包需要安装。以下是每个软件包的列表和简要描述：

+   `mysql-community-server`：数据库服务器和相关工具。

+   `mysql-community-client`：MySQL 客户端应用程序和工具。

+   `mysql-community-common`：服务器和客户端库的公共文件。

+   `mysql-community-devel`：MySQL 数据库客户端应用程序的开发头文件和库，例如 Perl MySQL 模块。

+   `mysql-community-libs`：某些语言和应用程序需要动态加载和使用 MySQL 的共享库（`libmysqlclient.so*`）。

+   `mysql-community-libs-compat`：旧版本的共享库。如果您安装了针对旧版本 MySQL 动态链接的应用程序，但希望升级到当前版本而不破坏库依赖关系，请安装此软件包。

# 如何操作...

让我们看看如何使用以下类型的包：

# 使用 RPM 包

1.  从 MySQL 下载页面[`dev.mysql.com/downloads/mysql/`](http://dev.mysql.com/downloads/mysql/)下载 MySQL RPM tar 包，选择您的操作系统和 CPU 架构。在撰写本文时，MySQL 8.0 尚未 GA。如果它仍处于开发系列中，请选择 Development Releases 选项卡以获取 MySQL 8.0，然后选择操作系统和版本：

```sql
shell> wget 'https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.3-0.1.rc.el7.x86_64.rpm-bundle.tar'
~
Saving to: ‘mysql-8.0.3-0.1.rc.el7.x86_64.rpm-bundle.tar’
~
```

1.  解压软件包：

```sql
shell> tar xfv mysql-8.0.3-0.1.rc.el7.x86_64.rpm-bundle.tar
```

1.  安装 MySQL：

```sql
shell> sudo rpm -i mysql-community-{server-8,client,common,libs}*
```

1.  RPM 无法解决依赖关系问题，安装过程可能会出现问题。如果遇到此类问题，请使用此处列出的`yum`命令（您应该可以访问依赖软件包）：

```sql
shell> sudo yum install mysql-community-{server-8,client,common,libs}* -y
```

1.  验证安装：

```sql
shell> rpm -qa | grep -i mysql-community
mysql-community-common-8.0.3-0.1.rc.el7.x86_64
mysql-community-libs-compat-8.0.3-0.1.rc.el7.x86_64
mysql-community-libs-8.0.3-0.1.rc.el7.x86_64
mysql-community-server-8.0.3-0.1.rc.el7.x86_64
mysql-community-client-8.0.3-0.1.rc.el7.x86_64
```

# 使用 APT 包

1.  从 MySQL 下载页面[`dev.mysql.com/downloads/mysql/`](http://dev.mysql.com/downloads/mysql/)下载 MySQL APT TAR：

```sql
shell> wget "https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-server_8.0.3-rc-1ubuntu16.04_amd64.deb-bundle.tar"
~
Saving to: ‘mysql-server_8.0.3-rc-1ubuntu16.04_amd64.deb-bundle.tar’
~
```

1.  解压软件包：

```sql
shell> tar -xvf mysql-server_8.0.3-rc-1ubuntu16.04_amd64.deb-bundle.tar 
```

1.  安装依赖项。如果尚未安装，您可能需要安装`libaio1`软件包：

```sql
shell> sudo apt-get install -y libaio1
```

1.  升级`libstdc++6`到最新版本：

```sql
shell> sudo add-apt-repository ppa:ubuntu-toolchain-r/test
shell> sudo apt-get update
shell> sudo apt-get upgrade -y libstdc++6
```

1.  将`libmecab2`升级到最新版本。如果未包括`universe`，则在文件末尾添加以下行（例如，`zesty`）：

```sql
shell> sudo vi /etc/apt/sources.list
deb http://us.archive.ubuntu.com/ubuntu zesty main universe

shell> sudo apt-get update
shell> sudo apt-get install libmecab2
```

1.  使用以下命令预配置 MySQL 服务器包。它会要求您设置 root 密码：

```sql
shell> sudo dpkg-preconfigure mysql-community-server_*.deb
```

1.  安装数据库公共文件包、客户端包、客户端元包、服务器包和服务器元包（按顺序）；您可以使用单个命令完成：

```sql
shell> sudo dpkg -i mysql-{common,community-client-core,community-client,client,community-server-core,community-server,server}_*.deb
```

1.  安装共享库：

```sql
shell> sudo dpkg -i libmysqlclient21_8.0.1-dmr-1ubuntu16.10_amd64.deb
```

1.  验证安装：

```sql
shell> dpkg -l | grep -i mysql
ii  mysql-client                8.0.3-rc-1ubuntu14.04 amd64 MySQL Client meta package depending on latest version
ii  mysql-common                8.0.3-rc-1ubuntu14.04 amd64 MySQL Common
ii  mysql-community-client      8.0.3-rc-1ubuntu14.04 amd64 MySQL Client
ii  mysql-community-client-core 8.0.3-rc-1ubuntu14.04 amd64 MySQL Client Core Binaries
ii  mysql-community-server      8.0.3-rc-1ubuntu14.04 amd64 MySQL Server
ii  mysql-community-server-core 8.0.3-rc-1ubuntu14.04 amd64 MySQL Server Core Binaires
ii  mysql-server                8.0.3-rc-1ubuntu16.04 amd64 MySQL Server meta package depending on latest version
```

# 使用通用二进制文件在 Linux 上安装 MySQL

使用软件包安装需要先安装一些依赖项，并可能与其他软件包冲突。在这种情况下，您可以使用下载页面上提供的通用二进制文件安装 MySQL。二进制文件是使用先进的编译器预编译的，并使用最佳选项构建以获得最佳性能。

# 如何做到...

MySQL 依赖于`libaio`库。如果未在本地安装此库，`数据目录`初始化和随后的服务器启动步骤将失败。

在基于 YUM 的系统上：

```sql
shell> sudo yum install -y libaio
```

在基于 APT 的系统上：

```sql
shell> sudo apt-get install -y libaio1
```

从 MySQL 下载页面下载 TAR 二进制文件，网址为[`dev.mysql.com/downloads/mysql/`](https://dev.mysql.com/downloads/mysql/)，然后选择 Linux - 通用作为操作系统并选择版本。您可以直接使用`wget`命令直接在服务器上下载：

```sql
shell> cd /opt
shell> wget "https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.3-rc-linux-glibc2.12-x86_64.tar.gz"
```

使用以下步骤安装 MySQL：

1.  添加`mysql`组和`mysql`用户。所有文件和目录都应该在`mysql`用户下：

```sql
shell> sudo groupadd mysql
shell> sudo useradd -r -g mysql -s /bin/false mysql
```

1.  这是安装位置（您可以将其更改为另一个位置）：

```sql
shell> cd /usr/local
```

1.  解压二进制文件。将解压后的二进制文件保留在相同位置，并将其符号链接到安装位置。通过这种方式，您可以保留多个版本，并且非常容易升级。例如，您可以下载另一个版本并将其解压到不同的位置；在升级时，您只需要更改符号链接：

```sql
shell> sudo tar zxvf /opt/mysql-8.0.3-rc-linux-glibc2.12-x86_64.tar.gz
mysql-8.0.3-rc-linux-glibc2.12-x86_64/bin/myisam_ftdump
mysql-8.0.3-rc-linux-glibc2.12-x86_64/bin/myisamchk
```

```sql
mysql-8.0.3-rc-linux-glibc2.12-x86_64/bin/myisamlog
mysql-8.0.3-rc-linux-glibc2.12-x86_64/bin/myisampack
mysql-8.0.3-rc-linux-glibc2.12-x86_64/bin/mysql
~
```

1.  创建符号链接：

```sql
shell> sudo ln -s mysql-8.0.3-rc-linux-glibc2.12-x86_64 mysql
```

1.  创建必要的目录并将所有权更改为`mysql`：

```sql
shell> cd mysql
shell> sudo mkdir mysql-files
shell> sudo chmod 750 mysql-files
shell> sudo chown -R mysql .
shell> sudo chgrp -R mysql .
```

1.  初始化`mysql`，生成临时密码：

```sql
shell> sudo bin/mysqld --initialize --user=mysql
~
2017-12-02T05:55:10.822139Z 5 [Note] A temporary password is generated for root@localhost: Aw=ee.rf(6Ua
~
```

1.  为 SSL 设置 RSA。有关 SSL 的更多详细信息，请参阅第十四章“使用 X509 部分设置加密连接”。请注意，为`root@localhost`生成了一个临时密码：eJQdj8C*qVMq

```sql
shell> sudo bin/mysql_ssl_rsa_setup
Generating a 2048 bit RSA private key
...........+++
....................................+++
writing new private key to 'ca-key.pem'
-----
Generating a 2048 bit RSA private key
...........................................................+++
...........................................+++
writing new private key to 'server-key.pem'
-----
Generating a 2048 bit RSA private key
.....+++
..........................+++
writing new private key to 'client-key.pem'
-----
```

1.  更改二进制文件的所有权为`root`，将数据文件的所有权更改为`mysql`：

```sql
shell> sudo chown -R root .
shell> sudo chown -R mysql data mysql-files
```

1.  将启动脚本复制到`init.d`：

```sql
shell> sudo cp support-files/mysql.server /etc/init.d/mysql
```

1.  将`mysql`的二进制文件导出到`PATH`环境变量：

```sql
shell> export PATH=$PATH:/usr/local/mysql/bin
```

1.  参考*启动或停止 MySQL 8*部分来启动 MySQL。

安装后，您将在`/usr/local/mysql`内获得以下目录：

| **目录** | **目录内容** |
| --- | --- |
| `bin` | `mysqld`服务器、客户端和实用程序 |
| `data` | 日志文件、数据库 |
| `docs` | 以 info 格式的 MySQL 手册 |
| `man` | Unix 手册页 |
| `include` | 包括（头）文件 |
| `lib` | 库 |
| `share` | 其他支持文件，包括错误消息、示例配置文件、用于数据库安装的 SQL |

# 还有更多...

还有其他安装方法，例如：

1.  从源代码编译。您可以从 Oracle 提供的源代码中编译和构建 MySQL，从而可以灵活定制构建参数、编译器优化和安装位置。强烈建议使用 Oracle 提供的预编译二进制文件，除非您需要特定的编译器选项或者您正在调试 MySQL。

这种方法很少使用，需要几个开发工具，超出了本书的范围。有关通过源代码安装的安装方法，您可以参考参考手册，网址为[`dev.mysql.com/doc/refman/8.0/en/source-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/source-installation.html)。

1.  使用 Docker。MySQL 服务器也可以使用 Docker 镜像安装和管理。有关安装、配置以及如何在 Docker 下使用 MySQL，请参阅[`hub.docker.com/r/mysql/mysql-server/`](https://hub.docker.com/r/mysql/mysql-server/)。

# 启动或停止 MySQL 8

安装完成后，您可以使用以下命令启动/停止 MySQL，这些命令因不同平台和安装方法而异。`mysqld`是`mysql`服务器进程。所有启动方法都调用`mysqld`脚本。

# 如何做...

让我们详细看看。除了启动和停止之外，我们还将了解有关检查服务器状态的一些内容。让我们看看如何。

# 启动 MySQL 8.0 服务器

您可以使用以下命令启动服务器：

1.  使用`service`：

```sql
shell> sudo service mysql start
```

1.  使用`init.d`：

```sql
shell> sudo /etc/init.d/mysql start
```

1.  如果找不到启动脚本（在进行二进制安装时），可以从解压位置复制。

```sql
shell> sudo cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
```

1.  如果您的安装包括`systemd`支持：

```sql
shell> sudo systemctl start mysqld
```

1.  如果没有`systemd`支持，可以使用`mysqld_safe`启动 MySQL。`mysqld_safe`是`mysqld`的启动脚本，用于保护`mysqld`进程。如果`mysqld`被杀死，`mysqld_safe`会尝试重新启动进程：

```sql
shell> sudo mysqld_safe --user=mysql &
```

启动后，

1.  服务器已初始化。

1.  SSL 证书和密钥文件在`数据目录`中生成。

1.  安装并启用了`validate_password`插件。

1.  创建了一个超级用户帐户，`root'@'localhost`。为超级用户设置了密码，并将其存储在错误日志文件中（不适用于二进制安装）。要显示它，请使用以下命令：

```sql
shell> sudo  grep "temporary password" /var/log/mysqld.log 
2017-12-02T07:23:20.915827Z 5 [Note] A temporary password is generated for root@localhost: bkvotsG:h6jD
```

您可以使用临时密码连接到 MySQL。

```sql
shell> mysql -u root -pbkvotsG:h6jD
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 7
Server version: 8.0.3-rc-log

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

1.  尽快使用生成的临时密码登录并为超级用户帐户设置自定义密码更改根密码：

```sql
# You will be prompted for a password, enter the one you got from the previous step

mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'NewPass4!';
Query OK, 0 rows affected (0.01 sec)

# password should contain at least one Upper case letter, one lowercase letter, one digit, and one special character, and that the total password length is at least 8 characters
```

# 停止 MySQL 8.0 服务器

停止 MySQL 并检查状态与启动它类似，只是一个单词的变化：

1.  使用`service`：

```sql
shell> sudo service mysqld stop
Redirecting to /bin/systemctl stop  mysqld.service
```

1.  使用`init.d`：

```sql
shell> sudo /etc/init.d/mysql stop
[ ok ] Stopping mysql (via systemctl): mysql.service.
```

1.  如果您的安装包括`systemd`支持（参见*使用 systemd 管理 MySQL 服务器*部分）：

```sql
shell> sudo systemctl stop mysqld
```

1.  使用`mysqladmin`：

```sql
shell> mysqladmin -u root -p shutdown
```

# 检查 MySQL 8.0 服务器的状态

1.  使用`service`：

```sql
shell> sudo systemctl status mysqld
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/mysqld.service.d
           └─override.conf
 Active: active (running) since Sat 2017-12-02 07:33:53 UTC; 14s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 10472 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 10451 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 10477 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─10477 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid --general_log=1

Dec 02 07:33:51 centos7 systemd[1]: Starting MySQL Server...
Dec 02 07:33:53 centos7 systemd[1]: Started MySQL Server.
```

1.  使用`init.d`：

```sql
shell> sudo /etc/init.d/mysql status
● mysql.service - LSB: start and stop MySQL
   Loaded: loaded (/etc/init.d/mysql; bad; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:systemd-sysv-generator(8)

Dec 02 06:01:00 ubuntu systemd[1]: Starting LSB: start and stop MySQL...
Dec 02 06:01:00 ubuntu mysql[20334]: Starting MySQL
Dec 02 06:01:00 ubuntu mysql[20334]:  *
Dec 02 06:01:00 ubuntu systemd[1]: Started LSB: start and stop MySQL.
Dec 02 06:01:00 ubuntu mysql[20334]: 2017-12-02T06:01:00.969284Z mysqld_safe A mysqld process already exists
Dec 02 06:01:55 ubuntu systemd[1]: Stopping LSB: start and stop MySQL...
Dec 02 06:01:55 ubuntu mysql[20445]: Shutting down MySQL
Dec 02 06:01:57 ubuntu mysql[20445]: .. *
Dec 02 06:01:57 ubuntu systemd[1]: Stopped LSB: start and stop MySQL.
Dec 02 07:26:33 ubuntu systemd[1]: Stopped LSB: start and stop MySQL.
```

1.  如果您的安装包括`systemd`支持（参见*使用 systemd 管理 MySQL 服务器*部分）：

```sql
shell> sudo systemctl status mysqld
```

# 卸载 MySQL 8

如果您在安装过程中出现问题或不想要 MySQL 8 版本，则可以使用以下步骤卸载。在卸载之前，请确保备份文件（参见第七章*备份*），如果需要，请停止 MySQL。

# 如何做...

在不同系统上，卸载将以不同的方式处理。让我们看看如何。

# 在基于 YUM 的系统上

1.  检查是否存在任何现有软件包：

```sql
shell> rpm -qa | grep -i mysql-community
mysql-community-libs-8.0.3-0.1.rc.el7.x86_64
mysql-community-common-8.0.3-0.1.rc.el7.x86_64
mysql-community-client-8.0.3-0.1.rc.el7.x86_64
mysql-community-libs-compat-8.0.3-0.1.rc.el7.x86_64
mysql-community-server-8.0.3-0.1.rc.el7.x86_64
```

1.  删除软件包。您可能会收到通知，有其他软件包依赖于 MySQL。如果您打算再次安装 MySQL，可以通过传递`--nodeps`选项忽略警告：

```sql
shell> rpm -e <package-name>
```

例如：

```sql
shell> sudo rpm -e mysql-community-server
```

1.  要删除所有软件包：

```sql
shell> sudo rpm -qa | grep -i mysql-community | xargs sudo rpm -e --nodeps
warning: /etc/my.cnf saved as /etc/my.cnf.rpmsave
```

# 在基于 APT 的系统上

1.  检查是否存在任何现有软件包：

```sql
shell> dpkg -l | grep -i mysql
```

1.  使用以下命令删除软件包：

```sql
shell> sudo apt-get remove mysql-community-server mysql-client mysql-common mysql-community-client mysql-community-client-core mysql-community-server mysql-community-server-core -y
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages will be REMOVED:
  mysql-client mysql-common mysql-community-client mysql-community-client-core mysql-community-server mysql-community-server-core mysql-server
0 upgraded, 0 newly installed, 7 to remove and 341 not upgraded.
After this operation, 357 MB disk space will be freed.
(Reading database ... 134358 files and directories currently installed.)
Removing mysql-server (8.0.3-rc-1ubuntu16.04) ...
Removing mysql-community-server (8.0.3-rc-1ubuntu16.04) ...
update-alternatives: using /etc/mysql/my.cnf.fallback to provide /etc/mysql/my.cnf (my.cnf) in auto mode
Removing mysql-client (8.0.3-rc-1ubuntu16.04) ...
Removing mysql-community-client (8.0.3-rc-1ubuntu16.04) ...
Removing mysql-common (8.0.3-rc-1ubuntu16.04) ...
Removing mysql-community-client-core (8.0.3-rc-1ubuntu16.04) ...
Removing mysql-community-server-core (8.0.3-rc-1ubuntu16.04) ...
Processing triggers for man-db (2.7.5-1) ...
```

或使用以下命令删除它们：

```sql
shell> sudo apt-get remove --purge mysql-\* -y
shell> sudo apt-get autoremove -y
```

1.  验证软件包是否已卸载：

```sql
shell> dpkg -l | grep -i mysql
ii  mysql-apt-config        0.8.9-1               all          Auto configuration for MySQL APT Repo.
rc  mysql-common            8.0.3-rc-1ubuntu16.04 amd64        MySQL Common
rc  mysql-community-client  8.0.3-rc-1ubuntu16.04 amd64        MySQL Client
rc  mysql-community-server  8.0.3-rc-1ubuntu16.04 amd64        MySQL Server
```

`rc`表示软件包已被删除（`r`），只保留了配置文件（`c`）。

# 卸载二进制文件

卸载二进制安装非常简单。您只需要删除符号链接：

1.  更改目录到安装路径：

```sql
shell> cd /usr/local
```

1.  检查`mysql`指向的位置，这将显示它引用的路径：

```sql
shell> sudo ls -lh mysql
```

1.  删除`mysql`：

```sql
shell> sudo rm mysql
```

1.  删除二进制文件（可选）：

```sql
shell> sudo rm -f /opt/mysql-8.0.3-rc-linux-glibc2.12-x86_64.tar.gz
```

# 使用`systemd`管理 MySQL 服务器

如果您使用 RPM 或 Debian 软件包服务器安装 MySQL，则启动和关闭由`systemd`管理。对于安装了 MySQL 的平台，不会安装`mysqld_safe`，`mysqld_multi`和`mysqld_multi.server`。MySQL 服务器的启动和关闭由`systemd`使用`systemctl`命令进行管理。您需要配置`systemd`如下。

基于 RPM 的系统使用`mysqld.service`文件，基于 APT 的系统使用`mysql.server`文件。

# 如何做...

1.  创建本地化的`systemd`配置文件：

```sql
shell> sudo mkdir -pv /etc/systemd/system/mysqld.service.d
```

1.  创建/打开`conf`文件：

```sql
shell> sudo vi /etc/systemd/system/mysqld.service.d/override.conf
```

1.  输入以下内容：

```sql
[Service]
LimitNOFILE=max_open_files (ex: 102400)
PIDFile=/path/to/pid/file (ex: /var/lib/mysql/mysql.pid)
Nice=nice_level (ex: -10)
Environment="LD_PRELOAD=/path/to/malloc/library" Environment="TZ=time_zone_setting"
```

1.  重新加载`systemd`：

```sql
shell> sudo systemctl daemon-reload
```

1.  对于临时更改，您可以在不编辑`conf`文件的情况下重新加载：

```sql
shell> sudo systemctl set-environment MYSQLD_OPTS="--general_log=1"
or unset using
shell> sudo systemctl unset-environment MYSQLD_OPTS
```

1.  修改`systemd`环境后，重新启动服务器以使更改生效。

启用`mysql.serviceshell> sudo systemctl`，并启用`mysql.service`：

```sql
shell> sudo systemctl unmask mysql.service
```

1.  重新启动`mysql`：

在 RPM 平台上：

```sql
shell> sudo systemctl restart mysqld
```

在 Debian 平台上：

```sql
shell> sudo systemctl restart mysql
```

# 从 MySQL 8.0 降级

如果您的应用程序表现不如预期，您可以随时降级到以前的 GA 版本（MySQL 5.7）。在降级之前，建议进行逻辑备份（参考第七章*备份*）。请注意，您只能降级一个先前的版本。假设您想要从 MySQL 8.0 降级到 MySQL 5.6，您必须先降级到 MySQL 5.7，然后从 MySQL 5.7 降级到 MySQL 5.6。

您可以通过两种方式完成：

+   原地降级（在 MySQL 8 内部降级）

+   逻辑降级

# 如何做...

在以下小节中，您将学习如何使用各种存储库、捆绑包等处理安装/卸载/升级/降级。

# 原地降级

在 MySQL 8.0 中的 GA 状态发布之间进行降级（请注意，您不能使用此方法降级到 MySQL 5.7）：

1.  关闭旧的 MySQL 版本

1.  替换 MySQL 8.0 二进制文件或旧的二进制文件

1.  在现有的`数据目录`上重新启动 MySQL

1.  运行`mysql_upgrade`实用程序

# 使用 YUM 存储库

1.  准备 MySQL 进行缓慢关闭，以确保撤消日志为空，并且数据文件在不同版本之间的文件格式差异的情况下已完全准备好：

```sql
mysql> SET GLOBAL innodb_fast_shutdown = 0;
```

1.  按照*停止 MySQL 8.0 服务器*部分中的说明关闭`mysql`服务器：

```sql
shell> sudo systemctl stop mysqld
```

1.  从`数据目录`中删除`InnoDB`重做日志文件（`ib_logfile*`文件），以避免降级问题与重做日志文件格式更改之间的关联，这些更改可能发生在版本之间：

```sql
shell> sudo rm -rf /var/lib/mysql/ib_logfile*
```

1.  降级 MySQL。要降级服务器，您需要卸载 MySQL 8.0，如*卸载 MySQL 8*部分中所述。配置文件将自动存储为备份。

列出可用版本：

```sql
shell> sudo yum list mysql-community-server
```

降级很棘手；最好在降级之前删除现有的软件包：

```sql
shell> sudo rpm -qa | grep -i mysql-community | xargs sudo rpm -e --nodeps
warning: /etc/my.cnf saved as /etc/my.cnf.rpmsave
```

安装旧版本：

```sql
shell> sudo yum install -y mysql-community-server-<version>
```

# 使用 APT 存储库

1.  重新配置 MySQL 并选择旧版本：

```sql
shell> sudo dpkg-reconfigure mysql-apt-config
```

1.  运行`apt-get update`：

```sql
shell> sudo apt-get update
```

1.  删除当前版本：

```sql
shell> sudo apt-get remove mysql-community-server mysql-client mysql-common mysql-community-client mysql-community-client-core mysql-community-server mysql-community-server-core -y

shell> sudo apt-get autoremove
```

1.  安装旧版本（自动选择，因为您已经重新配置）：

```sql
shell> sudo apt-get install -y mysql-server
```

# 使用 RPM 或 APT 捆绑包

卸载现有的软件包（参考*卸载 MySQL 8*部分）并安装新的软件包，可以从 MySQL 下载（参考*使用 RPM 或 DEB 文件安装 MySQL 8.0*部分）。

# 使用通用二进制文件

如果您通过二进制文件安装了 MySQL，您必须删除到旧版本的符号链接（参考*卸载 MySQL 8*部分），然后进行新安装（参考*使用通用二进制文件在 Linux 上安装 MySQL*部分）：

1.  按照*启动或停止 MySQL 8*部分中的说明启动服务器。请注意，所有版本的启动过程相同。

1.  运行`mysql_upgrade`实用程序：

```sql
shell> sudo mysql_upgrade -u root -p
```

1.  重新启动 MySQL 服务器，以确保对系统表所做的任何更改生效：

```sql
shell> sudo systemctl restart mysqld
```

# 逻辑降级

以下是步骤概述：

1.  使用逻辑备份从 MySQL 8.0 版本中导出现有数据（参考第七章*备份*中的逻辑备份方法）

1.  安装 MySQL 5.7

1.  将转储文件加载到 MySQL 5.7 版本中（参考*恢复数据*章节中的恢复方法）

1.  运行`mysql_upgrade`实用程序

以下是详细步骤：

1.  您需要对数据库进行逻辑备份。（参考第七章，*备份*中的`mydumper`进行更快的备份）：

```sql
shell> mysqldump -u root -p --add-drop-table --routines --events --all-databases --force > mysql80.sql
```

1.  按照*启动或停止 MySQL 8*部分中的说明关闭 MySQL 服务器。

1.  移动`数据目录`。如果要保留 MySQL 8，可以将`数据目录`移回（在步骤 1 中不需要恢复 SQL 备份）：

```sql
shell> sudo mv /var/lib/mysql /var/lib/mysql80
```

1.  降级 MySQL。要降级服务器，我们需要卸载 MySQL 8。配置文件会自动备份。

# 使用 YUM 存储库

卸载后，安装旧版本：

1.  切换存储库：

```sql
shell> sudo yum-config-manager --disable mysql80-community
shell> sudo yum-config-manager --enable mysql57-community
```

1.  验证`mysql57-community`已启用：

```sql
shell> yum repolist enabled | grep "mysql.*-community.*"
!mysql-connectors-community/x86_64 MySQL Connectors Community                 42
!mysql-tools-community/x86_64      MySQL Tools Community                      53
!mysql57-community/x86_64          MySQL 5.7 Community Server                227
```

1.  降级很棘手；最好在降级之前删除现有的软件包：

```sql
shell> sudo rpm -qa | grep -i mysql-community | xargs sudo rpm -e --nodeps
warning: /etc/my.cnf saved as /etc/my.cnf.rpmsave
```

1.  列出可用版本：

```sql
shell> sudo yum list mysql-community-server
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.rackspace.com
 * epel: mirrors.develooper.com
 * extras: centos.s.uw.edu
 * updates: mirrors.syringanetworks.net
Available Packages
mysql-community-server.x86_64   5.7.20-1.el7                         mysql57-community
```

1.  安装 MySQL 5.7：

```sql
shell> sudo yum install -y mysql-community-server
```

# 使用 APT 存储库

1.  重新配置`apt`以切换到 MySQL 5.7：

```sql
shell> sudo dpkg-reconfigure mysql-apt-config
```

1.  运行`apt-get update`：

```sql
shell> sudo apt-get update
```

1.  删除当前版本：

```sql
shell> sudo apt-get remove mysql-community-server mysql-client mysql-common mysql-community-client mysql-community-client-core mysql-community-server mysql-community-server-core -y
shell> sudo apt-get autoremove
```

1.  安装 MySQL 5.7：

```sql
shell> sudo apt-get install -y mysql-server
```

# 使用 RPM 或 APT 捆绑包

卸载现有的软件包（参考*卸载 MySQL 8*部分）并安装新的软件包，可以从 MySQL 下载（参考*使用 RPM 或 DEB 文件安装 MySQL 8*部分）下载。

# 使用通用二进制文件

如果通过二进制文件安装了 MySQL，必须删除到旧版本的符号链接（参考*卸载 MySQL 8*部分），然后进行新安装（参考*在 Linux 上使用通用二进制文件安装 MySQL*部分）。

降级 MySQL 后，必须恢复备份并运行`mysql_upgrade`实用程序：

1.  启动 MySQL（参考*启动或停止 MySQL 8*部分）。您需要再次重置密码。

1.  恢复备份（这可能需要很长时间，具体取决于备份的大小）。参考第八章，*恢复数据*，了解名为`myloader`的快速恢复方法：

```sql
shell> mysql -u root -p < mysql80.sql
```

1.  运行`mysql_upgrade`：

```sql
shell> mysql_upgrade -u root -p
```

1.  重新启动 MySQL 服务器，以确保对系统表所做的任何更改生效。参考*启动或停止 MySQL 8*部分：

```sql
shell> sudo /etc/init.d/mysql restart
```

# 升级到 MySQL 8.0

MySQL 8 使用包含事务表中数据库对象信息的全局`数据字典`。在以前的版本中，字典数据存储在元数据文件和非事务系统表中。您需要将`数据目录`从基于文件的结构升级到数据字典结构。

与降级一样，可以使用两种方法进行升级：

+   原地升级

+   逻辑升级

在升级之前，您还应该检查一些先决条件。

# 准备工作

1.  检查过时的数据类型或触发器，其缺少或空的定义者或无效的创建上下文：

```sql
shell> sudo mysqlcheck -u root -p --all-databases --check-upgrade
```

1.  不能有使用不支持本机分区的存储引擎的分区表。要识别这些表，执行此查询：

```sql
shell> SELECT TABLE_SCHEMA, TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE ENGINE NOT IN ('innodb', 'ndbcluster') AND CREATE_OPTIONS LIKE '%partitioned%';
```

如果有这些表，请将它们更改为`InnoDB`：

```sql
mysql> ALTER TABLE table_name ENGINE = INNODB;
```

或删除分区：

```sql
mysql> ALTER TABLE table_name REMOVE PARTITIONING;
```

1.  MySQL 5.7 `mysql`系统数据库中不能有与 MySQL 8.0 `数据字典`使用的表同名的表。要识别具有这些名称的表，执行此查询：

```sql
mysql> SELECT TABLE_SCHEMA, TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE LOWER(TABLE_SCHEMA) = 'mysql' and LOWER(TABLE_NAME) IN ('catalogs', 'character_sets', 'collations', 'column_type_elements', 'columns', 'events', 'foreign_key_column_usage', 'foreign_keys', 'index_column_usage', 'index_partitions', 'index_stats', 'indexes', 'parameter_type_elements', 'parameters', 'routines', 'schemata', 'st_spatial_reference_systems', 'table_partition_values', 'table_partitions', 'table_stats', 'tables', 'tablespace_files', 'tablespaces', 'triggers', 'version', 'view_routine_usage', 'view_table_usage');
```

1.  表中不能有外键约束名称超过 64 个字符。要识别约束名称过长的表，执行此查询：

```sql
mysql> SELECT CONSTRAINT_SCHEMA, TABLE_NAME, CONSTRAINT_NAME FROM INFORMATION_SCHEMA.REFERENTIAL_CONSTRAINTS WHERE LENGTH(CONSTRAINT_NAME) > 64;
```

1.  不受 MySQL 8.0 支持的表，如`ndb`，应移至`InnoDB`：

```sql
mysql> ALTER TABLE tablename ENGINE=InnoDB;
```

# 如何做...

与以前的方法一样，以下各小节将带您了解各种系统、捆绑包等的详细信息。

# 原地升级

以下是步骤概述：

1.  关闭旧的 MySQL 版本。

1.  用新的 MySQL 二进制文件或软件包替换旧的（详细步骤涵盖了不同类型安装方法）。

1.  在现有的`数据目录`上重新启动 MySQL。

1.  运行`mysql_upgrade`实用程序。

1.  在 MySQL 5.7 服务器上，如果有加密的`InnoDB`表空间，通过执行此语句来旋转`keyring`主密钥：

```sql
mysql> ALTER INSTANCE ROTATE INNODB MASTER KEY;
```

以下是详细步骤：

1.  配置您的 MySQL 5.7 服务器以执行慢关闭。通过慢关闭，`InnoDB`在关闭之前执行完整的清除和更改缓冲区合并，以确保撤消日志为空，并且数据文件在发布之间的文件格式差异的情况下已经准备就绪。

这一步是最重要的，因为如果没有进行这一步，您将遇到以下错误：

```sql
[ERROR] InnoDB: Upgrade after a crash is not supported. 
```

此重做日志是使用 MySQL 5.7.18 创建的。请按照[`dev.mysql.com/doc/refman/8.0/en/upgrading.html`](http://dev.mysql.com/doc/refman/8.0/en/upgrading.html)上的说明进行操作：

```sql
mysql> SET GLOBAL innodb_fast_shutdown = 0;
```

1.  按照*启动或停止 MySQL 8*部分中的描述关闭 MySQL 服务器。

升级 MySQL 二进制文件或软件包。

# 基于 YUM 的系统

1.  切换存储库：

```sql
shell> sudo yum-config-manager --disable mysql57-community
shell> sudo yum-config-manager --enable mysql80-community
```

1.  验证`mysql80-community`是否已启用：

```sql
shell> sudo yum repolist all | grep mysql8
mysql80-community/x86_64             MySQL 8.0 Community Server  enabled:     16
mysql80-community-source             MySQL 8.0 Community Server  disabled
```

1.  运行 yum update：

```sql
shell> sudo yum update mysql-server
```

# 基于 APT 的系统

1.  重新配置`apt`以切换到 MySQL 8.0：

```sql
shell> sudo dpkg-reconfigure mysql-apt-config
```

1.  运行`apt-get update`：

```sql
shell> sudo apt-get update
```

1.  删除当前版本：

```sql
shell> sudo apt-get remove mysql-community-server mysql-client mysql-common mysql-community-client mysql-community-client-core mysql-community-server mysql-community-server-core -y
shell> sudo apt-get autoremove
```

1.  安装 MySQL 8：

```sql
shell> sudo apt-get update
shell> sudo apt-get install mysql-server
shell> sudo apt-get install libmysqlclient21
```

# 使用 RPM 或 APT 捆绑包

卸载现有的软件包（参考*卸载 MySQL 8*部分），并安装新的软件包，可以从 MySQL 下载（参考*使用 RPM 或 DEB 文件安装 MySQL 8.0*部分）。

# 使用通用二进制文件

如果您通过二进制文件安装了 MySQL，则必须删除到旧版本的符号链接（参考*卸载 MySQL 8*部分），并进行新安装（参考*使用通用二进制文件在 Linux 上安装 MySQL*部分）。

启动 MySQL 8.0 服务器（参考*启动或停止 MySQL 8 以启动 MySQL*部分）。如果有加密的`InnoDB`表空间，请使用`--early-plugin-load`选项加载`keyring`插件。

服务器会自动检测`数据字典`表是否存在。如果没有，服务器将在`数据目录`中创建它们，填充它们的元数据，然后继续其正常的启动顺序。在此过程中，服务器将升级所有数据库对象的元数据，包括数据库、表空间、系统和用户表、视图和存储程序（存储过程和函数、触发器、事件调度器事件）。服务器还会删除以前用于存储元数据的文件。例如，升级后，您会注意到您的表不再有`.frm`文件。

服务器创建一个名为`backup_metadata_57`的目录，并将 MySQL 5.7 使用的文件移入其中。服务器将`event`和`proc`表重命名为`event_backup_57`和`proc_backup_57`。如果升级失败，服务器将所有更改恢复到`数据目录`。在这种情况下，您应该删除所有重做日志文件，在相同的`数据目录`上启动您的 MySQL 5.7 服务器，并修复任何错误的原因。然后，执行另一个 MySQL 5.7 服务器的慢关闭，并启动 MySQL 8.0 服务器再次尝试。

运行`mysql_upgrade`实用程序：

```sql
shell> sudo mysql_upgrade -u root -p
```

`mysql_upgrade`检查所有数据库中的所有表与当前版本的 MySQL 的不兼容性。它在 MySQL 5.7 和 MySQL 8.0 之间的`mysql`系统数据库中进行任何剩余的更改，以便您可以利用新的权限或功能。`mysql_upgrade`还将性能模式、`INFORMATION_SCHEMA`和`sys schema`对象更新到 MySQL 8.0 的最新状态。

重新启动 MySQL 服务器（参考*启动或停止 MySQL 8 以启动 MySQL*部分）。

# 逻辑升级

以下是步骤概述：

1.  使用`mysqldump`从旧的 MySQL 版本中导出现有数据

1.  安装新的 MySQL 版本

1.  将转储文件加载到新的 MySQL 版本中

1.  运行`mysql_upgrade`实用程序

以下是详细步骤：

1.  您需要对数据库进行逻辑备份（参考第七章，*备份*中的`mydumper`进行更快的备份）：

```sql
shell> mysqldump -u root -p --add-drop-table --routines --events --all-databases --ignore-table=mysql.innodb_table_stats --ignore-table=mysql.innodb_index_stats --force > data-for-upgrade.sql
```

1.  关闭 MySQL 服务器（参考*启动或停止 MySQL 8*部分）。

1.  安装新的 MySQL 版本（参考*就地升级*部分）。

1.  启动 MySQL 服务器（参考“启动或停止 MySQL 8”部分）。

1.  重置临时`root`密码：

```sql
shell> mysql -u root -p
Enter password: **** (enter temporary root password from error log)

mysql> ALTER USER USER() IDENTIFIED BY 'your new password';
```

1.  恢复备份（这可能需要很长时间，具体取决于备份的大小）。参考第八章“恢复数据”中的`myloader`快速恢复方法：

```sql
shell> mysql -u root -p --force < data-for-upgrade.sql
```

1.  运行`mysql_upgrade`实用程序：

```sql
shell> sudo mysql_upgrade -u root -p
```

1.  重新启动 MySQL 服务器（参考“启动或停止 MySQL 8”部分）。

# 安装 MySQL 实用工具

MySQL 实用工具为您提供非常方便的工具，可以在没有太多手动操作的情况下顺利进行日常操作。

# 如何做...

可以通过以下方式在基于 YUM 和 APT 的系统上安装。让我们来看看。

# 在基于 YUM 的系统上

从 MySQL 下载页面[https://dev.mysql.com/downloads/utilities/]下载文件，选择 Red Hat Enterprise Linux/Oracle Linux，或直接使用`wget`从此链接下载：

```sql
shell> wget https://cdn.mysql.com//Downloads/MySQLGUITools/mysql-utilities-1.6.5-1.el7.noarch.rpm

shell> sudo yum localinstall -y mysql-utilities-1.6.5-1.el7.noarch.rpm
```

# 在基于 APT 的系统上

从 MySQL 下载页面[https://dev.mysql.com/downloads/utilities/]下载文件，选择 Ubuntu Linux，或直接使用`wget`从此链接下载：

```sql
shell> wget "https://cdn.mysql.com//Downloads/MySQLGUITools/mysql-utilities_1.6.5-1ubuntu16.10_all.deb"
shell> sudo dpkg -i mysql-utilities_1.6.5-1ubuntu16.10_all.deb
shell> sudo apt-get install -f
```
