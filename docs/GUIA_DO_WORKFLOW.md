# Guia do workflow

Como usar o pacote `docs-skills` no dia a dia: qual skill chamar, quando, e o que entra e sai
de cada uma. Se você só vai ler um documento deste repositório, leia este.

## As skills

| Skill | Produz | Responde |
| --- | --- | --- |
| `doc-prd` | `docs/PRD.md` | Por que e o quê? |
| `doc-hld` | `docs/HLD.md` | Como o sistema se estrutura? |
| `doc-rfc` | `docs/RFC.md` | Como propomos resolver, e o que está em aberto? |
| `doc-adr` | `docs/adrs/ADR-NNN-*.md` | Por que decidimos exatamente assim? |
| `doc-fdd` | `docs/FDD.md` | Como construir, em detalhe? |
| `doc-tracker` | `docs/TRACKER.md` | De onde veio cada coisa? |
| `doc-mermaid` | `docs/mermaid/*-diagrams.md` | Como isso se parece? |
| `doc-c4` | `docs/c4/*.puml` | Como isso se parece, em camadas? |
| `doc-process-readme` | `README.md` | Como este pacote foi produzido? |

Você invoca todas diretamente. Nenhuma chama outra sozinha.

## Ordem de produção

A ordem importa mais do que parece, e a intuitiva costuma ser a errada.

```
fontes (transcrição, código, thread de decisão)
   │
   ├─► doc-adr        decisões primeiro: elas são o esqueleto de tudo
   │      │
   │      ├─► doc-rfc     a proposta se apoia nas decisões e linka os ADRs
   │      │      │
   │      │      └─► doc-fdd    o desenho técnico se constrói sobre a proposta
   │      │             │
   │      │             ├─► doc-mermaid    diagramas derivados do FDD
   │      │             │
   │      │             └─► doc-prd        por último: vira consolidação
   │      │                     │
   │      └─────────────────────┴─► doc-tracker    varre tudo e audita
   │                                    │
   └────────────────────────────────────┴─► doc-process-readme
```

**Por que ADRs primeiro.** As decisões são a espinha do "como implementar". Com elas
fechadas, o RFC vira narrativa e o FDD vira detalhamento. Sem elas, os dois ficam
argumentando a mesma escolha em lugares diferentes.

**Por que o PRD por último.** Ele opera no nível mais alto. Com RFC, FDD e ADRs em mãos, o
PRD é consolidação, não adivinhação, e para de contradizer os documentos abaixo dele.

**Por que o tracker no fim.** Ele audita o pacote inteiro. Rodá-lo antes é auditar um pacote
que ainda vai mudar.

Quando não existe fonte nenhuma e a feature ainda é uma ideia, inverta: `doc-prd` em modo
entrevista primeiro, e o resto depois.

## Os dois modos

Toda skill geradora opera em **FONTE** (extrai de artefatos) ou **ENTREVISTA** (pergunta,
uma pergunta por vez). O caso normal é misto: extrai o que os artefatos sustentam, pergunta o
resto. O contrato completo está em
[modos-fonte-e-entrevista.md](modos-fonte-e-entrevista.md).

O passo que mais protege o resultado é a **classificação**: cada item extraído vira
`DECIDIDO`, `DESCARTADO`, `ADIADO`, `EM ABERTO` ou `SECUNDÁRIO`. Só `DECIDIDO` vira
requisito. Descartado e adiado vão para fora de escopo; em aberto vira questão do RFC. Sem
isso, uma ideia rejeitada na reunião reaparece como requisito no sprint seguinte.

## Rastreabilidade

Cada skill geradora emite um sidecar em `docs/.trace/<doc>.jsonl` com a origem de cada item.
O `doc-tracker` agrega os sidecars e roda três verificações:

- todo caminho de arquivo citado existe no disco
- toda localização citada é válida na fonte
- cobertura: quantos itens têm origem identificável

Detalhes em [rastreabilidade.md](rastreabilidade.md).

## Alturas

Os documentos não se repetem, eles se encaixam. Antes de escrever um bloco, o teste está em
[alturas-de-documentos.md](alturas-de-documentos.md). Conteúdo duplicado entre dois
documentos é sinal de que ele está no lugar errado em pelo menos um.

## Parâmetros que valem conhecer

| Parâmetro | Em quais | Para quê |
| --- | --- | --- |
| `sources` | geradoras | Aciona o modo FONTE |
| `language` | todas, menos `doc-mermaid` | Idioma de saída. Padrão `pt-BR`. O `doc-mermaid` segue o idioma do documento de origem |
| `acceptance_criteria` | todas, menos `doc-mermaid` | Sua checklist de pronto, validada na fase de validação interna |
| `trace_sidecar` | geradoras | Liga ou desliga o sidecar de rastreabilidade |
| `error_code_prefix` | `doc-fdd` | Prefixo dos códigos de erro, por exemplo `WEBHOOK_` |
| `min_*` | várias | Pisos: requisitos, contratos, ADRs, prompts |

`acceptance_criteria` é o que mantém as skills genéricas: os requisitos específicos do seu
time, do seu cliente ou de um desafio entram como parâmetro, não hardcoded na skill.

## Receitas

### Reunião gravada virou decisão, e agora precisa virar documentação

```
doc-adr    sources=TRANSCRICAO.md, código
doc-rfc    sources=TRANSCRICAO.md, adrs_folder=docs/adrs
doc-fdd    sources=TRANSCRICAO.md, codebase_root=., rfc_path=docs/RFC.md
doc-prd    sources=TRANSCRICAO.md, rfc_path, fdd_path, adrs_folder
doc-tracker  documents=docs/**/*.md, sources=TRANSCRICAO.md
```

### Feature nova, nada escrito ainda

```
doc-prd    (entrevista)
doc-rfc    (entrevista, a partir do PRD)
doc-adr    (as decisões que o RFC fechou)
doc-fdd    (entrevista + código)
```

### Documentação pronta que você quer auditar

```
doc-tracker  documents=docs/**/*.md, sources=<as fontes originais>
```

Roda sozinho. Não precisa de sidecar: reconstrói a rastreabilidade a partir dos documentos e
reporta o que não tem origem.

### Documento técnico que precisa de diagramas

```
doc-mermaid  source_document=docs/FDD.md
doc-c4       source_document=docs/HLD.md
```

Os dois se complementam e não competem. O `doc-mermaid` gera diagramas embutidos no Markdown,
que renderizam direto no GitHub e servem para fluxo, algoritmo e contrato. O `doc-c4` gera
arquivos PlantUML separados por nível, no modelo C4 clássico, e serve quando você precisa da
progressão contexto, container, componente e código para públicos diferentes.

### Sistema existente que ninguém documentou

```
doc-hld    sources=<código>, codebase_root=.
doc-c4     source_document=docs/HLD.md
doc-adr    sources=<código>   (as decisões visíveis na estrutura)
```

O `doc-hld` lê o código antes de perguntar qualquer coisa: a arquitetura real é a que está no
código, não a que alguém acredita estar. Onde as duas divergem, ele registra a divergência,
que costuma ser o achado mais útil do documento.
