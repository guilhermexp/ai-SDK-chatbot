# Testes

## Playwright

- Instale navegadores: `pnpm exec playwright install`
- Execute testes: `pnpm test`

## Cobertura

- Testes E2E validam:
  - Autenticação (Clerk) via header
  - Fluxo de chat e streaming
  - Histórico e visibilidade
  - Upload de arquivos

## Dicas

- Garanta `POSTGRES_URL` para testes que persistem dados
- Ajuste timeouts para cenários de streaming quando necessário