# Deploy

## Vercel

- Preferível com AI Gateway
- Configure environment variables no dashboard
- Deploy contínuo via Git

## Variáveis

- Defina `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`, `CLERK_SECRET_KEY`
- Defina `AI_GATEWAY_API_KEY` (recomendado)
- Defina `POSTGRES_URL`, `BLOB_READ_WRITE_TOKEN`
- Opcional: `REDIS_URL` para streams retomáveis

## Observações

- Telemetria de IA disponível via Gateway
- Imagens remotas autorizadas em `next.config.ts`