# Dream Language Summary

The following concerns are listed in priority order, from most important to least important.

## 1. Concurrency model is the top priority

Concurrency is the primary design concern. It should shape the architecture of the system, not merely be a runtime facility layered on top.

The concern is not about execution primitives alone, such as goroutines, coroutines, green threads, fibers, or thread pools. Those only describe how work runs.

The deeper concern is how concurrent entities coordinate: how they communicate, how ownership is expressed, how state change is controlled, and how systems avoid collapsing back into shared-memory mutation.

Preferred models, in order, are **CSP, ConcurrentML, Actor model, then other common models**.

**Why this order:**
- **CSP** is preferred because it gives the leanest coordination model: independent concurrent entities communicate through explicit channels and protocol flow, with minimal conceptual overhead.
- **ConcurrentML** comes next because it preserves strong message-passing semantics while adding richer event composition.
- **Actor model** is still valuable, but often introduces heavier identity and mailbox semantics than necessary for many systems.
- Other models are less preferred because they tend to be more implicit, more operationally noisy, or more reliant on shared mutable coordination.

The main anti-pattern in concurrent and parallel design is **shared mutable memory**. It hides coordination behind mutation, moves complexity into locks, timing, and aliasing, makes correctness harder to reason about, couples execution and state too tightly, and scales poorly in conceptual clarity even when it works operationally.

The ideal language should therefore encourage **explicit coordination** rather than shared-state synchronization.

A preferred concurrent state shape follows naturally from this: mutations flow through channels or message streams, one logical mutation path applies them in order, and readers should not need to route through that same path when persistent snapshots make that unnecessary.

---

## 2. Persistent data structures are a major requirement

The ideal language should provide **persistent immutable data structures** as a core capability, especially maps and sets.

The essential properties are **structural sharing, persistent snapshots, safe concurrent reads, and value-oriented mutation**. The key point is not merely that snapshots are cheap, but that persistence is **not** full cloning and **not** ordinary copy-on-write. New versions should be created without rebuilding the whole structure, while previous versions remain valid and shareable.

This is foundational, not a minor convenience. It is one of the main enablers of clean concurrent design, because it allows state to evolve without forcing readers and writers into direct contention.

**Clojure's persistent hash-map / P-HAMT is the clearest reference point here and the best practical example of this style.**

---

## 3. Functional composition is preferred over OOP-style structure

Currying and functional composition are strongly preferred over inheritance-heavy or composition-heavy object-oriented design.

The desired qualities are **lean abstraction, composable transformations, scalable code through function combination, and dataflow-oriented design**. A key reference point here is **Clojure's transducers**, as an example of highly composable, reusable transformation logic decoupled from concrete containers and execution context.

OOP can express composition, but usually with more structural overhead: object identity becomes entangled with behavior, abstraction accumulates through wrappers, interfaces, and lifecycle coupling, and reuse becomes more indirect and ceremonious.

Currying and functional composition scale more cleanly because behavior can be built incrementally from smaller functions, abstractions stay close to data transformation, and composition remains lean instead of being mediated through object structure.

An especially important advantage is that FP and currying naturally favor **segregation of state/data from behaviors**. That separation makes systems easier to scale, easier to control, and easier to understand, because behavior can be composed independently of the concrete shape or ownership of state.

The preference is therefore not anti-abstraction, but a preference for **function-first abstraction** over object-first structure.

---

## 4. Correctness should come from specifications

Correctness matters, but the ideal route is not primarily through larger piles of unit tests.

The preferred model is **specification-driven correctness**, including declarative specs, invariant checking, schema-like validation, and generative or property-based testing derived from specs.

What is especially attractive in the Clojure model is that the spec is **inline with the execution code, part of the program structure itself, usable at runtime rather than only in tests, and able to support conformance during execution rather than merely offline validation**.

This matters because correctness then becomes part of the running system and its interfaces, not only something checked afterward by a separate testing layer.

**Clojure's `spec` is the reference model here.**

---

## 5. Secondary concerns

The following matter, but clearly less than the priorities above: pattern matching, strongly typed collections and generics, error handling style including monadic or composable forms, algebraic data types, static typing, dynamic typing, and general type-system richness.

Both of the following are acceptable: **strongly typed while dynamic**, or **strongly statically typed during development, but with inference strong enough that typing does not become practical friction**.

In other words, typing is welcome if it improves correctness in practice, but not if it adds ceremony without proportional value.

What matters more is whether the language supports **strong concurrency coordination, persistent immutable data structures, lean functional composition, and spec-driven correctness**.

---

## Closing note

For all of the reasons above, **Clojure is the closest existing candidate** to this dream language.

It comes closest because it combines **excellent concurrency semantics, persistent immutable data structures, function-first composition, transducer-style dataflow, and spec-oriented correctness thinking** in a way that already feels coherent rather than pieced together.

It also has a strong practical advantage: **Java interoperability**, which provides a very strong deployment and integration model by giving access to the JVM ecosystem without giving up Clojure's core language qualities.
