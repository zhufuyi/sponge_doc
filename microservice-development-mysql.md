
`⓶基于sql创建grpc服务`创建一个开发到部署的完整grpc服务代码，支持在grpc服务代码中批量添加标准化CRUD api接口代码而不需要编写任何一行go代码，实现api接口"低代码开发"；支持在grpc服务代码基础上二次开发，例如添加自定义api接口，只需在proto文件定义api接口，然后在生成的api接口模板编写业务逻辑代码。

因此`⓶基于sql创建grpc服务`sponge适合开发已选定数据库类型的微服务项目。

生成grpc服务代码时支持基于数据库 mysql、postgresql、tidb、sqlite，下面操作以mysql为例介绍grpc服务开发步骤，选用其他数据库类型的开发步骤一样。

<br>

### 🏷前期准备

开发grpc服务项目前准备：

- 已安装sponge
- mysql服务
- mysql表

> [!tip] 生成代码需要依赖mysql服务和mysql表，如果都没有准备好，这里有[docker启动mysql服务脚本](https://github.com/zhufuyi/sponge/blob/main/test/server/mysql/docker-compose.yaml)，启动mysql服务之后导入示例使用的[库和表的sql](https://github.com/zhufuyi/sponge_examples/blob/main/1_web-gin-CRUD/test/sql/user.sql)。

打开终端，启动sponge UI界面服务：

```bash
sponge run
```

在浏览器访问 http://localhost:24631 ，进入sponge生成代码的UI界面。

<br>

### 🏷创建grpc服务项目

进入sponge的UI界面：

1. 点击左边菜单栏【SQL】-->【创建grpc服务】；
2. 选择数据库`mysql`，填写`数据库dsn`，然后点击按钮`获取表名`，选择表名(可多选)；
3. 填写其他参数，鼠标放在问号`?`位置可以查看参数说明；

填写完参数后，点击按钮`下载代码`生成grpc服务完整项目代码，如下图所示：

![micro-rpc](assets/images/micro-rpc.png)

> [!tip] 等价命令 **sponge micro rpc --module-name=user --server-name=user --project-name=edusys --db-driver=mysql --db-dsn="root:123456@(192.168.3.37:3306)/school" --db-table=teacher**

> [!tip] 解压的grpc服务代码目录名称的格式是`服务名称-类型-时间`，可以修改目录名称(例如把名称中的类型和时间去掉)。

> [!tip] 成功生成代码之后会保存记录，方便下一次生成代码使用，如果`mysql dsn地址`不变，刷新或重新打开页面时会自动获取到表名，不需要点击按钮`获取表名`就可以直接选择表名。

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
│   ├─ cache
│   ├─ config
│   ├─ dao
│   ├─ ecode
│   ├─ model
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

如果没有`goland` IDE，也可以通过命令来测试，切换到`internal/service`目录，打开后缀为`_client_test.go`的测试文件，修改grpc方法的请求参数，执行测试命令`go test -run 测试函数名/grpc方法名`，示例： `go test -run Test_service_teacher_methods/GetByID`。

> [!tip] CRUD api 接口中有一个任意条件分页查询接口，有了这个接口后，可以少写很多api查询接口，点击查看<a href="/zh-cn/public-doc?id=%f0%9f%94%b9%e4%bb%bb%e6%84%8f%e6%9d%a1%e4%bb%b6%e5%88%86%e9%a1%b5%e6%9f%a5%e8%af%a2" target="_blank">任意条件分页查询接口的使用规则</a>。

<br>

### 🏷自动添加CRUD api接口

如果有新的mysql表需要生成CRUD api接口代码，

1. 点击左边菜单栏【Public】-->【生成service CRUD代码】；
2. 选择数据库`mysql`，填写`数据库dsn`，然后点击按钮`获取表名`，选择mysql表(可多选)；
3. 填写其他参数。

填写完参数后，点击按钮`下载代码`生成service CRUD代码，如下图所示：

![micro-rpc-service](assets/images/micro-rpc-service.png)

> [!tip] 等价命令 **sponge micro service --module-name=user --db-driver=mysql --db-dsn="root:123456@(192.168.3.37:3306)/school" --db-table=cause,teach**，有更简单的等价命令，使用参数`--out`指定user服务代码目录，直接合并代码到user服务代码，**sponge micro service --db-driver=mysql --db-dsn="root:123456@(192.168.3.37:3306)/school" --db-table=cause,teach --out=user**

生成的service CRUD代码目录如下，在目录`internal`和`api/user`下的子目录`cache`、`dao`、`ecode`、`model`、`service`、`v1`包含了以表名开头的go文件和测试文件。

```
.
├─ api
│   └─ user
│       └─ v1
└─ internal
     ├─ cache
     ├─ dao
     ├─ ecode
     ├─ model
     └─ service
```

解压代码，把目录`internal`和`api`移动到grpc服务代码目录下，就完成了在grpc服务中批量添加service CURD api接口。

> [!note] 移动目录`internal`和`api`正常情况下不会有冲突文件，如果有冲突文件，说明之前已经指定相同的mysql表来生成service CRUD代码了，此时忽略覆盖文件。

在终端执行命令：

```bash
# 生成与合并api接口相关代码
make proto

# 编译和运行服务
make run
```

> [!attention] 如果执行命令`make proto`时，报错 **api/types/types.proto: File not found.** 或 **internal\cache\xxx.go:40:38: undefined: model.CacheType**，请执行命令`make patch TYPE=init-mysql`，如果使用其他数据库类型，把命令中的mysql改为对应数据库类型名称。

同样的使用`goland`IDE打开项目代码，进入目录`internal/service`，打开后缀为`_client_test.go`测试文件，测试之前先填写请求参数。如果不使用`goland`IDE来测试，也可以在终端执行测试命令`go test -run 测试函数名/grpc方法名`。

<br>

批量添加标准化的CURD api接口代码到grpc服务项目代码中，不需要人工编写任何go代码。

<br>

### 🏷人工添加自定义api接口

项目中通常不止有标准化的CRUD api接口，还有自定义的api接口，添加自定义api接口的主要流程是`在proto文件定义api接口` --> `在生成的模板代码中编写业务逻辑代码`。

例如在本项目中添加一个登录接口步骤：

**(1) 在proto文件定义登录接口**

进入目录`api/user/v1`目录，打开文件`teacher.proto`，添加登录接口的描述信息：

```protobuf
import "validate/validate.proto";

service teacher {
  // ...

  // 登录
  rpc Login(LoginRequest) returns (LoginReply) {}
}

message LoginRequest {
  string email = 1 [(validate.rules).string.email = true];
  string password = 2 [(validate.rules).string.min_len = 6];
}

message LoginReply {
  uint64 id = 1;
  string token = 2;
}
```

> [!tip] 字段email和password后面的`validate.rules`是字段校验规则，点击查看更多[validate校验规则](https://github.com/envoyproxy/protoc-gen-validate#constraint-rules)，记得在proto文件添加 import "validate/validate.proto"。


添加api接口描述信息后，在终端执行命令：

```bash
# 生成与合并api接口相关代码
make proto
```

<br>

**(2) 编写业务逻辑代码**

进入目录`internal/service`，打开文件`teacher.go`，在Login方法函数下编写业务逻辑代码，例如验证密码、生成token等。

> [!tip] 在人工添加的自定义api接口中，可能需要对数据增删改查操作(也叫dao CRUD)，这些dao CRUD代码可以直接生成，不需要人工编写，点击查看<a href="/zh-cn/public-doc?id=%f0%9f%94%b9%e7%94%9f%e6%88%90%e5%92%8c%e4%bd%bf%e7%94%a8dao-crud%e4%bb%a3%e7%a0%81" target="_blank">生成和使用dao CRUD代码说明</a>。

> [!tip] 在人工添加的自定义api接口中，可能需要用到缓存，例如生成的token，这类string类型缓存代码可以直接生成，不需要人工编写，点击查看<a href="/zh-cn/public-doc?id=%f0%9f%94%b9%e7%94%9f%e6%88%90%e5%92%8c%e4%bd%bf%e7%94%a8cache%e4%bb%a3%e7%a0%81" target="_blank">生成和使用cache代码说明</a>。

<br>

**(3) 测试api接口**

编写完业务逻辑代码后，在终端执行命令：

```bash
# 编译和运行服务
make run
```

使用`goland`IDE，进入目录`internal/service`，打开后缀为`_client_test.go`测试文件(这里是teacher_client_test.go)，填写Login请求参数后，点击左边绿色按钮进行测试。

如果不使用`goland`IDE来测试，可以在终端执行测试命令`go test -run 测试函数名/grpc方法名`。

<br>

添加一个自定义api接口比较简单，只需在proto文件定义api接口描述信息，然后在生成的模板填写业务逻辑代码，grpc客户端测试代码是自动生成的，不需要借助第三方grpc客户端来测试，不需要像传统grpc开发那样人工编写完整api接口代码，让开发者专注在编写业务逻辑。

<br>

### 🏷调用其他grpc服务api接口

根据项目业务需求，有可能需要在本服务调用其他grpc服务的api接口，这里的其他grpc服务是指使用protobuf协议的grpc服务，包括其他语言实现的grpc服务，下面是调用其他grpc服务api接口的操作步骤：

**(1) 添加连接目标grpc服务代码**

如果想要在本服务内调用目标grpc服务api接口，首先要能够连接上目标grpc服务，下面自动生成grpc连接代码。

进入sponge的UI界面，点击左边菜单栏【Public】-->【生成grpc服务连接代码】，填写module名称，填写grpc服务名称(支持多个grpc服务名称，用逗号分隔)，填写完参数后，点击按钮`下载代码`生成grpc服务连接代码，如下图所示：

![micro-rpc-conn](assets/images/micro-rpc-conn.png)

> [!tip] 等价命令 **sponge micro rpc-conn --module-name=edusys  --rpc-server-name=user**。有更简单的等价命令，使用参数`--out`指定grpc服务代码目录，直接合并代码到本服务代码，**sponge micro rpc-conn --rpc-server-name=user --out=edusys**

生成的grpc服务连接代码目录如下：

```
.
└─ internal
    └─ rpcclient
```

> [!tip] grpc连接代码其实是grpc客户端连接代码，包括了服务发现、负载均衡、安全连接、链路跟踪、指标采集等设置，也可以添加自己定义的连接设置。

解压代码，把目录`internal`移动到本服务代码目录下。

> [!note] 移动目录`internal`到本服务目录下正常情况下不会有冲突文件，如果有冲突文件，说明之前已经指定相同的grpc服务名称来生成grpc服务连接代码了，此时忽略覆盖文件。

<br>

**(2) 配置目标grpc服务的地址**

添加连接目标grpc服务代码之后，在配置文件`configs/服务名称.yml`设置连接目标grpc服务的地址，主要配置内容如下：

```yaml
grpcClient:
  - name: "user"        # grpc服务名称
    host: "127.0.0.1"   # grpc服务地址，如果开启服务发现，此字段值无效
    port: 8282          # grpc服务端口，如果开启服务发现，此字段值无效
    registryDiscoveryType: ""  # 服务发现，默认关闭，支持consul, etcd, nacos
```

> [!tip] 更多grpcClient设置看`configs/服务名称.yml`，例如负载均衡、安全连接等。

如果连接多个grpc服务，需要设置多个grpc服务的地址，示例如下：

```yaml
grpcClient:
  - name: "user"
    host: "127.0.0.1"
    port: 18282
    registryDiscoveryType: ""
  - name: "relation"
    host: "127.0.0.1"
    port: 28282
    registryDiscoveryType: ""
  - name: "creation"
    host: "127.0.0.1"
    port: 38282
    registryDiscoveryType: ""
```

<br>

**(3) 复制目标grpc服务的proto文件**

虽然可以连接到目标grpc服务，但是不知道目标grpc服务哪些api接口可以调用，通过proto文件可以告诉本服务可以调用的api接口。

把目标grpc服务的proto文件复制出来，并移动到本服务的目录`api/目标grpc服务名称/v1`下。有了目标grpc服务proto文件，在本服务就可以知道有哪些api接口可以调用了。

如果目标grpc服务是使用sponge创建的，可以直接使用命令复制proto文件，非sponge创建的grpc服务则需要手动复制proto文件。

```bash
# 从其他grpc服务中复制proto文件到本服务项目中，如果有多个目标grpc服务目录，用逗号分隔
make copy-proto SERVER=../user

# 执行命令会把proto文件复制到本服务目录 api/目标grpc服务名称/v1 下。
```

> [!note] `make copy-proto`会把所有proto文件复制过来，如果proto文件存在，会覆盖proto文件，可以在目录`/tmp/sponge_copy_backup_proto_files`下找到覆盖前的备份proto文件。

<br>

**(4) 运行目标grpc服务**

如果使用sponge创建的grpc服务，执行命令`make run`运行目标grpc服务，如果不是sponge创建的grpc服务，请根据实际命令启动grpc服务。

<br>

### 🏷设置服务

创建的grpc服务代码中包含了丰富的组件，有些组件默认是关闭的，根据实际需要开启使用，统一在配置文件`configs/服务名称.yml`进行设置，配置文件里有详细说明。

> [!tip] 可以在服务代码中替换、添加自己的组件(grpc interceptor)，或者删除不需要的组件，在代码文件`internal/server/grpc.go`修改。

**默认开启的组件：**

- **logger**：日志组件，默认是输出到终端，默认输出日志格式是console，可以设置输出格式为json，设置日志保存到指定文件，日志文件切割和保留时间。
- **enableMetrics**：指标采集，默认路由`/metrics`。
- **enableStat**：资源统计，统计系统和本程序的cpu和内存资源使用信息，默认每分钟在日志打印一次，如果本程序占用系统资源持续超过80%(可设置)，后台自动采集profile保存到目录`/tmp/服务名称_profile`，可以后续进行离线分析。
- **cacheType**：缓存组件，默认是本地内存，可以改为redis，注意集群部署时必须使用redis。

**默认关闭的组件：**

- **enableHTTPProfile**：profile组件
- **enableLimit**：自适应限流组件
- **enableCircuitBreaker**：自适应熔断组件
- **enableTrace**：链路跟踪组件
- **registryDiscoveryType**：服务注册与发现组件
- **grpc安全：**
  - **serverSecure**：通过证书验证，支持服务端验证和双向验证
  - **enableToken**：通过token验证

其他配置的可以根据需要设置，也可以添加配置，如果添加或更改配置文件字段，需要更新对应的go结构体，在服务代码目录下的终端执行命令：

```bash
make update-config
```

---

相关视频介绍：

- [一键生成grpc服务完整项目代码](https://www.bilibili.com/video/BV1Tg4y1b79U/)
- [批量生成CRUD代码到grpc服务](https://www.bilibili.com/video/BV1TY411z7rY/)
