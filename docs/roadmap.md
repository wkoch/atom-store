# Roadmap do atom-store

Ordem obrigatória. Legenda `[ ]`/`[x]`. Este repo é dado assinado; o código (CLI `atom`) vem do repo
`atom`. Itens marcados **(core)** dependem de trabalho lá (ver `atom/docs/roadmap.md`, v1.1).

## S0 — `store/v1` em operação
- [x] `v1/config.json` com `index_signature_keys` e `artifacts_base`; publisher `atom` em `publishers/`.
- [ ] GitHub Pages servindo `v1/`; `atom source add oficial …/v1/ --keys …` instala um pacote real.
- [ ] `store.yml`: o passo de `--check-only` hoje engole falhas (`|| true`) e instala o `atom` de `main`.
      Fixar `ATOM_REV` numa tag do core e falhar o job quando a verificação falha.
- [ ] Primeiro PR vindo do `release.yml` do `atom-extensions` mesclado e assinado (aceite do E0 de lá).

Aceite: perfil limpo instala `atom/code-starter` a partir de `v1/`.

## S1 — `store/v2` lado a lado (ADR-024)
Pré-requisitos **(core)**: CLI instala de source v2 (`ext install/search/info/update` via `store::tuf`);
`expires` real aceito; ferramenta de publicação v2 (`docs/store-v2.md §8`).

- [ ] Chaves `v2` geradas e guardadas conforme `docs/operations.md` (root 2 de 3 offline; `snapshot` e
      `timestamp` como secrets do CI).
- [ ] `v2/metadata/1.root.json` publicado; sha256 divulgado fora de banda.
- [ ] CI de PR: valida snippets e o delegado `publisher:<pub>` assinado pelo publisher (chave de
      `publishers/<pub>.toml`, prefixo `[ST2-5]`), além do que já valida em `v1/`.
- [ ] CI de merge: gera `v1/` **e** `v2/` a partir dos mesmos snippets (mesmos artefatos e sha256);
      assina `targets`/`snapshot`/`timestamp` com versões incrementadas e expiração real.
- [ ] Cron diário (`schedule`) que re-assina só `timestamp.json`, com alerta em falha.
- [ ] Teste no CI: `atom source add … --root <sha>` + `atom ext install` contra o `v2/` publicado;
      metadado expirado de propósito é recusado.
- [ ] `v1/` continua publicado (aviso no cliente) pelo período de `atom/docs/deprecation.md`.

Aceite: perfil limpo instala `atom/code-starter` de `v2/`; `timestamp` vencido → `tuf expired`;
`atom source migrate` de uma source `v1/` funciona sem reinstalar pacotes.

## S2 — Canal de releases do core (opcional neste repo)
- [ ] **(core)** canal `v2/` de releases (`updates.md`). Se ficar neste repo: targets `core/<canal>.json` sob
      papel próprio, root embutido no binário.
