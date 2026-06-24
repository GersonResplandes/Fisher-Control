# Padrões de Qualidade

## Objetivo

Garantir que o projeto tenha qualidade profissional desde a fase 0, antes da implementação das funcionalidades de negócio.

## Fase 0 Obrigatória

A primeira entrega do projeto deve ser o setup técnico.

Nada de CRUD antes de:

- TypeScript estrito.
- ESLint rigoroso.
- Prettier configurado.
- Husky funcionando.
- lint-staged funcionando.
- commitlint funcionando.
- Logger estruturado.
- Validação de ambiente.
- Estrutura de pastas definida.
- Tratamento global de erros.
- Swagger preparado.
- Scripts padronizados.

## Qualidade de Código

### ESLint

Regras obrigatórias:

- Evitar `any`.
- Proibir `console.log`.
- Proibir variáveis não utilizadas.
- Proibir imports não utilizados.
- Padronizar ordenação de imports.
- Exigir tratamento explícito de promises.
- Evitar código morto.
- Evitar casts desnecessários.

O uso de `any` só pode ocorrer com justificativa pontual e preferencialmente com comentário curto.

### Prettier

O projeto deve ter `.prettierrc`.

Objetivo:

- Formatação padronizada.
- Diferenças de estilo não devem virar discussão.
- O código deve ser visualmente consistente.

### Importações

Importações devem seguir ordem previsível:

1. Bibliotecas externas.
2. Pacotes internos.
3. Imports relativos.
4. Tipos, quando separados.

## Hooks De Commit

Não haverá GitHub Actions inicialmente.

Qualidade será bloqueada localmente com:

- Husky.
- lint-staged.
- commitlint.

Hooks sugeridos:

### pre-commit

Executar em arquivos staged:

- ESLint com fix quando seguro.
- Prettier.
- Testes rápidos relacionados, quando viável.

### commit-msg

Validar Conventional Commits.

Exemplos válidos:

```text
feat(api): add tag assignment domain
fix(web): preserve leading zeros in tag code
docs(product): clarify external reader flow
test(api): cover chip reuse after harvest
```

### pre-push

Opcional, mas recomendado:

- Typecheck completo.
- Testes unitários.
- Testes de integração principais.

## Validação E Tipagem

### Zod

Zod deve validar:

- Body.
- Params.
- Query.
- Variáveis de ambiente.
- Payloads internos quando entram por fronteiras externas.

Validação deve acontecer antes de executar caso de uso.

### Zod Como Fonte De Verdade

Schemas Zod devem ser a base para:

- Validação.
- Tipos inferidos.
- Documentação OpenAPI.

Evitar duplicar schema em DTO manual, Swagger manual e tipo separado quando o Zod puder ser a origem comum.

### OpenAPI

Usar `@asteasolutions/zod-to-openapi` ou solução equivalente para gerar schemas OpenAPI a partir de Zod.

Swagger deve incluir:

- Descrição dos endpoints.
- Exemplos de request.
- Exemplos de response.
- Possíveis erros.
- Regras de autenticação.

## Variáveis De Ambiente

Criar `env.ts`.

A aplicação não deve iniciar se faltar variável obrigatória.

Obrigatório:

- `.env.example`
- Validação com Zod.
- Tipagem do objeto de ambiente.
- Separação clara entre variáveis públicas e privadas.

Nunca:

- Commitar `.env`.
- Commitar secrets em arquivo de teste.
- Usar fallback inseguro para segredo.

## Logs E Observabilidade

Usar logger estruturado.

Preferência: Pino.

Níveis:

- `info`
- `warn`
- `error`

Cada log relevante deve conter:

- `requestId`
- módulo ou serviço.
- ação executada.
- identificador da entidade quando seguro.
- stack trace em erro inesperado.

Nunca logar:

- Senha.
- Token.
- Cookie completo.
- URI de banco com credenciais.
- Dados sensíveis desnecessários.

`console.log` deve ser proibido por lint.

## Segurança

Regras mínimas:

- Validação obrigatória de todos os inputs.
- Rate limit para login e endpoints sensíveis.
- Headers de segurança quando aplicável.
- Hash de senha com algoritmo adequado.
- Tokens nunca devem ir para logs.
- Erros não devem vazar stack trace em produção.
- Autorização sempre no backend.

### Auditoria De Dependências

Executar `npm audit` periodicamente e antes de releases importantes.

Regras:

- Não usar `npm audit fix --force` sem revisão técnica.
- Separar vulnerabilidades de runtime das vulnerabilidades de ferramentas de desenvolvimento.
- Registrar exceções temporárias em ADR quando a correção segura ainda não existir.
- Priorizar atualizações compatíveis e validadas pela bateria local do projeto.

## Tratamento De Erros

Criar erro padrão, por exemplo `AppError`.

Formato de resposta:

```json
{
  "statusCode": 400,
  "code": "VALIDATION_ERROR",
  "message": "Dados inválidos.",
  "details": {}
}
```

Obrigatório:

- Handler global de erros.
- Códigos de erro de domínio.
- Mensagens claras.
- Respostas consistentes.

## Testes

Prioridade:

1. Regras de domínio.
2. Casos de uso.
3. Endpoints críticos.
4. Fluxos principais do front.

Casos que devem ter teste:

- Chip não pode ser associado a dois animais ativos.
- Animal não pode ter dois chips ativos.
- Abate encerra associação.
- Chip recuperado fica disponível.
- Código com zero à esquerda é preservado.
- Busca por código retorna animal ativo correto.
- Senha nunca é salva em texto puro.
- Usuário sem permissão não altera dados críticos.

## Qualidade De Frontend

O front deve ser tratado como parte do produto, não apenas como cliente de teste da API.

Regras:

- Toda tela de negócio deve ter estados de carregamento, erro, vazio e sucesso.
- Ações destrutivas ou irreversíveis devem exigir confirmação clara.
- Formulários devem mostrar campos obrigatórios, validação e feedback de erro sem depender apenas do backend.
- Tabelas devem ter colunas previsíveis, paginação e leitura confortável em desktop.
- Fluxos principais devem funcionar em viewport mobile básico sem sobreposição de texto ou controles.
- Componentes repetidos devem ser extraídos quando a repetição ficar clara.
- A interface não deve esconder regra de negócio: bloqueios importantes devem ser visíveis quando possível, mas a autorização real continua no backend.
- Telas criadas apenas para validar endpoints devem ser marcadas no planejamento como provisórias ou substituídas na fase de UX.

## Versionamento

Usar Conventional Commits e Versionamento Semântico.

Versões:

- `0.x`: desenvolvimento antes do MVP estável.
- `1.0.0`: MVP completo, documentado e demonstrável.

## Git

Obrigatório:

- `.gitignore` bem configurado.
- Nunca versionar `node_modules`.
- Nunca versionar `.env`.
- Nunca versionar arquivos de build.
- Nunca versionar dumps com dados reais.

## Referências

- Husky: https://typicode.github.io/husky/get-started.html
- lint-staged: https://github.com/lint-staged/lint-staged
- zod-to-openapi: https://github.com/asteasolutions/zod-to-openapi
- Pino: https://github.com/pinojs/pino
