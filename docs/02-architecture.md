# Arquitetura Alvo

## Objetivo

Reconstruir o Fisher Control com dois projetos independentes no mesmo repositório:

```text
api/
web/
docs/
```

Não usar monorepo, npm workspaces ou pacotes internos compartilhados. A separação deve ser simples e explícita.

## Stack

API:

- Node.js.
- Express.
- TypeScript.
- PostgreSQL.
- Prisma para schema, migrations e seed.
- `pg` ou Prisma Client para persistência, escolhido na Fase 1.
- Zod.
- OpenAPI/Swagger gerado a partir de Zod.
- Pino.
- Helmet, CORS e rate limit.

Web:

- Next.js.
- React.
- TypeScript.
- Tailwind CSS.
- Cliente HTTP tipado.
- Tipos locais gerados ou derivados do OpenAPI da API.

Raiz do repositório:

- README.
- documentação.
- hooks locais.
- scripts agregadores, se forem úteis.
- sem `workspaces`.

## Estrutura Alvo Da API

```text
api/
  src/
    app.ts
    server.ts
    container.ts

    common/
      errors/
      http/
      logger/
      middleware/

    config/
      env.ts

    database/
      database.ts
      migrations/

    domain/
      entities/
      services/
      value-objects/
      errors/

    contracts/
      schemas/
      openapi.ts
      index.ts

    modules/
      auth/
        auth.controller.ts
        auth.routes.ts
        auth.service.ts
        auth.types.ts
        password-hasher.service.ts
        sessions.repository.ts
        session-token.service.ts

      users/
        users.controller.ts
        users.routes.ts
        users.service.ts
        users.repository.ts

      species/
        species.controller.ts
        species.routes.ts
        species.service.ts
        species.repository.ts

      tanks/
        tanks.controller.ts
        tanks.routes.ts
        tanks.service.ts
        tanks.repository.ts

      tags/
        tags.controller.ts
        tags.routes.ts
        tags.service.ts
        tags.repository.ts
        tag-code-normalizer.ts

      tag-assignments/
        tag-assignments.controller.ts
        tag-assignments.routes.ts
        tag-assignments.service.ts
        tag-assignments.repository.ts

      animals/
        animals.controller.ts
        animals.routes.ts
        animals.service.ts
        animals.repository.ts
```

## Responsabilidades Por Camada

### Routes

Arquivos `*.routes.ts`.

Responsabilidades:

- Declarar caminhos HTTP.
- Aplicar middlewares de autenticação, autorização e validação.
- Encaminhar a chamada para o controller.

Não devem conter regra de negócio.

### Controllers

Arquivos `*.controller.ts`.

Responsabilidades:

- Ler `body`, `params` e `query` já validados.
- Chamar services.
- Definir status HTTP.
- Retornar resposta.

Não devem:

- Fazer consulta de banco.
- Tomar decisão de domínio.
- Normalizar regras complexas.

### Services

Arquivos `*.service.ts`.

Responsabilidades:

- Orquestrar casos de uso.
- Aplicar regras de negócio.
- Chamar repositories.
- Lançar erros de aplicação.

### Repositories

Arquivos `*.repository.ts`.

Responsabilidades:

- Acessar banco de dados.
- Isolar SQL, Prisma ou outra tecnologia de persistência.
- Retornar dados no formato esperado pelo service.

Não devem decidir regra de negócio.

### Domain

Código puro.

Não deve importar Express, banco, Prisma, HTTP ou Next.

Responsabilidades:

- Entidades.
- Value objects.
- Regras centrais.
- Serviços puros de domínio.

## Contratos

Os schemas Zod ficam na API em `api/src/contracts`.

Eles devem ser fonte de verdade para:

- Validação de entrada.
- Tipos internos da API.
- OpenAPI.

O frontend não importa código da API. O frontend deve consumir tipos locais em `web/src/types/api.ts`, gerados ou derivados de `api/openapi.json`.

## Erros

Formato padrão:

```json
{
  "code": "VALIDATION_ERROR",
  "message": "Dados inválidos.",
  "details": {},
  "statusCode": 400,
  "path": "/species",
  "requestId": "uuid",
  "timestamp": "2026-06-24T00:00:00.000Z"
}
```

Toda resposta de erro deve passar pelo handler global.

## Autenticação

Modelo inicial:

- Bearer token opaco.
- Sessão persistida no banco.
- Banco guarda apenas hash do token.
- Logout revoga sessão.
- RBAC no backend.

Papéis iniciais:

- `ADMIN`.
- `MANAGER`.
- `OPERATOR`.

## Estratégia Para Chips

O chip não é identidade permanente do animal.

O animal deve ser identificado pelo próprio registro. O chip é recurso físico reutilizável associado historicamente ao animal por `TagAssignment`.

Entrada inicial:

- código digitado manualmente a partir do leitor externo.

Preparação futura:

- leitor que emula teclado;
- leitura NFC/celular se a tecnologia mudar;
- importação via API.

## Estrutura Alvo Do Frontend

```text
web/
  src/
    app/
    components/
      ui/
    features/
      auth/
      dashboard/
      animals/
      assignments/
      chips/
      species/
      tanks/
      users/
    lib/
    types/
```

O frontend deve ser um sistema operacional de piscicultura, não uma landing page.

