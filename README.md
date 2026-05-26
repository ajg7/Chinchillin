# Chinchillin'

A programming language all about the world's best rodents, built in Rust as a learning project.

![chinchillin' logo](image.png)

---

## Table of Contents

- [Chinchillin'](#chinchillin)
  - [Table of Contents](#table-of-contents)
  - [Language Overview](#language-overview)
  - [Keyword Reference](#keyword-reference)
  - [Syntax Examples](#syntax-examples)
  - [Implementation Architecture](#implementation-architecture)
  - [Build \& Run](#build--run)
  - [Implementation Roadmap](#implementation-roadmap)
    - [Phase 0 – Bootstrap](#phase-0--bootstrap)
    - [Phase 1 – Lexer](#phase-1--lexer)
      - [`src/error.rs`](#srcerrorrs)
      - [`src/token.rs`](#srctokenrs)
      - [`src/lexer.rs`](#srclexerrs)
    - [Phase 2 – AST](#phase-2--ast)
    - [Phase 3 – Parser](#phase-3--parser)
      - [Helper methods (implement these first)](#helper-methods-implement-these-first)
      - [The precedence ladder](#the-precedence-ladder)
      - [Parsing `parse_call()`](#parsing-parse_call)
      - [Parsing statements](#parsing-statements)
    - [Phase 4 – Basic Interpreter](#phase-4--basic-interpreter)
      - [`src/environment.rs`](#srcenvironmentrs)
      - [`src/interpreter.rs` — Value enum](#srcinterpreterrs--value-enum)
      - [`src/interpreter.rs` — Interpreter struct](#srcinterpreterrs--interpreter-struct)
    - [Phase 5 – Control Flow](#phase-5--control-flow)
    - [Phase 6 – Functions \& Closures](#phase-6--functions--closures)
    - [Phase 7 – Arrays](#phase-7--arrays)
    - [Phase 8 – Classes \& Objects](#phase-8--classes--objects)
      - [Data structures](#data-structures)
      - [Class declaration (`Stmt::ClassDecl`)](#class-declaration-stmtclassdecl)
      - [Creating an instance (`adopt ClassName(args)`)](#creating-an-instance-adopt-classnameargs)
      - [Method calls and `chin` binding](#method-calls-and-chin-binding)
    - [Phase 9 – REPL \& Polish](#phase-9--repl--polish)
  - [Known Limitations](#known-limitations)
  - [End-to-End Test](#end-to-end-test)

---

## Language Overview

Chinchillin' is a dynamically-typed, procedural scripting language with a tree-walk interpreter written in Rust. Source files use the `.chin` extension.

**Execution pipeline:**

```
source text  →  Lexer  →  Vec<Token>  →  Parser  →  Vec<Stmt> (AST)  →  Interpreter  →  output
```

Each stage is a separate Rust module you build one at a time. By the end every stage produces something you can run and test.

---

## Keyword Reference

| Chinchillin'  | Equivalent        | Meaning                                        |
| ------------- | ----------------- | ---------------------------------------------- |
| `dust`        | `let` / `var`     | Declare a variable                             |
| `furry`       | `fn` / `function` | Declare a function                             |
| `bark`        | `print`           | Print a value to stdout                        |
| `scratch`     | `if`              | Conditional branch                             |
| `otherwise`   | `else`            | Alternate branch                               |
| `zoom`        | `for`             | Range-based for loop                           |
| `from` / `to` | —                 | Range delimiters for `zoom`                    |
| `nibble`      | `while`           | While loop                                     |
| `kick`       | `return`          | Return a value from a function                 |
| `fluff`       | `class`           | Declare a class                                |
| `adopt`       | `new`             | Create a new class instance                    |
| `chin`        | `self` / `this`   | Reference the current instance inside a method |
| `yep`         | `true`            | Boolean true                                   |
| `nope`        | `false`           | Boolean false                                  |
| `bald`        | `null` / `nil`    | Null value                                     |
| `and`         | `&&`              | Logical and                                    |
| `or`          | `\|\|`            | Logical or                                     |
| `not`         | `!`               | Logical not                                    |

---

## Syntax Examples

```
// Variables
dust x = 10;
dust name = "Chilly";
dust flag = yep;
dust nothing = bald;

// Print
bark(x);
bark("Hello, " + name + "!");

// If / else
scratch x > 5 {
    bark("big number!");
} otherwise {
    bark("small number.");
}

// While loop
dust i = 0;
nibble i < 5 {
    bark(i);
    i = i + 1;
}

// For loop (exclusive upper bound, like Rust 0..10)
zoom i from 0 to 10 {
    bark(i);
}

// Function
furry add(a, b) {
    wheek a + b;
}
dust result = add(3, 4);
bark(result);   // 7

// Closure
furry make_counter() {
    dust count = 0;
    furry inc() {
        count = count + 1;
        wheek count;
    }
    wheek inc;
}
dust c = make_counter();
bark(c());   // 1
bark(c());   // 2

// Array
dust arr = [10, 20, 30];
bark(arr[0]);    // 10
arr[1] = 99;
bark(arr[1]);    // 99

// Class
fluff Animal {
    furry init(name) {
        chin.name = name;
    }
    furry speak() {
        bark(chin.name + " says wheek!");
    }
}
dust pet = adopt Animal("Chilly");
pet.speak();   // Chilly says wheek!
```

---

## Implementation Architecture

```
chinchillin/
├── Cargo.toml
├── README.md
└── src/
    ├── main.rs          // Entry point: reads argv[1] as a file, or starts REPL
    ├── error.rs         // ChinchillinError enum (Lex, Parse, Runtime variants)
    ├── token.rs         // Token and TokenKind enums
    ├── lexer.rs         // Tokenizer: &str → Vec<Token>
    ├── ast.rs           // AST node types: Expr enum, Stmt enum
    ├── parser.rs        // Recursive descent parser: Vec<Token> → Vec<Stmt>
    ├── environment.rs   // Lexical scope: HashMap + Rc<RefCell<>> parent chain
    └── interpreter.rs   // Tree-walk evaluator + Value enum
```

---

## Build & Run

```bash
cargo build              # compile
cargo run myfile.chin    # run a file
cargo run                # start the REPL
cargo test               # run all unit tests
```

---

## Implementation Roadmap

Each phase ends with a runnable milestone and teaches specific Rust concepts.

---

### Phase 0 – Bootstrap

**Goal:** A compiling Rust project with all modules declared.

**Steps:**

1. `cargo init --name chinchillin` (already done — `Cargo.toml` and `src/main.rs` exist)
2. Create empty files: `src/error.rs`, `src/token.rs`, `src/lexer.rs`, `src/ast.rs`, `src/parser.rs`, `src/environment.rs`, `src/interpreter.rs`
3. Add `mod` declarations to `main.rs`:

   ```rust
   mod error;
   mod token;
   mod lexer;
   mod ast;
   mod parser;
   mod environment;
   mod interpreter;

   fn main() {
       println!("Hello, chin!");
   }
   ```

4. `cargo build` — everything compiles (empty modules are fine)

**Rust concepts introduced:**

- `mod` keyword — declares a module, which maps to a `.rs` file of the same name
- `cargo build` vs `cargo run` — build checks all modules compile; run also executes

---

### Phase 1 – Lexer

**Goal:** Turn raw source text into a flat list of tokens. After this phase you can print every token in a `.chin` file.

**Files:** `src/token.rs`, `src/lexer.rs`, `src/error.rs`

---

#### `src/error.rs`

Define the error type for the whole project:

```rust
use std::fmt;

pub enum ChinchillinError {
    Lex   { message: String, line: usize },
    Parse { message: String, line: usize },
    Runtime { message: String },
}

impl fmt::Display for ChinchillinError { ... }
impl std::error::Error for ChinchillinError {}
```

**Why `Display` and `Error`?** Implementing `std::fmt::Display` lets you print the error with `{}`. Implementing `std::error::Error` plugs into Rust's standard error ecosystem. Both are required traits for well-behaved error types.

---

#### `src/token.rs`

Two types live here:

**`TokenKind`** — an enum with every terminal symbol the lexer can produce:

```rust
#[derive(Debug, Clone, PartialEq)]
pub enum TokenKind {
    // Literals
    Number(f64),
    StringLit(String),
    Identifier(String),

    // Keywords
    Dust, Furry, Bark, Scratch, Otherwise,
    Zoom, From, To, Nibble, Wheek,
    Fluff, Adopt, Chin,
    Yep, Nope, Bald,
    And, Or, Not,

    // Operators
    Plus, Minus, Star, Slash, Percent,
    Equal, EqualEqual, BangEqual,
    Less, LessEqual, Greater, GreaterEqual,

    // Delimiters
    LeftParen, RightParen,
    LeftBrace, RightBrace,
    LeftBracket, RightBracket,
    Semicolon, Comma, Dot,

    Eof,
}
```

**`Token`** — pairs a kind with a line number (needed for error messages):

```rust
pub struct Token {
    pub kind: TokenKind,
    pub line: usize,
}
```

**Rust concepts introduced:**

- **Enums with data** — `Number(f64)` and `StringLit(String)` are variants that _carry_ a value. Each variant can hold a different type.
- **`#[derive(Debug, Clone, PartialEq)]`** — `Debug` gives you free `{:?}` printing; `Clone` lets you copy the token; `PartialEq` lets you use `==` in tests.
- **`pub`** — makes fields and types visible outside their module. Anything the parser needs to see must be `pub`.

---

#### `src/lexer.rs`

The `Lexer` struct and its `tokenize()` method:

```rust
pub struct Lexer {
    source: Vec<char>,   // source as individual characters
    pos: usize,          // current position
    line: usize,         // current line number (for error messages)
}

impl Lexer {
    pub fn new(source: &str) -> Self { ... }
    pub fn tokenize(&mut self) -> Result<Vec<Token>, ChinchillinError> { ... }
}
```

**Why `Vec<char>` instead of `&str`?**
Rust strings are UTF-8 encoded bytes — you cannot index into a `&str` by position like `source[3]`. Collecting into `Vec<char>` lets you index safely: `self.source[self.pos]`.

**How `tokenize()` works:**

```
loop:
  skip_whitespace_and_comments()
  if at end → push Eof token, break
  else       → scan_token() → push token
```

**Scanning individual tokens:**

```rust
fn scan_token(&mut self) -> Result<Token, ChinchillinError> {
    let ch = self.advance();   // consume next char
    match ch {
        '+' => Token(Plus),
        '-' => Token(Minus),
        '=' => {
            if next char is '=' { advance(); Token(EqualEqual) }
            else                { Token(Equal) }
        }
        '"'            => scan_string(),
        '0'..='9'      => scan_number(ch),
        'a'..='z'|'_' => scan_identifier_or_keyword(ch),
        _              => Lex error
    }
}
```

**Keyword detection** — when you finish scanning an identifier, check if it matches a known keyword:

```rust
match word.as_str() {
    "dust"      => TokenKind::Dust,
    "furry"     => TokenKind::Furry,
    "bark"      => TokenKind::Bark,
    // ... etc
    _           => TokenKind::Identifier(word),
}
```

**Skipping comments** — `//` comments: advance until `\n`. Increment `self.line` every time you see `\n` (in the comment skipper _and_ inside multi-line strings).

**Helper methods to implement:**

- `fn advance(&mut self) -> char` — consume and return `source[pos]`, then increment `pos`
- `fn peek_is(&self, ch: char) -> bool` — look at `source[pos]` without consuming

**Return type:** `Result<Vec<Token>, ChinchillinError>` — if any character is unexpected, return `Err(ChinchillinError::Lex { ... })`. The `?` operator propagates this error automatically.

**Rust concepts introduced:**

- **`Result<T, E>` and the `?` operator** — `?` on a `Result` either unwraps the value (if `Ok`) or returns the error immediately from the current function (if `Err`). This replaces try/catch from other languages.
- **`Vec<char>` indexing** — safe character-by-character access
- **`match` exhaustiveness** — the compiler forces you to handle every arm of the match. You cannot forget a case.

**Writing unit tests:**

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_number_token() {
        let mut lexer = Lexer::new("42");
        let tokens = lexer.tokenize().unwrap();
        assert_eq!(tokens[0].kind, TokenKind::Number(42.0));
    }
}
```

`#[cfg(test)]` means the block is only compiled during `cargo test`. `unwrap()` panics if the result is `Err` — fine for tests.

**Milestone verification:**

```rust
// In main.rs, temporarily:
let src = std::fs::read_to_string("test.chin").unwrap();
let tokens = lexer::Lexer::new(&src).tokenize().unwrap();
for t in &tokens {
    println!("{:?}", t);
}
```

Create `test.chin` containing `dust x = 42;` and confirm you see the token list printed.

---

### Phase 2 – AST

**Goal:** Define the data structures that represent a parsed program in memory. No parsing yet — just the types.

**File:** `src/ast.rs`

The AST has two enums. **`Expr`** represents things that produce a value. **`Stmt`** represents things with a side effect or control flow.

```rust
#[derive(Debug, Clone)]
pub enum BinaryOp { Add, Sub, Mul, Div, Mod, Eq, NotEq, Lt, Le, Gt, Ge, And, Or }

#[derive(Debug, Clone)]
pub enum UnaryOp { Neg, Not }

#[derive(Debug, Clone)]
pub enum Expr {
    Number(f64),
    StringLit(String),
    Bool(bool),
    Null,
    Variable(String),                              // reading a variable
    Assign(String, Box<Expr>),                     // x = expr
    BinaryOp { left: Box<Expr>, op: BinaryOp, right: Box<Expr> },
    UnaryOp  { op: UnaryOp, operand: Box<Expr> },
    Call     { callee: Box<Expr>, args: Vec<Expr> },
    Index    { object: Box<Expr>, index: Box<Expr> },         // arr[i]
    IndexAssign { object: Box<Expr>, index: Box<Expr>, value: Box<Expr> },
    ArrayLit(Vec<Expr>),
    Get { object: Box<Expr>, field: String },      // obj.field
    Set { object: Box<Expr>, field: String, value: Box<Expr> },
}

#[derive(Debug, Clone)]
pub enum Stmt {
    Expression(Expr),
    VarDecl    { name: String, initializer: Expr },
    Block(Vec<Stmt>),
    If         { condition: Expr, then_branch: Box<Stmt>, else_branch: Option<Box<Stmt>> },
    While      { condition: Expr, body: Box<Stmt> },
    For        { var: String, from: Expr, to: Expr, body: Box<Stmt> },
    FunctionDecl { name: String, params: Vec<String>, body: Vec<Stmt> },
    Return(Option<Expr>),
    ClassDecl  { name: String, methods: Vec<Stmt> },
    Print(Expr),
}
```

**Rust concepts introduced:**

- **`Box<T>`** — required for recursive types. `Expr::BinaryOp` contains `Box<Expr>` fields because a type cannot directly contain itself (that would be infinite size). `Box<T>` allocates on the heap and stores a pointer, breaking the cycle. The compiler will tell you if you forget: _"recursive type has infinite size"_.

- **`Option<T>`** — represents "maybe a value, maybe nothing". The `else_branch` is `Option<Box<Stmt>>` because not every `scratch` block has an `otherwise`. This forces you to handle the "no else" case explicitly everywhere — no null pointers.

- **`#[derive(Clone)]`** — you need to clone AST nodes when evaluating loops (the body gets executed multiple times). The derive macro auto-generates a `clone()` method as long as every field type is also `Clone`.

**Milestone verification:**
Construct an AST node manually in a `#[test]` and confirm it compiles:

```rust
let node = Expr::BinaryOp {
    left:  Box::new(Expr::Number(1.0)),
    op:    BinaryOp::Add,
    right: Box::new(Expr::Number(2.0)),
};
```

No runtime output yet — if `cargo build` succeeds, Phase 2 is done.

---

### Phase 3 – Parser

**Goal:** Turn `Vec<Token>` into `Vec<Stmt>`. After this phase you can parse a `.chin` file and print the full AST with `{:#?}`.

**File:** `src/parser.rs`

```rust
pub struct Parser {
    tokens: Vec<Token>,
    pos: usize,
}

impl Parser {
    pub fn new(tokens: Vec<Token>) -> Self { ... }
    pub fn parse(&mut self) -> Result<Vec<Stmt>, ChinchillinError> { ... }
}
```

#### Helper methods (implement these first)

```rust
fn peek(&self) -> &Token              // look at current token without consuming
fn advance(&mut self) -> Token        // consume and return current token
fn check(&self, kind: &TokenKind) -> bool   // is the current token this kind?
fn expect(&mut self, kind: TokenKind) -> Result<Token, ChinchillinError>
    // consume this token or return ParseError
```

**Note on `check` and `TokenKind`:** Some variants carry data (`Number(f64)`), so comparing kinds requires a bit of care. One approach: match on the variant name, not the whole value:

```rust
fn check_kind(&self) -> bool {
    matches!(&self.peek().kind, TokenKind::LeftParen)
}
```

#### The precedence ladder

This is the most important concept in the parser. Each grammar rule is one method. Operators lower in the list bind _less tightly_ than operators higher up — which means they are parsed _first_ (at the top of the call chain) and end up as the outermost node in the tree.

```
parse_expression()          ← entry point
  └─ parse_or()             ← lowest precedence
       └─ parse_and()
            └─ parse_equality()     ==  !=
                 └─ parse_comparison()  <  <=  >  >=
                      └─ parse_term()   +  -
                           └─ parse_factor()  *  /  %
                                └─ parse_unary()   not  -
                                     └─ parse_call()   f() arr[] obj.
                                          └─ parse_primary()  ← highest precedence
                                               (literals, variables, parenthesised exprs)
```

**Why this order?** Consider `1 + 2 * 3`. The parser enters `parse_term`, which calls `parse_factor` for the left side. `parse_factor` calls `parse_unary` → `parse_call` → `parse_primary` which returns `1`. Back in `parse_term`, the next token is `+`, so it calls `parse_factor` again for the right side. `parse_factor` sees `2`, then sees `*`, so it calls `parse_unary` → `parse_primary` which returns `3`, and wraps it: `Mul(2, 3)`. Now `parse_term` wraps: `Add(1, Mul(2, 3))`. Multiplication binds tighter — exactly right.

**Example: `parse_term`**

```rust
fn parse_term(&mut self) -> Result<Expr, ChinchillinError> {
    let mut left = self.parse_factor()?;
    while self.check_plus_or_minus() {
        let op = match self.advance().kind {
            TokenKind::Plus  => BinaryOp::Add,
            TokenKind::Minus => BinaryOp::Sub,
            _ => unreachable!(),
        };
        let right = self.parse_factor()?;
        left = Expr::BinaryOp { left: Box::new(left), op, right: Box::new(right) };
    }
    Ok(left)
}
```

Each level follows this same pattern: parse the left side using the _next higher_ level, loop while the current token is one of this level's operators, parse the right side, wrap in a `BinaryOp`.

#### Parsing `parse_call()`

After `parse_primary()`, loop while the next token is `(`, `[`, or `.`:

```
(  → consume args list → wrap in Expr::Call
[  → consume index expr → wrap in Expr::Index
.  → consume field name → wrap in Expr::Get
```

This handles chains like `obj.method(arg)[0].field` correctly because each iteration wraps the previous result.

#### Parsing statements

```rust
fn parse_statement(&mut self) -> Result<Stmt, ChinchillinError> {
    match self.peek().kind {
        TokenKind::Dust      => self.parse_var_decl(),
        TokenKind::Furry     => self.parse_function_decl(),
        TokenKind::Scratch   => self.parse_if(),
        TokenKind::Nibble    => self.parse_while(),
        TokenKind::Zoom      => self.parse_for(),
        TokenKind::Wheek     => self.parse_return(),
        TokenKind::Fluff     => self.parse_class(),
        TokenKind::LeftBrace => self.parse_block(),
        TokenKind::Bark      => self.parse_print(),
        _                    => self.parse_expression_statement(),
    }
}
```

**Parsing `zoom`** specifically handles `zoom i from 0 to 10 { }`:

```
expect(Zoom)
read identifier name  → loop variable
expect(From)
parse_expression()    → start value
expect(To)
parse_expression()    → end value
parse_block()         → body
→ Stmt::For { var, from, to, body }
```

**Assignment detection** — in `parse_expression_statement`, after parsing a full expression, if the next token is `=`:

- If the expression was `Expr::Variable(name)` → reparse as `Expr::Assign(name, rhs)`
- If the expression was `Expr::Get { object, field }` → reparse as `Expr::Set { object, field, value: rhs }`
- If the expression was `Expr::Index { object, index }` → reparse as `Expr::IndexAssign { object, index, value: rhs }`
- Otherwise → parse error ("invalid assignment target")

**Rust concepts introduced:**

- **Recursive descent maps directly to recursive function calls** — each grammar rule is one method. The call stack is the parse stack. This is the most natural way to write a parser.
- **Returning `Result<T, ChinchillinError>` from every method** — `?` propagates parse errors automatically. You only need to handle errors where you want to produce a useful message.
- **`#[derive(Debug)]` on AST nodes** — lets you print the whole tree with `println!("{:#?}", ast)` for free. Use this to verify parse output.
- **Cloning `String` out of tokens** — when you extract an identifier name from `TokenKind::Identifier(s)`, the `String` is inside the token. Since the token is in your `Vec<Token>`, you usually need to `.clone()` the string to use it elsewhere.

**Milestone verification:**
Parse `dust x = 1 + 2 * 3;` and print the AST. Confirm the top-level `BinaryOp` op is `Add`, and its right side is another `BinaryOp` with op `Mul`. If it's backwards, your precedence ladder is inverted.

---

### Phase 4 – Basic Interpreter

**Goal:** Evaluate variable declarations, arithmetic, and `bark`. First phase with real output.

**Files:** `src/environment.rs`, `src/interpreter.rs`

---

#### `src/environment.rs`

The environment stores variables and chains scopes:

```rust
pub struct Environment {
    values: HashMap<String, Value>,
    parent: Option<Rc<RefCell<Environment>>>,
}

impl Environment {
    pub fn new() -> Self                                     // global scope
    pub fn new_child(parent: Rc<RefCell<Environment>>) -> Self  // nested scope
    pub fn define(&mut self, name: &str, value: Value)          // create variable
    pub fn get(&self, name: &str) -> Option<Value>              // read (walks up chain)
    pub fn assign(&mut self, name: &str, value: Value) -> bool  // write (walks up, returns false if not found)
}
```

**`get` walks the parent chain:**

```rust
pub fn get(&self, name: &str) -> Option<Value> {
    if let Some(v) = self.values.get(name) {
        Some(v.clone())
    } else if let Some(parent) = &self.parent {
        parent.borrow().get(name)
    } else {
        None
    }
}
```

**`assign` also walks up** but only updates an _existing_ binding. If the variable doesn't exist anywhere in the chain, it returns `false` and the interpreter emits a runtime error. This prevents assigning to undeclared variables.

---

#### `src/interpreter.rs` — Value enum

```rust
#[derive(Clone)]
pub enum Value {
    Number(f64),
    Str(String),
    Bool(bool),
    Null,
    Array(Rc<RefCell<Vec<Value>>>),
    Function {
        params:  Vec<String>,
        body:    Vec<Stmt>,
        closure: Rc<RefCell<Environment>>,
    },
    Class(Rc<Class>),
    Instance(Rc<RefCell<Instance>>),
}
```

Implement `std::fmt::Display` for `Value` so `bark` can print anything. Numbers that are whole should print without a decimal: check `n == n.floor()` and format as integer.

---

#### `src/interpreter.rs` — Interpreter struct

```rust
pub struct Interpreter {
    pub globals: Rc<RefCell<Environment>>,
}

impl Interpreter {
    pub fn new() -> Self { ... }
    pub fn execute(&mut self, stmts: Vec<Stmt>) -> Result<(), ChinchillinError> { ... }

    fn exec_stmt(&mut self, env: Rc<RefCell<Environment>>, stmt: Stmt)
        -> Result<Option<Value>, ChinchillinError> { ... }

    fn eval_expr(&mut self, env: Rc<RefCell<Environment>>, expr: Expr)
        -> Result<Value, ChinchillinError> { ... }
}
```

**Why does `exec_stmt` return `Option<Value>`?**
Normally it returns `Ok(None)`. But when a `wheek` (return) statement is executed, it returns `Ok(Some(value))`. That `Some` is threaded back through every caller until it reaches the function call site, which catches it as the function's return value. This avoids needing exceptions or panics.

**Implement in Phase 4:**

- `Stmt::VarDecl` — eval initializer, call `env.borrow_mut().define(name, val)`
- `Stmt::Expression` — eval and discard
- `Stmt::Print` — eval and `println!("{}", val)`
- `Stmt::Block` — create child env, execute each statement
- `Expr::Number / StringLit / Bool / Null` — return literal value
- `Expr::Variable` — look up in env, error if not found
- `Expr::BinaryOp` — eval both sides, apply operator
- `Expr::Assign` — eval rhs, call `env.borrow_mut().assign(name, val)`, error if `false`

---

**The most important Rust concept: `Rc<RefCell<T>>`**

This is how you share mutable data across the interpreter. You will need it for environments.

- **`Rc<T>`** (Reference Counted) — allows _multiple owners_ of the same data. When you create a child scope, both the parent and child need to refer to the same environment object. `Box<T>` only allows one owner; `Rc<T>` allows many. Cloning an `Rc` is cheap (just increments a counter).

- **`RefCell<T>`** — allows _interior mutability_ — you can mutate data through a shared reference by calling `.borrow_mut()` at runtime. The borrow checker is enforced at runtime instead of compile time: if you hold a `.borrow()` and call `.borrow_mut()` on the same `RefCell`, it panics.

- **Together: `Rc<RefCell<Environment>>`** = "a shared, mutable environment". Pass it by cloning the `Rc` (cheap), borrow it when you need to read or write.

**Rule to avoid `RefCell` panics:** Keep borrows short-lived. Call `.borrow()`, do your work, let the borrow drop before calling `.borrow_mut()`. Never store a borrow in a variable that outlives a line.

**Milestone verification:**

```
dust x = 10;
dust y = x + 5;
bark(y);           // should print: 15
bark("Hello!");    // should print: Hello!
```

Also test scope shadowing:

```
dust a = 1;
{
    dust a = 2;
    bark(a);   // 2
}
bark(a);       // 1
```

---

### Phase 5 – Control Flow

**Goal:** Add `scratch`/`otherwise`, `nibble`, and `zoom` to the interpreter.

**File:** `src/interpreter.rs` (add new arms to `exec_stmt`)

**Truthiness rule:** Only `nope` and `bald` are falsy. Everything else (including `0`) is truthy. Implement a helper:

```rust
fn is_truthy(v: &Value) -> bool {
    match v {
        Value::Bool(false) | Value::Null => false,
        _ => true,
    }
}
```

**`Stmt::If`:**

```rust
let cond = self.eval_expr(env.clone(), condition)?;
if is_truthy(&cond) {
    self.exec_stmt(env, *then_branch)?;
} else if let Some(else_b) = else_branch {
    self.exec_stmt(env, *else_b)?;
}
```

**`Stmt::While`:**

```rust
loop {
    let cond = self.eval_expr(env.clone(), condition.clone())?;
    if !is_truthy(&cond) { break; }
    if let Some(ret) = self.exec_stmt(env.clone(), *body.clone())? {
        return Ok(Some(ret));   // early return from enclosing function
    }
}
```

**`Stmt::For` (zoom):**

```rust
let start = /* eval from expr as i64 */;
let end   = /* eval to   expr as i64 */;
for i in start..end {
    let loop_env = Rc::new(RefCell::new(Environment::new_child(env.clone())));
    loop_env.borrow_mut().define(&var, Value::Number(i as f64));
    if let Some(ret) = self.exec_stmt(loop_env, *body.clone())? {
        return Ok(Some(ret));
    }
}
```

`zoom i from 0 to 10` is exclusive of `10` (matching Rust's `0..10`).

**Important:** Every loop body checks `if let Some(ret) = ...` and propagates it. This is what makes `wheek` work inside a loop — the `Some(value)` bubbles all the way back to the function call site.

**Rust concepts introduced:**

- **`loop { ... break; }` vs `while` keyword** — `loop` is Rust's unconditional loop. Using it with a manual `break` is more idiomatic when the exit condition is in the middle of the loop.
- **`.clone()` on AST nodes** — loop conditions and bodies need to be evaluated multiple times, so they get cloned each iteration. This requires `#[derive(Clone)]` on `Expr` and `Stmt` (add those derives to `ast.rs`).
- **`f64 as i64` cast** — `zoom` steps by integers. Cast the bounds to `i64` for the range.

---

### Phase 6 – Functions & Closures

**Goal:** Add `furry`, `wheek`, and function calls. Closures (functions that capture variables from their defining scope) work automatically from the environment design.

**File:** `src/interpreter.rs`

**Declaring a function** (`Stmt::FunctionDecl`):

```rust
let func = Value::Function {
    params,
    body,
    closure: env.clone(),   // capture the environment at definition time
};
env.borrow_mut().define(&name, func);
```

**Calling a function** (`Expr::Call` where callee evaluates to `Value::Function`):

```rust
// Create a new scope whose PARENT is the closure (not the caller's scope)
let call_env = Rc::new(RefCell::new(Environment::new_child(closure)));
for (param, arg) in params.iter().zip(arg_vals.iter()) {
    call_env.borrow_mut().define(param, arg.clone());
}
for stmt in body {
    if let Some(ret) = self.exec_stmt(call_env.clone(), stmt)? {
        return Ok(ret);   // wheek found — return the value
    }
}
Ok(Value::Null)   // implicit return null
```

**Why parent = closure, not caller?**
This is the difference between _lexical scoping_ (what Chinchillin' uses) and _dynamic scoping_. With lexical scoping, a function can see variables from where it was _defined_. With dynamic scoping, it would see variables from where it was _called_. Lexical scoping is almost always what you want — it makes closures predictable.

**Closures work for free.** If `make_counter()` defines a local variable `count` and returns an `inc` function, that `inc` function's `closure` field holds an `Rc` to the environment that contains `count`. The reference count keeps that environment alive even after `make_counter()` returns. When `inc` is later called and reads/writes `count`, it finds it in its closure chain.

**Rust concepts introduced:**

- **`Iterator::zip()`** — pairs two iterators element-by-element: `params.iter().zip(args.iter())` gives `(param_name, arg_value)` pairs.
- **Values as data** — `Value::Function` stores `params`, `body`, and `closure` as regular data. Functions are first-class: you can assign them to variables, pass them as arguments, and return them.

---

### Phase 7 – Arrays

**Goal:** Array literals `[1, 2, 3]`, index reads `arr[0]`, and index writes `arr[1] = 99`.

**File:** `src/interpreter.rs`

Arrays are stored as `Rc<RefCell<Vec<Value>>>`. This means arrays are **reference types** — assigning an array to a new variable shares the same underlying storage, not a copy.

**Array literal** (`Expr::ArrayLit`):

```rust
let vals: Result<Vec<Value>, _> = elements
    .into_iter()
    .map(|e| self.eval_expr(env.clone(), e))
    .collect();
Ok(Value::Array(Rc::new(RefCell::new(vals?))))
```

**Index read** (`Expr::Index`):

```rust
// eval object → must be Value::Array
// eval index  → must be Value::Number
// arr.borrow().get(i).cloned()  →  Ok(value) or bounds error
```

**Index write** (`Expr::IndexAssign`):

```rust
// eval object, eval index, eval new value
// arr.borrow_mut()[i] = new_value
```

**Rust concepts introduced:**

- **`collect::<Result<Vec<_>, _>>()`** — collecting an iterator of `Result<T, E>` into a `Result<Vec<T>, E>` short-circuits on the first error. This is idiomatic Rust for "evaluate a list of things that might fail".
- **Reference semantics** — `dust b = a; b[0] = 99;` also changes `a[0]` because both variables hold `Rc` pointers to the same `Vec`. Document this in comments; it surprises people coming from Python/JavaScript.

---

### Phase 8 – Classes & Objects

**Goal:** `fluff` class declarations, `adopt` instance creation, method calls, `chin` self-reference.

**File:** `src/interpreter.rs`

#### Data structures

```rust
pub struct Class {
    pub name:    String,
    pub methods: HashMap<String, Value>,
}

pub struct Instance {
    pub class:  Rc<Class>,
    pub fields: HashMap<String, Value>,
}
```

Add two new variants to `Value`:

```rust
Value::Class(Rc<Class>)
Value::Instance(Rc<RefCell<Instance>>)
```

#### Class declaration (`Stmt::ClassDecl`)

Loop over the method `Stmt::FunctionDecl` items, build a `HashMap<String, Value::Function>`, then store a `Value::Class` in the environment.

#### Creating an instance (`adopt ClassName(args)`)

When `Expr::Call` evaluates to `Value::Class`:

1. Create a new `Instance` with empty `fields` and a reference to the class.
2. Wrap it: `Rc::new(RefCell::new(instance))`.
3. Look for an `init` method. If found, call it with `chin` bound to the new instance.
4. Return `Value::Instance(...)`.

#### Method calls and `chin` binding

`pet.speak()` is parsed as a `Call` where the callee is `Get { object: pet, field: "speak" }`.

When `Expr::Get` evaluates on an `Instance`:

- Check `instance.fields` first (fields shadow methods)
- Then check `class.methods`
- When returning a method, **bind `chin`**: create a child environment of the method's closure and define `"chin"` as the current instance. Return this as the function's closure.

```rust
// Binding chin when returning a method
let bound_env = Rc::new(RefCell::new(Environment::new_child(method_closure)));
bound_env.borrow_mut().define("chin", Value::Instance(inst_rc.clone()));
Value::Function { params, body, closure: bound_env }
```

This is called a **bound method**. Without it, `chin.name` inside the method would fail with "undefined variable 'chin'".

**`Expr::Set`** (writing `chin.name = value`):

```rust
// eval object → Value::Instance
// inst_rc.borrow_mut().fields.insert(field, val)
```

**Lookup order: fields first, then methods.** A field can shadow a method — this is correct for a dynamic language.

**No inheritance** — keep the class system minimal for now. The plan does not include `extends` or superclass lookup.

**Rust concepts introduced:**

- **`HashMap<String, Value>`** for dynamic field storage
- **Nested `Rc<RefCell<>>`** — `Value::Instance(Rc<RefCell<Instance>>)` follows the same pattern as environments
- **`Rc<Class>` without `RefCell`** — classes are immutable after creation (methods don't change), so no `RefCell` needed here

---

### Phase 9 – REPL & Polish

**Goal:** A usable command-line tool. Run files with `cargo run myfile.chin` or start an interactive REPL with `cargo run`.

**File:** `src/main.rs`

```rust
fn main() {
    let args: Vec<String> = std::env::args().collect();
    match args.len() {
        1 => run_repl(),
        2 => run_file(&args[1]),
        _ => {
            eprintln!("Usage: chinchillin [file.chin]");
            std::process::exit(1);
        }
    }
}
```

**`run_file`:** `std::fs::read_to_string(path)` → lex → parse → interpret. On any error, print to stderr with `eprintln!` and `process::exit(1)`.

**`run_repl`:** Loop reading lines from `stdin`. Print `chin> ` prompt before each line. On error, print it but keep the loop running (don't exit).

**Rust concepts introduced:**

- **`std::env::args()`** — command-line arguments as an iterator
- **`std::fs::read_to_string`** — reads a file to a `String`, returns `Result<String, io::Error>`
- **`eprintln!`** — prints to stderr (correct channel for errors; stdout is for program output)
- **`std::process::exit(code)`** — terminate with a specific exit code

---

## Known Limitations

- **No inheritance** — classes do not support `extends` or superclass method lookup
- **Deep recursion will stack overflow** — Rust's default stack is limited; deeply recursive `.chin` programs will crash
- **`zoom` uses integer steps only** — range bounds are cast to `i64`; fractional values are truncated
- **No standard library** — `bark` is the only built-in. No file I/O, no math functions, etc.
- **Single-threaded only** — `Rc<RefCell<>>` is not thread-safe by design

---

## End-to-End Test

Once all phases are complete, this program should run correctly:

```
// fizzbuzz.chin
furry fizzbuzz(n) {
    zoom i from 1 to n {
        scratch i % 15 == 0 {
            bark("FizzBuzz");
        } otherwise {
            scratch i % 3 == 0 {
                bark("Fizz");
            } otherwise {
                scratch i % 5 == 0 {
                    bark("Buzz");
                } otherwise {
                    bark(i);
                }
            }
        }
    }
}

fizzbuzz(16);
```

Run with `cargo run fizzbuzz.chin` and verify the output matches the expected FizzBuzz sequence.
