# Variáveis de Ambiente

## Essenciais

- `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`: chave pública do Clerk
- `CLERK_SECRET_KEY`: chave secreta do Clerk
- `POSTGRES_URL`: conexão Postgres (Drizzle)

## IA

- `AI_GATEWAY_API_KEY`: ativa o Vercel AI Gateway
- `XAI_API_KEY`: provedor direto xAI (opcional)
- `OPENAI_API_KEY`: provedor direto OpenAI (opcional)
- `OPENROUTER_API_KEY`: agregador OpenRouter (opcional)

## Armazenamento e Streams

- `BLOB_READ_WRITE_TOKEN`: upload público para Vercel Blob
- `REDIS_URL`: habilita streams retomáveis (opcional)

## MCP

- `MCP_SERVERS`: JSON string com servidores MCP. Exemplo:

```json
[
  {
    "name": "filesystem",
    "type": "stdio",
    "enabled": true,
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]
  }
]
```

Quando essa variável está presente o painel `/settings` exibe as configurações em modo somente leitura.

## Boas práticas

- Use `.env.local` para chaves reais
- Não versione `.env*`
