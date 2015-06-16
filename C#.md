##云api-SDK (C#)

###概述
-------

APICloud的 C# SDK 是基于WebClient和WebRequest构建. 它与APICloud开发的api进行交互,支持对象化操作，简化文件上传、多个文件上传、Relation相关操作,只需要在你的代码中做出一点点改变.
主要包括推送、统计、数据相关的操作，分别对应的命名空间为APICloud.Rest、APICloud.Push、APICloud.Analytics。



###新手上路

------

首先需要引入命名空间,需要使用那部分就引用各自的命名空间。

```C#
using APICloud.Rest;
using APICloud.Push;
using APICloud.Analytics;
```

然后需要配置进行调用的对象

```C#
var resouce = new Resource("appId", "appKey");
var push= new Push("appId", "appKey");
var Analytics = new Analytics("appId", "appKey");
```
最后就可以调用需要使用的方法，进行后续操作了。
```C#
//具体参数参考推送文档
//{
//  "title":"",
//  "content":"",
//  "type":1,
//  "platform":0
//}
var ret=push.Message(jsonStr);
```

```C#
//具体参数参考统计文档
var ret=Analytics.getAppStatisticDataById("2015-05-22", "2015-05-28");
```

```C#
//具体参数参考数据云文档
//modelName为class名称
var model=resouce.Factory(modelName);
model.Get()
```

###API
------
####数据api

**基本结构**
|-Resource
  |-SetHeader
  |-Batch
  |-Factory
    |-Get(+1 重载)
    |-Query(+1 重载)
    |-Create(+3 重载)
    |-Edit(+1 重载)
    |-Delete(+1 重载)
    |-Count(+2 重载)
    |-Exists
    |-FindOne(+1 重载)
    |-Login
    |-Logout(+1 重载)
    |-Verify
    |-Reset
    |-Upload(+4 重载)

#####**Resource**
------
Resource为整个SDK的入口，包括一些全局化使用的api，以及初始化请求的配置信息。

**参数**
- AppId(必填)-字符串类型,应用的id
- AppKey(必填)-字符串类型,应用的安全校验Key
- UrlBase(选填,方便调试使用)-字符串类型,接口请求地址

**示例**

```C#
var resouce = new Resource("A6986290476670", "8073AA0E-F22F-9B47-EBCB-8C64A127C298");
```

#####**SetHeader**
------
SetHeader为全局函数，即作用在所有Resource生成的对象上，主要是设置请求头部信息的。

**参数**
- key(必填)-字符串类型,请求头的键信息
- value(必填)-字符串类型,请求头的值信息

**返回值**
- 无

**示例**

```C#
//设置校验token，以方便通过ACL验证
resouce.SetHeader("Authorization", authorization);
```

#####**Batch**（+1重载）
------
Batch为全局函数，只能在Resource对象上调用，主要是为了减少网络交互的次数太多带来的时间浪费, 您可以在一个请求中对多个对象进行 create/update/delete 操作。

**参数**
- requests(必填)-批量请求的请求信息，类型参考如下
**requests类型**
- string
- List<Object>

**返回值**
- json数组的字符串

**示例**

```C#
//[
//  {
//    "method": "GET",
//    "path": "/mcm/api/file"
//  },
//  {
//    "method": "GET",
//    "path": "/mcm/api/file/count"
//  }
//]

var ret=resouce.Batch(jsonStr);
```

```C#
//List<Object>
var list =new List<Object>();
list.Add(new{
    method="GET",
    path="/mcm/api/file"
});
list.Add(new{
    method="GET",
    path = "/mcm/api/file/count"
});
var ret=resouce.Batch(list);
```

#####**Factory**
------
Factory为全局函数，只能在Resource对象上调用，主要是生成一个可进行后续增删改查操作的对象。

**参数**
- ClassName(必填)-字符串类型,对应数据云上的Class的名称

**返回值**
- Factory对象

**示例**

```C#
var model=resouce.Factory("modelName");
```

#####**Get(+1重载)**
------
Get为Factory为下的函数，主要是获取一个对象或获取对象下相关联的所有对象。

**参数**
- ObjectID(必填)-字符串类型,对应要获取对象的id
- Relation（选填)-字符串类型,对应Relation列的列名

**返回值**
- json对象字符串
- json数组字符串

**示例**

```C#
var ret=model.Get("5541d189490847810c2f8bc1");
```
```C#
var ret=model.Get("5541d189490847810c2f8bc1","books");
```

#####**Query(+1重载)**
------
Query为Factory为下的函数，主要是用于查询,具体参数请参考[数据云文档](http://docs.apicloud.com/%E4%BA%91API/data-cloud-api)。

**参数**
- Filter（选填)-字符串类型,具体参数请参考文档

**返回值**
- json数组字符串

**示例**

```C#
//返回所有,默认20条
var ret=model.Query();
```
```C#
//返回按条件查询的结果,默认20条
//{
//	"where":{
//		"id":"5541d189490847810c2f8bc1"
//	}
//}
var ret=model.Query(jsonStr);
```

#####**Create(+1重载)**
------
Create为Factory为下的函数，主要是用于向当前对象新增一条数据。

**参数**
- JsonDataStr（选填)-字符串类型,为要新增到数据库中的数据的json对象的字符串
- body（选填)-对象类型,为要新增到数据库中的数据的对象

**返回值**
- json对象字符串

**示例**

```C#
//{
//	"title":"APICloud"
//}
var ret=model.Create(jsonStr);
```
```C#
var ret=model.Create(new
{
    title = "APICloud"
});
```

#####**Create Relation(+1重载)**
------
Create为Factory为下的函数，主要是用于向当前对象相关联的对象下新增一条数据。

**参数**
- ObjectID(必填)-字符串类型,对应要获取对象的id
- Relation（选填)-字符串类型,对应Relation列的列名
- jsonDataStr（选填)-字符串类型,为要新增到数据库中的数据的json对象的字符串
- body（选填)-对象类型,为要新增到数据库中的数据的对象

**返回值**
- json对象字符串

**示例**

```C#
//{
//	"title":"APICloud"
//}
var ret=model.Create("5541d189490847810c2f8bc1","books",jsonStr);
```
```C#
var ret=model.Create("5541d189490847810c2f8bc1","books",new
{
    title = "APICloud"
});
```

#####**Edit(+1重载)**
------
Edit为Factory为下的函数，主要是用于更新给定的id的数据。

**参数**
- ObjectID(必填)-字符串类型,对应要获取对象的id
- jsonDataStr（选填)-字符串类型,为要更新到数据库中的数据的json对象的字符串
- body（选填)-对象类型,为要更新到数据库中的数据的对象

**返回值**
- json对象字符串

**示例**

```C#
//{
//	"title":"APICloud"
//}
var ret=model.Edit("5541d189490847810c2f8bc1",jsonStr);
```
```C#
var ret=model.Edit("5541d189490847810c2f8bc1",new
{
    title = "APICloud"
});
```

#####**Delete(+1重载)**
------
Delete为Factory下的函数，主要是用于删除给定id的数据或给定id的关联对象下的所有数据。

**参数**
- ObjectID(必填)-字符串类型,对应要获取对象的id
- Relation（选填)-字符串类型,对应Relation列的列名

**返回值**
- json对象字符串

**示例**

```C#
var ret=model.Delete("5541d189490847810c2f8bc1");
```
```C#
var ret=model.Delete("5541d189490847810c2f8bc1","books");
```

#####**Count(+1重载)**
------
Count为Factory下的函数，主要是用于统计数据条数或统计关联对象下的数据条数。

**参数**
- ObjectID(必填)-字符串类型,对应要获取对象的id
- Relation（选填)-字符串类型,对应Relation列的列名

**返回值**
- json对象字符串

**示例**

```C#
var ret=model.Count();
```
```C#
var ret=model.Count("5541d189490847810c2f8bc1","books");
```

#####**Count Relation**
------
Count为Factory下的函数，主要是用于统计数据条数或统计关联对象下的数据条数。

**参数**
- Filter（选填)-字符串类型,具体参数请参考[文档](http://docs.apicloud.com/%E4%BA%91API/data-cloud-api#6)

**返回值**
- json对象字符串

**示例**

```C#
//{
//	"where":{
//		"title":"APICloud"
//	}
//}
var ret=model.Count(jsonStr);
```


#####**Exists(+1重载)**
------
Exists为Factory下的函数，主要用于判断给定id的数据是否存在。

**参数**
- ObjectID(必填)-字符串类型,对应要获取对象的id

**返回值**
- json对象字符串

**示例**


```C#
var ret=model.Exists("5541d189490847810c2f8bc1");
```


#####**FindOne(+1重载)**
------
FindOne为Factory为下的函数，主要是用于查询返回第一条数据。

**参数**
- Filter（选填)-字符串类型,具体参数请参考文档

**返回值**
- json数组字符串

**示例**

```C#
var ret=model.FindOne();
```
```C#
//{
//	"where":{
//		"title":"APICloud"
//	}
//}
var ret=model.FindOne(jsonStr);
```


#####**Login**
------
Login为Factory下的函数，主要用于用户登录操作。

**参数**
- username(必填)-字符串类型,用户名
- password(必填)-字符串类型,密码

**返回值**
- json对象字符串

**示例**


```C#
var ret=model.Login("APICloud","111111");
```

#####**Logout(+1重载)**
------
Logout为Factory下的函数，主要用于用户注销操作。

**参数**
- token(选填)-字符串类型,登录完成后返回的token

**返回值**
- json对象字符串

**示例**


```C#
//如果用SetHeader方法设置过Authorization的请求头，则可直接注销
var ret=model.Logout();
```
```C#
var ret=model.Logout("token");
```

#####**Verify**
------
Verify为Factory下的函数，主要用于用户邮箱验证操作。

**参数**
- email(必填)-字符串类型,邮箱地址
- language(必填)-字符串类型,所用语言("zh-CN","en-US")
- username(必填)-字符串类型,用户名 

**返回值**
- json对象字符串

**示例**


```C#
var ret=model.Verify("APICloud@APICloud.com","zh-CN","APICloud");
```

#####**Reset**
------
Reset为Factory下的函数，主要用于用户忘记密码操作。

**参数**
- email(必填)-字符串类型,邮箱地址
- language(必填)-字符串类型,所用语言("zh-CN","en-US")
- username(必填)-字符串类型,用户名 

**返回值**
- json对象字符串

**示例**


```C#
var ret=model.Reset("APICloud@APICloud.com","zh-CN","APICloud");
```


#####**Upload(+1重载)**
------
Upload为Factory下的函数，主要用于上传文件相关操作。

**参数**
- stream(必填)-FileStream类型,文件对象
- map(选填)-Dictionary<string, string>类型,file表的自定义字段，如非string字段，需要JSON.stringify()处理一下

**返回值**
- json对象字符串

**示例**


```C#
var file = Factory("file");
using (var fs = new FileStream("file path", FileMode.Open))
{
    var ret = file.Upload(fs);
}
```

```C#
var dict=new Dictionary<string, string>();
dict["title"]="APICloud";

var file = Factory("file");
using (var fs = new FileStream("file path", FileMode.Open))
{
    var ret = file.Upload(fs,dict);
}
```

#####**Upload Relation(+1重载)**
------
Upload为Factory下的函数，主要用于file表作为Relation对象表的上传文件相关操作。

**参数**
- ObjectID(必填)-字符串类型,对应要获取对象的id
- Relation（选填)-字符串类型,对应Relation列的列名
- stream(必填)-FileStream类型,文件对象
- map(选填)-Dictionary<string, string>类型,file表的自定义字段，如非string字段，需要JSON.stringify()处理一下

**返回值**
- json对象字符串

**示例**


```C#
var file = Factory("file");
using (var fs = new FileStream("5541d189490847810c2f8bc1","files","file path", FileMode.Open))
{
    var ret = file.Upload(fs);
}
```

```C#
var dict=new Dictionary<string, string>();
dict["title"]="APICloud";

var file = Factory("file");
using (var fs = new FileStream("5541d189490847810c2f8bc1","files","file path", FileMode.Open))
{
    var ret = file.Upload(fs,dict);
}
```

#####**File字段上传示例**

```C#
using (var fs1 = new FileStream("file path", FileMode.Open))
{
    using (var fs2 = new FileStream("file path", FileMode.Open))
    {
        var ret = model.Create(new
        {
            file1 = fs1,
            file2 = fs2
        });
    }
}

```

####推送api

**基本结构**
|-Push
  |-Message(+1 重载)

#####**Message**
------
Message为Push下的函数，主要是是用来推送信息。

**参数**
- jsonStr(选填)-字符串类型,推送消息的json对象的字符串，具体参数参考[文档](http://docs.apicloud.com/%E4%BA%91API/push-cloud-api)
- body(选填)-Object类型,推送消息对象

**返回值**
- json对象字符串


**示例**

```C#
//{
//	"title":"APICloud",
//	"content":"",
//	"type":1,
//	"platform":0
//}
var ret=push.Message(jsonStr);
```
```C#
var pararm = new
{
    title = "test",
    content = "",
    type = 1,
    platform = 0
};
ret = push.Message(pararm);
```

####统计api

**基本结构**
|-Analytics
  |-getAppStatisticDataById
  |-getVersionsStatisticDataById
  |-getGeoStatisticDataById
  |-getDeviceStatisticDataById
  |-getExceptionsStatisticDataById
  |-getExceptionsDetailByTitle

#####**getAppStatisticDataById**
------
getAppStatisticDataById为Analytics下的函数，该接口主要用于获取用户指定应用ID及时间范围内的相关应用统计数据信息

**参数**
- startDate(必填)-字符串类型,开始日期
- endDate(必填)-字符串类型,结束日期

**返回值**
- json对象字符串


**示例**

```C#
var ret=Analytics.getAppStatisticDataById("2015-05-22", "2015-05-28");
```

#####**getVersionsStatisticDataById**
------
getVersionsStatisticDataById为Analytics下的函数，该接口主要用于获取用户指定应用ID及时间范围内相关应用各版本的统计数据信息

**参数**
- startDate(必填)-字符串类型,开始日期
- endDate(必填)-字符串类型,结束日期

**返回值**
- json对象字符串


**示例**

```C#
var ret=Analytics.getVersionsStatisticDataById("2015-05-22", "2015-05-28");
```

#####**getGeoStatisticDataById**
------
getGeoStatisticDataById为Analytics下的函数，该接口主要用于获取用户指定应用ID及时间范围内的应用下各版本地理分布统计数据信息

**参数**
- startDate(必填)-字符串类型,开始日期
- endDate(必填)-字符串类型,结束日期
- versionCode(必填)-字符串类型,版本号

**返回值**
- json对象字符串


**示例**

```C#
var ret=Analytics.getGeoStatisticDataById("2015-05-22", "2015-05-28"，"");
```

#####**getDeviceStatisticDataById**
------
getDeviceStatisticDataById为Analytics下的函数，该接口主要用于获取用户指定应用ID及时间范围内的应用下各版本设备信息分布统计数据信息

**参数**
- startDate(必填)-字符串类型,开始日期
- endDate(必填)-字符串类型,结束日期

**返回值**
- json对象字符串


**示例**

```C#
var ret=Analytics.getDeviceStatisticDataById("2015-05-22", "2015-05-28");
```

#####**getExceptionsStatisticDataById**
------
getExceptionsStatisticDataById为Analytics下的函数，该接口主要用于获取用户指定应用ID及时间范围内的应用下各版本异常错误统计数据信息

**参数**
- startDate(必填)-字符串类型,开始日期
- endDate(必填)-字符串类型,结束日期

**返回值**
- json对象字符串


**示例**

```C#
var ret=Analytics.getExceptionsStatisticDataById("2015-05-22", "2015-05-28");
```

#####**getExceptionsDetailByTitle**
------
getExceptionsDetailByTitle为Analytics下的函数，该接口主要用于根据应用异常错误摘要获取异常错误详细信息

**参数**
- title(必填)-字符串类型,异常错误摘要

**返回值**
- json对象字符串


**示例**

```C#
var ret=Analytics.getExceptionsDetailByTitle("2015-05-22", "2015-05-28");
```
