# atom-store (loja de referência `store/v1` + `store/v2`)

Loja de pacotes servida como **arquivos estáticos**. Specs (cópias do repo `atom`, que é a fonte da
verdade): `docs/store.md` (`store/v1`, congelada) e `docs/store-v2.md` (`store/v2`, TUF, core v1.1).
Operação (chaves, publishers, cron, incidentes): `docs/operations.md`. Plano: `docs/roadmap.md`.
Regras: `AGENTS.md`.

Estado (2026-11-05): `v1/` em operação parcial (config + publisher `atom`, sem pacotes ainda).
`v2/` ainda não existe: depende da integração `store/v2` na CLI do core e da ferramenta de publicação
v2 (`docs/store-v2.md §8–9`). Durante a transição a loja publica `v1/` e `v2/` lado a lado, a partir
dos mesmos snippets.

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

## Fluxo de publicação `store/v2` (planejado)

Os mesmos snippets do passo 2 alimentam o `v2/`. Diferenças (spec `docs/store-v2.md §2, §8`):

- O publisher assina o **próprio papel delegado** `publisher:<pub>` (índice + hashes dos artefatos)
  e manda o metadado assinado no PR. A loja nunca recebe a chave privada do publisher.
- O CI da loja verifica o delegado (chave de `publishers/<pub>.toml`, prefixo `<pub>/*`), regenera
  `catalog.json` e assina `targets`, `snapshot` e `timestamp` com versões incrementadas.
- Um cron diário re-assina só o `timestamp` (expira em 1 dia). Sem cron, clientes param de instalar.
- Usuários: `atom source add oficial <url>/v2/ --root <sha256 do 1.root.json>` ou
  `atom source migrate oficial --root <sha256>`. O sha256 é publicado fora de banda (este README e as
  notas de release do core).

Os comandos exatos dependem da ferramenta de publicação v2 que o core ainda vai expor.

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

`.github/workflows/store.yml` (só `v1/` hoje): valida índices/catalog (PRs) e, no `main`
com o secret `STORE_PRIVKEY`, re-assina índices + regenera o `catalog`
via `atom ext publish --reindex`. Sem o secret, o job só verifica
(`--check-only`) e o mantenedor assina localmente. Os jobs `v2` (verificação de PR, assinatura no
merge, cron de `timestamp`) estão descritos em `docs/roadmap.md`.
