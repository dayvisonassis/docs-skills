# Esqueleto de HLD (modelo de saída)

Textos entre colchetes são instruções e não aparecem na saída.

```markdown
# HLD: [nome do sistema ou módulo]

| | |
| --- | --- |
| **Versão** | [versão] |
| **Data** | [YYYY-MM-DD] |
| **Responsável técnico** | [nome] |
| **Documentos relacionados** | [PRD](PRD.md), [ADRs](adrs/) |

---

## Objetivo técnico

[O objetivo técnico do sistema ou módulo e qual problema arquitetural ele resolve. Não repita
a justificativa de negócio do PRD; referencie.]

**Dependências com outros sistemas**
- [sistema e natureza da dependência]

---

## Arquitetura geral

[Topologia em alto nível: camadas, serviços, processos, pipelines. Como as peças se
relacionam.]

**Ambiente de implantação**
- [cloud, on-premises ou híbrido, e a topologia]

**Tecnologias principais**
- [tecnologia e por que ela está aqui]

**Padrões adotados**
- [padrão arquitetural e onde se aplica]

---

## Componentes e responsabilidades

| Componente | Responsabilidades | Dependências |
| --- | --- | --- |
| [nome] | [o que faz, em uma linha] | [de quem depende] |

[Responsabilidade que não cabe em uma linha é sinal de componente mal delimitado.]

---

## Fluxo de requisições e de dados

**Fluxo de requisição**
1. [passo, dizendo onde valida, onde transforma, onde persiste]

**Fluxo de dados**
- [origem → transformação → destino]

**Fronteiras transacionais**
- [o que entra em transação, o que fica fora, e o que acontece se falhar no meio]

---

## Modelo de dados (alto nível)

**Entidades principais**
- [entidade e o que ela representa]

**Relações**
- [relação entre entidades]

**Fonte de verdade**
- [qual sistema ou tabela é a autoridade sobre cada dado]

**Retenção e versionamento**
- [políticas, quando houver]

---

## Interfaces públicas

| Nome | Tipo | Protocolo | Exposição | Limites e SLA |
| --- | --- | --- | --- | --- |
| [nome] | [API \| Queue \| Stream \| SDK] | [REST, gRPC, Kafka] | [interna \| externa] | [meta numérica] |

[Nomeie a interface e o protocolo. O contrato detalhado é do FDD.]

---

## Escalabilidade e disponibilidade

**Abordagem geral**
- [estratégia de scaling e resiliência]

**Técnicas aplicadas**
- [load balancing, cache, autoscaling, particionamento, sharding, backpressure]

**Meta de disponibilidade**
- [meta numérica. Confira se a topologia descrita sustenta essa meta]

---

## Segurança

**Autenticação**
- [mecanismo]

**Autorização**
- [modelo de permissão]

**Proteção de dados**
- [criptografia em trânsito e em repouso, tratamento de dado pessoal, retenção]

**Gestão de segredos**
- [onde vivem e como são rotacionados]

---

## Observabilidade

**Logs**
- [formato, campos obrigatórios, o que nunca é logado]

**Métricas**
- [métricas essenciais por interface e por componente]

**Tracing**
- [spans principais, propagação, amostragem]

**Dashboards e alertas**
- [painel ou alerta mínimo, com o gatilho]

---

## Riscos arquiteturais e mitigação

### [risco em uma frase]

- **Probabilidade:** [baixa | média | alta]
- **Impacto:** [consequência esperada]
- **Mitigação:**
  - [ação]
- **Plano de contingência:** [plano B]

---

## ADRs e próximos passos

**Decisões registradas**
- [ADR-NNN](adrs/ADR-NNN-....md) [título]

**Decisões pendentes**
- [o que falta decidir e qual critério vai fechar a decisão]

**Próximos passos**
- [ação técnica até os FDDs]
```

## Notas sobre o preenchimento

**Objetivo técnico.** O erro mais comum é reescrever o PRD aqui. O HLD assume que a
justificativa de negócio já existe e responde à pergunta seguinte: como o sistema se estrutura
para atender a ela.

**Componentes.** A tabela precisa das duas colunas. Componente sem dependência declarada é
componente que ninguém verificou. Se todos dependem de todos, o problema não é a
documentação, é a arquitetura, e vale dizer isso.

**Fronteiras transacionais.** Não está no esqueleto original do curso, mas é onde os bugs
reais nascem. Se o sistema tem transação, diga o que entra, o que fica fora e o que acontece
se o processo morrer no meio.

**Meta de disponibilidade.** Confira contra a topologia antes de escrever. Uma meta de 99,9%
com um ponto único de falha no desenho é contradição, não aspiração.

**Interfaces.** Nomeie e classifique. Payload, header e status code são do FDD.
