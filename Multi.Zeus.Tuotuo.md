# 如何变成把Tuotuo变成多宿主#

需要升级全部的Json.net **因为tuotuo用的json.net版本比较老**
```shell
PM> update-package Newtonsoft.Json
```

1.	`Mintcode.Public.Data`项目 添加 `Mintcode.Dianna.Outland` 的 `Mintcode.Dianna.Multitenant.Service` 项目引用

2.	修改Mintcode.Public.Data.DbBase类的构造函数
```csharp
public DbBase(string connectionStringName)
{
   //从配置文件里获取连接字符串
   var connStr = ConfigurationManager.ConnectionStrings[connectionStringName].ConnectionString;
   if (!string.IsNullOrEmpty(ConfigurationManager.ConnectionStrings[connectionStringName].ProviderName))
      _ProviderName = ConfigurationManager.ConnectionStrings[connectionStringName].ProviderName;
   else
      throw new Exception("ConnectionStrings中没有配置提供程序ProviderName！");
   //获取数据库的类库
   _DbFactory = DbProviderFactories.GetFactory(_ProviderName);
   _BbConnecttion = _DbFactory.CreateConnection();
   //修改为对应宿主的连接字符串

+   var hostname = System.Web.HttpContext.Current.Request.Url.Host;
+   IMultitenantService svc = new Mintcode.Dianna.Multitenant.Service.MultitenantService().GetAllTenantInfo();
+   var tenant = svc.FirstOrDefault(t => t.Hostnames != null && t.Hostnames.Contains(hostname));

+   _BbConnecttion.ConnectionString = (tenant ??
+       new Mintcode.Dianna.Multitenant.Service.Models.TenantInfo()
+       {
+           ConnectionString = connStr
+       }).ConnectionString;
   //_BbConnecttion.Open();
   //获取数据源名称
   SetParamPrefix();
}
```

3. 因为存在`Task.Factory.StartNew` 跨线程调用，`HttpContext`会失效
所以把
```csharp
new bll() 
```
提取到Task.Factory.StartNew 外面


