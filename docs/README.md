# Fisher Control - Planejamento

Este diretório é a fonte de verdade da nova versão profissional do Fisher Control.

O projeto original já cumpriu seu papel como entrega técnica/acadêmica no contexto IFMA/FAPEMA. Esta versão tem outro objetivo: transformar a ideia em um produto de portfólio, com regras reais de domínio, validação, segurança, documentação, testes, organização e capacidade de evolução.

## Estado Do Repositório

Atualmente o repositório contém a base documental e o README principal.

A aplicação deve ser criada a partir destes documentos, começando pela Fase 0 do roadmap.

## Documentos

- [00 - Visão do Produto](./00-product-vision.md)
- [01 - Modelo de Domínio](./01-domain-model.md)
- [02 - Arquitetura Alvo](./02-architecture.md)
- [03 - Padrões de Qualidade](./03-quality-standards.md)
- [04 - Roadmap de Implementação](./04-roadmap.md)
- [05 - Decisões Técnicas](./05-decisions.md)
- [06 - Fundação de Frontend e UX Operacional](./06-frontend-ux-foundation.md)
- [07 - Gestão de Animais](./07-animal-management.md)
- [Especificação de frontend](./fisher_control_spec.md)

## Ordem De Leitura Antes De Codar

1. Leia a visão do produto.
2. Leia o modelo de domínio.
3. Leia a arquitetura alvo.
4. Leia os padrões de qualidade.
5. Siga o roadmap sem pular a Fase 0.

## Princípios

1. Prometer apenas o que o sistema realmente entrega.
2. Separar identidade do animal da identidade do chip.
3. Tratar chips como recursos reutilizáveis.
4. Preparar a arquitetura para leitores futuros, sem priorizar automação agora.
5. Usar Zod como fonte de verdade para validação e documentação.
6. Separar controller, service, repository e domínio.
7. Bloquear código ruim antes do commit com hooks locais.
8. Manter documentação viva e sincronizada com o código.
9. Não deixar telas provisórias definirem a experiência final do produto.
