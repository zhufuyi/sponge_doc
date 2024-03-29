
> [!tip] 如果有新的问题可以在这里添加 `https://go-sponge.com/qa/question` ，复制url在浏览器新标签上访问。

<br>

<h3>如何设置请求参数校验规则？</h3>

<br>

**对于 `⓵基于sql创建web服务` 代码**

**解答：**

> 通过手动添加tag来指定哪些参数是必填的，例如在`internal/types/xxx_types.go`文件，具体检验规则看 [https://github.com/go-playground/validator](https://github.com/go-playground/validator )。

```go
type CreateUserRequest struct {
	Name     string `json:"name" binding:"min=2"`    // 用户名
	Email    string `json:"email" binding:"email"`   // 邮箱
	Password string `json:"password" binding:"md5"`  // 密码
	Gender int `json:"password" binding:""`          // 性别，非必须
}
```

> 其中binding是指定该字段的校验规则，如果指定规则是`required`表示必填字段，如果binding tag为空或删除binding表示则非必填字段。

<br>

**对于其他4中方式创建的服务代码**

**解答：**

> 在proto文件添加标注来设置校验规则，例如 `api/user/v1/user.proto`文件，具体检验规则看 [https://github.com/bufbuild/protoc-gen-validate](https://github.com/bufbuild/protoc-gen-validate)。

```protobuf
import "validate/validate.proto";

message CreateUserRequest {
  string name = 1 [(validate.rules).string.min_len = 2];         // 用户名
  string email = 2 [(validate.rules).string.email = true];          // 邮箱
  string password = 3 [(validate.rules).string.min_len = 10];   // 密码
  int32 gender = 4; // 性别，非必须
}
```

<br>

---

<h3>如何使用jwt鉴权中间件？</h3>

<br>

**解答：**


> sponge支持常用鉴权和自定义鉴权两种方式，根据实际选择一种即可，点击查看 [jwt鉴权使用示例](https://github.com/zhufuyi/sponge/blob/main/pkg/gin/middleware/README.md#jwt-authorization-middleware)。


<br>

---

<h3>在windows环境，在界面下载代码文件(zip)，使用windows自带解压软件直接打开下载的代码压缩文件为空</h3>

<br>

**解答：**

> 安装一个解压缩软件7zip或WinRAR来解压下载的代码文件(zip)。

<br>

---

<h3>执行命令make proto报错</h3>

<br>

**错误信息类型1：**

```
api/types/types.proto: File not found.
api/xxx/v1/xxx.proto:5:1: Import "api/types/types.proto" was not found or had errors.
api/xxx/v1/xxx.proto:246:3: "types.Params" is not defined.
make: *** [Makefile:81: proto] Error 1
```

**解答：**

> 切换到项目代码目录下，在终端执行命令 `make patch TYPE=types-pb`

<br>

**错误信息类型2：**

```bash
# xxx/internal/cache
internal\cache\xxx.go:40:38: undefined: model.CacheType
make: *** [Makefile:103: run] Error 1
```

**解答：**

> 切换到项目代码目录下，在执行命令 `make patch TYPE=init-mysql`，如果使用其他数据库类型，把命令中的mysql改为对应数据库类型名称。

<br>

**错误信息类型3：**

```
api/xxx/v1/xxx.proto:68:24: Option "(tagger.tags)" unknown. Ensure that your proto definition file imports the proto which defines the option.
make: *** [Makefile:81: proto] Error 1
```

**解答：**

> 在`api/xxx/v1/xxx.proto`文件导入依赖 `import "tagger/tagger.proto";`

<br>

**错误信息类型4：**

```
api/xxx/v1/xxx.proto:232:24: Option "(validate.rules)" unknown ......
```

**解答：** 

> 在`api/xxx/v1/xxx.proto`文件导入依赖`import "validate/validate.proto";`

<br>

**错误信息类型5：**

```
go: finding module for package github.com/zhufuyi/user/api/user/v1
go: finding module for package github.com/zhufuyi/user/api/types
github.com/zhufuyi/user/internal/service imports
        github.com/zhufuyi/user/api/user/v1: cannot find module providing package github.com/zhufuyi/user/api/user/v1: module github.com/zhufuyi/user/api/user: git ls-remote -q origin in D:\work\golang\package\pkg\mod\cache\vcs\95f49bd61e3a79c5db13de92b1180ce3b9787b1b3450a19ca7caaf2d9e0e0794: exit status 128:
        fatal: Cannot prompt because user interactivity has been disabled.
        fatal: could not read Username for 'https://github.com': terminal prompts disabled
Confirm the import path was entered correctly.
If this is a private repository, see https://golang.org/doc/faq#git_https for additional information.
```

**原因：**

> 生成项目代码时，module名称使用了域名+路径(例如`github.com/zhufuyi/user`)，make proto内部会先执行`go mod tidy`命令，因为mudule名称为`github.com/zhufuyi/user`，在service或handler目录下代码文件import了`github.com/zhufuyi/user`，本地没有找到`github.com/zhufuyi/user`，执行`go get github.com/zhufuyi/user`出现了错误。

**解答：**

> 在终端执行命令 `bash scripts/protoc.sh`，让本地存在`github.com/zhufuyi/user`代码就不会报错了。


<br>

---

<h3>在swagger界面请求api时出现跨域错误</h3>

<br>

**错误信息：**

```
Failed to fetch.
Possible Reasons:

CORS
Network Failure
URL scheme must be "http" or "https" for CORS request.
```

**原因：**

> 修改了配置文件的默认端口，或者时在跨主机的浏览器的swagger界面请求，而服务端默认的swagger的base URL仍然是localhost:8080，造成了这个跨域错误。

**解答：**

> - 如果是`基于sql创建web服务`，在main.go文件修改localhost:8080，然后执行命令`make docs`
> - 如果是`基于protobuf创建web服务`，在每个proto文件修改localhost:8080，然后执行命令`make proto`

<br>

---

<h3>在zsh执行命令生成代码报错</h3>

<br>

**例如错误信息：**

```
zsh: no matches found: --db-dsn=root:123456@(127.0.0.1:3306)/school
```

**解答：**

> 如果使用 `zsh` shell解析器执行sponge命令，参数--db-dsn值需要加双引号，示例：`--db-dsn="root:123456@(127.0.0.1:3306)/school"`。

<br>

---

<h3>在前端请求api，服务端接收到的请求参数字段值为空</h3>

<br>

在`⓷基于protobuf创建web服务`，`⓹基于protobuf创建grpc网关服务`中，下面是常见的三种情况:

(1) 请求查询参数无法获取，例如请求地址 http://localhost:8080/api/v1/getUser?name=Tom ，服务端获取请求参数`name`为空。

**解答：**

> 在对应的proto文件中的message定义的字段添加tag，例如 `string name = 1 [(tagger.tags) = "form:/"name/"];`，然后执行`make proto`命令。


<br>

(2) 路径参数无法获取，例如请求地址 http://localhost:8080/api/v1/user/1 ，其中1是路径参数id的值，服务端获取不到id值。

**解答：** 

> 在proto文件定义路径参数，例如 /api/v1/user/{id}，而且定义的message必须包含路径参数的名称id，该id名称必须添加tag说明，例如 `int64 id = 1 [(tagger.tags) = "uri:/"id/""];`，然后执行`make proto`命令。

<br>

(3) 请求参数的json名称不一致，请求参数的字段使用了蛇形命名(snake case)，而xxx.pb.go文件下的字段的json tag是峰命名(Camel Case)。

请求示例：
```bash
curl -X POST http://localhost:8080/api/v1/user/register -H 'Content-Type: application/json'-d \
'{
  "my_email": "string",
}'
```

xxx.pb.go文件下的字段的json tag值是`myEmail`，json名称与请求参数名称不一致。

**解答：**

> 把请求参数`my_email`改为`myEmail`可以正常访问。如果确实需要使用蛇形命名`my_email`，则需要在proto文件的message对应字段添加json tag，例如 `string my_email = 1 [json_name ="my_email"]`，确保前端和后端JSON名称一致，然后执行`make proto`命令。

<br>

---

<h3>如何连接多个mysql数据库，实现读写分离</h3>

<br>

**解答：**

打开配置文件`configs/xxx.yml`

**(1) 设置一主多从方式示例：**

```yaml
mysql:
  dsn: "root:123456@(127.0.0.1:3306)/account?parseTime=true&loc=Local&charset=utf8,utf8mb4"
  enableLog: true
  maxIdleConns: 3
  maxOpenConns: 100
  connMaxLifetime: 30
  slavesDsn:
    - "root:123456@(192.168.3.38:3306)/account?parseTime=true&loc=Local&charset=utf8,utf8mb4"
    - "root:123456@(192.168.3.39:3306)/account?parseTime=true&loc=Local&charset=utf8,utf8mb4"
```

**(2) 设置多主多从方式示例：**

```yaml
mysql:
  dsn: "root:123456@(127.0.0.1:3306)/account?parseTime=true&loc=Local&charset=utf8,utf8mb4"
  enableLog: true
  maxIdleConns: 3
  maxOpenConns: 100
  connMaxLifetime: 30
  slavesDsn:
    - "root:123456@(192.168.3.38:3306)/account?parseTime=true&loc=Local&charset=utf8,utf8mb4"
    - "root:123456@(192.168.3.39:3306)/account?parseTime=true&loc=Local&charset=utf8,utf8mb4"
  mastersDsn:
    - "root:123456@(127.0.0.1:3306)/account?parseTime=true&loc=Local&charset=utf8,utf8mb4"
    - "root:123456@(192.168.3.40:3306)/account?parseTime=true&loc=Local&charset=utf8,utf8mb4"
```


<br>

---

<h3>id字段名称不一致问题</h3>

<br>

**原因：**

> sponge生成的id字段名称是ID或xxxID，而protoc插件生成代码字段是Id或xxxId，本来是同一个字段但出现了不一致情况。 

**解答：**

> 注意不能使用copier复制，只能手工单独赋值该字段。

<br>

---

<h3>在linux下，下载服务代码后，执行make命令卡死</h3>

<br>

**解答：**

> 打开`Makefile`文件，注释这一行 `PKG_LIST := $(shell go list ${PKG}/... | grep -v /vendor/ | grep -v /api/)`

<br>

---

<h3>如何使用服务注册与发现</h3>

<br>

**注意事项：**

> - 在goland IDE测试api接口时，停止在golang IDE配置的代理。
> - 在终端启动服务时停止使用网络代理。

服务配置：

```yaml
app:
  # 不要写127.0.0.1，写服务器真实ip或域名，如果服务注册与发现平台(etcd、consul、nacos)不是在本地部署，定时监控检查host会失败。
  host: "192.168.3.79"
  registryDiscoveryType: "consul" # 可以填写etcd、consul、nacos，必须填写配置，例如consul

# 如果在本服务连接其他grpc服务，并且使用服务发现的话，配置如下：
grpcClient:
  - name: "user" # 必填，用于服务发现
    host: "127.0.0.1" # 可以不填
    port: 36790 # 可以不填
    registryDiscoveryType: "consul" # 可以填写 etcd、consul、nacos，必须填写配置，例如consul
```

如果使用dtm使用服务注册与发现，也要写真实ip或域名

```yaml
MicroService:
   EndPoint: 'grpc://192.168.3.79:36790'
```

<br>

使用nacos作为服务注册与发现时，注册服务端和服务发现端的配置namespaceID值必须一致

例如 grpc服务A要把自己服务真实地址注册到nacos，在服务A的配置文件下的nacos的配置

```yaml
nacosRd:
  ipAddr: "127.0.0.1"
  port: 8848
  namespaceID: "3454d2b5-2455-4d0e-bf6d-e033b086bb4c" # namespace id
```

另一个grpc服务B需要通过名称获取到A服务的真实地址，在服务B的配置文件下的nacos的配置

```yaml
nacosRd:
  ipAddr: "127.0.0.1"
  port: 8848
  namespaceID: "3454d2b5-2455-4d0e-bf6d-e033b086bb4c" # namespace id
```

服务A和服务B下的nacos配置namespaceID值必须相同，可以同时为空。

<br>

---

<h3>在线上运行的服务如何获取profile到本地进行性能分析</h3>

sponge生成的服务支持通过 **http api 接口** 和 **系统信号** 两种方式来采集profile。

**(1) 通过http api 接口采集profile**

> 需要开启http api 接口功能，在configs目录下yaml文件设置 `enableMetrics: true`，默认路由是`/debug/pprof`。

<br>

**(2) 通过系统信号采集profile**

```bash
# 通过名称查看服务pid(第二列)
ps aux | grep 服务名称

# 发送信号给服务
kill -trap pid值
```

服务收到系统信号通知后，开始采集profile并保存到`/tmp/服务名称_profile目录`，默认采集时长为60秒，60秒后自动停止采集profile，如果只想采集30秒，发送第一次信号开始采集，大概30秒后再发送第二次信号停止采集profile，类似开关。默认采集cpu、memory、goroutine、block、mutex、threadcreate六种类型profile，文件格式`日期时间_pid_服务名称_profile类型.out`，

<br>

**(3) 自适应采集profile**

> 需要开启资源统计功能，在configs目录下yaml文件设置 `enableStat: true`。

把**系统信号采集profile**与**资源统计的告警功能**结合起来实现，告警条件：

- 记录程序的cpu使用率连续3次(默认每分钟一次)，3次平均使用率超过80%时触发告警。
- 记录程序的物理内存使用率3次(默认每分钟一次)，3次平均占用系统内存超过80%时触发告警。
- 如果持续超过告警阈值，默认间隔15分钟发出一次告警。

采集的profile保存到`/tmp/服务名称_profile`目录。

<br>

---

<h3>常见注意事项</h3>

<br>

- 在`⓵基于sql创建web服务`代码中，如果修改了swagger注解后，必须执行命令`make docs`，然后重启服务`make run`才能生效。
- 如果`基于sql来创建web或grpc服务`时使用了嵌入Model，删除是软删除，否则是物理删除。
- 在`基于protobuf来创建web或grpc服务`代码中，执行命令`make proto`生成的xxx.pb.go代码中结构体字段的tag json默认是去掉omitempty属性，如果想保留omitempty，打开scripts/protoc.sh，注释掉这行命令`sponge del-omitempty --dir=$protoBasePath --suffix-name=pb.go > /dev/null`


