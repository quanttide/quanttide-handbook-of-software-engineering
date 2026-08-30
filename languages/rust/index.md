# Rust 语言解析器

## 设计背景

qtcloud-code 的语言解析器基于 tree-sitter 构建，当前支持 Rust、Python、Go、Dart、TypeScript 五种语言。项目处于早期阶段，语言层面尚未深度优化，主要通过提高测试覆盖率和内部自举逻辑来提升质量。

## 解析器架构

### 核心接口

所有语言解析器实现 `LanguageParser` trait：

```rust
pub trait LanguageParser {
    fn language_name(&self) -> &'static str;
    fn file_extensions(&self) -> &'static [&'static str];
    fn parse(&mut self, file_path: &Path, source: &str) -> Option<ParseResult>;
}
```

### 当前实现

以 Rust 解析器为例，每个语言需要定义结构体并实现 trait：

```rust
pub struct RustParser {
    parser: tree_sitter::Parser,
}

impl RustParser {
    pub fn new() -> Result<Self, String> {
        let mut parser = tree_sitter::Parser::new();
        parser
            .set_language(&tree_sitter_rust::LANGUAGE.into())
            .map_err(|e| format!("设置 Rust 语言失败: {}", e))?;
        Ok(Self { parser })
    }
}

impl LanguageParser for RustParser {
    fn language_name(&self) -> &'static str {
        "Rust"
    }

    fn file_extensions(&self) -> &'static [&'static str] {
        &["rs"]
    }

    fn parse(&mut self, file_path: &Path, source: &str) -> Option<ParseResult> {
        let tree = self.parser.parse(source, None)?;
        Some(ParseResult {
            file_path: file_path.to_string_lossy().to_string(),
            tree,
            source: source.to_string(),
        })
    }
}
```

## 优化方向

### 宏批量生成

当前每个解析器的结构体定义和 trait 实现存在大量重复代码。考虑使用宏来消除这些重复：

```rust
create_parser!(
    RustParser,
    "Rust",
    tree_sitter_rust::LANGUAGE,
    &["rs"]
);

create_parser!(
    PythonParser,
    "Python",
    tree_sitter_python::LANGUAGE,
    &["py", "py3"]
);

create_parser!(
    TypeScriptParser,
    "TypeScript",
    tree_sitter_typescript::LANGUAGE_TYPESCRIPT,
    &["ts", "tsx"]
);

create_parser!(
    CSharpParser,
    "C#",
    tree_sitter_c_sharp::LANGUAGE,
    &["cs"]
);
```

宏的适用场景是灵活地编写大量重复代码。实际使用中需要权衡：大多数情况下宏并非必要，只有当概念结构稳定且重复模式明确时才值得引入。

### 配置化语言定义

当前语言信息（名称、tree-sitter 对象、文件扩展名）硬编码在代码中。可考虑通过配置文件（如 TOML）来定义语言性质，使用户能够自行配置新语言支持，而无需修改代码。

这一设计方向与项目的扩展性需求一致。现阶段作为内源项目，仅硬编码了实际使用的语言，接口扩展的复杂度尚未成为瓶颈。

## 相关决策

- 解析器设计接口的扩展难度较高，当前阶段优先保证核心功能稳定
- 宏的使用需谨慎，避免过度抽象导致代码可读性下降
- 配置化方案待项目进入公开阶段后再评估必要性
