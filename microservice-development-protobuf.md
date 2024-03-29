
`⓸基于protobuf创建grpc服务`创建通用的grpc服务代码，可以选用任意的数据库和orm，也可以选用sponge默认支持的数据库类型和orm。而`⓶基于sql创建grpc服务`(<a href="/zh-cn/microservice-development-sql" target="_blank">grpc服务开发(sql)</a>)只支持mysql、mongodb、postgresql、tidb、sqlite数据库类型，这是两种方式创建grpc服务的主要区别，可以把`⓶基于sql创建grpc服务`看作是`⓸基于protobuf创建grpc服务`一个子集。

`⓸基于protobuf创建grpc服务`, 如果选用其他数据库类型(非sponge支持)，需要人工编写dao、model、数据库初始化等代码，不支持自动生成；如果选用sponge支持的数据，则可以自动生成dao、model、数据库初始化等代码。

`⓸基于protobuf创建grpc服务`适合通用的grpc服务项目开发。

<br>

## 🏷选用mysql进行grpc服务开发

### 🔹前期准备

开发grpc服务项目前准备：

- 已安装sponge
- mysql服务
- mysql表
- proto文件，例如[user.proto](https://github.com/zhufuyi/sponge_examples/blob/main/2_web-gin-protobuf/api/user/v1/user.proto)。

> [!tip] 生成service CRUD代码需要依赖mysql服务和mysql表，如果都没有准备好，这里有[docker启动mysql服务脚本](https://github.com/zhufuyi/sponge/blob/main/test/server/mysql/docker-compose.yaml)，启动mysql服务之后导入示例使用的[库和表的sql](https://github.com/zhufuyi/sponge_examples/blob/main/1_web-gin-CRUD/test/sql/user.sql)。

打开终端，启动sponge UI界面服务：

```bash
sponge run
```

在浏览器访问 http://localhost:24631 ，进入sponge生成代码的UI界面。

<br>

### 🔹创建grpc服务

进入sponge的UI界面：

1. 点击左边菜单栏【Protobuf】-->【创建grpc服务】；
2. 选择proto文件(可多选)；
3. 接着填写其他参数，鼠标放在问号`?`位置可以查看参数说明。

填写完参数后，点击按钮`下载代码`生成grpc服务项目代码，如下图所示：

![micro-rpc-pb](assets/images/micro-rpc-pb.png)

> [!tip] 等价命令 **sponge micro rpc-pb --module-name=user --server-name=user --project-name=edusys --protobuf-file=./user.proto**

> [!tip] 解压的grpc服务代码目录名称的格式是`服务名称-类型-时间`，可以修改目录名称(例如把名称中的类型和时间去掉)。

> [!tip] 成功生成代码之后会保存记录，方便下一次生成代码使用，刷新或重新打开页面时显示上一次部分参数。

这是创建的grpc服务代码目录：

```
.
├─ api
│   ├─ types
│   └─ user
│       └─ v1
├─ cmd
│   └─ user
│       ├─ initial
│       └─ main.go
├─ configs
├─ deployments
│   ├─ binary
│   ├─ docker-compose
│   └─ kubernetes
├─ docs
├─ internal
│   ├─ config
│   ├─ ecode
│   ├─ server
│   └─ service
└─ scripts
```

创建的grpc服务代码结构鸡蛋模型：

![micro-rpc-pb-anatomy](assets/images/micro-rpc-pb-anatomy.png)

解压代码文件，打开终端，切换到grpc服务代码目录，执行命令：

```bash
# 生成与合并api接口相关代码
make proto

# 编译和运行服务
make run
```

> [!note] 开发过程中会经常使用 `make proto`命令，内部执行一系列生成代码子命令：生成api接口的`模板代码`、`错误码`、`grpc客户端测试代码`、`相关的*.pb.go`，`自动合并api接口相关代码`。合并代码时不用担心覆盖已编写业务逻辑代码问题，就算出现意外(断电)，可以在 `/tmp/sponge_merge_backup_code` 目录下找到每次合并前的备份代码，如果是windows环境则存放在 `C:\Users\你的用户名\AppData\Local\Temp\sponge_merge_backup_code`。如果在proto文件添加或更新了api接口描述信息，需要执行这个命令，否则不需要执行。

使用`goland`IDE打开项目代码，进入目录`internal/service`，打开后缀为`_client_test.go`的测试文件，这里包括了在proto文件定义的每个api接口的测试和性能压测函数，测试前先填写请求参数，类似在swagger界面测试api接口，如下图所示：

![micro-rpc-test](assets/images/micro-rpc-test.png)

如果没有`goland` IDE，也可以通过命令来测试，切换到`internal/service`，打开后缀为`_client_test.go`的测试文件，修改grpc方法的请求参数，执行测试命令`go test -run 测试函数名/grpc方法名`，示例： `go test -run Test_service_teacher_methods/GetByID`。

<br>

### 🔹自动添加CRUD api接口

自动添加CRUD api接口与`grpc服务开发(sql)`章节中`自动添加CRUD api接口`一样，点击查看
<a href="/zh-cn/microservice-development-sql?id=%f0%9f%8f%b7%e8%87%aa%e5%8a%a8%e6%b7%bb%e5%8a%a0crud-api%e6%8e%a5%e5%8f%a3" target="_blank">自动添加CRUD api接口文档</a>。


> [!tip] CRUD api 接口中有一个任意条件分页查询接口，有了这个接口后，可以少写很多api查询接口，点击查看<a href="/zh-cn/public-doc?id=%f0%9f%94%b9%e4%bb%bb%e6%84%8f%e6%9d%a1%e4%bb%b6%e5%88%86%e9%a1%b5%e6%9f%a5%e8%af%a2" target="_blank">任意条件分页查询接口的使用规则</a>。

<br>

### 🔹人工添加自定义api接口

人工添加自定义api接口与`grpc服务开发(sql)`章节中`人工添加自定义api接口`一样，点击查看
<a href="/zh-cn/microservice-development-sql?id=%f0%9f%8f%b7%e4%ba%ba%e5%b7%a5%e6%b7%bb%e5%8a%a0%e8%87%aa%e5%ae%9a%e4%b9%89api%e6%8e%a5%e5%8f%a3" target="_blank">人工添加自定义api接口文档</a>。

<br>

### 🔹调用其他grpc服务api接口

调用其他grpc服务api接口与`grpc服务开发(sql)`章节中`调用其他grpc服务api接口`一样，点击查看

<a href="/zh-cn/microservice-development-sql?id=%f0%9f%8f%b7%e8%b0%83%e7%94%a8%e5%85%b6%e4%bb%96grpc%e6%9c%8d%e5%8a%a1api%e6%8e%a5%e5%8f%a3" target="_blank">调用其他grpc服务api接口文档</a>。

<br>

### 🔹设置服务

设置服务与`grpc服务开发(sql)`章节中`设置服务`一样，点击查看
<a href="/zh-cn/microservice-development-sql?id=%f0%9f%8f%b7%e8%ae%be%e7%bd%ae%e6%9c%8d%e5%8a%a1" target="_blank">设置服务文档</a>。

<br>

## 🏷选用其他数据库进行grpc服务开发

`⓸基于protobuf创建grpc服务`默认不包括操作数据库相关代码，可以选用任何数据库类型作为数据存储，上面介绍了选用mysql(sponge支持的数据库类型mysql、mongodb、postgresql、tidb、sqlite)进行web开发的具体过程，操作起来简单方便。

虽然sponge不支持通过其他数据库类型来生成操作数据库相关代码，但基于proto文件生成`api接口模板代码`、`grpc客户端测试代码`、`错误码`、`自动合并模板代码`等，减少了不少原本需要人工编写的代码，比传统的grpc服务开发更简单方便。

`选用其他数据库类型`与`选用mysql`的grpc服务开发流程基本一样，没什么区别，最大的不同点是数据库操作相关代码，前者是人工编写，后者是自动生成。

<br>

### 🔹前期准备

开发grpc服务项目前准备：

- 已安装sponge
- 数据库服务
- proto文件，例如[user.proto](https://github.com/zhufuyi/sponge_examples/blob/main/2_web-gin-protobuf/api/user/v1/user.proto)。

打开终端，启动sponge UI界面服务：

```bash
sponge run
```

在浏览器访问 http://localhost:24631 ，进入sponge生成代码的UI界面。

<br>

### 🔹创建grpc服务

请看上面章节 <a href="/zh-cn/microservice-development-protobuf?id=%f0%9f%94%b9%e5%88%9b%e5%bb%bagrpc%e6%9c%8d%e5%8a%a1" target="_blank">创建grpc服务文档</a>。

<br>

### 🔹初始化数据库

**(1) 添加数据库配置**

打开配置文件`configs/服务名称.yml`，添加数据地址配置，示例：

```yml
# elasticsearch settings
elasticsearch:
  addr: "http://localhost:9200"
```

打开终端，切换到服务目录下，执行命令更新配置go结构体代码：

```bash
make update-config
```

<br>

**(2) 添加初始化数据代码**

在目录`internal/model`下新建文件`init.go` ，在这里填写连接数据代码，可以参考[mysql初始化](https://github.com/zhufuyi/sponge/blob/main/internal/model/init.go)。

然后进入目录`cmd/服务名称/initial`，打开`initApp.go`，替换默认注释的mysql和cache初始化代码。同时打开`registerClose.go`，替换默认注释的mysql和cache关闭连接代码。

<br>

### 🔹添加自定义api接口

> [!note] 在api接口模板代码编写业务逻辑代码时，如果涉及到对数据操作，需要人工编写`model`、`dao`等代码。

添加自定义api接口与`grpc服务开发(sql)`章节中`人工添加自定义api接口`一样，点击查看
<a href="/zh-cn/microservice-development-sql?id=%f0%9f%8f%b7%e4%ba%ba%e5%b7%a5%e6%b7%bb%e5%8a%a0%e8%87%aa%e5%ae%9a%e4%b9%89api%e6%8e%a5%e5%8f%a3" target="_blank">人工添加自定义api接口文档</a>。

> [!tip] 在人工添加的自定义api接口中，可能需要用到缓存，例如生成的token，这类string类型缓存代码可以直接生成，不需要人工编写，点击查看<a href="/zh-cn/public-doc?id=%f0%9f%94%b9%e7%94%9f%e6%88%90%e5%92%8c%e4%bd%bf%e7%94%a8cache%e4%bb%a3%e7%a0%81" target="_blank">生成和使用cache代码说明</a>。

<br>

### 🔹调用其他grpc服务api接口

调用其他grpc服务api接口与`grpc服务开发(sql)`章节中`调用其他grpc服务api接口`一样，点击查看

<a href="/zh-cn/microservice-development-sql?id=%f0%9f%8f%b7%e8%b0%83%e7%94%a8%e5%85%b6%e4%bb%96grpc%e6%9c%8d%e5%8a%a1api%e6%8e%a5%e5%8f%a3" target="_blank">调用其他grpc服务api接口文档</a>。

<br>

### 🔹设置服务

设置服务与`grpc服务开发(sql)`章节中`设置服务`一样，点击查看
<a href="/zh-cn/microservice-development-sql?id=%f0%9f%8f%b7%e8%ae%be%e7%bd%ae%e6%9c%8d%e5%8a%a1" target="_blank">设置服务文档</a>。

<br>

---

相关视频介绍：

- [一键生成通用grpc服务项目代码](https://www.bilibili.com/video/BV1WY4y1X7zH/)
- [批量生成任意API接口代码到grpc服务](https://www.bilibili.com/video/BV1Yo4y1q76o/)
