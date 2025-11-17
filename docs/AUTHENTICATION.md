# Autenticação (Clerk)

- Provider: `<ClerkProvider>` em `app/layout.tsx`.
- Componentes: `<SignedIn>`, `<SignedOut>`, `<UserButton>`, `<SignInButton>`, `<SignUpButton>`.
- Middleware: `clerkMiddleware()` em `middleware.ts` protege rotas e APIs.
- Server-side: `auth()` de `@clerk/nextjs/server` para obter `{ userId }` nas páginas/rotas.

## Variáveis

- `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`
- `CLERK_SECRET_KEY`

## Gating

- Páginas e APIs sensíveis verificam `userId` para autorizar ações e filtrar dados por usuário.