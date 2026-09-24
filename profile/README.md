# leocaolab

**Rust infrastructure for AI-native software and for multi-core Python — and the benchmarks to prove it.**

Two threads run through the lab. **AI infrastructure**: a runtime for agents,
a way for them to act on the web, and a reviewer that keeps their code honest.
**Multi-interpreter Python**: using every CPU core from a single Python
process, and rebuilding the pieces that stop working when you do. Each one is
measured, never just claimed. *Don't say it's fast. Prove it.*

## 1 · AI infrastructure

Agents need three things to be production-grade: a runtime to run on, a way to
act on the world, and a way to keep their code honest.

### 🤖 [TARS](https://github.com/leocaolab/tars) · Rust · Apache-2.0
A Rust-first multi-agent LLM runtime. A dozen providers behind one trait, a composable
middleware pipeline, an Agent abstraction you hand tasks to, and Python + Node
bindings — with observability built in, not bolted on.
- Typed error hierarchy (`Permanent` / `Retryable` / `RateLimited` / `Auth`).
- Multi-tenancy enforced at every layer; cache hit/miss observable per call.
- Same Pipeline runs identically local (in-mem) and in a service (Redis + S3).

### 🏄 [SiliconSurfer](https://github.com/leocaolab/SiliconSurfer) · MCP browser
An MCP-compatible browser built for LLM agents. Where Playwright MCP dumps 25k
tokens of raw HTML, SiliconSurfer returns **finished data** — clean Markdown,
interactive element refs, structured JSON — across 5 vision modes.
- **30/30** extraction eval (vs Jina 20/30), **5/5** E2E (vs browser-use 0/5).
- **6.2× faster**; ~1ms warm calls via a shared daemon.

### 🔍 [arc](https://github.com/leocaolab/getarc) · code review · free
A.R.C. (Adversarial Resolution Cycle) — a code-review tool where a **Critic** LLM
files findings and a **Fixer** (Claude / Codex CLI) edits, looping until they
converge or you step in. Single Rust binary; state in `.arc/` (SQLite + SARIF).
Free binaries (engine closed-source). An earlier open-source Python version lives
at [arc-cli](https://github.com/leocaolab/arc-cli).

## 2 · Multi-interpreter Python

Since PEP 684, every sub-interpreter can have its own GIL. That means N
interpreters on N cores in **one** process, with no `multiprocessing` memory
tax. The catch is that every extension a sub-interpreter imports has to be
safe there, and most of the ecosystem isn't: it stores Python objects in
process-global state. We build the pieces that make it work: the server, the
JSON layer, and the Rust binding layer underneath.

### 🔥 [Pyronova](https://github.com/leocaolab/pyronova) · Rust
A high-performance Python web framework powered by a Rust core. Built on
Per-Interpreter GILs (PEP 684), it runs Python handlers across **every** CPU core
in a single process — no per-process memory tax of `multiprocessing`.
- **902k req/s** pipelined plaintext, **423k** on the standard baseline (8C/16T).
- **2.7× Robyn** at equal scale, in **1/3 the memory**.
- Sustained 400k QPS: **4 MB RSS growth over 73.8M requests** — ~0 B/req, zero leaks.

### 🧊 [isojson](https://github.com/leocaolab/isojson) · Rust
Fast JSON for Python that works in per-interpreter-GIL sub-interpreters, where
orjson refuses to load. It keeps orjson's API, and its output is byte-identical
to orjson's for the types it supports.
- **4–5× orjson's best single-process throughput**: 8 sub-interpreters in one
  process vs orjson on threads sharing one GIL (3.9× on Linux x86_64, 5.3× on
  macOS arm64). **~4× stdlib `json`** on the same sub-interpreters.
- **Pure Rust**, with no C anywhere. No override flags, no per-worker copies,
  and it can be called from any thread.
- **318/318 JSONTestSuite.** Found a silent data-corruption bug in simd-json,
  reported it, and sent the fix upstream
  ([#481](https://github.com/simd-lite/simd-json/issues/481)).

### 🦀 [PyO3, per-interpreter](https://github.com/leocaolab/pyo3) · Rust · experimental
A PyO3 fork that moves PyO3's process-global state **per interpreter**:
`#[pyclass]` and exception types, module objects, the `PyOnceLock` and
`intern!` caches, and the deferred-decref pool. PyO3 extensions then load
and run correctly in strict own-GIL sub-interpreters, where upstream PyO3
rejects them.
- **1.2× → 9.5× scaling** creating objects on 12 own-GIL workers. Upstream
  shares one type object across all interpreters, so every core contends on
  a single refcount.
- **Unmodified polars in 4/4 strict sub-interpreters** (upstream: 0/4, and
  1/4 even with the override flag), built from source against the fork.
  Compared with a copy of polars per worker: **35% less memory, ~80× faster
  cold start**. Its wrong-interpreter results (3 of 4 interpreters) and
  teardown segfaults (5 in 10 runs) are gone, with no change to polars.
- Stress testing found four open issues, including a cross-interpreter free
  in PyO3's global reference pool. Three are fixed. The fourth was mostly a
  measurement error: import cost is 1.02× upstream, not the 3.7× first
  reported. In all, **22 defects fixed**, each pinned by a regression test
  ([bug ledger](https://github.com/leocaolab/pyo3/blob/main/SUBINTERP-FIXES.md#bug-ledger)),
  and CI runs on every push. Synced with upstream `main`; the design and
  measurements are posted on [PyO3#3451](https://github.com/PyO3/pyo3/issues/3451).
- **polars Python UDFs in `group_by().agg()` run in the right interpreter.**
  polars calls them on its own thread pool; upstream PyO3 attaches that thread
  to the main interpreter. The fork attaches it to the interpreter that loaded
  that copy of polars: 7/7 UDF paths correct with one copy per worker.
  [What changes vs upstream, feature by feature](https://github.com/leocaolab/pyo3#how-it-differs-from-upstream-pyo3).
- **Pyronova runs on it** (v2.7.1+), and since v2.8.0 every worker loads the
  real Pyronova engine through it.

## How they fit

Two stacks, held to one standard. **TARS** is the runtime agents run on,
**SiliconSurfer** is how they read and act on the web, and **arc** keeps the
code honest. Underneath, **Pyronova** serves those workloads on every core of
one process. **isojson** and the **PyO3 fork** make the libraries it depends
on safe to run there, and Pyronova's own Rust core is next to move onto the
fork.

Mostly Rust under the hood. Every performance claim is backed by a
reproducible benchmark, and everything is built in public.

---

*Writing soon at [leocaolab-blog.pages.dev](https://leocaolab-blog.pages.dev)*
