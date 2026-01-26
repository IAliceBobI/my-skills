---
name: rust-test
description: 使用 rust-quality-guard skill 执行指定的 Rust 测试
parameters:
  - name: name
    type: string
    required: true
    description: 要执行的测试函数名称
tags:
  - rust
  - testing
---

# Rust 测试执行

> **快捷方式**: 使用 `rust-quality-guard` skill 提供的 `run_rust_tests.py` 脚本

## 快速使用

```bash
# 使用 rust-quality-guard skill 提供的脚本
python3 scripts/run_rust_tests.py {{name}}

# 如果项目使用 test-utils 特性
python3 scripts/run_rust_tests.py {{name}} --features test-utils
```

## 脚本位置

脚本位于 `rust-quality-guard` skill 中：
- `scripts/run_rust_tests.py` - 测试执行和分析脚本

如果当前项目中没有该脚本：
```bash
cp /Users/chenwei/.claude/plugins/cache/my-marketplace/myskills/4.1.12/skills/rust-quality-guard/scripts/run_rust_tests.py scripts/
```

## 脚本功能

`run_rust_tests.py` 会自动：

1. **环境检测**：
   - 检测是否安装 `cargo-nextest`
   - 检测项目是否使用 `test-utils` 特性
   - 检测 `.config/nextest.toml` 配置文件

2. **搜索测试**：
   - 在项目中查找测试函数 `{{name}}`
   - 显示找到的 package、file 和 test 名称

3. **执行测试**：
   - 自动选择最佳工具（cargo-nextest 或 cargo test）
   - 显示详细输出和错误堆栈
   - 提供失败分析和修复建议

## 脚本使用示例

```bash
# 运行所有测试
python3 scripts/run_rust_tests.py

# 运行指定测试
python3 scripts/run_rust_tests.py test_login

# 启用 features
python3 scripts/run_rust_tests.py --features "test-utils"
python3 scripts/run_rust_tests.py --all-features

# 指定包
python3 scripts/run_rust_tests.py --package my-package test_login
```

## test-utils 特性

如果项目使用了 `test-utils` 特性：

### 在 Cargo.toml 中声明

```toml
[features]
test-utils = []
```

### 在源码中使用

```rust
#[cfg(feature = "test-utils")]
pub mod testing {
    pub fn create_test_client() -> Client {
        Client::new_for_testing()
    }
}
```

### 运行测试时启用

```bash
# ✅ 正确
cargo test --features test-utils
python3 scripts/run_rust_tests.py --features test-utils

# ❌ 错误（如果代码依赖 test-utils）
cargo test
```

## 关于 cargo-nextest

**cargo-nextest** 是比 `cargo test` 更快的测试运行器：

- ✅ 性能提升 20-30%
- ✅ 智能重试 flaky tests
- ✅ 更好的输出显示
- ✅ 超时控制
- ✅ JUnit 报告支持

安装方法：
```bash
cargo install cargo-nextest
```

## 直接使用 cargo 命令

如果不想使用脚本，也可以直接使用 cargo 命令：

### 使用 cargo-nextest (推荐)

```bash
cargo nextest run \
    --package <package> \
    --features test-utils \
    --test-name <test_name> \
    --no-capture \
    --success-output=immediate
```

### 使用 cargo test (fallback)

```bash
cargo test \
    --package <package> \
    --test <file> \
    --features test-utils \
    -- {{name}} --exact --nocapture
```

## 详细文档

更多测试最佳实践和调试技巧，请参考：
- `myskills:rust-quality-guard` skill
- `references/testing_best_practices.md` - 测试最佳实践
- `references/error_handling_patterns.md` - 错误处理模式

## 注意事项

- 测试名称必须完全匹配
- 支持 `#[test]` 和 `#[tokio::test]` 等测试宏
- `--no-capture` / `--nocapture` 选项会显示 println! 输出
- 脚本会自动处理多个匹配测试的情况
- 如果找不到测试，会提示检查名称

## 常见问题

### 测试失败时的分析流程

1. 脚本会单独执行失败的测试
2. 对比批量和单独执行结果：
   - 批量失败但单独通过 → 测试隔离问题 🟠
   - 批量和单独都失败 → 代码逻辑问题 🔴
3. 提供详细的修复建议

### 测试隔离问题

症状：批量执行失败，单独执行通过

原因：共享资源污染、状态冲突

解决方案：
- 添加测试隔离
- 使用独立的测试账户
- 在测试前重置状态

### 逻辑错误问题

症状：批量和单独执行都失败

解决方案：
- 查看错误堆栈信息
- 分析失败原因
- 修复代码逻辑
