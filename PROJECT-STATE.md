# PROJECT-STATE.md — Stato del Progetto Rosetta

> Fonte unica e compatta dello stato operativo di **Rosetta** per riprendere il lavoro tra sessioni senza saturare il contesto.

- **Progetto**: `Rosetta` (Linguaggio di programmazione AI-native, trasversale, non human-friendly)
- **Stato corrente**: `M0 — Analisi v0.1 Congelata (Pronto per repository GitHub & avvio M1)`
- **Ultimo aggiornamento**: `2026-11-22`
- **Specifica completa**: [`docs/analisi/01-analisi-rosetta-v0.1.md`](docs/analisi/01-analisi-rosetta-v0.1.md)

---

## 1. Decisioni Architetturali Congelate (v0.1)

| ID | Decisione | Sintesi Operativa |
|---|---|---|
| `DEC-01` | **Core 100% Trasversale** | Semantica agnostica da JVM/Spring/Node/Python; target multipli pluggabili (Target #1 concreto = **JVM 21/25 LTS**). |
| `DEC-02` | **3 KPI Co-Paritari** | Ottimizzazione congiunta di: (1) Token totali (scrittura + contesto + manutenzione), (2) Iterazioni fino al verde (`pass@1`, $R$, $R_{\text{fault}}$), (3) Costo/velocità di test e verifica. |
| `DEC-03` | **Modello a 4 Costrutti** | Solo `Schema` (algebra + invarianti), `Port` (capability/bridge tipizzati), `Artifact` (funzionalità di alto livello = Dataflow + Effect DAG senza flusso imperativo), `Topology` (wiring statico). |
| `DEC-04` | **Formato Fase 1 Pragmatico** | Grafo strutturato canonico ultra-compatto indirizzato per `node_id`, vincolabile da grammatica CFG/GBNF e modificabile con patch atomiche per nodo (niente DB binari custom in Fase 1). |
| `DEC-05` | **Architettura Causale Errori** | Zero `null`/eccezioni, Total Error Algebra, `origin_node`, **Backward Dataflow Slicing**, **Deterministic Replay Capsule** (`< 5 ms`), **Unified AI Diagnostic Envelope** (40–100 token). |
| `DEC-06` | **Polyglot Bridge Incrementale** | Astrazione agnostica `ForeignRegistry -> CapabilityStub -> Port` con sanitizzazione al confine; implementazione incrementale partendo da **Bridge #1 = Maven/JVM**. |
| `DEC-07` | **Umano "ai Morsetti" (Black-Box)** | Sviluppo interno black-box per l'umano; verifica esterna agnostica dall'implementazione su morsetti HTTP/OpenAPI + DB + File su disco + Certificato di Correttezza e Costo. |
| `DEC-08` | **Benchmark a 2 Livelli** | **Livello 1 (`Bench-01`)**: app 3-tier da 6 endpoint (REST + validazione + DB + scrittura atomica file) + 3 manutenzioni + 3 fault-injection. **Livello 2**: `RealWorld (Conduit)` + export file. |

---

## 2. Roadmap & Milestone Tracker

- [x] **M0 — Fondazione, Team Skill & Analisi v0.1**: Definizione obiettivi, benchmark, differenziatori competitivi e documentazione persistente disaccoppiata.
- [ ] **M1 — Morsetti Black-Box `Bench-01` & Baseline Concreta #1**: Specifica OpenAPI + suite black-box HTTP/DB/FS + 3 harness fault-injection + implementazione Baseline #1 (Java 21 / Spring Boot) con raccolta metrica iniziale.
- [ ] **M2 — Schema Canonico Rosetta v0.1 & Prototipo Compilatore/Runner**: Grammatica CFG/GBNF, IR `HIR -> MIR -> LIR`, Emitter JVM Target #1 + Maven Bridge #1 (JDBC/HTTP/FS), Unified Causal Envelope, Backward Slicing e Replay Capsules.
- [ ] **M3 — `Bench-01` in Rosetta v0.1**: Implementazione completa dei 6 endpoint in Rosetta, verifica `100%` ai medesimi morsetti black-box di M1, test pure-effect e Replay Capsules `< 5 ms`.
- [ ] **M4 — Benchmark Empirico #1 (Dati Reali)**: Confronto misurato su `>= 2` tokenizer + iterazioni ($R$, $R_{\text{fault}}$) per creazione da zero, 3 task di manutenzione e 3 fault-injection vs Baseline #1 (+ confronto architetturale con Vera / Codong / ilo).
- [ ] **M5 — Scalata a Livello 2 (`RealWorld Conduit`) & Nuovi Bridge/Target**: Verifica di scalabilità sub-lineare del contesto su 20+ endpoint ed estensione ad altri ecosistemi (npm / PyPI / WASM).

---

## 3. KPI Scoreboard (Live)

| Metrica | Baseline #1 (Java/Spring) | Rosetta v0.1 | Target | Stato |
|---|---|---|---|---|
| Token totali costruzione `Bench-01` (multi-tokenizer) | `TBD (M1)` | `TBD (M4)` | `<= -60%` vs Baseline | In attesa |
| Iterazioni a convergenza costruzione (`R`) | `TBD (M1)` | `TBD (M4)` | `<= 1` (syntax `pass@1 ~100%`) | In attesa |
| Token totali 3 task di manutenzione (`MNT-1..3`) | `TBD (M1)` | `TBD (M4)` | `<= -70%` vs Baseline | In attesa |
| Iterazioni riparazione fault-injection (`R_fault`) | `TBD (M1)` | `TBD (M4)` | Mediana `<= 1` | In attesa |
| Tempo esecuzione suite core logica/effetti | `TBD (M1)` | `TBD (M3)` | `< 1s` (in-memory pure handlers) | In attesa |
| Parità ai morsetti black-box (`Bench-01`) | `TBD (M1)` | `TBD (M3)` | `100%` pass identico | In attesa |

---

## 4. Prossimo Passo Immediato

1. Creazione repository GitHub da parte dell'utente e primo push dello stato `M0`.
2. Avvio **M1**: costruzione dei **morsetti black-box di `Bench-01`** (`benchmarks/bench-01-terminals/`) e della **Baseline #1** (`benchmarks/baseline-java-spring/`).
