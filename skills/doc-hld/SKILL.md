---
name: doc-hld
description: |
  Write an HLD (High-Level Design): the architecture of a system or module — topology,
  components and their responsibilities, request and data flows, high-level data model,
  public interfaces, scalability, security and observability. Works from existing sources
  (code, decision thread, prior docs) or through a structured interview. Use when: (1) a
  system or module needs its overall architecture written down, (2) a team needs the
  technical foundation before designing individual features, (3) an existing architecture
  must be documented for newcomers. Keywords: "hld", "high level design", "arquitetura",
  "desenho de arquitetura", "topologia", "visão arquitetural", "architecture document".
---

# doc-hld

Generate an HLD: the document that answers *"como o sistema se estrutura?"*.

The HLD is the technical foundation the FDDs stand on. It describes the **system**; an FDD
describes **one feature inside it**. If a section only makes sense for a single feature, it
belongs in the FDD.

Output language: **Portuguese by default**, or the `language` parameter.

## Base docs (read when relevant)

- Operating modes and the extraction contract: `https://github.com/dayvisonassis/docs-skills/blob/main/docs/modos-fonte-e-entrevista.md`
- Traceability and the sidecar format: `https://github.com/dayvisonassis/docs-skills/blob/main/docs/rastreabilidade.md`
- Document altitudes: `https://github.com/dayvisonassis/docs-skills/blob/main/docs/alturas-de-documentos.md`

The output skeleton is in `references/template-hld.md`. The interview guide is in
`references/guia-de-entrevista.md`.

## INPUT

| Parameter | Required | Default | Meaning |
| --- | --- | --- | --- |
| `output_path` | no | `docs/HLD.md` | Where the HLD goes |
| `sources` | no | — | Artifacts to extract from. Triggers FONTE mode |
| `codebase_root` | no | repo root | Code to analyse for the real architecture |
| `prd_path` / `adrs_folder` | no | — | Upstream documents to build on and link |
| `language` | no | `pt-BR` | Output language |
| `export_json` | no | `ask` | `true`, `false` or `ask`. Schema in `references/schema-hld.json` |
| `acceptance_criteria` | no | — | Path or inline checklist validated in Phase 4 |
| `trace_sidecar` | no | `true` | Emit `docs/.trace/HLD.jsonl` |

## OUTPUT

- The HLD at `output_path`
- The structured JSON, when requested — English keys, Portuguese values
- The traceability sidecar, when enabled

---

## EXECUTION

### Phase 1 — Determine the mode and gather material

Follow the mode contract in the base doc.

**When a codebase exists, read it before asking anything.** An HLD written from an interview
about a system that already runs describes the architecture someone *believes* is in place.
The code is the authority on what is actually there. Where they diverge, document what the
code does and flag the divergence — that gap is often the most useful thing in the document.

Map from the code: entry points, module and layer boundaries, dependency direction, where
persistence happens, where external calls happen, cross-cutting middleware, and the data
schema.

### Phase 2 — Write

Follow `references/template-hld.md`. Section-specific requirements:

- **Objetivo técnico** — what the system solves, technically. Do not restate the PRD's
  business case; link it.
- **Arquitetura geral** — topology, layers, main technologies with the reason each is there,
  deployment environment and architectural patterns in use.
- **Componentes e responsabilidades** — a table. Every component gets responsibilities **and**
  dependencies. A component whose responsibilities you cannot state in one line is either
  poorly bounded or two components.
- **Fluxo de requisições e de dados** — end to end. Where validation happens, where
  transformation happens, where data lands. Name the transaction boundaries.
- **Modelo de dados** — main entities, relationships and, explicitly, the **source of truth**.
- **Interfaces públicas** — a table with protocol, exposure (internal or external) and limits.
- **Escalabilidade e disponibilidade** — the strategy and the numeric target.
- **Segurança** — authentication, authorization, secrets management, encryption in transit and
  at rest, personal-data handling.
- **Observabilidade** — logs, metrics, tracing, dashboards and alerts.
- **Riscos arquiteturais** — each with probability, impact, mitigation and contingency.
- **ADRs e próximos passos** — decisions already recorded, decisions still pending with the
  criteria to settle them, and the technical path to the FDDs.

Where the sources give no answer, apply the defaults from the base doc and mark them
`(hipótese)`.

### Phase 3 — Validation (internal)

Run this checklist. Fix and re-run, up to 3 iterations.

- [ ] All mandatory sections present and non-empty
- [ ] The technical objective does not restate the PRD's business case
- [ ] The architecture supports the declared non-functional requirements. Check this
      explicitly: an availability target of 99.9% with a single point of failure in the
      topology is a contradiction, not an aspiration
- [ ] Every component has responsibilities **and** dependencies
- [ ] Request and data flows are complete end to end
- [ ] The data model names the source of truth
- [ ] Every public interface has protocol and exposure
- [ ] Scalability and availability carry numeric targets
- [ ] Every risk has probability, impact, mitigation and contingency
- [ ] Every file path and every ADR link cited exists
- [ ] Where the document diverges from the code, the divergence is flagged
- [ ] Hypotheses are visibly marked
- [ ] Altitude guard passes (see below)
- [ ] `acceptance_criteria`, when provided, satisfied item by item

**Altitude guard.** The HLD has leaked into FDD territory if it specifies a full endpoint
contract, error codes, retry values or column names. Naming an interface and its protocol is
HLD; specifying its payload is FDD. It has leaked into PRD territory if it argues business
value.

### Phase 4 — Write and verify

1. Write `output_path`.
2. Handle `export_json` as configured.
3. Write the sidecar when enabled: `HLD-COMP-NN`, `HLD-FLUXO-NN`, `HLD-IFACE-NN`,
   `HLD-RISCO-NN`.
4. **Read back** the file and confirm the mandatory sections are present.
5. Report: path, components documented, interfaces listed, divergences found between document
   and code, and anything left as a hypothesis.

---

## ALWAYS

- Read the code before describing an architecture that already exists.
- Give every component both responsibilities and dependencies.
- Name the source of truth for the data.
- Put a number on availability and scalability targets.
- Flag divergence between the intended architecture and the real one.
- Mark hypotheses visibly.

## NEVER

- Describe the architecture someone believes is in place when the code says otherwise.
- Specify endpoint payloads, error codes or column names.
- Restate the PRD's business justification.
- Leave a component without stated dependencies.
- Claim an availability target the topology cannot deliver.

## EDGE CASES

- **Greenfield, no code:** run in ENTREVISTA mode and mark the whole document as a proposed
  architecture rather than a description. The distinction matters to every later reader.
- **The code contradicts the intended architecture** (a layer importing what it should not, a
  component doing two jobs): document what exists, then state the intended rule and the
  divergence. Do not silently document the intention as if it were reality.
- **The system is too large for one HLD:** propose splitting by bounded context, one HLD per
  context plus a short document describing how they connect. Do not produce a 40-page HLD
  nobody reads.
- **No ADRs exist:** list the decisions visible in the code as candidates and suggest running
  `doc-adr`. An architecture with no recorded decisions is a documentation gap worth naming.
