# Operação da loja

## Setup inicial `store/v1` (uma vez)
1. `atom ext keygen --out ~/.config/atom/keys/store.ed25519` → guarde offline; a pública (`ed25519:…`) é a raiz.
2. Copie `v1/config.json.example` → `v1/config.json`, preencha `index_signature_keys` e `artifacts_base`.
3. Assine: `atom ext publish --store-dir v1 --store-key ~/.config/atom/keys/store.ed25519 --reindex`.
4. Secret `STORE_PRIVKEY` = conteúdo do arquivo da chave (opcional; sem ele o CI só valida e você assina local).
5. GitHub Pages servindo a raiz do repo → base `https://wkoch.github.io/atom-store/v1/`.
6. Usuários: `atom source add oficial https://wkoch.github.io/atom-store/v1/ --keys ed25519:<raiz>`.
   O cliente v1.1 mostra aviso para sources v1; a source padrão embutida no binário deve ser a `v2/`.

## Setup inicial `store/v2` (quando a ferramenta v2 do core existir)
Spec: `docs/store-v2.md`. Formato TUF próprio do Atom (§1.1): não use `tuftool`/`python-tuf`.

| Papel | Chave | Onde fica | Expiração | Limiar |
|---|---|---|---|---|
| `root` | 3 chaves | offline, 3 mantenedores | 1 ano | 2 de 3 |
| `targets` | 1 chave | offline (mantenedor) | 90 dias | 1 |
| `snapshot` | 1 chave | secret `STORE_SNAPSHOT_KEY` | 7 dias | 1 |
| `timestamp` | 1 chave | secret `STORE_TIMESTAMP_KEY` | 1 dia | 1 |
| `publisher:<pub>` | do publisher | com o publisher (`atom`: secret `ATOM_PUBLISHER_KEY` no `atom-extensions`) | 90 dias | publisher escolhe |

1. Gere as chaves com `atom ext keygen` (uma por arquivo). Nunca reutilize a chave da raiz `v1` em `v2`.
2. Gere `v2/metadata/1.root.json` com os papéis acima (`consistent_snapshot` pode ficar `false`).
3. Publique o sha256 de `1.root.json` fora de banda (README, notas de release do core) e peça ao repo
   `atom` para embutir a source oficial `v2/` + esse sha256 no binário.
4. Usuários: `atom source add oficial https://wkoch.github.io/atom-store/v2/ --root <sha256>` ou
   `atom source migrate oficial --root <sha256>`.

## Rotina `store/v2`
- **Cron diário:** re-assina `timestamp.json` (nova versão, `expires` = agora + 1 dia). Alerta se o job
  falhar: em ~24h todos os clientes recusam a loja (`tuf expired`).
- **Snapshot:** re-assinado a cada merge e, no mínimo, semanalmente (expira em 7 dias).
- **Targets:** muda quando entra/sai um publisher ou muda o catálogo; renovar antes de 90 dias.
- **Root:** renovar antes de 1 ano gerando `N+1.root.json` assinado por 2 de 3 do root atual e do novo.
- Versões de metadado **só sobem**. Nunca republique uma versão menor ou igual com conteúdo diferente
  (o cliente guarda a maior versão vista por papel e recusa regressão).

## Publishers
`publishers/<pub>.toml`:
```toml
# uma chave por linha; o CLI (atom-core store.rs) lê TODA string entre aspas como chave,
# então não ponha outros campos com aspas aqui (contato vai no PR/README).
key = "ed25519:BASE64..."
```
O publisher `atom` é o `atom-extensions` (secret `ATOM_PUBLISHER_KEY` lá). Em `v2` a mesma chave pública
entra na delegação `publisher:<pub>` do `targets` (paths `<pub>/*` e `index/<pub>/*`).

## Artefatos
- Preferência: anexados a GitHub Releases do repo de origem (`artifacts_base` absoluto por URL no índice).
- Alternativa: `v1/artifacts/<pub>/` neste repo (simples, mas o repo cresce; evitar para binários).
- `v2/`: mesmo arquivo, mesmo sha256 que o `v1/`; o hash vale pelo papel delegado.

## Incidentes
- **Versão com bug:** `"yanked": true` na versão (PR), CI re-assina (`v1`) e o publisher re-assina o
  delegado com versão nova (`v2`). Clientes deixam de resolver para ela.
- **Chave de publisher vazada:** `v1`: remover de `publishers/` e do `publisher_keys` dos índices, yank das
  versões assinadas após o vazamento, nova chave, republicar. `v2`: trocar a chave da delegação no
  `targets` (nova versão), publisher re-assina o delegado com a chave nova.
- **Chave de `timestamp`/`snapshot` vazada:** gerar nova e publicar `N+1.root.json` com a troca
  (assinado por 2 de 3 do root). Clientes seguem a cadeia sozinhos.
- **Chave da loja vazada (`v1`):** nova raiz em `config.json`; todos os usuários precisam `atom source` com
  a nova chave (o cliente recusa troca silenciosa). Comunicar fora de banda. Motivo a mais para migrar
  para `v2`.
- **Root `v2` comprometido abaixo do limiar** (1 de 3): rotacionar root com as 2 restantes. Acima do
  limiar: novo `1.root.json` e novo sha256 fora de banda (equivalente a loja nova).
