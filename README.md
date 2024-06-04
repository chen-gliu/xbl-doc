# 使用说明文档
## 要求
JDK17+  
推荐使用IDEA 2021.1.3/Eclipse 2021以上版本

## 快速开始
### 添加依赖
按需引入相应的依赖包。
```xml
<parent>
    <artifactId>xbl-all</artifactId>
    <groupId>com.super.xiaobailong</groupId>
    <version>5.0.8.M</version>
</parent>
```
   ```xml
   <dependencies>
       <dependency>
           <groupId>com.super.xiaobailong</groupId>
           <artifactId>xbl.id</artifactId>
           <scope>compile</scope>
       </dependency>
   
       <dependency>
           <groupId>com.super.xiaobailong</groupId>
           <artifactId>xbl.webserver</artifactId>
           <scope>compile</scope>
       </dependency>
   
       <dependency>
           <groupId>com.super.xiaobailong</groupId>
           <artifactId>xbl.jdbc</artifactId>
           <scope>compile</scope>
       </dependency>
   
       <dependency>
           <groupId>org.postgresql</groupId>
           <artifactId>postgresql</artifactId>
       </dependency>
       <dependency>
            <groupId>com.super.xiaobailong</groupId>
            <artifactId>xbl.sso</artifactId>
       </dependency>
   </dependencies>
   ```
### 声明对外接口
```java
public RouterFunction<ServerResponse> orderHandlerRouter() {
        return RouterFunctions.route()
                .POST("/order/insert", orderHandler::insert)
                .GET("/order/findOrderById", orderHandler::findOrderById)
                .build();
    }
```
### 创建服务
```java
public static void main(String[] args) {
        MyRouterFunctions myRouterFunction = new MyRouterFunctions();
        HttpServer.create()
                //添加请求接口
                .mapping(myRouterFunction.myRouterFunction())
                //启动服务，绑定端口
                .bind();
    }
```
默认配置下Web应用的端口为80。

### Get请求
```java
public ServerResponse findOrderById(ServerRequest serverRequest) {
        //获取单个参数
        Long id = serverRequest.queryParam("id", ServerRequest.LONG_FUNCTION);
        return ok().body(orderService.findOrderById(id), Order.class);
    }
```
```java
public ServerResponse findOrderById(ServerRequest serverRequest) {
        //直接返回Get请求的参数转换为对象
        Order order = serverRequest.queryParams(Order.class);
    }
```

### Post请求 content-type：json
```java
   public ServerResponse insert(ServerRequest serverRequest) {
        //获取json参数
        OrderRequest orderRequest = serverRequest.body(OrderRequest.class);
        //返回结果
        return ok().body(orderService.insert(order), Long.class);
    }
```
### Post请求 content-type：form-data
```java
public ServerResponse updateKuaidinumById(ServerRequest serverRequest) {
        //获取form-data的单个参数
        long id = serverRequest.queryBodyParam("id", ServerRequest.LONG_FUNCTION);
        String kuaidinum = serverRequest.queryBodyParam("kuaidinum");
        return ok().body(orderService.updateKuaidinumById(id, kuaidinum), Integer.class);
    }
```
```java
public ServerResponse updateKuaidinumById(ServerRequest serverRequest) {
        //获取form-data的参数转为的对象
        Order order = serverRequest.body(Order.class);
        //todo 
    }
```

### JDBC 新增（文档中有更简便的写法）
```java
String sql = """
                INSERT INTO t_order(
                    id, user_id, kuaidinum, kuaidicom, send_name,
                    send_mobile, send_country, send_province, send_city, send_district,
                    send_address, rec_name, rec_mobile, rec_country, rec_province,
                    rec_city, rec_district, rec_address, create_time, update_time
                )
                VALUES(
                    ?, ?, ?, ?, ?,
                    ?, ?, ?, ?, ?,
                    ?, ?, ?, ?, ?,
                    ?, ?, ?, ?, ?
                );
                """;
        // 参数
        SQLParams sqlParams = new SQLParams(20);
        sqlParams.put(order.getId());
        sqlParams.put(order.getUserId());
        //省略
        int rows = JDBC_TEMPLATE.insert(sql, sqlParams);
        return rows > 0 ? order.getId() : -1;
    }

```
### JDBC 修改（文档中有更简便的写法）
```java
public int updateKuaidinumById(long id, String kuaidinum) {
        // sql
        String sql = """
                UPDATE t_order
                SET kuaidinum = ?, update_time = ?
                WHERE id = ?
                """;
        // 参数
        SQLParams sqlParams = new SQLParams(3);
        sqlParams.put(kuaidinum);
        sqlParams.put(LocalDateTime.now());
        sqlParams.put(id);
        return JDBC_TEMPLATE.update(sql, sqlParams);
    }
```
### JDBC 删除（文档中有更简便的写法）
```java
public int deleteById(long id) {
        // sql
        String sql = """
                DELETE FROM t_order
                WHERE id = ?
                """;
        // 参数
        SQLParams sqlParams = new SQLParams(1);
        sqlParams.put(id);
        return JDBC_TEMPLATE.delete(sql, sqlParams);
    }
```
### JDBC 查询列表（文档中有更简便的写法）
````java
/**
     * 数据库查询样例
     * 添加@AutoMapping注解后会自动映射结果。
     * @param id
     * @return
     */
    @AutoMapping
    public Order findOrderById(long id) {
        // sql
        String sql = """
                SELECT  id, user_id, kuaidinum, kuaidicom, send_name,
                        send_mobile, send_country, send_province, send_city, send_district,
                        send_address, rec_name, rec_mobile, rec_country, rec_province,
                        rec_city, rec_district, rec_address, create_time, update_time
                WHERE id = ?
                """;
        // 参数
        SQLParams sqlParams = new SQLParams(1);
        sqlParams.put(id);
        return JDBC_TEMPLATE.proxyQueryForEntity(sql, sqlParams);
    }
````
### JDBC 查询单个（文档中有更简便的写法）
```java
public static final String SIMPLE_SQL = "SELECT FEXPID,FDBID,FTRADETIME,FSOURCE FROM T_MKT_EXPRESS WHERE FEXPID = ?";

    @AutoMapping
    public SimpleResponse querySimpleResponse(Long id) {
        SQLParams sqlParams = new SQLParams(1);
        sqlParams.put(id);
        return JDBC_TEMPLATE.proxyQueryForEntity(SIMPLE_SQL, sqlParams);
    }
```
### JDBC count（文档中有更简便的写法）
```java
@AutoMapping
public Long countSimpleList(Long dbId, Long offset, Long pageSize) {
        SQLParams sqlParams = new SQLParams(3);
        sqlParams.put(dbId);
        return JDBC_TEMPLATE.count(SIMPLE_LIST_SQL,sqlParams);
        }
```




## 性能压测

CPU:8核，内存:15G   
同时部署应用和数据库`PostgreSQL`。应用配置了10G堆内存，数据库配置了4G的Buffer Pool。   
执行语句：`SELECT FEXPID,FDBID,FTRADETIME,FSOURCE FROM T_MKT_EXPRESS WHERE FEXPID = ?`

![img.png](img/img.png)

执行语句：`SELECT FEXPID,FDBID,FTRADETIME,FSOURCE FROM T_MKT_EXPRESS WHERE FDBID = ? LIMIT ? OFFSET ?`

![img_1.png](img/img_1.png)




## WEB框架使用指南
### 配置WEB服务
默认读取resources目录下的HttpServer.properties配置文件进行配置。   
目前支持的配置参数如下：
~~~properties
# 端口 默认为：80
port=9511
# BossGroup线程池数 : 负责客户端的连接。默认为：1
bossThreadCount=1
# WorkerGroup线程池数 : 负责客户端连接的数据读写。默认为：8
workerThreadCount=8
   ~~~
### Http请求参数获取
#### GET请求
获取单个参数时默认返回的结果类型为String。
```java
//获取单个参数。
String id = serverRequest.queryParam("id");
```
如果需要指定返回的参数，可以指定类型转换函数。框架默认提供了基本类型转换的函数式方法。
```java
//获取单个LONG类型的参数，ServerRequest.LONG_FUNCTION为框架提供的函数式方法。
Long id = serverRequest.queryParam("id", ServerRequest.LONG_FUNCTION);
```
框架也提供了更简洁的获取指定类型参数的方式。
```java
//直接返回指定的类型
Long id = serverRequest.queryLongParam("id");
```
如果当获取的参数为空时，可以通过以下方式**指定默认值**。
```java
Long id=serverRequest.queryParam("id",ServerRequest.LONG_FUNCTION,1L);
Long id=serverRequest.queryLongParam("id",1L);
```
通过`queryParams`方法直接将参数转为对象。
```java
public ServerResponse findOrderById(ServerRequest serverRequest){
        //直接返回Get请求的参数转换为对象
        Order order=serverRequest.queryParams(Order.class);
}
```
##### 😃 Get获取参数最佳实践
- 参数两个及以下推荐使用`queryParam`方法一个个的获取Get请求的参数。
- 请求参数在两个以上的推荐使用`queryParams`方法获取**Get请求的参数转成的Java对象**。
- 尽量使用`queryParams`转换为对象，避免频繁获取也可以直接在对象上使用注解对参数进行合法校验，编程效率更高。


#### POST请求

`Post`请求，可以通过`body`方法将请求参数转换为对象。
```java
public ServerResponse insert(ServerRequest serverRequest) {
        //获取json参数
        OrderRequest orderRequest = serverRequest.body(OrderRequest.class);
        //返回结果
        return ok().body(orderService.insert(order), Long.class);
}
```
如果`content-type`为`form-data`也可以和`Get`请求使用同样的方法获取参数。
```java
//自定义转换函数，获取参数
Long id=serverRequest.queryParam("id",ServerRequest.LONG_FUNCTION);
//获取指定类型的参数
Long id=serverRequest.queryLongParam("id");
//直接将参数转为对象
Order order=serverRequest.queryParams(Order.class);
```

```
与Spring获取参数的区别  
对于`content-type`为`form-data`类型的`Post`请求和`Get`请求获取参数的方式可以互通。可以同时使用`params`，`body`方法直接将`Post`和`Get`请求的参数合并转为对象。   
比如`sntExpress/querySimpleListPage?dbId=1212`该请求是`content-type`为`form-data`的`Post`请求，使用`body`转为Java对象时，该对象的dbId字段会设置为`1212`。
上面的链接也可以使用`params`获取到完整的对象信息。
```
##### 😃 Post获取参数最佳实践
- 推荐使用`body`方法获取`Post`请求的参数，在类对象上使用注解对参数进行合法校验。
#### 🙁获取参数不推荐用法
- 使用`params`或`body`方法获取到对象后，继续使用编程的方式去校验参数合法性。
```java
ChannelInfoRequest channelInfo = request.queryParams(ChannelInfoRequest.class);
//下面这一段校验可以省略，在ChannelInfoRequest.class中使用注解进行参数合法性校验
ValidateBuilder.build()
                .validate(Check.NotEmpty, channelInfo.getChannelName(), "渠道名称不能为空!")
                .validate(Check.Length, channelInfo.getChannelName(), "20", "渠道名称不能大于20字")
                .validate(Check.NotEmpty, channelInfo.getChannelType(), "请选择渠道类型")
                .validate(Check.Pattern, channelInfo.getChannelName(), CHAR_REGEXP, "仅支持输入中文,英文,数字,点号,下划线,英文括号组合")
                .doCheck()
                .ifNotPassedThrowException();
```
- 自己手动转换类型
```java
//不推荐，可以使用serverRequest.queryLongParam("id");直接获取到对应类型的参数。
 Long id = Long.valueOf(request.queryParam("id"));
```
- 使用`queryParams`或`body`方法后仍使用`queryParam`单个获取参数。
```java
ChannelPageInfoResponse channelPageInfo = request.queryParams(ChannelPageInfoResponse.class);
//上面已经设置了pageNo，不需要另外获取。
channelPageInfo.setPageNo(request.queryIntegerParam("pageNo", 1));
channelPageInfo.setPageSize(request.queryIntegerParam("pageSize", 10));
```
- 需要登录的接口，不需要在业务中再次校验DBID是否为空


#### 获取Cookie
```java
HttpCookie cookie=serverRequest.getCookie("name");
```
#### 设置Cookie
```java
Cookie cookie = new DefaultCookie(cookie名称, cookie内容);
cookie.setPath(xx);
cookie.setDomain(xx);
cookie.setMaxAge(xx);
request.exchange().getResponse().addCookie(cookie);
```
#### 获取Header
```java
String header = serverRequest.getHeader("name");
```
#### 设置Header
```java
request.exchange().getResponse().setHeader("name",object);
```
#### 获取上传文件
上传文件的默认大小为5M。如果需要变更，可以在创建`HttpServer`时指定大小。
```java
HttpServer.create()
         //设置最大上传文件大小 单位byte
          .maxUploadFileSize(5 * 1024 * 1024)
         //启动服务，绑定端口
          .bind();
```
##### 获取上传的所有文件
```java
Map<String, MultipartFile> fileMap = serverRequest.queryFiles();
```
##### 获取指定的上传文件
```java
MultipartFile multipartFile = serverRequest.queryFile("fileName");
```
##### 获取文件后缀
```java
multipartFile.getSuffix();
```
⚠️暂时不支持分片上传，最大上传文件的大小尽量设置的不要过大，避免内存溢出。
##### 获取上传文件示例
下面是一个获取上传的文件并存储到指定路径的例子。
```java
MultipartFile multipartFile = serverRequest.queryFile("file");
File file = new File("e:\\test\\" + multipartFile.getOriginalFilename());
multipartFile.transferTo(file);
```
### 请求参数校验
#### 编码校验参数示例
```java
 String kuaidinum = serverRequest.queryBodyParam("kuaidinum");
 //校验queryBodyParam,多个校验如果有一个参数非法，就抛出异常;
        // 如果需要校验所有参数,调用doCheckAll方法
 ValidateBuilder.build()
        .Validate(Check.NotEmpty,kuaidinum,"")
        .Validate(Check.Length,kuaidinum,"2,19","")
        .doCheck()
        .ifNotPassedThrowException();
 //如果在一个线程中复用,先调用.clear方法清除保存验证相关信息的容器内容
 ValidateBuilder.build()
        .clear()
        .Validate(Check.Chinese,kuaidinum,"")
        .doCheck().ifNotPassedThrowException();
 //校验queryParam
 Long id = serverRequest.queryParam("id", ServerRequest.LONG_FUNCTION);
 ValidateBuilder.build()
        .Validate(Check.NotNull,id,"")
        .Validate(Check.gt,id,"0","")
        .doCheck()
        .ifNotPassedThrowException();
 //返回错误信息不抛出异常
ValidateBuilder.build()
        .Validate(Check.NotEmpty,kuaidinum,"")
        .Validate(Check.Length,kuaidinum,"2,19","")
        .doCheck()
        .getFailedMsgs()
//返回校验失败的条数
ValidateBuilder.build()
        .Validate(Check.NotEmpty,kuaidinum,"")
        .Validate(Check.Length,kuaidinum,"2,19","XX字段长度超出")
        .doCheck()
        .getFailedCounts()
```

#### 注解校验参数示例
一般在实体类的字段上添加注解,此处列举部分使用方法;
后期根据需求可新增更多校验方法;
message为可选,如果需要自定义返回信息才传
```java
1.参数必须为空
注解使用方法:@Validate(fun= Check.Null)
2.参数必须不为空
@Validate(fun= Check.NotNull)
3.参数的必须为空
@Validate(fun= Check.isEmpty)
4.参数必须非空
@Validate(fun= Check.isNotEmpty)
5.参数必须为 true
//* 支持Boolean类型 * 支持String类型
@Validate(fun= Check.isTrue)
6.参数必须为 false
@Validate(fun= Check.isFalse)
7.参数必须是一个日期 yyyy-MM-dd
//判断参数是否是一个日期
//支持Date类型
//支持LocalDate类型
//支持String类型，yyyy-MM-dd、yyyyMMdd、yyyy/MM/dd格式； 默认仅支持yyyy-MM-dd
@Validate(fun= Check.isDate,express = "yyyy-MM-dd")
8.参数必须是一个日期时间 yyyy-MM-dd HH:mm:ss
//判断参数是否是一个日期
// 支持Date类型
// 支持LocalDateTime类型
//支持String类型，yyyy-MM-dd HH:mm:ss、yyyyMMddHHmmss、yyyy/MM/dd HH:mm:ss格式； 默认仅支持yyyy-MM-dd HH:mm:ss
@Validate(fun= Check.isDate,express = "yyyyMMddHHmmss",message="参数必须是一个日期时间")
9.参数必须在合适的范围内
// * 判断参数的取值范围，逗号隔开，无空格；闭区间
//* 支持Integer、Long、Short、Float、Double、BigDecimal
@Validate(fun= Check.inRange,express = "2,90")
10.参数长度必须在指定范围内
//* 判断参数的取值范围，逗号隔开，无空格；闭区间
//* 判断String的length范围, rangeStr取值举例："6,18"
@Validate(fun= Check.inLength,express = "2,90",message="字段长度超出")
```

### ip黑名单过滤
IP黑名单列表保存在配置文件ip_list中,后期会通过配置中心配置文件内容
如果是在黑名单列表，返回true;白名单返回false
如果在全局使用ip黑名单过滤,在全局的拦截器上调用该方法
```java
boolean isblack = IPFiter.checkIp(serverRequest);
//ip_list配置格式如下
192.168.1.0/24
8.8.8.8
```


### 定义对外接口
框架提供函数式对外接口定义。目前只支持Get和Post两种请求。
```java
public RouterFunction<ServerResponse> orderHandlerRouter() {
    return RouterFunctions.route()
            .POST("/order/deleteById", orderHandler::deleteById)
            .GET("/order/findOrderById", orderHandler::findOrderById)
            .build();
}
```
#### 接口返回Json结果封装
##### 默认返回结果统一封装
```java
public ServerResponse getUserInfo(ServerRequest serverRequest) {
    return ok().body(userService.getUserInfo(token.getName()));
}
```
使用`com.superxiaobailong.webserver.server.DefaultServerResponse.BodyBuilder.ok().body()`返回的结果默认会使用`com.superxiaobailong.core.model.Result`封装，结果会在`data`字段填充。
```java
返回示例
{
    "message": null,
    "status": 200,
    "data": xxxx
}
```
##### 自定义返回结果统一封装
在服务创建的时候指定全局封装逻辑。
```java
HttpServer.create()
        // 添加请求接口
        .mapping(new Routers().route())
        //自定义全局结果封装
        .resultWrapper(result -> {
            //封装逻辑
            return 封装后的结果;
        })
        .bind();
```
##### 自定义全局异常处理
```java
HttpServer.create()
        // 添加请求接口
        .mapping(new Routers().route())
        //自定义全局异常封装
        .exceptionWrapper(e -> {
            //异常处理
            return 封装后的结果;
        })
        .bind();
```
##### 跳过统一结果封装
如果返回结果不想进行统一封装，可以使用`com.superxiaobailong.webserver.server.DefaultServerResponse.BodyBuilder.ok().render()`返回结果。注意：`contentType`一定要设置为`Json`。
```java
//自定义返回结果
 Map<String,Object> map = new HashMap<>(3);
        map.put("status",0);
        map.put("data","xbl");
        map.put("message",null);
        return ok().contentType(MediaType.APPLICATION_JSON).render("",map);
```
```
//返回结果
{
    "data": "xbl",
    "message": null,
    "status": 0
}
```
#### 接口返回伪静态页面
固定将静态文件放在项目的./resources/templates目录下,以文件名以`.html`结尾。
框架底层使用`thymeleaf`渲染静态页面，所以`HTML`页面需要动态渲染的部分需要遵从`thymeleaf`语法。

伪静态接口定义与普通Json接口没有区别。
```java
//定义伪静态接口
public RouterFunction<ServerResponse> indexRoute() {
     return RouterFunctions.route()
                           .GET("/index.html", indexHandler::index)
                           .GET("/orders.html", indexHandler::orders)
                           .build();
}
```
接口返回
```java
 public ServerResponse orders(ServerRequest serverRequest) {
    BasePageInfo pageInfo = serverRequest.queryParams(BasePageInfo.class);
    Page<Order> page = orderService.listPageAll(pageInfo.getPageNo(), pageInfo.getPageSize(), pageInfo.getOffset());
    Map<String, Object> params = new HashMap<>();
    params.put("orders", page);
    return ok().contentType(MediaType.TEXT_HTML).render("/orders", params);
}
```





#### 😃 接口定义最佳实践
按业务种类将接口分开定义，可以使用`RouterFunction`的`and`操作符将多个`RouterFunction`拼接起来。
```java
public RouterFunction<ServerResponse> myRouterFunction() {
    return orderHandlerRouter()
            .and(payHandlerRouter());
}

public RouterFunction<ServerResponse> orderHandlerRouter() {
    return RouterFunctions.route()
            .POST("/order/insert", orderHandler::insert)
            .build();
}

public RouterFunction<ServerResponse> payHandlerRouter() {
    return RouterFunctions.route()
            .POST("/pay/insert", payHandler::insert)
            .build();
}
```

### 过滤器
#### 过滤器的定义
框架的过滤器实现是在对外接口上进行扩展的。`route`的`Builder`提供`filter`方法添加过滤器。
```java
public RouterFunction<ServerResponse> orderHandlerRouter() {
        return RouterFunctions.route()
                .GET("/order/findOrderById", orderHandler::findOrderById)
                .filter((serverRequest, next) -> {
                    LOGGER.info("过滤器");
                    //todo 过滤逻辑
                    return next.handle(serverRequest);
                })
                .build();
    }
```
指定接口进行过滤。
```java
public RouterFunction<ServerResponse> orderHandlerRouter() {
    return RouterFunctions.route()
            .GET("/order/findOrderById", orderHandler::findOrderById, ((serverRequest, next) -> {
                LOGGER.info("过滤器222");
                return next.handle(serverRequest);
            }))
            .build();
}
```
指定接口添加多个过滤器。
```java
public RouterFunction<ServerResponse> orderHandlerRouter() {
    List<HandlerFilterFunction<ServerResponse, ServerResponse>> filterFunctionList = new ArrayList<>();
    filterFunctionList.add((serverRequest, next) -> {
        LOGGER.info("过滤器");
        return next.handle(serverRequest);
    });
    return RouterFunctions.route()
            .GET("/order/findOrderById", orderHandler::findOrderById, filterFunctionList)
            .build();
}
```
**通过以上方式定义的过滤器都只会对当前`Builder`的`Router`生效。** 也就是说通过上面的方式定义的过滤器是局部生效的。
```java
public RouterFunction<ServerResponse> myRouterFunction() {
    return orderHandlerRouter()
            .and(payHandlerRouter());
}

public RouterFunction<ServerResponse> orderHandlerRouter() {
    return RouterFunctions.route()
            .GET("/order/findOrderById", orderHandler::findOrderById)
            .filter((serverRequest, next) -> {
                LOGGER.info("过滤器");
                return next.handle(serverRequest);
            })
            .build();
}

public RouterFunction<ServerResponse> payHandlerRouter() {
    return RouterFunctions.route()
            .POST("/pay/insert", payHandler::insert)
            .build();
}
```
上面的例子中，在`orderHandlerRouter`中定义的过滤器不会对`payHandlerRouter`中定义的`route`产生过滤行为。

如果想实现定义的过滤器在全局生效，可以在对多个`RouterFunction`进行拼接后再添加过滤。
```java
public RouterFunction<ServerResponse> myRouterFunction() {
    return orderHandlerRouter()
            .and(payHandlerRouter()).filter((serverRequest, next) -> {
                LOGGER.info("过滤器");
                return next.handle(serverRequest);
            });
}
```

#### 过滤器的执行顺序
**局部过滤器的执行顺序按定义的顺序进行执行。**
```java
public RouterFunction<ServerResponse> orderHandlerRouter() {
        return RouterFunctions.route()
                .GET("/order/findOrderById", orderHandler::findOrderById)
                .filter((serverRequest, next) -> {
                    LOGGER.info("过滤器1");
                    return next.handle(serverRequest);
                })
                .filter((serverRequest, next) -> {
                    LOGGER.info("过滤器2");
                    return next.handle(serverRequest);
                })
                .filter((serverRequest, next) -> {
                    LOGGER.info("过滤器3");
                    return next.handle(serverRequest);
                })
                .build();
    }
```
上面的例子中过滤器的执行顺序为：过滤器1->过滤器2->过滤器3。

**全局过滤器的执行顺序为后定义的先执行。**
```java
public RouterFunction<ServerResponse> myRouterFunction() {
    return orderHandlerRouter()
            .and(payHandlerRouter())
            .filter((serverRequest, next) -> {
                LOGGER.info("过滤器5");
                return next.handle(serverRequest);
            })
            .filter((serverRequest, next) -> {
                LOGGER.info("过滤器6");
                return next.handle(serverRequest);
            });
}
```
上面的例子中过滤器的执行顺序为：过滤器6->过滤器5。

**既有全局过滤器又有局部过滤器的执行顺序为：先执行全局过滤器，然后再执行局部过滤器。**  在执行全局过滤器阶段按全局过滤器的顺序执行，在执行局部过滤器阶段按局部过滤器的顺序执行。

```java
public RouterFunction<ServerResponse> myRouterFunction() {
    return orderHandlerRouter()
            .and(payHandlerRouter())
            .filter((serverRequest, next) -> {
                LOGGER.info("过滤器5");
                return next.handle(serverRequest);
            })
            .filter((serverRequest, next) -> {
                LOGGER.info("过滤器6");
                return next.handle(serverRequest);
            });
}

public RouterFunction<ServerResponse> orderHandlerRouter() {
    return RouterFunctions.route()
            .GET("/order/findOrderById", orderHandler::findOrderById)
            .filter((serverRequest, next) -> {
                LOGGER.info("过滤器1");
                return next.handle(serverRequest);
            })
            .filter((serverRequest, next) -> {
                LOGGER.info("过滤器2");
                return next.handle(serverRequest);
            })
            .filter((serverRequest, next) -> {
                LOGGER.info("过滤器3");
                return next.handle(serverRequest);
            })
            .build();
}
```
上面的例子中过滤器的执行顺序为：过滤器6->过滤器5->过滤器1->过滤器2->过滤器3。

### 拦截器
框架中提供的拦截器也是基于对外接口上进行扩展，底层实现基于过滤器。
```java
public RouterFunction<ServerResponse> orderHandlerRouter() {
    return RouterFunctions.route()
            .GET("/order/findOrderById", orderHandler::findOrderById)
            .before((request) -> {
                //todo 
                return request;
            })
            .after((request, response) -> {
                //todo 
                return response;
            })
            .build();
}
```
`before`方法在执行route的逻辑前执行，多个`before`与过滤器的执行顺序根据定义的先后顺序执行。   
`after`方法则在route的逻辑执行后执行，多个`after`按定义的顺序执行。   
使用单个过滤器也可以实现`before`，`after`的功能。

### 线程池隔离
框架支持特定的业务在特定的线程池中执行，如果没有指定线程池则会在框架默认提供的线程池中执行。
#### 局部线程池隔离定义
使用`runOnScheduler`方法指定线程池，局部定义的`route`对应的业务逻辑就会在该线程池中执行。
```java
//线程数，队列长度，线程名称，存活时间，守护线程
Scheduler scheduler2 = Schedulers.newElastic(4, 4, "test2", 1, ThreadTypeEnum.DAEMON);
public RouterFunction<ServerResponse> sntExpressRoute() {
    return RouterFunctions.route()
            .GET("/sntExpress/querySimpleList", expressHandler::querySimpleList)
            .runOnScheduler(scheduler2)
            .build();

}
```
#### 指定接口线程池定义
在`GET`或`POST`方法中指定线程池来实现接口级别的线程池隔离。

```java
public RouterFunction<ServerResponse> sntExpressRoute() {
    return RouterFunctions.route()
            .GET(scheduler, "/sntExpress/findPddSyncOrderCount", expressHandler::findPddSyncOrderCount)
            .build();
}
```
#### 同时定义局部与接口线程池隔离
```java
public RouterFunction<ServerResponse> sntExpressRoute(){
        return RouterFunctions.route()
        .POST("/sntExpress/updateTabId",expressHandler::updateTabId)
        .GET(scheduler,"/sntExpress/findPddSyncOrderCount",expressHandler::findPddSyncOrderCount)
        .runOnScheduler(scheduler2)
        .build();
}
```
`/sntExpress/findPddSyncOrderCount`会优先在接口级线程池`scheduler`中执行，不会在局部线程池`scheduler2`中执行。其他接口则在局部线程池`scheduler2`中执行。

#### 全局线程池
全局的线程池隔离由框架默认提供，支持由启动参数设置大小。    
`kd.schedulers.defaultElasticSize`设置全局线程池的大小，默认值为20。   
`kd.schedulers.defaultElasticQueueSize`设置全局线程池队列的大小，默认值10000。

### 限流
基于令牌桶算法实现限流。令牌桶的令牌数最多会存储一秒的令牌数，也就是可以应对最多一秒的请求数的突发流量。
#### 局部限流

局部限流的作用范围为`Builder`与局部过滤器同理，同一个`Builder`创建的`route`都在该限制范围内，这些`route`共享一个限制值。通过指定`每秒的允许的请求数`和`超过执行请求数后的fallback方法`
来实现限流。如果未指定fallback方法，在超过指定的请求数后会直接抛出异常。

```java
public RouterFunction<ServerResponse> sntExpressRoute() {
    return RouterFunctions.route()
            .GET(scheduler, "/sntExpress/findPddSyncOrderCount", expressHandler::findPddSyncOrderCount)
            .limiter(1, (request -> {
                return DefaultServerResponse.BodyBuilder.error().body("", String.class);
            }))
            .build();
}

```
#### 接口级别限流
```java
public RouterFunction<ServerResponse> sntExpressRoute() {
    return RouterFunctions.route()
            .POST("/sntExpress/updateTabId", expressHandler::updateTabId)
            .limiter(RequestPredicates.GET("/pattern"), 1, (request -> {
                return DefaultServerResponse.BodyBuilder.error().body("", String.class);
            }))
            .build();
}
```
上面的例子中指定了`RequestPredicates.GET("/pattern")`，所以`Get`的`/pattern`请求会被限流。

#### 需要预热的接口限流

除了普通的令牌数限流外，框架还支持需要预热的接口限流场景。实现的方式是在系统没有请求的情况下，令牌的发放速度会变慢，也就是接口允许每秒的请求数会降低。最冷的时候令牌发放的速度是正常情况下的3倍，随着系统请求逐渐变多，发放的速度会逐渐变快直到和设置的发放速度一致。

```java
public RouterFunction<ServerResponse> sntExpressRoute() {
    return RouterFunctions.route()
            .GET(scheduler, "/sntExpress/findPddSyncOrderCount", expressHandler::findPddSyncOrderCount)
            //每秒允许的请求数，预热时间
            .limiter(1, Duration.ofSeconds(10))
            .build();
}
```

```java
              ^ throttling
              |
        cold  +                  /
     interval |                 /.
              |                / .
              |               /  .  
              |              /   .    
              |             /    .
              |            /     .
              |           /      .
       stable +----------/  WARM .
     interval |          .   UP  .
              |          . PERIOD.
              |          .       .
            0 +----------+-------+--------------→ storedPermits
              0 thresholdPermits maxPermits
              

```
#### 😃 限流最佳实践
- 需要预热的接口要设置合理的预热时间。
- 启动后需要加载缓存的接口最好设置预热时间。
- 不能处理突发流量（请求数突然变高一秒，然后又恢复正常）的接口需要设置预热时间，因为默认是可以处理突发1秒的请求数。
- 对于不需要预热，没有缓存并且可以处理突发流量的接口推荐不设置预热时间，极端情况下一秒最多处理设定值两倍的请求量。

### Session
Session的默认空闲两小时过期。可以在配置文件`HttpServer.property`中设置`maxSessionIdealSeconds`进行变更。

#### Session配置
在创建`HttpServer`时可以设置Session的`最大空闲时间`，`Session存储方式`。
```java
public static void main(String[] args) {
        MyRouterFunctions myRouterFunction = new MyRouterFunctions();
        HttpServer.create()
                //session最大的空闲时间。单位：秒
                .maxSessionIdealSeconds(60 * 60 * 2)
                //设置session存储模式
                .sessionStoreStrategy(SessionStoreStrategyEnum.JDBC)
                .bind();
    }
```
框架目前支持三种Session存储方式：`关系型数据库-JDBC`，`内嵌数据库-LevelDB`，`内存-Memory`。默认为`关系型数据库-JDBC`，也推荐使用`关系型数据库-JDBC`。
##### 使用关系型数据库-JDBC存储Session
```java
public static void main(String[] args) {
        MyRouterFunctions myRouterFunction = new MyRouterFunctions();
        HttpServer.create()
                //配置使用关系型数据库来存储Session
                .sessionStoreStrategy(SessionStoreStrategyEnum.JDBC)
                .bind();
    }
```
###### 指定数据库与连接池
默认Session和业务使用同一个数据库和同一个数据库连接池访问数据库。也可以手动指定数据库和数据库连接池。
```java
//另外指定线程池进行Session的数据库访问
public static void main(String[] args) {
        //创建一个数据库连接池配置
        DataSourceConfigProperties dataSourceConfigProperties = new DataSourceConfigProperties();
        dataSourceConfigProperties.setConnTimeOut(3000);
        dataSourceConfigProperties.setDriverClassName("org.postgresql.Driver");
        dataSourceConfigProperties.setIdleTimeout(30000);
        dataSourceConfigProperties.setUsername("xx");
        dataSourceConfigProperties.setPassword("zz");
        dataSourceConfigProperties.setUrl("jdbc:postgresql://10.240.1.154:5432/snt-order");
        dataSourceConfigProperties.setPoolName("session-pool");
        dataSourceConfigProperties.setMaximumPoolSize(2);

        HttpServer.create()
                .sessionStoreStrategy(SessionStoreStrategyEnum.JDBC)
                //使用指定的数据库配置
                .sessionJdbcConfigProperties(dataSourceConfigProperties)
                //启动服务，绑定端口
                .bind();
}
```
###### 关系型数据库Session缓存说明
- 内存中存放Session有内存溢出风险   
  在内存中存储Session如果控制不好Session的个数与大小，会造成内存溢出。
- Session只存放数据库可能会更浪费内存以及导致性能下降   
  为了避免内存溢出可以不在内存中存储Session，每个请求都单独去数据库获取Session。这种情况下，如果某用户加载一个页面同时发起多个接口请求，处理这些请求的接口会都去数据库加载Session，此时内存中同一个Session就会存储多次。用户请求是一个连续的过程，所以一些活跃用户的Session其实也是一直在内存中。如果没有控制好Session获取反而更占用内存。在处理完接口逻辑后，修改Session最后更新时间时也会发生数据库锁竞争，导致性能降低。

框架Session存储使用了数据库+内存混合的方式。数据库存储全量的Session数据，内存中存放部分热点的Session。处理请求的接口会先到缓存查询是否命中，未命中则会去数据库查询（多个请求也只会有一个请求查询数据库，其他线程等待结果），避免多次存储相同的Session在内存。Session过期时间定时去数据库修改，避免发生锁竞争。
- 避免内存溢出    
  在GC压力较大的情况下，发生GC就会对Session进行回收。    
  在GC压力较小的情况下，会根据当前空闲的内存*固定值计算出一个最大空闲时间，基于LRU算法淘汰Session。
- 避免更新最后访问时间产生锁    
  定时将内存中的Session的最后访问时间同步到数据库。  
  在Session被GC掉后，将其最后访问时间同步到数据库。

##### 使用内嵌数据库LevelDB存储Session
LevelDB的数据时存储在Java应用运行的机器上。如果在k8s中部署想要在pod重启后Session数据仍存在需要配置持久卷。   
使用LevelDB需要在应用中引入相应的Java包，不需要指定版本号。
```
<dependency>
    <groupId>org.iq80.leveldb</groupId>
    <artifactId>leveldb</artifactId>
</dependency>
<dependency>
    <groupId>org.iq80.leveldb</groupId>
    <artifactId>leveldb-api</artifactId>
</dependency>
```
##### 😃 Session存储最佳实践
- 优先选择关系型数据库存储Session信息。除非一些特殊场景，禁止只使用内存存储Session。
- 存储Session的数据库连接池另外配置，最好不要和业务数据库连接池混合使用。
- 不要在Session中存储大对象。注意：Session最大为4kb，如果超过最大值会抛出异常。



#### 设置登录信息
通过`Builder`的`auth`方法设置登录信息。同时，执行完`auth`方法后Session会被设置到线程的上下文中。
```java
public RouterFunction<ServerResponse> sntExpressRoute(){
        return RouterFunctions.route()
        .POST("/sntExpress/updateTabId",expressHandler::updateTabId)
        .auth(serverRequest->{
        //todo 校验用户信息
        //返回需要设置到session的用户信息
        return object;
        })
        .build();
}
```
如果想在`RouterFunction`的拼接中做到部分接口无需登录也可以访问的话可以使用下面的方式。
```java
return healthRouter().and(userRouter())
                .and(sentMailRouter())
                .and(extensionRouter())
                .auth((serverRequest) -> new LoginFilter().apply(serverRequest))
                .and(channelRouter())
                .and(sentExpressRouter()).and(testRouter());
```
在`auth`后定义的`channelRouter()`等接口就不会进行`auth`中的校验。
但是在`Builder`中定义的都会执行，不论`auth`定义在最前面还是最后面，这个`Builder`中的接口
访问时如果没有登录，则会触发`auth`中的逻辑。

#### 😃 登录校验最佳实践
- 将接口按需要登录和不需要登录分开在不同的`Builder`中定义，使用`RouterFunction`拼接时需要登录的接口放在`auth`方法前面，不需要登录的放在`auth`方法后面。
- 对外开放的接口（不携带Cookie），禁止使用auth方法校验，因为每一次请求都会创建一个新的Session。

#### 在业务代码中获取登录信息
```java
User user=(User)ThreadContext.getPrincipal();
```
这里获取的信息就是在`auth`方法中返回的对象。
#### 快速获取dbId
如果需要快速获取到dbId，在`auth`中设置登录信息时，设置的对象需要继承`com.superxiaobailong.webserver.security.open.Identify`。
```java
//快速获取DbId
ThreadContext.getDbId();
```

#### 在业务代码中获取Session
```java
//通过线程上下文获取,如果不存在session会直接抛出异常。
ThreadContext.getSession();
//通过request获取，如果不存在session会直接抛出异常。
serverRequest.session();
```

#### 在Session中设置/获取业务信息

```java
ThreadContext.getSession().setAttribute("key","value");
ThreadContext.getSession().getAttribute("key");
```

#### 在当前线程中设置/获取业务信息

```java
ThreadContext.put("key","value");
ThreadContext.get("key");
```
#### 登出
```java
ThreadContext.getSubject().logout();
```
### 分页查询请求
#### 请求参数
接收请求参数的对象需要继承`com.superxiaobailong.core.model.BasePageInfo`,
`BasePageInfo`定义了基本的分页信息，包括`页号`，`页大小`。
- 如果前端传递的参数中未设置`页号`则会设定其默认值为1。
- 如果前端传递的参数中未设置`页大小`则会设定其默认值为10。
- 通过`getOffset`方法获取`offset`不需要再进行计算。
- `页号`最大不能超过10000，`页大小`不能超过50。
- 对于不需要计算总行数的场景，可以设置`queryCount`参数为`false`。在应用中判断该参数是否查询总行数。

#### 返回结果
返回结果可以使用`com.superxiaobailong.core.model.Page`中的`build`方法进行构建。
可以设置请求参数快速构建返回结果，避免重复设置`页号`，`页大小`。   
对于不需要计算总行数的场景，可以通过`Page`的`hasNextPage`字段判断是否有下一页。

#### 分页查询示例
请求参数类定义
```java
//分页的请求参数类需要继承BasePageInfo，其中已经封装了分页查询需要的信息
public class OrderPageRequest extends BasePageInfo {
    private String search;

}
```
分页参数获取
```java
public Page pageQuery(OrderPageRequest orderPageRequest) {
        //获取页大小
        orderPageRequest.getPageSize();
        //获取分页offset，自动根据PageNo与PageSize计算。
        orderPageRequest.getOffset();
        //执行分页查询
        ...
        
        //结果封装，不需要查询总数的场景
        Page.build(orderPageRequest, Collections.emptyList());
        
        //结果封装，需要查询总数的场景
        Page.build(orderPageRequest,Collections.emptyList(),100L);
    }
```

### 定时任务
如果需要开启定时任务需要在项目中引入对应的Jar包。
```
<!-- 定时任务 -->
<dependency>
    <groupId>com.super.xiaobailong</groupId>
    <artifactId>xbl.scheduling</artifactId>
    <scope>compile</scope>
</dependency>
```
启动方法中需要添加启动类。
```
public class Application {
    public static void main(String[] args) {
        HttpServer.create()
                // 添加请求接口
                .mapping(new Routers().route())
                // 启动服务，绑定端口
                //如果需要开启定时任务还需要设置启动类Application.class
                .bind(Application.class);
    }
}
```
约定需要使用定时任务的类，单例的定义使用`getInstanse`方法获取。    
在需要定时执行的方法上添加@Scheduled注解，并设定cron表达式。（<a href="http://cron.ciding.cc/"> cron表达式在线生成</a>）
```java
@Scheduled(cron = "0/10 * * * * ?")
public void task() {
    logger.info("定时任务已经被执行。。");
}
```

## JDBC框架使用指南
### 配置数据库连接信息
默认读取resources目录下的xblPool.properties配置文件进行配置。   
目前支持的配置参数如下：
   ~~~properties
   # 数据库驱动
   driverClassName=org.postgresql.Driver
   # 数据库地址
   url=jdbc:postgresql://10.240.1.43:5432/snt-order?reWriteBatchedInserts=true&assumeMinServerVersion=14.0
   # 用户名
   username=postgres
   # 密码
   password=kuaidi100
   # 连接池名字
   poolName=myPool
   # 连接超时时间（重试时间），单位毫秒
   connTimeOut=3000
   # 空闲连接存活时间
   idleTimeout=30000
   # 最大连接数
   maximumPoolSize=25
   ~~~

### 新增
##### 旧写法
使用`JDBC_TEMPLATE.insert`方法进行新增语句执行，结果返回新增的行数。   
查询语句中待替换的参数使用`?`占位，定义`SQLParams`来进行参数填充。注意`SQLParams`添加参数的顺序要与`SQL`中参数占位符的顺序一致。
```java
String sql = """
         INSERT INTO t_order(
             id, user_id, kuaidinum, kuaidicom, send_name,
             send_mobile, send_country, send_province, send_city, send_district,
             send_address, rec_name, rec_mobile, rec_country, rec_province,
             rec_city, rec_district, rec_address, create_time, update_time
         )
         VALUES(
             ?, ?, ?, ?, ?,
             ?, ?, ?, ?, ?,
             ?, ?, ?, ?, ?,
             ?, ?, ?, ?, ?
         );
         """;
 // 参数
 SQLParams sqlParams = new SQLParams(20);
 sqlParams.put(order.getId());
 sqlParams.put(order.getUserId());
 //省略n行
 int rows = JDBC_TEMPLATE.insert(sql, sqlParams);
```
##### 新写法
使用@Query和@Modifying注解进行新增语句执行，结果返回新增行数。
```java
@Query("""
        INSERT INTO t_order(
                id, user_id, kuaidinum, kuaidicom, send_name,
                send_mobile, send_country, send_province, send_city, send_district,
                send_address, rec_name, rec_mobile, rec_country, rec_province,
                rec_city, rec_district, rec_address, create_time, update_time
            )
            VALUES(
                #order.id#, #order.userId#, #order.kuaidinum#, #order.kuaidicom#, #order.sendName#,
                #order.sendMobile#, #order.sendCountry#, #order.sendProvince#, #order.sendCity#, #order.sendDistrict#,
                #order.sendAddress#, #order.recName#, #order.recMobile#, #order.recCountry#, #order.recProvince#,
                #order.recCity#, #order.recDistrict#, #order.recAddress#, #order.createTime#, #order.updateTime#
            )
        """)
@Modifying
public int insert(@Param("order") Order order) {
    //结果占位符，固定写法。
    return ProxySQLResult.ok();
}
```
- `@Query`注解中为执行的SQL语句，其中占位符从`?`变为`#`包围的变量，`#`包围的变量名称对应方法参数列表的`@Param`的名称。
- `@Modifying`表示这个是一个修改语句，会影响数据库表中的数据。
- 使用`@Query`注解和`@Modifying`注解进行新增查询时，返回结果为新增的行数，结果类型固定为`int`，不能为对应包装类型，否则编译会失败。
- 方法体中固定写为`ProxySQLResult.ok();`，作为结果的占位符。

<a id= "info"></a>
##### 占位符说明
```
`@Param("name")`对应`@Query`中`SQL`语句中的`#name#`占位符;
`@Query`中`SQL`语句的`#order.id#`对应`方法参数列表的`@Param("order")`参数的id字段。也就是说会使用调用order.getId()返回的值填充对应的#order.id#占位符。
```
<a id= "zuijia"></a>
##### 最佳实践
推荐使用@Query注解进行SQL语句查询。相比老的查询方式新得注解写法有以下几个好处：
- 简化代码编写，提升开发效率。
- 便于维护。老的查询方式依赖参数的顺序，新的使用名称匹配，避免后续参数调整顺序后出现参数无法对应的问题。



### 修改
##### 旧写法
使用`JDBC_TEMPLATE.update`方法进行修改语句执行，结果返回影响行数。  
查询语句中待替换的参数使用`?`占位，定义`SQLParams`来进行参数填充。注意`SQLParams`添加参数的顺序要与`SQL`中参数占位符的顺序一致。
```java
public int updateKuaidinumById(long id, String kuaidinum) {
    // sql
    String sql = """
            UPDATE t_order
            SET kuaidinum = ?, update_time = ?
            WHERE id = ?
            """;
    // 参数
    SQLParams sqlParams = new SQLParams(3);
    sqlParams.put(kuaidinum);
    sqlParams.put(LocalDateTime.now());
    sqlParams.put(id);
    return JDBC_TEMPLATE.update(sql, sqlParams);
}
```
##### 新写法
使用@Query和@Modifying注解进行修改语句执行，结果返回修改行数。
```java
@Query("UPDATE t_order SET kuaidinum = #kuaidinum#, kuaidicom = #kuaidicom# WHERE id = #id#")
@Modifying
public int updateKuaidinumAndkuaidicomById(@Param("id") Long id, @Param("kuaidinum") String kuaidinum,
                                           @Param("kuaidicom") String kuaidicom) {
    //占位符，固定写法。
    return ProxySQLResult.ok();
}
```
##### <a href= "#info">占位符说明<a>
##### <a href= "#zuijia">最佳实践<a>

### 删除
##### 旧写法
使用`JDBC_TEMPLATE.delete`方法进行删除语句执行，结果返回影响行数。  
查询语句中待替换的参数使用`?`占位，定义`SQLParams`来进行参数填充。注意`SQLParams`添加参数的顺序要与`SQL`中参数占位符的顺序一致。
```java
public int deleteById(long id) {
    // sql
    String sql = """
            DELETE FROM t_order
            WHERE id = ?
            """;
    // 参数
    SQLParams sqlParams = new SQLParams(1);
    sqlParams.put(id);
    return JDBC_TEMPLATE.delete(sql, sqlParams);
}
```
##### 新写法
使用@Query和@Modifying注解进行删除语句执行，结果返回删除行数。
```java
@Query(" DELETE FROM t_order WHERE id = #id#")
@Modifying
public int deleteById(@Param("id") Long id) {
    //占位符，固定写法。
    return ProxySQLResult.ok();
}
```
##### <a href= "#info">占位符说明<a>
##### <a href= "#zuijia">最佳实践<a>

### 查询列表
##### 旧写法
使用`JDBC_TEMPLATE.proxyQueryForList`方法进行列表查询语句执行，结果返回匹配的行。  
查询语句中待替换的参数使用`?`占位，定义`SQLParams`来进行参数填充。注意`SQLParams`添加参数的顺序要与`SQL`中参数占位符的顺序一致。
```java
public static final String SIMPLE_LIST_SQL = "SELECT FEXPID,FDBID,FTRADETIME,FSOURCE FROM T_MKT_EXPRESS WHERE FDBID = ? LIMIT ? OFFSET ?";
//添加@AutoMapping注解后会自动映射结果。
@AutoMapping
public List<SimpleResponse> querySimpleList(Long dbId, Long offset, Long pageSize) {
    SQLParams sqlParams = new SQLParams(3);
    sqlParams.put(dbId);
    sqlParams.put(pageSize);
    sqlParams.put(offset);
    return JDBC_TEMPLATE.proxyQueryForList(SIMPLE_LIST_SQL,sqlParams);
}
```
##### 新写法
使用@Query注解进行列表查询语句执行，结果返回匹配的行。
```java
@Query("""
        SELECT  id, user_id, kuaidinum, kuaidicom, send_name, 
                send_mobile, send_country, send_province, send_city, send_district, 
                send_address, rec_name, rec_mobile, rec_country, rec_province,  
                rec_city, rec_district, rec_address, create_time, update_time 
        FROM t_order
        WHERE rec_mobile = #recMobile# 
        LIMIT #pageSize# OFFSET #offset# 
        """)
public List<Order> findByRecMobile(@Param("recMobile") String recMobile, @Param("pageSize") Integer pageSize,
                                   @Param("offset") Long offset) {
    return ProxySQLResult.ok();
}
```
##### <a href= "#info">占位符说明<a>
##### <a href= "#zuijia">最佳实践<a>

### 查询单条记录
##### 旧写法
使用`JDBC_TEMPLATE.proxyQueryForEntity`方法执行查询语句，结果返回对应的查询结果，只会返回单条记录。  
查询语句中待替换的参数使用`?`占位，定义`SQLParams`来进行参数填充。注意`SQLParams`添加参数的顺序要与`SQL`中参数占位符的顺序一致。
```java
public static final String SIMPLE_SQL = "SELECT FEXPID,FDBID,FTRADETIME,FSOURCE FROM T_MKT_EXPRESS WHERE FEXPID = ?";
//添加@AutoMapping注解后会自动映射结果。
@AutoMapping
public SimpleResponse querySimpleResponse(Long id) {
    SQLParams sqlParams = new SQLParams(1);
    sqlParams.put(id);
    return JDBC_TEMPLATE.proxyQueryForEntity(SIMPLE_SQL, sqlParams);
}
``` 
##### 新写法
```java
@Query("""
        SELECT  id, user_id, kuaidinum, kuaidicom, send_name, 
            send_mobile, send_country, send_province, send_city, send_district, 
            send_address, rec_name, rec_mobile, rec_country, rec_province,  
            rec_city, rec_district, rec_address, create_time, update_time 
        FROM t_order 
        WHERE rec_mobile = #recMobile#  
        LIMIT 1 
        """)
public Order findByRecMobileOne(@Param("recMobile") String recMobile) {
    return ProxySQLResult.ok();
}
```
##### <a href= "#info">占位符说明<a>
##### <a href= "#zuijia">最佳实践<a>

### 执行SQL以及参数打印
日志配置文件添加打印`com.superxiaobailong.jdbc.query`路径日志输出级别为`DEBUG`。
```
<logger name="com.superxiaobailong.jdbc.query" level="debug"></logger>
```

### 游标遍历
对于需要做全表扫描或者大量数据获取的场景建议使用游标进行数据遍历。在`Service`层指定处理游标返回的每行数据的处理逻辑，在`Dao`层使用`JDBC_TEMPLATE.proxyHandlerCursor`进行游标遍历。   
如果不执行`stopRunner`，则游标会使用用户指定的函数处理所有满足条件的行。如果需要提前结束游标遍历，可以执行`stopRunner`方法。

```java
//service层方法
// 处理游标查询数据的handler
public void cursorService() {
    BiConsumer<Order, Runnable> handler = (order, stopRunner) -> {
        // todo根据返回的结果进行业务逻辑处理
        
        ...
        
        // 提前终止游标
        if (如果需要提前终止游标) {
            //关闭游标
            stopRunner.run();
        }
    };
    orderRepository.listOrderByRecMobile(recMobile, handler);
}

```   

```java
//dao层方法
@AutoMapping
public void listOrderByRecMobile(long recMobile,  BiConsumer<Order,Runnable> handler) {
    // sql
    String sql = """
            SELECT  id, user_id, kuaidinum, kuaidicom, send_name,
                    send_mobile, send_country, send_province, send_city, send_district,
                    send_address, rec_name, rec_mobile, rec_country, rec_province,
                    rec_city, rec_district, rec_address, create_time, update_time
            FROM t_order
            WHERE rec_mobile = ?
            """;
    // 参数
    SQLParams sqlParams = new SQLParams(1);
    sqlParams.put(recMobile);
    //使用默认的超时时间（5s）默认的buffer(100)
    JDBC_TEMPLATE.proxyHandlerCursor(sql, sqlParams, handler);
    //使用指定的超时时间和buffer大小
    JDBC_TEMPLATE.proxyHandlerCursor( sql,  sqlParams,  handler, 100, Duration.ofSecond(10)); 
}
```
⚠️**游标的默认超时时间为5秒**
#### 游标连接池说明
为了避免游标执行时间过长影响到正常业务，游标默认和普通业务使用不同的数据库连接池。游标的默认连接池大小为1，可以通过启动参数进行配置。
```
//游标数据库连接池大小配置
-Dkd.jdbc.cursorConnectionPoolSize = 2
```

### 事务
支持编程式事务和声明式事务两种方式。
#### 编程式事务
使用编程式事务步骤如下
- 获取事务管理器`TransactionManager`
- 事务属性定义`TransactionDefinition`
- 通过之前获取的事务管理器和定义的事务属性来创建事务模板实例`TransactionTemplate`
- 执行模板`TransactionTemplate`

##### 编程式事务使用示例
```java
public void testCodeTransaction() {
    //定义了一个事务传播为`REQUIRED`，事务隔离级别为`READ_COMMITTED`，事务超时时间为1秒的只读事务
    TransactionDefinition transactionDefinition = new DefaultTransactionDefinition(Propagation.REQUIRED,
                Isolation.READ_COMMITTED, Duration.ofSeconds(1), TransactionTypeEnum.READ_ONLY);
    //获取默认事务管理器
    TransactionManager transactionManager = Transactions.getDefaultTransactionManager();
    //创建事务模板
    TransactionTemplate transactionTemplate = new TransactionTemplate(transactionManager,
            transactionDefinition);
    //执行需要在事务中的逻辑
    transactionTemplate.execute(() -> transaction());
}

public Object transaction() {
    //需要在事务中执行的逻辑
    return null;
}
```
#### 声明式事务
在方法上添加`@Transactional`注解，该方法的逻辑就会在事务中执行。       
`@Transactional`注解一共有四个可选的配置参数：`事务传播机制`,`事务隔离级别`,`事务超时时间`,`是否为只读事务`。
##### 声明式事务使用示例
```java
//声明事务传播为`REQUIRED`，事务隔离级别为`READ_COMMITTED`，事务超时时间为5秒的只读事务
@Transactional(isolation = Isolation.READ_COMMITTED,propagation = Propagation.REQUIRED,
        timeout = 5, readOnly = true)
public void testInTransactionTwoInsert() {
    //事务逻辑
}
```



##### 与Spring声明式事务的区别
本框架中private方法，同类方法之间的调用事务也可以生效。   
Spring默认事务没有超时时间，框架默认事务超时时间为5秒。


#### 事务定义说明
##### 事务传播机制说明
支持两种事务传播机制`REQUIRED`和`NESTED`，默认值为`REQUIRED`。对应枚举类为`com.superxiaobailong.jdbc.transaction.Propagation`。   
`REQUIRED`:表示必须在事务中执行。如果当前不存在事务，会创建一个新事务。如果存在事务，则会沿用该事务。     
`NESTED`:表示嵌套事务。如果当前不存在事务，会创建一个新事务。如果存在事务，则会创建一个嵌套事务。嵌套事务回滚不会影响外部事务回滚。
##### 事务隔离级别说明
事务隔离级别的默认值为`DEFAULT`。对应的枚举类为`com.superxiaobailong.jdbc.transaction.Isolation`。
- 使用数据库默认的隔离级别(DEFAULT)。
- 读未提交(READ_UNCOMMITTED)
- 读已提交(READ_COMMITTED)
- 可重复读(REPEATABLE_READ)
- 序列化(SERIALIZABLE)

##### 只读事务说明
如果接口只涉及数据库查询，没有变更数据库数据，则建议将事务设置为只读事务。

##### 事务超时时间说明
事务默认超时时间5秒，可以手动设置指定的值。

##### 事务定义生效顺序说明
```java
@Transactional(isolation = Isolation.READ_COMMITTED, propagation = Propagation.REQUIRED, timeout = 5, readOnly = true)
public void transactionA() {
    //事务逻辑
    transactionB();
}

@Transactional(isolation = Isolation.REPEATABLE_READ, propagation = Propagation.REQUIRED, timeout = 2, readOnly = false)
public void transactionB() {
    //事务逻辑
}
```
上面这段代码，调用`transactionA`方法然后在`transactionA`中调用有事务注解的`transactionB`方法，但是方法`transactionA`与`transactionB`事务的定义不完全相同。这种情况下，框架会使用最外层`transactionA`的事务定义来创建事务。

#### 🙁事务不推荐用法
在事务中发起远程调用（HTTP等）。在事务中发起远程调用，框架会抛出异常。

⚠️注意：使用事务时，对应类的单例写法固定为下面的方式。需要有一个无参的构造函数。
```java
private final static BusinessExpressService INSTANCE = new BusinessExpressService();
public static BusinessExpressService getInstance() {
    return INSTANCE;
}
```










 




































