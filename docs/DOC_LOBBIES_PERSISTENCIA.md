# Lobbies e Persistência – Estado Atual

Este documento resume como os lobbies funcionam, como os dados são persistidos e como o sistema se comporta com ou sem banco de dados configurado.

## Modelos

- `Lobby`
  - Campos: `id`, `inviteCode @unique`, `createdAt`, `players (relation)`.
  - Função: representa uma sessão de jogo (pública ou privada) que organiza usuários.

- `LobbyPlayer`
  - Campos: `id`, `lobbyId (relation)`, `playerId`, `name`, `avatar?`, `joinedAt`.
  - Função: vínculo de jogadores a um `Lobby`.

Fonte: `prisma/schema.prisma`.

## Persistência

- Com `DATABASE_URL` definido:
  - Prisma conecta em Postgres.
  - Dados de lobby e jogadores são persistidos nas tabelas geradas pelo schema.

- Sem `DATABASE_URL`:
  - Fallback em memória para partes críticas (principalmente usuários e ranking).
  - Lobbies podem operar parcialmente em memória ou ser omitidos em produção.

## Endpoints Relevantes (conceito)

- `POST /api/lobbies`: cria lobby com `inviteCode` único.
- `POST /api/lobbies/join`: adiciona `LobbyPlayer` ao lobby existente.
- `GET /api/lobbies/:id`: retorna lobby e seus `players`.
- `DELETE /api/lobbies/:id`: encerra/limpa um lobby.

Nota: a API pode estar parcialmente implementada no momento, com foco prioritário em usuários e ranking.

## Fluxo Básico

1. Usuário realiza login (`/login`).
2. Cria ou entra em um lobby via convite.
3. Lobby mantém relação com jogadores e organiza partidas.
4. Ranking e conquistas são atualizados conforme partidas.

## Operação com e sem Banco

- Com banco:
  - Fluxos completos, persistência garantida.
  - Consultas otimizadas, possibilidade de índices e extensões (`pgcrypto`, `citext`).

- Sem banco:
  - Funcionalidades críticas usam memória para não quebrar a UX.
  - Dados não persistem entre reinícios.

## Boas Práticas

- Validar `inviteCode` único no DB.
- Evitar expor e-mail em respostas.
- Normalizar entradas com `citext` quando disponível.
- Adicionar observabilidade para lobbies (contagem, tempo ativo, jogadores).

## Próximos Passos Sugeridos

- Consolidar rotas de lobby (`create`, `join`, `get`, `delete`).
- Adicionar testes de integração.
- Criar métricas e painéis (ex.: `/api/lobbies/stats`).