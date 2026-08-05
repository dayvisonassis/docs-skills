# docs-skills — Design docs para Claude Code

Um conjunto portátil de skills que produzem documentação técnica a partir do que você já
tem: uma transcrição de reunião, o código do sistema, uma thread de decisão. Ou, quando não
há nada ainda, a partir de uma entrevista estruturada.

> Novo por aqui? Comece pelo **[docs/GUIA_DO_WORKFLOW.md](docs/GUIA_DO_WORKFLOW.md)**. Ele
> explica qual skill chamar, em que ordem e por quê, sem precisar abrir nenhum `SKILL.md`.

---

## O que tem aqui

```
docs-skills/
├── skills/
│   ├── doc-prd/              # PRD de feature: por que e o quê
│   ├── doc-hld/              # arquitetura do sistema: como ele se estrutura
│   ├── doc-rfc/              # proposta técnica submetida à revisão
│   ├── doc-adr/              # ADRs em formato MADR, uma decisão por arquivo
│   ├── doc-fdd/              # especificação de implementação
│   ├── doc-tracker/          # matriz de rastreabilidade e auditoria contra alucinação
│   ├── doc-mermaid/          # diagramas Mermaid derivados de um documento técnico
│   ├── doc-c4/               # diagramas C4 em PlantUML, um arquivo por nível
│   └── doc-process-readme/   # o README que documenta como o pacote foi produzido
└── docs/                     # docs-base compartilhados + o guia do workflow
```

Cada skill é autossuficiente: basta copiar ou instalar a pasta. As referências longas ficam
em `references/` dentro de cada skill, carregadas sob demanda; os docs-base compartilhados
são linkados por URL absoluta e resolvem a partir de qualquer instalação.

---

## As três ideias por trás do pacote

### 1. Dois modos, porque a informação nem sempre está na sua cabeça

Toda skill geradora funciona em **modo FONTE** (extrai de artefatos existentes) ou **modo
ENTREVISTA** (pergunta, uma pergunta por vez, com resumo e confirmação a cada etapa). O caso
normal é misto: extrai o que os artefatos sustentam e pergunta o resto.

### 2. Classificação antes de geração

Nem tudo que foi dito em uma reunião vira requisito. Cada item extraído é classificado em
`DECIDIDO`, `DESCARTADO`, `ADIADO`, `EM ABERTO` ou `SECUNDÁRIO`. Só `DECIDIDO` vira
requisito. O que foi rejeitado vai para "fora de escopo", nomeado, com a razão. O que ficou
em aberto vira questão do RFC.

Sem esse passo, uma ideia que a reunião descartou reaparece como requisito, e ninguém percebe
até alguém tentar implementá-la.

### 3. Rastreabilidade como teste de alucinação

Todo item registrado carrega a origem: `[09:17] Diego` para transcrição,
`src/modules/orders/order.service.ts` para código. Se a origem não pode ser preenchida, o
item foi inventado.

O `doc-tracker` transforma isso em auditoria automática: confere se todo caminho de arquivo
citado existe no disco, se toda localização é válida na fonte, e qual a cobertura real. É o
único teste objetivo de alucinação que funciona sem um humano reler tudo.

---

## Requisitos

Funciona em **macOS, Linux e Windows**.

- **Node.js** (que traz o `npx`), único requisito da Opção A.
  - macOS: `brew install node`, ou o instalador em nodejs.org
  - Linux: `apt install nodejs npm`, `dnf install nodejs`, ou o gerenciador da sua distro
  - Windows: instalador em nodejs.org
- **Suporte a symlink**, usado pelo `npx skills`: nativo em macOS e Linux; no Windows, ative
  o *Modo de Desenvolvedor* (ou rode como administrador), senão o instalador cai para cópia.

Sem Node.js? Use a **Opção B**, que não depende de nada.

## Instalação

As skills são descobertas por convenção: qualquer pasta sob um diretório de skills que
contenha um `SKILL.md` válido é carregada. Há dois escopos:

| Escopo | Local | Vale para |
| --- | --- | --- |
| **Usuário (global)** | `~/.claude/skills/` (Windows: `C:\Users\<você>\.claude\skills\`) | todos os projetos |
| **Projeto** | `<projeto>/.claude/skills/` | só aquele projeto |

### Opção A — `npx skills`

```bash
npx skills add dayvisonassis/docs-skills --skill "*"
# ou escolhendo:
npx skills add dayvisonassis/docs-skills --skill doc-adr,doc-rfc,doc-fdd,doc-prd,doc-tracker
```

> ⚠️ **Prefira a instalação por projeto à global.** Quando a mesma skill existe nos dois
> escopos, uma das cópias vence em silêncio. Se a global estiver desatualizada, você roda
> instruções velhas sem nenhum aviso. Se instalar globalmente, ressincronize a cada
> atualização, e não compare por tamanho de arquivo no Windows: só o CRLF já infla centenas
> de bytes. Compare com `diff --strip-trailing-cr`.

### Opção B — copiar as pastas

**macOS / Linux**

```bash
cp -r skills/* ~/.claude/skills/          # global
cp -r skills/* <projeto>/.claude/skills/  # por projeto
```

**Windows (PowerShell)**

```powershell
Copy-Item -Recurse skills\* "$env:USERPROFILE\.claude\skills\"
Copy-Item -Recurse skills\* "<projeto>\.claude\skills\"
```

> Copie a **pasta inteira** da skill, incluindo `references/`, não só o `SKILL.md`. A pasta
> `docs/` não precisa ser copiada: as skills linkam os docs-base por URL absoluta. Copie
> `docs/` só se quiser o guia disponível localmente também.

### Verificar

```bash
npx skills list
```

Ou abra o Claude Code no projeto e confira se os nomes `doc-*` aparecem na lista de skills.

---

## O fluxo em uma olhada

```
fontes (transcrição, código, thread)
   │
   ├─► doc-adr ─► doc-rfc ─► doc-fdd ─► doc-prd
   │                 │          │
   │              doc-hld    doc-mermaid / doc-c4
   │
   └────────────► doc-tracker ─► doc-process-readme
```

Decisões primeiro, porque elas são o esqueleto do "como implementar". PRD por último, porque
com o resto pronto ele vira consolidação em vez de adivinhação. Tracker no fim, porque ele
audita o pacote inteiro.

Receitas prontas e a explicação de cada passo estão no
[guia do workflow](docs/GUIA_DO_WORKFLOW.md).

---

## Idioma

Saída em **português por padrão**, com o parâmetro `language` para gerar em outro idioma. As
instruções internas dos `SKILL.md` são em inglês; os documentos gerados, não. Termos técnicos
consagrados (outbox, worker, backoff, payload, span) permanecem em inglês em qualquer idioma
de saída.

## Adaptar ao seu contexto

As skills são genéricas de propósito. O que é específico do seu time entra por parâmetro:

- `acceptance_criteria` — sua definição de pronto, validada na fase de validação interna de
  cada skill
- `error_code_prefix`, `min_functional_requirements`, `min_contracts`, `max_pages` — pisos e
  tetos do seu padrão

Assim a mesma skill serve para um projeto pessoal, para o trabalho e para uma entrega
avaliada contra uma rubrica, sem fork.

## Relação com o `sdd-skills`

O [`sdd-skills`](https://github.com/dayvisonassis/sdd-skills) roda o fluxo de
desenvolvimento: spec, contrato, implementação, avaliação. Este pacote roda o fluxo de
documentação. Eles são complementares e podem conviver instalados juntos.

Uma observação sobre nomes: o `sdd-skills` tem um `prd-writer` que escreve um PRD **de
produto** em inglês, com 9 seções, grafo de dependências e ondas de execução. O `doc-prd`
daqui escreve um PRD **de feature** em português, com requisitos funcionais e não funcionais,
decisões com trade-off, riscos e estratégia de testes. Alturas diferentes, formatos
diferentes, sem colisão de nome.

## Licença

[MIT](LICENSE) © dayvisonassis
