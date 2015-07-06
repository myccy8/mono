# 如何在 CentOS 7 上部署 Dianna 项目 #
该项目正在 monobox (192.168.1.188) 上进行验证 [.](#mintcodediannaserverbus)

## 部署狄安娜项目 ##
#### Mintcode.Dianna.Chat ###
1.StackExchange.Redis 需要用mono bulid下
```Bash
xbuild ~/StackExchange.Redis/*.sln
```
2.StackExchange.Redis.RedisKey类的ToString方法无效 
*Invalid IL code in StackExchange.Redis.RedisKey:ToString (): IL_0007: brfalse.s IL_0019*

```csharp
 return (string)(this) ?? "(null)";
```
改成
```csharp
return   (!this.IsNull) ? (string)this : "(null)";
```

3.Failed to find or load the registered .Net Framework Data Provider 'MySql.Data.MySqlClient'.
添加Mysql.Data.dll并且在web.config添加
```xml
  <system.data>
    <DbProviderFactories>
      <remove invariant="MySql.Data.MySqlClient" />
      <add name="MySQL Data Provider" 
           invariant="MySql.Data.MySqlClient" 
           description=".Net Framework Data Provider for MySQL" 
      	   type="MySql.Data.MySqlClient.MySqlClientFactory, MySql.Data, Version=6.9.5.0, Culture=neutral, PublicKeyToken=c5687fc88969c44d" />
    </DbProviderFactories>
  </system.data>
 ```

##### 配置信息 #####
```Bash
sudo vim /etc/nginx/conf.d/mintcode.dianna.chat.conf
```
```Nginx
server {
   listen          80;
   server_name     mintcode.dianna.chat;
   access_log      /var/log/nginx/mintcode.dianna.chat.access.log;
   error_log       /var/log/nginx/mintcode.dianna.chat.error.log;
   charset		   utf-8;
   location / {
       proxy_pass  http://192.168.1.175:912;
       proxy_set_header Host $host;
       proxy_redirect off;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   }

   location ~*^/chat(.*) {
       root            /home/mintcode/Mintcode.Dianna.Chat/Mintcode.Dianna.Chat;
       rewrite (?i)/chat/?(.*)$ /$1 break;
       fastcgi_pass    127.0.0.1:9001;
       include         /etc/nginx/fastcgi_params;
   }
}

```
```Bash
fastcgi-mono-server4 \
--applications=mintcode.dianna.chat:/:/home/mintcode/Mintcode.Dianna.Chat/Mintcode.Dianna.Chat/ \
--socket="tcp:127.0.0.1:9001" \
--printlog
```

#### Mintcode.Dianna.ServerBus ###

1.错误信息
```Bash
Default Error: 0 : The service terminated abnormally Topshelf.HostConfigurationException: The service was not properly configured:
[Failure] Command Line An unknown command-line option was found: ARGUMENT: "
[Success] Name TuotuoRabbitCenter
[Success] Description Redis接收RabbitMq数据的服务
[Success] ServiceName TuotuoRabbitCenter
  at Topshelf.Configurators.ValidateConfigurationResult.CompileResults (IEnumerable`1 results) [0x00000] in <filename unknown>:0
  at Topshelf.HostFactory.New (System.Action`1 configureCallback) [0x00000] in <filename unknown>:0
  at Topshelf.HostFactory.Run (System.Action`1 configureCallback) [0x00000] in <filename unknown>:0
```
下载Topself.Linux的源码 注释LinuxConfigurationExtensions类中的
```csharp
configurator.ApplyCommandLine(MonoHelper.GetUnparsedCommandLine())
```
2.Mysql和Redis错误参考上面

##### 配置信息 #####
```Bash
mono \
/home/mintcode/Mintcode.Dianna.Chat/Service\ Bus/Mintcode.Dianna.ServiceBus/bin/Debug/Mintcode.Dianna.ServiceBus.exe \
```