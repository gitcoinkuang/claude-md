# CLAUDE.md

## 背景

我平时主要编写独立 Python 脚本（single-file scripts），很少开发完整项目或 Python 包。

以下规则作为默认约定，适用于所有 Python 脚本开发任务。

目标： - 优先简单、稳定、可维护 - 避免过度设计 -
保持单文件脚本的便携性 - 输出代码必须可以直接运行

# 运行环境

-   Python 版本：3.11+（除非项目另有要求）
-   默认使用 uv 管理脚本依赖
-   优先使用 PEP 723 内联依赖声明，而不是维护 requirements.txt、虚拟环境

脚本依赖格式：

``` python
# /// script
# dependencies = ["requests", "rich"]
# ///
```

运行方式：

``` bash
uv run script.py
```

规则： - 只使用标准库时，不需要添加 PEP 723 依赖声明 - PEP 723
默认不要固定版本 - 只有存在明确兼容性问题时才指定版本 - 不主动创建
requirements.txt - 不主动创建 pyproject.toml - 不主动创建 setup.py

# 代码风格

-   所有代码注释必须使用中文
-   遵循 PEP 8
-   行宽默认 100 字符
-   函数必须添加类型注解（type hints）
-   变量和函数使用 snake_case 命名
-   常量使用全大写命名
-   类名使用 PascalCase 命名

docstring 只需要说明： - 脚本用途 - 基本使用方式

# 依赖原则

-   优先使用 Python 标准库
-   非必要不引入第三方依赖
-   添加第三方库前确认标准库无法满足需求
-   避免引入功能重复的依赖
-   优先选择维护活跃、社区成熟的库

# 命令行规范

-   命令行参数必须使用 argparse
-   不允许手写 sys.argv 解析

# 项目结构约定

默认： - 单文件脚本 - 不主动拆分模块 - 不主动创建包结构

除非明确要求，否则不要： - 创建多个 Python 文件 - 创建 package - 创建
src 目录 - 创建复杂项目结构

复杂逻辑： - 拆分函数 - 保持 main 简洁 - 不要把所有代码堆在 main()

必须包含：

``` python
if __name__ == "__main__":
    main()
```

# 日志规范

需要日志时使用 logging。

简单一次性脚本可以使用 print。

如果使用 RichHandler，并且 logger.info() 输出带颜色 markup：

必须设置：

``` python
RichHandler(markup=True)
```

# Rich 使用规范

适合使用： - 进度显示 - 表格输出 - 批量任务状态

不适合： - 简单脚本 - 单次输出 - 为了美观强行增加依赖

# 环境变量规范

如果需要： - API Key - Token - Cookie - 密码 - 私密配置/开关

必须： - 写入 .env - 自动检测并读取当前工作目录下的 .env 文件写入 os.environ

自动加载范式（优先标准库，避免强依赖 python-dotenv）：

```python
def _load_dotenv(env_path: str = ".env") -> None:
    if not os.path.exists(env_path):
        return
    try:
        with open(env_path, "r", encoding="utf-8") as f:
            for line in f:
                line = line.strip()
                if not line or line.startswith("#") or "=" not in line:
                    continue
                key, val = line.split("=", 1)
                key, val = key.strip(), val.strip().strip("'\"")
                if key and key not in os.environ:
                    os.environ[key] = val
    except Exception:
        pass

_load_dotenv()
```

禁止： - 只使用 os.getenv() 而不自动加载 .env - 硬编码敏感信息 - 日志输出敏感数据 - 提交 .env 文件

# 修改已有代码原则

修改已有脚本时：

必须先理解现有逻辑。

要求： - 不随意重构 - 不改变已有功能 - 不改变已有调用方式 -
保留已有兼容性 - 优先最小修改

检查： - Python版本兼容 - 第三方依赖 - API是否废弃 - 方法是否仍然可用

# 功能修改原则

不要主动增加用户未要求的功能。

禁止默认添加： - 缓存系统 - 数据库 - Web框架 - 多线程 - 异步架构 -
配置系统 - 插件系统

除非用户明确要求。

# 异常处理规范

禁止裸 except。

禁止：

``` python
except:
    pass
```

要求： - Exception 必须记录具体原因 - 不允许静默忽略异常 -
文件、网络、API操作必须捕获异常

# 网络请求规范

所有 HTTP 请求必须设置 timeout。

必须处理： - 请求失败 - 超时 - 网络异常 - 非200状态码 - JSON解析失败

禁止无限重试和无限并发请求。

# 数据处理规范

处理外部数据必须验证。

包括： - 文件内容 - JSON数据 - API返回 - 用户输入

要求： - 文件不存在需要提示 - JSON解析失败需要捕获 -
大文件避免一次性加载

# 并发规范

默认使用同步代码。

只有明确需要性能提升时才使用： - threading - multiprocessing - asyncio

使用并发时： - 限制并发数量 - 捕获任务异常 - 防止资源耗尽

# 安全规范

禁止： - 硬编码密码 - 硬编码 Token - 硬编码 API Key - 输出完整 Cookie -
输出 Authorization

日志需要隐藏敏感信息。

# 默认不做的事情

除非明确要求：

不要： - Flask/Django/FastAPI Web 服务 - pytest测试框架 - Docker配置 -
CI/CD配置 - requirements.txt - pyproject.toml - setup.py -
多文件项目结构

# FastAPI

-   当我明确要求使用 FastAPI 时，默认采用 FastAPI + Uvicorn + Jinja2 + HTMX 架构
-   项目必须包含 Web 前端界面，不允许只生成纯后端 API 服务
-   优先使用 Jinja2 进行服务端模板渲染，使用 HTMX 实现页面动态交互，减少不必要的 JavaScript 代码
-   如果页面非常简单，可以不使用 Jinja2，但仍必须提供可访问的 HTML 前端页面
-   除非明确要求，否则不要引入 React、Vue、Next.js 等前端框架
-   API 接口可以作为前端的数据交互方式存在，但不能替代完整的 Web 页面

## FastAPI 安全规范

- 生产环境默认关闭 FastAPI 自动文档：
  - 禁止暴露 `/docs`
  - 禁止暴露 `/redoc`
  - 禁止暴露 `/openapi.json`
- 如果需要 API 文档调试，必须：
  - 使用环境变量控制开启

示例：

```python
app = FastAPI(
    docs_url=None,
    redoc_url=None,
    openapi_url=None
)
```

# 代码检查

输出代码前必须检查：

-   Python语法
-   import是否正确
-   变量是否定义
-   类型是否正确
-   API是否可用
-   方法是否废弃
-   是否可以直接运行

# 终端体验

长时间运行任务必须提供进度显示。

优先使用：

``` python
rich.progress
```

适用于： - 批量处理 - 网络请求 - 文件扫描 - 数据转换

# 交付方式

默认： - 输出完整可运行代码 - 不输出伪代码 - 不只提供 diff 或代码片段

脚本开头保留用途说明和运行示例：

``` python
# 用途：批量检测接口状态
# Usage: uv run script.py --input urls.txt
```

修改代码时只说明： - 修改内容 - 解决的问题

不要输出大量无关解释。
