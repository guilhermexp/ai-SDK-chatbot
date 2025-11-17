# Model Context Protocol (MCP)

Integração completa com o Model Context Protocol para expandir as capacidades do chatbot com ferramentas externas.

## O que é MCP?

O Model Context Protocol (MCP) é um protocolo padronizado que permite que aplicações de IA se conectem a servidores que fornecem ferramentas (tools), recursos (resources) e prompts adicionais. Com MCP, você pode:

- Conectar o chatbot a sistemas de arquivos
- Acessar APIs externas (GitHub, bancos de dados, etc.)
- Executar comandos personalizados
- Integrar serviços de terceiros

## Arquitetura

```
┌─────────────────┐
│   Chatbot UI    │
└────────┬────────┘
         │
┌────────▼────────┐
│   MCP Manager   │  (lib/mcp/manager.ts)
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
┌───▼───┐ ┌──▼───┐
│ stdio │ │ SSE  │  (Transportes)
└───┬───┘ └──┬───┘
    │        │
┌───▼────────▼───┐
│  MCP Servers   │
└────────────────┘
```

## Configuração

### Via Interface (Recomendado)

1. Acesse `/settings` no chatbot
2. Na seção "Model Context Protocol", clique em "Adicionar Servidor"
3. Preencha os dados do servidor:
   - **Nome:** Identificador único (ex: "filesystem", "github")
   - **Tipo:** stdio ou SSE
   - **Habilitado:** Ativar/desativar o servidor
   - **Configurações específicas do tipo**

### Via Variável de Ambiente

Defina `MCP_SERVERS` no `.env.local`:

```bash
MCP_SERVERS='[
  {
    "name": "filesystem",
    "type": "stdio",
    "enabled": true,
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/seu-usuario"],
    "env": {}
  },
  {
    "name": "github",
    "type": "stdio",
    "enabled": true,
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-github"],
    "env": {
      "GITHUB_TOKEN": "ghp_..."
    }
  }
]'
```

**Nota:** Quando `MCP_SERVERS` está definida, a UI entra em modo somente leitura.

## Tipos de Transporte

### stdio (Standard Input/Output)

Executa um processo local e comunica via stdin/stdout.

**Configuração:**
```json
{
  "name": "filesystem",
  "type": "stdio",
  "enabled": true,
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"],
  "env": {
    "VAR_NAME": "value"
  }
}
```

**Campos:**
- `command`: Comando executável (npx, node, python, etc.)
- `args`: Array de argumentos para o comando
- `env`: Variáveis de ambiente (opcional)

**Casos de uso:**
- Servidores MCP locais
- Scripts personalizados
- Ferramentas do sistema de arquivos

### SSE (Server-Sent Events)

Conecta a um servidor HTTP que implementa o protocolo MCP via SSE.

**Configuração:**
```json
{
  "name": "remote-api",
  "type": "sse",
  "enabled": true,
  "url": "https://api.example.com/mcp",
  "headers": {
    "Authorization": "Bearer token",
    "X-Custom-Header": "value"
  }
}
```

**Campos:**
- `url`: Endpoint do servidor MCP
- `headers`: Headers HTTP (opcional)

**Casos de uso:**
- APIs remotas
- Serviços em cloud
- Microserviços

## Servidores MCP Oficiais

### Filesystem
Acesso a arquivos e diretórios locais.

```bash
npx -y @modelcontextprotocol/server-filesystem /caminho/para/diretorio
```

**Ferramentas:**
- `read_file`: Ler conteúdo de arquivo
- `write_file`: Escrever em arquivo
- `list_directory`: Listar diretório
- `create_directory`: Criar diretório
- `move_file`: Mover/renomear arquivo
- `search_files`: Buscar arquivos

### GitHub
Interação com repositórios GitHub.

```bash
npx -y @modelcontextprotocol/server-github
```

**Variáveis de ambiente:**
```bash
GITHUB_TOKEN=ghp_...
```

**Ferramentas:**
- `search_repositories`: Buscar repositórios
- `get_file_contents`: Ler arquivo do repo
- `create_pull_request`: Criar PR
- `create_issue`: Criar issue
- `list_commits`: Listar commits

### PostgreSQL
Consultas SQL em bancos PostgreSQL.

```bash
npx -y @modelcontextprotocol/server-postgres postgresql://...
```

**Ferramentas:**
- `query`: Executar SQL
- `list_tables`: Listar tabelas
- `describe_table`: Descrever schema

### Brave Search
Busca na web via Brave Search API.

```bash
npx -y @modelcontextprotocol/server-brave-search
```

**Variáveis de ambiente:**
```bash
BRAVE_API_KEY=...
```

**Ferramentas:**
- `brave_web_search`: Buscar na web
- `brave_local_search`: Busca local

## Gerenciamento via API

### Listar Servidores e Status

```bash
GET /api/mcp
```

**Resposta:**
```json
{
  "servers": [
    {
      "name": "filesystem",
      "type": "stdio",
      "enabled": true,
      "status": "connected",
      "tools": ["filesystem_readFile", "filesystem_writeFile"]
    }
  ],
  "totalTools": 6
}
```

### Adicionar Servidor

```bash
POST /api/mcp/servers
Content-Type: application/json

{
  "name": "new-server",
  "type": "stdio",
  "enabled": true,
  "command": "npx",
  "args": ["-y", "package-name"]
}
```

### Atualizar Servidor

```bash
PUT /api/mcp/servers
Content-Type: application/json

{
  "name": "existing-server",
  "enabled": false
}
```

### Remover Servidor

```bash
DELETE /api/mcp/servers?name=server-name
```

## Uso no Chat

Quando servidores MCP estão conectados, suas ferramentas ficam disponíveis automaticamente para o chatbot.

**Exemplo de conversa:**

```
Usuário: Liste os arquivos no diretório /tmp

Assistente: [Chama filesystem_listDirectory com path="/tmp"]
Encontrei 5 arquivos em /tmp:
- file1.txt
- file2.log
- ...
```

As ferramentas MCP são prefixadas com o nome do servidor:
- `filesystem_readFile`
- `github_searchRepositories`
- `postgres_query`

## Componentes da Interface

### MCPSection
Componente principal em `/settings` que exibe:
- Lista de servidores configurados
- Status de conexão (conectado, desconectado, erro)
- Número de ferramentas disponíveis
- Botões para adicionar/editar/remover servidores

### MCPServerDialog
Modal para configurar servidores MCP com:
- Campos específicos para stdio e SSE
- Validação de configuração
- Preview de ferramentas disponíveis (após conexão)

### MCPConnectionIndicator
Indicador visual no chat mostrando:
- Servidores ativos
- Ferramentas sendo utilizadas
- Status de conexão em tempo real

## Troubleshooting

### Servidor não conecta

1. Verifique se o comando está correto (stdio)
2. Verifique se a URL está acessível (SSE)
3. Confira variáveis de ambiente necessárias
4. Veja logs no console do servidor

### Ferramentas não aparecem

1. Certifique-se que o servidor está `enabled: true`
2. Verifique se o servidor implementa o protocolo MCP corretamente
3. Recarregue a página após adicionar servidor
4. Veja `/api/mcp` para status detalhado

### Erro de permissão (stdio)

- No macOS/Linux: verifique permissões do comando
- Tente executar o comando manualmente primeiro
- Use caminhos absolutos quando possível

### Timeout em SSE

- Aumente timeout na configuração do servidor
- Verifique conectividade de rede
- Confirme que o servidor responde no formato MCP

## Desenvolvendo seu Servidor MCP

### Estrutura Básica

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server({
  name: "my-server",
  version: "1.0.0"
}, {
  capabilities: {
    tools: {}
  }
});

// Registrar ferramenta
server.setRequestHandler("tools/list", async () => ({
  tools: [
    {
      name: "my_tool",
      description: "Descrição da ferramenta",
      inputSchema: {
        type: "object",
        properties: {
          param: { type: "string" }
        }
      }
    }
  ]
}));

server.setRequestHandler("tools/call", async (request) => {
  const { name, arguments: args } = request.params;
  
  if (name === "my_tool") {
    // Lógica da ferramenta
    return {
      content: [
        {
          type: "text",
          text: "Resultado da ferramenta"
        }
      ]
    };
  }
});

// Iniciar servidor
const transport = new StdioServerTransport();
await server.connect(transport);
```

### Boas Práticas

1. **Nomes descritivos:** Use nomes claros para ferramentas
2. **Schema completo:** Defina `inputSchema` detalhado
3. **Tratamento de erros:** Retorne erros informativos
4. **Documentação:** Descreva bem cada ferramenta
5. **Validação:** Valide inputs antes de processar
6. **Timeout:** Implemente timeouts em operações longas

## Segurança

### Considerações Importantes

1. **Permissões de arquivo (stdio):**
   - Limite acesso a diretórios específicos
   - Não permita acesso root sem necessidade
   - Valide caminhos para prevenir directory traversal

2. **Autenticação (SSE):**
   - Use HTTPS sempre
   - Implemente autenticação adequada
   - Rotacione tokens regularmente

3. **Validação de input:**
   - Sanitize todos os inputs
   - Valide tipos e formatos
   - Previna injection attacks

4. **Variáveis de ambiente:**
   - Não versione `.env.local`
   - Use secrets managers em produção
   - Rotacione credenciais periodicamente

## Recursos Adicionais

- [MCP Specification](https://modelcontextprotocol.io/docs)
- [Official Servers](https://github.com/modelcontextprotocol/servers)
- [SDK Documentation](https://github.com/modelcontextprotocol/typescript-sdk)
- [Community Servers](https://github.com/punkpeye/awesome-mcp-servers)
