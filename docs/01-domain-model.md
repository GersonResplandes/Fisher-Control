# Modelo de Domínio

## Ideia Central

Animal e chip não são a mesma coisa.

O animal tem histórico biológico próprio. O chip é um identificador físico reutilizável. Quando um peixe é abatido, o chip pode ser coletado e implantado em outro peixe. Portanto, o código do chip não pode ser tratado como identidade permanente do animal.

## Entidades Principais

### User

Representa uma pessoa com acesso ao sistema.

Campos principais:

- `id`
- `name`
- `email`
- `passwordHash`
- `role`
- `status`
- `createdAt`
- `updatedAt`

Regras:

- Email deve ser único.
- Senha nunca deve ser armazenada em texto puro.
- Permissões devem ser aplicadas no backend, não apenas na interface.

### Species

Representa uma espécie de animal aquático.

Campos principais:

- `id`
- `name`
- `scientificName`
- `description`
- `createdAt`
- `updatedAt`

Regras:

- Nome deve ser obrigatório.
- Nome pode ser único se fizer sentido operacional.

### Tank

Representa um tanque físico.

Campos principais:

- `id`
- `name`
- `capacity`
- `width`
- `height`
- `status`
- `notes`
- `createdAt`
- `updatedAt`

Regras:

- Capacidade deve ser positiva.
- Tanque pode estar ativo, em manutenção ou inativo.

### Animal

Representa o peixe ou animal aquático acompanhado pelo sistema.

Campos principais:

- `id`
- `speciesId`
- `currentTankId`
- `sex`
- `birthDate`
- `motherAnimalId`
- `notes`
- `status`
- `createdAt`
- `updatedAt`

Status sugeridos:

- `ACTIVE`
- `SLAUGHTERED`
- `DEAD`
- `SOLD`
- `TRANSFERRED`
- `ARCHIVED`

Regras:

- Animal deve ter espécie.
- Animal ativo deve poder ter no máximo um chip ativo associado.
- O histórico do animal não deve ser apagado após abate.
- Matriz ou mãe deve ser opcional.

### IdentificationTag

Representa o chip físico.

Campos principais:

- `id`
- `code`
- `technology`
- `status`
- `createdAt`
- `updatedAt`

Tecnologias sugeridas:

- `EXTERNAL_RFID_READER`
- `NFC_MOBILE`
- `QR_CODE`
- `MANUAL_CODE`
- `OTHER`

Status sugeridos:

- `AVAILABLE`
- `ASSIGNED`
- `LOST`
- `DAMAGED`
- `RETIRED`

Regras:

- Código deve ser string.
- Código não deve ser convertido para número.
- Zeros à esquerda devem ser preservados.
- Um chip só pode ter uma associação ativa por vez.
- Chip recuperado após abate pode voltar para `AVAILABLE`.

### TagAssignment

Representa a associação histórica entre chip e animal.

Campos principais:

- `id`
- `tagId`
- `animalId`
- `assignedAt`
- `unassignedAt`
- `assignReason`
- `unassignReason`
- `createdByUserId`

Motivos de associação:

- `INITIAL_IMPLANT`
- `REIMPLANT_AFTER_HARVEST`
- `REPLACEMENT`
- `MANUAL_CORRECTION`

Motivos de desassociação:

- `HARVEST`
- `DEATH`
- `LOST_TAG`
- `DAMAGED_TAG`
- `MANUAL_CORRECTION`

Regras:

- Não pode existir mais de uma associação ativa para o mesmo chip.
- Não pode existir mais de uma associação ativa para o mesmo animal.
- Reutilização do chip deve criar nova associação, nunca sobrescrever a anterior.
- Consultar um código deve retornar o animal da associação ativa.

### HarvestEvent

Representa o abate de um ou mais animais.

Campos principais:

- `id`
- `date`
- `responsibleUserId`
- `notes`
- `createdAt`

### HarvestItem

Representa cada animal abatido dentro de um evento de abate.

Campos principais:

- `id`
- `harvestEventId`
- `animalId`
- `tagRecovered`
- `recoveredTagId`
- `finalWeight`
- `notes`

Regras:

- Ao registrar abate, o animal deve sair de `ACTIVE`.
- Se o chip for recuperado, a associação ativa deve ser encerrada e o chip deve voltar para `AVAILABLE`.
- Se o chip não for recuperado, o chip deve ser marcado como `LOST` ou `RETIRED`.

### SpawningEvent

Representa um evento de desova.

Campos principais:

- `id`
- `animalId`
- `date`
- `weightBefore`
- `weightAfter`
- `eggWeight`
- `hormoneDosage`
- `hormoneAppliedAt`
- `responsibleUserId`
- `notes`
- `createdAt`
- `updatedAt`

Regras:

- Pesos não podem ser negativos.
- Peso depois da desova não deve ser maior que peso antes sem justificativa.
- Evento deve pertencer a um animal existente.
- O responsável deve ser registrado.

### MonitoringRecord

Representa uma medição durante a desova.

Campos principais:

- `id`
- `spawningEventId`
- `recordedAt`
- `temperature`
- `hourDegree`

Regras:

- Temperatura deve ser numérica.
- Registro deve pertencer a uma desova.

### TagReadEvent

Representa uma tentativa de leitura ou entrada de código.

Campos principais:

- `id`
- `rawValue`
- `normalizedValue`
- `source`
- `technology`
- `readAt`
- `userId`

Fontes sugeridas:

- `MANUAL_INPUT`
- `EXTERNAL_READER_TYPED`
- `MOBILE_NFC`
- `API_IMPORT`

Regras:

- A busca por chip deve passar por normalização.
- A origem da leitura deve ser registrada quando relevante.
- Essa entidade prepara o sistema para leitores futuros sem alterar o núcleo do domínio.

### AuditLog

Representa ações importantes no sistema.

Campos principais:

- `id`
- `actorUserId`
- `action`
- `entity`
- `entityId`
- `requestId`
- `metadata`
- `createdAt`

Regras:

- Não armazenar senha, token ou segredo.
- Registrar contexto suficiente para auditoria.

## Consultas Importantes

- Buscar animal ativo por código de chip.
- Ver histórico de um animal.
- Ver histórico de uso de um chip.
- Listar chips disponíveis.
- Listar animais sem chip ativo.
- Listar animais abatidos.
- Gerar relatório de desova.
- Gerar relatório de reutilização de chips.

## Regra Mais Importante

O código do chip identifica o chip, não o animal.

O animal atualmente associado ao chip é descoberto pela associação ativa em `TagAssignment`.
