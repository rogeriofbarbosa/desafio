---
status: proposta
date: 2026-11-02
decision-makers: arquitetura do Einstein (alçada a confirmar), sobre proposta de Rogério Barbosa
consulted: Compliance e dono da base do Einstein (pendente)
informed: banca do desafio de arquitetura
---
# RDA-0001: Enumerar os requisitos na base do Einstein para provar a completude da análise

**Finalidade:** explicar como verificar que a análise trata todos os requisitos aplicáveis, quais alternativas foram consideradas e quanto essa verificação custa.

**Identificador:** RDA-0001
**Sistema / domínio:** Agente de Completude de Normativos e Cadeia de Valor
**Estado:** proposta
**Autores:** Rogério Barbosa (arquiteto, autor da proposta)
**Responsável pela decisão:** arquitetura do Einstein (pendente)
**Data da proposta / da decisão:** 2026-10-31 / pendente
**DDA vinculado:** [`../dda/01-agente-de-completude.md`](../dda/01-agente-de-completude.md), §6 (DEC-01), §8 (CMP-04), §10 (DATA-02, DATA-11), §11 (API-06), versão 0.1
**Projetos afetados:** executor de completude, executor de consolidação, PostgreSQL

## 0. Controle e governança do registro

| Campo | Valor |
|---|---|
| ID e título | RDA-0001: Enumerar os requisitos na base do Einstein para provar a completude da análise |
| Categoria | Dados / IA |
| Abrangência | camada determinística da análise; todos os referenciais |
| Estado e validade | proposta; 2026-11-02 |
| Decisor com alçada | arquitetura do Einstein (pendente) |
| Autores / consultados / discordâncias | Rogério Barbosa / Compliance e dono da base (pendente) / nenhuma registrada |
| DDA, especificação, requisito, incidentes associados | DDA §6 DEC-01; [especificação](../../especificacao/pendentes/agente-de-completude-2026-11.md) §3.2; RF-02, RF-03, RNF-10 |
| Registros relacionados | é percorrido pelo orquestrador de [RDA-0002](0002-fila-na-entrada-e-orquestrador-duravel.md) |
| Repositório e revisão | `arquitetura_7.md`, revisão 7, ADR-01 |
| Data de reavaliação ou gatilho | resposta a INC-01 (listagem na base) e a INC-02 (aplicabilidade) |

### 0.1 Histórico de estados

| Data | Estado anterior e novo | Quem autorizou | Justificativa / referência |
|---|---|---|---|
| 2026-10-31 | rascunho para proposta | autor | revisão 7 da proposta; o catálogo próprio no PostgreSQL foi retirado na revisão do desenho |
| 2026-11-02 | proposta registrada como RDA | autor | este arquivo |

## 1. Declaração objetiva da decisão

**Questão a decidir:** como verificar que todo requisito aplicável do referencial recebeu tratamento? As opções são enumerar os requisitos na base do Einstein, manter um catálogo próprio, buscar por similaridade ou deixar um agente definir a pesquisa.

**Decisão proposta:** listar na base do Einstein todos os requisitos do referencial vigentes na data de corte e avaliar cada um. O PostgreSQL registra apenas o universo da análise. A prova de completude depende de a base fornecer a listagem completa.

## 2. Contexto, problema e forças em conflito

**Situação atual (AS-IS):** o Compliance revisa os normativos manualmente, e o sistema ainda não existe. O desafio informa que a JCI e a legislação vigente já estão na base do Einstein (slides 2 e 3).
**Problema ou risco que motiva a escolha:** a busca vetorial recupera os trechos mais semelhantes, mas pode deixar obrigações de fora sem registrar a omissão. Um agente que escolhe o que pesquisar e quando parar também não permite comprovar a cobertura.
**Por que decidir agora:** a definição da camada determinística orienta as demais decisões.
**Forças em conflito:** enumerar os requisitos permite comprovar a completude, mas exige uma fonte externa capaz de listá-los. Um catálogo próprio permitiria enriquecer os requisitos, ao custo de manter uma segunda fonte. A avaliação por requisito também tem custo diferente do de uma pesquisa livre.
**Linha de base:** as métricas do processo manual serão levantadas no piloto (VAL-08 do DDA).
**Premissas declaradas:** a base permite listar todos os requisitos de um referencial vigentes em uma data, com identificador, versão e início de vigência (PRE-01). Até a definição do critério de aplicabilidade, todos se aplicam, e qualquer exclusão exige motivo (PRE-02).
**Fora do escopo:** garantir que o modelo interprete corretamente cada requisito ou definir o critério de aplicabilidade (INC-02).

## 3. Fatores de decisão, requisitos, restrições e critérios

### 3.1 Critérios verificáveis

| ID | Critério | Natureza | Regra ou métrica de aceite | Importância | Evidência / fonte |
|---|---|---|---|---|---|
| CR-01 | provar que todo requisito aplicável recebeu estado | eliminatório | listados = aplicáveis + excluídos com motivo; aplicáveis = soma das classificações A | obrigatório | RNF-10 |
| CR-02 | rastrear cada conclusão até a versão do requisito | eliminatório | todo item A cita requisito e versão | obrigatório | PRI-03 |
| CR-03 | não manter cópia da fonte regulatória | eliminatório | nenhum texto de requisito persistido pela solução | obrigatório | RES-02, decisão do responsável na revisão do desenho |
| CR-04 | custo e latência limitados | preferência | orçamento por requisito; duração da etapa A dentro do prazo | alta | DDA §17 |
| CR-05 | depender pouco de premissas não verificadas | preferência | premissas com prova marcada | média | DDA §16 |

**Restrições não negociáveis:** RES-02 (JCI e legislação vêm da base); RES-03 (inferência pelo LLM Gateway).
**Requisitos afetados:** RF-02, RF-03, RNF-10, RNF-11.
**Condições de análise:** o piloto ocorre em laboratório clínico, com referenciais configurados pelo Compliance. A quantidade N de requisitos por análise só será conhecida após a consulta à base.

### 3.2 Regra de priorização

Primeiro se aplicam os critérios eliminatórios. As preferências não recebem pesos numéricos.

## 4. Alternativas consideradas

| Alternativa | Descrição / mecanismo | Atende as restrições? | Benefícios | Limitações e riscos | Evidência disponível |
|---|---|---|---|---|---|
| ALT-A | listar na base do Einstein a cada análise; universo registrado com versão | sim, se PRE-01 se confirmar | prova de completude; fonte única; nada a sincronizar | depende da listagem e do tempo de leitura da base | premissa PRE-01; prova em VAL-02 |
| ALT-B | catálogo próprio de requisitos no PostgreSQL, curado e sincronizado | não (CR-03) | permite enriquecer requisitos (critério de evidência, prioridade) | duplica uma base que já existe; exige sincronização e dono; "de onde vieram esses dados?" | - |
| ALT-C | RAG top-k: trazer os trechos mais semelhantes e deixar o modelo concluir | não (CR-01) | simples e barato | requisito não recuperado some sem rastro | - |
| ALT-D | supervisor baseado em modelo decide o que pesquisar e quando parar | não (CR-01) | explora perguntas abertas | sem prova de cobertura; custo imprevisível | - |
| ALT-E | status quo: revisão manual | - | sem risco de IA | não escala aos cerca de 700 responsáveis | material do desafio |

**Descartadas antes da comparação:** nenhuma.
**Incertezas não resolvidas:** PRE-01 (listagem completa) e INC-02 (aplicabilidade).
**Dependência de fornecedor e reversibilidade:** a solução depende da capacidade de listagem oferecida pelo dono da base. Se ela não existir, será necessário solicitá-la. A última alternativa é um catálogo enumerável mantido pelo Compliance, o que exige reabrir este RDA.

## 5. Comparação das alternativas

| Critério | ALT-A | ALT-B | ALT-C | ALT-D | Fundamentação e grau de confiança |
|---|---|---|---|---|---|
| CR-01 (eliminatório) | atende | atende | falha | falha | por construção; alto |
| CR-02 (eliminatório) | atende | atende | atende | incerto | por construção; médio |
| CR-03 (eliminatório) | atende | falha | atende | atende | por construção; alto |
| CR-04 | similar | similar | melhor | pior | julgamento arquitetural; médio |
| CR-05 | pior (depende de PRE-01) | melhor | melhor | melhor | premissa não verificada; alto |
| Reversibilidade | média | média | alta | alta | troca da fonte do universo |

**Análise crítica:** apenas ALT-A e ALT-B atendem a CR-01, mas ALT-B viola CR-03. Por isso, ALT-A é a opção selecionada, com o risco concentrado em PRE-01.
**Teste de sensibilidade:** se a base não listar (PRE-01 falsa), ALT-A deixa de atender CR-01 e ALT-B passa a ser a única que atende, com custo de sincronização.
**Validade da comparação:** 2026-10-31, desenho da revisão 7.

## 6. Decisão e justificativa

**Opção selecionada:** ALT-A.
**Formulação normativa:** cada análise deve enumerar os requisitos na base do Einstein e registrar o tratamento dado a cada um. A capacidade de listagem completa da base é condição para comprovar a completude.
**Justificativa principal:** única alternativa que atende CR-01, CR-02 e CR-03 ao mesmo tempo.
**Alternativas descartadas:** ALT-B viola CR-03, enquanto ALT-C e ALT-D violam CR-01. ALT-E não atende à escala esperada.
**Condições de aceitação:** VAL-02 (listagem completa de um referencial do piloto, igual à referência oficial do recorte).
**Data efetiva:** após o aceite e VAL-02.
**Limites:** não decide o critério de aplicabilidade nem a interpretação de cada requisito.

## 7. Como a decisão altera o desenho

| Aspecto | Mudança resultante | Artefato a alterar | Responsável |
|---|---|---|---|
| Componentes e fronteiras | executor de completude lista e lê requisitos na base | DDA §8 (CMP-04), cenário 2 | arquiteto |
| Dados e consistência | PostgreSQL guarda só o universo (requisito, versão, vigência, aplicável ou excluído com motivo) | DDA §10 (DATA-02, DATA-11) | arquiteto |
| Contratos e eventos | interface de leitura da base | DDA §11 (API-06), pendente | Einstein |
| Plataforma e operação | base é dependência crítica da etapa de preparação | DDA §14.1 | operação |
| Segurança e privacidade | sem mudança | - | - |

**Restrições de implementação:** o universo é gravado uma vez por análise. A consolidação verifica o invariante de CR-01 em cada execução.

## 8. Consequências

### 8.1 Benefícios esperados

- A contagem dos requisitos permite verificar a completude de cada análise (RNF-10).
- A base do Einstein permanece como fonte única da JCI e da legislação, sem cópia de texto nem sincronização de requisitos na solução.

### 8.2 Custos e consequências negativas assumidas

- Se a base estiver indisponível durante a preparação, a análise termina em FAILED.
- Uma leitura na base por requisito (t_b na conta de duração do DDA §17).
- Enriquecer requisitos com critérios de evidência exigiria um catálogo próprio e uma nova decisão de arquitetura.

### 8.3 Outras consequências e obrigações

- Obrigação de definir o critério de aplicabilidade antes da expansão (INC-02).
- A regra de "atualizar" (DDA §9) passa a depender do início de vigência informado pela base.

## 9. Riscos, controles, reversibilidade e exceções

| ID | Risco / modo de falha | Probabilidade / impacto | Mitigação e teste | Risco residual / quem aceita | Gatilho |
|---|---|---|---|---|---|
| AR-01 | a base não lista todos os requisitos de um referencial na data | desconhecida / alto | VAL-02 antes da Entrega 3 | declarar o alcance da análise / Compliance | VAL-02 falha |
| AR-02 | critério de aplicabilidade indefinido infla o denominador | alta / médio | PRE-02; exclusão só com motivo; revisão humana | recomendações sem pertinência / Compliance | expansão a outras áreas |
| AR-03 | base lenta ou indisponível | desconhecida / médio | prazo da etapa; novas tentativas; FAILED explicável | análise refeita / operação | latência t_b acima de 1 s |

**Raio de impacto:** toda a análise, pois a etapa A depende do universo de requisitos.
**Reversibilidade:** a troca da fonte do universo afeta apenas a preparação (CMP-04).
**Condições de revogação:** VAL-02 falha sem perspectiva de a base ganhar a capacidade.

## 10. Plano de validação e critérios de comprovação

| ID | Hipótese decisiva | Experimento / verificação | Critério de sucesso | Evidência | Estado / prazo |
|---|---|---|---|---|---|
| TST-01 | a base lista todos os requisitos do referencial na data | listar um referencial do piloto e comparar com a referência oficial do recorte | listas iguais | relatório da comparação | pendente / Entrega 2 |
| TST-02 | o universo cobre a listagem | contagem em cada análise de teste | listados = aplicáveis + excluídos | consolidação | pendente / Entrega 3 |
| TST-03 | todo aplicável recebe classificação | contagem no resultado | aplicáveis = soma das classificações A | resultado | pendente / Entrega 3 |

**Confirmação contínua:** a consolidação só grava o resultado se o invariante de CR-01 for atendido. Um teste de schema verifica se o PostgreSQL contém alguma tabela com texto de requisito e, nesse caso, falha.

## 11. Migração, entrega, rollback e execução

**Impacto no legado e compatibilidade:** N/A, sistema novo.
**Critérios de go/no-go:** TST-01 aprovado.
**Rollback ou correção para a frente:** se TST-01 falhar, reabrir este RDA antes da Entrega 3.

| Etapa | Entrega | Dependência | Evidência de conclusão | Responsável / prazo |
|---|---|---|---|---|
| IMP-01 | acesso de leitura à base (API-06) | INC-01 | TST-01 | Einstein e arquiteto / Entrega 2 |
| IMP-02 | preparação do universo (CMP-04) | IMP-01 | TST-02 | arquiteto / Entrega 3 |

## 12. Responsabilidades, consultas e decisão formal

| Participante | Papel no processo | Posição / parecer | Data / link |
|---|---|---|---|
| Rogério Barbosa | autor / recomendação | ALT-A | 2026-10-31 |
| Compliance | consultado | pendente | - |
| dono da base do Einstein | consultado | pendente | - |
| arquitetura do Einstein | decisor com alçada | pendente | - |

**Dissenso registrado:** nenhum.
**Exceções concedidas:** nenhuma.

## 13. Dependências, rastreabilidade e revisão

**Depende de:** PRE-01, PRE-02.
**Contradiz ou substitui:** substitui o catálogo próprio das revisões 4 a 6 da proposta.
**DDA afetado:** §6, §8, §9, §10, §11, versão 0.1.
**Requisitos e riscos afetados:** RF-02, RF-03, RNF-10; RIS-01, RIS-02.
**Monitoramento da hipótese após a adoção:** contagens por análise; responsável: arquiteto.
**Revisão planejada:** após VAL-02.
**Gatilhos para novo RDA:** base sem listagem; necessidade de enriquecer requisitos.

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
- Proposta, revisão 7: `arquitetura_7.md`, §1.1 e ADR-01.
- Michael Nygard, "Documenting Architecture Decisions": https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions
- MADR 4.0.0: https://adr.github.io/madr/
