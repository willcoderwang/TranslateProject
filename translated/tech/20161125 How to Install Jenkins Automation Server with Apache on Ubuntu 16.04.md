如何在 Ubuntu 16.04 下安装基于 Apache 的 Jenkins 自动化服务器
============================================================


Jenkins 是一个复刻于 Hudson 项目的自动化服务器。 Jenkins 是一个基于服务器的应用， 运行在 Java servlet 容器中， 它支持包括 Git, SVN 和 Mercurial 在内的许多SCM（源代码控制管理）软件系统。 Jenkins 提供了数百插件来自动化你的项目。 Jenkins 由 Kohsuke Kawaguchi 创建， 首次发布于2011年， 使用 MIT 许可证， 它是自由软件。

在这个教程中， 我将向你展示如何在 Ubuntu Server 16.04 上安装最新版的 Jenkins。 我们将以我们自己的域名运行 Jenkins， 安装和配置 Jenkins 使它以反向代理运行在 apache web 服务器下。

### 前提

*   Ubuntu Server 16.04 - 64 位
*   Root 权限

### 第 1 步 - 安装 Java OpenJDK 7

Jenkins 基于 Java， 所以我们需要安装 Java OpenJDK 7 版本。 在这一步， 我们将从一个 PPA 库安装 Java 7, 我们首先来添加这个 PPA 库。

默认情况下， Ubuntu 16.04 没有安装 python-software-properties 包来管理 PPA 库， 所以我们首先需要安装它。 使用 apt 命令来安装 python-software-properties。

apt-get install python-software-properties

然后， 添加 Java PPA 库。

add-apt-repository ppa:openjdk-r/ppa
敲回车

更新 Ubuntu 库并使用 apt 命令安装 Java OpenJDK。

apt-get update
apt-get install openjdk-7-jdk

使用以下命令验证安装：

java -version

你会得到服务器上所安装的 Java 版本。

[
 ![Install Java 7 openJDK on Ubuntu 16.04](https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/1.png) 
][9]

### 第 2 步 - 安装 Jenkins

Jenkins 提供了一个 Ubuntu 库， 我们从这个库安装 Jenkins。

使用以下命令添加 Jenkins 密钥和库。

wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
echo 'deb https://pkg.jenkins.io/debian-stable binary/' | tee -a /etc/apt/sources.list

更新库并安装 Jenkins。

apt-get update
apt-get install jenkins

安装完成后， 使用 systemctl 命令启动 Jenkins。

systemctl start jenkins

检查 Jenkins 默认端口（8080端口) 来验证 Jenkins 是否在运行。 我使用 netstat 命令来检查， 命令如下：

netstat -plntu

Jenkins 已经安装了， 并运行于8080端口。

[
 ![Jenkins has been installed on port 8080](https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/2.png) 
][10]

### 第 3 步 - 安装 Apache 并设置为 Jenkins 的反向代理

在本教程中， 我们把 Jenkins 运行于 Apache web 服务器之下， 我们将设置 Apache 为 Jenkins 的反向代理。 首先，我安装 apache 并启用一些所需的模块， 我将使用域名 my.jenkins.id 来为 Jenkins 创建 virtual host 文件。 请使用你自己的域名，并在所有的它出现的配置文件中替换它。

从 Ubuntu 库中安装 apache2 web 服务器。

apt-get install apache2

安装完成后，启用 proxy 和 proxy_http 模块， 这样我们就可以把 apache 设置为 Jenkins 的前端服务器/反向代理。

a2enmod proxy
a2enmod proxy_http

Next, create a new virtual host file in the sites-available directory.
然后， 在 sites-available 目录下创建一个新的 virtual host 文件。

cd /etc/apache2/sites-available/
vim jenkins.conf

粘贴下面的 virtual host 设置。

```
<Virtualhost *:80>
    ServerName        my.jenkins.id
    ProxyRequests     Off
    ProxyPreserveHost On
    AllowEncodedSlashes NoDecode

    <Proxy http://localhost:8080/*>
      Order deny,allow
      Allow from all
    </Proxy>

    ProxyPass         /  http://localhost:8080/ nocanon
    ProxyPassReverse  /  http://localhost:8080/
    ProxyPassReverse  /  http://my.jenkins.id/
</Virtualhost>
```

保存文件。 并使用 a2ensite 命令激活 Jenkins virtual host。

a2ensite jenkins

重启 Apache 和 Jenkins。

systemctl restart apache2
systemctl restart jenkins

检查端口 80 和 8000 是否被 Jenkins 和 Apache 占用。

netstat -plntu

[
 ![Check that Apache and Jenkins are running](https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/3.png) 
][11]

### 第 4 步 - 设置 Jenkins

Jenkins 在以 “my.jenkins.id" 的域名运行。 打开你的 web 浏览器并输入 URL。 你将会跳转到一个页面， 要求你初始化你的 admin 密码。 Jenkins 已经产生了一个密码， 所以你只需查看并复制到密码输入框即可。

使用 cat 命令查看初始 admin 密码。
Show initial admin password Jenkins with cat command.

cat /var/lib/jenkins/secrets/initialAdminPassword

a1789d1561bf413c938122c599cf65c9

[
 ![Get the Jenkins admin password](https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/4.png) 
][12]

粘贴结果到密码框， 并点击  “**Continue**”

[
 ![Jenkins Installation and Configuration](https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/5.png) 
][13]

我们为 Jenkins 安装一些插件， 以便以后使用。 选择 “**Install Suggested Plugins**”， 并点击它。

[
 ![Install jenkins Plugins](https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/6.png) 
][14]

Jenkins 正在安装插件。

[
 ![Jenkins plugins get installed](https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/7.png) 
][15]

插件安装完成后， 我们要创建一个新的 admin 密码。 输入你的 admin 用户名， 密码， 邮件等等，并点击 “**Save and Finish**”。

[
 ![Create Jenkins admin account](https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/8.png) 
][16]

点击开始并开始使用 Jenkins. 你被会重定向到 Jenkins admin 仪表盘。

[
 ![Get redirected to the admin dashboard](https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/9.png) 
][17]

Jenkins 安装和配置成功完成

[
 ![The Jenkins admin dashboard](https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/10.png) 
][18]

### 第 5 步 - Jenkins 安全

在 Jenkins admin 仪表盘, 我们需要为 Jenkins 设置标准安全设置， 点击 “**Manage Jenkins**”， 然后点击 “**Configure Global Security**”。

[
 ![Jenkins Global Security Settings](https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/11.png) 
][19]

Jenkins 在 “**Access Control**” 部分中提供了几种认证方法。 我选择 “**Matrix-based Security**” 以便可以控制所有的用户权限。 在 “**User/Group**” 中启用 admin 用户并点击 **add**。 点击 **checking all options** 赋予 admin 所有权限， 并只给 anonymous 读取权限。 点击 “**Save**”。

[
 ![Configure Jenkins Permissions](https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/12.png) 
][20]

你会被重定向到仪表盘， 如果有登录选项， 只需输入你的 admin 用户的密码。

### 第 6 步 - 测试一个简单的自动任务

在这个部分，我只是想通过一个简单的任务来测试 Jenkins 服务器。 我将创建一个简单任务来测试 Jenkins， 并使用 top 命令查看服务器负荷。

在 Jenkins admin 仪表盘， 点击 “**Create New Job**”。

[
 ![Create a new Job in Jenkins](https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/13.png) 
][21]

Enter the job name, I'll use 'Checking System' here, select '**Freestyle Project**' and click '**OK**'.
输入任务名， 我这时使用 “Checking System”， 选择 “**Freestyle Project**” 并点击 “**OK**”。

[
 ![Configure new Jenkins Job](https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/14.png) 
][22]

进入 “**Build**” 标签页。 在 “**Add build step**”， 选择 “**Execute shell**” 选项。

在框中输入下列命令。

top -b -n 1 | head -n 5

点击 '**Save**'。

[
 ![Start a Jenkins Job](https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/15.png) 
][23]

现在你在 “Project checking system” 任务页。 点击 “**Build Now**” 运行任务 “checking system”。

After the job has been executed, you will see the '**Build History**', click on the first job to see the results.
任务运行完成后，你将看到 “**Build History**”， 点击第一个任务查看结果。

以下是 Jenkins运行任务的结果。

[
 ![Build and run a Jenkins Job](https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/16.png) 
][24]

Ubuntu 16.04 上基于 Apache web 服务器的 Jenkins 安装成功完成。

--------------------------------------------------------------------------------

via: https://www.howtoforge.com/tutorial/how-to-install-jenkins-with-apache-on-ubuntu-16-04/

作者：[Muhammad Arul ][a]
译者：[willcoderwang](http://wangzk.win)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [LCTT](https://github.com/LCTT/TranslateProject) 组织编译，[Linux中国](https://linux.cn/) 荣誉推出

[a]:https://twitter.com/howtoforgecom
[1]:https://www.howtoforge.com/tutorial/how-to-install-jenkins-with-apache-on-ubuntu-16-04/#prerequisite
[2]:https://www.howtoforge.com/tutorial/how-to-install-jenkins-with-apache-on-ubuntu-16-04/#step-install-java-openjdk-
[3]:https://www.howtoforge.com/tutorial/how-to-install-jenkins-with-apache-on-ubuntu-16-04/#step-install-jenkins
[4]:https://www.howtoforge.com/tutorial/how-to-install-jenkins-with-apache-on-ubuntu-16-04/#step-install-and-configure-apache-as-reverse-proxy-for-jenkins
[5]:https://www.howtoforge.com/tutorial/how-to-install-jenkins-with-apache-on-ubuntu-16-04/#step-configure-jenkins
[6]:https://www.howtoforge.com/tutorial/how-to-install-jenkins-with-apache-on-ubuntu-16-04/#step-jenkins-security
[7]:https://www.howtoforge.com/tutorial/how-to-install-jenkins-with-apache-on-ubuntu-16-04/#step-testing-a-simple-automation-job
[8]:https://www.howtoforge.com/tutorial/how-to-install-jenkins-with-apache-on-ubuntu-16-04/#reference
[9]:https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/big/1.png
[10]:https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/big/2.png
[11]:https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/big/3.png
[12]:https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/big/4.png
[13]:https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/big/5.png
[14]:https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/big/6.png
[15]:https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/big/7.png
[16]:https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/big/8.png
[17]:https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/big/9.png
[18]:https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/big/10.png
[19]:https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/big/11.png
[20]:https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/big/12.png
[21]:https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/big/13.png
[22]:https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/big/14.png
[23]:https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/big/15.png
[24]:https://www.howtoforge.com/images/how-to-install-jenkins-with-apache-on-ubuntu-16-04/big/16.png
