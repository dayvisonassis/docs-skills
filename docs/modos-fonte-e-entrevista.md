# Modos de operação: FONTE e ENTREVISTA

> Doc-base compartilhado pelas skills geradoras de documento do pacote `docs-skills`
> (`doc-prd`, `doc-hld`, `doc-rfc`, `doc-fdd`, `doc-adr`).
> As skills de derivação (`doc-tracker`, `doc-mermaid`, `doc-c4`, `doc-process-readme`)
> operam sempre a partir de artefatos e não usam este contrato.

Toda skill geradora precisa das mesmas informações para preencher seu esqueleto. O que muda é
**de onde a informação vem**.

| Modo | Quando | De onde vem a informação |
| --- | --- | --- |
| **FONTE** | Já existem artefatos: transcrição de reunião, código, PRD anterior, thread de decisão, ata, e-mail | Extração dos artefatos |
| **ENTREVISTA** | Existe uma ideia na cabeça de alguém, e mais nada | Perguntas ao usuário, uma por vez |
| **MISTO** | O caso normal | Extrai o que os artefatos sustentam, pergunta o resto |

## Detecção do modo

1. Se o parâmetro `sources` foi informado → **FONTE**.
2. Senão, procure candidatos no projeto: `docs/`, `*.md` na raiz, transcrições, ADRs
   existentes, código-fonte. Se encontrar algo plausível, **proponha** ao usuário:
   *"Encontrei `TRANSCRICAO.md` e o código em `src/`. Uso como fonte?"*
3. Senão → **ENTREVISTA**.

Não presuma modo FONTE em silêncio a partir de arquivos que o usuário não citou. Confirme.

---

## MODO FONTE

### 1. Inventário

Liste as fontes e dê um ID curto a cada uma (`T1` para a transcrição, `C1` para o
código, `D1` para um doc anterior). Reporte o inventário ao usuário antes de extrair.

Leia cada fonte **por inteiro**. Ler só o começo de uma transcrição de 300 linhas é a causa
número um de documento raso: as decisões costumam ser fechadas no último terço da reunião.

### 2. Extração carimbada

Cada fato extraído carrega sua origem. Sem origem, o fato não entra no documento.

| Tipo de fonte | Formato da localização | Exemplo |
| --- | --- | --- |
| Transcrição com timestamps | `[hh:mm] Falante` | `[09:17] Diego` |
| Transcrição sem timestamps | `Falante, trecho N` | `Sofia, bloco 4` |
| Código | caminho do arquivo, opcionalmente o símbolo | `src/modules/orders/order.service.ts` |
| Documento | caminho e seção | `docs/PRD.md, seção Escopo` |

### 3. Classificação (o passo que mais protege o documento)

Nem tudo que foi dito vira requisito. Classifique **cada item extraído**:

| Classe | Significado | Para onde vai |
| --- | --- | --- |
| `DECIDIDO` | Fechado, com acordo explícito na fonte | Requisito, decisão, ADR |
| `DESCARTADO` | Levantado e rejeitado | Fora de escopo; alternativa descartada no RFC |
| `ADIADO` | Aceito em princípio, jogado para depois | Fora de escopo, marcado como fase futura |
| `EM ABERTO` | Discutido sem conclusão | Questões em aberto do RFC |
| `SECUNDÁRIO` | Detalhe técnico mencionado de passagem | FDD, se sustentar a implementação; senão, descartar |

Sinais linguísticos úteis em transcrições: *"fechado então"*, *"vamos de"*, *"decidido"* →
`DECIDIDO`. *"não vamos fazer"*, *"descarta"*, *"não compensa"* → `DESCARTADO`. *"fase 2"*,
*"depois a gente vê"*, *"fica pra próxima"* → `ADIADO`. *"precisa confirmar"*, *"não sei
ainda"*, *"vamos medir"* → `EM ABERTO`.

Na dúvida entre `DECIDIDO` e `EM ABERTO`, escolha `EM ABERTO` e pergunte. O custo de
registrar uma decisão que não foi tomada é muito maior que o de fazer uma pergunta.

**Um item `DESCARTADO` ou `ADIADO` nunca aparece como requisito.** Ele aparece nomeado na
seção de exclusões, e é isso que prova que a leitura da fonte foi cuidadosa.

### 4. Lista de lacunas

O esqueleto do documento tem campos obrigatórios. Compare o que foi extraído com o que o
esqueleto exige e monte a lista de lacunas.

Para cada lacuna, escolha uma de três saídas, nesta ordem de preferência:

1. **Perguntar** ao usuário (uma pergunta por vez, com 2 ou 3 opções plausíveis quando fizer sentido).
2. **Marcar como hipótese** de forma visível no documento: `(hipótese)`. Use quando a lacuna
   for secundária e o usuário tiver pedido para não ser interrompido.
3. **Omitir** a subseção, quando o esqueleto permitir.

Nunca preencha uma lacuna com invenção silenciosa. Um número inventado num documento de
design vira requisito no sprint seguinte.

### 5. Saída

O documento no formato do esqueleto, mais o **sidecar de rastreabilidade** descrito em
[rastreabilidade.md](https://github.com/dayvisonassis/docs-skills/blob/main/docs/rastreabilidade.md).

---

## MODO ENTREVISTA

Conduza uma entrevista estruturada. As regras abaixo valem para todas as skills geradoras.

**Ritmo**
- Uma pergunta por vez. Aguarde a resposta antes da próxima.
- Nada de perguntas duplas (*"qual o volume e qual a latência?"* são duas perguntas).
- Ao fim de cada etapa, um resumo de 3 a 6 linhas do que você entendeu, e um pedido de
  confirmação antes de seguir.

**Quando o usuário não sabe**
- Ofereça 2 ou 3 opções plausíveis com o trade-off de cada uma.
- Se ele escolher, o item entra como decisão dele.
- Se ele não escolher, aplique o default inteligente e marque `(hipótese)`.

**Consistência**
- Se uma resposta contradiz outra anterior, aponte a contradição e peça o ajuste antes de
  continuar. Não tente conciliar sozinho.

**Ao final**
- Resuma o entendimento completo, peça confirmação e avise que vai gerar o documento.

### Defaults inteligentes

Use apenas quando o usuário não souber responder, e **sempre marcados como hipótese**:

- Latência p95 de APIs síncronas menor que 150 ms
- Disponibilidade alvo de 99,9% para sistemas voltados a cliente externo; 99,5% para internos
- Observabilidade mínima: logs estruturados, métricas de erro por endpoint, tracing distribuído ponta a ponta
- Segurança mínima: autenticação, autorização por papel, auditoria de alterações sensíveis
- Operações críticas sobre estado compartilhado devem ser transacionais

---

## MODO MISTO

O mais comum. Rode FONTE primeiro, depois use a lista de lacunas como roteiro de entrevista.
Ao apresentar o documento, deixe claro o que veio de onde:

> Extraí 24 itens da transcrição e 6 do código. Ficaram 3 lacunas: volume esperado de
> eventos, janela de retenção e responsável pelo RFC. Vou perguntar uma por vez.

---

## Estilo da saída

Vale para os dois modos:

- Idioma de saída: **português por padrão**, ou o valor do parâmetro `language`.
- Termos técnicos consagrados permanecem em inglês (outbox, worker, backoff, retry, span,
  payload, endpoint, timeout).
- Ortografia correta, com acentos e cedilha.
- Sem emojis.
- Sem travessão `—` no corpo dos documentos gerados.
- Números concretos sempre que a fonte permitir. "Rápido" não é requisito; "p95 abaixo de
  150 ms" é.
