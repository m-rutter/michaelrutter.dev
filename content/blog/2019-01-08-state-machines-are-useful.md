+++
title = "State machines are useful"

draft = false

[taxonomies]
categories = ["programming", "typescript", "finite state machine", "rust"]
+++

I can overthink things, which as programmer can be a virtue and a torture; often
the latter. When I'm implementing something that persists state and involves
user input, I/O or fallible operations, mutations without a predictale order, or
god forbid all of the above then its easy for all of the potential edgecases to
overwhelm and induce panic. Why did I become a web developer?

Over time I've learned, but sometimes forget, that most problems I encounter of
this sort above can be be sketched out visually and modeled as a [finite state
machine][fsm-wiki]. Also, fortunately these days I'm writing in languages like
TypeScript and Rust, which have type systems which can make implementing and
using these things much less error prone.

<!-- more -->

```rust
enum State{
    Initial,
    A(String),
    B(usize),
    C(String, usize)
    Final(f64, String, usize)
}

enum Msg {
    Reset,
    SetA(String)
    SetB(usize)
}
```

```ts
type State =
    | { state: "Initial" }
    | { state: "A"; string: "string" }
    | { state: "B"; number: number }
    | { state: "C"; string: string; number: number }
    | { state: "Final"; float: number; string: string; number: number };
```

[fsm-wiki]: https://en.wikipedia.org/wiki/Finite-state_machine
