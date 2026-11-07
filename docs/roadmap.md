# Roadmap do atom-store

Ordem obrigatória. Legenda `[ ]`/`[x]`. Este repo é dado assinado; o código (CLI `atom`) vem do repo
`atom`. Itens marcados **(core)** dependem de trabalho lá (ver `atom/docs/roadmap.md`, v1.1).

## S0 — `store/v1` em operação
- [x] `v1/config.json` com `index_signature_keys` e `artifacts_base`; publisher `atom` em `publishers/`.
- [x] `store.yml`: `--check-only` estrito (sem `|| true`); `atom` instalado de tag pinada
      (`ATOM_REV: v1.1`; nunca `main`).
- [ ] GitHub Pages servindo `v1/`; `atom source add oficial …/v1/ --keys …` instala um pacote real.
- [ ] Primeiro PR vindo do `release.yml` do `atom-extensions` mesclado e assinado (aceite do E0 de lá).

Aceite: perfil limpo instala `atom/code-starter` a partir de `v1/`.

## S1 — `store/v2` lado a lado (ADR-024)
Pré-requisitos **(core)** entregues na tag `v1.1`: CLI instala de source v2
(`ext install/search/info/update` via `store::tuf`); `expires` real aceito;
ferramenta de publicação v2 (`atom ext publish --v2`, `docs/store-v2.md §8`).

- [ ] Chaves `v2` geradas e guardadas conforme `docs/operations.md` (root 2 de 3 offline; `snapshot` e
      `timestamp` como secrets do CI).
- [ ] `v2/metadata/1.root.json` publicado; sha256 divulgado fora de banda.
- [x] CI de PR pronto (`validate-v2`): valida snippets e o delegado `publisher:<pub>` assinado pelo
      publisher (chave de `publishers/<pub>.toml`, prefixo `[ST2-5]`), além do que já valida em `v1/`.
      Em espera até a ferramenta v2 do core existir.
- [x] CI de merge pronto (`publish-v2`): gera `v1/` **e** `v2/` a partir dos mesmos snippets (mesmos
      artefatos e sha256); assina `targets`/`snapshot`/`timestamp` com versões incrementadas e
      expiração real. Em espera até a ferramenta v2 existir.
- [x] Cron diário pronto (`timestamp`): re-assina só `timestamp.json`, com alerta em falha
      (falha do cron notifica os watchers). Em espera até a ferramenta v2 existir.
- [x] Teste no CI pronto (`e2e-v2`): `atom source add … --root <sha>` + `atom ext install` contra o
      `v2/` publicado; metadado expirado de propósito é recusado; `source migrate` sem reinstalar.
      Em espera até a ferramenta v2 existir.
- [ ] `v1/` continua publicado (aviso no cliente) pelo período de `atom/docs/deprecation.md`.

Aceite: perfil limpo instala `atom/code-starter` de `v2/`; `timestamp` vencido → `tuf expired`;
`atom source migrate` de uma source `v1/` funciona sem reinstalar pacotes.

## S2 — Canal de releases do core (opcional neste repo)
- [ ] **(core)** canal `v2/` de releases (`updates.md`). Se ficar neste repo: targets `core/<canal>.json` sob
      papel próprio, root embutido no binário.
