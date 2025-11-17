# Arquitetura

- App Router (`app/`): páginas e rotas (server components, server actions, APIs).
- UI (`components/`): elementos visuais, input multimodal, mensagens, sidebar.
- Autenticação (`Clerk`): provider no `app/layout.tsx`, gating via `@clerk/nextjs/server`.
- Dados (`lib/db/`): Drizzle + Postgres para chats, mensagens, streams, documentos.
- IA (`lib/ai/`): resolver dinâmico de provedores e catálogo de modelos.
- APIs (`app/(chat)/api/*`): chat/stream, histórico, documento, sugestões, votos, upload.

## Fluxo de Chat

- Cliente usa `useChat` (AI SDK) para enviar/receber mensagens.
- API `/api/chat` valida sessão (`auth()`), carrega histórico, salva mensagens e inicia stream.
- Ferramentas (tools) opcionais: clima, criação/atualização de documento, sugestões.

## Persistência

- Tabelas: `chat`, `message`, `stream`, `document`, `suggestion`, `vote`, `user`.
- Mutações executadas por queries em `lib/db/queries.ts`.

## Streams

- Resumable streams: desativados sem `REDIS_URL`. Quando configurados, permitem retomar geração.

## UI/Estado

- Sidebar: histórico paginado (SWR), exclusão e visibilidade.
- Input multimodal: texto e anexos (upload para Blob).