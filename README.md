# Fisher Control

Fisher Control é uma nova versão profissional, orientada a portfólio, de um projeto de piscicultura desenvolvido e entregue no contexto técnico/acadêmico do IFMA/FAPEMA.

A proposta atual é transformar a ideia validada em uma aplicação moderna para gestão de animais aquáticos, com identificação por chip, controle de tanques, espécies, histórico de vínculos, abate, desova, reaproveitamento de chips e relatórios.

## Estado Atual

O repositório está na etapa de fundação documental. A implementação será iniciada a partir destes documentos para manter arquitetura, regras de negócio e experiência de uso consistentes desde o começo.

Neste momento, a base versionada é:

```text
docs/
README.md
```

Ainda não existe aplicação executável, API, frontend, Docker, `package.json`, hooks ou scripts. A próxima etapa é iniciar a Fase 0 a partir da documentação.

## Direção Técnica

A implementação planejada deve seguir uma estrutura simples, sem monorepo e sem npm workspaces:

```text
api/  Node.js + Express + TypeScript
web/  Next.js + React + Tailwind
docs/ Planejamento, decisões e especificações
```

A API deve ser organizada por módulos, com separação explícita entre rotas, controllers, services e repositories.

## Documentação

- [Índice de planejamento](./docs/README.md)
- [Visão do produto](./docs/00-product-vision.md)
- [Modelo de domínio](./docs/01-domain-model.md)
- [Arquitetura alvo](./docs/02-architecture.md)
- [Padrões de qualidade](./docs/03-quality-standards.md)
- [Roadmap de implementação](./docs/04-roadmap.md)
- [Decisões técnicas](./docs/05-decisions.md)
- [Fundação de frontend e UX](./docs/06-frontend-ux-foundation.md)
- [Gestão de animais](./docs/07-animal-management.md)

## Próximo Passo

Iniciar pela Fase 0:

1. Criar estrutura `api/` e `web/`.
2. Configurar TypeScript, ESLint, Prettier, Husky, lint-staged e commitlint.
3. Configurar Docker Compose com PostgreSQL.
4. Criar validação de ambiente.
5. Criar base Express com middlewares globais.
6. Criar base Next.js com Tailwind.
7. Validar scripts locais de qualidade.

Nenhum CRUD de negócio deve ser implementado antes da Fase 0 estar concluída.
