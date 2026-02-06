---
title: MoonBit CLI Argparser 設計仕様
publish: false
tags:
  - cli
  - parser
aliases:
  - MoonBit CLI Argparser 設計仕様
created: 2026-01-04T20:50:52+09:00
modified: 2026-02-05T14:07:15+09:00
---

# MoonBit CLI Argparser 設計仕様

## 概要

MoonBitでGit/Cargo風のCLIツールを構築するためのargparserライブラリ設計。 Haskellのoptparse-applicative/optparse-simpleの使いやすさを参考にしつつ、MoonBitの言語制約内で実現。

## 設計方針

### 基本アーキテクチャ

- **Options/Command/SubCommand構成**: Git風の階層的コマンド構造
- **型安全性**: `Parser[T]`で結果の型を保証
- **宣言的API**: ビルダーパターン + Smart Constructors
- **バリデーション**: オプション単位でvalidator関数を設定可能

### MoonBitの制約への対応

- **Trait systemの限界**: Associated TypesやType Parametersがないため、タグ付きユニオンで統一
- **Applicativeの代替**: ビルダーパターンでメソッドチェーンによる合成
- **エラーハンドリング**: `Result`型と`raise`の組み合わせ

## Core Types

```moonbit
// Phantom type for type safety
enum Completeness {
  Complete
  Incomplete
}

// Parser with result type
struct Parser[T] {
  name : String
  description : String
  options : Array[OptionSpec]
  positionals : Array[PositionalSpec]
  subcommands : Array[SubcommandSpec]
  builder : (ParsedArgs) -> T
}

// Option specifications
enum OptionSpec {
  StringOption(StringOptionDef)
  IntOption(IntOptionDef)
  BoolFlag(BoolFlagDef)
  StringListOption(StringListOptionDef)
}

struct StringOptionDef {
  key : String
  short : String?
  long : String?
  help : String
  metavar : String        // e.g. "FILE", "PATH"
  required : Bool
  default : String?
  validator : (String) -> Bool
}

struct IntOptionDef {
  key : String
  short : String?
  long : String?
  help : String
  metavar : String
  required : Bool
  default : Int?
  validator : (Int) -> Bool
}

struct BoolFlagDef {
  key : String
  short : String?
  long : String?
  help : String
  default : Bool
}

struct StringListOptionDef {
  key : String
  short : String?
  long : String?
  help : String
  metavar : String
  separator : String      // e.g. "," for --tags=foo,bar,baz
}

struct PositionalSpec {
  key : String
  help : String
  metavar : String
  required : Bool
  default : String?
}

struct SubcommandSpec {
  name : String
  parser : Parser[Unit]
}

// Parsed result
struct ParsedArgs {
  command_path : Array[String]
  values : Map[String, ArgValue]
  positionals : Array[String]
}

enum ArgValue {
  Str(String)
  Int(Int)
  Bool(Bool)
  StrList(Array[String])
}

// Error types
enum ParseError {
  MissingRequired(String)
  InvalidValue(String, String)
  UnknownOption(String)
  UnknownCommand(String)
  InvalidFormat(String)
  HelpRequested(Array[String])  // command path
}

type! ParseResult = Result[ParsedArgs, ParseError]
```

## Builder API

### 基本的なOption Builders

```moonbit
fn str_option(
  key : String,
  ~short? : String,
  ~long? : String,
  ~help : String = "",
  ~metavar : String = "STRING",
  ~required : Bool = false,
  ~default? : String,
  ~validator : (String) -> Bool = fn(_) { true }
) -> OptionSpec

fn int_option(
  key : String,
  ~short? : String,
  ~long? : String,
  ~help : String = "",
  ~metavar : String = "INT",
  ~required : Bool = false,
  ~default? : Int,
  ~validator : (Int) -> Bool = fn(_) { true }
) -> OptionSpec

fn flag(
  key : String,
  ~short? : String,
  ~long? : String,
  ~help : String = "",
  ~default : Bool = false
) -> OptionSpec

fn str_list_option(
  key : String,
  ~short? : String,
  ~long? : String,
  ~help : String = "",
  ~metavar : String = "LIST",
  ~separator : String = ","
) -> OptionSpec

fn positional(
  key : String,
  ~help : String = "",
  ~metavar : String = "ARG",
  ~required : Bool = true,
  ~default? : String
) -> PositionalSpec
```

### Smart Constructors (よく使うパターン)

```moonbit
// File path with validation
fn file_option(
  key : String,
  ~short? : String,
  ~long? : String,
  ~help : String = "",
  ~required : Bool = false,
  ~must_exist : Bool = false
) -> OptionSpec {
  let validator = if must_exist {
    fn(path : String) { file_exists(path) }
  } else {
    fn(_) { true }
  }
  
  str_option(
    key,
    short=short,
    long=long,
    help=help,
    metavar="FILE",
    required=required,
    validator=validator
  )
}

// Port number with range validation
fn port_option(
  key : String,
  ~short? : String,
  ~long? : String,
  ~help : String = "",
  ~default : Int = 8080
) -> OptionSpec {
  int_option(
    key,
    short=short,
    long=long,
    help=help,
    metavar="PORT",
    default=default,
    validator=fn(p) { p > 0 && p <= 65535 }
  )
}

// URL with basic validation
fn url_option(
  key : String,
  ~short? : String,
  ~long? : String,
  ~help : String = "",
  ~required : Bool = false
) -> OptionSpec {
  str_option(
    key,
    short=short,
    long=long,
    help=help,
    metavar="URL",
    required=required,
    validator=fn(s) { s.starts_with("http://") || s.starts_with("https://") }
  )
}
```

### Parser Builder

```moonbit
fn parser[T](
  name : String,
  ~description : String = "",
  ~builder : (ParsedArgs) -> T
) -> Parser[T] {
  {
    name,
    description,
    options: [],
    positionals: [],
    subcommands: [],
    builder
  }
}

fn add_option[T](self : Parser[T], opt : OptionSpec) -> Parser[T] {
  let mut new_options = self.options.copy()
  new_options.push(opt)
  { ..self, options: new_options }
}

fn add_positional[T](self : Parser[T], pos : PositionalSpec) -> Parser[T] {
  let mut new_positionals = self.positionals.copy()
  new_positionals.push(pos)
  { ..self, positionals: new_positionals }
}

fn add_subcommand[T](
  self : Parser[T],
  name : String,
  subparser : Parser[Unit]
) -> Parser[T] {
  let mut new_subcommands = self.subcommands.copy()
  new_subcommands.push({ name, parser: subparser })
  { ..self, subcommands: new_subcommands }
}

// Convenience: add multiple options at once
fn with_options[T](self : Parser[T], opts : Array[OptionSpec]) -> Parser[T] {
  let mut result = self
  for opt in opts {
    result = result.add_option(opt)
  }
  result
}
```

## Type-Safe Accessors

```moonbit
// With defaults
fn get_string(self : ParsedArgs, key : String, ~default : String = "") -> String {
  match self.values.get(key) {
    Some(Str(s)) => s
    _ => default
  }
}

fn get_int(self : ParsedArgs, key : String, ~default : Int = 0) -> Int {
  match self.values.get(key) {
    Some(Int(i)) => i
    _ => default
  }
}

fn get_bool(self : ParsedArgs, key : String) -> Bool {
  match self.values.get(key) {
    Some(Bool(b)) => b
    _ => false
  }
}

fn get_string_list(self : ParsedArgs, key : String) -> Array[String] {
  match self.values.get(key) {
    Some(StrList(arr)) => arr
    _ => []
  }
}

// Require versions (raise on missing)
fn require_string(self : ParsedArgs, key : String) -> String!ParseError {
  match self.values.get(key) {
    Some(Str(s)) => s
    _ => raise MissingRequired(key)
  }
}

fn require_int(self : ParsedArgs, key : String) -> Int!ParseError {
  match self.values.get(key) {
    Some(Int(i)) => i
    _ => raise MissingRequired(key)
  }
}
```

## 使用例: Eg-walker CLI

```moonbit
struct EgWalkerOpts {
  file : String
  verbose : Bool
  json_output : Bool
}

struct InsertOpts {
  text : String
  position : Int
  actor : String
}

struct DeleteOpts {
  start : Int
  end : Int
}

struct SyncOpts {
  remote : String
  dry_run : Bool
}

enum EgWalkerCommand {
  Insert(EgWalkerOpts, InsertOpts)
  Delete(EgWalkerOpts, DeleteOpts)
  Sync(EgWalkerOpts, SyncOpts)
}

fn egwalker_parser() -> Parser[EgWalkerCommand] {
  // Global options (inherited by all subcommands)
  let global_opts = [
    file_option("file", short="-f", long="--file", required=true, help="CRDT state file"),
    flag("verbose", short="-v", long="--verbose", help="Verbose output"),
    flag("json", long="--json", help="JSON output format")
  ]
  
  let insert_cmd = parser(
    "insert",
    description="Insert text at position",
    builder=fn(args) {
      let global = {
        file: args.require_string("file")!,
        verbose: args.get_bool("verbose"),
        json_output: args.get_bool("json")
      }
      let insert = {
        text: args.require_string("text")!,
        position: args.require_int("position")!,
        actor: args.get_string("actor", default="default")
      }
      Insert(global, insert)
    }
  )
  .with_options(global_opts)
  .with_options([
    str_option("text", short="-t", long="--text", required=true, help="Text to insert"),
    int_option("position", short="-p", long="--pos", required=true, help="Insert position"),
    str_option("actor", short="-a", long="--actor", default="default", help="Actor ID")
  ])
  
  let delete_cmd = parser(
    "delete",
    description="Delete text range",
    builder=fn(args) {
      let global = {
        file: args.require_string("file")!,
        verbose: args.get_bool("verbose"),
        json_output: args.get_bool("json")
      }
      let delete = {
        start: args.require_int("start")!,
        end: args.require_int("end")!
      }
      Delete(global, delete)
    }
  )
  .with_options(global_opts)
  .with_options([
    int_option("start", long="--start", required=true, help="Start position"),
    int_option("end", long="--end", required=true, help="End position")
  ])
  
  let sync_cmd = parser(
    "sync",
    description="Sync CRDT state with remote",
    builder=fn(args) {
      let global = {
        file: args.require_string("file")!,
        verbose: args.get_bool("verbose"),
        json_output: args.get_bool("json")
      }
      let sync = {
        remote: args.require_string("remote")!,
        dry_run: args.get_bool("dry-run")
      }
      Sync(global, sync)
    }
  )
  .with_options(global_opts)
  .with_options([
    url_option("remote", short="-r", long="--remote", required=true, help="Remote URL"),
    flag("dry-run", long="--dry-run", help="Dry run without actual sync")
  ])
  
  parser("eg-walker", description="Eg-walker CRDT operations", builder=fn(_) {
    abort("No subcommand specified")
  })
  .add_subcommand("insert", insert_cmd)
  .add_subcommand("delete", delete_cmd)
  .add_subcommand("sync", sync_cmd)
}

// Main execution
fn main {
  let p = egwalker_parser()
  
  match parse(p, @array.from_iter(/* CLI args */)) {
    Ok(Insert(global, insert)) => {
      if global.verbose {
        println("Inserting '\{insert.text}' at position \{insert.position}")
      }
      execute_insert(global.file, insert)
      if global.json_output {
        print_json_result()
      }
    }
    Ok(Delete(global, delete)) => {
      execute_delete(global.file, delete)
    }
    Ok(Sync(global, sync)) => {
      if sync.dry_run {
        println("Dry run: would sync with \{sync.remote}")
      } else {
        execute_sync(global.file, sync.remote)
      }
    }
    Err(HelpRequested(cmd_path)) => {
      println(generate_help(p, cmd_path))
    }
    Err(e) => {
      eprintln("Error: \{e}")
      eprintln("\nUse --help for usage information")
      exit(1)
    }
  }
}
```

## Parser Implementation (核心ロジック)

### 再帰的コマンドパーサー

```moonbit
fn parse[T](parser : Parser[T], args : Array[String]) -> ParseResult!ParseError {
  let parsed = parse_command(parser, args, [])?
  Ok(parsed)
}

fn parse_command[T](
  cmd : Parser[T],
  args : Array[String],
  command_path : Array[String]
) -> ParseResult!ParseError {
  let mut options : Map[String, ArgValue] = Map::new()
  let mut positionals : Array[String] = []
  let mut current_path = command_path.copy()
  current_path.push(cmd.name)
  
  // Initialize defaults
  for opt in cmd.options {
    match opt {
      StringOption(def) => {
        if let Some(default) = def.default {
          options[def.key] = Str(default)
        }
      }
      IntOption(def) => {
        if let Some(default) = def.default {
          options[def.key] = Int(default)
        }
      }
      BoolFlag(def) => {
        options[def.key] = Bool(def.default)
      }
      StringListOption(_) => {
        // No default for lists
      }
    }
  }
  
  let mut i = 0
  let mut parsing_options = true
  
  while i < args.length() {
    let arg = args[i]
    
    // Check for help flag
    if arg == "-h" || arg == "--help" {
      raise HelpRequested(current_path)
    }
    
    // "--" stops option parsing
    if arg == "--" {
      parsing_options = false
      i += 1
      continue
    }
    
    if parsing_options && arg.starts_with("-") {
      // Parse option
      let (key, value) = parse_option(cmd, arg, args, &mut i)?
      options[key] = value
    } else {
      // Could be subcommand or positional
      if let Some(subcmd) = find_subcommand(cmd, arg) {
        // Parse subcommand recursively
        let remaining = args.slice(i + 1, args.length())
        return parse_command(subcmd.parser, remaining, current_path)
      } else {
        // Positional argument
        positionals.push(arg)
      }
    }
    
    i += 1
  }
  
  // Validate required options
  validate_required_options(cmd, options)?
  
  // Validate required positionals
  validate_required_positionals(cmd, positionals)?
  
  Ok({ command_path: current_path, options, positionals })
}

fn find_subcommand[T](cmd : Parser[T], name : String) -> SubcommandSpec? {
  for sub in cmd.subcommands {
    if sub.name == name {
      return Some(sub)
    }
  }
  None
}

fn validate_required_options[T](cmd : Parser[T], options : Map[String, ArgValue]) -> Unit!ParseError {
  for opt in cmd.options {
    match opt {
      StringOption(def) => {
        if def.required && not(options.contains(def.key)) {
          raise MissingRequired(def.key)
        }
      }
      IntOption(def) => {
        if def.required && not(options.contains(def.key)) {
          raise MissingRequired(def.key)
        }
      }
      _ => ()
    }
  }
}

fn validate_required_positionals[T](cmd : Parser[T], positionals : Array[String]) -> Unit!ParseError {
  let required_count = cmd.positionals.iter()
    .filter(fn(p) { p.required })
    .count()
  
  if positionals.length() < required_count {
    raise MissingRequired("positional arguments")
  }
}
```

### オプションパース

```moonbit
fn parse_option[T](
  cmd : Parser[T],
  arg : String,
  args : Array[String],
  i : &mut Int
) -> Result[(String, ArgValue), ParseError] {
  if arg.starts_with("--") {
    parse_long_option(cmd, arg, args, i)
  } else {
    parse_short_option(cmd, arg, args, i)
  }
}

fn parse_long_option[T](
  cmd : Parser[T],
  arg : String,
  args : Array[String],
  i : &mut Int
) -> Result[(String, ArgValue), ParseError] {
  // Handle --key or --key=value
  let (key, inline_value?) = match arg.split_once('=') {
    Some((k, v)) => (k, Some(v))
    None => (arg, None)
  }
  
  let opt_spec = find_option_by_long(cmd, key)?
  
  match opt_spec {
    BoolFlag(def) => Ok((def.key, Bool(true)))
    StringOption(def) => {
      let value = get_option_value(inline_value, args, i, key)?
      validate_and_parse_string(def, value)
    }
    IntOption(def) => {
      let value = get_option_value(inline_value, args, i, key)?
      validate_and_parse_int(def, value)
    }
    StringListOption(def) => {
      let value = get_option_value(inline_value, args, i, key)?
      let items = value.split(def.separator)
      Ok((def.key, StrList(items)))
    }
  }
}

fn parse_short_option[T](
  cmd : Parser[T],
  arg : String,
  args : Array[String],
  i : &mut Int
) -> Result[(String, ArgValue), ParseError] {
  // Handle -v or -f value or -fvalue
  let flag = arg.substring(0, 2)
  let opt_spec = find_option_by_short(cmd, flag)?
  
  match opt_spec {
    BoolFlag(def) => Ok((def.key, Bool(true)))
    StringOption(def) => {
      let value = if arg.length() > 2 {
        Some(arg.substring(2))  // -fvalue
      } else {
        None
      }
      let value_str = get_option_value(value, args, i, flag)?
      validate_and_parse_string(def, value_str)
    }
    IntOption(def) => {
      let value = if arg.length() > 2 {
        Some(arg.substring(2))
      } else {
        None
      }
      let value_str = get_option_value(value, args, i, flag)?
      validate_and_parse_int(def, value_str)
    }
    StringListOption(def) => {
      let value = if arg.length() > 2 {
        Some(arg.substring(2))
      } else {
        None
      }
      let value_str = get_option_value(value, args, i, flag)?
      let items = value_str.split(def.separator)
      Ok((def.key, StrList(items)))
    }
  }
}

fn get_option_value(
  inline? : String,
  args : Array[String],
  i : &mut Int,
  flag : String
) -> Result[String, ParseError] {
  match inline {
    Some(v) => Ok(v)
    None => {
      i.val += 1
      if i.val >= args.length() {
        Err(InvalidFormat("Missing value for \{flag}"))
      } else {
        Ok(args[i.val])
      }
    }
  }
}

fn find_option_by_long[T](cmd : Parser[T], long : String) -> Result[OptionSpec, ParseError] {
  for opt in cmd.options {
    match opt {
      StringOption(def) => {
        if let Some(l) = def.long {
          if l == long {
            return Ok(opt)
          }
        }
      }
      IntOption(def) => {
        if let Some(l) = def.long {
          if l == long {
            return Ok(opt)
          }
        }
      }
      BoolFlag(def) => {
        if let Some(l) = def.long {
          if l == long {
            return Ok(opt)
          }
        }
      }
      StringListOption(def) => {
        if let Some(l) = def.long {
          if l == long {
            return Ok(opt)
          }
        }
      }
    }
  }
  Err(UnknownOption(long))
}

fn find_option_by_short[T](cmd : Parser[T], short : String) -> Result[OptionSpec, ParseError] {
  for opt in cmd.options {
    match opt {
      StringOption(def) => {
        if let Some(s) = def.short {
          if s == short {
            return Ok(opt)
          }
        }
      }
      IntOption(def) => {
        if let Some(s) = def.short {
          if s == short {
            return Ok(opt)
          }
        }
      }
      BoolFlag(def) => {
        if let Some(s) = def.short {
          if s == short {
            return Ok(opt)
          }
        }
      }
      StringListOption(def) => {
        if let Some(s) = def.short {
          if s == short {
            return Ok(opt)
          }
        }
      }
    }
  }
  Err(UnknownOption(short))
}

fn validate_and_parse_string(def : StringOptionDef, value : String) -> Result[(String, ArgValue), ParseError] {
  if not(def.validator(value)) {
    Err(InvalidValue(def.key, "validation failed"))
  } else {
    Ok((def.key, Str(value)))
  }
}

fn validate_and_parse_int(def : IntOptionDef, value : String) -> Result[(String, ArgValue), ParseError] {
  match value.parse_int() {
    Some(i) => {
      if not(def.validator(i)) {
        Err(InvalidValue(def.key, "validation failed"))
      } else {
        Ok((def.key, Int(i)))
      }
    }
    None => Err(InvalidValue(def.key, "expected integer"))
  }
}
```

## Help Generation (optparse-applicative風)

```moonbit
fn generate_help[T](parser : Parser[T], command_path : Array[String]) -> String {
  let mut lines = []
  
  // Usage line
  let usage = build_usage_line(parser, command_path)
  lines.push(usage)
  lines.push("")
  
  // Description
  if parser.description != "" {
    lines.push(parser.description)
    lines.push("")
  }
  
  // Available commands
  if not(parser.subcommands.is_empty()) {
    lines.push("Available commands:")
    let max_cmd_len = parser.subcommands.iter()
      .map(fn(s) { s.name.length() })
      .max()
      .unwrap_or(10)
    
    for subcmd in parser.subcommands {
      let cmd_padded = subcmd.name.pad_end(max_cmd_len + 2)
      lines.push("  \{cmd_padded}\{subcmd.parser.description}")
    }
    lines.push("")
  }
  
  // Positional arguments
  if not(parser.positionals.is_empty()) {
    lines.push("Arguments:")
    for pos in parser.positionals {
      let required_mark = if pos.required { "" } else { " (optional)" }
      lines.push("  \{pos.metavar.pad_end(20)}\{pos.help}\{required_mark}")
    }
    lines.push("")
  }
  
  // Options
  if not(parser.options.is_empty()) {
    lines.push("Options:")
    for opt in parser.options {
      lines.push(format_option_help(opt))
    }
    lines.push("")
  }
  
  // Footer
  lines.push("Use '\{command_path.join(" ")} <command> --help' for more information.")
  
  lines.join("\n")
}

fn format_option_help(opt : OptionSpec) -> String {
  match opt {
    StringOption(def) => {
      let flags = format_flags(def.short, def.long, Some(def.metavar))
      let required = if def.required { " (required)" } else { "" }
      let default = match def.default {
        Some(d) => " [default: \{d}]"
        None => ""
      }
      "  \{flags.pad_end(30)}\{def.help}\{required}\{default}"
    }
    IntOption(def) => {
      let flags = format_flags(def.short, def.long, Some(def.metavar))
      let required = if def.required { " (required)" } else { "" }
      let default = match def.default {
        Some(d) => " [default: \{d}]"
        None => ""
      }
      "  \{flags.pad_end(30)}\{def.help}\{required}\{default}"
    }
    BoolFlag(def) => {
      let flags = format_flags(def.short, def.long, None)
      "  \{flags.pad_end(30)}\{def.help}"
    }
    StringListOption(def) => {
      let flags = format_flags(def.short, def.long, Some(def.metavar))
      "  \{flags.pad_end(30)}\{def.help} (comma-separated)"
    }
  }
}

fn format_flags(short? : String, long? : String, metavar? : String) -> String {
  let mut parts = []
  
  if let Some(s) = short {
    if let Some(m) = metavar {
      parts.push("\{s} <\{m}>")
    } else {
      parts.push(s)
    }
  }
  
  if let Some(l) = long {
    if let Some(m) = metavar {
      parts.push("\{l}=<\{m}>")
    } else {
      parts.push(l)
    }
  }
  
  parts.join(", ")
}

fn build_usage_line[T](parser : Parser[T], command_path : Array[String]) -> String {
  let mut parts = ["Usage:", command_path.join(" ")]
  
  if not(parser.subcommands.is_empty()) {
    parts.push("<COMMAND>")
  }
  
  if not(parser.options.is_empty()) {
    parts.push("[OPTIONS]")
  }
  
  for pos in parser.positionals {
    if pos.required {
      parts.push("<\{pos.metavar}>")
    } else {
      parts.push("[\{pos.metavar}]")
    }
  }
  
  parts.join(" ")
}
```

## 実装優先順位

### Phase 1: Core Types & Basic Parsing

- [ ] ArgValue, OptionSpec, ParsedArgs の定義
- [ ] 基本的な Parser[T] 構造体
- [ ] シンプルな long option パース (`--key=value`)
- [ ] エラー型の定義

### Phase 2: Builder API

- [ ] str_option, int_option, flag の実装
- [ ] parser() コンストラクタ
- [ ] add_option, add_positional メソッド
- [ ] 基本的なテストケース

### Phase 3: Advanced Parsing

- [ ] Short option サポート (`-f value`, `-fvalue`)
- [ ] Subcommand の再帰的パース
- [ ] Positional arguments
- [ ] `--` による option parsing 終了

### Phase 4: Validation & Help

- [ ] Validator 関数のサポート
- [ ] Required option/positional の検証
- [ ] Help generation (basic)
- [ ] Help generation (advanced with formatting)

### Phase 5: Smart Constructors & Polish

- [ ] file_option, port_option, url_option
- [ ] with_options メソッド
- [ ] エラーメッセージの改善
- [ ] 型安全なアクセサの最適化

### Phase 6: Advanced Features (Optional)

- [ ] String list options with separators
- [ ] Option groups (mutual exclusivity)
- [ ] Environment variable fallback
- [ ] Completion script generation

## テスト戦略

### Unit Tests

```moonbit
test "parse long option with value" {
  let opt = str_option("input", long="--input")
  let result = parse_long_option(opt, "--input=file.txt", [], &mut 0)
  assert_eq!(result, Ok(("input", Str("file.txt"))))
}

test "parse short option with value" {
  let opt = str_option("file", short="-f")
  let args = ["-f", "test.txt"]
  let result = parse_short_option(opt, "-f", args, &mut 0)
  assert_eq!(result, Ok(("file", Str("test.txt"))))
}

test "validate required option" {
  let opts = [str_option("input", required=true)]
  let values = Map::new()
  assert_raises!(validate_required_options(opts, values))
}

test "parse subcommand" {
  let sub = parser("add", builder=fn(_) { () })
  let main = parser("git", builder=fn(_) { () })
    .add_subcommand("add", sub)
  let result = parse(main, ["git", "add"])
  assert_eq!(result?.command_path, ["git", "add"])
}
```

### Integration Tests

```moonbit
test "eg-walker insert command" {
  let p = egwalker_parser()
  let args = ["eg-walker", "-f", "state.json", "insert", "-t", "hello", "-p", "0"]
  match parse(p, args) {
    Ok(Insert(global, insert)) => {
      assert_eq!(global.file, "state.json")
      assert_eq!(insert.text, "hello")
      assert_eq!(insert.position, 0)
    }
    _ => fail("Expected Insert command")
  }
}

test "help generation" {
  let p = egwalker_parser()
  let help = generate_help(p, ["eg-walker"])
  assert!(help.contains("Usage: eg-walker <COMMAND>"))
  assert!(help.contains("insert"))
  assert!(help.contains("delete"))
  assert!(help.contains("sync"))
}
```

### Property-Based Tests (将来的に)

```moonbit
// QuickCheck風のテスト
property "parsing then serializing is identity" {
  forall (valid_args : Array[String]) {
    let parsed = parse(parser, valid_args)?
    let serialized = serialize(parsed)
    assert_eq!(parse(parser, serialized), parsed)
  }
}
```

## プロジェクト構成

```
moonbit-argparse/
├── src/
│   ├── types.mbt          # Core types
│   ├── builder.mbt        # Builder API
│   ├── parser.mbt         # Parsing logic
│   ├── help.mbt           # Help generation
│   ├── validators.mbt     # Common validators
│   └── smart_ctors.mbt    # Smart constructors
├── test/
│   ├── unit/
│   │   ├── parser_test.mbt
│   │   ├── builder_test.mbt
│   │   └── help_test.mbt
│   └── integration/
│       └── examples_test.mbt
├── examples/
│   ├── simple.mbt         # Simple CLI example
│   ├── git_like.mbt       # Git-style subcommands
│   └── egwalker.mbt       # Eg-walker example
└── moon.mod.json
```

## 参考資料

### Haskell Libraries

- [optparse-applicative](https://hackage.haskell.org/package/optparse-applicative)
- [optparse-simple](https://github.com/fpco/optparse-simple)
- [記事: Make CLI with Haskell in 2018](https://matsubara0507.github.io/posts/2018-05-10-make-cli-with-haskell-in-2018)

### MoonBit Documentation

- [公式ドキュメント](https://docs.moonbitlang.com/en/latest/)
- [Standard Library](https://mooncakes.io/docs/moonbitlang/core)

### 類似プロジェクト

- Rust: clap, structopt
- Go: cobra, flag
- Python: click, argparse

## 次のステップ

1. **プロジェクト初期化**
    
    ```bash
    moon new moonbit-argparse
    cd moonbit-argparse
    ```
    
2. **Phase 1実装開始**
    
    - `src/types.mbt` に Core Types を実装
    - 簡単なテストケースで動作確認
3. **Claude Codeで実装**
    
    ```bash
    # このMarkdownファイルを保存後
    claude code "moonbit-argparse-design.md を読んで、Phase 1から実装を開始してください"
    ```
    
4. **段階的な機能追加**
    
    - 各Phaseを順番に実装
    - テストを書きながら進める
    - Eg-walkerプロジェクトで実用テスト

## 設計上の注意点

### MoonBit特有の考慮事項

1. **文字列操作の制約**
    
    - `split_once`, `starts_with`, `substring` などの可用性を確認
    - 必要に応じて自前実装
2. **配列操作**
    
    - `Array.copy()`, `Array.slice()` の動作確認
    - Immutableな更新パターンの活用
3. **Map操作**
    
    - `Map.get()`, `Map.contains()` の戻り値型
    - `Map[String, ArgValue]` の型推論
4. **エラーハンドリング**
    
    - `Result` vs `raise` の使い分け
    - エラーメッセージの国際化対応（将来）

### パフォーマンス最適化

1. **Option検索の高速化**
    
    - 線形探索 → Map化の検討
    - 頻繁に使われるオプションのキャッシュ
2. **文字列アロケーション削減**
    
    - String builderの活用
    - 不要なcopyの削減
3. **再帰の最適化**
    
    - Tail call optimizationの確認
    - Subcommand深度の制限

---

**このドキュメントはClaude Codeに渡して実装を開始できます。** 実装中に設計変更があれば、このドキュメントを更新してください。