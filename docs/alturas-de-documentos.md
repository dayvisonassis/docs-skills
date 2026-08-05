# Alturas de documentos

> Doc-base compartilhado pelas skills do pacote `docs-skills`.
> Ele existe para responder uma pergunta só: **este conteúdo pertence a este documento?**

## A ideia central

Cada documento do pacote opera em uma **altura** diferente. Eles não se repetem, eles se
encaixam. Quando o mesmo conteúdo aparece em dois documentos, isso não é redundância
saudável: é sinal de que ele está no lugar errado em pelo menos um dos dois.

| Documento | Papel | Altura | Pergunta que responde |
| --- | --- | --- | --- |
| **PRD** | Problema, público, escopo e métricas de sucesso | Produto / negócio | *Por que e o quê?* |
| **HLD** | Arquitetura geral do sistema ou módulo, componentes e integrações | Arquitetura de sistema | *Como o sistema se estrutura?* |
| **RFC** | Proposta técnica submetida à revisão: abordagem, alternativas e questões em aberto | Arquitetura de proposta | *Como pretendemos resolver, e o que ainda está em aberto?* |
| **ADR** | Uma decisão arquitetural isolada, com contexto e consequências | Decisão pontual | *Por que decidimos exatamente assim?* |
| **FDD** | Especificação de implementação: fluxos, contratos, erros, integração com o código | Implementação | *Como construir, em detalhe?* |
| **Tracker** | Rastreabilidade de cada item ao código ou às fontes | Transversal | *De onde veio cada coisa?* |
| **Diagramas** | Visualização derivada de um documento técnico | Transversal | *Como isso se parece?* |

Em uma frase: o **RFC propõe e abre para revisão**, os **ADRs registram cada decisão
fechada** e o **FDD detalha como construir**.

## Relação entre RFC, ADR e FDD

Esses três são os que mais se confundem na prática.

```
RFC          "Propomos usar o padrão Outbox. Consideramos fila dedicada e
             notificação síncrona. Ficou em aberto o volume esperado."
              │
              ├─► ADR  "Decidimos Outbox no MySQL. Alternativa: fila dedicada.
              │        Consequência negativa: polling adiciona latência."
              │
              └─► FDD  "Tabela webhook_outbox(id, event_id, payload, status,
                       attempts, next_retry_at). O worker faz SELECT ... FOR
                       UPDATE SKIP LOCKED a cada 2s, em lotes de 50."
```

- O **RFC** cita a decisão e o porquê em um parágrafo. Não traz o DDL.
- O **ADR** aprofunda **aquela decisão só**, com alternativas e consequências. Não traz o DDL.
- O **FDD** traz o DDL, os nomes de coluna, os índices e o intervalo de polling.

## Teste de altura

Antes de escrever um bloco em qualquer documento, aplique:

| Se o bloco... | Ele pertence a |
| --- | --- |
| justifica valor para o usuário ou para o negócio | PRD |
| define uma meta numérica de resultado (adoção, redução de custo, SLA de negócio) | PRD |
| descreve a topologia geral do sistema e como os módulos se falam | HLD |
| apresenta uma abordagem para revisão, com alternativas ainda em discussão | RFC |
| registra uma escolha já fechada, com o que se ganhou e o que se perdeu | ADR |
| nomeia tabela, coluna, endpoint, header, código de erro, timeout ou caminho de arquivo | FDD |
| aponta a origem de um item em uma fonte | Tracker |

## Sintomas de vazamento de altura

Cada skill do pacote checa estes sintomas na sua fase de validação interna:

- **RFC com contrato completo de endpoint** (payload de request e response, status codes):
  vazamento de FDD. Mantenha no RFC a menção de que haverá uma API de gestão; o contrato
  fica no FDD.
- **ADR que decide três coisas ao mesmo tempo**: não é um ADR, são três. Um ADR registra
  uma decisão.
- **PRD com nome de tabela ou de classe**: vazamento de FDD. O PRD fala em capacidade, não
  em implementação.
- **FDD explicando por que a feature é importante para o cliente**: vazamento de PRD. O FDD
  assume que a motivação de negócio já foi estabelecida e referencia o PRD.
- **FDD repetindo a análise de alternativas**: vazamento de ADR. O FDD implementa a decisão
  e linka o ADR.
- **RFC de dez páginas**: um RFC é conciso (2 a 4 páginas). Se cresceu, provavelmente
  absorveu conteúdo de FDD.

## Referência cruzada em vez de repetição

Quando um documento precisa de um conteúdo que vive em outra altura, ele **linka**:

```markdown
A escolha do padrão Outbox está registrada em [ADR-001](adrs/ADR-001-outbox-no-mysql.md).
O contrato completo dos endpoints de gestão está em [FDD, seção 5](FDD.md#5-contratos-públicos).
```

Isso mantém uma única fonte de verdade por item e faz o pacote inteiro envelhecer melhor:
quando a decisão muda, muda em um lugar só.
