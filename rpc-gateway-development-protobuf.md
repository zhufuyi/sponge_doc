grpc网关是grpc服务的统一入口，它可以为客户端提供负载均衡、路由、安全、监控等功能，从而提高微服务的性能、可用性和安全性。

**主要作用：**

- 通常位于客户端和grpc服务之间，它可以将客户端的请求均匀地分发到多个grpc服务，从而提高微服务的吞吐量和可用性。
- 根据客户端的请求，将请求转发到相应的grpc服务，从而简化客户端的开发。
- 提供身份认证、授权、加密等安全功能，从而保护grpc服务的安全。
- 收集grpc服务的运行状态信息，并提供监控功能，从而帮助用户管理grpc服务。

**适用场景：**

- 在微服务架构中，grpc网关可以作为微服务的统一入口，简化客户端的开发，并提高微服务的扩展性和可维护性。
- 在分布式系统中，grpc网关可以提供负载均衡、路由、安全等功能，从而提高分布式系统的性能和可用性。
- 在跨平台系统种，grpc网关可以支持不同平台的grpc服务，从而帮助用户构建跨平台的grpc系统。

这里`⓹基于protobuf创建grpc网关服务`是grpc服务的统一入口的web服务，下面介绍grpc网关服务开发具体流程。

<br>

### 🏷前期准备

开发grpc网关服务前准备：

- 已安装sponge
- proto文件，例如[user_gw.proto](https://github.com/zhufuyi/sponge_examples/blob/main/5_micro-gin-rpc-gateway/user-gateway/api/user_gw/v1/user_gw.proto)。
- grpc服务[user](https://github.com/zhufuyi/sponge_examples/tree/main/4_micro-grpc-protobuf)，也可以临时快速创建一个grpc服务，点击查看<a href="/zh-cn/microservice-development-protobuf?id=%f0%9f%94%b9%e5%88%9b%e5%bb%bagrpc%e6%9c%8d%e5%8a%a1" target="_blank">创建grpc服务文档</a>。

打开终端，启动sponge UI界面服务：

```bash
sponge run
```

在浏览器访问 http://localhost:24631 ，进入sponge生成代码的UI界面。

<br>

### 🏷创建grpc网关服务项目

进入sponge的UI界面，点击左边菜单栏【Protobuf】-->【创建grpc网关项目】，选择proto文件(可多选)，接着填写其他参数，鼠标放在问号`?`位置可以查看参数说明，填写完参数后，点击按钮`下载代码`生成grpc网关服务项目代码，如下图所示：

![micro-rpc-gw-pb](assets/images/micro-rpc-gw-pb.png)

> [!tip] 等价命令 **sponge micro rpc-gw-pb --module-name=edusys --server-name=edusys --project-name=edusys --protobuf-file=./user_gw.proto**

> [!tip] 解压的grpc网关服务代码目录名称的格式是`服务名称-类型-时间`，可以修改目录名称(例如把名称中的类型和时间去掉)。

> [!tip] 成功生成代码之后会保存记录，方便下一次生成代码使用，刷新或重新打开页面时显示上一次部分参数。

解压代码文件，这是创建的grpc网关服务代码目录：

```
.
├─ api
│   └─ edusys
│       └─ v1
├─ cmd
│   └─ edusys
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
│   ├─ routers
│   ├─ server
│   └─ service
└─ scripts
```

创建的grpc网关服务代码结构鸡蛋模型：

![micro-rpc-gw-pb-anatomy](assets/images/micro-rpc-gw-pb-anatomy.png)


<br>

### 🏷对接grpc服务相关操作

#### 🔹添加连接grpc服务代码

想要在grpc网关服务里调用grpc服务api接口，首先要能够连接上grpc服务，下面自动生成grpc服务连接代码。

进入sponge的UI界面，点击左边菜单栏【Public】-->【生成grpc服务连接代码】，填写module名称，填写grpc服务名称(支持多个grpc服务名称，用逗号分隔)，填写完参数后，点击按钮`下载代码`生成grpc服务连接代码，如下图所示：

![micro-rpc-conn](assets/images/micro-rpc-conn.png)

> [!tip] 等价命令 **sponge micro rpc-conn --module-name=edusys  --rpc-server-name=user**。有更简单的等价命令，使用参数`--out`指定grpc网关服务代码目录，直接合并代码到grpc网关服务代码，**sponge micro rpc-conn --rpc-server-name=user --out=edusys**

生成的grpc服务连接代码目录如下：

```
.
└─ internal
    └─ rpcclient
```

> [!tip] grpc连接代码其实是grpc客户端连接代码，包括了服务发现、负载均衡、安全连接、链路跟踪、指标采集等设置，也可以添加自己定义的连接设置。

解压代码，把目录`internal`移动到grpc网关服务代码目录下。

> [!note] 移动目录`internal`到grpc服务目录下正常情况下不会有冲突文件，如果有冲突文件，说明之前已经指定相同的grpc服务名称来生成grpc服务连接代码了，此时忽略覆盖文件。

<br>

#### 🔹配置grpc服务地址

添加连接grpc服务代码之后，在配置文件`configs/服务名称.yml`设置连接grpc服务的地址，主要配置内容如下：

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

#### 🔹添加grpc服务的proto文件

虽然在grpc网关服务可以连接到grpc服务，但是不知道grpc服务哪些api接口可以调用，通过proto文件可以告诉grpc网关服务可以调用的api接口。

把grpc服务代码目录的`api/grpc服务名称/v1/xxx.proto`文件复制出来，并移动到grpc网关服务代码的`api`目录。有了grpc服务proto文件，grpc网关服务就可以知道有哪些api接口可以调用了。

切换到grpc网关服务目录，打开终端，执行命令：

```bash
# 从其他grpc服务中复制proto文件到本服务项目中，如果有多个grpc服务目录，用逗号分隔
make copy-proto SERVER=../user
```

> [!note] `make copy-proto`会把所有proto文件复制过来，如果proto文件存在，会覆盖proto文件，可以在目录`/tmp/sponge_copy_backup_proto_files`下找到覆盖前的备份proto文件。

<br>

#### 🔹运行已经准备好的grpc服务

在已经准备好的grpc服务[user](https://github.com/zhufuyi/sponge_examples/tree/main/4_micro-grpc-protobuf)目录下打开终端，执行命令：

```bash
# 生成与合并api接口相关代码
make proto

# 编译和运行服务
make run
```

<br>

### 🏷人工添加api接口

在grpc网关服务添加新的api接口比较简单，主要流程是`在proto文件定义api接口描述信息` --> `在模板文件编写具体逻辑代码`。

> [!note] `在proto文件定义api接口描述信息`是在`api/服务名称/v1`目录下的proto文件添加api接口描述信息，也可以创建新的proto文件。

<br>

**(1) 在proto文件定义api接口描述信息**

进入目录`api/edusys/v1`目录，打开文件`user_gw.proto`，添加修改密码接口的描述信息：

```protobuf
import "validate/validate.proto";
import "tagger/tagger.proto";

service user {
  // ...

  // 修改密码
  rpc ChangePassword(ChangePasswordRequest) returns (ChangeRegisterReply)  {
    option (google.api.http) = {
      post: "/api/v1/user/change_password"
      body: "*"
    };
    option (grpc.gateway.protoc_gen_openapiv2.options.openapiv2_operation) = {
      summary: "修改密码",
      description: "修改密码",
      security: {
        security_requirement: {
          key: "BearerAuth";
          value: {}
        }
      }
    };
  }
}

message ChangePasswordRequest {
  uint64 id = 1 [(validate.rules).uint64.gte  = 1, (tagger.tags) = "uri:\"id\"" ];
  string password = 2 [(validate.rules).string.min_len = 6];
}

message ChangePasswordReply {
}
```

> [!tip] 字段id和password后面的`validate.rules`是字段校验规则，点击查看更多[validate校验规则](https://github.com/envoyproxy/protoc-gen-validate#constraint-rules)，而`tagger.tags`是结构体字段tag，这里表示路径参数id。记得在proto文件添加 import "validate/validate.proto" 和 "tagger/tagger.proto"。

添加api接口描述信息后，在终端执行命令：

```bash
# 生成与合并api接口相关代码
make proto
```

> [!note] 开发过程中会经常使用 `make proto`命令，内部执行一系列生成代码子命令：生成api接口的`模板代码`、`错误码`、`注册路由代码`、`swagger文档`、`相关的*.pb.go`，`自动合并api接口模板代码`。合并代码时不用担心覆盖已编写业务逻辑代码问题，就算出现意外(断电)，可以在 `/tmp/sponge_merge_backup_code` 目录下找到每次合并前的备份代码，如果是windows环境则存放在 `C:\Users\你的用户名\AppData\Local\Temp\sponge_merge_backup_code`。如果在proto文件添加或更新了api接口描述信息，需要执行这个命令，否则不需要执行。

<br>

**(2) 在模板文件编写具体逻辑代码**

有了`连接grpc服务代码`、`grpc服务api接口`、`生成的模板代码`之后，接下来可以编写具体逻辑代码了，执行命令：

如果只是简单的把http请求转发给grpc服务处理，生成的模板代码默认已经实现了，不需要写go代码，删除`panic("implement me")`，然后释放`example`下面的注释代码，然后简单的调整一下代码即可。例如打开`internal/service/user_gw.go`文件，释放注释后的代码如下：

```go
package service

import (
	userV1 "edusys/api/user/v1"
    // ......
)

var _ edusysV1.UserLogicer = (*userClient)(nil)

type userClient struct {
	// example:
	userCli userV1.UserClient
}

// NewUserClient create a client
func NewUserClient() edusysV1.UserLogicer {
	return &userClient{
		// example:
		userCli: userV1.NewUserClient(rpcclient.GetUserRPCConn()),
	}
}

// Register 注册
func (c *userClient) Register(ctx context.Context, req *edusysV1.RegisterRequest) (*edusysV1.RegisterReply, error) {
	// example:
	err := req.Validate()
	if err != nil {
		logger.Warn("req.Validate error", logger.Err(err), logger.Any("req", req), interceptor.ServerCtxRequestIDField(ctx))
		return nil, ecode.StatusInvalidParams.Err()
	}

	reply, err := c.userCli.Register(ctx, &userV1.RegisterRequest{
		Email:    req.Email,
		Password: req.Password,
	})
	if err != nil {
		return nil, err
	}

	return &edusysV1.RegisterReply{
		Id: reply.Id,
	}, nil
}

// Login 登录
func (c *userClient) Login(ctx context.Context, req *edusysV1.LoginRequest) (*edusysV1.LoginReply, error) {
	// example:
	// ......
}

// Logout 登出
func (c *userClient) Logout(ctx context.Context, req *edusysV1.LogoutRequest) (*edusysV1.LogoutReply, error) {
	// example:
	// ......
}

// ChangePassword 修改密码
func (c *userClient) ChangePassword(ctx context.Context, req *edusysV1.ChangePasswordRequest) (*edusysV1.ChangeRegisterReply, error) {
	// example:
	// ......
}
```

<br>

上面是从单个grpc服务中获取数据，实际使用中有可能需要从多个grpc服务中获取数据，然后组装成客户端所需的数据，只需在`xxxClient`结构体引入多个grpc服务client接口，然后实例化，示例代码如下：

```go
package service

import (
	userV1 "edusys/api/user/v1"
	relationV1 "edusys/api/relation/v1"
	creationV1 "edusys/api/creation/v1"
    // ......
)

var _ edusysV1.UserLogicer = (*userClient)(nil)

type userClient struct {
	// example:
	userCli userV1.UserClient
	relationCli relationV1.RelationClient
	creationCli creationV1.CreationClient
}

// NewUserClient create a client
func NewUserClient() edusysV1.UserLogicer {
	return &userClient{
		// example:
		userCli: userV1.NewUserClient(rpcclient.GetUserRPCConn()),
		relationCli: userV1.NewRelationClient(rpcclient.GetRelationRPCConn()),
		creationCli: userV1.NewCreationClient(rpcclient.GetCreationRPCConn()),
	}
}

// ......

```

<br>

**(3) 运行grpc网关服务**

切换到grpc网关服务代码目录，执行命令：

```bash
# 编译和运行服务
make run
```

在浏览器打开 [http://localhost:8080/apis/swagger/index.html](http://localhost:8080/apis/swagger/index.html)，可以在页面上看到api接口，如下图所示。请求api接口，测试是否能够正常调用grpc服务接口。

> [!warning] 在没有编写具体逻辑代码之前，直接在swagger页面请求，会返回错误码500，因为生成的模板代码(internal/service/xxx.go)下每个方法函数下都有一行代码 `panic("implement me")`，提示需要实现具体逻辑代码。

![web-http-swagger](assets/images/web-http-pb-swagger.png)

> [!note] 如果在配置文件 `configs/服务名称.yml` 修改了http下的端口号，例如把默认值8080改为9090，则必须在目录 `api/user/v1` 下所有proto文件里的host字段改为 `localhost:9090`，然后执行命令`make proto`，否则因为端口不一致造成请求失败。

<br>

### 🏷设置grpc网关服务

创建的grpc网关服务代码中包含了丰富的组件，有些组件默认是关闭的，根据实际需要开启使用，统一在配置文件`configs/服务名称.yml`进行设置，配置文件里有详细说明。

> [!tip] 可以在服务代码中替换、添加自己的组件(gin middleware)，或者删除不需要的组件，在代码文件`internal/routers/routers.go`修改。

> [!tip] 如果api接口需要添加鉴权，在各个`internal/routers/proto文件_router.go`下，释放默认注释的`middleware.Auth()`代码，支持单独路由和分组路由两种方式。当然也可以使用自己的鉴权中间件，如果使用自己的鉴权中间件，需要把`internal/routers/routers.go`下的`jwt.Init`改为自己鉴权初始化。

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
- **grpc客户：**
  - **enableLoadBalance**：负载均衡
  - **serverSecure**：通过证书验证，支持服务端验证和双向验证
  - **enableToken**：通过token验证

其他配置的可以根据需要设置，也可以添加配置，如果添加或更改配置文件字段，需要更新对应的go结构体，在服务代码目录下的终端执行命令:

```bash
make update-config
```

<br>

---

相关视频介绍：

- [一键生成grpc网关服务项目代码](https://www.bilibili.com/video/BV1mV4y1D7k9/)
