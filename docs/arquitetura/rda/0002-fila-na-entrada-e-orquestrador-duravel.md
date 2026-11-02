---
status: proposta
date: 2026-11-02
decision-makers: arquitetura do Einstein (alçada a confirmar), sobre proposta de Rogério Barbosa
consulted: plataforma Einstein+ (pendente)
informed: banca do desafio de arquitetura
---
# RDA-0002: Usar a fila só na entrada e um orquestrador durável que chama os executores

**Finalidade:** registrar como a solução trata análises demoradas: aceitação, execução, prazos, falhas e término.

**Identificador:** RDA-0002
**Sistema / domínio:** Agente de Completude de Normativos e Cadeia de Valor
**Estado:** proposta
**Autores:** Rogério Barbosa (arquiteto, autor da proposta)
**Responsável pela decisão:** arquitetura do Einstein (pendente)
**Data da proposta / da decisão:** 2026-10-31 / pendente
**DDA vinculado:** [`../dda/01-agente-de-completude.md`](../dda/01-agente-de-completude.md), §6 (DEC-02), §8 (CNT-01 a CNT-05), §9, §11, §12, versão 0.1
**Projetos afetados:** API de Completude, fila de comandos, orquestrador durável, executores de completude e de consolidação

## 0. Controle e governança do registro

| Campo | Valor |
|---|---|
| ID e título | RDA-0002: Usar a fila só na entrada e um orquestrador durável que chama os executores |
| Categoria | Integração / Plataforma |
| Abrangência | execução de toda análise |
| Estado e validade | proposta; 2026-11-02 |
| Decisor com alçada | arquitetura do Einstein (pendente) |
| Autores / consultados / discordâncias | Rogério Barbosa / plataforma Einstein+ (pendente) / nenhuma registrada |
| DDA, especificação, requisito, incidentes associados | DDA §6 DEC-02; [especificação](../../especificacao/pendentes/agente-de-completude-2026-11.md) §4.5 e §4.6; RF-07, RNF-01, RNF-06 |
| Registros relacionados | percorre o universo de [RDA-0001](0001-enumerar-requisitos-pela-base-do-einstein.md) |
| Repositório e revisão | `arquitetura_7.md`, revisão 7, ADR-02 |
| Data de reavaliação ou gatilho | passagem do piloto a produção (SKU do motor); entrada de um segundo comando (DEB-01) |

### 0.1 Histórico de estados

| Data | Estado anterior e novo | Quem autorizou | Justificativa / referência |
|---|---|---|---|
| 2026-10-31 | rascunho para proposta | autor | revisão 7; a fila de tarefas e a de resultados da revisão 6 foram retiradas na revisão do desenho |
| 2026-11-02 | proposta registrada como RDA | autor | este arquivo |

## 1. Declaração objetiva da decisão

**Questão a decidir:** como aceitar o pedido de imediato e conduzir uma análise que dura minutos, com fan-out por requisito, sem perder pedidos nem duplicar resultados? As alternativas incluem filas em todo o fluxo, orquestração durável com fila apenas na entrada, coordenador próprio, coreografia, motor de lote e processamento síncrono.

**Decisão proposta:** aceitar o pedido com HTTP 202, publicar um comando no Service Bus e conduzir a análise com um orquestrador durável que chama os executores por activities. O motor fornece prazos e fan-in, reduzindo os componentes necessários. Se ele parar, as análises em curso pausam até a retomada.

## 2. Contexto, problema e forças em conflito

**Situação atual (AS-IS):** N/A, sistema novo. O slide 4 prevê uma Fila (TBD) e um Front (TBD) na plataforma. O slide 5 pergunta como os endpoints tratam análises demoradas.
**Problema ou risco que motiva a escolha:** o turno do Copilot Studio e a requisição HTTP expiram antes de a análise terminar. A execução faz uma chamada por requisito e depende da base do Einstein, do RAG e do gateway, cada um com seu perfil de falha. Toda análise precisa chegar a um estado final com motivo registrado.
**Por que decidir agora:** define os contratos da API e a forma de toda a execução.
**Forças em conflito:** a independência dos executores em relação ao motor exige mais filas e mecanismos próprios. Usar despacho, novas tentativas, prazos e fan-in do motor reduz esse trabalho, mas torna a execução dependente de sua disponibilidade e do custo da SKU.
**Linha de base:** não há medições. As latências são premissas (DDA §17).
**Premissas declaradas:** a proposta usa Service Bus Premium e Durable Task Scheduler com SDK Python (PRE-10). Segundo a documentação oficial conferida em 31/10/2026, a SKU Consumption não tem alta disponibilidade e retém o histórico por até 30 dias.
**Fora do escopo:** cancelamento, reanálise e escolha da SKU de produção.

## 3. Fatores de decisão, requisitos, restrições e critérios

### 3.1 Critérios verificáveis

| ID | Critério | Natureza | Regra ou métrica de aceite | Importância | Evidência / fonte |
|---|---|---|---|---|---|
| CR-01 | aceitar o pedido sem esperar a análise | eliminatório | 202 com p95 ≤ 2 s, com análises em execução | obrigatório | RNF-01 |
| CR-02 | nenhum pedido perdido com o motor parado | eliminatório | pedidos aceitos esperam e executam depois | obrigatório | QS-01 |
| CR-03 | nenhuma análise nem resultado duplicado em reentrega | eliminatório | uma instância e um resultado por análise | obrigatório | QS-02 |
| CR-04 | término sempre explicável | eliminatório | prazo por etapa; SUCCEEDED, PARTIALLY_SUCCEEDED ou FAILED | obrigatório | DDA §9 |
| CR-05 | menos peças e menos código a escrever | preferência | número de filas e de mecanismos próprios | alta | julgamento arquitetural |
| CR-06 | executores trabalhando com o motor parado | preferência | trabalho em curso continua | média | - |
| CR-07 | ocupar a Fila TBD da plataforma | preferência | Service Bus como Fila da plataforma | média | slide 4 |

**Restrições não negociáveis:** RES-01 (reaproveitar a plataforma); RES-05 (sem código).
**Requisitos afetados:** RF-07, RNF-01, RNF-06.
**Condições de análise:** o piloto prevê poucas análises simultâneas. A quantidade N de requisitos por análise ainda é desconhecida.

### 3.2 Regra de priorização

Aplicam-se primeiro os critérios eliminatórios. Entre as alternativas aprovadas, CR-05 desempata.

## 4. Alternativas consideradas

| Alternativa | Descrição / mecanismo | Atende as restrições? | Benefícios | Limitações e riscos | Evidência disponível |
|---|---|---|---|---|---|
| ALT-A | fila só na entrada; orquestrador durável chama os executores por activities | sim | fan-in, prazos e novas tentativas nativos; poucas peças | análises em curso pausam com o motor parado | documentação do Durable Task |
| ALT-B | fila em todo o caminho: comandos, tarefas por executor e resultados | sim | executores continuam com o motor parado; escala pela fila | mais filas, mensagens de resultado, chaves de tarefa e reconciliação para reproduzir o que o motor já faz | revisão 6 da proposta |
| ALT-C | coordenador próprio sobre a fila e o PostgreSQL, com mensagens agendadas | sim | sem motor | escrever e testar estado, prazos e fan-in | - |
| ALT-D | coreografia pura, sem orquestrador | não (CR-04) | desacopla mais | não há lugar para prazos e prova de cobertura | - |
| ALT-E | Batch do Workflow AI (Haystack), previsto para out-26 | pendente | reuso | o slide não descreve fan-in com retomada | slide 4 |
| ALT-F | Durable Functions ou Temporal | sim | maduros | outro host ou outra plataforma | - |
| ALT-G | status quo: processamento síncrono na API | não (CR-01) | simples | o turno expira | - |

**Descartadas antes da comparação:** ALT-D e ALT-G, por critério eliminatório.
**Incertezas não resolvidas:** capacidades do Batch do Haystack.
**Dependência de fornecedor e reversibilidade:** o Service Bus e o Durable Task Scheduler vinculam a implementação à Azure. A adoção de ALT-F preserva a estrutura da solução.

## 5. Comparação das alternativas

| Critério | ALT-A | ALT-B | ALT-C | ALT-F | Fundamentação e grau de confiança |
|---|---|---|---|---|---|
| CR-01 (eliminatório) | atende | atende | atende | atende | 202 antes da execução; alto |
| CR-02 (eliminatório) | atende | atende | atende | atende | fila guarda pedidos; alto |
| CR-03 (eliminatório) | atende | atende | atende | atende | instância por análise, duplicatas, chave única; alto |
| CR-04 (eliminatório) | atende | atende | atende | atende | timers e fan-in; alto |
| CR-05 | melhor | pior | pior | similar | número de peças; médio |
| CR-06 | pior | melhor | similar | pior | por construção; alto |
| CR-07 | atende | atende | atende | atende | - |
| Reversibilidade | média | média | baixa | média | - |

**Análise crítica:** ALT-A e ALT-B passam nos critérios eliminatórios. ALT-B ganha em CR-06, mas perde em CR-05. No piloto, a pausa durante uma falha do motor gerenciado é aceitável, e o esforço para reproduzir despacho, novas tentativas e fan-in fora dele não se justifica.
**Teste de sensibilidade:** se o motor parar com frequência ou a SKU de produção não tiver alta disponibilidade, ALT-B volta a ser competitiva.
**Validade da comparação:** 2026-10-31.

## 6. Decisão e justificativa

**Opção selecionada:** ALT-A.
**Formulação normativa:** a fila de comandos deve ligar apenas a API ao orquestrador. O orquestrador durável chama os executores por activities e controla prazos, novas tentativas e fan-in. A execução depende da disponibilidade do motor.
**Justificativa principal:** atende CR-01 a CR-04 e vence CR-05.
**Alternativas descartadas:** ALT-B duplica funções do motor, e ALT-C exige implementá-las. ALT-D e ALT-G falham nos critérios eliminatórios. ALT-E não tem capacidade documentada para o fluxo, e ALT-F acrescenta um host ou uma plataforma sem ganho identificado.
**Condições de aceitação:** VAL-04 e VAL-05 do DDA.
**Data efetiva:** após o aceite.
**Limites:** não decide a SKU de produção nem o cancelamento.

## 7. Como a decisão altera o desenho

| Aspecto | Mudança resultante | Artefato a alterar | Responsável |
|---|---|---|---|
| Componentes e fronteiras | API, fila de comandos, orquestrador, dois executores sem estado | DDA §8 | arquiteto |
| Dados e consistência | gravação idempotente por análise e requisito; resultado único; slots com lease | DDA §10 | arquiteto |
| Contratos e eventos | `StartAnalysis` com `MessageId` igual ao id da análise; activities | DDA §11 | arquiteto |
| Plataforma e operação | Service Bus Premium; Durable Task Scheduler Consumption com Private Endpoint | DDA §12 a §14 | plataforma |
| Segurança e privacidade | mensagens e histórico só com IDs | DDA §13 | arquiteto |

**Diagrama:** [análise demorada](../diagramas/analise-demorada.png).
**Restrições de implementação:** toda chamada de I/O deve ocorrer em uma activity. Cada análise avalia até quatro requisitos em paralelo e tem prazo definido por etapa.

## 8. Consequências

### 8.1 Benefícios esperados

- O pedido é aceito em segundos, independentemente da duração da análise (RNF-01).
- Os pedidos ficam preservados quando o motor para. A execução retoma do histórico sem refazer requisitos já avaliados.
- Uma única fila atende à entrada, e o motor fornece despacho, novas tentativas, prazos e fan-in.

### 8.2 Custos e consequências negativas assumidas

- A indisponibilidade do motor pausa as análises em curso e aumenta a espera do gestor.
- O Durable Task Scheduler acrescenta um serviço que não consta do slide 4.
- A sessão por análise só terá função se houver um segundo comando, como o cancelamento (DEB-01). Ela garante um único receptor e a ordem dos comandos, mas o MVP tem apenas o comando de início.
- Sem outbox, a aceitação depende de o cliente repetir após 503 (DEB-02).

### 8.3 Outras consequências e obrigações

- A versão do orquestrador é fixada no início da instância. Como não há pausas humanas, a convivência com versões antigas dura minutos.
- Painel de análises aceitas sem instância.

## 9. Riscos, controles, reversibilidade e exceções

| ID | Risco / modo de falha | Probabilidade / impacto | Mitigação e teste | Risco residual / quem aceita | Gatilho |
|---|---|---|---|---|---|
| AR-01 | motor indisponível | baixa / médio | fila guarda pedidos; retomada; VAL-05 | pausa / plataforma, no piloto | indisponibilidade recorrente |
| AR-02 | activity executada duas vezes | média / baixo | chave única por análise e requisito; resultado único | - | - |
| AR-03 | publicação do comando falha | baixa / baixo | 503 e repetição idempotente; alerta | análise aceita parada até a repetição / operação | alerta disparado |
| AR-04 | executor nunca responde | baixa / médio | prazo por etapa; INCONCLUSIVE na consolidação | PARTIALLY_SUCCEEDED | prazo vencido recorrente |

**Raio de impacto:** todas as análises em curso.
**Reversibilidade:** migrar para ALT-B acrescenta filas e mensagens sem mudar os executores.
**Condições de revogação:** indisponibilidade do motor acima do aceitável em produção.

## 10. Plano de validação e critérios de comprovação

| ID | Hipótese decisiva | Experimento / verificação | Critério de sucesso | Evidência | Estado / prazo |
|---|---|---|---|---|---|
| TST-01 | aceitação rápida sob carga | teste de carga com análises em execução | p95 ≤ 2 s | relatório de carga | pendente / Entrega 3 |
| TST-02 | motor parado no meio da avaliação | falha injetada | nenhum pedido perdido; requisitos avaliados não refeitos | traces e contagens | pendente / Entrega 3 |
| TST-03 | publicação falha | falha injetada no Service Bus | 503; repetição gera uma instância | traces | pendente / Entrega 3 |
| TST-04 | activity repetida | repetição forçada | um item e um resultado | banco | pendente / Entrega 3 |

**Confirmação contínua:** a esteira executa testes de recuperação e verifica a ausência de I/O no código do orquestrador e de gravações não idempotentes. A inspeção do namespace deve confirmar que existe apenas uma fila de comandos.

## 11. Migração, entrega, rollback e execução

**Impacto no legado e compatibilidade:** N/A, sistema novo.
**Critérios de go/no-go:** TST-01 a TST-04.

| Etapa | Entrega | Dependência | Evidência de conclusão | Responsável / prazo |
|---|---|---|---|---|
| IMP-01 | Service Bus, Scheduler e orquestrador mínimo | - | um comando inicia uma instância que chama um executor e termina | plataforma e arquiteto / Entrega 1 |
| IMP-02 | etapas completas | RDA-0001 | TST-01 a TST-04 | arquiteto / Entrega 3 |

## 12. Responsabilidades, consultas e decisão formal

| Participante | Papel no processo | Posição / parecer | Data / link |
|---|---|---|---|
| Rogério Barbosa | autor / recomendação | ALT-A | 2026-10-31 |
| responsável pelo desenho | revisão | retirou a fila de tarefas e definiu que o orquestrador chama os executores | revisão do desenho |
| plataforma Einstein+ | consultado | pendente | - |
| arquitetura do Einstein | decisor com alçada | pendente | - |

**Dissenso registrado:** nenhum.

## 13. Dependências, rastreabilidade e revisão

**Depende de:** PRE-10; RDA-0001 (universo a percorrer).
**Contradiz ou substitui:** substitui a "fila em todo o caminho" da revisão 6.
**DDA afetado:** §6, §8, §9, §11, §12, §14, versão 0.1.
**Requisitos e riscos afetados:** RF-07, RNF-01, RNF-06; RIS-07, DEB-01, DEB-02.
**Monitoramento da hipótese após a adoção:** idade da fila, análises aceitas sem instância, prazos vencidos.
**Revisão planejada:** antes de produção, com a escolha da SKU.
**Gatilhos para novo RDA:** segundo comando (cancelamento); indisponibilidade recorrente do motor.

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

- Material do desafio: `desafio_arquitetura_compliance.pptx`, slides 4 e 5.
- Proposta, revisão 7: `arquitetura_7.md`, §3.2, §5, §6.1 e ADR-02.
- Microsoft, Asynchronous Request-Reply pattern: https://learn.microsoft.com/en-us/azure/architecture/patterns/asynchronous-request-reply
- Microsoft, Durable Task Scheduler: https://learn.microsoft.com/azure/durable-task/scheduler/durable-task-scheduler
- Microsoft, Durable Task Scheduler billing (SKU Consumption sem alta disponibilidade; histórico até 30 dias): https://learn.microsoft.com/en-us/azure/durable-task/scheduler/durable-task-scheduler-billing
- Microsoft, Service Bus message sessions: https://learn.microsoft.com/en-us/azure/service-bus-messaging/message-sessions
- Microsoft, Service Bus duplicate detection: https://learn.microsoft.com/en-us/azure/service-bus-messaging/duplicate-detection
- Microsoft, Service Bus dead-letter queues: https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-dead-letter-queues
