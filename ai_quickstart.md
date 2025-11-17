# AI Quickstart: Next.js AI Chatbot

**Última Atualização:** 2025-11-17 23:45  
**Status:** Produção - Codebase limpa e documentada  
**Workspace:** /Users/guilhermevarela/Public/ai-chat-sdk

## Visão Geral do Projeto

Chatbot de IA completo construído com Next.js 16, AI SDK da Vercel e integração com múltiplos provedores de IA. Oferece interface de chat em tempo real, criação de documentos, suporte a anexos multimodais e integração extensível via Model Context Protocol (MCP).

## Stack Tecnológica

### Core
- **Next.js** 16.0.3 (App Router, RSC, Server Actions)
- **React** 19 (Server Components)
- **TypeScript** 5.6.3
- **Node.js** 18+ / pnpm 9.12.3

### Autenticação
- **Clerk** 6.35.1
  - OAuth e email/senha
  - Middleware de proteção de rotas
  - Componentes: SignedIn/SignedOut, UserButton

### IA e Streaming
- **Vercel AI SDK** 5.0.93
- **Provedores:**
  - @ai-sdk/xai 2.0.33 (Grok Vision, Grok Reasoning)
  - @ai-sdk/openai 2.0.67 (GPT-4o Mini)
  - @ai-sdk/google 2.0.33 (Gemini 1.5 Flash)
  - @ai-sdk/gateway 2.0.9 (AI Gateway)
- **resumable-stream** 2.0.0 (streams com Redis)
- **tokenlens** 1.3.0 (tracking de uso)

### Database e Storage
- **Drizzle ORM** 0.34.0
- **PostgreSQL** (Neon em produção)
- **Vercel Blob** 0.24.1 (uploads)
- **Redis** 5.0.0 (opcional, para resumable streams)

### UI
- **Tailwind CSS** 4.1.13
- **shadcn/ui** (Radix UI)
- **Lucide Icons**
- **CodeMirror 6** (editor de código)
- **Prosemirror** (editor de texto rico)
- **Shiki** (syntax highlighting)

### Desenvolvimento
- **Playwright** 1.50.1 (testes E2E)
- **Biome** 2.2.2 (lint/format)
- **drizzle-kit** 0.25.0
- **tsx** 4.19.1

## Arquitetura

```mermaid
graph TB
    A[Cliente Browser] --> B[Next.js App Router]
    B --> C{Clerk Auth}
    
    C -->|Autenticado| D[Rotas de Chat]
    C -->|Não autenticado| E[Landing Page]
    
    D --> F[/api/chat]
    F --> G[AI Provider Resolver]
    
    G --> H[AI Gateway]
    G --> I[Direct Providers]
    
    H --> J[XAI/Grok]
    H --> K[OpenAI]
    H --> L[Google Gemini]
    
    I --> J
    I --> K
    I --> L
    
    F --> M[(PostgreSQL)]
    F --> N[(Redis)]
    F --> O[Vercel Blob]
    F --> P[MCP Manager]
    
    P --> Q[MCP Server stdio]
    P --> R[MCP Server SSE]
    
    style A fill:#e1f5ff
    style M fill:#ffe1e1
    style J fill:#e1ffe1
    style K fill:#e1ffe1
    style L fill:#e1ffe1
```

## Modelo de Dados

### Entidades Principais

**User**
- Gerenciado pelo Clerk
- `id`: text (UUID do Clerk)
- `email`: varchar(64)

**Chat**
- `id`: UUID (PK)
- `userId`: text (FK → User)
- `createdAt`: timestamp
- `title`: text
- `visibility`: enum ['public', 'private']
- `lastContext`: jsonb (métricas de uso)

**Message_v2**
- `id`: UUID (PK)
- `chatId`: UUID (FK → Chat)
- `role`: varchar (user, assistant, system)
- `parts`: json (conteúdo da mensagem)
- `attachments`: json (anexos)
- `createdAt`: timestamp

**Document**
- `id`: UUID
- `createdAt`: timestamp (parte da PK composta)
- `userId`: text (FK → User)
- `title`: text
- `content`: text
- `kind`: enum ['text', 'code', 'image', 'sheet']

**Suggestion**
- `id`: UUID (PK)
- `documentId`: UUID + `documentCreatedAt`: timestamp (FK → Document)
- `userId`: text (FK → User)
- `originalText`: text
- `suggestedText`: text
- `description`: text
- `isResolved`: boolean
- `createdAt`: timestamp

**Vote_v2**
- `chatId`: UUID (FK → Chat)
- `messageId`: UUID (FK → Message_v2)
- `isUpvoted`: boolean
- PK: (chatId, messageId)

**Stream**
- `id`: UUID (PK)
- `chatId`: UUID (FK → Chat)
- `createdAt`: timestamp
- Para resumable streaming com Redis

## Configuração Inicial

### 1. Variáveis de Ambiente

Crie `.env.local`:

```bash
# Autenticação (obrigatório)
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...

# Database (obrigatório)
POSTGRES_URL=postgresql://user:pass@host:5432/db

# Storage (obrigatório para uploads)
BLOB_READ_WRITE_TOKEN=vercel_blob_...

# AI Gateway (recomendado)
AI_GATEWAY_API_KEY=vgai_...

# Provedores diretos (opcional)
XAI_API_KEY=xai-...
OPENAI_API_KEY=sk-...
GOOGLE_API_KEY=...

# Redis (opcional - resumable streams)
REDIS_URL=redis://...

# MCP (opcional - via env)
MCP_SERVERS='[...]'
```

### 2. Instalação

```bash
cd ai-chatbot
pnpm install
pnpm db:migrate
pnpm dev
```

Acesse: http://localhost:3000

## Fluxos Principais

### Fluxo de Autenticação

1. Usuário acessa aplicação
2. Clerk middleware verifica sessão
3. Se não autenticado → exibe landing page com botões Sign In/Up
4. Após autenticação → acesso completo
5. `auth()` nas rotas protegidas obtém `userId`

### Fluxo de Chat com Streaming

1. **Input:**
   - Usuário digita mensagem em `MultimodalInput`
   - Anexos são enviados para Vercel Blob
   - `useChat` hook envia mensagem via POST /api/chat

2. **Processamento:**
   - API valida sessão com Clerk
   - Carrega histórico do chat (PostgreSQL)
   - Resolve provedor de IA baseado no modelo selecionado
   - Inicia geração com streaming

3. **Streaming:**
   - Resposta enviada via Server-Sent Events
   - `smoothStream` atualiza UI incrementalmente
   - Componente `ThinkingMessage` mostra "Pensando..." com animação
   - Ferramentas (tools) exibidas em `ToolInvocation` component

4. **Persistência:**
   - Mensagem salva em `message_v2` table
   - Métricas de uso atualizadas
   - Se Redis disponível, stream pode ser resumido

### Fluxo de Criação de Documentos

1. IA decide criar documento (tool call)
2. Server Action cria entrada em `document` table
3. Documento renderizado em `Artifact` component (painel direito)
4. Tipos suportados: text, code, image, sheet
5. Usuário pode solicitar sugestões de melhorias
6. Sugestões salvas em `suggestion` table

### Integração MCP

1. **Configuração:**
   - Via UI em `/settings` → salva em `lib/mcp/user-servers.json`
   - Via env var `MCP_SERVERS` → modo read-only

2. **Carregamento:**
   - `MCPManager` em `lib/mcp/manager.ts`
   - Conecta a servidores stdio ou SSE
   - Descobre ferramentas disponíveis
   - Prefixa ferramentas com nome do servidor

3. **Uso:**
   - Ferramentas MCP adicionadas dinamicamente
   - Exemplo: `filesystem_readFile`, `github_searchRepositories`
   - IA escolhe ferramenta apropriada durante conversa

## APIs Disponíveis

### Chat
- `POST /api/chat` - Enviar mensagem e receber stream
- `GET /api/chat/[id]/stream` - Resumir stream interrompido

### Histórico
- `GET /api/history` - Listar chats (paginado, SWR)
- `DELETE /api/history` - Deletar todos os chats
- `DELETE /api/history?id=...` - Deletar chat específico

### Documentos
- `POST /api/document` - Criar/atualizar documento

### Sugestões
- `POST /api/suggestions` - Criar sugestão para documento

### Votos
- `POST /api/vote` - Votar em mensagem (útil/não útil)

### Arquivos
- `POST /api/files/upload` - Upload para Vercel Blob

### MCP
- `GET /api/mcp` - Status de servidores e ferramentas
- `POST /api/mcp/servers` - Adicionar servidor
- `PUT /api/mcp/servers` - Atualizar servidor
- `DELETE /api/mcp/servers?name=...` - Remover servidor

### Utilitários
- `GET /ping` - Health check

## Componentes Principais

### Chat Interface
- `components/chat.tsx` - Container principal
- `components/messages.tsx` - Lista de mensagens
- `components/message.tsx` - Mensagem individual
- `components/thinking-message.tsx` - Indicador de processamento
- `components/multimodal-input.tsx` - Input com anexos

### Sidebar
- `components/app-sidebar.tsx` - Sidebar principal
- `components/sidebar-history.tsx` - Histórico de chats
- `components/sidebar-user-nav.tsx` - Navegação do usuário

### Artifacts
- `components/artifact.tsx` - Container de documento
- `components/document.tsx` - Visualização de documento
- `components/code-editor.tsx` - Editor CodeMirror
- `components/diffview.tsx` - Visualização de diffs

### Ferramentas (Tools)
- `components/elements/tool.tsx` - UI de tool invocation
- `components/message.tsx` - LinkPreview e Sources components

### Settings
- `components/settings/mcp-section.tsx` - Painel MCP
- `components/settings/mcp-server-dialog.tsx` - Modal de configuração

## Modelos de IA Disponíveis

### Padrão (myProvider em lib/ai/provider.ts)
- **chat-model:** XAI Grok-2 Vision (multimodal)
- **chat-model-reasoning:** XAI Grok-3 Mini (raciocínio)
- **title-model:** XAI Grok-2 (títulos)
- **artifact-model:** XAI Grok-2 (documentos)

### Selecionáveis pelo Usuário
- Grok Vision (default)
- Grok Reasoning
- OpenAI GPT-4o Mini
- Gemini 1.5 Flash

## Ferramentas (Tools) Nativas

Definidas em `lib/tools/`:

1. **getWeather**
   - Obter informações meteorológicas
   - Input: location (string)

2. **createDocument**
   - Criar novo documento
   - Input: title, kind, content

3. **updateDocument**
   - Atualizar documento existente
   - Input: id, description, content

4. **requestSuggestions**
   - Gerar sugestões de melhoria
   - Input: documentId

## Scripts de Desenvolvimento

```bash
# Servidor
pnpm dev                # Dev server (Turbopack)
pnpm build             # Production build
pnpm start             # Production server

# Database
pnpm db:generate       # Gerar migrations
pnpm db:migrate        # Aplicar migrations
pnpm db:studio         # Drizzle Studio
pnpm db:push           # Push schema
pnpm db:pull           # Pull schema

# Qualidade
pnpm lint              # Biome lint + format
pnpm test              # Playwright E2E
```

## Deployment

### Vercel (Recomendado)

1. Conectar repositório Git ao Vercel
2. Configurar variáveis de ambiente no dashboard
3. Deploy automático a cada push na branch main

**Vantagens:**
- AI Gateway com OIDC automático
- Edge functions otimizadas
- Blob storage integrado
- Analytics e monitoring

### Self-Hosting

**Requisitos:**
- Node.js 18+
- PostgreSQL database
- Redis (opcional)
- Blob storage (S3-compatible)
- SSL/TLS certificate

**Variáveis obrigatórias:**
- POSTGRES_URL
- BLOB_READ_WRITE_TOKEN
- NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY
- CLERK_SECRET_KEY
- AI provider keys ou AI_GATEWAY_API_KEY

## Testes

### Playwright E2E

```bash
# Instalar browsers
pnpm exec playwright install

# Executar testes
pnpm test

# UI mode
pnpm exec playwright test --ui

# Debug
pnpm exec playwright test --debug
```

**Cobertura:**
- Fluxo de autenticação
- Envio de mensagens e streaming
- Upload de arquivos
- Criação de documentos
- Histórico de chats

## Segurança

### Implementado
- ✅ Autenticação via Clerk
- ✅ Proteção de rotas (middleware)
- ✅ SQL injection prevention (Drizzle ORM)
- ✅ XSS protection (React)
- ✅ Input validation (Zod schemas)
- ✅ HTTPS obrigatório em produção
- ✅ Environment variables para secrets

### Boas Práticas
- Não versionar `.env.local`
- Rotacionar API keys regularmente
- Revisar permissões de MCP servers
- Usar secrets manager em produção
- Implementar rate limiting em APIs públicas

## Performance

### Otimizações Implementadas
- Server Components para SSR
- Streaming responses (SSE)
- SWR para cache client-side
- Resumable streams com Redis
- Image optimization (Next.js)
- Code splitting automático
- Token usage tracking

### Métricas Monitoradas
- Tempo de resposta da IA
- Uso de tokens por requisição
- Taxa de erro de streaming
- Latência de database queries

## Modificações Locais

### Configuração Atual
- ✅ Database: Neon PostgreSQL (migrado de local)
- ✅ Auth: Clerk desabilitado em dev mode (fake userId: "00000000-0000-0000-0000-000000000001")
- ✅ Tradução: Português (pt-BR)
- ✅ MCP: Deepwiki server configurado
- ✅ UI: ThinkingMessage com animação e "Pensando..." em português
- ✅ UI: Sources e LinkPreview components ativos
- ✅ UI: formatToolName para nomes legíveis de ferramentas MCP
- ✅ Fixes: Hydration errors resolvidos (suppressHydrationWarning)
- ✅ Fixes: Streaming working end-to-end
- ✅ Código limpo: Logs de debug removidos (mantidos apenas console.error críticos)
- ✅ Documentação atualizada: README.md, MCP.md, ai_quickstart.md

### Debug/Logging
- **Produção:** ~10 console.error para erros críticos apenas
- **Removidos:** Todos console.log/debug de desenvolvimento
- **Mantidos:** Erros de MCP loading, chat errors, parsing errors

## Troubleshooting Comum

### Streaming não funciona
- ✅ Verificar `fetchWithErrorHandlers` não está consumindo stream
- ✅ Verificar content-type headers (text/event-stream)
- ✅ Testar com curl para isolar problema

### MCP server não conecta
- Verificar comando/URL
- Conferir variáveis de ambiente
- Testar comando manualmente
- Ver logs em `/api/mcp`

### Hydration errors
- Adicionar `suppressHydrationWarning` em componentes Radix
- Verificar diferenças server/client rendering

### Database errors
- Executar `pnpm db:migrate`
- Verificar `POSTGRES_URL`
- Conferir schema compatibility

## Recursos Adicionais

### Documentação Interna
- `docs/ARCHITECTURE.md` - Arquitetura detalhada
- `docs/AUTHENTICATION.md` - Clerk integration
- `docs/ENVIRONMENT.md` - Variáveis de ambiente
- `docs/DEVELOPMENT.md` - Guia de desenvolvimento
- `docs/TESTING.md` - Testes
- `docs/DEPLOYMENT.md` - Deploy
- `docs/MCP.md` - Model Context Protocol

### Links Externos
- [Next.js Docs](https://nextjs.org/docs)
- [AI SDK Docs](https://sdk.vercel.ai/docs)
- [Clerk Docs](https://clerk.com/docs)
- [Drizzle ORM](https://orm.drizzle.team)
- [MCP Spec](https://modelcontextprotocol.io)

---

**Última análise:** 2025-11-17  
**Próxima revisão:** Após mudanças significativas na arquitetura
