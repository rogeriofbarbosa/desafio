---
status: proposta
date: 2026-11-02
decision-makers: arquitetura do Einstein (alçada a confirmar), sobre proposta de Rogério Barbosa
consulted: plataforma Einstein+ (pendente)
informed: banca do desafio de arquitetura
---
# RDA-0004: Guardar a conversa do agente na Memória Redis da plataforma, separada do registro das análises

**Finalidade:** definir onde guardar a conversa e os rascunhos do contexto, por que separá-los do registro das análises e qual runtime usar para o agente.

**Identificador:** RDA-0004
**Sistema / domínio:** Agente de Completude de Normativos e Cadeia de Valor
**Estado:** proposta
**Autores:** Rogério Barbosa (arquiteto, autor da proposta)
**Responsável pela decisão:** arquitetura do Einstein (pendente)
**Data da proposta / da decisão:** 2026-10-31 / pendente
**DDA vinculado:** [`../dda/01-agente-de-completude.md`](../dda/01-agente-de-completude.md), §6 (DEC-04), §8 (CNT-06, CNT-10), §10 (DATA-01, DATA-07), versão 0.1
**Projetos afetados:** agente de contexto, API de Completude, PostgreSQL

## 0. Controle e governança do registro

| Campo | Valor |
|---|---|
| ID e título | RDA-0004: Guardar a conversa do agente na Memória Redis da plataforma, separada do registro das análises |
| Categoria | Dados / IA / Plataforma |
| Abrangência | sub-agente de processos (cenário 1) |
| Estado e validade | proposta; 2026-11-02 |
| Decisor com alçada | arquitetura do Einstein (pendente) |
| Autores / consultados / discordâncias | Rogério Barbosa / plataforma Einstein+ (pendente) / nenhuma registrada |
| DDA, especificação, requisito, incidentes associados | DDA §6 DEC-04; [especificação](../../especificacao/pendentes/agente-de-completude-2026-11.md) §3.1; RF-01 |
| Registros relacionados | - |
| Repositório e revisão | `arquitetura_7.md`, revisão 7, Q4 e §7.4 |
| Data de reavaliação ou gatilho | situação da Memória Redis (prevista, set-26) e da ligação Copilot Studio e LangChain |

### 0.1 Histórico de estados

| Data | Estado anterior e novo | Quem autorizou | Justificativa / referência |
|---|---|---|---|
| 2026-10-31 | rascunho para proposta | autor e responsável pelo desenho | o responsável definiu dois bancos separados e escolheu a memória Redis da plataforma |
| 2026-11-02 | proposta registrada como RDA | autor | este arquivo |

## 1. Declaração objetiva da decisão

**Questão a decidir:** onde guardar a conversa e os rascunhos até a confirmação do gestor, e em qual runtime executar o agente? As opções de armazenamento são o banco das análises, uma memória da plataforma, um PostgreSQL separado ou o estado próprio de um serviço de agentes.

**Decisão proposta:** usar a Memória Redis de curto prazo da plataforma para a conversa e os rascunhos, com o agente em LangChain (Agentes Online). O PostgreSQL das análises recebe apenas o contexto confirmado. A escolha reaproveita a plataforma, mas depende da disponibilidade de um item previsto para set-26.

## 2. Contexto, problema e forças em conflito

**Situação atual (AS-IS):** N/A, sistema novo. O slide 4 mostra a memória ligada aos Agentes Online, com Redis de curto prazo previsto para set-26 e memória de longo prazo para 2027. Também informa que o LangChain está implantado nos Agentes Online desde set-26.
**Problema ou risco que motiva a escolha:** o rascunho é descartável e muda a cada turno. Já o registro das análises precisa ser auditável e conter apenas o que o gestor confirmou. Guardá-los juntos reúne dados com ciclos de vida, responsáveis e credenciais diferentes.
**Por que decidir agora:** define o cenário 1 e o que a API grava.
**Forças em conflito:** reaproveitar a memória da plataforma evita um banco próprio, mas depende da disponibilidade prevista. O rascunho precisa expirar, enquanto um banco durável exigiria uma rotina de limpeza.
**Linha de base:** nenhuma.
**Premissas declaradas:** a Memória Redis está disponível (DEP-02), e o Copilot Studio encaminha a conversa ao agente em LangChain (PRE-07).
**Fora do escopo:** memória de longo prazo, prevista para 2027.

## 3. Fatores de decisão, requisitos, restrições e critérios

### 3.1 Critérios verificáveis

| ID | Critério | Natureza | Regra ou métrica de aceite | Importância | Evidência / fonte |
|---|---|---|---|---|---|
| CR-01 | nada no registro das análises antes da confirmação | eliminatório | a primeira gravação no PostgreSQL é o POST do pedido | obrigatório | cenário 1 |
| CR-02 | agente sem acesso ao registro das análises | eliminatório | identidade do agente sem permissão no PostgreSQL | obrigatório | PRI-04 |
| CR-03 | dado descartável separado de registro auditável | eliminatório | dois bancos, com donos e credenciais distintos | obrigatório | decisão do responsável pelo desenho |
| CR-04 | reaproveitar a plataforma | preferência | componente do slide 4 | alta | RES-01 |
| CR-05 | expiração natural dos rascunhos | preferência | expiração configurada nas chaves | média | - |
| CR-06 | perda de rascunho tolerável | preferência | perder um rascunho só obriga a refazer a conversa | média | cenário 1 |

**Restrições não negociáveis:** RES-01, RES-03.
**Requisitos afetados:** RF-01.

### 3.2 Regra de priorização

Aplicam-se primeiro os critérios eliminatórios. CR-04 tem prioridade sobre CR-05 e CR-06.

## 4. Alternativas consideradas

| Alternativa | Descrição / mecanismo | Atende as restrições? | Benefícios | Limitações e riscos | Evidência disponível |
|---|---|---|---|---|---|
| ALT-A | Memória Redis da plataforma; agente em LangChain | sim | reuso; expiração natural; separação | depende de item previsto (set-26) | slide 4 |
| ALT-B | um único PostgreSQL para rascunhos e registro | não (CR-03) | uma peça a menos | mistura ciclos de vida, donos e credenciais | - |
| ALT-C | PostgreSQL separado, só para a memória | sim | sob controle do produto; durável | componente novo; expiração por rotina | - |
| ALT-D | Foundry Agent Service com estado próprio | sim | serviço gerenciado de agentes | em homologação (dez-26); duplica a memória da plataforma, guardando a conversa em recursos próprios | documentação do Foundry (standard agent setup) |
| ALT-E | status quo da revisão 6: rascunho gravado no PostgreSQL pela API, por rotas do agente | não (CR-01, CR-02) | um caminho só | o agente escreve no registro; rascunho vira registro | revisão 6 da proposta |

**Descartadas antes da comparação:** ALT-B e ALT-E, por critério eliminatório.
**Incertezas não resolvidas:** DEP-02 e PRE-07.
**Dependência de fornecedor e reversibilidade:** trocar ALT-A por ALT-C muda só a ferramenta de rascunho do agente.

## 5. Comparação das alternativas

| Critério | ALT-A | ALT-C | ALT-D | Fundamentação e grau de confiança |
|---|---|---|---|---|
| CR-01 (eliminatório) | atende | atende | atende | por construção; alto |
| CR-02 (eliminatório) | atende | atende | atende | por construção; alto |
| CR-03 (eliminatório) | atende | atende | atende | por construção; alto |
| CR-04 | melhor | pior | similar | slide 4; alto |
| CR-05 | melhor | pior | similar | médio |
| CR-06 | atende | atende | atende | médio |
| Reversibilidade | alta | alta | média | - |

**Análise crítica:** ALT-A oferece reuso e expiração natural. Se a Memória Redis não estiver disponível, ALT-C é a contingência. ALT-D depende de dois itens em homologação e duplica a memória.
**Teste de sensibilidade:** sem a Memória Redis em set-26, ALT-C passa à frente.
**Validade da comparação:** 2026-10-31.

## 6. Decisão e justificativa

**Opção selecionada:** ALT-A, com ALT-C como contingência.
**Formulação normativa:** o sub-agente de processos deve executar em LangChain e guardar a conversa e os rascunhos na Memória Redis da plataforma. A API só deve gravar o contexto no PostgreSQL das análises após a confirmação do gestor. Se o Redis estiver indisponível, a contingência é um PostgreSQL separado para a memória (ALT-C).
**Justificativa principal:** atende aos eliminatórios e vence CR-04 e CR-05.
**Alternativas descartadas:** ALT-B e ALT-E violam critérios eliminatórios. ALT-C acrescenta um componente e fica como contingência. ALT-D duplica a memória da plataforma.
**Condições de aceitação:** VAL-01, VAL-03 e VAL-09 do DDA.
**Data efetiva:** após o aceite e a Entrega 1.
**Limites:** não decide a memória de longo prazo.

## 7. Como a decisão altera o desenho

| Aspecto | Mudança resultante | Artefato a alterar | Responsável |
|---|---|---|---|
| Componentes e fronteiras | agente em LangChain; Memória Redis da plataforma | DDA §8 (CNT-10) | arquiteto |
| Dados e consistência | rascunhos expiram; o contexto confirmado vai no pedido e é gravado pela API | DDA §10 (DATA-01, DATA-07) | arquiteto |
| Contratos e eventos | ferramentas do agente só para RAG e memória | DDA §11 (API-05) | arquiteto |
| Plataforma e operação | dependência de item previsto | DDA §2.2 (DEP-02) | plataforma |
| Segurança e privacidade | identidade do agente sem acesso ao PostgreSQL nem à API | DDA §13 (SEC-06) | segurança |

**Diagramas:** [cenário 1](../diagramas/cenario-1-contexto.png); [dados, análise sob demanda](../diagramas/dados-analise.png).

## 8. Consequências

### 8.1 Benefícios esperados

- O registro das análises só contém o que o gestor confirmou.
- A memória da plataforma oferece expiração natural dos rascunhos.
- O agente não tem permissão para confirmar o contexto nem alterar o registro das análises.

### 8.2 Custos e consequências negativas assumidas

- Dependência de item previsto (set-26).
- Perda de rascunho obriga a refazer a conversa.

### 8.3 Outras consequências e obrigações

- A integração entre Copilot Studio e LangChain precisa ser demonstrada (VAL-01).

## 9. Riscos, controles, reversibilidade e exceções

| ID | Risco / modo de falha | Probabilidade / impacto | Mitigação e teste | Risco residual / quem aceita | Gatilho |
|---|---|---|---|---|---|
| AR-01 | Memória Redis indisponível | desconhecida / médio | ALT-C | componente novo / plataforma | VAL-09 falha |
| AR-02 | Copilot Studio não alcança o agente | desconhecida / médio | agente pelos Conectores de Canal | - | VAL-01 falha |
| AR-03 | dado pessoal digitado na conversa | média / médio | guardrail de PII no gateway | depende de DEP-04 | caso de teste |

**Raio de impacto:** só o cenário 1.
**Reversibilidade:** alta, pois basta trocar a ferramenta de rascunho.
**Condições de revogação:** falha em VAL-09 sem previsão de disponibilidade da memória.

## 10. Plano de validação e critérios de comprovação

| ID | Hipótese decisiva | Experimento / verificação | Critério de sucesso | Evidência | Estado / prazo |
|---|---|---|---|---|---|
| TST-01 | Memória Redis utilizável pelo agente | prova de conceito | rascunhos gravados e expirados | chaves e expiração | pendente / Entrega 1 |
| TST-02 | Copilot Studio alcança o agente | prova de conceito | conversa ponta a ponta | traces | pendente / Entrega 1 |
| TST-03 | agente sem acesso ao registro | teste de identidade | acesso negado ao PostgreSQL e à API | relatório | pendente / Entrega 1 |

**Confirmação contínua:** um teste de identidade verifica se o agente recebeu permissão no PostgreSQL das análises ou escopo na API e falha se houver acesso. Um teste de schema detecta tabelas de rascunho nesse banco.

## 11. Migração, entrega, rollback e execução

**Impacto no legado e compatibilidade:** N/A, sistema novo.

| Etapa | Entrega | Dependência | Evidência de conclusão | Responsável / prazo |
|---|---|---|---|---|
| IMP-01 | agente em LangChain com a Memória Redis | DEP-02, PRE-07 | TST-01 a TST-03 | plataforma e arquiteto / Entrega 1 |

## 12. Responsabilidades, consultas e decisão formal

| Participante | Papel no processo | Posição / parecer | Data / link |
|---|---|---|---|
| Rogério Barbosa | autor / recomendação | ALT-A com ALT-C de contingência | 2026-10-31 |
| responsável pelo desenho | revisão | dois bancos separados; memória Redis da plataforma | revisão do desenho |
| plataforma Einstein+ | consultado | pendente | - |
| arquitetura do Einstein | decisor com alçada | pendente | - |

## 13. Dependências, rastreabilidade e revisão

**Depende de:** DEP-02, PRE-07.
**Contradiz ou substitui:** substitui o rascunho no PostgreSQL e o agente no Foundry da revisão 6.
**DDA afetado:** §6, §8, §10, §11, §13, versão 0.1.
**Requisitos e riscos afetados:** RF-01; RIS-03, RIS-04.
**Revisão planejada:** Entrega 1.
**Gatilhos para novo RDA:** memória de longo prazo; troca de runtime de agentes.

## 14. Checklist de qualidade

- [x] Uma única decisão significativa, expressa objetivamente.
- [x] Contexto, problema, urgência, alternativas e status quo descritos.
- [x] Critérios eliminatórios separados das preferências.
- [x] Métricas, premissas, trade-offs e fontes explícitos.
- [x] Benefícios e consequências negativas assumidos sem omissão relevante.
- [x] Comparação que não mascara incerteza com pontuação arbitrária.
- [ ] Responsáveis, alçada, estado e data de efetividade (decisor pendente).
- [x] Limites, riscos, migração, rollback e reversibilidade descritos.
- [x] Plano de verificação e links de evidência.
- [x] DDA e registros relacionados atualizados.
- [x] Política de reavaliação e de substituição histórica.

## 15. Referências

- Material do desafio: `desafio_arquitetura_compliance.pptx`, slide 4.
- Proposta, revisão 7: `arquitetura_7.md`, Q4, §3.3 e §7.4.
- Microsoft, Foundry Agent Service standard agent setup: https://learn.microsoft.com/azure/foundry/agents/concepts/standard-agent-setup
