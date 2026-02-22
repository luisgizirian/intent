# Intent Language Examples

> *"LLMs make it easier to write code, but harder to keep systems correct — and languages exist to solve the latter."*

---

## What is Intent?

**Intent** is a programming language designed for the LLM era. As large language models become routine coding partners, the challenge shifts: it's no longer hard to *write* code, it's hard to keep systems *correct* as code is generated, modified, and regenerated at scale.

Intent solves this by making meaning machine-verifiable. Every function carries explicit **contracts** (preconditions and postconditions), every data structure enforces **invariants**, every side effect is declared through an **effect system**, and every sensitive operation is gated by a **capability**. When LLMs — or human developers — generate code inside an Intent codebase, the language itself validates that the generated code respects the system's intent.

### Why Intent exists

| Problem | What Intent provides |
|---|---|
| LLMs generate plausible but incorrect code | Contracts catch violations at compile time or runtime |
| Side effects go untracked across refactors | Effect system makes every side effect explicit |
| Security invariants are easy to break silently | Capability system limits what each function can do |
| "What does this code mean?" is hard to answer | Intent blocks express high-level goals in verifiable form |
| Null / error handling is inconsistent | `Result<T, E>` and `Option<T>` are first-class |

### Key qualities

- **Semantic clarity** — code communicates its *purpose*, not just its *mechanism*
- **Machine-checkable** — invariants, contracts and intents are verified statically or at runtime
- **LLM-friendly** — rails that guide code generation toward correctness by design
- **Effect-safe** — side effects must be declared; pure functions are provably pure
- **Capability-secure** — functions get only the permissions they explicitly request
- **Expressive** — algebraic data types, generics, pattern matching, and higher-order functions

---

## Examples: A Guided Exploration

Each example illuminates a specific set of concepts in Intent. They're designed to build on each other, so following the suggested path helps concepts compound naturally — but feel free to explore based on curiosity or need.

**No hierarchies. No levels. Just ideas that build on each other.**

> **A note on learning:** These examples aren't a ladder to climb. There's no "beginner" or "advanced" — only different concepts that become clearer when you've seen what came before. Our competitive instincts want to rush to the "hard stuff," but understanding comes from spending time with each idea. Take your time. Master the fundamentals. They're not stepping stones to skip over; they're the foundation everything else rests on.

---

### 1. Effect System Basics — [`hello.intent`](hello.intent)

**Focus:** How Intent tracks side effects

The simplest possible Intent program. It demonstrates:

- Declaring an **effect** (`IO`) to represent side-effectful operations
- Writing a `main` function that uses the effect
- The `@effect[...]` annotation that makes I/O explicit

```intent
effect IO {
    fn write(s: String) -> Void
    fn read() -> String
}

fn main() -> Void @effect[IO] {
    IO.write("Hello, Intent!")
    IO.write("Welcome to the LLM-era programming language.")
}
```

**Take-away:** In Intent, every function that performs I/O must declare it. A function without an `@effect` annotation is guaranteed to have no side effects — the compiler enforces this.

---

### 2. Contracts & Invariants — [`contracts.intent`](contracts.intent)

**Focus:** Preconditions, postconditions, and type-level guarantees

Introduces the contract system at the heart of Intent — how to make promises that the compiler can verify.

Highlights:

- `@requires` — a precondition that callers must satisfy
- `@ensures` — a postcondition the implementation must guarantee
- `@invariant` on a struct field — a property that must hold for every value of the type
- `pure fn` — a function that is guaranteed to have no side effects
- `Result<T, E>` — explicit, typed error handling

```intent
fn divide(a: Int, b: Int) -> Int
    @requires b != 0
    @ensures result * b == a
{
    return a / b
}

struct Temperature {
    celsius: Float64,
    @invariant celsius >= -273.15
}
```

**Take-away:** Contracts let you express *what* a function promises without hiding that meaning inside the implementation. The compiler (or a verifier) can check that callers respect preconditions and that implementations meet postconditions.

---

### 3. Algebraic Types & Pattern Matching — [`patterns.intent`](patterns.intent)

**Focus:** Modeling domains with precision and exhaustiveness

Explores algebraic data types (enums), pattern matching, and struct destructuring — tools that make illegal states unrepresentable.

Highlights:

- Enum variants with associated data (`Circle(Float64)`, `Rectangle(Float64, Float64)`)
- `match` with **guards** (`if r > 100.0`)
- Nested pattern matching (`Result<Option<T>, E>`)
- Struct destructuring in match arms
- `pure fn` with a postcondition

```intent
enum Shape {
    Circle(Float64),
    Rectangle(Float64, Float64),
    Triangle(Float64, Float64, Float64),
    Point,
}

pure fn area(shape: Shape) -> Float64
    @ensures result >= 0.0
{
    match shape {
        Shape::Circle(radius) => 3.14159 * radius * radius,
        Shape::Rectangle(w, h) => w * h,
        ...
    }
}
```

**Take-away:** Enums and pattern matching let you model domain concepts precisely and handle every case explicitly. Guards make branching logic readable while keeping it verifiable.

---

### 4. Composing Concepts: A Banking System — [`banking.intent`](banking.intent)

**Focus:** How multiple Intent features work together in realistic code

A real-world scenario that demonstrates how effects, invariants, named contracts, and intent blocks combine to create correct-by-construction systems.

Highlights:

- `struct Account` with `@invariant balance >= 0.0` — balances can never go negative
- A **named contract** `SafeTransfer` that specifies pre/postconditions for transfers
- An **intent block** `MoneyConservation` — a high-level property expressed independently of the implementation
- `old(...)` — refers to the value a variable held *before* the function ran
- Multiple effects (`Database`, `Logging`) declared together
- `Result`-based error propagation with `?`

```intent
contract SafeTransfer {
    @requires amount > 0.0
    @requires from.balance >= amount
    @ensures from.balance == old(from.balance) - amount
    @ensures to.balance == old(to.balance) + amount
}

intent MoneyConservation {
    @ensures from.balance + to.balance == old(from.balance) + old(to.balance)
}

fn transfer(from: mut Account, to: mut Account, amount: Float64)
    -> Result<Void, TransferError>
    @contract SafeTransfer
    @intent MoneyConservation
    @effect[Database, Logging]
{ ... }
```

**Take-away:** Named contracts and intent blocks separate *what* a function must do from *how* it does it. This separation is invaluable in LLM-assisted development: you specify the intent once and let the model fill in the implementation — confident that any deviation will be caught.

---

### 5. Verification in Practice — [`sorting.intent`](sorting.intent)

**Focus:** Specifying and verifying algorithmic properties

Demonstrates formal specifications using intent blocks, loop invariants, and verification properties for classic algorithms.

Highlights:

- An `intent Sorted<T: Ord>` block with `forall` quantifiers — a machine-checkable specification of what "sorted" means
- Loop invariants via `@invariant` inside `while` loops
- `@requires` that `binarySearch` only accepts a pre-sorted array
- `@ensures` that uses a `match` expression as a postcondition
- Partition function with detailed contracts on array ranges

```intent
intent Sorted<T: Ord> {
    @ensures forall i in 0..result.len() - 1: result[i] <= result[i + 1]
    @ensures result.len() == input.len()
    @ensures forall x in input: result.contains(x)
}

pure fn bubbleSort<T: Ord>(input: [T]) -> [T]
    @intent Sorted<T>
{ ... }
```

**Take-away:** Intent blocks elevate verification from "the code does this" to "the code satisfies this mathematical property." Any conforming implementation — written by a human or generated by an LLM — can be validated against the specification.

---

### 6. Capability-Based Security — [`capabilities.intent`](capabilities.intent)

**Focus:** Fine-grained permission control at the language level

Shows how Intent's capability system makes security explicit and auditable — functions declare exactly what they can access, and the compiler enforces it.

Highlights:

- Defining capabilities (`FileSystem`, `Network`, `SystemInfo`) with structured permission fields
- `@capability` annotations that limit what a function is allowed to do
- Sandboxed functions with *no* capabilities — provably limited to pure computation
- Combining capabilities with effects: capabilities gate *what*, effects track *that*
- Principle of least privilege enforced at the language level

```intent
capability FileSystem {
    read: Bool,
    write: Bool,
    execute: Bool,
}

fn loadConfig(path: String) -> Result<Config, Error>
    @capability FileSystem { read: true, write: false, execute: false }
    @effect[FileIO]
{ ... }

fn processSandboxed(data: String) -> String
    @capability FileSystem { read: false, write: false, execute: false }
    @capability Network { connect: false, listen: false, maxConnections: 0 }
{
    // Provably isolated — no I/O possible
    return data.toUpperCase()
}
```

**Take-away:** Capabilities give every function a declaration of trust. Security audits, LLM-generated code reviews, and automated policy enforcement all become tractable because permissions are in the source, not inferred from behaviour.

---

## Suggested Exploration Path

Each example builds on concepts from earlier ones, so this order helps ideas compound naturally:

```
hello.intent          ← Effect system foundation
     │
     ▼
contracts.intent      ← Adding correctness guarantees  
     │
     ▼
patterns.intent       ← Precise domain modeling
     │
     ▼
banking.intent        ← Seeing how concepts compose
     │
     ▼
sorting.intent        ← Formal verification properties
     │
     ▼
capabilities.intent   ← Security through declarations
```

**But remember:** These aren't levels to climb. If you're curious about capabilities or verification, jump there directly. You can always circle back when you encounter an unfamiliar concept.

---

## Running the Examples

```bash
# Build the compiler first
npm run build

# Run an example
npm run example

# Or run a specific file directly
node dist/cli.js run examples/hello.intent
node dist/cli.js run examples/contracts.intent
node dist/cli.js run examples/banking.intent
```

See the [project README](../README.md) for full CLI reference and installation instructions.
