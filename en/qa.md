
> [!tip] If there is a new issue you can add `https://go-sponge.com/qa/question` here, copy the url to access it on a new tab in your browser.

<br>

<h3>How to Set Validation Rules for Request Parameters?</h3>

<br>

**For `⓵ Web Services Created Based on SQL` Code**

**Answer:**

> Manually add tags to specify which parameters are required. For example, in the `internal/types/xxx_types.go` file, check the validation rules at [https://github.com/go-playground/validator](https://github.com/go-playground/validator).

```go
type CreateUserRequest struct {
    Name     string `json:"name" binding:"min=2"`
    Email    string `json:"email" binding:"email"`
    Password string `json:"password" binding:"md5"`
    Gender   int    `json:"password" binding:""`
}
```

> The `binding` tag specifies the validation rules for the field. If the rule is set to `required`, the field is mandatory. If the binding tag is empty or removed, it indicates an optional field.

<br>

**For Other 4 Ways of Creating Service Code**

**Answer:**

> Add annotations to the proto file to set validation rules. For example, in the `api/user/v1/user.proto` file, refer to [https://github.com/bufbuild/protoc-gen-validate](https://github.com/bufbuild/protoc-gen-validate).

```protobuf
import "validate/validate.proto";

message CreateUserRequest {
  string name = 1 [(validate.rules).string.min_len = 2];         // Username
  string email = 2 [(validate.rules).string.email = true];      // Email
  string password = 3 [(validate.rules).string.min_len = 10];   // Password
  int32 gender = 4; // Gender, optional
}
```

<br>

---

<h3>How to Use JWT Authorization Middleware?</h3>

<br>

**Answer:**

> Sponge supports commonly used and custom authorization methods. Choose one based on your requirements. Check [JWT Authorization Middleware Example](https://github.com/zhufuyi/sponge/blob/main/pkg/gin/middleware/README.md#jwt-authorization-middleware) for more details.

<br>

---

<h3>On Windows, Opening Downloaded Code ZIP File Results in an Empty Folder Using the Default Unzip Software</h3>

<br>

**Answer:**

> Install a decompression software like 7zip or WinRAR to extract the downloaded code ZIP file.

<br>

---

<h3>Error When Executing the `make proto` Command</h3>

<br>

**Error Type 1:**

```
api/types/types.proto: File not found.
api/xxx/v1/xxx.proto:5:1: Import "api/types/types.proto" was not found or had errors.
api/xxx/v1/xxx.proto:246:3: "types.Params" is not defined.
make: *** [Makefile:81: proto] Error 1
```

**Answer:**

> Change to the project code directory and run the command `make patch TYPE=types-pb`.

<br>

**Error Type 2:**

```bash
# xxx/internal/cache
internal\cache\xxx.go:40:38: undefined: model.CacheType
make: *** [Makefile:103: run] Error 1
```

**Answer:**

> Change to the project code directory and run the command `make patch TYPE=mysql-init`.

<br>

**Error Type 3:**

```
api/xxx/v1/xxx.proto:68:24: Option "(tagger.tags)" unknown. Ensure that your proto definition file imports the proto which defines the option.
make: *** [Makefile:81: proto] Error 1
```

**Answer:**

> Add the import dependency `import "tagger/tagger.proto";` in the `api/xxx/v1/xxx.proto` file.

<br>

**Error Type 4:**

```
api/xxx/v1/xxx.proto:232:24: Option "(validate.rules)" unknown ......
```

**Answer:**

> Add the import dependency `import "validate/validate.proto";` in the `api/xxx/v1/xxx.proto` file.

<br>

**Error Type 5:**

```
go: finding module for package github.com/zhufuyi/user/api/user/v1
go: finding module for package github.com/zhufuyi/user/api/types
github.com/zhufuyi/user/internal/service imports
        github.com/zhufuyi/user/api/user/v1: cannot find module providing package github.com/zhufuyi/user/api/user/v1: module github.com/zhufuyi/user/api/user: git ls-remote -q origin in D:\work\golang\package\pkg\mod\cache\vcs\95f49bd61e3a79c5db13de92b1180ce3b9787b1b3450a19ca7caaf2d9e0e0794: exit status 128

:
        fatal: Cannot prompt because user interactivity has been disabled.
        fatal: could not read Username for 'https://github.com': terminal prompts disabled
Confirm the import path was entered correctly.
If this is a private repository, see https://golang.org/doc/faq#git_https for additional information.
```

**Cause:**

> When generating the project code, the module name is specified as a domain + path (e.g., `github.com/zhufuyi/user`). The `make proto` command internally executes `go mod tidy`. If the module name is `github.com/zhufuyi/user`, and the code in the service or handler directory imports `github.com/zhufuyi/user`, but the local `github.com/zhufuyi/user` is not found, executing `go get github.com/zhufuyi/user` will result in an error.

**Answer:**

> Run the command `bash scripts/protoc.sh` in the terminal to ensure that the local `github.com/zhufuyi/user` code exists.

<br>

---

<h3>CORS Error When Requesting API in Swagger Interface</h3>

<br>

**Error Message:**

```
Failed to fetch.
Possible Reasons:

CORS
Network Failure
URL scheme must be "http" or "https" for CORS request.
```

**Cause:**

> If the default port in the configuration file is changed, or if the Swagger interface requests from a different host, while the server's Swagger base URL is still set to `localhost:8080`, it results in a CORS error.

**Answer:**

> - If it's `Web Services Created Based on SQL`, modify `localhost:8080` in the `main.go` file and then execute the `make docs` command.
> - If it's `Web Services Created Based on Protobuf`, modify `localhost:8080` in each proto file and then execute the `make proto` command.

<br>

---

<h3>Error When Executing Commands in Zsh</h3>

<br>

**Example Error Message:**

```
zsh: no matches found: --db-dsn=root:123456@(127.0.0.1:3306)/school
```

**Answer:**

> When using the `zsh` shell to execute sponge commands, make sure to enclose the `--db-dsn` value in double quotes, for example, `--db-dsn="root:123456@(127.0.0.1:3306)/school"`.

<br>

---

<h3>Request Parameters Field Values Are Empty When Making API Requests from the Frontend</h3>

<br>

In `Create Services Based on Protobuf`, `Create GRPC Gateway Service Based on Protobuf`, and `Create GRPC Service Based on Protobuf`, the following are common scenarios:

(1) Query parameters cannot be retrieved. For example, when making a request to the URL `http://localhost:8080/api/v1/getUser?name=Tom`, the server cannot retrieve the value of the `name` parameter.

**Answer:**

> Add tags to the fields defined in the message in the corresponding proto file. For example, `string name = 1 [(tagger.tags) = "form:/name/"];`, and then execute the `make proto` command.

<br>

(2) Path parameters cannot be retrieved. For example, when making a request to the URL `http://localhost:8080/api/v1/user/1`, where `1` is the value of the `id` path parameter, the server cannot retrieve the value of the `id` parameter.

**Answer:**

> Define the path parameter in the proto file, for example, `/api/v1/user/{id}`, and ensure that the message defining the path parameter includes the name of the path parameter `id`. Add a tag to the `id` field, for example, `int64 id = 1 [(tagger.tags) = "uri:/id/"];`, and then execute the `make proto` command.

<br>

(3) JSON names of request parameters are inconsistent. If the field names in the request parameters use snake_case, and the fields in the xxx.pb.go file have JSON tags in Camel Case, it may lead to inconsistency.

**Answer:**

> Either change the request parameter `my_email` to `myEmail` to match the xxx.pb.go field, or add a JSON tag to the corresponding field in the proto file, for example, `string my_email = 1 [json_name ="my_email"]`, and ensure consistency between the frontend and backend JSON names. Then, execute the `make proto` command.

<br>

---

<h3>How to Connect to Multiple MySQL Databases for Read-Write Separation</h3>

<br>

**Answer:**

Open the configuration file `configs/xxx.yml`.

(1) Example of Setting Up Master-Slave:

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

(2) Example of Setting Up Multiple Masters and Slaves:

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

<h3>Id Field Name Inconsistency Issue</h3>

<br>

**Cause:**

> The generated `id` field name by sponge is `ID` or `xxxID`, while the field generated by protoc plugin is `Id` or `xxxId`, causing inconsistency.

**Answer:**

> Be cautious with using copier for copying; instead, manually assign the value to the field.

<br>

---

<h3>Issue with Freezing during Execution of 'make' Command After Downloading Service Code on Linux</h3>

<br>

**Answer:**

> Open the `Makefile` file and comment out this line `PKG_LIST := $(shell go list ${PKG}/... | grep -v /vendor/ | grep -v /api/)`

<br>

---

<h3>How to Use Service Registration and Discovery</h3>

<br>

**Considerations:**

> - When testing API interfaces in the Goland IDE, stop using the proxy configured in the Golang IDE.
> - When starting the service in the terminal, stop using network proxies.

Service Configuration:

```yaml
app:
  # Do not use 127.0.0.1, use the actual IP or domain of the server. If the service registration and discovery platform (etcd, consul, nacos) is not deployed locally, monitoring checks for the host will fail.
  host: "192.168.3.79"
  registryDiscoveryType: "consul" # Can be etcd, consul, nacos; must be configured, e.g., consul

# If connecting to other grpc services in this service and using service discovery, configure as follows:
grpcClient:
  - name: "user" # Required for service discovery
    host: "127.0.0.1" # Optional
    port: 36790 # Optional
    registryDiscoveryType: "consul" # Can be etcd, consul, nacos; must be configured, e.g., consul
```

If using DTM with service registration and discovery, use the real IP or domain as well:

```yaml
MicroService:
   EndPoint: 'grpc://192.168.3.79:36790'
```

<br>

When using Nacos as the service registration and discovery, the `namespaceID` value must be consistent for both the service registration server and the service discovery client.

For example, if grpc service A registers its actual address with Nacos, the configuration under Nacos in service A's configuration file:

```yaml
nacosRd:
  ipAddr: "127.0.0.1"
  port: 8848
  namespaceID: "3454d2b5-2455-4d0e-bf6d-e033b086bb4c" # namespace id
```

Another grpc service B needs to get the real address of A service by name, in service B's configuration file under Nacos:

```yaml
nacosRd:
  ipAddr: "127.0.0.1"
  port: 8848
  namespaceID: "3454d2b5-2455-4d0e-bf6d-e033b086bb4c" # namespace id
```

The `namespaceID` values in the Nacos configurations of services A and B must be the same and can be simultaneously empty.

<br>

---

<h3>How to Retrieve Profiles from a Service Running in Production for Performance Analysis</h3>

The service generated by Sponge supports two methods, **http api interface** and **system signals**, for collecting profiles.

**(1) Collecting Profiles via HTTP API Interface**

> HTTP API functionality needs to be enabled. Set `enableMetrics: true` in the YAML file under the `configs` directory. The default route is `/debug/pprof`.

<br>

**(2) Collecting Profiles via System Signals**

```bash
# View the service PID by name (second column)
ps aux | grep service_name

# Send a signal to the service
kill -trap pid_value
```

After receiving a system signal notification, the service starts collecting profiles and saves them to the `/tmp/service_name_profile` directory. The default collection duration is 60 seconds, automatically stopping profile collection after 60 seconds. If you only want to collect for 30 seconds, send the first signal to start collection, and after approximately 30 seconds, send the second signal to stop profile collection, acting like a switch. By default, it collects six types of profiles: CPU, memory, goroutine, block, mutex, and threadcreate. The file format is `date_time_pid_service_name_profile_type.out`.

<br>

**(3) Adaptive Profile Collection**

> Resource monitoring functionality needs to be enabled. Set `enableStat: true` in the YAML file under the `configs` directory.

Combine the **system signal profile collection** with the **alarm functionality for resource statistics**. The alarm conditions are:

- Record the CPU usage of the program continuously for 3 times (default every minute), triggering an alarm when the average usage exceeds 80% for 3 times.
- Record the physical memory usage of the program for 3 times (default every minute), triggering an alarm when the average memory usage exceeds 80% for 3 times.
- If it continues to exceed the alarm threshold, send an alarm every 15 minutes by default.

Captured profiles are saved to the `/tmp/service_name_profile` directory.

<br>

---

<h3>Common Considerations</h3>

<br>

- In the code for `⓵ web service based on sql`, if you modify swagger annotations, you must execute the command `make docs`, and then restart the service with `make run` to take effect.
- When creating a web or grpc service `based on sql`, if you use an embedded model, deletion is soft delete; otherwise, it is a physical delete.
- In the code for `creating web or grpc service based on protobuf`, when executing the command `make proto`, the default json tag for structure fields in the generated xxx.pb.go code is without the omitempty property. If you want to keep omitempty, open scripts/protoc.sh and comment out this line `sponge del-omitempty --dir=$protoBasePath --suffix-name=pb.go > /dev/null`

<br>
