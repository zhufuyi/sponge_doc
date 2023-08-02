
建议创建web服务时候使用**ui界面**方式，其他使用**命令行**方式，这样可以节省复制代码操作。当然可以只使用**ui界面**方式生成代码，也可以只用**命令行**方式生成代码，下面使用**命令行**方式介绍。

### 根据sql创建http服务

**根据sql创建http服务**的意思是在创建http服务工程项目中，顺便生成数据表的CRUD逻辑代码，使得生成的http服务代码编译运行就有api使用，当然后面新添加数据表可以继续生成新的api代码。

#### 创建一个表

根据mysql的数据表的创建信息来生成代码，需要先准备一个mysql服务([docker安装mysql](https://github.com/zhufuyi/sponge/blob/main/test/server/mysql/docker-compose.yaml))，例如mysql有一个数据库school，数据库下有一个数据表teacher，如下面sql所示：

```sql
CREATE DATABASE IF NOT EXISTS school DEFAULT CHARSET utf8mb4 COLLATE utf8mb4_unicode_ci;

use school;

create table teacher
(
    id          bigint unsigned auto_increment
        primary key,
    created_at  datetime     null,
    updated_at  datetime     null,
    deleted_at  datetime     null,
    name        varchar(50)  not null comment '用户名',
    password    varchar(100) not null comment '密码',
    email       varchar(50)  not null comment '邮件',
    phone       varchar(30)  not null comment '手机号码',
    avatar      varchar(200) not null comment '头像',
    gender      tinyint      not null comment '性别，1:男，2:女，其他值:未知',
    age         tinyint      not null comment '年龄',
    birthday    varchar(30)  not null comment '出生日期',
    school_name varchar(50)  not null comment '学校名称',
    college     varchar(50)  not null comment '学院',
    title       varchar(10)  not null comment '职称',
    profile     text         not null comment '个人简介'
)
    comment '老师';

create index teacher_deleted_at_index
    on teacher (deleted_at);
```

把SQL DDL导入mysql创建一个数据库school，school下面有一个表teacher。

<br>

#### 生成http服务代码

打开终端，执行命令：

```bash
sponge web http \
  --module-name=edusys \
  --server-name=edusys \
  --project-name=edusys \
  --repo-addr=zhufuyi \
  --db-dsn=root:123456@(192.168.3.37:3306)/school \
  --db-table=teacher \
  --out=./edusys
```

查看参数说明命令`sponge web http -h`，注意参数**repo-addr**是镜像仓库地址，默认值是`image-repo-host`，这个参数用在构建docker镜像、k8s的部署脚本上，如果使用[docker官方镜像仓库](https://hub.docker.com/)，只需填写注册docker仓库的用户名，如果使用私有仓库地址，需要填写完整仓库地址。

<br>

生成完整的http服务代码是在当前目录edusys下，目录结构如下：

```
.
├── build
├── cmd
│    └── edusys
│          └── initial
├── configs
├── deployments
│    ├── docker-compose
│    └── kubernetes
├── docs
├── internal
│    ├── cache
│    ├── config
│    ├── dao
│    ├── ecode
│    ├── handler
│    ├── model
│    ├── routers
│    ├── server
│    └── types
└── scripts
```

在edusys目录下的Makefile文件，集成了编译、测试、运行、部署等相关命令，切换到edusys目录下运行服务：

```bash
# 更新swagger文档
make docs

# 编译和运行服务
make run
```

复制 http://localhost:8080/swagger/index.html 到浏览器测试CRUD接口，如图3-1所示。

![http-swag](https://raw.githubusercontent.com/zhufuyi/sponge_doc/main/assets/img/http-swag.jpg)
*图3-1 http swagger文档界面*

<br>

服务默认只开启了指标采集接口、每分钟的资源统计信息，其他服务治理默认是关闭。在实际应用中，根据需要做一些操作：

- 使用redis作为缓存，打开配置文件`configs/edusys.yml`，把**cacheType**字段值改为redis，并且填写**redis**配置地址和端口。
- 默认限流、熔断、链路跟踪、服务注册与发现功能是关闭的，可以打开配置文件`configs/edusys.yml`开启相关功能，如果开启链路跟踪功能，必须填写jaeger配置信息；如果开启服务注册与发现功能，必须填写consul、etcd、nacos其中一种配置信息。
- 如果增加或修改了配置字段名称，执行命令 `sponge config --server-dir=./edusys`更新对应的go struct，只修改字段值不需要执行更新命令。
- 修改CRUD接口对应的错误码信息，打开`ingernal/ecode/teacher_http.go`，修改变量**teacherNO**值，这是唯一不重复的数值，返回信息说明根据自己需要修改，对teacher表操作的接口自定义错误码都在这里添加。

<br>

#### 生成handler代码

handler代码由对应表的model代码、dao代码、CRUD逻辑代码、注册路由、错误码等组成，生成的handler代码直接复制到web项目目录中，表示新增对表的增删改查接口。

一个服务中，通常不止一个数据表，如果添加了新数据表，生成的handler代码如何自动填充到已存在的http服务代码中呢，需要用到`sponge web handler`命令，例如添加了两个新数据表**course**和**teach**：

```sql
create table course
(
    id         bigint unsigned auto_increment
        primary key,
    created_at datetime    null,
    updated_at datetime    null,
    deleted_at datetime    null,
    code       varchar(10) not null comment '课程代号',
    name       varchar(50) not null comment '课程名称',
    credit     tinyint     not null comment '学分',
    college    varchar(50) not null comment '学院',
    semester   varchar(20) not null comment '学期',
    time       varchar(30) not null comment '上课时间',
    place      varchar(30) not null comment '上课地点'
)
    comment '课程';

create index course_deleted_at_index
    on course (deleted_at);


create table teach
(
    id           bigint unsigned auto_increment
        primary key,
    created_at   datetime    null,
    updated_at   datetime    null,
    deleted_at   datetime    null,
    teacher_id   bigint      not null comment '老师id',
    teacher_name varchar(50) not null comment '老师名称',
    course_id    bigint      not null comment '课程id',
    course_name  varchar(50) not null comment '课程名称',
    score        char(5)     not null comment '学生评价教学质量，5个等级：A,B,C,D,E'
)
    comment '老师课程';

create index teach_course_id_index
    on teach (course_id);

create index teach_deleted_at_index
    on teach (deleted_at);

create index teach_teacher_id_index
    on teach (teacher_id);
```

<br>

生成包含CRUD业务逻辑的handler代码：

```bash
sponge web handler \
  --db-dsn=root:123456@(192.168.3.37:3306)/school \
  --db-table=course,teach \
  --out=./edusys
```

查看参数说明命令`sponge web handler -h`，参数`out`是指定已存在的服务文件夹edusys，如果参数`out`为空，必须指定`module-name`参数，在当前目录生成handler子模块代码，然后把handler代码复制到文件夹edusys，两种方式效果都一样。

执行命令后，在`edusys/internal`目录下生成了course和teach相关的代码：

```
.
└── internal
      ├── cache
      ├── dao
      ├── ecode
      ├── handler
      ├── model
      ├── routers
      └── types
```

<br>

切换到edusys目录下执行命令运行服务：

```bash
# 更新swagger文档
make docs

# 编译和运行服务
make run
```

复制 http://localhost:8080/swagger/index.html 到浏览器测试CRUD接口，如图3-2所示。

![http-swag2](https://raw.githubusercontent.com/zhufuyi/sponge_doc/main/assets/img/http-swag2.jpg)
*图3-2 http swagger文档界面*

<br>

实际使用中需要修改自定义的CRUD接口返回错误码和信息，打开文件`ingernal/ecode/course_http.go`修改变量**courseNO**值，打开文件`ingernal/ecode/teach_http.go`修改变量**teachNO**值。

虽然生成了每个数据表的CRUD接口，不一定适合实际业务逻辑，就需要手动添加业务逻辑代码了，数据库操作代码填写到`internal/dao`目录下，业务逻辑代码填写到`internal/handler`目录下。

<br>

### 根据proto文件创建http服务

根据sql生成的web服务(只有增删改查接口)，满足不了实际业务需求(例如user服务需要注册、登录、登出接口)，为了让自定义接口也可以自动生成代码，因此产生了通过protobuf描述接口并生成代码方式，这种方式生成接口代码更有普适性。

**根据proto文件创建http服务**的意思是在创建http服务项目中，顺便生成proto对应的模板代码，使得生成的http服务代码编译运行就有api使用，当然后面添加新proto文件可以继续生成新的api。

#### 自定义接口

protocol buffers 语法规则看[官方文档 ](https://developers.google.com/protocol-buffers/docs/overview#syntax)，下面是一个示例文件 teacher.proto 内容，每个方法定义了路由和swagger文档的描述信息，实际应用中根据需要在 message 添加 tag 和 validate 的描述信息。

```protobuf
syntax = "proto3";

package api.edusys.v1;

import "google/api/annotations.proto";
import "protoc-gen-openapiv2/options/annotations.proto";

option go_package = "edusys/api/edusys/v1;v1";

// 生成*.swagger.json基本信息
option (grpc.gateway.protoc_gen_openapiv2.options.openapiv2_swagger) = {
  host: "localhost:8080"
  base_path: ""
  info: {
    title: "edusys api docs";
    version: "v0.0.0";
  };
  schemes: HTTP;
  schemes: HTTPS;
  consumes: "application/json";
  produces: "application/json";
};

service teacher {
  rpc Register(RegisterRequest) returns (RegisterReply) {
    // 设置路由
    option (google.api.http) = {
      post: "/api/v1/Register"
      body: "*"
    };
    // 设置路由对应的swagger文档
    option (grpc.gateway.protoc_gen_openapiv2.options.openapiv2_operation) = {
      summary: "注册用户",
      description: "提交信息注册",
      tags: "teacher",
    };
  }

  rpc Login(LoginRequest) returns (LoginReply) {
    option (google.api.http) = {
      post: "/api/v1/login"
      body: "*"
    };
    option (grpc.gateway.protoc_gen_openapiv2.options.openapiv2_operation) = {
      summary: "登录",
      description: "登录",
      tags: "teacher",
    };
  }
}

message RegisterRequest {
  string name = 1;
  string email = 2;
  string password = 3;
}

message RegisterReply {
  int64   id = 1;
}

message LoginRequest {
  string email = 1;
  string password = 2;
}

message LoginReply {
  string token = 1;
}
```

<br>

#### 生成http服务代码

打开终端，执行命令：

```bash
sponge web http-pb \
  --module-name=edusys \
  --server-name=edusys \
  --project-name=edusys \
  --repo-addr=zhufuyi \
  --protobuf-file=./teacher.proto \
  --out=./edusys
```

查看参数说明命令 `sponge web http-pb -h`，支持\*号匹配(示例`--protobuf-file=*.proto`)，表示根据批量proto文件生成代码，多个proto文件中至少包括一个service，否则不允许生成代码。

生成http服务代码的目录如下所示，与`sponge web http`生成的http服务代码目录有一些区别，新增了proto文件相关的**api**和**third_party**目录，internal目录下没有**cache**、**dao**、**model**、**handler**、**types**目录，其中**handler**是存放业务逻辑模板代码目录，通过命令会自动生成。

```
.
├── api
│    └── edusys
│          └──v1
├── build
├── cmd
│    └── edusys
│          └── initial
├── configs
├── deployments
│    ├── docker-compose
│    └── kubernetes
├── docs
├── internal
│    ├── config
│    ├── ecode
│    ├── routers
│    └── server
├── scripts
└── third_party
```

切换到edusys目录下执行命令运行服务：

```bash
# 生成*pb.go文件、生成handler模板代码、更新swagger文档
make proto

# 编译和运行服务
make run
```

复制 http://localhost:8080/apis/swagger/index.html 到浏览器测试接口，如图3-3所示，请求会返回500错误，因为模板代码(internal/handler/teacher_logic.go文件)直接调用`panic("implement me")`，这是为了提示要填写业务逻辑代码。

![http-pb-swag](https://raw.githubusercontent.com/zhufuyi/sponge_doc/main/assets/img/http-pb-swag.jpg)
*图3-3 http swagger文档界面*

<br>

#### 添加新的api接口

根据业务需求，需要添加新接口，分为两种情况：

**(1) 在原来proto文件添加新接口**

打开 `api/edusys/v1/teacher.proto`，例如添加**bindPhone**方法，并填写路由和swagger文档描述信息，完成添加一个新接口。

执行命令：

```bash
# 生成*pb.go文件、生成handler模板代码、更新swagger文档
make proto
```

在`internal/handler`和`internal/ecode`两个目录下生成新的模板文件，然后复制最新生成的模板代码到业务逻辑代码区：

- 在`internal/handler`目录下生成了后缀为 **.gen.日期时间** 模板代码文件(示例teacher_logic.go.gen.xxxx225619)，因为teacher_logic.go已经存在，不会把写的业务逻辑代码覆盖，所以生成新文件。打开文件 teacher_logic.go.gen.xxxx225619，把添加方法**bindPhone**接口的模板代码复制到teacher_logic.go文件中，然后填写业务逻辑代码。
- 在`internal/ecode`目录下生成了后缀为 **.gen.日期时间** 模板代码文件，把 **bindPhone** 接口错误码复制到 teacher_http.go 文件中。
- 删除所有后缀名为 **.gen.日期时间** 文件。

<br>

**(2) 在新proto文件添加接口**

例如新添加了**course.proto**文件，**course.proto**下的接口必须包括路由和swagger文档描述信息，查看 **章节3.2.1**，把**course.proto**文件复制到`api/edusys/v1`目录下，完成新添加接口。

执行命令：

```bash
# 生成*pb.go文件、生成handler模板代码、更新swagger文档
make proto
```

在 `internal/handler`、`internal/ecode`、 `internal/routers` 三个目录下生成**course**名称前缀的代码文件，只需做下面两个操作：

- 在`internal/handler/course.go`文件填写业务代码。
- 在`internal/ecode/course_http.go`文件修改自定义错误码和信息说明。

<br>

#### 完善http服务

把自定义接口描述信息写proto文件，根据proto生成的api只有模板代码，业务逻辑还没实现，因此没有`dao`、`cache`、`model`模块代码，需要使用者自己去实现，当然sponge也提供了生成`dao`、`cache`、`model`代码命令，如果当前项目刚好是使用mysql数据库和redis缓存，可以使用**sponge web dao命令**工具直接生成`dao`、`cache`、`model`代码，然后在代码模板上引用即可。

生成CRUD操作数据库代码命令：

```bash
sponge web dao \
  --db-dsn=root:123456@(192.168.3.37:3306)/school \
  --db-table=teacher \
  --include-init-db=true \
  --module-name=edusys \
  --out=./edusys
```

查看参数说明命令 `sponge web dao -h`，参数`--include-init-db`在一个服务中只使用一次，下一次生成`dao`代码时去掉参数`--include-init-db`，否则会造成无法生成最新的`dao`代码，原因是db初始化代码已经存在。

使用sponge生成的`dao`代码，需要做一些操作：

- 在服务的初始化和释放资源代码中加入mysql和redis，打开`cmd/edusys/initial/initApp.go`文件，把调用mysql和redis初始化代码反注释掉，打开`cmd/edusys/initial/registerClose.go`文件，把调用mysql和redis释放资源代码反注释掉，初始代码是一次性更改。
- 生成的`dao`代码，并不能和自定义方法**register**和**login**完全对应，需要手动在文件`internal/dao/teacher.go`补充代码，然后在`internal/handler/teacher.go`填写业务逻辑代码，业务代码中返回错误使用`internal/ecode`目录下定义的错误码，如果直接返回错误信息，请求端会收到unknown错误信息，也就是未定义错误信息。
- 默认使用了本地内存做缓存，改为使用redis作为缓存，在配置文件`configs/edusys.yml`修改字段**cacheType**值为redis，并填写redis地址和端口。

切换到edusys目录下再次运行服务：

```bash
go mod tidy

# 编译和运行服务
make run
```

打开 http://localhost:8080/apis/swagger/index.html 再次请求接口，可以正常返回数据了。

<br>

### 总结

生成http服务代码有mysql和proto文件两种方式：

- 根据sql生成的http服务代码包括每个数据表的CRUD接口代码，后续添加新接口，可以参考CRUD方式添加业务逻辑代码，新添加的接口需要手动填写swagger描述信息。
- 根据proto文件生成的http服务虽然不包括操作数据库代码，也没有CRUD接口逻辑代码，根据需要可以使用`sponge web dao`命令生成操作数据库代码。添加了新的接口，除了生成handler模板代码，swagger文档、路由注册代码、接口的错误码会自动生成。

两种方式都可以完成同样的http服务接口，根据实际应用选择其中一种，如果做后台管理服务，使用sql直接生产CRUD接口代码，可以少写代码。对于多数需要自定义接口服务，使用proto文件方式生成的http服务，这种方式自由度也比较高，写好proto文件之后，除了业务逻辑代码，其他代码都是通过插件生成。
