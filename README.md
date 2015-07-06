# tuotuo-multi-tenant #
[如何在 CentOS 7 上部署 Tuotuo 项目](./MONO.md)
[如何在 CentOS 7 上部署 Dianan 项目](./Dianna.Chat.md)
[如何变成把Tuotuo变成多宿主](./Multi.Zeus.Tuotuo.md)
[如何在 Centos7 上部署服务](./run-mono-service.md)
## 初步方案: ##
参考项目: [SaasKit](https://github.com/saaskit/saaskit)
### § Tuotuo.Multi.Tenant ###
    var tenant = Request.GetOwinContext().Get<TenantInstance>(Constants.OwinCurrentTenant);

## 另外一种可行方案: ##
Autofac Extras: Multitenant Application Support: [nuget](https://www.nuget.org/packages/Autofac.Extras.Multitenant/), [doc](http://autofac.readthedocs.org/en/latest/advanced/multitenant.html)
### § Tuotuo.Multi.Tenant.DI ###
当前进度: 已经实现

## 候选参考方案: ##
* [SIMPLE MULTITENANCY WITH ASP.NET MVC 4](http://www.dennisonpro.info/simple-multitenancy-with-asp-net-mvc-4/)

# 数据库连接原型 #

### § Tuotuo.Multi.Tenant.EF ###
采用 Entity Framework 的项目，可以使用 [Multi-Tenant With Code First](https://entityframework.codeplex.com/discussions/462765) 中提到的方式实现

1.Context.cs.t4
```C#
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    ...
    OnModelCreatingEx(modelBuilder);
```
2.IACenterEntitiesEx.cs
```C#
public partial class IACenterEntities : IDbModelCacheKeyProvider
{
    public static IACenterEntities Create(string connectionString, Guid tenantId)
    {
        return new IACenterEntities(connectionString, tenantId);
    }

    internal IACenterEntities(string connectionString, Guid tenantId)
        : base(connectionString)
    {
        Database.SetInitializer<IACenterEntities>(null);
        this.SchemaName = tenantId.ToString("N");
    }

    public string SchemaName { get; private set; }

    protected void OnModelCreatingEx(DbModelBuilder modelBuilder)
    {
        if (this.SchemaName != null)
        {
            modelBuilder.HasDefaultSchema(this.SchemaName);
        }
    }

    public string CacheKey
    {
        get { return this.SchemaName; }
    }
}
```
3.app.config
```xml
<entityFramework>
  <defaultConnectionFactory type="MySql.Data.Entity.MySqlConnectionFactory, MySql.Data.Entity.EF6">
  </defaultConnectionFactory>
```

### § MySQLTester ###
将 MySQL 作为单例运行，以便进行针对 MySQL 数据库的相关测试

## ORM ##
* [Dapper](https://github.com/StackExchange/dapper-dot-net)
* [Simple.Data](http://simplefx.org/simpledata/docs/index.html)

## Data Migration ##
* [FluentMigrator](https://github.com/schambers/fluentmigrator)

# 发送邮件 #

## Background ##
* [Hangfire](http://hangfire.io/) ([github](https://github.com/HangfireIO/Hangfire/))[**高级版本要钱**](http://hangfire.io/pricing/index.html)
* [Quartz.NET](http://www.quartz-scheduler.net/) ([github](https://github.com/quartznet/quartznet))
* [CrystalQuartz](https://github.com/guryanovev/CrystalQuartz)

# 权限认证 #
考虑采用 OAuth2/OpenID 的方式实现

## Thinktecture.IdentityServer3 ##
[Thinktecture IdentityServer3](https://identityserver.github.io/Documentation/) ([github](https://github.com/IdentityServer/Thinktecture.IdentityServer3))

## Thinktecture IdentityModel ##
[Thinktecture IdentityModel](https://github.com/IdentityModel/Thinktecture.IdentityModel)
