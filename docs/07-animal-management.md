# Gestão De Animais

## Status

Planejada.

Este documento define a intenção da fase de Gestão de Animais. Ele não descreve implementação existente.

## Objetivo

Implementar o ciclo de vida do animal sem confundir animal com chip.

## Entidade Animal

Campos planejados:

- `id`.
- `speciesId`.
- `currentTankId`.
- `sex`.
- `birthDate`.
- `motherAnimalId`.
- `notes`.
- `status`.
- `createdAt`.
- `updatedAt`.

Status planejados:

- `ACTIVE`.
- `SLAUGHTERED`.
- `DEAD`.
- `SOLD`.
- `TRANSFERRED`.
- `ARCHIVED`.

## Regras

- Novo animal deve iniciar como ativo.
- Animal ativo pode estar sem chip.
- Animal pode ter no máximo um chip ativo.
- Animal abatido, morto, vendido, transferido ou arquivado deve ficar protegido contra alterações operacionais indevidas.
- Encerrar o ciclo do animal não deve apagar histórico.
- A busca por chip deve resolver a associação ativa, não um campo direto no animal.

## Endpoints Planejados

```text
POST   /animals
GET    /animals
GET    /animals/:id
PATCH  /animals/:id
DELETE /animals/:id
GET    /animals/:id/tag-history
```

## Tela Planejada

Listagem:

- busca;
- filtro por espécie;
- filtro por tanque;
- filtro por status;
- filtro por presença de chip;
- paginação;
- ações de ver, editar e remover quando permitido.

Detalhe:

- dados principais;
- chip ativo;
- histórico de chips;
- eventos futuros de abate, desova e monitoramento.

## Fora Do Escopo Desta Fase

- Abate completo.
- Desova.
- Monitoramento.
- Relatório final.
- Leitura automática por celular.

