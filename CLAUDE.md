# CLAUDE.md

## 背景
我平时主要写独立的 Python 脚本(single-file scripts),很少做成完整的包/项目结构。以下是默认约定,适用于所有项目;

## 运行环境
- Python 版本:3.11+(除非项目另有要求)
- 依赖管理:优先用 PEP 723 内联依赖声明 + `uv run` 执行,而不是维护 `requirements.txt` 或虚拟环境。脚本头部格式:

  ```python
  # /// script
  # dependencies = ["requests", "rich"]
  # ///
  ```

  运行方式:`uv run script.py`
- 只用标准库时,不需要任何依赖声明,也不需要这个块。

## 代码风格
- 代码注释必须使用中文
- 自由判断是否需要使用rich来增加数据客观性和终端观赏度
- 如果需要环境变量，必须写在.env文件里，然后脚本使用
- 当通过 logger.info() 打印带有颜色的字符串时，需要在初始化 RichHandler 时加入 markup=True 参数
- 遵循 PEP 8,行宽 100
- 函数签名带类型注解(type hints)
- 命令行参数用 `argparse`,不要手写 `sys.argv` 解析
- 优先标准库,非必要不引入第三方依赖
- 命名:变量/函数用 snake_case,常量全大写
- docstring 只需一段话说明脚本用途和用法,不要求完整的 Google/Numpy 风格

## 结构约定
- 默认单文件脚本,不主动拆成多文件/包,除非我明确要求
- 必须有 `if __name__ == "__main__":` 入口
- 逻辑复杂时拆成函数,不要全塞进 `main()`
- 需要输出日志时用 `logging` 模块;简单一次性脚本可以直接 `print`
- 错误处理:对可预期的错误给出清晰提示;不要用裸 `except:` 吞异常

## 默认不做的事
- 不引入 Flask/Django 等 web 框架,除非我明确要求做 web 服务
- 不自动生成 `requirements.txt` / `setup.py` / `pyproject.toml`,除非我要求做成可安装的包
- 不引入 pytest 等测试框架,除非我明确要求写测试
- 不主动把单文件脚本重构成多文件项目结构

## 交付方式
- 改完直接给出完整可运行的脚本,不要只给 diff 或片段
- 脚本开头保留一行简短注释说明用途和用法示例(如 `# Usage: uv run script.py --input file.csv`)