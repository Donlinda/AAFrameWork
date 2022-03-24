AA.Dapper用法
实体映射配置
使用DommelEntityMap类映射属性名称。创建一个派生类，在构造函数中设置Map方法，可指定某个属性映射到数据库列名

   public class UserInfoMap : DommelEntityMap<UserInfo>
    {
        public UserInfoMap()
        {
            ToTable("Sys_UserInfo");//映射具体的表名
            Map(p => p.SysNo).IsKey();//指定主键 ,IsIdentity是否自增
            Map(p=>p.GmtCreate).ToColumn("GmtCreateDate"); //属性名和数据库列名 可以不同
            Map(p=>p.LastLoginDate).Ignore();//一些计算属性，可以忽略不需要跟数据库列进行映射
        }
    }
使用MapConfiguration.Init方法，把映射类初始化，后续就可以使用了

 public static void InitMapCfgs()
        {
            var fluentMapconfig = new List<Action<FluentMapConfiguration>>();
            fluentMapconfig.Add(cfg =>
            {
                cfg.AddMap(new UserInfoMap());
            });
            MapConfiguration.Init(fluentMapconfig);
        }
开始使用AA.Dapper
使用DapperContext设置数据库连接和数据库类型是sqlserver还是mysql
    public class AADapperContext : DapperContext
    {
        public AADapperContext() : base(new NameValueCollection()
        {
            ["aa.dataSource.AaCenter.connectionString"] = "Data Source =.; Initial Catalog = AaCenter;User ID = sa; Password = 123;",
            ["aa.dataSource.AaCenter.provider"] = "SqlServer"
        })
        { }
    }
仓储包含了大部分的操作，同时支持Async操作
IDapperRepository<UserInfo> userInfoRepository = new DapperRepository<UserInfo>();
插入实体
 var user = new UserInfo()
            {
                UserName = "chengTian",
                RealName = "成天",
                GmtCreate = DateTime.Now,
                GmtModified = DateTime.Now
            };
  var result = userInfoRepository.Insert(user);
修改实体
 var user = userInfoRepository.Get(1);
            user.GmtModified = DateTime.Now;
 var result = userInfoRepository.Update(user);
获取实体
   var user = userInfoRepository.Get(1);//By 主键
   var users = userInfoRepository.GetAll();//所有
   var users = userInfoRepository.Select(p => p.UserName == "chengTian");//谓词
删除实体
   var user = userInfoRepository.Get(1);
   var result = userInfoRepository.Delete(user);
分页
var result = _userRepository.From(sql =>  
            {  
                sql.Select()  
                   .Where(x=>x.UserName==input.UserName)  
                   .Page(input.PageIndex, input.PageSize);  
            });  
事务
using (var dbTransaction = dapperContext.BeginTransaction())  
 {  
     try  
     {  
         var user = new UserInfo()  
         {  
             UserName = "chengTian",  
             RealName = "成天",  
             GmtCreate = DateTime.Now,  
             GmtModified = DateTime.Now,  
             LastLoginDate = DateTime.Now  
         };  
         userInfoRepository.Insert(user);  
         userAlbumRepository.Insert(new UserAlbum  
         {  
             Pic = "image/one.jgp"  
         });  
         dapperContext.Commit();  
     }  
     catch (Exception ex)  
     {  
         dbTransaction.Rollback();  
     }  
 }  
 
动态Expression
var where = DynamicWhereExpression.Init<User>();  
if (input.RealName != "")  
{  
   where = where.And(x => x.RealName.Contains(input.RealName));  
}  
if (input.SysNo!=Guid.Empty)   
{  
    where = where.And(x=>x.SysNo==input.SysNo);  
}  
if (input.SysNos.Any())   
{  
    where = where.And(x=>input.SysNos.Contains(x.SysNo));  
}  
var result = _userRepository.From(sql =>  
{  
    sql.Select()  
       .Where(where)  
       .Page(input.PageIndex, input.PageSize);  
});  
