# AGENTS.md — atom-store

Este repo é **dado assinado**, não código. Spec: `docs/store.md` (cópia de `atom/docs/specs/store.md`,
congelada em `store/v1`; a fonte da verdade é o repo `atom`).

Regras:
- Nunca commitar chave privada. A da loja vive só no secret `STORE_PRIVKEY` (ou com o mantenedor offline).
- Nunca editar à mão `*.json.sig`, `catalog.json` ou versões já publicadas. Versão publicada é imutável;
  retirada = `"yanked": true`.
- Mudanças em `v1/index/**` chegam por PR (gerado pelo `release.yml` do `atom-extensions` ou por publishers terceiros).
- Novo publisher = PR adicionando `publishers/<pub>.toml` com chave pública; revisão humana obrigatória.
- `v1/config.json` muda raramente; troca de chave da loja exige ação de todos os usuários (ver `docs/operations.md`).
