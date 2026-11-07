# atom-store (loja de referência `store/v1` + `store/v2`)

Loja de pacotes servida como **arquivos estáticos**. Specs (cópias do repo `atom`, que é a fonte da
verdade): `docs/store.md` (`store/v1`, congelada) e `docs/store-v2.md` (`store/v2`, TUF, core v1.1).
Operação (chaves, publishers, cron, incidentes): `docs/operations.md`. Plano: `docs/roadmap.md`.
Regras: `AGENTS.md`.

Estado: `v1/` em operação parcial (config + publisher `atom`, sem pacotes ainda; CI estrito com
`ATOM_REV: v1.0.0` pinada, sem `|| true`). `v2/` ainda não existe: depende da ferramenta de publicação
v2 do core (`docs/store-v2.md §8`). Os jobs v2 do `store.yml` (`validate-v2`, `publish-v2`, `timestamp`,
`e2e-v2`) estão prontos e sondam `atom ext publish --v2`; sem a ferramenta, emitem notice e passam sem
publicar. Durante a transição a loja publica `v1/` e `v2/` lado a lado, a partir dos mesmos snippets.

```
atom-store/
  README.md
  .github/workflows/store.yml        # CI v1: valida índices + regenera catalog + assina
  publishers/<pub>.toml              # chaves públicas aprovadas (`key = "ed25519:…"`); v2: chave do papel publisher:<pub>
  v1/config.json                     # nome, index_signature_keys, artifacts_base
  v1/catalog.json(+.sig)             # índice leve gerado pelo publish/CI
  v1/index/<pub>/<nome>.json(+.sig)  # índice esparso por pacote
  v1/artifacts/<pub>/<arq>.tar.gz    # artefatos (ou Releases, ver abaixo)
  v2/metadata/                       # (planejado) TUF: N.root.json, timestamp, snapshot, targets, publisher:<pub>.json
  v2/targets/                        # (planejado) catalog.json + <pub>/<nome>/<versão>/<arq>.tar.gz
```

## Fluxo de publicação `store/v1`

Na máquina do publisher (com o CLI `atom` instalado):

```sh
# 1. gera par de chaves (uma vez)
atom ext keygen --out ~/.config/atom/keys/publisher.ed25519
#    → imprime `ed25519:BASE64` (pub); mande ao mantenedor da loja.

# 2. empacota (gera tarball + .sig + snippet do índice), uma vez por plataforma (`--platform`)
atom ext pack ./meu-pacote --key ~/.config/atom/keys/publisher.ed25519 --out-dir ./out

# 3. publica no checkout da loja (mescla índice, assina, regen catalog)
atom ext publish --store-dir ./v1 --store-key ~/.config/atom/keys/store.ed25519 ./out/*.index.json

# 4. GitHub: git add v1 publishers && git commit && git push
#    anexe os .tar.gz ao Release da versão e abra o PR.
#    Com --github o comando imprime esse lembrete.
```

A chave da loja (`store.ed25519`) fica com o mantenedor; a raiz é fixada
no cliente via `atom source add oficial <url>/v1/ --keys ed25519:…`.
Mudança de chave exige ação do usuário (o cliente recusa sig de chave nova).

## Fluxo de publicação `store/v2` (bloqueado no core — jobs prontos, em espera)

Bloqueio exato: falta no core a **ferramenta de publicação v2** (`docs/store-v2.md §8`):
lado publisher (a partir dos snippets `.index.json` do `atom ext pack`, gerar/atualizar
`publisher:<pub>.json` assinado com a chave do publisher, `version` incrementada) e lado loja
(aceitar o delegado assinado — assinatura contra `publishers/<pub>.toml` + prefixo `[ST2-5]` —,
regenerar `catalog.json`, assinar `targets`/`snapshot`/`timestamp` com `version` incrementada e
`expires` real). O `store::tuf::publish_repo` atual não serve: exige as seeds de todos os publishers,
usa a chave root também para `targets`, grava `version = 1` e `expires` sentinela (`§9`). Nenhum formato
próprio da loja: os jobs chamam estritamente `atom ext publish --v2 …` (com `--check-only`,
`--snapshot-key/--timestamp-key`, `--refresh-timestamp-only`, `--expire-timestamp-now` para o teste
de expiração) assim que o core os expuser.

Enquanto isso os jobs v2 emitem notice e passam sem publicar.

Quando a ferramenta existir, os mesmos snippets do passo 2 alimentam o `v2/`
(spec `docs/store-v2.md §2, §8`), além do que já se valida em `v1/`:

- O publisher assina o **próprio papel delegado** `publisher:<pub>` (índice + hashes dos artefatos)
  e manda o metadado assinado no PR. A loja nunca recebe a chave privada do publisher.
- O CI da loja verifica o delegado (chave de `publishers/<pub>.toml`, prefixo `<pub>/*` e
  `index/<pub>/*` — `[ST2-5]`), regenera `catalog.json` e assina `targets`, `snapshot` e `timestamp`
  com versões incrementadas e expiração real.
- Um cron diário re-assina só o `timestamp` (expira em 1 dia). Sem cron, clientes param de instalar
  (a falha do cron notifica os watchers do repo).
- Usuários: `atom source add oficial <url>/v2/ --root <sha256 do 1.root.json>` ou
  `atom source migrate oficial --root <sha256>`. O sha256 é publicado fora de banda (este README e as
  notas de release do core). `source migrate` não reinstala pacotes.

## Artefatos: no repo ou no Release

- Lojas pequenas/`file://`: commit dos `.tar.gz` em `v1/artifacts/` (o
  `publish` já copia para lá). No `v2/`, artefatos ficam em `targets/<pub>/<nome>/<versão>/`.
- GitHub com Releases: suba os `.tar.gz` como anexos do Release e edite o
  `url` do artefato no índice para a URL do anexo antes de assinar
  (`artifacts_base` em `config.json` cobre o prefixo comum). No `v2/` a URL absoluta é aceita, mas o
  hash continua vindo do papel delegado (`store-v2.md §1.1`).

## Verificação (o que o cliente checa)

`store/v1` (spec §4):

1. `.sig` do índice válida por uma `index_signature_keys` da loja;
2. `sha256` do download confere;
3. assinatura do artefato válida por uma `publisher_keys` do índice;
4. `atom.toml` interno confere com o índice (name, version, kind,
   capabilities, provides).

Adulterar 1 byte do artefato → instalação recusada (passo 2); trocar o
`sha256` no índice sem a chave do publisher → recusada (passo 3);
re-assinar o índice sem a chave da loja → recusada (passo 1).

`store/v1` não protege contra um mirror que serve para sempre um índice antigo validamente assinado;
o cliente v1.1 mostra aviso para sources v1. `store/v2` (spec §4, `[ST2-1..5]`) acrescenta
expiração, versões monotônicas, limiar de chaves e delegação por publisher.

## CI

`.github/workflows/store.yml`: job `validate` (PRs) valida índices v1 e roda
`atom ext publish --store-dir v1 --store-key /dev/null --check-only` em modo estrito (sem `|| true`);
o `atom` vem da tag pinada `ATOM_REV` (nunca `main`). No `main`, `resign` re-assina índices + `catalog`
via `atom ext publish --reindex` com o secret `STORE_PRIVKEY`. Jobs v2 (`validate-v2`, `publish-v2`,
`timestamp` diário, `e2e-v2`) estão prontos e sondam `atom ext publish --v2`; sem a ferramenta do core
(`docs/store-v2.md §8`), emitem notice e passam sem publicar. Detalhes em `docs/roadmap.md` e
`docs/operations.md`.
