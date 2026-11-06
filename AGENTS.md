# AGENTS.md（全局偏好）

> 本文件只写个人通用偏好；每个项目的用途、命令、特殊约束写在项目自己的 AGENTS.md 里。
> 「必须 / 禁止」是硬性规则；「默认 / 优先」可在有充分理由时偏离，但要在回复中说明原因。

## 背景

我主要编写独立的 Go 单文件工具：CLI、自动化脚本、API 客户端、网络工具、后台服务、Web 管理工具。
目标：简单、稳定、可维护；不过度设计；优先标准库；代码可直接运行；便于单文件发布。

## 沟通与语言

- 所有回复、代码注释、日志消息、错误文本必须使用中文；标识符使用英文，禁止拼音命名。
- 新建文件时给出完整可运行代码，禁止伪代码；文件开头加两行说明（已有文件不要加）：

  ```go
  // 用途：批量检测接口状态
  // 用法：go run . --input urls.txt
  ```

- 修改已有代码时，回复只说明「改了什么、解决了什么问题」，不贴整份代码，不写无关解释。

## 工作方式

- 默认直接动手，不反复确认。需求有歧义且会影响实现方向时，只问一个问题；否则按最简方案实现，并在回复中写明所做的假设。
- 修改已有代码前，必须先阅读现有逻辑、理解数据流、检查调用关系，并优先复用已有实现。
- 最小修改：不随意重构，不改变已有功能，保留兼容性，不整体重排格式，不碰无关文件。
- 禁止凭空编造 API 或假设未知接口；不确定时先用 `go doc 包名.符号` 核实。
- 不引入需求之外的功能。

## 完成前必须验证

```bash
gofmt -l .        # 应无输出
go vet ./...
go build ./...
staticcheck ./... # 如已安装；可发现已废弃 API
```

无法运行时，在回复中说明原因。

## 默认不做（除非我明确要求）

Docker、CI/CD、Kubernetes、微服务拆分、数据库设计、前端框架（React / Vue / Next.js）、复杂目录与多层 package。
不主动写测试；需要时只用标准库 `testing`，不引入第三方测试框架。

## 环境与网络代理

- Go 1.26+（项目另有要求除外），使用 Go Modules：`go mod init project-name`。
- 优先使用新版标准库特性（`log/slog`、`slices`、`maps`、`for range` 整数、`ServeMux` 方法路由），禁止使用已废弃 API（如 `io/ioutil`）。
- 仅当下载依赖或访问外网失败时，对**单条命令**临时设置代理，禁止写入代码、`go env` 或配置文件：

  ```bash
  HTTPS_PROXY=http://127.0.0.1:10808 HTTP_PROXY=http://127.0.0.1:10808 go mod tidy
  # PowerShell：$env:HTTPS_PROXY="http://127.0.0.1:10808"
  ```

## Go 开发原则

- 优先：简单代码、明确流程、少量抽象。一个文件能解决的，不拆 package。
- 禁止：过度封装、无意义接口、复杂设计模式、为未来需求提前设计。
  - 同一逻辑出现 3 次以上再抽函数；接口只在有 2 个以上实现时才定义。
  - 单个函数不超过约 50 行，复杂逻辑拆成小函数。
- 优先标准库。新增依赖前必须确认：标准库无法解决、项目维护活跃、依赖稳定；禁止为几十行代码引入大型库。
- 命名：包内函数和变量用小驼峰（`parseConfig`），仅需导出时才用大驼峰；缩写保持大写（`userID`、`baseURL`）；getter 不加 `Get` 前缀；常量用 `MaxRetryCount` 形式，不用全大写。
- `main` 只负责启动，必须使用 `run() error` 模式，保证 `defer` 能执行：

  ```go
  func main() {
  	if err := run(); err != nil {
  		slog.Error("程序退出", "error", err)
  		os.Exit(1)
  	}
  }

  func run() error {
  	// 收到 Ctrl+C 或 SIGTERM 时取消 ctx，实现优雅退出
  	ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
  	defer stop()
  	return serve(ctx)
  }
  ```

## 错误处理与稳定性

- 必须处理所有错误；禁止用 `panic(err)` 处理普通业务错误；禁止在 `main` 之外使用 `log.Fatal` / `os.Exit`。
- 错误必须包含「操作名称 + 失败原因」，用 `%w` 包装；每一层只补充本层上下文，避免层层重复：

  ```go
  return fmt.Errorf("读取配置失败: %w", err)
  ```

- 外部数据（JSON、文件、API 返回、用户输入）必须校验，并处理文件不存在、解析失败、格式错误。
- 网络请求、文件操作、数据库操作、外部接口调用必须处理异常并设置超时。
- 长期运行程序：用 `signal.NotifyContext` 优雅退出，`http.Server` 用 `Shutdown`；`recover` 只放在 goroutine 顶层并记录日志，不要到处使用。

## 日志与输出

- 日志使用 `log/slog`，输出到 stderr；消息用中文，key 用英文，并带上关键上下文：

  ```go
  logger.Error("请求失败", "url", url, "error", err)
  ```

- CLI 的结果与进度属于程序输出，可用 `fmt.Fprintln(os.Stdout / os.Stderr)`；禁止用 `fmt.Println` 充当正式日志。
- 日志中禁止出现 API Key、Token、Cookie、Authorization、密码。

## 配置与安全

- 敏感信息（API Key、Token、Cookie、密码）禁止硬编码。来源优先级：环境变量 > 配置文件；命令行参数只用于非敏感项（会留在 shell 历史和进程列表里）。
- 不主动读取、输出或提交 `.env` 及含密钥的配置文件；提供 `config.example.json` 作为示例。
- 网络服务必须有输入验证、请求限制、超时（`http.Server` 设置 `ReadHeaderTimeout`）；默认监听 `127.0.0.1`，对外开放必须加鉴权。

## HTTP 与并发

- 所有 HTTP 请求必须：自建 `http.Client` 并设置 `Timeout`（禁止用默认 client）、检查错误、检查状态码；在 err 检查之后立即 `defer resp.Body.Close()`；用 `io.LimitReader` 限制读取大小。
- 重试最多 3 次并指数退避；禁止无限等待、无限重试。
- 并发必须有上限（带缓冲 channel 信号量或 worker pool）；用 `context` 控制取消与超时；保证 goroutine 能退出、资源被释放；禁止在循环里无限 `go task()`。
- 可使用 `sync.WaitGroup`（Go 1.25+ 可用 `wg.Go`）。

## CLI

- 默认使用标准库 `flag`；子命令很多（约 5 个以上）时才考虑 cobra。
- 必须支持 `--help`、`--version`、`--config`；版本号通过 `-ldflags "-X main.version=..."` 注入（`var version = "dev"`）。
- 批量处理、网络扫描、文件处理、数据转换等长任务必须显示进度（进度条或状态输出，写 stderr）。

## Web 与 API

- 默认使用 `net/http`（Go 1.22+ 支持 `"GET /items/{id}"` 路由）；只有路由很多或需要复杂中间件链时才考虑 Gin。
- 页面默认 `html/template` + HTMX，用 `//go:embed` 或内联常量打包，`htmx.min.js` 本地化，不依赖 CDN；禁止引入大型前后端分离架构。
- API 必须：参数验证、明确错误返回、统一响应格式、超时控制，并用 `http.MaxBytesReader` 限制请求体大小。
- JSON 必须定义结构体，避免 `map[string]interface{}`：

  ```go
  type Response struct {
  	Code int    `json:"code"`
  	Msg  string `json:"msg"`
  }
  ```

## 数据库

- 默认 `database/sql`；确需 ORM 时用 GORM，不为简单 CRUD 引入 ORM。
- 需要 SQLite 时优先纯 Go 驱动（如 `modernc.org/sqlite`），避免 CGO 破坏单文件发布。

## 项目结构

- 简单工具：`main.go`（可选 `config.go`）+ `go.mod` + `README.md`；能放一个文件就不拆。
- 复杂项目才使用 `cmd/`、`internal/`；确有对外复用需求才加 `pkg/`。
- `README.md` 必须有，且至少包含：用途、用法、配置项、示例。

## 构建与发布

- 开发运行：`go run .`
- 发布构建（单文件、可复现）：

  ```bash
  go build -trimpath -ldflags="-s -w " -o app .
  ```
