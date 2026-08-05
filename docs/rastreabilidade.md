# Rastreabilidade

> Doc-base compartilhado pelas skills do pacote `docs-skills`.

## Por que rastrear

Um agente de IA lendo uma transcrição de 55 minutos e um código de 40 arquivos produz um
documento plausível com facilidade. Produzir um documento **verdadeiro** é outra coisa.

A rastreabilidade é o mecanismo que separa os dois. A regra é simples e não tem exceção:

> **Todo item registrado em um documento tem uma origem identificável em uma fonte.**
> Se você não consegue preencher a localização de um item, ele foi inventado. Corrija a
> origem ou remova o item.

Isso não é burocracia. É o único teste objetivo de alucinação que funciona sem um humano
reler tudo.

## O sidecar

Cada skill geradora emite, junto do documento, um arquivo de rastreabilidade em
**JSON Lines** (uma linha por item, extensão `.jsonl`):

```
docs/
├── FDD.md
└── .trace/
    └── FDD.jsonl
```

Formato de cada linha:

```json
{"id":"FDD-CONTRATO-01","documento":"docs/FDD.md","tipo":"Contrato","resumo":"POST /webhooks/endpoints cria endpoint com secret gerado","fonte":"TRANSCRICAO","localizacao":"[09:31] Larissa"}
```

| Campo | Conteúdo |
| --- | --- |
| `id` | Identificador único e estável do item. Convenção: `<DOC>-<TIPO>-<NN>` |
| `documento` | Caminho do documento onde o item aparece |
| `tipo` | Requisito Funcional, Requisito Não Funcional, Decisão, Restrição, Trade-off, Contrato, Risco, Métrica, Exclusão |
| `resumo` | Uma linha descrevendo o item |
| `fonte` | Rótulo da fonte. Convenções comuns: `TRANSCRICAO`, `CODIGO`, `DOC`, `ENTREVISTA`, `HIPOTESE` |
| `localizacao` | Onde exatamente na fonte. Ver a tabela de formatos em [modos-fonte-e-entrevista.md](https://github.com/dayvisonassis/docs-skills/blob/main/docs/modos-fonte-e-entrevista.md) |

O sidecar é o que permite ao `doc-tracker` montar a matriz de rastreabilidade
**mecanicamente**, em vez de reler todos os documentos e torcer para não esquecer nada.

### Convenção de IDs

| Prefixo | Uso |
| --- | --- |
| `PRD-FR-NN` | Requisito funcional do PRD |
| `PRD-NFR-NN` | Requisito não funcional do PRD |
| `PRD-OBJ-NN` | Objetivo com métrica |
| `PRD-OUT-NN` | Item fora de escopo |
| `PRD-RISCO-NN` | Risco |
| `RFC-ALT-NN` | Alternativa considerada |
| `RFC-OPEN-NN` | Questão em aberto |
| `FDD-FLUXO-NN` | Fluxo detalhado |
| `FDD-CONTRATO-NN` | Contrato público |
| `FDD-ERRO-NN` | Entrada da matriz de erros |
| `FDD-INT-NN` | Ponto de integração com o sistema existente |
| `ADR-NNN` | O ADR inteiro (um ADR é um item) |

IDs são estáveis: uma vez atribuídos, não são renumerados quando um item é removido. Isso
mantém válidas as referências cruzadas entre documentos.

### Quando o sidecar não existe

Skills antigas, documentos escritos à mão ou pacotes vindos de outro fluxo não têm sidecar.
O `doc-tracker` funciona mesmo assim: ele varre os documentos, identifica os itens e pede a
origem do que não conseguir resolver. O sidecar é uma otimização de confiabilidade, não uma
dependência.

### Sidecars no repositório

Os sidecars vivem em `docs/.trace/`. São úteis quando os documentos evoluem: ao regerar um
documento, a skill reaproveita os IDs já atribuídos. Se a entrega precisar de uma estrutura
de pastas exata, a pasta pode ser removida antes do commit final sem perda para os
documentos, que são autossuficientes.

## Validações automáticas

O `doc-tracker` roda três verificações que valem por uma revisão humana:

### 1. Existência dos caminhos citados

Todo caminho de arquivo mencionado em qualquer documento é verificado no disco. Caminho
inexistente significa uma de duas coisas, ambas graves: a IA inventou o arquivo, ou o
documento está desatualizado em relação ao código.

### 2. Cobertura

Percentual de itens identificáveis nos documentos que têm linha no tracker, e a distribuição
por fonte. Cobertura baixa não é um problema de tracker: é um sinal de que há conteúdo nos
documentos sem origem.

### 3. Coerência de localização

Timestamps citados existem na transcrição; nomes de falantes batem com os participantes
declarados; seções citadas existem nos documentos referenciados. Um `[09:17] Diego` numa
reunião em que o Diego só entrou às 09:05 e falou pela primeira vez às 09:06 é verificável,
e vale a pena verificar.

## O que fazer quando um item não tem origem

Em ordem:

1. **Procurar de novo.** O item pode estar na fonte com outras palavras.
2. **Perguntar ao usuário.** Ele pode ter o contexto que a fonte não registrou.
3. **Marcar como hipótese.** `fonte: HIPOTESE`, e o documento traz `(hipótese)` visível no
   texto.
4. **Remover o item.** Se ele não sustenta nem hipótese, não deveria estar no documento.

O que nunca fazer: deixar o item no documento com uma origem vaga ou inventada. Uma
localização falsa é pior que uma linha ausente, porque compra confiança que não foi
merecida.
