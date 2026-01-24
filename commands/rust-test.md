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
cargo test --package <package> --test <file> -- {{name}} --exact --nocapture
```

## 注意事项

- 测试名称必须完全匹配
- 支持 `#[test]` 和 `#[tokio::test]` 等测试宏
- `--nocapture` 选项会显示 println! 输出
- 如果找到多个匹配的测试，会询问用户选择哪一个
- 如果没有找到，会提示用户检查名称
