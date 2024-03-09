
### 🏷日志

`logger`是基于[zap](https://github.com/uber-go/zap)封装的组件，点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/logger#logger)。

在sponge创建的web或grpc服务中，日志组件默认是开启的，默认是输出到终端，默认输出日志格式是console，可以设置输出格式为json，设置日志保存到指定文件，日志文件切割和保留时间。

在`configs`目录下yaml配置文件设置字段`logger`：

```yaml
# logger 设置
logger:
  level: "info"            # 输出日志级别 debug, info, warn, error，默认是debug
  format: "console"        # 输出格式，console或json，默认是console
  isSave: false            # false:输出到终端，true:输出到文件，默认是false
  logFileConfig:           # isSave=true时有效
    filename: "out.log"    # 文件名称，默认值out.log
    maxSize: 20            # 最大文件大小(MB)，默认值10MB
    maxBackups: 50         # 保留旧文件的最大个数，默认值100个
    maxAge: 15             # 保留旧文件的最大天数，默认值30天
    isCompression: true    # 是否压缩/归档旧文件，默认值false
```

> [!tip] 查看服务日志说明：字符串标记 `<<<<` 表示请求开始的日志，字符串标记 `>>>>` 表示请求结束返回的日志，都是成对出现的，可以通过相同的request_id看到请求开始到结束整条链路的日志。

<br>

### 🏷gin与常用中间件集合

gin中间件包括了日志、jwt鉴权、跨域cors、自适应限流、自适应熔断、链路跟踪、metrics采集、Request id，点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/gin/middleware#example-of-use)。

在`configs`目录下yaml配置文件设置字段`http`：

```yaml
http:
  port: 8080       # 端口
  readTimeout: 5   # 读超时
  writeTimeout: 5  # 写超时
```

常用的中间件已经在生成的web服务代码`internal/routers/routers.go`中。

<br>

### 🏷grpc与常用拦截器集合

- grpc服务端组件，点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/grpc/server)。
- grpc客户端组件，点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/grpc/client)。

在`configs`目录下yaml配置文件设置grpc服务端字段`grpc`：

```yaml
grpc:
  port: 8282          # 监听端口
  httpPort: 8283      # profile 和 metrics 的http监听端口
  readTimeout: 5      # 读超时(s)
  writeTimeout: 5     # 写超时(s)
  # 安全参数设置
  # 如果 type=""，表示无安全连接，无需填写任何参数
  # 如果 type="one-way"，表示服务器端认证，只需填写 "certFile "和 "keyFile "字段
  # 如果 type="two-way"，表示同时进行客户端和服务器端认证，应填写所有字段
  serverSecure:
    type: ""               # 类型，"", "one-way", "two-way"
    caFile: ""             # ca文件，仅在"two-way"时有效，绝对路径
    certFile: ""           # 服务端cert文件，绝对路径
    keyFile: ""            # 服务端key文件，绝对路径
```

在`configs`目录下yaml配置文件设置grpc客户端字段`grpcClient`：

```yaml
grpcClient:
  - name: "serverNameExample"   # grpc 服务名称，用于服务发现
    host: "127.0.0.1"           # grpc 服务ip, 用于直接连接
    port: 8282                  # grpc 服务端口
    registryDiscoveryType: ""   # 注册和发现类型：Consul、etcd、nacos（如果为空），使用主机和端口连接服务器
    enableLoadBalance: true     # 是否开启负载均衡
    # 安全连接设置
    # type="", 这意味着没有安全连接，无需填写任何参数
    # type="one-way", 表示服务器端认证，只需填写 "serverName "和 "certFile "字段
    # type="two-way", 是指客户端和服务器端认证，请填写所有字段
    clientSecure:
      type: ""           # 类似："", "one-way", "two-way"
      serverName: ""     # 域名, e.g. *.foo.com
      caFile: ""         # 客户端 ca 文件，仅在 "two-way"中有效，绝对路径
      certFile: ""       # 客户端证书文件，绝对路径，如果 secureType="one-way"，请在此处填写服务器端证书文件
      keyFile: ""        # 客户端密钥文件，仅在 "two-way"情况下有效，绝对路径
    clientToken:
      enable: false      # 是否启用令牌身份验证
      appID: ""          # app id
      appKey: ""         # app key
```

<br>

grpc拦截器分为客户端和服务端，包括了日志、jwt鉴权、recovery、自适应限流、自适应熔断、链路跟踪、metrics采集、Request id、重试、超时、token，点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/grpc/interceptor#example-of-use)。

常用的拦截器已经在生成的grpc服务代码`internal/server/grpc.go`中。

<br>

### 🏷gorm

`ggrom`是基于[gorm](https://github.com/go-gorm/gorm)封装的数据库组件，在gorm基础上增加了链路跟踪、自定义条件查询等功能。点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/ggorm#examples-of-use)。

如果数据库使用 **mysql** 或 **tidb**，在`configs`目录下yaml文件设置：

```yaml
# database setting
database:
  driver: "mysql"
  mysql:
    # dsn format,  <user>:<pass>@(127.0.0.1:3306)/<db>?[k=v& ......]
    dsn: "root:123456@(127.0.0.1:3306)/account?parseTime=true&loc=Local&charset=utf8mb4"
    enableLog: true             # 是否开启日志
    maxIdleConns: 3             # 设置空闲连接池中的最大连接数
    maxOpenConns: 100           # 设置打开的数据库连接数上限
    connMaxLifetime: 30         # 设置连接可重复使用的最长时间，单位(分)
    #slavesDsn:                 # 设置从 mysql dsn
    #  - "你的 dsn 1"
    #  - "你的 dsn 2"
    #mastersDsn:                # 设置 masters mysql dsn，数组类型，非必填字段，如果只有一个 master，则无需设置 mastersDsn 字段，默认 dsn 字段为 mysql master.
    #  - "你的 master dsn"
```

<br>

如果数据库使用 **postgresql**，在`configs`目录下yaml文件设置：

```yaml
# database setting
database:
  driver: "postgresql"
  postgres:
    # dsn format,  <username>:<password>@<hostname>:<port>/<db>?[k=v& ......]
    dsn: "root:123456@192.168.3.37:5432/account?sslmode=disable"
    enableLog: true             # 是否开启日志
    maxIdleConns: 3             # 设置空闲连接池中的最大连接数
    maxOpenConns: 100           # 设置打开的数据库连接数上限
    connMaxLifetime: 30         # 设置连接可重复使用的最长时间，单位(分)
```

<br>

如果数据库使用 **sqlite**，在`configs`目录下yaml文件设置：

```yaml
# database setting
database:
  driver: "sqlite"
  sqlite:
    dbFile: "test/sql/sqlite/sponge.db"   # 如果在 Windows 环境中，路径分隔符为\\
    enableLog: true             # 是否开启日志
    maxIdleConns: 3             # 设置空闲连接池中的最大连接数
    maxOpenConns: 100           # 设置打开的数据库连接数上限
    connMaxLifetime: 30         # 设置连接可重复使用的最长时间，单位(分)
```

<br>

### 🏷mongodb

`mgo` 基于官方库[mongo](https://github.com/mongodb/mongo-go-driver)封装的库，点击查看[使用示例](https://github.com/zhufuyi/sponge/blob/main/pkg/mgo/README.md#example-of-use)。

在`configs`目录下yaml文件设置：

```yaml
# database setting
database:
  driver: "mongodb"
  mongodb:
    # dsn format,  <username>:<password>@<hostname1>:<port1>[,<hostname2>:<port2>,......]/<db>?[k=v& ......]
    dsn: "root:123456@192.168.3.37:27017/account?connectTimeoutMS=15000"
```

<br>

### 🏷redis

`goredis`是基于[go-redis](https://github.com/go-redis/redis)封装的Nosql组件，在gorm基础上增加了链路跟踪功能，点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/goredis#example-of-use)。

在`configs`目录下yaml文件设置字段`redis`，支持设置sentinel和cluster：

```yaml
redis:
  # dsn 格式， [user]:<pass>@127.0.0.1:6379/[db], 默认用户为 default，redis 6.0 及以上版本支持 user。
  dsn: "default:123456@127.0.0.1:6379/0"
  dialTimeout: 10       # 连接超时, 单位(秒)
  readTimeout: 2        # 读超时, 单位(秒)
  writeTimeout: 2       # 写超时, 单位(秒)
  # sentinelAddrs: ["127.0.0.1:6379", "127.0.0.1:6380"]
  # clusterAddrs: ["127.0.0.1:6379", "127.0.0.1:6380"]
```

<br>

### 🏷 消息队列

`rabbitmq`是基于[amqp091-go](https://github.com/rabbitmq/amqp091-go)封装的消息组件，支持自动重新连接和自定义队列参数设置，点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/rabbitmq#example-of-use)。

在`configs`目录下yaml文件设置字段`rabbitmq`：

```yaml
rabbitmq:
  dsn: "amqp://guest:guest@127.0.0.1:5672/"
```

<br>

### 🏷服务注册与发现

- 服务注册[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/servicerd/registry#example-of-use)。
- 服务发现[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/servicerd/discovery#example-of-use)。

服务注册与发现支持consul、etcd、nacos三种类型：

- consul客户端，点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/consulcli#example-of-use)。
- etcd客户端，点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/etcdcli#example-of-use)。
- nacos客户端，点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/nacoscli#example-of-use)。

<br>

在`configs`目录下yaml文件设置，选择其中一种即可。

```yaml
# (1) 在app下设置字段registryDiscoveryType，例如使用etcd作为注册与发现，还要设置etcd服务地址
# (2) 在app下填写字段host值，注意：如果etcd和host不在同一台机器，host必须填写本地的ip或域名，不能填写127.0.0.1
app:
  registryDiscoveryType: "etcd"   # 支持consul、etcd、nacos三种方式
  host: "127.0.0.1"

etcd:
  addrs: ["127.0.0.1:2379"]



# 如果选择consul或nacos作为服务注册与发现，配置与etcd类似

consul:
  addr: "127.0.0.1:8500"

nacosRd:
  ipAddr: "127.0.0.1"
  port: 8848
  namespaceID: "public"
```

<br>

### 🏷资源统计

`stat`是对系统和服务资源进行使用情况统计的组件，点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/stat#example-of-use)。

在sponge创建的web或grpc务中，资源统计组件默认是开启的。

在`configs`目录下yaml配置文件设置字段`enableStat`：

```yaml
app:
  enableStat: true    # 是否开启资源统计，true:启用，false:关闭
```

默认每分钟统计一次并输出到日志，资源统计包括系统和服务本身这两部分的cpu和内存相关的数据。

<br>

### 🏷配置中心

sponge生成的web或grpc务默认支持[Nacos](https://nacos.io/zh-cn/docs/v2/what-is-nacos.html)配置中心，配置中心作用是对不同环境、不同服务的配置统一管理，有效的解决地静态配置的缺点。

在本地启动nacos服务，这是[nacos服务启动配置](https://github.com/zhufuyi/sponge/tree/main/test/server/nacos)，启动nacos服务之后，在浏览器打开管理界面 http://localhost:8848/nacos/index.html ，登录账号密码进入主界面。

以`⓵基于sql创建web服务`代码为例使用配置中心nacos，在nacos界面创建一个名称空间`user`，然后新建配置，Data ID值为`user.yml`，Group值为`dev`，配置内容值`configs/user.yml`文件内容，如图下图所示：

![nacos-config](assets/images/nacos-config.jpg)

<br>

打开user目录下配置中心文件`configs/user_cc.yml`，填写nacos配置信息：

```yaml
# nacos settings
nacos:
  ipAddr: "127.0.0.1"    # ip
  port: 8848             # 端口
  scheme: "http"         # http or https
  contextPath: "/nacos"  # 路径
  namespaceID: "你的name space id"
  group: "dev"           # 组名，例如dev, prod, test
  dataID: "user.yml"     # 配置文件 id
  format: "yaml"         # 配置文件类型 json,yaml,toml
```

编译和启动user服务：

```bash
# 切换到main.go位置
cd cmd/user

# 编译
go build

# 运行服务，参数`-c`表示指定配置文件，参数`-enable-cc`表示从配置中心获取配置。
./user -enable-cc -c=../../configs/user_cc.yml
```

> [!tip] 如果使用docker或kubernetes部署，只需把默认启动服务命令，修改为以配置中心方式启动服务。sponge生成的部署脚本包括了两种启动服务命令，只需选择一种。

<br>

### 🏷自适应限流

自适应限流是默认阈值与系统资源使用情况综合决定是否限流。由于不同服务器的处理能力不一样，在服务器比较多时候，参数不好设置，根据服务器处理能力自适应限流，避免了每个服务手动设置参数的麻烦。点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/shield/ratelimit#example-of-use)。

在sponge创建的web或grpc务中，限流组件默认是关闭的。

在`configs`目录下yaml文件设置字段`enableLimit`：

```yaml
app:
  enableLimit: false    # 是否开启自适应限流，true:开启, false:关闭
```

虽然有默认的阈值和系统资源配额，可以修改这些默认值。

对于`web`服务，在`internal/routers/routers.go`里的RateLimit修改默认值，示例：

```go
	// limit middleware
	if config.Get().App.EnableLimit {
		r.Use(middleware.RateLimit(
			middleware.WithWindow(time.Second*5),
			middleware.WithBucket(200),
			middleware.WithCPUThreshold(600),
			middleware.WithCPUQuota(0),
		))
	}
```

对于`grpc服务`，在`internal/server/grpc.go`里的UnaryServerRateLimit修改默认值，示例：

```go
	// limit interceptor
	if config.Get().App.EnableLimit {
		unaryServerInterceptors = append(unaryServerInterceptors, interceptor.UnaryServerRateLimit(
			interceptor.WithWindow(time.Second*5),
			interceptor.WithBucket(200),
			interceptor.WithCPUThreshold(600),
			interceptor.WithCPUQuota(0),
		))
	}
```

<br>

### 🏷自适应熔断

自适应熔断是根据请求错误率与系统资源使用情况综合决定是否熔断。由于不同服务器的处理能力不一样，在服务器比较多时候，参数不好设置，根据服务器处理能力自适应熔断，避免了每个服务手动设置参数的麻烦。点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/shield/circuitbreaker#example-of-use)。

在sponge创建的web或grpc务中，自适应熔断组件默认是关闭的。

在`configs`目录下yaml配置文件设置字段`enableCircuitBreaker`：

```yaml
app:
  enableCircuitBreaker: false    # 是否开启自适应熔断，true:开启, false:关闭
```

对于web服务，默认对500和503错误码有效，其他错误不会触发熔断，可以增加其他错误码，在`internal/routers/routers.go`里的CircuitBreaker添加错误码，示例：

```go
	// circuit breaker middleware
	if config.Get().App.EnableCircuitBreaker {
		r.Use(middleware.CircuitBreaker(
			middleware.WithValidCode(403),
		))
	}
```

对于grpc服务，对默认Internal(13)和Unavailable(14)错误码有效，其他错误不会触发熔断，可以增加其他错误码，在`internal/server/grpc.go`里的CircuitBreaker添加错误码，示例：
```go
	// circuit breaker interceptor
	if config.Get().App.EnableCircuitBreaker {
		unaryServerInterceptors = append(unaryServerInterceptors, interceptor.UnaryServerCircuitBreaker(
			// set grpc code for circuit breaker, default already includes codes.Internal and codes.Unavailable
			interceptor.WithValidCode(codes.Unauthenticated),
		))
	}
```

<br>

### 🏷链路跟踪

链路跟踪是基于[go.opentelemetry.io/otel](https://github.com/open-telemetry/opentelemetry-go)封装的组件，点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/tracer#example-of-use)。

在sponge创建的web或grpc务中，链路跟踪组件默认是关闭的。

在`configs`目录下yaml配置文件设置字段`enableTrace`和`jaeger`：

```yaml
app:
  enableTrace: false         # 是否开启追踪，true:启用，false:关闭，如果是true，必须设置jaeger配置。
  tracingSamplingRate: 1.0   # 链路跟踪采样率, 范围0~1.0浮点数, 0表示不采样, 1.0表示采样所有链路

jaeger:
  agentHost: "127.0.0.1"
  agentPort: 6831
```

<br>

#### 🔹启动jaeger和elasticsearch服务

链路跟踪使用jaeger，存储使用elasticsearch，在本地使用[docker-compose](https://github.com/docker/compose/releases)启动两个服务。

**(1) elasticsearch服务**

这是 [elasticsearch服务的启动脚本](https://github.com/zhufuyi/sponge/tree/main/test/server/elasticsearch)，`.env`文件是elasticsearch的启动配置，启动elasticsearch服务：

> docker-compose up -d

<br>

**(2) jaeger服务**

这是 [jaeger服务的启动脚本](https://github.com/zhufuyi/sponge/tree/main/test/server/jaeger)，`.env`文件是配置jaeger信息，启动jaeger服务：

> docker-compose up -d

在浏览器访问jaeger查询主页 [http://localhost:16686](http://localhost:16686) 。

<br>

#### 🔹单服务链路跟踪示例

以`⓵基于sql创建web服务`代码为例，修改配置文件`configs/user.yml`，开启链路跟踪功能(字段enableTrace)，并且填写jaeger配置信息。

如果想跟踪redis，启用redis缓存，把yaml配置文件里的缓存类型字段**cacheType**值改为redis，并配置redis地址，同时在本地使用docker启动redis服务，这是[redis服务启动脚本](https://github.com/zhufuyi/sponge/tree/main/test/server/redis)。

运行web服务：

```bash
# 编译和运行服务
make run
```

复制 [http://localhost:8080/swagger/index.html](http://localhost:8080/apis/swagger/index.html) 到浏览器访问swagger主页，以请求get查询为例，连续请求同一个id两次，链路跟踪如下图所示。

![one-server-trace](assets/images/one-server-trace.jpg)

<br>

从图中可以看到第一次请求有4个span，分别是：

- 请求接口 /api/v1/teacher/1
- 查询redis
- 查询mysql
- 设置redis缓存

说明第一次请求从redis查找，没有命中缓存，然后从mysql读取数据，最后设置缓存。

第二次请求只有2个span，分别是：

- 请求接口 /api/v1/teacher/1
- 查询redis

说明第二次请求直接命中缓存，比第一次少了查询mysql和设置缓存过程。

这些span是自动生成的，很多时候需要手动添加自定义span，添加span示例：

```go
import "github.com/zhufuyi/sponge/pkg/tracer"

tags := map[string]interface{}{"foo": "bar"}
_, span := tracer.NewSpan(ctx, "spanName", tags)  
defer span.End()
```

<br>

#### 🔹多服务链路跟踪示例

以一个极简版的电商微服务集群为例，点击查看[源码](https://github.com/zhufuyi/sponge_examples/tree/main/6_micro-cluster)，一个共四个服务**shopgw**、**product**、**inventory**、**comment**，分别修改4个服务yaml配置(在configs目录下)，开启链路跟踪功能，并且填写jaeger配置信息。

在 **product**、**inventory**、**comment** 三个服务的**internal/service**目录下找到模板文件，填充代码替代`panic("implement me")`，使得代码可以正常执行，并且手动添加一个**span**，添加随机延时。

启动 **shopgw**、**product**、**inventory**、**comment** 四个服务，在浏览器访问 [http://localhost:8080/apis/swagger/index.html](http://localhost:8080/apis/swagger/index.html) ，执行get请求，链路跟踪界面如下图所示。

![multi-servers-trace](assets/images/multi-servers-trace.jpg)

<br>

从图中可以看到共有10个span，主要链路：

- 请求接口/api/v1/detail
- shopgw 服务调用product的grpc客户端
- product 的grpc服务端
- product 服务中手动添加的mockDAO
- shopgw 服务调用inventory的grpc客户端
- inventory 的grpc服务端
- inventory 服务中手动添加的mockDAO
- shopgw 服务调用comment的grpc客户端
- comment 的grpc服务端
- comment 服务中手动添加的mockDAO

shopgw服务串行调用了**product**、**inventory**、**comment** 三个服务获取数据，实际中可以改为并行调用会更节省时间，但是要注意控制协程数量。

<br>

### 🏷Prometheus和Grafana监控

监控是服务程序提供metrics，[Prometheus](https://prometheus.io/docs/introduction/overview)采集到metrics，然后在[Grafana](https://grafana.com/docs/)展示。

- 点击查看`web`服务监控[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/gin/middleware/metrics#example-of-use)。
- 点击查看`grpc服务`监控[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/grpc/metrics#example-of-use)。

在sponge创建的web或grpc务中，指标采集功能默认是开启的，默认路由是`/metrics`。

在`configs`目录下yaml配置文件设置字段`enableMetrics`：

```yaml
app:
  enableMetrics: true    # 是否开启指标采集，true：启用，false：关闭
```

#### 🔹启动Prometheus和Grafana服务

**(1) prometheus服务**

这是 [prometheus服务启动脚本](https://github.com/zhufuyi/sponge/tree/main/test/server/monitor/prometheus)，启动prometheus服务：

```bash
docker-compose up -d
```

在浏览器访问prometheus主页 [http://localhost:9090](http://localhost:9090/) 。

<br>

**(2) grafana服务**

这是 [grafana服务启动脚本](https://github.com/zhufuyi/sponge/tree/main/test/server/monitor/grafana)，启动grafana服务：

```bash
docker-compose up -d
```

在浏览器访问 grafana 主页面 [http://localhost:33000](http://localhost:33000) ，设置prometheus的数据源 `http://localhost:9090` 。

> [!attention] 在grafana导入监控面板的json的**datasource**值，必须与在grafana设置的prometheus的数据源名称(这里是**Prometheus**)要一致，否则图标上无法显示数据。

<br>

#### 🔹web服务监控示例

以`⓵基于sql创建web服务`代码为例，默认提供指标接口 [http://localhost:8080/metrics](http://localhost:8080/metrics) 。

**(1) 在prometheus添加监控目标**

打开prometheus配置文件`prometheus.yml`，添加采集目标：

```bash
  - job_name: 'http-edusys'
    scrape_interval: 10s
    static_configs:
      - targets: ['localhost:8080']
```

> [!attention] 在启动Prometheus服务前，必须将文件`prometheus.yml`权限改为`0777`，否则使用vim修改`prometheus.yml`文件无法同步到容器中。

执行请求使prometheus配置生效 `curl -X POST http://localhost:9090/-/reload` ，稍等一会，然后在浏览器访问 [http://localhost:9090/targets](http://localhost:9090/targets)，检查新添加的采集目标是否生效。

<br>

**(2) 在grafana添加监控面板**

把 [http 监控面板](https://github.com/zhufuyi/sponge/blob/main/pkg/gin/middleware/metrics/gin_grafana.json) 导入到grafana，如果监控界面没有数据显示，检查json里的数据源名称与grafana配置prometheus数据源名称是否一致。

<br>

**(3) 压测接口，观察监控数据**

使用[wrk](https://github.com/wg/wrk)工具压测接口

```bash
# 接口1
wrk -t2 -c10 -d10s http://192.168.3.27:8080/api/v1/teacher/1

# 接口2
wrk -t2 -c10 -d10s http://192.168.3.27:8080/api/v1/course/1
```

监控界面如下图所示：

![http-grafana](assets/images/http-grafana.jpg)

<br>

#### 🔹grpc服务监控示例

以`⓶基于sql创建grpc服务`代码为例，默认提供指标接口 [http://localhost:8283/metrics](http://localhost:8283/metrics) 。

**(1) 在prometheus添加监控目标**

打开prometheus配置文件`prometheus.yml`，添加采集目标：

```yaml
  - job_name: 'rpc-server-user'
    scrape_interval: 10s
    static_configs:
      - targets: ['localhost:8283']
```

> [!attention] 在启动Prometheus服务前，必须将文件`prometheus.yml`权限改为`0777`，否则使用vim修改`prometheus.yml`文件无法同步到容器中。

执行请求使prometheus配置生效 `curl -X POST http://localhost:9090/-/reload` ，稍等一会，然后在浏览器访问 [http://localhost:9090/targets](http://localhost:9090/targets)， 检查新添加的采集目标是否生效。

<br>

**(2) 在grafana添加监控面板**

把 [grpc server 监控面板](https://github.com/zhufuyi/sponge/blob/main/pkg/grpc/metrics/server_grafana.json) 导入到grafana，如果监控界面没有数据显示，检查json里的数据源名称与grafana配置prometheus数据源名称是否一致。

<br>

**(3) 压测grpc方法，观察监控数据**

使用`Goland` IDE打开`internal/service/teacher_client_test.go`文件，对**Test_teacherService_methods** 或 **Test_teacherService_benchmark** 下各个方法进行测试。

监控界面如下图所示。
![rpc-grafana](assets/images/rpc-grafana.jpg)

<br>

上面是grpc服务端的监控，grpc的客户端的监控也类似，[grpc client 监控面板](https://github.com/zhufuyi/sponge/blob/main/pkg/grpc/metrics/client_grafana.json) 。

<br>

#### 🔹在prometheus自动添加和移除监控目标

实际使用中服务数量比较多，手动添加监控目标到prometheus比较繁琐，也容易出错。prometheus支持使用`consul`的服务注册与发现进行动态配置，自动添加和移除监控目标。

在本地启动 consul 服务，这是 [consul 服务启动脚本](https://github.com/zhufuyi/sponge/tree/main/test/server/consul)，启动consul服务：

```bash
docker-compose up -d
```

打开 prometheus 配置 prometheus.yml，添加consul配置：

```yaml
  - job_name: 'consul-micro-exporter'
    consul_sd_configs:
      - server: 'localhost:8500'
        services: []  
    relabel_configs:
      - source_labels: [__meta_consul_tags]
        regex: .*user.*
        action: keep
      - regex: __meta_consul_service_metadata_(.+)
        action: labelmap
```

执行请求使prometheus配置生效 `curl -X POST http://localhost:9090/-/reload` 。

在prometheus配置好consul服务发现之后，接着把服务的地址信息推送到consul，推送信息 user_exporter.json 文件内容如下：

```json
{
  "ID": "user-exporter",
  "Name": "user",
  "Tags": [
    "user-exporter"
  ],
  "Address": "localhost",
  "Port": 8283,
  "Meta": {
    "env": "dev",
    "project": "user"
  },
  "EnableTagOverride": false,
  "Check": {
    "HTTP": "http://localhost:8283/metrics",
    "Interval": "10s"
  },
  "Weights": {
    "Passing": 10,
    "Warning": 1
  }
}
```

> curl -XPUT --data @user_exporter.json http://localhost:8500/v1/agent/service/register

稍等一会，然后在浏览器打开 [http://localhost:9090/targets](http://localhost:9090/targets)  检查新添加的采集目标是否生效。然后关闭服务，稍等一会，检查是否自动移除采集目标。

> [!tip] 在web或grpc务中，通常是使用程序代码自动把json信息提交给consul，不是通过命令，web或grpc务正常启动服务后，Prometheus就可以动态获取到监控目标，web或grpc务停止后，Prometheus自动移除监控目标。

<br>

### 🏷采集go程序profile

通常使用pprof工具来发现和定位程序问题，特别是线上go程序出现问题时希望自动的把程序运行现场(profile)保存下来，再使用工具pprof分析定位问题。

sponge生成的服务支持`http接口`和`系统信号通知`两种方式采集profile，默认开启`系统信号通知`方式。

<br>

#### 🔹通过http api接口采集profile

> [!note] 要能够使用http api 接口采集profile，需要在`configs`目录下yaml文件设置 `enableMetrics: true`，默认路由是`/debug/pprof`。

点击查看通过http api接口采集profile[使用示例](https://github.com/zhufuyi/sponge/blob/main/pkg/prof/README.md#sampling-profile-by-http)。

通常在开发或测试时使用，如果线上开启会有一点点性能损耗，根据实际情况是否开启使用。除了支持go语言本身提供默认的profile分析，还支持io分析，路由是`/debug/pprof/profile-io`。

- 对于web服务，默认采集profile地址 http://localhost:8080/debug/pprof
- 对于grpc服务，默认采集profile地址 http://localhost:8283/debug/pprof

结合**go tool pprof**工具，任意时刻都可以分析当前程序运行状况。

<br>

#### 🔹通过系统信号通知采集profile

点击查看通过系统信号通知采集profile[使用示例](https://github.com/zhufuyi/sponge/blob/main/pkg/prof/README.md#sampling-profile-by-system-notification-signal)。

使用`http接口`方式，程序后台一直定时记录profile相关信息等，绝大多数时间都不会去读取这些profile，可以改进一下，只有需要的时候再开始采集profile，采集完后自动关闭，sponge生成的服务支持监听系统信号来开启和停止采集profile，默认使用了 **SIGTRAP**(5) 系统信号(linux环境建议改为SIGUSR1，windows环境不支持SIGUSR1)，发送信号给服务：

```bash
# 通过名称查看服务pid(第二列)
ps aux | grep 服务名称

# 发送信号给服务
kill -trap pid值
# kill -usr1 pid值
```

服务收到系统信号通知后，开始采集profile并保存到`/tmp/服务名称_profile`目录，默认采集时长为60秒，60秒后自动停止采集profile，如果只想采集30秒，发送第一次信号开始采集，大概30秒后再发送第二次信号停止采集profile，类似开关。默认采集**cpu**、**memory**、**goroutine**、**block**、**mutex**、**threadcreate**六种类型profile，文件格式`日期时间_pid_服务名称_profile类型.out`，示例：

```
xxx221809_58546_user_cpu.out
xxx221809_58546_user_mem.out
xxx221809_58546_user_goroutine.out
xxx221809_58546_user_block.out
xxx221809_58546_user_mutex.out
xxx221809_58546_user_threadcreate.out
```

因为trace的profile文件相对比较大，因此默认没有采集，根据实际需要可以开启采集trace(在服务初始化时调用`prof.EnableTrace()`)。

获得离线文件后，使用pprof工具使用交互式或界面方式进行分析：

```bash
# 交互式
go tool pprof [options] source

# 界面
go tool pprof -http=[host]:[port] [options] source
```

<br>

#### 🔹自适应采集profile

> [!note] 要能够使用系统信号来通知采集profile，需要在configs目录下yaml文件设置 `enableStat: true`

在线上运行的服务，没有出问题时基本不会去手动采集profile，但是又想在服务发出告警同时采集profile文件。为了解决这个问题，sponge创建的web或grpc务默认支持自适应采集profile功能，是把`系统信号通知采集profile`与`资源统计的告警功能`结合起来实现的，告警条件：

- 记录程序的cpu使用率连续3次(默认每分钟一次)，3次平均使用率超过80%时触发告警。
- 记录程序的物理内存使用率3次(默认每分钟一次)，3次平均占用系统内存超过80%时触发告警。
- 如果持续超过告警阈值，默认间隔15分钟发出一次告警。

触发告警时，程序内部调用kill函数发送系统信号通知采集profile，采集的profile文件保存到`/tmp/服务名称_profile`目录，即使在半夜程序的cpu或内存过高，第二天也可以通过分析profile来发现程序哪里造成cpu或内存过高。

> [!note] 自适应采集profile功能不支持windows环境。

<br>

### 🏷配置解析

`conf`是解析配置到go结构体组件，支持`yaml`、`json`、`toml`三种文件格式，点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/conf#conf)。

<br>

### 🏷错误码

`errcode`包括http和grpc两种类型错误码，每种类型又分为系统级别和业务级别错误码，支持http和grpc错误码转换。点击查看[错误码规则和使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/errcode#errcode)。

<br>

### 🏷jwt鉴权

`jwt`是生成和解析token组件，基于[golang-jwt](https://github.com/golang-jwt/jwt)，点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/jwt#jwt)。

`jwt`作为gin中间件，支持自定义字段和鉴权方法，点击查看[使用示例](https://github.com/zhufuyi/sponge/blob/main/pkg/gin/middleware/README.md#jwt-authorization-middleware)。

<br>

### 🏷优雅启动与停止服务

`app`是优雅地启动和停止服务组件，使用[errgroup](https://github.com/zhufuyi/sponge/blob/main/pkg/app/golang.org/x/sync/errgroup)确保同时正确启动多个服务，点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/app#example-of-use)。

<br>

### 🏷http客户端

`gohttp`是封装http库的组件，点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/gohttp#example-of-use)。

<br>

### 🏷加密解密

`gocrypto`常用的单向加密、双向机密、非对称加密的库，点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/gocrypto#example-of-use)。

<br>

### 🏷定时任务

`gocron`是基于[cron](https://github.com/robfig/cron)封装的定时任务库，点击查看[使用示例](https://github.com/zhufuyi/sponge/tree/main/pkg/gocron#example-of-use)。
