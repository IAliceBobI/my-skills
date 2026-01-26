---
description: "使用 rust-quality-guard skill 检查 Rust 代码中的错误容忍和掩盖错误问题"
---

# Rust 错误容忍检查

> **快捷方式**: 使用 `rust-quality-guard` skill 提供的自动化脚本
>
> 本命令调用 `myskills:rust-quality-guard` skill 的检查脚本，快速发现代码中可能掩盖错误的模式。

## 快速使用

```bash
# 使用 rust-quality-guard skill 提供的脚本
python3 scripts/check_error_tolerance.py

# 检查指定目录
python3 scripts/check_error_tolerance.py src/

# 检查其他项目
python3 scripts/check_error_tolerance.py ../my-project
```

## 脚本位置

脚本位于 `rust-quality-guard` skill 中：
- `scripts/check_error_tolerance.py` - 自动化检查脚本

如果当前项目中没有该脚本，可以通过以下方式获取：
```bash
# 假设 skill 安装在以下路径
cp /Users/chenwei/.claude/plugins/cache/my-marketplace/myskills/4.1.12/skills/rust-quality-guard/scripts/check_error_tolerance.py scripts/
```

## 检查内容

脚本会自动检查以下问题模式（详见 `rust-quality-guard` skill）：

### 🔴 高严重度问题
- `unwrap()` - 生产环境直接 panic
- 数据库操作静默失败 - `.execute(...).await.ok()`
- `unwrap_or_default()` - 掩盖真实错误
- `unwrap_or()` - 网络错误被掩盖
- `let _ =` - 忽略重要返回值
- `assert!` - release 模式被优化掉

### 🟡 中严重度问题
- `expect()` 缺少有用信息
- `panic!` 使用不当
- `ok()` 静默忽略错误
- `parse().unwrap()` - 解析失败 panic
- 直接数组索引 - 越界访问

### 🟢 低严重度问题
- `todo!()` 和 `unimplemented!()` 在生产代码
- 未使用 `#[must_use]` 警告

## FAIL FAST 原则

**最重要**的错误处理原则：错误必须传播，不能静默失败。

### ❌ 禁止模式

```rust
// 错误被记录但继续执行 - 这是错误的！
if let Err(e) = operation() {
    log::error!("Failed: {}", e);
}
// 继续执行...

// unwrap_or 静默回退
let value = risky_operation().unwrap_or(default_value);

// ok() 丢弃错误
let value = risky_operation().ok();

// let _ = 忽略结果
let _ = risky_operation();
```

### ✅ 正确模式

```rust
// 使用 ? 传播错误
operation()?;

// 添加错误上下文
operation().context("Failed to initialize service")?;

// 记录并传播
operation()
    .map_err(|e| {
        tracing::error!("Operation failed: {e}");
        e
    })?;
```

**记住**: 添加日志**不是**处理错误。错误必须传播！

## test-utils 特性

如果项目使用了 `test-utils` 特性：
- 测试辅助代码使用 `#[cfg(feature = "test-utils")]` 门控
- 运行测试时启用该特性: `--features test-utils`
- 生产构建不包含测试代码

## 详细文档

更多错误处理最佳实践和修复示例，请参考：
- `myskills:rust-quality-guard` skill
- `references/error_handling_patterns.md`
- `references/testing_best_practices.md`

## Clippy 配置

```toml
# clippy.toml
allow-expect-in-tests = true
allow-unwrap-in-tests = true

disallowed-methods = [
    { path = "std::result::Result::unwrap", reason = "Use ? operator instead" },
    { path = "std::option::Option::unwrap", reason = "Use ? operator or ok_or instead" },
]
```

运行 Clippy：
```bash
cargo clippy -- -W clippy::unwrap_used -W clippy::expect_used
```

## 检查清单

提交代码前确认：
- [ ] 生产代码中没有 `unwrap()`（除非有明确注释）
- [ ] 所有 `expect()` 都包含有用的上下文
- [ ] 数据库操作失败都有错误处理
- [ ] 关键业务逻辑（金额、余额等）不使用默认值掩盖错误
- [ ] 通过 `check_error_tolerance.py` 检查无高严重度问题
- [ ] 通过 `cargo clippy` 检查无警告
