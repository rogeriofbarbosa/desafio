---
status: proposta
date: 2026-11-02
decision-makers: arquitetura do Einstein (alçada a confirmar), sobre proposta de Rogério Barbosa
consulted: Compliance (pendente)
informed: banca do desafio de arquitetura
---
# RDA-0003: Carregar o benchmark de mercado periodicamente, fora da análise

**Finalidade:** explicar como as práticas de mercado da camada livre são carregadas e usadas, mantendo a coleta externa separada de cada análise.

**Identificador:** RDA-0003
**Sistema / domínio:** Agente de Completude de Normativos e Cadeia de Valor
**Estado:** proposta
**Autores:** Rogério Barbosa (arquiteto, autor da proposta)
**Responsável pela decisão:** arquitetura do Einstein (pendente)
**Data da proposta / da decisão:** 2026-10-31 / pendente
**DDA vinculado:** [`../dda/01-agente-de-completude.md`](../dda/01-agente-de-completude.md), §6 (DEC-03), §8 (CNT-09, CMP-07), §10 (DATA-09, DATA-10), §13 (PRIV-02), versão 0.1
**Projetos afetados:** Rotina B, executor de completude, RAG, Blob

## 0. Controle e governança do registro

| Campo | Valor |
|---|---|
| ID e título | RDA-0003: Carregar o benchmark de mercado periodicamente, fora da análise |
| Categoria | Dados / Segurança / IA |
| Abrangência | camada livre; itens C |
| Estado e validade | proposta; 2026-11-02 |
| Decisor com alçada | arquitetura do Einstein (pendente) |
| Autores / consultados / discordâncias | Rogério Barbosa / Compliance (pendente) / nenhuma registrada |
| DDA, especificação, requisito, incidentes associados | DDA §6 DEC-03; [especificação](../../especificacao/pendentes/agente-de-completude-2026-11.md) §3.6; RF-05, RF-09, RNF-05 |
| Registros relacionados | - |
| Repositório e revisão | `arquitetura_7.md`, revisão 7, ADR-03 |
| Data de reavaliação ou gatilho | pedido do Compliance por curadoria prévia; utilidade dos itens C medida no piloto |

### 0.1 Histórico de estados

| Data | Estado anterior e novo | Quem autorizou | Justificativa / referência |
|---|---|---|---|
| 2026-10-31 | rascunho para proposta | autor | revisão 7; a pesquisa online por análise e a curadoria no PostgreSQL foram retiradas na revisão do desenho |
| 2026-11-02 | proposta registrada como RDA | autor | este arquivo |

## 1. Declaração objetiva da decisão

**Questão a decidir:** quando e como as práticas de mercado entram na análise? As opções são pesquisar a cada análise, realizar cargas periódicas com ou sem aprovação prévia, ou dispensar a camada livre.

**Decisão proposta:** coletar periodicamente as práticas por tema, a partir das fontes da allowlist, e indexá-las no RAG com fonte e data. A análise compara as práticas já indexadas. Assim, a coleta tem custo por carga e não exige acesso à internet durante a análise, embora as práticas possam ficar defasadas até a próxima carga.

## 2. Contexto, problema e forças em conflito

**Situação atual (AS-IS):** N/A, sistema novo. O desafio pede uma camada livre, que traz benchmark de mercado ainda fora da base do Einstein (slides 2 e 3).
**Problema ou risco que motiva a escolha:** a pesquisa externa a cada análise aumenta a latência e o consumo de tokens. Ela também expõe a execução a conteúdo não confiável e pode levar dados internos para a internet em consultas montadas a partir do contexto.
**Por que decidir agora:** define se a análise tem saída para a internet, o que muda a segurança e o custo.
**Forças em conflito:** consultas a cada análise oferecem práticas mais recentes, mas aumentam o custo e o risco. A curadoria prévia melhora o controle da qualidade, ao custo de um fluxo de aprovação.
**Linha de base:** nenhuma.
**Premissas declaradas:** o Compliance define a allowlist e os temas (PRE-11). O responsável pelo desenho considera que as práticas de mercado mudam com baixa frequência, mas essa hipótese ainda não foi medida.
**Fora do escopo:** promover uma prática de mercado a requisito. Essa decisão cabe ao dono da base do Einstein.

## 3. Fatores de decisão, requisitos, restrições e critérios

### 3.1 Critérios verificáveis

| ID | Critério | Natureza | Regra ou métrica de aceite | Importância | Evidência / fonte |
|---|---|---|---|---|---|
| CR-01 | nenhuma consulta externa com conteúdo interno | eliminatório | nenhum componente que lê normativos sai para a internet | obrigatório | PRI-06 |
| CR-02 | camada livre presente | eliminatório | itens C no resultado | obrigatório | slides 2 e 3 |
| CR-03 | procedência de toda prática | eliminatório | fonte, data da coleta e cópia em todo item C | obrigatório | PRI-03 |
| CR-04 | custo do benchmark independente do número de análises | preferência | custo por carga, não por análise | alta | DDA §17 |
| CR-05 | análise mais rápida | preferência | etapa C sem chamadas externas | alta | - |
| CR-06 | práticas atuais | preferência | defasagem máxima igual ao período de carga | média | - |

**Restrições não negociáveis:** RES-03 (inferência pelo gateway).
**Requisitos afetados:** RF-05, RF-09, RNF-05.
**Condições de análise:** MVP com um tema e uma allowlist reduzida.

### 3.2 Regra de priorização

Aplicam-se primeiro os critérios eliminatórios. CR-04 e CR-05 têm prioridade sobre CR-06.

## 4. Alternativas consideradas

| Alternativa | Descrição / mecanismo | Atende as restrições? | Benefícios | Limitações e riscos | Evidência disponível |
|---|---|---|---|---|---|
| ALT-A | Rotina B periódica, agendada no Airflow: coleta, triagem, cópia no Blob, indexação no RAG; a análise só compara | sim | análise sem internet; custo por carga; procedência por construção | práticas defasadas até a próxima carga; sem aprovação prévia | - |
| ALT-B | pesquisa externa a cada análise, por um executor com acesso à internet | sim, com consultas verificadas e aprovadas | mercado mais recente | análise mais lenta e cara; consultas a montar e verificar a partir do contexto; internet no caminho da análise | revisão 6 da proposta |
| ALT-C | ALT-A com curadoria das práticas antes de indexar | sim | mais controle de qualidade | exige fluxo de aprovação e registro próprio que o desafio não pede para a camada livre | - |
| ALT-D | status quo: sem camada livre | não (CR-02) | simples | não atende ao desafio | - |

**Descartadas antes da comparação:** ALT-D.
**Incertezas não resolvidas:** utilidade dos itens C para o Compliance (VAL-06).
**Dependência de fornecedor e reversibilidade:** não há nova dependência de fornecedor. ALT-C pode ser adotada depois sem alterar a análise.

## 5. Comparação das alternativas

| Critério | ALT-A | ALT-B | ALT-C | Fundamentação e grau de confiança |
|---|---|---|---|---|
| CR-01 (eliminatório) | atende | atende com controles | atende | por construção; alto para A e C, médio para B |
| CR-02 (eliminatório) | atende | atende | atende | - |
| CR-03 (eliminatório) | atende | atende | atende | registro da coleta |
| CR-04 | melhor | pior | similar a A | por construção; alto |
| CR-05 | melhor | pior | melhor | por construção; alto |
| CR-06 | pior | melhor | pior | período de carga |
| Reversibilidade | alta | média | alta | - |

**Análise crítica:** ALT-B oferece práticas mais recentes, mas aumenta a latência, o consumo de tokens e a superfície de ataque de cada análise. ALT-C acrescenta aprovação prévia a ALT-A e permanece como possibilidade de evolução.
**Teste de sensibilidade:** se o Compliance exigir aprovação antes do uso, ALT-C passa a ser a escolha, sem mudar a análise.
**Validade da comparação:** 2026-10-31.

## 6. Decisão e justificativa

**Opção selecionada:** ALT-A.
**Formulação normativa:** a Rotina B deve coletar e indexar as práticas por tema, restrita à allowlist. A análise usa o material indexado sem acesso à internet. No MVP, as práticas não exigem aprovação prévia e podem permanecer defasadas até a próxima carga.
**Justificativa principal:** atende aos eliminatórios por construção e vence CR-04 e CR-05.
**Alternativas descartadas:** ALT-B aumenta o custo e o risco de cada análise. ALT-C exige um fluxo de aprovação e um registro que o desafio não pede. ALT-D deixa de atender à camada livre exigida.
**Condições de aceitação:** VAL-07 (nenhuma chamada de análise alcança a internet).
**Data efetiva:** após o aceite.
**Limites:** não define os temas nem a allowlist (Compliance).

## 7. Como a decisão altera o desenho

| Aspecto | Mudança resultante | Artefato a alterar | Responsável |
|---|---|---|---|
| Componentes e fronteiras | Rotina B agendada no Airflow; nenhum executor com internet | DDA §8 (CNT-09) | arquiteto |
| Dados e consistência | práticas no RAG com fonte, data, tema e referência à cópia; cópias no Blob; nada no PostgreSQL | DDA §10 (DATA-09, DATA-10) | arquiteto |
| Contratos e eventos | etapa C compara práticas já indexadas | especificação §3.2 | arquiteto |
| Plataforma e operação | só a Rotina B sai para a internet, restrita à allowlist | DDA §12.1 | plataforma |
| Segurança e privacidade | nenhuma consulta externa carrega conteúdo de análise | DDA §13 (PRIV-02) | segurança |

**Diagramas:** [reuso, carga periódica](../diagramas/reuso-carga.png); [dados, carga periódica](../diagramas/dados-carga.png).

## 8. Consequências

### 8.1 Benefícios esperados

- Nenhuma chamada de análise alcança a internet.
- Custo do benchmark por carga, não por análise.
- Etapa C mais rápida e previsível.

### 8.2 Custos e consequências negativas assumidas

- Práticas defasadas até a próxima carga.
- Sem aprovação prévia, o controle de qualidade dos itens C depende da allowlist e da revisão humana.

### 8.3 Outras consequências e obrigações

- O Compliance mantém temas e allowlist.
- A Rotina B verifica a procedência do material e faz a triagem de tentativas de injeção de instruções.

## 9. Riscos, controles, reversibilidade e exceções

| ID | Risco / modo de falha | Probabilidade / impacto | Mitigação e teste | Risco residual / quem aceita | Gatilho |
|---|---|---|---|---|---|
| AR-01 | prática de baixa qualidade | média / baixo | allowlist, procedência, decisão humana | itens C descartados / Compliance | rejeição alta de itens C |
| AR-02 | instrução escondida em página externa | média / médio | triagem na Rotina B; comparação sem ferramentas e com schema fechado | - | caso adversarial |
| AR-03 | fonte indisponível na carga | média / baixo | carga seguinte; painel das rotinas | práticas mais antigas | carga atrasada |

**Raio de impacto:** apenas os itens C, que não alteram os itens A.
**Reversibilidade:** adotar ALT-C exige alterar a Rotina B. Voltar a ALT-B acrescenta um executor. As avaliações A e B permanecem iguais.
**Condições de revogação:** exigência de atualidade maior que o período de carga.

## 10. Plano de validação e critérios de comprovação

| ID | Hipótese decisiva | Experimento / verificação | Critério de sucesso | Evidência | Estado / prazo |
|---|---|---|---|---|---|
| TST-01 | análise sem internet | teste de rede e de identidade dos executores | nenhuma saída | relatório | pendente / Entrega 4 |
| TST-02 | procedência completa | inspeção dos itens C | 100% com fonte, data e cópia | resultado | pendente / Entrega 4 |
| TST-03 | utilidade | julgamento do Compliance no piloto | itens C aceitos como oportunidade | decisões | pendente / Entrega 4 |

**Confirmação contínua:** a política de rede permite saída para a internet apenas à Rotina B. Um teste de contrato rejeita itens C sem fonte, data ou cópia.

## 11. Migração, entrega, rollback e execução

**Impacto no legado e compatibilidade:** N/A, sistema novo.

| Etapa | Entrega | Dependência | Evidência de conclusão | Responsável / prazo |
|---|---|---|---|---|
| IMP-01 | Rotina B com um tema e allowlist reduzida | PRE-11 | práticas indexadas e cópias no Blob | arquiteto / Entrega 4 |
| IMP-02 | etapa C da análise | IMP-01 | TST-01 a TST-03 | arquiteto / Entrega 4 |

## 12. Responsabilidades, consultas e decisão formal

| Participante | Papel no processo | Posição / parecer | Data / link |
|---|---|---|---|
| Rogério Barbosa | autor / recomendação | ALT-A | 2026-10-31 |
| responsável pelo desenho | revisão | definiu o benchmark como rotina de carga periódica; Rotina B não grava no PostgreSQL; sem curadoria prévia | revisão do desenho |
| Compliance | consultado | pendente | - |
| arquitetura do Einstein | decisor com alçada | pendente | - |

## 13. Dependências, rastreabilidade e revisão

**Depende de:** PRE-11.
**Contradiz ou substitui:** substitui o executor de pesquisa externa e a aprovação de consultas da revisão 6.
**DDA afetado:** §6, §8, §10, §12, §13, versão 0.1.
**Requisitos e riscos afetados:** RF-05, RF-09, RNF-05; RIS-09.
**Monitoramento da hipótese após a adoção:** taxa de aceitação dos itens C; idade da última carga.
**Revisão planejada:** fim do piloto.
**Gatilhos para novo RDA:** curadoria prévia exigida; necessidade de atualidade.

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

- Material do desafio: `desafio_arquitetura_compliance.pptx`, slides 2 e 3.
- Proposta, revisão 7: `arquitetura_7.md`, §4.2 e ADR-03.
- OWASP, Top 10 for LLM Applications: https://owasp.org/www-project-top-10-for-large-language-model-applications/
