---
name: doc-c4
description: |
  Generate C4 model diagrams (Context, Container, Component, Code) in PlantUML from a
  technical document such as an FDD, HLD or architecture spec, producing one .puml file per
  level plus an analysis file, and optionally rendering them to PNG. Use when: (1) an
  architecture needs the standard C4 visual treatment, (2) a design document should be
  turned into layered diagrams for different audiences, (3) someone asks for context,
  container or component diagrams. Keywords: "c4", "plantuml", "diagrama de contexto",
  "container diagram", "component diagram", "modelo c4", "arquitetura visual".
---

# doc-c4

Generate C4 diagrams in PlantUML from a technical document.

Two rules carry this skill. **Generate a level only when the document supports it** — a
Container diagram invented from a document that never named a technology is fiction with a
legend. **Your deliverable is files** — if you did not call Write for each `.puml`, the task
failed, no matter how good the diagram content in your reply was.

## INPUT

| Parameter | Required | Default | Meaning |
| --- | --- | --- | --- |
| `source_document` | **yes** | — | Technical document to read (FDD, HLD, spec) |
| `output_folder` | no | `docs/c4` | Where the files go |
| `levels` | no | `auto` | Which levels to attempt. `auto` decides by sufficiency |
| `generate_png` | no | `false` | Render PNGs with the `plantuml` CLI |

## OUTPUT

- One `.puml` per generated level: `<feature>-c1.puml` … `<feature>-c4.puml`
- One `<feature>-c4.md` with the analysis. **Zero PlantUML code inside it**
- PNGs, when `generate_png` is on

Level-specific syntax, allowed elements and parameter order are in
`references/sintaxe-c4-plantuml.md`. **Read it before writing any diagram** — the C4-PlantUML
parameter order differs between `System_Ext` and `Container`, and getting it wrong renders
visible artifacts into the image.

---

## Language

The output matches the **language of the source document**. Detect it and follow it.

- Correct accents and cedilla everywhere: labels, descriptions, notes, titles.
- Technical terms stay in English: Service, Collector, Tracer, Logger, Span, Container,
  Database, API, REST, Redis, Kafka, Docker, Kubernetes.
- No emojis.

---

## EXECUTION

### Phase 1 — Read and map

Read `source_document` in full. Detect the language. Then map:

- **Explicit elements:** public interfaces, components and their relationships, named
  technologies with versions, external systems, data structures.
- **Inferences you would need to make**, each with the section that supports it. Minimise
  these, and mark them in the analysis file.
- **Exclusions:** anything the document marks as out of scope. These never appear in a
  diagram.
- **Component nature**, which decides the whole shape of C1 and C2:

| Signal | Nature | Consequence |
| --- | --- | --- |
| "embedded", "library", "SDK", "in-process" | Runs inside a host process | **Not** a separate `System()` or `Container()`. Mention it in the host's description |
| "service", "API", "microservice", "server", "worker" | Separate execution context | Gets its own `System()` / `Container()` |

Getting this wrong produces a diagram that misrepresents the deployment topology, which is the
single most damaging error at C1 and C2.

### Phase 2 — Assess sufficiency per level

For each level, decide whether the document supports it:

| Level | Requires | Generate when |
| --- | --- | --- |
| **C1 Context** | The system's purpose, external actors, external systems it integrates with | Business context is described |
| **C2 Container** | Technology stack, deployment units, how containers communicate | Technologies and deployment are specified |
| **C3 Component** | Internal breakdown, component responsibilities and interactions | Internal architecture is described |
| **C4 Code** | Interface signatures, data structures, algorithms, patterns | Code-level detail actually exists |

A level without enough information is **SKIPPED**, with the reason recorded in the analysis
file. Two accurate diagrams beat four with invented content.

### Phase 3 — Generate and write files

For each supported level, generate the PlantUML following
`references/sintaxe-c4-plantuml.md`, and **call Write immediately for that file**. Do not
batch the writes to the end; a long generation that fails midway leaves nothing on disk.

Then write `<feature>-c4.md` with: files created, levels skipped and why, explicit elements
found, inferences made with their justification, exclusions confirmed, component nature with
the reasoning, and a short description of each diagram with its audience.

### Phase 4 — Internal review

Silent quality gate. Re-read `source_document` and **every `.puml` you wrote**, then find and
fix:

- elements in the diagram that are absent from the document — **fabrication, remove**
- elements required by the document that are missing
- wrong technology names or versions
- relationships that contradict the document
- missing accents, or technical terms wrongly translated
- wrong detail level: implementation inside C1, or C4 without code detail
- excluded items present
- an embedded library drawn as a separate system

Fix everything with Edit. Do not document this review in the `.md` file, and do not add an
"issues resolved" section.

### Phase 5 — PNG rendering (only when `generate_png` is on)

1. Check availability: `plantuml -version`.
2. Render each `.puml`.
3. **On a syntax error, do not skip the diagram — fix it.** Read the error, locate the line,
   Edit the file, re-run. Up to 3 attempts per file, then report the failure and move on.
   The most common cause is `SHOW_LEGEND()` left in a C4 Code diagram, which is a class
   diagram and does not support it.
4. If `plantuml` is not installed, report it with the install command for the platform and
   continue. Missing PNGs do not fail the task; missing `.puml` files do.

### Phase 6 — Validate and report

- [ ] A `.puml` file exists on disk for every level generated
- [ ] The `.md` file exists and contains **no** PlantUML code
- [ ] Every `.puml` is standalone, with `@startuml` and `@enduml`
- [ ] `!pragma charset UTF-8` is the second line of every file
- [ ] Language matches the source document, accents correct
- [ ] Technical terms in English
- [ ] Parameter order correct per element type
- [ ] `SHOW_LEGEND()` in C1 to C3, absent in C4
- [ ] No fabricated elements, no excluded items
- [ ] Detail progresses C1 → C2 → C3 → C4

Report: detected language, every file created, levels skipped with reasons, the count
("N arquivos .puml para N diagramas"), and PNG results when applicable.

---

## ALWAYS

- Read `references/sintaxe-c4-plantuml.md` before writing diagram code.
- Call Write for each `.puml` as soon as it is generated.
- Put `!pragma charset UTF-8` as the second line of every file.
- Skip a level rather than invent content for it.
- Treat an embedded library as part of its host, never as a separate system.
- Record every inference with the section that supports it.

## NEVER

- Finish without writing the `.puml` files.
- Put PlantUML code inside the `.md` file.
- Invent an element the document does not contain.
- Diagram anything the document excludes.
- Put technology detail in C1, or leave C4 without code-level detail.
- Reference the source document's section numbers inside diagram labels or notes.
- Use emojis.
- Use `SHOW_LEGEND()` in a C4 Code diagram.

## EDGE CASES

- **Only C1 is supported by the document:** generate C1, skip the rest with reasons, and say
  what would be needed for each. That is a correct outcome, not a failure.
- **The document describes a library, not a service:** C1 and C2 show the host application
  with the library named in its description. This is the error most worth guarding against.
- **`plantuml` is not installed:** the `.puml` files are the deliverable and are already
  complete. Report the missing tool with install instructions.
- **The document contradicts itself:** diagram the version stated most explicitly and report
  the contradiction.
- **`source_document` does not exist:** ask for the correct path. Do not guess from the
  folder.
