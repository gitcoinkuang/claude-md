# AGENTS.md（全局偏好 · Rust）

> 本文件只写个人通用偏好；每个项目的用途、命令、特殊约束写在项目自己的 AGENTS.md 里。
> 「必须 / 禁止」是硬性规则；「默认 / 优先」可在有充分理由时偏离，但要在回复中说明原因。

## 背景

我主要编写独立的 Rust 工具：CLI、自动化脚本、API 客户端、网络工具、后台服务、Web 管理工具。
目标：简单、稳定、可维护；不过度设计；依赖精简；代码可直接运行；便于发布为单个可执行文件。

## 沟通与语言

- 所有回复、代码注释、日志消息、错误文本必须使用中文；标识符使用英文，禁止拼音命名。
- 新建文件时给出完整可运行代码，禁止伪代码；`src/main.rs` 开头加两行说明（已有文件不要加）：

  ```rust
  //! 用途：批量检测接口状态
  //! 用法：cargo run -- --input urls.txt
  ```

- 修改已有代码时，回复只说明「改了什么、解决了什么问题」，不贴整份代码，不写无关解释。

## 工作方式

- 默认直接动手，不反复确认。需求有歧义且会影响实现方向时，只问一个问题；否则按最简方案实现，并在回复中写明所做的假设。
- 以下操作必须先征得同意：删除文件、新增预批准清单之外的 crate、升级依赖大版本、使用 nightly 或 `unsafe`、git push、破坏性命令、写入项目目录之外。
- 修改已有代码前，必须先阅读现有逻辑、理解数据流、检查调用关系，并优先复用已有实现。
- 最小修改：不随意重构，不改变已有功能，保留兼容性，不整体重排格式，不碰无关文件。
- 禁止凭空编造 crate、API 或 feature 名。不确定时查 docs.rs，或在本机用 `cargo doc`、阅读 `~/.cargo/registry/src` 中的源码核实；crate 的 API 随大版本变化很大，一律以 `Cargo.lock` 实际锁定的版本为准。
- 不引入需求之外的功能。

## 完成前必须验证

```bash
cargo fmt --check
cargo clippy --all-targets --locked -- -D warnings   # 警告一律视为错误（含 deprecated）
cargo build --locked                                 # 确保 Cargo.lock 不被悄悄修改
```

刚增删依赖后，先正常执行一次 `cargo build` 更新 `Cargo.lock`，再用 `--locked` 验证。有测试时再运行 `cargo test`。无法运行时，在回复中说明原因。

## 默认不做（除非我明确要求）

Docker、CI/CD、Kubernetes、微服务拆分、数据库设计、前端框架（React / Vue / Next.js）、workspace 与多 crate 拆分。
不主动写测试；需要时只用内置 `#[test]`，不引入额外测试框架。

## 环境与网络代理

- 使用最新稳定版 toolchain（rustup），edition 以 `cargo new` 默认值为准；禁止使用 nightly 特性。
- 新建项目：`cargo new project-name`；运行：`cargo run -- <参数>`。
- 仅当下载依赖或访问外网失败时，对**单条命令**临时设置代理，禁止写入代码或 cargo 配置文件：

  ```bash
  HTTPS_PROXY=http://127.0.0.1:10808 HTTP_PROXY=http://127.0.0.1:10808 cargo fetch
  # PowerShell：$env:HTTPS_PROXY="http://127.0.0.1:10808"
  ```

## Rust 开发原则

- 优先：简单代码、明确流程、少量抽象。一个 `src/main.rs` 能解决的，不拆模块。
- 禁止：过度封装、无意义 trait、复杂泛型、`macro_rules!`、为未来需求提前设计。
  - 优先拥有所有权的类型（`String`、`Vec`，必要时 `clone()`），避免复杂生命周期；不为性能提前优化。
  - 同一逻辑出现 3 次以上再抽函数；trait 只在有 2 个以上实现时才定义。
  - 单个函数不超过约 50 行，复杂逻辑拆成小函数。
  - 同步优先：只有确实需要并发 I/O 时才用 async / tokio。
- 命名遵循 Rust 惯例：函数、变量、模块用 `snake_case`；类型、trait、枚举用 `UpperCamelCase`；常量用 `SCREAMING_SNAKE_CASE`（如 `MAX_RETRY_COUNT`）；getter 不加 `get_` 前缀。
- 在 `main.rs` 开头加 `#![forbid(unsafe_code)]`；确需 `unsafe` 先征得同意，并写 `// SAFETY:` 中文注释。
- `main` 只做薄入口，错误直接返回，由 `anyhow` 输出错误链并以非零状态码退出：

  ```rust
  fn main() -> anyhow::Result<()> {
      init_logging();
      run()
  }

  fn run() -> anyhow::Result<()> {
      // 主流程
      Ok(())
  }
  ```

  需要 async 时改用 `#[tokio::main] async fn main()`，其余结构不变。

## 依赖原则

- 标准库能解决的，不引入 crate。注意 Rust 标准库不含 HTTP、JSON、异步运行时、命令行解析，这些使用下面的主流 crate。
- **预批准清单**（可直接使用，可按需增删）：`clap`（derive）、`serde`、`serde_json`、`toml`、`anyhow`、`thiserror`、`tokio`、`reqwest`、`tracing`、`tracing-subscriber`、`axum`、`askama`、`indicatif`、`rusqlite`。
- 清单之外的 crate，添加前必须在回复中说明：标准库和清单都无法解决、近期仍在维护、口碑稳定；禁止为几十行代码引入大型 crate。
- 用 `cargo add` 管理依赖，遵循 Cargo 默认的版本兼容约束，不要手写版本号；按需设置 `default-features = false` 以减小体积和编译时间。
- 二进制项目必须提交 `Cargo.lock`，依赖版本以它为准，保证不同时间构建的结果一致；`.gitignore` 忽略 `target/`。
- 禁止主动执行 `cargo update`；修改依赖前先查看现有 `Cargo.lock`，改完用 `git diff Cargo.lock` 确认只有目标 crate 及其必要的传递依赖发生变化，无关变化必须还原。
- 禁止为修复一个小问题顺便升级整个依赖树；确需升级时只升级目标 crate（`cargo update -p <crate>`）并在回复中说明，大版本升级仍须先征得同意。
- 新增 crate 时，若最新版本刚发布不足约两周，改用上一个稳定版本（`cargo add 名称@版本`）。

## 错误处理与稳定性

- 必须处理所有错误并用 `?` 向上传递；非测试代码禁止 `unwrap()` / `expect()`（编译期可证明的不变量除外，且 `expect` 必须写明中文原因）；禁止用 `panic!` 处理普通业务错误；禁止在 `main` 之外调用 `std::process::exit`。
- 应用层错误用 `anyhow`，并用 `.context()` 补充「操作名称 + 失败原因」；每一层只补充本层上下文，避免重复堆叠：

  ```rust
  let text = std::fs::read_to_string(&path)
      .with_context(|| format!("读取配置失败: {}", path.display()))?;
  ```

- 只有调用方需要区分错误类型时，才用 `thiserror` 定义错误枚举。
- 外部数据（JSON、文件、API 返回、用户输入）必须校验：用 serde 反序列化为结构体并处理失败，处理文件不存在、解析失败、格式错误；索引和切片用 `.get()`，避免越界 panic。
- 网络请求、文件操作、数据库操作、外部接口调用必须处理异常并设置超时。
- 长期运行程序：监听 `tokio::signal::ctrl_c()`（Unix 上同时监听 SIGTERM）优雅退出；Cargo.toml 中不要设置 `panic = "abort"`，保证单个任务 panic 不拖垮整个进程。

## 日志与输出

- 日志使用 `tracing`（`tracing-subscriber` 初始化，输出到 stderr）；消息用中文，字段名用英文，并带上关键上下文：

  ```rust
  tracing::error!(%url, error = ?err, "请求失败");
  ```

- CLI 的结果与进度属于程序输出，可用 `println!` / `eprintln!`；禁止用 `println!` 充当日志。
- 日志中禁止出现 API Key、Token、Cookie、Authorization、密码；含密钥字段的结构体不要直接 `derive(Debug)` 后打印。

## 配置与安全

- 敏感信息（API Key、Token、Cookie、密码）禁止硬编码。来源优先级：环境变量（`std::env::var`）> 配置文件（toml）；命令行参数只用于非敏感项（会留在 shell 历史和进程列表里）。环境变量缺失时，错误信息中要带上变量名。
- 不主动读取、输出或提交 `.env` 及含密钥的配置文件；提供 `config.example.toml` 作为示例。
- 网络服务必须有输入验证、请求限制、超时；默认只监听 `127.0.0.1`，对外开放必须加鉴权。

## HTTP 与并发

- 所有 HTTP 请求必须：用 `reqwest::Client::builder().timeout(...)` 构建并**复用**同一个 `Client`（禁止无超时的默认 client，禁止每次请求新建）；检查错误；用 `error_for_status()` 或显式检查状态码；限制响应体大小，不要无上限读取大响应。Body 由 RAII 自动释放，无需手动关闭。
- TLS 使用 rustls 而非 OpenSSL（便于静态链接和跨平台）：`default-features = false` 并启用 rustls 对应 feature，feature 名以所用版本的 docs.rs 为准。
- 重试最多 3 次并指数退避；禁止无限等待、无限重试。
- 并发必须有上限：用 `tokio::sync::Semaphore` 配合 `tokio::task::JoinSet`；禁止在循环里无限 `tokio::spawn`；用 `tokio::time::timeout` 控制超时，用 `tokio::select!` 响应退出信号，保证任务能取消、能退出。
- 禁止在 async 代码中调用阻塞操作（耗时同步 I/O 或 CPU 密集任务用 `spawn_blocking`）；禁止跨 `.await` 持有 `std::sync::Mutex` 的 guard；禁止 `unbounded_channel`，使用带容量的 `mpsc::channel(n)`。

## CLI

- 默认使用 `clap`（derive）；自动提供 `--help`；必须支持 `--version`（`#[command(version)]`，取自 Cargo.toml）和 `--config`。
- 批量处理、网络扫描、文件处理、数据转换等长任务必须显示进度（`indicatif` 进度条或状态输出，写 stderr）。

## Web 与 API

- 默认使用 `axum`（配合 `tokio`）；不主动使用其他 Web 框架，禁止大型前后端分离架构。
- 页面默认 `askama` + HTMX，用 `include_str!` / `include_bytes!` 把 `htmx.min.js` 等静态资源编进二进制，不依赖 CDN；禁止引入 React / Vue / Next.js。
- API 必须：参数验证、明确错误返回、统一响应格式、超时控制（简单场景用 `tokio::time::timeout`，需要统一中间件时再引入 `tower-http` 并说明）；请求体大小限制可用 axum 的 `DefaultBodyLimit` 调整。
- JSON 必须定义 serde 结构体，避免把 `serde_json::Value` 当主要数据结构：

  ```rust
  #[derive(Serialize)]
  struct ApiResponse<T> {
      code: i32,
      msg: String,
      data: Option<T>,
  }
  ```

## 数据库

- 默认 `rusqlite`（同步；启用 `bundled` feature 免装系统库，需要本机有 C 编译器）；需要异步或多种数据库时才用 `sqlx`。
- 不为简单 CRUD 引入 ORM（diesel / sea-orm）。

## 项目结构

- 简单工具：`Cargo.toml` + `src/main.rs` + `README.md`；需要时再加 `src/config.rs`。
- 复杂项目才拆 `src/lib.rs` 与模块；不主动创建 workspace。
- `README.md` 必须有，且至少包含：用途、用法、配置项、示例。

## 构建与发布

- 开发运行：`cargo run -- <参数>`；发布构建：`cargo build --release`。
- 发布体积与性能，在 Cargo.toml 中配置：

  ```toml
  [profile.release]
  lto = true
  codegen-units = 1
  strip = true
  # 仅 CLI 工具可加 panic = "abort"；常驻服务不要
  ```

- 需要静态链接或跨平台时按需指定 `--target`（如 `x86_64-unknown-linux-musl`，需配合 rustls）。
- 版本号只维护在 `Cargo.toml` 的 `version`，代码中用 `env!("CARGO_PKG_VERSION")` 读取，不要手写。
