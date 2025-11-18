# Guia Rápido do Jogo (QuizMaster)

Este guia ajuda você a validar rapidamente se o app está funcionando e como navegar nas principais telas do jogo.

## Como iniciar

- Inicie o servidor de desenvolvimento: `npm run dev`
- Abra no navegador: `http://localhost:3000`

## Fluxo básico

- Página inicial: `http://localhost:3000/`
- Registro: `http://localhost:3000/register`
  - Crie uma conta com nome e e-mail.
  - Se `DATABASE_URL` estiver configurado, o usuário é salvo no PostgreSQL.
- Login: `http://localhost:3000/login`
  - Entre com seu e-mail.
  - O campo `lastLoginAt` é atualizado.

## Jogo e modos

- Jogo (quiz): `http://localhost:3000/game`
  - Responda perguntas, ganhe XP e avance de nível.
  - O progresso atualiza na API de usuários (`updateStats`).
- Batalha (modos):
  - Boss: `http://localhost:3000/battle/boss`
  - Survival: `http://localhost:3000/battle/survival`
  - Team Conquest: `http://localhost:3000/battle/team-conquest`

## Ranking e conquistas

- Ranking: `http://localhost:3000/ranking`
  - Lista ordenada por `xp`, depois `level` e `streak`.
- Conquistas: `http://localhost:3000/achievements`
  - Visualize badges e progressos.

## Biblioteca e estudos

- Biblioteca: `http://localhost:3000/library`
- Modo Estudos: `http://localhost:3000/modo-estudos`

## API útil para testes

- `GET /api/users?action=ranking` — verifica ordenação do ranking.
- `POST /api/users` `{ action: "register", name, email }` — cria usuário.
- `POST /api/users` `{ action: "login", email }` — faz login.
- `POST /api/users` `{ action: "updateStats", userId, correct, total, xpGained }` — atualiza estatísticas.
- `PUT /api/questions/{id}` — atualiza uma questão (DB/arquivo).
- `DELETE /api/questions/{id}` — remove uma questão.

## Banco de dados

- Verificar conexão: `npm run db:check`
  - Se aparecer "DATABASE_URL não definido", o app usa fallback em memória.
  - Configure `.env` com `DATABASE_URL=postgresql://...` e rode:
    - `npm run db:generate`
    - `npm run db:push`
    - Opcional: `npm run db:extensions`

## Dicas

- Se estiver em memória, os dados se perdem ao reiniciar.
- Para demonstrar persistência, configure `DATABASE_URL` e utilize registro/login.