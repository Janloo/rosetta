# AGENTS.md — Rosetta Project Directives

> Local instructions for autonomous engineering agents working inside the **Rosetta** repository.

---

## 1. Repository Isolation & Scope

- **100% Autonomous Workspace**: `rosetta` is an independent, transversal AI-native programming language, compiler, and benchmark project.
- **Zero External Workspace Coupling**: Never read from, reference, import conventions from, or emit files into any external repository or unrelated workspace directory on disk. All specifications, benchmarks, tools, and tests for Rosetta live strictly inside this repository.
- **Language-Agnostic Core**: Keep the Rosetta language core, IR, and specification completely stack-agnostic. Concrete targets and baselines (such as JVM 21/25 + Maven as Target #1 and Java/Spring Boot as Concrete Baseline #1) live in dedicated adapter/benchmark directories.

---

## 2. Single Source of Truth (Low-Context Resume Protocol)

To avoid saturating conversation context across sessions, read files in this exact order when starting a task:

1. [`PROJECT-STATE.md`](PROJECT-STATE.md) — Current milestone status, frozen architectural decisions (`DEC-01`..`DEC-08`), KPI scoreboard, and active next step.
2. [`docs/analisi/01-analisi-rosetta-v0.1.md`](docs/analisi/01-analisi-rosetta-v0.1.md) — Complete formal analysis, 4-construct model, Causal Error Architecture, Polyglot Bridge spec, and `Bench-01` / `RealWorld` benchmark specifications.
3. Update `PROJECT-STATE.md` at the completion of every milestone or measurable benchmark run.

---

## 3. Specialist Team Roster

Coordinate engineering tasks across the following specialist roles:

- **`language-designer-agent`** (`pl-theory-ai-paradigms`, `causal-error-localization`): Core 4-construct Dataflow DAG model, type/effect system, Canonical Normal Form, Total Error Algebra.
- **`compiler-runtime-agent`** (`compiler-ir-engineering`, `polyglot-dependency-bridge`, `causal-error-localization`, `jvm-bytecode-runtime-interop`): Frontend codecs, target-agnostic HIR/MIR/LIR, Unified Causal Diagnostics, Polyglot Bridge #1 (Maven/JVM), JVM 21/25 Target #1 bytecode emission.
- **`ai-tokenomics-agent`** (`ai-token-economics-llm-ergonomics`, `causal-error-localization`): 3-KPI cost modeling (tokens, convergence iterations, test cost), multi-tokenizer BPE/SentencePiece audits, CFG/GBNF grammars, `Bench-01` & `RealWorld` empirical benchmarks.
- **`formal-verification-agent`** (`formal-verification-testability`, `causal-error-localization`, `qa-load-tester`): Pure effect handlers, Deterministic Simulation Testing (DST), Replay Capsules, Z3/CVC5 SMT verification, mutation testing, and black-box "human at the terminals" harnesses.
- **`spring-developer-agent` / `database-design-agent` / `qa-tester-agent` / `security-auditor-agent`**: Used strictly for building/auditing concrete reference baselines (e.g., Baseline #1 Java/Spring Boot) and verifying black-box parity and capability security.

---

## 4. Mandatory Quality & Verification Gates

Before marking any milestone complete, verify all applicable gates:

1. **Compiler & Type/Effect Tests**: Unit tests covering canonical parsing, bidirectional type/effect inference, Maranget exhaustiveness, linearity, and IR lowering.
2. **Target Runtime & Bytecode Verification**: `java -Xverify:all`, `javap -v` inspection, Virtual Thread pinning checks, and real JDBC/FS smoke execution for Target #1.
3. **Causal Error Localization & Replay Capsules**: Verify every error emits the Unified AI Diagnostic Envelope with exact `origin_node`, minimal `causal_slice`, zero framework stack-trace noise, and `< 5 ms` in-memory `replay_capsule` reproduction.
4. **Black-Box "Human at the Terminals" Parity**: Every benchmark application (Rosetta vs concrete baseline) must pass the identical external HTTP/DB/FS black-box test suite.
5. **Multi-Tokenizer Benchmarks**: Record real token counts across `>= 2` tokenizers plus iteration counts (`pass@1`, $R$, $R_{\text{fault}}$) inside `PROJECT-STATE.md`.

---

## 5. Git Hygiene

- Keep repository artifacts clean and deterministic (no temporary build outputs or secrets committed; respect `.gitignore`).
- Do **not** run `git commit`, `git push`, or create branches unless explicitly requested by the user.
