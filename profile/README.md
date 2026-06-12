# leocaolab

**The infrastructure layer for AI-native software — and the benchmarks to prove it.**

Agents need three things production-grade: a runtime fast enough to serve them, a
way to act on the world, and a way to keep their code honest. That's the lab:
fast Python served by a Rust core, a Rust-first agent runtime, a browser agents can
actually read, an AI reviewer that holds the line — each one measured, never just
claimed. *Don't say it's fast. Prove it.*

## Projects

### 🔥 [Pyronova](https://github.com/leocaolab/pyronova) · Rust
A high-performance Python web framework powered by a Rust core. Built on
Per-Interpreter GILs (PEP 684), it runs Python handlers across **every** CPU core
in a single process — no per-process memory tax of `multiprocessing`.
- **902k req/s** pipelined plaintext, **423k** on the standard baseline (8C/16T).
- **2.7× Robyn** at equal scale, in **1/3 the memory**.
- Sustained 400k QPS: **4 MB RSS growth over 73.8M requests** — ~0 B/req, zero leaks.

### 🤖 [TARS](https://github.com/leocaolab/tars) · Rust · Apache-2.0
A Rust-first multi-agent LLM runtime. 10+ providers behind one trait, a composable
middleware pipeline, an Agent abstraction you hand tasks to, and Python + Node
bindings — with observability built in, not bolted on.
- Typed error hierarchy (`Permanent` / `Retryable` / `RateLimited` / `Auth`).
- Multi-tenancy enforced at every layer; cache hit/miss observable per call.
- Same Pipeline runs identically local (in-mem) and in a service (Redis + S3).
- *If you want to prototype fast, use LangChain. If you want to serve agents in
  production with the predictability of a database — TARS.*

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

## How they fit

These aren't four side projects — they're one stack. **Pyronova** serves agent
workloads at multi-core speed; **TARS** is the runtime those agents run on;
**SiliconSurfer** is how they read and act on the web; and **arc** keeps all of
it honest by reviewing the code. Mostly Rust under the hood, every performance
claim backed by a reproducible benchmark, built in public.

---

*Writing soon at blog.leocaolab.com*
