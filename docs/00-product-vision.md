# Visão do Produto

## Nome

Fisher Control

## Proposta

Sistema web para gestão de animais aquáticos em piscicultura, com identificação por código de chip lido em equipamento externo, controle de tanques, espécies, histórico individual, eventos de desova, abate, reaproveitamento de chips e relatórios.

## Origem

O Fisher Control nasce a partir de um projeto técnico/acadêmico desenvolvido e entregue no contexto IFMA/FAPEMA.

A versão atual será planejada e implementada como um produto de portfólio: mais organizada, auditável, testável e próxima de uma aplicação real de operação em piscicultura.

## Problema

Produtores e equipes técnicas precisam acompanhar animais aquáticos individualmente, mas o manejo envolve dados dispersos: código do chip, espécie, tanque, histórico de desova, peso, monitoramento de temperatura e reutilização dos chips após o abate.

O sistema deve tornar esse histórico confiável, consultável e auditável.

## Usuários

- Administrador: gerencia usuários, permissões, cadastros e regras.
- Técnico ou operador: registra animais, chips, desovas, movimentações e abates.
- Visitante ou leitor: consulta informações sem alterar dados críticos.

## Escopo Principal

- Autenticação e autorização.
- CRUD de usuários.
- CRUD de espécies.
- CRUD de tanques.
- Cadastro e gestão de chips.
- Cadastro e gestão de animais.
- Associação histórica entre chip e animal.
- Busca por código lido no leitor externo.
- Registro de desova.
- Monitoramento de temperatura e hora-grau.
- Registro de abate e recuperação de chip.
- Dashboard com dados reais.
- Relatório de desova em PDF.
- Documentação OpenAPI.

## Fora do Escopo Inicial

- Leitura direta do chip por celular.
- Integração automática com leitor físico atual.
- Aplicativo mobile nativo.
- IoT em tempo real.
- Machine learning.
- Multi-tenant complexo.

Esses pontos podem ser considerados no futuro, mas não devem contaminar o MVP.

## Identificação Por Chip

Os chips atuais não são lidos por celulares comuns. O fluxo correto é:

1. O chip é implantado no peixe.
2. Um leitor próprio lê o chip.
3. O leitor exibe um código único.
4. O usuário informa esse código no sistema.
5. O sistema localiza ou cadastra o animal associado.

Portanto, a entrada manual do código não é uma limitação do sistema. Ela é uma consequência técnica do tipo de chip usado.

## Preparação Para Futuro

Mesmo sem priorizar automação, a arquitetura deve permitir adaptação rápida se a piscicultura adotar chips lidos por celular.

Para isso:

- O animal não deve depender diretamente de um campo simples `rfidCode`.
- O chip deve ser uma entidade própria.
- A leitura do chip deve ser representada como uma fonte de identificação.
- O sistema deve aceitar diferentes tecnologias de leitura.
- A migração de dados deve preservar histórico de associações antigas.

## Objetivo de Portfólio

O projeto deve demonstrar:

- CRUD profissional.
- Modelagem de domínio.
- Regras de negócio reais.
- Validação em tempo de execução.
- Segurança básica bem aplicada.
- Documentação sincronizada.
- Testes relevantes.
- Arquitetura limpa.
- Uso disciplinado de Git, commits e versionamento.
- Fluxos de interface coerentes com a operação real.

O diferencial não é apenas cadastrar dados. O diferencial é mostrar que o sistema entende o ciclo de vida real do peixe e do chip.
