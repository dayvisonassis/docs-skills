# Guia de entrevista para HLD

Usado no modo ENTREVISTA. As regras gerais de ritmo estão em
[modos-fonte-e-entrevista.md](https://github.com/dayvisonassis/docs-skills/blob/main/docs/modos-fonte-e-entrevista.md).

Quando existe código, **leia o código antes de perguntar**. A entrevista serve para o que o
código não conta: intenção, restrição externa e decisão ainda não implementada.

## Mensagem inicial

> Vou te fazer perguntas sobre objetivo técnico, arquitetura, componentes, fluxos, dados,
> interfaces, escalabilidade, segurança, observabilidade e riscos. No final entrego o HLD no
> formato padrão e, se quiser, também em JSON estruturado. Podemos começar com um resumo
> técnico do sistema e qual problema arquitetural ele resolve?

## As 11 etapas

### 1. Contexto e objetivo técnico

- Qual o objetivo técnico do sistema ou módulo?
- Quais problemas do estado atual serão endereçados?
- Quais sistemas ou features se conectam a este?

### 2. Arquitetura geral

- Qual a topologia em alto nível? Camadas, microsserviços, agentes, pipelines.
- Quais as tecnologias principais, e por que cada uma?
- Qual o ambiente de implantação?
- Que padrões arquiteturais estão em uso? Event-driven, hexagonal, CQRS, REST, gRPC.

### 3. Componentes e responsabilidades

- Quais os componentes principais e o papel de cada um?
- Quais as dependências internas e externas de cada um?
- Quem persiste dado, quem faz cache, quem orquestra fluxo?

### 4. Fluxo de requisições e de dados

- Qual o caminho ponta a ponta de uma requisição típica?
- Onde acontece validação, transformação, enfileiramento?
- Onde e como os dados são persistidos e replicados?
- Onde ficam as fronteiras transacionais?

### 5. Modelo de dados em alto nível

- Quais as entidades principais e como se relacionam?
- Qual sistema é a fonte de verdade de cada dado?
- Existe política de retenção ou de versionamento?

### 6. Interfaces públicas

- Que interfaces o sistema expõe? APIs, filas, streams, SDKs.
- Quais protocolos e formatos?
- Cada uma é interna ou externa? Que limites e SLAs valem?

### 7. Escalabilidade e disponibilidade

- Qual a estratégia de scaling? Horizontal, particionamento, sharding.
- Usa cache, rate limiting, backpressure?
- Qual a meta de disponibilidade, e como o sistema se recupera de falha?

### 8. Segurança

- Como funcionam autenticação e autorização?
- Onde vivem os segredos, e como são rotacionados?
- Há criptografia em trânsito e em repouso?
- Como o sistema trata dado pessoal?

### 9. Observabilidade

- Qual a política de log estruturado?
- Quais métricas são essenciais por interface e por componente?
- Existe tracing distribuído? Com que amostragem?
- Quais painéis e alertas são indispensáveis?

### 10. Riscos arquiteturais

- Quais os principais riscos técnicos?
- Para cada um: probabilidade, impacto, mitigação e plano de contingência.

### 11. ADRs e próximos passos

- Que decisões já estão registradas?
- Que decisões ainda estão pendentes, e qual critério vai fechá-las?
- Quais os próximos passos técnicos até os FDDs?

## Checagens antes de gerar

- O objetivo técnico está claro e não repete o PRD.
- A arquitetura descrita sustenta os requisitos não funcionais declarados. Em especial,
  confira a meta de disponibilidade contra a topologia.
- Todo componente tem responsabilidades e dependências explícitas.
- Os fluxos de requisição e de dados estão completos ponta a ponta.
- O modelo de dados nomeia a fonte de verdade.
- Toda interface pública tem protocolo e exposição.
- Escalabilidade e disponibilidade têm metas numéricas.
- Segurança e observabilidade têm práticas verificáveis, não intenções.
- Todo risco tem probabilidade, impacto, mitigação e plano de contingência.
- ADRs registrados e decisões pendentes estão listados.
