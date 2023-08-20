
<p align="center">
<img width="500px" src="https://raw.githubusercontent.com/zhufuyi/sponge_doc/main/assets/img/logo.png">
</p>

<div align=center>

[![Go Report](https://goreportcard.com/badge/github.com/zhufuyi/sponge)](https://goreportcard.com/report/github.com/zhufuyi/sponge)
[![codecov](https://codecov.io/gh/zhufuyi/sponge/branch/main/graph/badge.svg)](https://codecov.io/gh/zhufuyi/sponge)
[![Go Reference](https://pkg.go.dev/badge/github.com/zhufuyi/sponge.svg)](https://pkg.go.dev/github.com/zhufuyi/sponge)
[![Go](https://github.com/zhufuyi/sponge/workflows/Go/badge.svg?branch=main)](https://github.com/zhufuyi/sponge/actions)
[![License: MIT](https://img.shields.io/github/license/zhufuyi/sponge)](https://img.shields.io/github/license/zhufuyi/sponge)

</div>

<br>

[sponge](https://github.com/zhufuyi/sponge) 是一个集成了`自动生成代码`、`微服务框架`、`通用基础开发框架`的golang生产力工具。sponge拥有丰富的生成代码命令，生成不同的功能代码可以组合成完整的服务(类似人为打散的海绵细胞可以自动重组成一个新的海绵)。代码解耦模块化设计，很容易构建出从开发到部署的完整工程项目，让你开发web或微服务项目轻而易举、事半功倍，使用golang也可以"低代码开发"。

<br>

**📖 功能特点：**

> 🔸生成代码命令UI界面化，简单易用。
> 
> 🔸自动合并新增模板代码，实现api接口"低代码开发"。
>
> 🔸统一错误码，支持http与grpc错误码相互转换。
>
> 🔸代码解耦模块化设计，丰富功能组件开箱即用。
>
> 🔸Web框架 [gin](https://github.com/gin-gonic/gin)
>
> 🔸RPC框架 [grpc](https://github.com/grpc/grpc-go)
>
> 🔸配置解析 [viper](https://github.com/spf13/viper)
>
> 🔸配置中心 [nacos](https://github.com/alibaba/nacos)
>
> 🔸日志组件 [zap](https://go.uber.org/zap)
>
> 🔸数据库orm组件 [gorm](https://gorm.io/gorm)
>
> 🔸缓存组件 [go-redis](https://github.com/go-redis/redis), [ristretto](github.com/dgraph-io/ristretto)
>
> 🔸自动化api接口文档 [swagger](https://github.com/swaggo/swag), [protoc-gen-openapiv2](https://github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2)
>
> 🔸鉴权 [jwt](https://github.com/golang-jwt/jwt)
>
> 🔸参数校验 [validator](https://github.com/go-playground/validator)
>
> 🔸自适应限流 [ratelimit](pkg/shield/ratelimit)
>
> 🔸自适应熔断 [circuitbreaker](pkg/shield/circuitbreaker)
>
> 🔸链路跟踪 [opentelemetry](https://go.opentelemetry.io/otel)
>
> 🔸指标监控 [prometheus](https://github.com/prometheus/client_golang/prometheus), [grafana](https://github.com/grafana/grafana)
>
> 🔸服务注册与发现 [etcd](https://github.com/etcd-io/etcd), [consul](https://github.com/hashicorp/consul), [nacos](https://github.com/alibaba/)
>
> 🔸自适应采集 [profile](https://go.dev/blog/pprof)
>
> 🔸资源统计 [gopsutil](https://github.com/shirou/gopsutil)
>
> 🔸代码规范检查 [golangci-lint](https://github.com/golangci/golangci-lint)
>
> 🔸持续集成部署 [jenkins](https://github.com/jenkinsci/jenkins), [docker](https://www.docker.com/), [kubernetes](https://github.com/kubernetes/kubernetes)

<br>

**🤝 反馈与交流**

> [!tip] 在使用过程中有任何问题和想法，请在这里提 [Issue](https://github.com/zhufuyi/sponge/issues)。

如果对您有个帮助给个**star⭐**，欢迎加入**go sponge微信交流群**。

<p>
<img width="300px" src="https://raw.githubusercontent.com/zhufuyi/sponge_doc/main/assets/img/wechat.jpg">
</p>
