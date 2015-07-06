# 如何在 CentOS 7 上部署 Tuotuo 项目 #
该项目正在 monobox (192.168.1.188) 上进行验证 [.](#E89Ad5Xk)

## 相关配置 ##
### 配置 CentOS 7 ###
安装基本的工具
```Bash
sudo yum install vim rsync -y
```
添加 [EPEL](https://fedoraproject.org/wiki/EPEL) 源
```Bash
sudo yum install epel-release -y
yum repolist
```

### 安装 Mono ###
#### 添加 Mono 源 ####
```Bash
sudo yum install yum-utils -y
rpm --import "http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF"
yum-config-manager --add-repo http://download.mono-project.com/repo/centos/
yum repolist
```

#### 安装 Mono ####
```Bash
sudo yum install mono-complete -y
sudo mkdir -p /etc/mono/registry/LocalMachine
```

###### 参考资料 ######
1. http://www.mono-project.com/docs/getting-started/install/linux/#centos-fedora-and-derivatives

### 安装 Nginx ###
```Bash
sudo yum install nginx -y
```

#### 配置防火墙 ####
方案1：使用 `firewall-cmd`
```Bash
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-port=8080/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --zone=public --list-all
```
方案2：编辑相关文件
```Bash
sudo vim /etc/firewalld/zones/public.xml
sudo firewall-cmd --reload
```

#### 验证 Nginx ####
```Bash
sudo systemctl start nginx
```
使用浏览器访问 `http://hostname:80` ，可以看到 nginx 默认的欢迎页面

#### 配置 Nginx ####
```Bash
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.org
sudo vim /etc/nginx/nginx.conf
```

###### 参考资料 ######
1. Mono: http://www.mono-project.com/docs/web/fastcgi/nginx/
2. Nginx: http://nginx.org/en/docs/beginners_guide.html
3. [*How To Install Nginx on CentOS 7*](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-centos-7)

### 安装 FastCGI ###
```Bash
sudo yum install xsp -y
```

#### 配置 FastCGI ####
```Bash
sudo vim /etc/nginx/fastcgi_params
```
```Ini
# Mono, http://www.mono-project.com/docs/web/fastcgi/nginx/
fastcgi_param  PATH_INFO          "";
fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
```

###### 参考资料 ######
1. http://www.mono-project.com/docs/web/fastcgi/

### 安装 MariaDB ###
```Bash
sudo yum install mariadb-server -y
sudo systemctl start mariadb
```
```Bash
mysql_secure_installation
```
```Bash
Enter current password for root (enter for none): press Enter
Set root password? [Y/n]                          y
New password:                                     p@ssw0rd
Re-enter new password:                            p@ssw0rd
Remove anonymous users? [Y/n]                     y
Disallow root login remotely? [Y/n]               n
Remove test database and access to it? [Y/n]      y
Reload privilege tables now? [Y/n]                y
```
```Bash
sudo systemctl enable mariadb
```

### *安装 MariaDB* ###
添加 MariaDB 源 ([link](https://downloads.mariadb.org/mariadb/repositories/#mirror=neusoft&distro=CentOS&distro_release=centos7-amd64--centos7&version=10.0))
```Bash
sudo vim /etc/yum.repos.d/MariaDB.repo
```
```Ini
# MariaDB 10.0 CentOS repository list - created 2015-02-12 07:25 UTC
# http://mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.0/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```
安装 MariaDB
```Bash
yum repolist
sudo yum install MariaDB-server -y
```

###### 参考资料 ######
1. [Installing MariaDB with yum](https://mariadb.com/kb/en/mariadb/yum/)

### *安装 MySQL* ###
*因不明原因搁浅*  
添加 MySQL 源
```Bash
curl -O http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
sudo yum install -y mysql-community-release-el7-5.noarch.rpm
yum repolist
```
安装 MySQL Server
```Bash
sudo yum install -y mysql-community-server
```
SELinux
```Bash
sudo grep -i 'mysql' /var/log/audit/audit.log
sudo semanage permissive -a mysqld_t
```
启动 MySQL 进程
```Bash
sudo systemctl start mysqld
```
初始化 MySQL
```Bash
mysql_secure_installation
```
```Ini
Enter current password for root (enter for none): <enter>
```

###### 参考资料 ######
1. [Download MySQL Yum Repository](http://dev.mysql.com/downloads/repo/yum/)
2. http://linux.die.net/man/8/mysqld_selinux

## 部署拓拓项目 ##
### 注意事项 ###
1. 请在 Visual Studio 的项目中，选择 "启用 NuGet 程序包还原"
2. 请务必使用 NuGet 来管理项目所需的程序外部依赖
3. 鉴于当前单元测试框架 MSTest 并不支持 NuGet，请尽量避免使用
4. System.Web.UnvalidatedRequestValuesBase 未在 Mono 中实现 ([link](http://go-mono.com/status/status.aspx?reference=4.5&profile=4.5&assembly=System.Web))
5. System.Web 中的 System.Web.HttpApplication.RegisterModule 方法未在 Mono 中实现  
导致 Microsoft.Owin.Host.SystemWeb 报告 "Missing method..." ([link](https://bugzilla.xamarin.com/show_bug.cgi?id=21810))

### 共通点 ###
1.把源代码迁入 Linux
由于存在权限等问题，可以采用压缩包的方式
```Bash
tar czf <project_name>.tar.gz /path/to/project --exclude='packages' --exclude='bin' --exclude='obj'
rsync -ze ssh <project_name>.tar.gz <username>@<hostname>:~/
```
```Bash
tar xzf <project_name>.tar.gz
sudo find /path/to/project -type d -exec chmod 755 {} +
find /path/to/project -type f -exec chmod 644 {} +  
```
2.使用 xbuild 进行编译
如果项目目录中没有 `*.msbuildproj`，可以把 [build.msbuildproj](./build.msbuildproj) 复制到项目目录中
```Bash
xbuild /path/to/project/build.msbuildproj
```
编译时可以使用参数，如：
```Bash
xbuild /path/to/project/build.msbuildproj /t:RestorePackages /p:Configuration=Debug /p:Platform="Any CPU"
```
###### 参考资料 ######
1. [Cross Platform Builds on Windows and Mono with MSBuild](https://peteris.rocks/blog/cross-platform-builds-on-windows-and-mono-with-msbuild/)

***
本片断可以被上述 `build.msbuildproj` 替代，此处仅作为历史资料保留
```Bash
mono /path/to/project/.nuget/NuGet.exe restore /path/to/project/<project_name>.sln
xbuild /path/to/project/<project_name>.sln
```
编译时可以使用参数，如：
```Bash
xbuild /path/to/project/<project_name>.sln /p:Configuration=Release /t:Rebuild;Package
```
***
3.配置 nginx
```Bash
sudo vim /etc/nginx/conf.d/<project_name>.conf
sudo systemctl reload nginx
```
4.反编译 (可用于排错) ([link](http://www.mono-project.com/docs/tools+libraries/tools/monodis/))
```Bash
monodis /path/to/project/assembly_name
```

##### SELinux #####
```Bash
sudo less /var/log/audit/audit.log
> type=AVC msg=audit(1423605678.278:2384): avc:  denied  { name_connect } for  pid=22637 comm="nginx" dest=8080 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:http_cache_port_t:s0 tclass=tcp_socket

sudo yum install policycoreutils-python -y
sudo semanage permissive -a httpd_t
```

###### 参考资料 ######
1. http://nginx.com/blog/nginx-se-linux-changes-upgrading-rhel-6-6/
2. http://www.if-not-true-then-false.com/2011/install-nginx-php-fpm-on-fedora-centos-red-hat-rhel/

### 配置示例 ###
#### xjhero ####
```Bash
sudo vim /etc/nginx/conf.d/xjhero.conf
```
```Nginx
server {
        listen          80;
        server_name     monobox;
        access_log      /var/log/nginx/xjhero.access.log;

        location / {
                root            /var/www/xjhero/;
                fastcgi_pass    127.0.0.1:8080;
                include         /etc/nginx/fastcgi_params;
        }
}
```
```Bash
fastcgi-mono-server4 \
--applications=monobox:/:/var/www/xjhero/ \
--socket="tcp:127.0.0.1:8080" \
--printlog
```

##### Troubleshooting #####
**Q:** Could not locate Razor Host Factory type: System.Web.Mvc.MvcWebRazorHostFactory, System.Web.Mvc ...  
**A:** ~/Views/Web.config 中 system.web.webPages.razor/host 的版本号需要和所使用的 System.Web.Mvc 版本一致 ([link](http://stackoverflow.com/a/26128304))
```xml
<host factoryType="System.Web.Mvc.MvcWebRazorHostFactory, System.Web.Mvc, Version=5.2.3.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
```

### 特殊情况 ###
#### Zeus.Tuotuo.IACenter ###
1.生成项目时出错，尝试以下命令：
```Bash
xbuild ~/Zeus.Tuotuo.IACenter/Zeus.Tuotuo.IACenter.Model/*.csproj
xbuild ~/Zeus.Tuotuo.IACenter/Zeus.Tuotuo.IACenter.Common/*.csproj
xbuild ~/Zeus.Tuotuo.IACenter/Zeus.Tuotuo.IACenter.Web/*.csproj

xbuild ~/Zeus.Tuotuo.IACenter/*.sln
```
2.因为 MVC 5 对 Mono 的支持差，所以降为 MVC 4，注意修改 Web.config 中的版本号 ([link](http://stackoverflow.com/a/24629599))
```PowerShell
PM> Install-Package Microsoft.AspNet.Mvc -Version 4.0.40804
```
3.因为 Linux 对文件路径大小写敏感，所以加入了一个 [CaseInsensitiveViewEngine.cs](./CaseInsensitiveViewEngine.cs)  ([link](http://hugoware.net/blog/ignoring-case-with-mono-mvc))  
在 Global.asax 中加入：
```C#
CaseInsensitiveViewEngine.Register(ViewEngines.Engines);
```
<a id="E89Ad5Xk"></a>4.运行时提示错误：Unable to load the specified metadata resource.  
原因：IACenter.edmx 文件-属性-元数据项目处理：嵌入输出程序集中，xbuild 在生成 dll 时，未将 IACenter.csdl/msl/ssdl 嵌入到 dll 中  
解决方案：可以考虑换用 [Code First][] 模式

##### 配置信息 #####
```Bash
sudo vim /etc/nginx/conf.d/zeus.tuotuo.iacenter.conf
```
```Nginx
server {
        listen          80;
        server_name     zeus.tuotuo.iacenter;
        access_log      /var/log/nginx/zeus.tuotuo.iacenter.access.log;

        location / {
                root            /home/mintcode/Zeus.Tuotuo.IACenter/Zeus.Tuotuo.IACenter.Web/;
                fastcgi_pass    127.0.0.1:8084;
                include         /etc/nginx/fastcgi_params;
        }
}
```
```Bash
fastcgi-mono-server4 \
--applications=zeus.tuotuo.iacenter:/:/home/mintcode/Zeus.Tuotuo.IACenter/Zeus.Tuotuo.IACenter.Web/ \
--socket="tcp:127.0.0.1:8084" \
--printlog
```

#### Zeus.Tuotuo ###
1. Mono 不支持 Microsoft.VisualStudio.TestTools.UnitTesting
2. Linux 不支持 xcopy

~~故此在解决方案设置了 Mono-Debug 配置（复制自 Debug），去除了所有的测试项目~~
故此，在 Release 配置中去除了所有的测试项目
```Bash
xbuild Zeus.Tuotuo/build.msbuildproj
```

##### 配置信息 #####
```Bash
sudo vim /etc/nginx/conf.d/zeus.tuotuo.conf
```
```Nginx
server {
        listen          80;
        server_name     zeus.tuotuo;
        access_log      /var/log/nginx/zeus.tuotuo.access.log;

        location / {
                root            /home/mintcode/Zeus.Tuotuo/Zeus.Tuotuo.Web/;
                fastcgi_pass    127.0.0.1:8085;
                include         /etc/nginx/fastcgi_params;
        }
}
```
```Bash
fastcgi-mono-server4 \
--applications=zeus.tuotuo:/:/home/mintcode/Zeus.Tuotuo/Zeus.Tuotuo.Web/ \
--socket="tcp:127.0.0.1:8085" \
--printlog
```

###### 参考资料 ######
1. Stack Overflow: [Exclude .csproj from VisualStudio 2012 .sln when using Mono Compiler](http://stackoverflow.com/a/18148043)

## 王的宝藏 ##
### Code First ###
[Code First]: #code-first
如何实现多站点，请参考原型项目 [Tuotuo.Multi.Tenant.EF](./Tuotuo.Multi.Tenant.EF) ([link](https://entityframework.codeplex.com/discussions/462765))

#### 方案 1 ####
安装 [EF Power Tools](https://visualstudiogallery.msdn.microsoft.com/72a60b14-1581-4b9b-89f2-846072eff19d/) 生成 Model ([link](https://msdn.microsoft.com/en-us/data/jj593170)) ([snapshot](./images/screencapture-msdn-microsoft-com-en-us-data-jj593170.png))  

安装成功后，在项目的右键菜单中会出现一个 `Entity Framework`  
![项目右键](./images/ef-pt-menu.png)

选择 `Reverse Engineer Code First` 并设置数据库连接  
![EF 配置](./images/ef-pt-conf-conn.png)

在项目中会自动添加 `EntityFramework` 的依赖，并会生成对应的代码  
![EF 生成](./images/ef-pt-codes.png)

如果需要修改生成的模型，请选择 `CustomizeReverse Engineer Templates`  
![EF 模板](./images/ef-pt-template.png)

#### 方案 2 ####
**注意**：该方案在 `Zeus.Tuotuo.IACenter` 中实践时，无法正确地解析 `T_IA_USERLOGIN_LOG` 表的主键的自增长属性，经过摸索未能解决该问题。

使用前请安装 `EntityFramework` 的依赖
```PowerShell
PM> Install-Package EntityFramework
```
采用 [EntityFramework.CodeTemplates.CSharp](http://www.nuget.org/packages/EntityFramework.CodeTemplates.CSharp) 生成 Model ([link](https://msdn.microsoft.com/en-gb/data/dn753860))  
1.Add the default templates to your project
```PowerShell
PM> Install-Package EntityFramework.CodeTemplates.CSharp
```
2.Customize the templates  
3.Run the reverse engineer process ([link](https://msdn.microsoft.com/en-gb/data/jj200620))  
![添加-新建项](./images/ef-add-file.png)  
![来自数据库的 Code First](./images/ef-wizard-code-first.png)

#### 备注 ####
如果使用 Code First 模式，可以考虑安装 EFInteractiveViews ([link](http://blog.3d-logic.com/2013/12/14/using-pre-generated-views-without-having-to-pre-generate-views-ef6/))
>To be able to work with different databases Entity Framework abstracts the store as a set of views which are later used to create queries and CUD (Create/Update/Delete) commands. EF generates views the first time it needs them (typically on the first query). For bigger or more complicated models view generation may take significant time and negatively impact the startup time of the application.

```PowerShell
PM> Install-Package EFInteractiveViews
```
在 Global.asax 中加入：
```C#
using (var ctx = new IACenterEntities())
{
    var viewPath = HostingEnvironment.MapPath("~/bin/ef_iacenter_views.xml");

    InteractiveViews.SetViewCacheFactory(ctx, new FileViewCacheFactory(viewPath));
}
```

### 开源项目 ###
1. [SlowMonkey](https://github.com/LHCGreg/SlowMonkey): app.config transformation that works on Mono
2. [MiniProfiler](http://miniprofiler.com/): A simple but effective mini-profiler for .NET and Ruby.

### 参考资料 ###
#### Mono ####
1. ASP.NET: http://www.mono-project.com/docs/web/aspnet/
2. FAQ: ASP.NET：http://www.mono-project.com/docs/faq/aspnet/
3. Application Portability: http://www.mono-project.com/docs/getting-started/application-portability/

#### Nginx ####
1. Nginx Pitfalls: http://wiki.nginx.org/Pitfalls
