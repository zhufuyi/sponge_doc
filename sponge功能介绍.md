### sponge 命令框架

生成代码基于**Yaml**、**SQL DDL**和**Protocol buffers**三种方式，每种方式拥有生成不同功能代码，生成代码的框架图如图1-1所示：

<p align="center">
<img width="1500px" src="https://raw.githubusercontent.com/zhufuyi/sponge_doc/main/assets/img/sponge-framework.png">
</p>
<p align="center">图1-1 sponge生成代码框架图</p>

<br>

- **Yaml**生成配置文件对应go struct代码。
- **SQL DDL**生成的代码包括 http、handler、 dao、 model、 proto、rpc、service，分为web和rpc两大类型，web和rpc类型都拥有自己的子模块，每个模块代码都可以单独生成，模块之间像洋葱一层一层独立解耦。代码包括了标准化的CRUD(增删改查)业务逻辑，可以直接编译就使用。
  - **web类型代码**： 生成http服务代码包括 handler、 dao、model 三个子模块代码，如图所示向内包含，同样原理，生成handler模块代码包含dao、 model两个子模块代码。
  - **rpc类型代码**：生成rpc服务代码包括 service、dao、model、protocol buffers 四个子模块，如图所示向内包含，生成service模块代码包括 dao、model、protocol buffers 三个子模块代码。
- **Protocol buffers**生成的代码包括 http-pb, rpc-pb, rpc-gw-pb，同样分为web和rpc两大类型，其中 http-pb, rpc-pb 通常结合**SQL DDL**生成的dao、model代码使用。
  - **http-pb**：http服务代码包括router、handler模板两个子模块，不包括操作数据库子模块，后续的业务逻辑代码填写到handler模板文件上。
  - **rpc-pb**：rpc服务代码包括service模板一个子模块，不包括操作数据库模块，后续的业务逻辑代码填写到service模板文件上。
  - **rpc-gw-pb**：rpc网关其实是http服务，包括router和service模板两个子模块，这里的service模板代码是调用rpc服务相关的业务逻辑代码。

在同一个文件夹内，如果发现最新生成代码和原来代码冲突，sponge会取消此次生成流程，不会对原来代码有任何修改，因此不必担心写的业务逻辑代码被覆盖问题。

<br>

### 微服务框架

sponge创建的微服务代码框架如图1-2所示，这是典型的微服务分层结构，具有高性能，高扩展性，包含常用的服务治理功能。

<p align="center">
<img width="1200px" src="https://raw.githubusercontent.com/zhufuyi/sponge_doc/main/assets/img/microservices-framework.png">
</p>
<p align="center">图1-2 微服务框架图</p>

<br>

微服务主要功能：

- Web 框架 [gin](https://github.com/gin-gonic/gin)
- RPC 框架 [grpc](https://github.com/grpc/grpc-go)
- 配置解析 [viper](https://github.com/spf13/viper)
- 配置中心 [nacos](https://github.com/alibaba/nacos)
- 日志 [zap](https://go.uber.org/zap)
- 数据库组件 [gorm](https://gorm.io/gorm)
- 缓存组件 [go-redis](https://github.com/go-redis/redis) [ristretto](github.com/dgraph-io/ristretto)
- 文档 [swagger](https://github.com/swaggo/swag)
- 鉴权 [jwt](https://github.com/golang-jwt/jwt)
- 校验 [validator](https://github.com/go-playground/validator)
- 限流 [ratelimit](pkg/shield/ratelimit)
- 熔断 [circuitbreaker](pkg/shield/circuitbreaker)
- 链路跟踪 [opentelemetry](https://go.opentelemetry.io/otel)
- 监控 [prometheus](https://github.com/prometheus/client_golang/prometheus), [grafana](https://github.com/grafana/grafana)
- 服务注册与发现 [etcd](https://github.com/etcd-io/etcd), [consul](https://github.com/hashicorp/consul), [nacos](https://github.com/alibaba/)
- 性能分析 [go profile](https://go.dev/blog/pprof)
- 代码规范检查 [golangci-lint](https://github.com/golangci/golangci-lint)
- 持续集成部署 CICD [jenkins](https://github.com/jenkinsci/jenkins) [docker](https://www.docker.com/), [kubernetes](https://github.com/kubernetes/kubernetes)

<br>

代码目录结构遵循 [project-layout](https://github.com/golang-standards/project-layout)，代码目录结构如下所示：

```bash
.
├── api            # proto文件和生成的*pb.go目录
├── assets         # 其他与资源库一起使用的资产(图片、logo等)目录
├── build          # 打包和持续集成目录
├── cmd            # 程序入口目录
├── configs        # 配置文件的目录
├── deployments    # IaaS、PaaS、系统和容器协调部署的配置和模板目录
├─ docs            # 设计文档和界面文档目录
├── internal       # 私有应用程序和库的代码目录
│    ├── cache        # 基于业务包装的缓存目录
│    ├── config       # Go结构的配置文件目录
│    ├── dao          # 数据访问目录
│    ├── ecode        # 自定义业务错误代码目录
│    ├── handler      # http的业务功能实现目录
│    ├── model        # 数据库模型目录
│    ├── routers      # http路由目录
│    ├── rpcclient    # 连接rpc服务的客户端目录
│    ├── server       # 服务入口，包括http、rpc等
│    ├── service      # rpc的业务功能实现目录
│    └── types        # http的请求和响应类型目录
├── pkg            # 外部应用程序可以使用的库目录
├── scripts        # 用于执行各种构建、安装、分析等操作的脚本目录
├── test           # 额外的外部测试程序和测试数据
└── third_party    # 外部帮助程序、分叉代码和其他第三方工具
```

web服务和rpc服务目录结构基本一致，其中有一些目录是web服务独有(internal目录下的routers、handler、types)，有一些目录是rpc服务独有(internal目录下的service)。
