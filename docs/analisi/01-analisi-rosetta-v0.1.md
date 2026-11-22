# Analisi & Specifica Architetturale — Rosetta v0.1

> **Stato documento**: `Congelato per Milestone M0` (`2026-11-22`)  
> **Ambito**: Linguaggio di programmazione AI-native, trasversale, non human-friendly, ad altissima efficienza di risorse IA e localizzazione causale deterministica degli errori.

---

## 1. Visione e Principi Fondamentali

**Rosetta** parte da un ribaltamento netto rispetto ai linguaggi tradizionali: l'essere umano **non scrive, non legge e non debugga** il codice sorgente interno. I costrutti storici nati per la memoria a breve termine umana — flusso di controllo imperativo riga-per-riga, variabili mutabili, gerarchie OOP, file sparsi per strati tecnici (`Controller/Service/DTO/Entity/Mapper/Repository`), stack trace verbosi — rappresentano puro attrito per un'IA.

Principi cardine di Rosetta:

1. **Zero Compromessi Human-Friendly**: nessuna sintassi pensata per il "piacere di lettura" umano né costrutti ridondanti per esprimere la stessa semantica.
2. **100% Trasversale (Stack-Agnostic)**: il modello semantico di Rosetta non appartiene a JVM, Node, Python o .NET; i runtime e gli ecosistemi concreti sono target pluggabili.
3. **Artifact-Centric, non File-Centric**: l'unità di lavoro dell'IA è una funzionalità applicativa completa e autocontenuta, non un puzzle di 8 file separati.
4. **L'Umano è "ai Morsetti" (Black-Box Verification)**: lo sviluppo interno è una scatola nera; l'umano fornisce contratti/specifiche in ingresso e verifica correttezza, integrità dati, file prodotti e costi ai morsetti esterni tramite test agnostici dall'implementazione.
5. **Localizzazione Causale Nativa**: ogni errore — in compilazione, verifica formale, FFI o runtime — deve indicare matematicamente *dove* è nato, *da quali dati* proviene e *come riprodurlo* in millisecondi.

---

## 2. Obiettivi Misurabili & 3 KPI Co-Paritari

Rosetta evita il *"code-golf trap"* (accorciare i sorgenti con simboli criptici che frammentano i tokenizer BPE/SentencePiece e aumentano gli errori dell'IA). Si ottimizza il costo completo del ciclo di vita:

$$\text{TotalCost} = T_{\text{spec}} + T_{\text{context\_read}} + T_{\text{reasoning}} + T_{\text{output}} + \sum_{k=1}^{R} (T_{\text{diag}_k} + T_{\text{repair}_k}) + T_{\text{tests}}$$

Tre KPI obbligatori di pari priorità:

1. **KPI-1 — Token Totali End-to-End**: somma di token letti in contesto ($T_{\text{context\_read}}$), ragionamento ($T_{\text{reasoning}}$), generazione iniziale ($T_{\text{output}}$) e modifiche incrementali di manutenzione ($M$), misurata su almeno **2 tokenizer reali** (`o200k_base`/`cl100k_base` e `Gemini/SentencePiece`).
2. **KPI-2 — Iterazioni di Convergenza ($R$)**: tasso `pass@1` sintattico/semantico (tramite grammatica canonica vincolabile CFG/GBNF) e soprattutto numero medio di turni di riparazione $R_{\text{fault}}$ su errori reali a runtime (target: mediana $R_{\text{fault}} \le 1$).
3. **KPI-3 — Costo e Velocità di Verifica**: eliminazione dei mock pesanti e del boot di container per testare la logica di business tramite sostituzione deterministica dei `Port`, abbinata a test black-box agnostici ai morsetti.

---

## 3. Modello Computazionale Rosetta v0.1 (I 4 Costrutti Ortogonali)

Un sistema Rosetta è descritto interamente attraverso **4 costrutti ortogonali**:

| Costrutto | Ruolo | Cosa elimina rispetto ai linguaggi classici |
|---|---|---|
| **`Schema`** | Algebra dei tipi (Sum/Product ADTs), record strutturali, **invarianti/refinement nativi** (`Amount > 0`, `Text[1..64]`) | Eliminazione di `null` e duplicazione `Entity` / `Command` / `RequestDTO` / `ResponseDTO` / `Mapper` |
| **`Port`** | Connettori tipizzati verso il mondo esterno e librerie di terze parti: `HttpIn`, `Db`, `Fs`, `Clock`, `Auth`, `Ext[Ecosystem:Coord]` | Eliminazione di side effect nascosti, annotations magiche, reflection DI e eccezioni non dichiarate |
| **`Artifact`** | **Unità fondamentale**: modella una funzionalità di alto livello (`Record.Create`, `Record.Export`) come **Grafo Aciclico Diretto (Dataflow + Effect DAG)** con nodi immutabili (`node_id`) | Eliminazione del flusso imperativo sequenziale riga-per-riga, delle mutazioni locali e della dispersione multi-file |
| **`Topology`** | Mappa dichiarativa statica che collega ogni `Port` a un provider di produzione o a un handler puro in memoria per i test | Eliminazione di file YAML/XML di configurazione runtime opachi e framework di mock |

### 3.1 Proprietà dell'`Artifact` Dataflow DAG

- **Ordine dedotto dal grafo**: l'esecuzione (e il parallelismo sicuro) deriva esclusivamente dalle dipendenze tra nodi ed effect token (`Db`, `Fs`), mai dall'ordine lessicale delle righe.
- **Routing Totale degli Esiti**: ogni nodo fallibile produce una *Union chiusa* (`Ok | Err1 | Err2`); il compilatore rifiuta qualsiasi Artifact che lasci un ramo d'errore scollegato.
- **Oracoli Co-Locati**: precondizioni, postcondizioni e invarianti di proprietà possono essere agganciati direttamente al nodo/Artifact.

### 3.2 Formato Nativo Fase 1 (Pragmatico)

Per ottenere risultati misurabili rapidamente senza sovra-ingegneria infrastrutturale:
- Rappresentazione testuale/strutturata **rigidamente canonica** a grafo con `node_id` espliciti (1 intento semantico = 1 forma canonica, zero zucchero sintattico opzionale).
- Progettata per **Constrained Decoding** (grammatica LL(1)/CFG/GBNF a zero ambiguità).
- Modifiche incrementali tramite **patch atomiche node-addressed**: `add_node`, `replace_node(id, ...)`, `wire_error(from, variant, to)`, `delete_node(id)`.

### 3.3 Sketch Concettuale Esemplificativo *(bozza non congelata)*

```text
schema Record { id: Uuid, code: Str[1..32], title: Str[1..120], amount: Dec{>0}, status: Draft|Active|Archived, ver: Int{>=1} }

artifact Record.Create [HttpIn, Db, Clock] -> HttpOut {
  n1 = HttpIn.json(POST "/records", {code, title, amount})
       | Err.Schema -> out:422
  n2 = Clock.now()
  n3 = Db.tx.insert("records", {id: uuid(), ...n1.ok, status: Draft, ver: 1, created_at: n2})
       | Err.Unique("uq_records_code") -> out:409("DUPLICATE_CODE", n1.ok.code)
       | Err.DbIo -> out:500
  n4 = HttpOut.json(201, n3.ok)
}
```
*(In ~10 righe canoniche convivono contratto HTTP, validazione tipizzata, transazione DB, gestione esplicita del conflitto `409` e risposta `201` senza stato mutabile né stack OOP).*

---

## 4. Architettura Causale degli Errori ("Root Cause sempre compresa dall'IA")

Il punto debole principale nello sviluppo assistito da IA è la diagnostica frammentata (eccezioni non locali, stack trace di centinaia di righe, errori DB/FS opachi, impossibilità di riprodurre il guasto al volo). Rosetta lo affronta con **5 livelli integrati**:

1. **Algebra Totale degli Errori**: zero `null`, zero `throw`, zero `try/catch` sparsi; ogni errore attraversa gli archi espliciti del tipo somma.
2. **Provenienza Node-Addressed**: ogni fallimento identifica esattamente `origin: { artifact, node, hash }` e la catena minima di nodi Rosetta attraversati, azzerando il rumore degli stack trace dei runtime sottostanti.
3. **Backward Dataflow Slicing Automatico**: poiché l'Artifact è un DAG SSA immutabile, al fallimento di `n_k` il runtime estrae il **sottografo minimo a monte** con i valori concreti transitati sugli archi. L'IA vede immediatamente dove si è generato il dato errato.
4. **Deterministic Replay Capsules (`< 5 ms`)**: ogni `Port` registra ingressi ed esiti delle chiamate esterne; in caso di errore live (es. su DB reale o disco) viene salvata una capsula compatta rieseguibile in memoria in `< 5 ms` per verificare la fix senza riavviare server o DB.
5. **Unified AI Diagnostic Envelope**: identico schema JSON (~40–100 token, zero prosa) per errori di parsing, tipi, SMT, test, FFI e runtime:

```json
{
  "stage": "RUNTIME_PORT",
  "code": "DB_UNIQUE_VIOLATION",
  "origin": { "artifact": "Record.Create", "node": "n3_db_insert" },
  "root_cause": { "port": "Db", "sqlstate": "23505", "constraint": "uq_records_code", "field": "code" },
  "causal_slice": [
    { "node": "n1_http_in", "out": { "code": "REC-42" } }
  ],
  "expected_vs_actual": { "expected": "Unique(records.code)", "actual": "Exists('REC-42')" },
  "replay_capsule": "cap:7a91c0",
  "repair_hints": [
    { "op": "wire_error", "from": "n3_db_insert.Err.UniqueViolation", "to": "http_409" }
  ]
}
```

### Tassonomia Canonica Errori 3-Tier

- `VALIDATION` (violazione schema/refinement su ingress HTTP/query)
- `INVARIANT` (controesempio SMT o regola di dominio violata)
- `CONTRACT` (scostamento rispetto al contratto OpenAPI/payload atteso)
- `DB_CONSTRAINT` (`SQLSTATE`, vincolo `UNIQUE`/`FK`/`CHECK`/`NOT NULL`, tabella/colonna)
- `DB_CONNECTIVITY` (pool/timeout/connessione)
- `FS_IO` (operazione, path, `errno` OS es. `EACCES`/`ENOENT`)
- `EXT_FFI` (eccezione o null intercettato al confine di una libreria esterna)
- `CONCURRENCY` (conflitto di optimistic lock `version`)
- `BUG_INTERNAL` (anomalia interna del compilatore/runtime con capsule)

---

## 5. Trasversalità Multi-Target & Polyglot Dependency Bridge ("Una cosa per volta")

Per evitare un linguaggio isolato inutilizzabile su problemi reali, Rosetta separa la semantica dai binding di ecosistema:

1. **Modello Astratto (`ForeignRegistry -> CapabilityStub -> Port`)**:
   - Un harvester legge i metadati nativi dell'ecosistema esterno (`.pom`/`.jar`, `.d.ts`, `.pyi`, `.wit`, `.dll`) e proietta **solo le firme effettivamente usate** in uno stub Rosetta compatto (pochi token, nessuna documentazione umana ridondante).
   - Il codice di business vede solo `Port` (`Ext[Maven:"..."]`), con **sanitizzazione obbligatoria al confine**: `null -> Option<T>`, eccezioni/codici nativi $\rightarrow$ Error Union tipizzata + metadati causali.
   - Nei test ogni `Port` esterna è sostituibile da un handler deterministico in memoria.
2. **Esecuzione a 2 Modalità**:
   - *Tier A (Same-Target Native Linking)*: overhead zero quando target compilato ed ecosistema libreria coincidono.
   - *Tier B (Cross-Ecosystem Bridge — fasi successive)*: WASM Component Model (WIT), FFM o IPC tipizzato per ecosistemi misti.
3. **Target Concreto #1 & Bridge #1 (Fase Iniziale)**:
   - **Target #1**: **JVM 21/25 LTS** (bytecode diretto con `-Xverify:all` e Project Loom Virtual Threads per I/O non bloccante senza complessità reactive).
   - **Bridge #1**: **Maven** (driver JDBC per qualsiasi database relazionale, HTTP server leggero, File I/O atomico via `java.nio.file`).

---

## 6. L'Umano "ai Morsetti" & Protocollo di Benchmark

L'umano controlla esclusivamente i **morsetti esterni**:
- **Morsetti in Ingresso**: contratto OpenAPI/REST, schema relazionale e vincoli, path/formato file su disco, matrice risposte e casi di errore.
- **Morsetti in Uscita**: suite black-box automatica identica su Rosetta e sulle baseline tradizionali + **Certificato di Correttezza e Costo** (copertura contratti, 0 rami non gestiti, mutation score $\ge 90\%$, token spesi, iterazioni $R$ e $R_{\text{fault}}$).

### 6.1 Livello 1 — `Rosetta Bench-01` (App Web 3-Tier, 6 Endpoint)

Modello relazionale portabile (`records`, eseguibile su H2 / PostgreSQL / qualsiasi DB JDBC):
- `id` (UUID PK), `code` (VARCHAR UNIQUE NOT NULL), `title` (VARCHAR NOT NULL), `amount` (DECIMAL > 0), `status` (`DRAFT | ACTIVE | ARCHIVED`), `version` (INT >= 1, optimistic locking), `created_at`, `updated_at`.

| # | Endpoint | Semantica richiesta ai morsetti | Esiti HTTP attesi |
|---|---|---|---|
| 1 | `POST /records` | Validazione payload (`code`, `title`, `amount > 0`), `INSERT` DB, gestione conflitto `UNIQUE(code)` | `201`, `400/422`, `409` |
| 2 | `GET /records/{id}` | Validazione UUID path, lookup DB | `200`, `400`, `404` |
| 3 | `GET /records?status=&page=&size=` | Validazione query (`page >= 0`, `1 <= size <= 100`), filtro opzionale + paginazione deterministica | `200`, `400/422` |
| 4 | `PUT /records/{id}` | Update transazionale con controllo optimistic lock (`expectedVersion`) | `200`, `400/422`, `404`, `409` |
| 5 | `DELETE /records/{id}` | Cancellazione con verifica esistenza e integrità | `204`, `400`, `404` |
| 6 | `POST /records/{id}/export` | Lettura DB + **scrittura atomica file su disco** (es. JSON/CSV in directory configurata) + mapping errori I/O | `200`, `404`, `500/503` tipizzato |

**Prove di evoluzione e diagnosi su `Bench-01`**:
- **3 Task di Manutenzione**: `MNT-1` aggiunta campo validato `currency` (`EUR|USD|GBP`); `MNT-2` regola di stato (`ARCHIVED` non modificabile via `PUT` -> `409`); `MNT-3` aggiunta formato `CSV` + checksum SHA-256 su `POST /records/{id}/export`.
- **3 Fault-Injection Reali** *(per misurare $R_{\text{fault}}$)*: `FLT-1` violazione DB constraint non mappata / errore SQL; `FLT-2` errore permessi / directory mancante su export file; `FLT-3` regressione logica su filtro/paginazione o optimistic lock.

### 6.2 Livello 2 — `RealWorld (Conduit)` + File Export *(dopo M4)*

Estensione a 20+ endpoint (utenti, articoli, commenti, tag, preferiti, feed, auth JWT, export) per dimostrare che il costo di contesto dell'IA cresce in modo sub-lineare ($O(1)\text{--}O(\log N)$) al crescere dell'applicazione.

---

## 7. Confronto con Progetti Simili di Linguaggi Nativi IA (2025–2026)

Il panorama censito attorno a `agentlanguages.dev` (e allo studio arXiv *"The Best Programming Language for Tokenmaxxing"*, luglio 2026) mostra tre approcci principali:

- **Verification Camp (`Vera` / `aallan/vera`, `AILANG`, `Zero`, `With`)**: *Vera* elimina i nomi di variabili con slot De Bruijn (`@Int.0`), usa contratti `requires`/`ensures`/`effects` con Z3 e compila a WASI; *AILANG* applica effetti row-polimorfici (`! {IO}`). Ottimi sulla verifica statica, ma lavorano ancora a livello di singole funzioni e mancano di bridge verso ecosistemi DB/enterprise reali e causal slicing a runtime.
- **Syntactic Camp (`Codong`, `ilo`, `NERD`, `X07`, `Mog`, `LLMLang`, `Sever`)**: puntano su forma canonica unica, compattezza sintattica o AST JSON/SSA ed errori JSON di compilazione. Ottimi spunti per densità token, ma deboli su workflow 3-tier reali, fault-injection runtime e riuso librerie.
- **Ibridi (`MoonBit`, `BAML`) & Precursori (`Unison`, `Koka`, `Darklang`)**: *MoonBit* integra un agente ma resta human-readable; *BAML* è un DSL per chiamate LLM; *Unison* e *Koka* forniscono basi teoriche fondamentali (content-addressing e algebraic effects), ma pensate per programmatori umani.

**Posizionamento unico di Rosetta**: unisce (1) **Artifact Dataflow DAG di alto livello**, (2) **Polyglot Dependency Bridge concreto (Maven/JVM #1)** e (3) **Causal Error Localization a runtime + Morsetti Black-Box** per l'umano.

---

## 8. Struttura Target del Repository & Roadmap

```text
rosetta/
├── AGENTS.md                         # Direttive operative locali per gli agenti
├── PROJECT-STATE.md                  # Stato compatto, decisioni congelate, KPI live
├── README.md                         # Panoramica pubblica del progetto
├── docs/
│   └── analisi/
│       └── 01-analisi-rosetta-v0.1.md
├── benchmarks/
│   ├── bench-01-terminals/           # (M1) Morsetti black-box agnostici (HTTP + DB + FS + Faults)
│   └── baseline-java-spring/         # (M1) Baseline concreta #1 (Java 21 / Spring Boot)
├── compiler/                         # (M2) Frontend canonico, HIR/MIR/LIR, Causal Slicer, Bridge #1
├── runtime-jvm/                      # (M2) Micro-kernel Virtual Threads, Replay Capsules, JDBC/FS Ports
└── examples/
    └── bench-01-rosetta/             # (M3) Implementazione Bench-01 in Rosetta v0.1
```

- **M0 (Completata)**: Team skill, analisi v0.1, stato progetto, repository pronto per GitHub.
- **M1**: Morsetti Black-Box `Bench-01` + Baseline Java/Spring con misurazione reale token e iterazioni.
- **M2**: Schema canonico + grammatica GBNF + prototipo compilatore/runner JVM + Causal Envelope.
- **M3**: `Bench-01` in Rosetta al 100% dei morsetti + Replay Capsules `< 5 ms`.
- **M4**: Report empirico comparativo su token, iterazioni ($R$, $R_{\text{fault}}$), manutenzioni e fault-injection.
- **M5**: Scalata a `RealWorld (Conduit)` e nuovi bridge/target (npm, PyPI, WASM).
