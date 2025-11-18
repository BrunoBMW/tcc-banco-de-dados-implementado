# Fluxo de Acesso e Cadastro – Estado Atual

Este documento descreve como o acesso e cadastro de usuários funciona após remover a página `/entrada` e desativar o middleware de bloqueio. O site volta ao comportamento original: conteúdo acessível sem gate de cadastro, com login/cadastro via API.

## Visão Geral
- Sem bloqueio por cookies: não há redirecionamento automático para páginas de cadastro.
- Cadastro/Login via API de usuários com fallback em memória quando o banco falha.
- Status do banco disponível via endpoint dedicado.

## Endpoints
- `GET /api/db-check`: retorna o status da conexão do banco.
  - Exemplo de resposta com banco OK: `{ connected: true, provider: 'postgresql', userCount: 42 }`.
- `GET /api/users`: lista usuários. Se Prisma falhar, cai para memória.
- `GET /api/users?action=ranking`: ranking por `xp`, `level`, `streak`.
- `GET /api/users?action=profile&userId=<id>`: dados do usuário por `id`.
- `POST /api/users?action=register`: cria usuário por e-mail.
  - Regras: exige `email`; valida duplicidade; oculta `email` na resposta.
  - Fallback: persiste em memória quando o banco indisponível.
- `POST /api/users?action=login`: login por e-mail.
  - Atualiza `lastLoginAt`; cria usuário caso não exista.
  - Fallback: memória quando banco indisponível.
- `POST /api/users?action=addAchievement`: adiciona conquista a um usuário.
  - Regras: exige `userId` e `achievementId`.

## Comportamento quando o Banco Falha
- A API tenta o Prisma e captura `PrismaClientInitializationError`.
- Em erro, utiliza estrutura em memória para não quebrar o site.
- `email` nunca é incluído nos objetos de usuário da resposta pública.

## Páginas Relevantes
- `/login`: tela de login integrada à API.
- `/register`: tela de cadastro (opcional no fluxo; mantida como entregue originalmente).
- Não há mais `/entrada`: arquivo desativado (renomeado).

## Operacional
- Configurar banco real:
  1. Defina `DATABASE_URL` no `.env` (Neon/Supabase/Render).
  2. Execute `npm run db:generate` e `npm run db:push`.
  3. Opcional: `npm run db:extensions` e `npm run db:check`.
  4. Verifique `GET /api/db-check`.

## Observações
- Com o banco ativo, os dados serão persistidos em Postgres; sem banco, apenas em memória.
- O build de produção pode acusar problemas na rota `/dashboard` (coleta de dados). Ajustar essa rota antes do deploy.

## Histórico
- Página `/entrada` e middleware de gate desativados para retornar ao estado original.
- API fortalecida com `try/catch` e fallback para evitar 500 quando Prisma falhar.