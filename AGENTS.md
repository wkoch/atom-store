# AGENTS.md — atom-store

Este repo é **dado assinado**, não código. Specs: `docs/store.md` (`store/v1`, congelada) e
`docs/store-v2.md` (`store/v2`, TUF, core v1.1) — cópias de `atom/docs/specs/`; a fonte da verdade é o
repo `atom`. Ao mudar a spec lá, copie para cá no mesmo dia. Plano: `docs/roadmap.md`.

Regras:
- Nunca commitar chave privada. A da loja `v1` vive só no secret `STORE_PRIVKEY` (ou com o mantenedor offline).
  Em `v2`: `root` e `targets` offline; só `snapshot`/`timestamp` como secrets do CI.
- Nunca editar à mão `*.json.sig`, `catalog.json`, `v2/metadata/**` ou versões já publicadas. Versão publicada
  é imutável; retirada = `"yanked": true`.
- `v2`: versão de metadado só sobe; `expires` sempre RFC 3339 UTC real (nunca sentinela). O formato é o
  perfil TUF do Atom (`store-v2.md §1.1`); ferramentas TUF genéricas não servem.
- `v2`: a loja nunca recebe chave privada de publisher; o publisher manda o delegado `publisher:<pub>` já assinado.
- `v1/` e `v2/` saem dos mesmos snippets no mesmo CI (mesmos artefatos e sha256).
- Mudanças em `v1/index/**` (e no delegado `v2`) chegam por PR (gerado pelo `release.yml` do `atom-extensions`
  ou por publishers terceiros).
- Novo publisher = PR adicionando `publishers/<pub>.toml` com chave pública; revisão humana obrigatória.
- `v1/config.json` muda raramente; troca de chave da loja exige ação de todos os usuários (ver `docs/operations.md`).

<!-- trail:start -->
## Trail — repo context graph

This repo is indexed in `.trail/cache/`: small linked markdown nodes that explain each
system and carry exact file:line spans, kept in sync with the code through git.

For ANY task here — understanding how something works, finding where code lives,
or scoping a change — get context from the graph before grepping or opening
source files. Re-ask freely (it's cheap) and reuse literal identifiers you
already have (symbol, error string, file name) as the query. New to this repo?
Run `trail map` first — a token-budgeted orientation (dir clusters, hubs,
hotspots), no LLM, no key.

- Run `trail ask "<your question>" --source` → ranked nodes with the relevant
  code spans inlined (each hit's ≤8-line crux by default; `--full` for whole
  definitions when the crux isn't enough). Match the tool to the task shape:
  for understanding or editing, the top node IS the answer — cite its
  `covers:` file:line spans and edit straight from `--source`. For
  exhaustive tasks ("every occurrence / every caller of this pattern"), ranked
  results are top-N, not complete — run `trail grep "<literal>"` instead
  (exhaustive over indexed files, grouped by enclosing symbol), falling back
  to raw `grep -rn` only for unindexed files.
- `trail skeleton <file>` → every definition's signature + span, ~10× cheaper
  than reading the file; use it to skim an API surface.
- `trail callers <symbol>` gives precomputed, exact edges — who calls this.
  Add `--direction out` for what it calls, or `--depth N` to walk
  transitively for the full blast radius. For structural questions, skip
  ranking and use this directly.
- Or browse: `.trail/cache/INDEX.md` lists every node; follow the links.
- Monorepos and folders of multiple repos rank fairly across sub-projects —
  hits carry `[scope/]` labels naming which one they're from. Narrow with
  `trail ask "<task>" --in <scope>/` once you know where you're working.

If a returned span is truncated ("+N more lines"), open the file at that exact
range before finalizing. Only open source files when a node genuinely lacks a
needed detail, and then at the exact file:line the node points to — never
re-read whole files.

After big code changes, refresh the graph with `trail build` (deterministic,
no API key, $0).
<!-- trail:end -->
