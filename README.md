# AI Chatbot com Next.js

Chatbot completo com Next.js 16, AI SDK, suporte a múltiplos provedores de IA e integração Model Context Protocol (MCP).

## Recursos

- **Framework:** Next.js 16 (App Router) com React Server Components
- **IA:** Suporte a múltiplos provedores (XAI/Grok, OpenAI, Google Gemini) via AI SDK
- **Autenticação:** Clerk com suporte a OAuth e email
- **Database:** PostgreSQL com Drizzle ORM
- **Storage:** Vercel Blob para uploads de arquivos
- **UI:** Tailwind CSS + shadcn/ui (Radix UI)
- **MCP:** Integração com servidores Model Context Protocol (stdio e SSE)
- **Streaming:** Respostas em tempo real com Server-Sent Events
- **Testes:** Playwright para testes E2E

## Início Rápido

```bash
# Instalar dependências
pnpm install

# Configurar variáveis de ambiente (veja seção abaixo)
cp .env.example .env.local

# Executar migrações do banco de dados
pnpm db:migrate

# Iniciar servidor de desenvolvimento
pnpm dev
```

Acesse `http://localhost:3000`

## Variáveis de Ambiente

Crie um arquivo `.env.local` na raiz do projeto:

```bash
# Autenticação (obrigatório)
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...

# Banco de dados (obrigatório)
POSTGRES_URL=postgresql://...

# Storage (obrigatório para uploads)
BLOB_READ_WRITE_TOKEN=vercel_blob_...

# AI Gateway (recomendado)
AI_GATEWAY_API_KEY=vgai_...

# Provedores de IA (opcionais - usar se não tiver AI Gateway)
XAI_API_KEY=xai-...
OPENAI_API_KEY=sk-...
GOOGLE_API_KEY=...

# Redis (opcional - para streams resumíveis)
REDIS_URL=redis://...

# MCP Servers (opcional - configuração via variável de ambiente)
MCP_SERVERS='[{"name":"filesystem","type":"stdio","enabled":true,"command":"npx","args":["-y","@modelcontextprotocol/server-filesystem","/tmp"]}]'
```

## Estrutura do Projeto

```
ai-chatbot/
├── app/                    # App Router (páginas e APIs)
│   ├── (chat)/            # Rotas de chat
│   │   ├── api/          # API endpoints
│   │   └── chat/         # Páginas de chat
│   └── settings/         # Configurações (MCP)
├── components/            # Componentes React
│   ├── ui/              # Componentes base (shadcn/ui)
│   └── settings/        # Componentes de configuração
├── lib/                  # Lógica de negócio
│   ├── ai/             # Provedores e modelos de IA
│   ├── db/             # Schema e queries do banco
│   ├── mcp/            # Gerenciamento de servidores MCP
│   └── tools/          # Ferramentas nativas do chatbot
├── docs/                # Documentação técnica
└── tests/              # Testes E2E com Playwright
```

## Funcionalidades

### Chat Inteligente
- Streaming de respostas em tempo real
- Suporte a múltiplos modelos de IA
- Histórico de conversas com busca
- Mensagens com anexos (imagens, arquivos)
- Indicador visual de processamento

### Ferramentas (Tools)
- **Documentos:** Criar e editar documentos (texto, código, planilhas)
- **Clima:** Obter informações meteorológicas
- **Sugestões:** Melhorias automáticas para documentos
- **MCP:** Ferramentas dinâmicas de servidores MCP conectados

### Model Context Protocol (MCP)
- Interface de configuração em `/settings`
- Suporte a transportes stdio e SSE
- Descoberta automática de ferramentas
- Visualização de status de conexão
- Configuração via UI ou variável de ambiente

### Modelos Disponíveis
- **Grok Vision** (XAI) - Multimodal com visão
- **Grok Reasoning** (XAI) - Raciocínio aprimorado
- **GPT-4o Mini** (OpenAI) - Rápido e eficiente
- **Gemini 1.5 Flash** (Google) - Alta velocidade

## Scripts Disponíveis

```bash
# Desenvolvimento
pnpm dev              # Servidor de desenvolvimento
pnpm build           # Build de produção
pnpm start           # Servidor de produção

# Banco de dados
pnpm db:generate     # Gerar migrações
pnpm db:migrate      # Aplicar migrações
pnpm db:studio       # Abrir Drizzle Studio
pnpm db:push         # Push do schema
pnpm db:pull         # Pull do schema

# Qualidade
pnpm lint            # Lint e formatação
pnpm test            # Testes E2E
```

## Configuração de Servidores MCP

Acesse `/settings` para adicionar servidores MCP. Exemplos:

### Servidor stdio (executa localmente)
```json
{
  "name": "filesystem",
  "type": "stdio",
  "enabled": true,
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"],
  "env": {}
}
```

### Servidor SSE (conexão HTTP)
```json
{
  "name": "remote-api",
  "type": "sse",
  "enabled": true,
  "url": "https://api.example.com/mcp",
  "headers": {
    "Authorization": "Bearer token"
  }
}
```

## Deploy

### Vercel (Recomendado)
1. Conecte seu repositório no Vercel
2. Configure as variáveis de ambiente no dashboard
3. Deploy automático a cada push

### Requisitos para Deploy
- PostgreSQL database
- Vercel Blob storage (ou alternativa compatível com S3)
- Redis (opcional, para streams resumíveis)
- Variáveis de ambiente configuradas

## Desenvolvimento

### Convenções
- Server Components para renderização inicial
- Client Components (`"use client"`) apenas quando necessário
- Server Actions para mutações
- Tailwind CSS para estilização
- TypeScript strict mode

### Adicionar um Novo Provedor de IA
1. Instalar SDK: `pnpm add @ai-sdk/provider-name`
2. Adicionar configuração em `lib/ai/provider.ts`
3. Adicionar modelos em `lib/ai/models.ts`
4. Configurar variável de ambiente

## Documentação Adicional

- [Arquitetura](./docs/ARCHITECTURE.md) - Estrutura e fluxos do sistema
- [Autenticação](./docs/AUTHENTICATION.md) - Integração com Clerk
- [Variáveis de Ambiente](./docs/ENVIRONMENT.md) - Configurações detalhadas
- [Desenvolvimento](./docs/DEVELOPMENT.md) - Guia para desenvolvedores
- [Testes](./docs/TESTING.md) - Como executar e criar testes
- [Deploy](./docs/DEPLOYMENT.md) - Guia de deployment

## Tecnologias Principais

- Next.js 16.0.3
- React 19
- TypeScript 5.6.3
- Clerk 6.35.1
- AI SDK 5.0.93
- Drizzle ORM 0.34.0
- Tailwind CSS 4.1.13
- Playwright 1.50.1

## Licença

MIT

## Suporte

Para problemas ou dúvidas, abra uma issue no repositório.
