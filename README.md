> **Disclaimer:** The project is based on a sample lesson I asked GH Copilot to put together on writing a compiler for a simple language now in the LLM era. It's not intended to be a usable or even complete project. It's more on the realms of becoming a learning journey throughout the intricacies of compiler programming and building. Many unknowns yet.

# Intent Programming Language

**Intent** is a programming language designed for the LLM era, focusing on **machine-verifiable intent**, **semantic clarity**, and **long-lived system preservation**.

> *"LLMs make it easier to write code, but harder to keep systems correct — and languages exist to solve the latter."*

## Why Intent?

In an era where LLMs generate massive amounts of code:
- **Without rails, LLMs hallucinate**
- **With rails, they amplify productivity**

Intent provides those rails through:
- 🔒 **Contracts** - Preconditions, postconditions, and invariants
- ⚡ **Effects** - Explicit side-effect tracking
- 🛡️ **Capabilities** - Permission-based security
- 🎯 **Intent Blocks** - High-level goal specification
- ✅ **Machine-Checkable** - Verify meaning, not just syntax

## Quick Start

### Installation

```bash
# Clone the repository
git clone https://github.com/luisgizirian/compiler.git
cd compiler

# Install dependencies
npm install

# Build the compiler
npm run build

# Run an example
npm run example
```

### Hello World

Create a file `hello.intent`:

```intent
effect IO {
    fn write(s: String) -> Void
}

fn main() -> Void @effect[IO] {
    IO.write("Hello, Intent!")
}
```

Compile and run:

```bash
intent run hello.intent
```

## Language Features

### Functions with Contracts

Contracts specify what must be true before and after function execution:

```intent
fn divide(a: Int, b: Int) -> Int
    @requires b != 0              // Precondition: b must not be zero
    @ensures result * b == a      // Postcondition: result is correct
{
    return a / b
}
```

### Structs with Invariants

Invariants ensure data consistency:

```intent
struct BankAccount {
    balance: Float64,
    
    @invariant balance >= 0.0     // Balance can never be negative
}
```

### Effect System

Effects make side effects explicit:

```intent
effect IO {
    fn read() -> String
    fn write(s: String) -> Void
}

// Pure function - no side effects
pure fn add(a: Int, b: Int) -> Int {
    return a + b
}

// Effectful function - must declare effects
fn greet(name: String) -> Void @effect[IO] {
    IO.write("Hello, " + name)
}
```

### Capability System

Capabilities restrict what code can do:

```intent
capability FileSystem {
    read: Bool,
    write: Bool,
}

fn loadConfig() -> Config
    @capability FileSystem { read: true, write: false }
{
    // Can only read files, not write
}
```

### Intent Blocks

Intent blocks express high-level goals:

```intent
intent Sorted<T: Ord> {
    @ensures forall i < result.len() - 1: result[i] <= result[i + 1]
    @ensures result.len() == input.len()
}

fn sort<T: Ord>(input: [T]) -> [T]
    @intent Sorted<T>
{
    // Any correct implementation works
}
```

### Pattern Matching

Powerful pattern matching with guards:

```intent
enum Shape {
    Circle(Float64),
    Rectangle(Float64, Float64),
}

fn area(shape: Shape) -> Float64 {
    match shape {
        Shape::Circle(r) => 3.14159 * r * r,
        Shape::Rectangle(w, h) => w * h,
    }
}
```

### Result Types for Error Handling

Explicit error handling with Result types:

```intent
fn parseNumber(s: String) -> Result<Int, ParseError> {
    // ...
}

fn main() -> Result<Void, Error> {
    let num = parseNumber("42")?  // Propagate error with ?
    Ok(())
}
```

## CLI Usage

```bash
# Compile to JavaScript
intent compile src/main.intent -o dist/main.js

# Type-check without compiling
intent check src/main.intent

# Compile and run immediately
intent run examples/hello.intent

# Start interactive REPL
intent repl

# Show help
intent --help
```

### Options

| Option | Description |
|--------|-------------|
| `--target, -t` | Output target: `javascript`, `typescript` |
| `--output, -o` | Output file path |
| `--module, -m` | Module system: `esm`, `commonjs` |
| `--no-contracts` | Disable runtime contract checking |
| `--verify` | Verification level: `full`, `runtime`, `trusted` |
| `--watch, -w` | Watch mode - recompile on changes |

## Type System

### Primitive Types

```intent
Int, Int8, Int16, Int32, Int64    // Signed integers
UInt                               // Unsigned integer
Float32, Float64                   // Floating point
Bool                               // Boolean
Char                               // Unicode character
String                             // UTF-8 string
Void                               // No value
Never                              // Never returns
```

### Composite Types

```intent
[Int]                  // Dynamic array
[Int; 10]              // Fixed-size array
(Int, String)          // Tuple
Int?                   // Optional (may be nil)
Result<T, E>           // Success or error
fn(Int) -> Int         // Function type
&T                     // Immutable reference
&mut T                 // Mutable reference
```

## Project Structure

```
compiler/
├── src/
│   ├── index.ts        # Main compiler API
│   ├── cli.ts          # Command-line interface
│   ├── lexer/          # Tokenization
│   ├── parser/         # AST generation
│   ├── analyzer/       # Type checking & verification
│   └── codegen/        # JavaScript code generation
├── examples/           # Example programs
│   ├── hello.intent
│   ├── banking.intent
│   ├── sorting.intent
│   ├── capabilities.intent
│   └── patterns.intent
└── docs/
    └── LANGUAGE_SPEC.md
```

## Examples

See the `examples/` directory for complete examples:

- **hello.intent** - Basic hello world with effects
- **banking.intent** - Contracts, invariants, and intent blocks
- **sorting.intent** - Pure functions and verification
- **capabilities.intent** - Capability-based security
- **patterns.intent** - Pattern matching and enums

## Comparison with Other Languages

| Feature | Intent | Rust | TypeScript | Haskell |
|---------|--------|------|------------|---------|
| Contracts | ✅ First-class | ❌ | ❌ | ⚠️ Limited |
| Effect System | ✅ Built-in | ❌ | ❌ | ✅ Monads |
| Capabilities | ✅ Built-in | ❌ | ❌ | ❌ |
| Intent Blocks | ✅ Unique | ❌ | ❌ | ❌ |
| Memory Safety | ✅ | ✅ | ⚠️ GC | ✅ GC |
| Null Safety | ✅ | ✅ | ⚠️ Optional | ✅ |
| LLM-Friendly | ✅ Designed for | ⚠️ | ⚠️ | ⚠️ |

## Design Philosophy

### 1. Intent Over Implementation
Express *what* you want with verifiable *constraints*, not just how to do it.

### 2. Contracts First
All functions have explicit contracts - preconditions, postconditions, and invariants.

### 3. Effect Tracking
Side effects are explicit and tracked by the type system.

### 4. Semantic Preservation
Code meaning is preserved across refactoring and generation.

### 5. Machine-Checkable
All invariants can be verified statically or at runtime.

## Roadmap

- [x] Lexer and tokenization
- [x] Parser and AST
- [x] Type system and semantic analysis
- [x] JavaScript code generation
- [x] Runtime contract checking
- [x] CLI and REPL
- [x] **TypeScript-Go (tsgo) testing** - 3.8x faster builds verified
- [ ] TypeScript output
- [ ] LSP support (IDE integration)
- [ ] Static verification (SMT solver integration)
- [ ] Package manager
- [ ] Standard library

## Development

### Fast Builds with TypeScript-Go (Optional)

The Intent compiler can be built with Microsoft's new TypeScript-Go (tsgo) compiler for significantly faster build times:

```bash
# Install the TypeScript-Go preview
npm install -D @typescript/native-preview

# Build with tsgo (3.8x faster)
npx tsgo --project tsconfig.json
```

**Performance:** tsgo builds the Intent compiler in ~0.4s vs ~1.5s with standard tsc.

See [docs/TSGO_TESTING_GUIDE.md](docs/TSGO_TESTING_GUIDE.md) for complete testing guide and benchmarks.

## Contributing

Contributions are welcome! Please read the language specification in `docs/LANGUAGE_SPEC.md` before contributing.

## License

MIT License - see [LICENSE](LICENSE) for details.
