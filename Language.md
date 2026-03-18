---
layout: page
title: "💡 Ideal Language"
---

The following concerns are listed in priority order, from most important to least important.

## 1. Concurrency: the primary design concern

Concurrency is the primary design concern. It shapes the architecture of the system, rather than merely being a runtime facility layered on top.

The concern is not about execution primitives alone, such as goroutines, coroutines, green threads, fibers, or thread pools. Those only describe how work runs.

The deeper concern is how concurrent entities coordinate: how they communicate, how ownership is expressed, how state change is controlled, and how systems avoid collapsing back into shared-memory mutation.

Preferred models, in order, are **CSP, ConcurrentML, Actor model, then other common models**.

**Why this order:**
- **CSP** is preferred because it gives the leanest coordination model: independent concurrent entities communicate through explicit channels and protocol flow, with minimal conceptual overhead.
- **ConcurrentML** comes next because it preserves strong message-passing semantics while adding richer event composition.
- **Actor model** is still valuable, but often introduces heavier identity and mailbox semantics than necessary for many systems.
- Other models are less preferred because they tend to be more implicit, more operationally noisy, or more reliant on shared mutable coordination.

The main anti-pattern in concurrent and parallel design is **shared mutable memory**. It hides coordination behind mutation, moves complexity into locks, timing, and aliasing (multiple references pointing to the same mutable state), makes correctness harder to reason about, couples execution and state too tightly, and scales poorly in conceptual clarity even when it works operationally.

The ideal language should therefore encourage **explicit coordination** rather than shared-state synchronization.

A preferred concurrent state shape follows naturally from this: mutations flow through channels or message streams, one logical mutation path applies them in order, and readers should not need to route through that same path when persistent snapshots make that unnecessary.

---

## 2. Persistent immutable data structures

The ideal language should provide **persistent immutable data structures** as a core capability, especially maps and sets.

The essential properties are **structural sharing, persistent snapshots, safe concurrent reads, and value-oriented mutation**. The key point is not merely that snapshots are cheap, but that persistence is **not** full cloning and **not** ordinary copy-on-write. New versions should be created without rebuilding the whole structure, while previous versions remain valid and shareable.

This is foundational, not a minor convenience. It is one of the main enablers of clean concurrent design, because it allows state to evolve without forcing readers and writers into direct contention.

**Clojure's persistent hash-map `P-HAMT` is the clearest reference point here and the best practical example of this style.**

---

## 3. Function-first composition and dataflow

Functional composition and currying are strongly preferred over inheritance-heavy or composition-heavy object-oriented design. The core ideal is that behavior is assembled by combining small, stateless functions — not by structuring objects with embedded state and lifecycle.

The clearest expression of this is **Clojure's transducers** — composable transformation pipelines that are fully decoupled from their input source and output target, so the same transformation logic runs unchanged across sequences, channels, or any reducible context.

```clojure
;; A transducer is just a function — it holds no state and knows nothing
;; about where data comes from or where it goes.
;; With comp, transformations apply left-to-right: filter runs first, then map.
(def xf
  (comp
    (filter odd?)
    (map #(* % %))))

;; The same xf works unchanged across completely different execution contexts:
(transduce xf + [1 2 3 4 5])          ;; => 35  (reduce over a vector)
(into [] xf [1 2 3 4 5])              ;; => [1 9 25]  (build a vector)
(sequence xf [1 2 3 4 5])             ;; => (1 9 25)  (lazy sequence)
(chan 10 xf)                          ;; core.async channel — same xf, async pipeline
```

OOP can express composition, but usually with more structural overhead: object identity becomes entangled with behavior, abstraction accumulates through wrappers, interfaces, and lifecycle coupling, and reuse becomes more indirect and ceremonious.

Currying and functional composition scale more cleanly because behavior can be built incrementally from smaller functions, abstractions stay close to data transformation, and composition remains lean instead of being mediated through object structure.

An especially important advantage is that FP and currying naturally favor **segregation of state/data from behaviors**. That separation makes systems easier to scale, easier to control, and easier to understand, because behavior can be composed independently of the concrete shape or ownership of state.

The preference is therefore not anti-abstraction, but a preference for **function-first abstraction** over object-first structure.

---

## 4. Specification-driven correctness

Correctness matters, but the ideal route is not primarily through larger piles of unit tests.

The preferred model is **specification-driven correctness**, including declarative specs, invariant checking, schema-like validation, and generative or property-based testing derived from specs.

What is especially attractive in the Clojure `spec` is that it is **inline with the execution code, being part of the program flow itself, functioning at runtime rather than only in tests, and able to support conformance of inputs and outputs during execution rather than merely offline validation**.

This matters because correctness then becomes the contract of the running system and its interfaces, not only something checked on the side by a separate testing layer. CI/CD pipelines do not warrant the correctness of the production system.

---

## 5. Secondary concerns

The following matter, but clearly less than the priorities above: pattern matching, strongly typed collections and generics, error handling style including monadic or composable forms, algebraic data types, polymorphism for code sharing, static or dynamic typing, and strength of type enforcement.

Specifically for dynamic vs static typing, both of the following are acceptable: **strongly typed while dynamic**, or **strongly statically typed during development, with inference strong enough that typing does not become practical friction**. In other words, typing is welcome if it improves correctness in practice, but not if it adds ceremony without proportional value.

---

## Closing note

For all of the reasons above, **Clojure is the closest existing candidate** to be the ideal language.

It comes closest because it combines **excellent concurrency semantics including CSP, native persistent immutable data structures, function-first composition rooted from Lisp, transducer-style dataflow, and spec-oriented correctness modeling** in a way that already feels coherent rather than glued together forcefully.

It also has a strong practical advantage: being a JVM language with **full Java interoperability**, which provides a very strong deployment and integration model to one of the largest ecosystems without giving up Clojure's core language qualities.
