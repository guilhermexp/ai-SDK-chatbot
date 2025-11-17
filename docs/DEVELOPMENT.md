# Desenvolvimento

## Requisitos

- Node.js LTS
- pnpm
- Postgres (quando persistência é necessária)

## Scripts

- `pnpm dev`: inicia servidor de desenvolvimento
- `pnpm build`: migrações e build
- `pnpm start`: inicia servidor de produção local
- `pnpm lint`: checagens e formatação
- `pnpm test`: executa Playwright

## Fluxo

1. Configure `.env.local`
2. `pnpm install`
3. `pnpm db:migrate`
4. `pnpm dev`

## Convenções

- App Router com RSC/Server Actions
- Componentes em `components/` com Tailwind e Radix
- Queries e schemas organizados em `lib/db/`