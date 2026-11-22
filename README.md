# Rosetta

**Rosetta** è un linguaggio di programmazione sperimentale **AI-native, trasversale e non human-friendly**, progettato per minimizzare congiuntamente:

1. il consumo totale di token (scrittura, lettura contesto, manutenzione),
2. il numero di iterazioni necessarie all'IA per convergere alla soluzione corretta (`pass@1` e diagnosi errori $R_{\text{fault}} \le 1$),
3. il costo e i tempi di verifica formale ed end-to-end.

L'essere umano non scrive né legge il codice interno: interagisce esclusivamente **"ai morsetti"** (specifiche e contratti in ingresso, suite di test black-box su API/DB/File e certificati di correttezza in uscita).

---

## Pilastri Architetturali

- **Modello a 4 Costrutti Ortogonali (`Schema`, `Port`, `Artifact`, `Topology`)**: sostituisce il flusso imperativo sequenziale e la frammentazione OOP multi-file con grafi dataflow/effetti di alto livello indirizzati per `node_id`.
- **Architettura Causale degli Errori**: algebra totale degli errori (zero `null`, zero eccezioni non tracciate), `origin_node` esatto, *Backward Dataflow Slicing* automatico sul grafo immutabile, *Deterministic Replay Capsules* (`< 5 ms` in memoria) e diagnostica JSON compatta unificata.
- **Trasversalità Multi-Target & Polyglot Dependency Bridge**: semantica agnostica dal runtime; primo target concreto **JVM 21/25 LTS** con **Bridge #1 Maven** per riusare driver e librerie reali dietro `Port` tipizzati e sanitizzati.
- **Verifica Black-Box ("Human at the Terminals")**: stessa suite di test esterni eseguibile sia contro Rosetta sia contro baseline tradizionali (Java/Spring Boot, TypeScript/Node, Python) su applicazioni 3-tier reali.

---

## Stato e Documentazione

- Stato operativo e KPI aggiornati: [`PROJECT-STATE.md`](PROJECT-STATE.md)
- Documento di analisi e specifica v0.1: [`docs/analisi/01-analisi-rosetta-v0.1.md`](docs/analisi/01-analisi-rosetta-v0.1.md)
- Direttive locali per agenti IA: [`AGENTS.md`](AGENTS.md)
