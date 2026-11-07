---
title: "Em spec, só comece um item de lista com um id de cláusula se for a definição dele"
author: William
date: 2026-11-05
branch: docs/store-v2
cost: { minutes: 11, tokens: 294367 }
id: l-b33411a1
scope: repo
status: active
src: agent
touches:
  - README.md
  - docs/operations.md
  - docs/roadmap.md
---

## Decided
Ao citar uma cláusula existente num item de lista novo (notas de estado, pendências), o id não pode abrir o item: o checker trata id no início de item/parágrafo como **definição**, e uma segunda definição derruba o CI. Cite no meio da frase ("Motivo da recusa de `[ST2-2]`: ...").

## Watch out
Id novo definido sem teste `// spec: <id>` vira WARN hoje (FAIL só para [CF-3]), mas é dívida: não crie ids novos em notas de pendência; descreva em prosa até haver teste.

In the code: `scripts/spec-ids` (rode com `sh scripts/spec-ids`; não é Python), regra em `docs/specs/conformance.md §1`.
