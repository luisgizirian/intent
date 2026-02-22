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

## Examples (Beginner → Advanced)

The examples below are ordered from simplest to most sophisticated. Start from the top if you are new to Intent; jump ahead if you already have some familiarity.

---

### 1. Hello World — [`hello.intent`](hello.intent)

**Level:** Beginner

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

### 2. Contracts — [`contracts.intent`](contracts.intent)

**Level:** Beginner–Intermediate

Introduces **preconditions**, **postconditions**, and **struct invariants** — the contract system at the heart of Intent.

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

### 3. Pattern Matching — [`patterns.intent`](patterns.intent)

**Level:** Intermediate

Explores **algebraic data types** (enums), **pattern matching**, and **struct destructuring** — tools for modelling complex domains clearly and exhaustively.

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

### 4. Banking System — [`banking.intent`](banking.intent)

**Level:** Intermediate–Advanced

A real-world scenario that combines **multiple effects**, **struct invariants**, **named contracts**, and **intent blocks** to model a correct-by-construction banking transfer.

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

### 5. Sorting & Verification — [`sorting.intent`](sorting.intent)

**Level:** Advanced

Demonstrates **intent blocks**, **loop invariants**, and **formal verification** properties for classic algorithms: bubble sort, binary search, and quicksort.

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

**Level:** Advanced

Shows how Intent's **capability system** enables fine-grained, declaration-first security. Functions explicitly request only the permissions they need — and the system enforces it.

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

## Learning Path

```
hello.intent          ← Start here (effects)
     │
     ▼
contracts.intent      ← Contracts, invariants, pure functions
     │
     ▼
patterns.intent       ← Algebraic types, pattern matching
     │
     ▼
banking.intent        ← Named contracts, intent blocks, effects
     │
     ▼
sorting.intent        ← Verification, forall, loop invariants
     │
     ▼
capabilities.intent   ← Capability-based security
```

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
