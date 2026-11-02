# Arquitetura: DDA, RDAs e diagramas

**Data:** 02/11/2026
**Responsável:** Rogério Barbosa

## Conteúdo

| Caminho | Conteúdo |
|---|---|
| [`dda/01-agente-de-completude.md`](dda/01-agente-de-completude.md) | o Documento de Descrição da Arquitetura, utilizado modelo versão 2.0 (ISO/IEC/IEEE 42010, arc42 e C4 adaptados) |
| [`rda/`](rda/) | Registro de Decisão de Arquitetura (RDA/ADR) nos arquivos, `NNNN-titulo-curto.md`|
| `diagramas/*.png` | imagens |

## Convenções

- Uma nova decisão exige um novo RDA. Preserve o registro aceito e marque seu estado como
  `substituída por RDA-NNNN` quando houver substituição.
- Estados de RDA: `proposta`, `em avaliação`, `aceita`, `rejeitada`, `descontinuada`,
  `substituída por RDA-NNNN`.

## Decisões (RDAs)

| RDA | Decisão | Estado | Data |
|---|---|---|---|
| [RDA-0001](rda/0001-enumerar-requisitos-pela-base-do-einstein.md) | enumerar os requisitos na base do Einstein para provar a completude da análise | proposta | 2026-11-02 |
| [RDA-0002](rda/0002-fila-na-entrada-e-orquestrador-duravel.md) | usar a fila só na entrada e um orquestrador que chama os executores | proposta | 2026-11-02 |
| [RDA-0003](rda/0003-benchmark-por-carga-periodica.md) | carregar o benchmark de mercado periodicamente, fora da análise | proposta | 2026-11-02 |
| [RDA-0004](rda/0004-memoria-do-agente-separada-do-registro.md) | guardar a conversa do agente na Memória Redis da plataforma, separada do registro das análises | proposta | 2026-11-02 |
