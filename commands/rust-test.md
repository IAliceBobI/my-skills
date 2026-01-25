---
name: rust-test
description: 执行指定的 Rust 测试
parameters:
  - name: name
    type: string
    required: true
    description: 要执行的测试函数名称
tags:
  - rust
  - testing
---

执行名为 **{{name}}** 的 Rust 测试。

## 工作流程

1. **搜索测试位置**：在项目中查找测试函数 `{{name}}`
2. **确认信息**：显示找到的 package、file 和 test 名称， 如果信息唯一就不要确认了。
3. **执行测试**：运行测试并显示完整输出

## 执行的命令

```bash
# 如果项目使用 test-utils 特性,需要添加 --features test-utils
cargo test --package <package> --test <file> --features test-utils -- {{name}} --exact --nocapture
```

**注意**: 如果项目没有使用 test-utils 特性,可以省略 `--features test-utils` 参数。

## test-utils 特性检测

在执行测试前,会自动检测项目是否使用了 `test-utils` 特性:

```bash
# 检查 Cargo.toml 中是否声明了 test-utils 特性
grep -r "test-utils" --include="Cargo.toml" .
```

**如果检测到 test-utils 特性**:
- 自动在测试命令中添加 `--features test-utils`
- 确保测试能够访问源码中的测试辅助函数

**关于 test-utils 的更多信息**:
- test-utils 是一个条件编译特性,用于在源码中提供测试辅助功能
- 使用 `#[cfg(feature = "test-utils")]` 门控的代码只在测试时编译
- 生产构建不会包含这些测试辅助代码,减小二进制大小

## 注意事项

- 测试名称必须完全匹配
- 支持 `#[test]` 和 `#[tokio::test]` 等测试宏
- `--nocapture` 选项会显示 println! 输出
- 如果找到多个匹配的测试，会询问用户选择哪一个
- 如果没有找到，会提示用户检查名称
