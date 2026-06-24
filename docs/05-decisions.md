# Decisões Técnicas

As decisões abaixo orientam a implementação da versão profissional do Fisher Control. Elas não significam que já existe aplicação pronta neste repositório.

## ADR-001 - Sem GitHub Actions No Início

Status: aceita.

O projeto não usará CI com GitHub Actions na fase inicial.

Motivo:

- Manter o fluxo simples.
- Priorizar qualidade local com hooks.
- Evitar configurar automação antes de estabilizar arquitetura.

Consequências:

- Husky será obrigatório.
- lint-staged será obrigatório.
- commitlint será obrigatório.
- O README deve ensinar a rodar verificações manualmente.

## ADR-002 - Sem Monorepo E Sem Workspaces

Status: aceita.

O repositório terá dois projetos independentes:

```text
api/
web/
```

Não usar npm workspaces, `packages/*` ou imports internos compartilhados.

Motivo:

- O usuário quer uma estrutura mais normal e fácil de entender.
- O projeto deve ser simples de apresentar em portfólio.
- A API deve ser a fonte de verdade dos contratos.

Consequências:

- `api/package.json` e `web/package.json` são independentes.
- O frontend não importa arquivos da API.
- Tipos do frontend devem ser locais ou gerados a partir de OpenAPI.

## ADR-003 - Express Em Vez De NestJS

Status: aceita.

A API será Node.js com Express e TypeScript.

Motivo:

- Reduzir abstração de framework.
- Evidenciar melhor controllers, routes, services e repositories.
- Facilitar entendimento do fluxo HTTP.

Consequências:

- Guards, pipes e filters serão middlewares.
- Controllers serão classes ou funções finas.
- Rotas serão declaradas explicitamente em `*.routes.ts`.

## ADR-004 - Organização Por Módulo

Status: aceita.

A API será organizada por feature dentro de `api/src/modules`.

Exemplo:

```text
modules/animals/
  animals.controller.ts
  animals.routes.ts
  animals.service.ts
  animals.repository.ts
```

Motivo:

- Evita pastas globais gigantes como `controllers/`, `services/` e `repositories/`.
- Mantém cada contexto próximo.
- Facilita manutenção e leitura.

Consequências:

- Cada módulo expõe suas próprias rotas.
- Services não devem depender de Express.
- Repositories não devem conter regra de negócio.

## ADR-005 - Chip Não É Identidade Permanente Do Animal

Status: aceita.

O chip será modelado como `IdentificationTag`, separado de `Animal`.

Motivo:

Na operação real, peixes podem ser abatidos, os chips coletados e implantados em outros peixes.

Consequências:

- `Animal` não deve ter apenas um campo `rfidCode`.
- A relação entre animal e chip deve ser histórica.
- `TagAssignment` representa associação ativa e histórico.

## ADR-006 - Reutilização De Chip Preserva Histórico

Status: aceita.

Reutilizar chip cria uma nova associação. A associação anterior deve ser encerrada, nunca sobrescrita.

Consequências:

- Relatórios de períodos anteriores continuam corretos.
- Busca operacional usa apenas associação ativa.
- Histórico do chip mostra todos os animais anteriores.

## ADR-007 - Preparar Para Leitura Por Celular Sem Implementar Agora

Status: aceita.

A arquitetura terá uma abstração de leitura de chip, mas o MVP usará entrada manual.

Motivo:

Os chips atuais exigem leitor próprio. Celulares comuns não leem essa frequência.

Consequências:

- Criar `TagReaderPort` quando o fluxo de leitura for implementado.
- Registrar origem da leitura em `TagReadEvent`.
- Evitar acoplamento da busca ao formulário manual.

## ADR-008 - PostgreSQL E Prisma

Status: aceita.

Usar PostgreSQL com Prisma para schema, migrations e seed.

A camada de runtime pode usar Prisma Client ou `pg`; a escolha final deve ser feita na Fase 1 considerando clareza, testes e controle transacional.

Motivo:

O domínio possui relações fortes, histórico, constraints e regras de unicidade parcial.

## ADR-009 - Zod Como Fonte De Verdade

Status: aceita.

Zod será usado como fonte principal para validação de entrada, inferência de tipos da API e geração de OpenAPI.

Consequências:

- Schemas devem ser bem nomeados.
- OpenAPI deve ser gerado a partir dos schemas.
- Mudança de contrato deve passar pelos schemas.

## ADR-010 - Sessões Opacas Revogáveis

Status: aceita.

Usar tokens opacos de sessão no formato Bearer, armazenando apenas hash do token no banco.

Motivo:

Logout real e revogação imediata são mais simples e auditáveis com sessão persistida do que com JWT stateless.

Consequências:

- `SESSION_SECRET` obrigatório.
- Token puro só aparece no login.
- Banco guarda apenas hash.
- Logout revoga sessão.

## ADR-011 - Índices Parciais Para Associação Ativa

Status: aceita.

Usar índices únicos parciais no PostgreSQL para garantir:

- um chip tem no máximo uma associação ativa;
- um animal tem no máximo um chip ativo.

Motivo:

Essa regra é central e deve ser protegida no banco, não apenas na aplicação.

## ADR-012 - Frontend Operacional, Não Landing Page

Status: aceita.

O frontend deve nascer como sistema operacional de piscicultura.

Consequências:

- Primeira tela autenticada deve favorecer operação.
- Cadastros não devem dominar a experiência.
- Componentes devem priorizar clareza, velocidade e baixa carga cognitiva.
- O visual deve remeter ao ambiente aquático sem virar decoração vazia.
