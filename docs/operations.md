# Operação da loja

## Setup inicial (uma vez)
1. `atom ext keygen --out ~/.config/atom/keys/store.ed25519` → guarde offline; a pública (`ed25519:…`) é a raiz.
2. Copie `v1/config.json.example` → `v1/config.json`, preencha `index_signature_keys` e `artifacts_base`.
3. Assine: `atom ext publish --store-dir v1 --store-key ~/.config/atom/keys/store.ed25519 --reindex`.
4. Secret `STORE_PRIVKEY` = conteúdo do arquivo da chave (opcional; sem ele o CI só valida e você assina local).
5. GitHub Pages servindo a raiz do repo → base `https://wkoch.github.io/atom-store/v1/`.
6. Usuários: `atom source add oficial https://wkoch.github.io/atom-store/v1/ --keys ed25519:<raiz>`.
   Sugestão: embutir essa source como padrão no binário `atom` (repo core).

## Publishers
`publishers/<pub>.toml`:
```toml
# uma chave por linha; o CLI (atom-core store.rs) lê TODA string entre aspas como chave,
# então não ponha outros campos com aspas aqui (contato vai no PR/README).
key = "ed25519:BASE64..."
```
O publisher `atom` é o `atom-extensions` (secret `ATOM_PUBLISHER_KEY` lá).

## Artefatos
- Preferência: anexados a GitHub Releases do repo de origem (`artifacts_base` absoluto por URL no índice).
- Alternativa: `v1/artifacts/<pub>/` neste repo (simples, mas o repo cresce; evitar para binários).

## Incidentes
- **Versão com bug:** `"yanked": true` na versão (PR), CI re-assina. Clientes deixam de resolver para ela.
- **Chave de publisher vazada:** remover de `publishers/` e do `publisher_keys` dos índices, yank das versões
  assinadas após o vazamento, nova chave, republicar.
- **Chave da loja vazada:** nova raiz em `config.json`; todos os usuários precisam `atom source` com a nova chave
  (o cliente recusa troca silenciosa). Comunicar fora de banda.
