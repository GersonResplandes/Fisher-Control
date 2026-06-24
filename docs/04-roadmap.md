# Roadmap De Implementação

## Princípio

A implementação será iniciada a partir desta base documental. Nenhuma fase do novo produto está concluída.

Trabalhar em fases pequenas, verificáveis e documentadas. Não avançar para CRUD de negócio antes da Fase 0.

## Fase 0 - Setup Profissional

Status: planejada.

Objetivo: criar a fundação técnica do repositório.

Entregas:

- Estrutura `api/`, `web/` e `docs/`.
- `package.json` da raiz sem workspaces.
- `package.json` independente em `api/`.
- `package.json` independente em `web/`.
- TypeScript strict.
- ESLint rigoroso.
- Prettier.
- Husky.
- lint-staged.
- commitlint.
- Conventional Commits.
- `.gitignore`.
- `.env.example` na raiz, API e web quando necessário.
- Docker Compose com PostgreSQL.
- Validação de ambiente com Zod.
- Logger estruturado.
- Handler global de erros.
- Base Express com `app.ts` e `server.ts`.
- Base Next.js com Tailwind.
- Swagger/OpenAPI preparado.
- Scripts `dev`, `build`, `lint`, `format`, `test`, `typecheck`.

Critério de conclusão:

- `npm run lint` passa.
- `npm run typecheck` passa.
- `npm test` passa, mesmo que com smoke tests.
- `npm run build` passa.
- Um commit ruim é bloqueado localmente.
- A aplicação falha ao iniciar se faltar env obrigatória.
- Não existe `console.log`.
- Não existe `any` sem justificativa.

## Fase 1 - Domínio, Banco E Contratos

Status: planejada.

Objetivo: modelar o núcleo do sistema antes dos endpoints.

Entregas:

- Entidades de domínio.
- Value objects.
- Regras puras de domínio.
- Prisma schema.
- Migrations.
- Seeds demonstrativos.
- Schemas Zod.
- OpenAPI inicial.
- Testes unitários de domínio.

Entidades iniciais:

- User.
- Species.
- Tank.
- Animal.
- IdentificationTag.
- TagAssignment.
- TagReadEvent.
- AuditLog.

Entidades previstas para fases futuras:

- HarvestEvent.
- HarvestItem.
- SpawningEvent.
- MonitoringRecord.

## Fase 2 - API Base, Autenticação E Usuários

Status: planejada.

Objetivo: criar base segura de acesso.

Entregas:

- `auth.controller.ts`.
- `auth.routes.ts`.
- `auth.service.ts`.
- Login.
- Logout.
- Hash de senha.
- Sessão opaca revogável.
- Middleware `authRequired`.
- Middleware `requireRoles`.
- CRUD de usuários.
- Testes de autenticação e autorização.

## Fase 3 - Cadastros Base

Status: planejada.

Objetivo: implementar espécies e tanques com a arquitetura correta.

Entregas:

- Módulo `species`.
- Módulo `tanks`.
- Controllers, routes, services e repositories separados.
- Filtros.
- Paginação.
- Validação com Zod.
- OpenAPI completo desses módulos.
- Testes de service e endpoints.

## Fase 4 - Gestão De Chips

Status: planejada.

Objetivo: tratar chip como recurso independente.

Entregas:

- Módulo `tags`.
- CRUD de chips.
- Status do chip.
- Busca por código.
- Histórico de uso do chip.
- Preservação de zeros à esquerda.
- Normalização como string.
- Testes de unicidade e normalização.

## Fase 5 - Associação Chip-Animal

Status: planejada.

Objetivo: implementar a regra mais importante do sistema.

Entregas:

- Módulo `tag-assignments`.
- Associar chip a animal.
- Encerrar associação.
- Reutilizar chip disponível.
- Impedir duas associações ativas do mesmo chip.
- Impedir dois chips ativos no mesmo animal.
- Buscar animal ativo por código de chip.

## Fase 6 - Fundação De Frontend E UX Operacional

Status: planejada.

Objetivo: criar a base visual e estrutural do frontend antes de expandir fluxos.

Entregas:

- Layout autenticado.
- Navegação operacional.
- Cliente HTTP tipado.
- Tratamento de sessão.
- Componentes de tabela, formulário, modal, toast, paginação, status e confirmação perigosa.
- Estados de carregamento, erro, vazio e sucesso.
- Responsividade básica em celular, tablet e desktop.
- Tela de login alinhada à identidade aquática operacional.

## Fase 7 - Gestão De Animais

Status: planejada.

Objetivo: implementar o ciclo de vida do animal.

Entregas:

- Módulo `animals`.
- CRUD de animais.
- Status do animal.
- Vínculo com espécie e tanque.
- Histórico individual.
- Filtros por espécie, tanque, status e chip.
- Tela de detalhes do animal.

## Fase 8 - Abate E Reaproveitamento De Chips

Status: planejada.

Objetivo: representar a operação real da piscicultura.

Entregas:

- Registrar abate individual.
- Registrar abate em lote.
- Informar se chip foi recuperado.
- Liberar chip recuperado.
- Marcar chip perdido ou inutilizado.
- Preservar histórico do animal abatido.
- Relatório de chips disponíveis e reaproveitados.

## Fase 9 - Desova E Monitoramento

Status: planejada.

Objetivo: recriar o núcleo científico do projeto original.

Entregas:

- Eventos de desova.
- Registro de peso antes e depois.
- Peso de ovos.
- Hormônio.
- Monitoramento de temperatura.
- Hora-grau.
- Histórico por animal.

## Fase 10 - Relatórios E Dashboard

Status: planejada.

Objetivo: transformar dados em leitura útil.

Entregas:

- Dashboard com dados reais.
- Indicadores de animais ativos.
- Indicadores de tanques.
- Indicadores de desova.
- Indicadores de chips disponíveis, atribuídos, perdidos e reutilizados.
- Relatório PDF de desova.
- Relatório de histórico do animal.
- Relatório de reutilização de chips.

## Fase 11 - Preparação Para Novas Tecnologias De Leitura

Status: planejada.

Objetivo: deixar a arquitetura pronta para automação futura.

Entregas:

- `TagReaderPort`.
- Implementação `ManualTagReader`.
- Registro de `TagReadEvent`.
- Normalização de código.
- Contrato preparado para futuras fontes.

Não implementar leitura real por celular nesta fase.

## Fase 12 - Hardening

Status: planejada.

Objetivo: elevar maturidade antes de portfólio.

Entregas:

- Testes de integração da API.
- Testes principais do frontend.
- Swagger completo.
- Guia de setup.
- Diagramas simples.
- Revisão de segurança.

## Fase 13 - Portfólio E Demonstração

Status: planejada.

Objetivo: tornar o projeto apresentável.

Entregas:

- README com narrativa.
- Prints ou GIFs.
- Dados seed demonstrativos.
- Roteiro de demonstração.
- Deploy opcional.
- Changelog.
- Release `1.0.0`.
