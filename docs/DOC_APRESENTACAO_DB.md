# Apresentação – Conexão PostgreSQL 17 com Prisma

Este documento resume tudo que foi configurado para a apresentação do TCC: banco de dados, extensões, tabelas, endpoints e scripts de operação. O objetivo é deixar o uso simples, previsível e à prova de falhas (com fallback em memória/arquivo quando não houver banco).

## Visão Geral
- Banco: `PostgreSQL 17` (via Docker ou instância local).
- ORM: `Prisma` com client gerado em `node_modules/@prisma/client`.
- Fallback: se `DATABASE_URL` não estiver definido, as APIs continuam funcionando com dados em memória (usuários) e arquivo JSON (questões).

## Extensões Postgres
- `pgcrypto`: funções criptográficas e suporte a `gen_random_uuid()`.
- `citext`: tipo de texto case-insensitive (útil para e-mails).
- `pg_stat_statements`: observabilidade de queries.

Como habilitar:
- `npm run db:extensions` (requer `DATABASE_URL` válido e Postgres acessível).

## Tabelas (schema Prisma)
- `User`
  - `id (uuid)`, `name`, `email (unique)`, `level`, `xp`, `totalQuizzes`, `correctAnswers`, `streak`, `achievements (String[])`, `createdAt`, `lastLoginAt`.
  - Uso: ranking, perfil, login/registro e atualização de estatísticas.
- `Question`
  - `id (uuid)`, `enunciado`, `alternativas (String[])`, `respostaCorreta`, `tema?`, `explicacao?`, `subject?`, `dificuldade?`, `createdAt`.
  - Uso: consulta/filtro de questões e CRUD simples.

## Scripts Importantes
- `npm run db:up`: sobe Postgres via Docker (imagem `postgres:17`).
- `npm run db:generate`: gera o client do Prisma a partir do schema.
- `npm run db:migrate`: cria migration e aplica (`dev`) com nome `init`.
- `npm run db:push`: aplica o schema direto no banco (sem migrations).
- `npm run db:seed`: popula usuários de exemplo.
- `npm run db:check`: verifica conexão e conta registros de `User`.
- `npm run db:extensions`: habilita `pgcrypto`, `citext`, `pg_stat_statements`.
- `npm run db:studio`: abre o Prisma Studio para inspecionar dados.

## Variáveis de Ambiente
- `.env` deve conter:
  - `DATABASE_URL="postgresql://postgres:postgres@localhost:5432/quizmaster?schema=public"`
  - Se não definido, as APIs usam fallback e continuam funcionais.

## Endpoints (uso simples)
- Usuários (`/api/users`):
  - `GET /api/users?action=ranking` – ranking ordenado por `xp > level > streak`.
  - `GET /api/users?action=profile&userId=<id>` – perfil sem expor `email`.
  - `POST` com corpo:
    - `{"action":"register","name":"Nome","email":"email@exemplo.com"}`
    - `{"action":"login","email":"email@exemplo.com"}`
    - `{"action":"updateStats","userId":"<id>","correct":N,"total":M,"xpGained":X}`
    - `{"action":"addAchievement","userId":"<id>","achievementId":"first-quiz"}`
- Questões (`/api/questions`):
  - `GET` com filtros opcionais: `subject`, `topic`, `difficulty` (`facil|medio|dificil`).
    - Mescla dados do DB (se ativo), do arquivo `lib/questions.json` e da base compilada, removendo duplicados.
  - `POST` com corpo de questão: cria no DB quando disponível; senão, salva no arquivo.
- Questões por ID (`/api/questions/[id]`):
  - `PUT`: atualiza questão. Com DB ativo, usa `id` UUID; em fallback de arquivo, usa formato `dificuldade-index` (ex.: `facil-5`).
  - `DELETE`: exclui questão por `id` (DB) ou por `dificuldade-index` (arquivo).

## Fluxo de Apresentação (Passo-a-passo)
1. Instalar deps: `npm install`
2. Subir DB (opcional): `npm run db:up`
3. Copiar `.env.example` para `.env` e manter/ajustar `DATABASE_URL`.
4. Gerar client: `npm run db:generate`
5. Aplicar schema: `npm run db:push` ou `npm run db:migrate`
6. Habilitar extensões: `npm run db:extensions`
7. Seed de usuários: `npm run db:seed`
8. Verificação: `npm run db:check` e `npm run db:studio`
9. Rodar app: `npm run dev`

## Estratégia de Robustez
- Sempre que `DATABASE_URL` não existir ou o banco não estiver acessível, as APIs caem para armazenamentos alternativos (memória/arquivo). Isso garante que a apresentação não quebre por indisponibilidade do banco.
- Quando o banco está ativo, toda persistência acontece no Postgres via Prisma.

## O que foi ajustado para “onde dá erro”
- Conflito de IDs em `/api/questions/[id]`:
  - Agora aceita `id` UUID (DB) e mantém suporte a `dificuldade-index` no fallback de arquivo, evitando 404 indevidos.
- Tipagem e import paths:
  - Checagem de tipos (`npm run type-check`) passou, evitando erros de build.
- Schema Prisma simplificado:
  - Arrays (`achievements`, `alternativas`) usam `String[]` sem atributo específico de coluna, reduzindo risco de incompatibilidades de migração.

---
Qualquer teste rápido:
- Ranking: acesse `GET /api/users?action=ranking`.
- Criar questão: `POST /api/questions` com `enunciado`, `alternativas`, `respostaCorreta`, `dificuldade`.
- Atualizar/excluir questão: `PUT/DELETE /api/questions/[id]` (UUID se no DB; `dificuldade-index` se usando arquivo).