# Fisher Control — Especificação de Frontend

> **Versão:** 1.2 — FINAL | **Data:** 2026-06-08  
> **Status:** Referência de UX para a implementação planejada. Não descreve uma aplicação já disponível neste repositório.

> Este documento deve orientar design, fluxo e componentes quando a Fase 6 for iniciada novamente. Em caso de conflito com `02-architecture.md`, `04-roadmap.md` ou `05-decisions.md`, os documentos de arquitetura, roadmap e ADRs prevalecem.

---

## Registro de Decisões Finais

Todas as 14 lacunas foram respondidas. Nenhuma suposição remanescente.

| ID  | Decisão final                                                                                                                                                                                                                                  |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| L1  | Volume inicial: dezenas a centenas. Arquitetura preparada para milhares: paginação obrigatória de **20 itens/página**, busca e filtros sempre server-side, nunca carregar tudo de uma vez. Backend já trabalha com limite máximo de 100 itens. |
| L2  | Código de chip é **string normalizada**: A–Z, 0–9, hífen; máx **64 caracteres**; uppercase; sem espaços. Nunca número. O novo contrato deve nascer alinhado em 64 caracteres.                                             |
| L3  | Sem limite de vínculos históricos por animal. UI trata histórico como **lista potencialmente longa**: paginado, ordenado por data desc, com agrupamento temporal opcional.                                                                     |
| L4  | Tanque pode conter espécies diferentes. Filtros cobrem: tanque + espécie + status + chip.                                                                                                                                                      |
| L5  | Sistema totalmente responsivo: desktop (admin/dados), tablet (operação campo), mobile (consulta + ações simples).                                                                                                                              |
| L6  | Sem recuperação de senha e sem SSO no MVP. Login/logout com sessão é suficiente.                                                                                                                                                               |
| L7  | Monitoramento pertence ao `SpawningEvent`. `SpawningEvent` pertence ao animal. Tanque e espécie são contexto, não chave primária do monitoramento.                                                                                             |
| L8  | Abate em lote: seleção manual com atalho por tanque, revisão obrigatória antes da confirmação.                                                                                                                                                 |
| L9  | **Exportação CSV** desejável em relatórios. Importação fora do MVP.                                                                                                                                                                            |
| L10 | Sem notificações em tempo real no MVP. Refresh manual.                                                                                                                                                                                         |
| L11 | Espécie: nome popular, nome científico, descrição/observações. Sem parâmetros biológicos no MVP.                                                                                                                                               |
| L12 | Tanque: nome, capacidade, **largura, altura**, status, observações. Localização em `observações` por ora.                                                                                                                                      |
| L13 | Sem fotos/documentos no MVP.                                                                                                                                                                                                                   |
| L14 | Assumir que o leitor externo mostra o código, mas o sistema não depende disso. A confirmação operacional é sempre do Fisher Control.                                                                                                           |
| CSS | Tailwind CSS com `tailwind.config.ts` como fonte única de tokens. Sem valores literais no JSX.                                                                                                                                                 |

---

## Parte 1 — Riscos de Produto, UX e Usabilidade

### 🔴 Críticos

**R1 — Confusão chip ≠ animal**  
Toda tela onde chip e animal coexistem deve tornar a distinção explícita e permanente. Chip é um instrumento físico temporário; a identidade é do animal.

**R2 — Perda acidental de vínculo ativo**  
Encerramento de vínculo é irreversível na prática. Modal obrigatório com resumo dos dados antes da confirmação. Enter não confirma.

**R3 — Input alfanumérico com erro silencioso**  
`0963` e `963` são chips distintos se cadastrados separadamente. A normalização (uppercase, sem espaços) deve ser transparente: o operador vê o código normalizado antes de qualquer ação. O sistema nunca colapsa zeros à esquerda.

**R4 — Abate em lote irreversível em escala**  
Um filtro errado pode marcar dezenas de animais como abatidos. A etapa de revisão nominada (animal por animal) é não negociável antes da confirmação final.

### 🟡 Moderados

**R5 — Histórico longo sem paginação**  
Um chip reutilizado muitas vezes ao longo de anos gera histórico extenso. Paginação e agrupamento temporal obrigatórios na UI de histórico.

**R6 — Exportação CSV sem filtro explícito**  
O operador deve saber exatamente o que está exportando antes de baixar o arquivo. O botão de export deve refletir os filtros ativos.

**R7 — Alinhamento do contrato de chip**  
O novo contrato deve aceitar 64 caracteres. A UI deve usar 64 como limite para evitar dados que a lógica de domínio rejeite.

---

## Parte 2 — Arquitetura de Informação

### 2.1 Hierarquia de Entidades

```
Sistema
├── Usuários & Permissões
│   └── [Perfil: Admin | Técnico | Visitante]
│
├── Cadastros Base
│   ├── Espécies          (nome popular, nome científico, descrição)
│   ├── Tanques           (nome, capacidade, largura, altura, status, observações)
│   └── Chips             (código: string A-Z0-9- ≤64 chars, status)
│
├── Animais               (entidade central)
│   ├── Dados biológicos + espécie + tanque
│   ├── Vínculo atual com chip (0 ou 1 ativo)
│   └── Histórico de vínculos  (lista paginada, sem limite)
│
├── Vínculos Chip-Animal
│   ├── Criação
│   └── Encerramento (motivo obrigatório)
│
└── Eventos Operacionais
    ├── Abate Individual
    ├── Abate em Lote          [futuro — especificado]
    ├── SpawningEvent (Desova) [futuro]
    │   └── MonitoringRecord   [futuro]
    └── Exportações CSV        [relatórios]
```

### 2.2 Navegação Principal (Sidebar)

```
[Logo Fisher Control]

─── Operação ──────────────
  Painel Operacional
  Vínculos Chip-Animal

─── Cadastros ─────────────
  Animais
  Espécies
  Tanques
  Chips

─── Eventos ───────────────
  Abate Individual
  Abate em Lote        [EM BREVE — sem link no menu]
  Desova               [EM BREVE — sem link no menu]
  Monitoramento        [EM BREVE — sem link no menu]

─── Análise ───────────────
  Relatórios

─── Administração ─────────
  Usuários & Permissões  [Admin]

─── ────────────────────────
  [Avatar] Nome do usuário
  Sair
```

---

## Parte 3 — Fluxos Principais

### Fluxo 1 — Registrar animal e vincular chip

```
Cadastrar Animal → espécie, tanque, observações
  └─► Animal criado (status: sem chip)
        └─► Vínculos → Nova Associação
              └─► ChipCodeInput → lookup automático
                    ├─ Chip livre → confirmar → vínculo ativo ✓
                    └─ Chip vinculado → bloquear + mostrar animal atual
```

### Fluxo 2 — Abate individual com recuperação de chip

```
Abate Individual (Etapa 1: identificar)
  └─► ChipCodeInput → lookup → confirmar animal exibido
        └─► (Etapa 2: dados) data, peso, pergunta sobre chip
              └─► (Etapa 3: confirmação) resumo → confirmar abate
                    └─► Animal abatido + vínculo encerrado (Abate) + chip recuperado ou não
```

### Fluxo 3 — Abate em lote

```
(Etapa 1) Filtrar por tanque → selecionar animais individualmente ou "selecionar todos"
  └─► (Etapa 2) Data + observações gerais
        └─► (Etapa 3) Revisão: por animal — recuperar chip? (sim/não)
              └─► (Etapa 4) Confirmação nominada → processar → relatório de resultado
```

### Fluxo 4 — Consultar histórico de chip

```
Chips → buscar por código
  └─► Detalhe do chip → status atual + histórico paginado
        └─► Cada linha: animal, espécie, período, motivo → clicável para detalhe do animal
```

---

## Parte 4 — Especificação de Telas

---

### Tela 01 — Login

**Rota:** `/login` | **Usuário-alvo:** Todos

#### Wireframe

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│              [Logo Fisher Control]                       │
│              Sistema de Gestão de Piscicultura           │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │  E-mail *                                        │   │
│  │  [________________________________]              │   │
│  │                                                  │   │
│  │  Senha *                                    [👁]  │   │
│  │  [________________________________]              │   │
│  │                                                  │   │
│  │  [        Entrar no sistema        ]             │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  Fisher Control v1.0                                     │
└──────────────────────────────────────────────────────────┘
```

> Sem link "Esqueci minha senha" (L6: sem recuperação de senha no MVP).

#### Estados

| Estado             | Comportamento                                               |
| ------------------ | ----------------------------------------------------------- |
| Carregando         | Botão desabilitado + spinner inline, campos readonly        |
| Erro de credencial | Banner: "E-mail ou senha incorretos" — sem especificar qual |
| Erro de rede       | Banner: "Não foi possível conectar. Tente novamente."       |
| Sucesso            | Redirect para rota solicitada originalmente ou `/dashboard` |

#### Acessibilidade

- `autofocus` no campo e-mail
- Enter no campo senha submete o form
- Mensagem de erro com `role="alert"` + `aria-live="polite"`

#### Critérios de Aceite

- [ ] Campos vazios bloqueiam submit
- [ ] Erro de credencial não revela qual campo falhou
- [ ] Redirect pós-login respeita a rota originalmente solicitada
- [ ] Enter no campo senha submete

---

### Tela 02 — Painel Operacional

**Rota:** `/dashboard` | **Usuário-alvo:** Técnico (primário), Admin

#### Wireframe Desktop

```
┌─ Sidebar ─────┐ ┌─ Conteúdo ───────────────────────────────────────────┐
│               │ │  Painel Operacional                  [Atualizar ↻]    │
│               │ │  Atualizado: 11:54                                     │
│               │ │                                                        │
│               │ │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐  │
│               │ │  │ 142          │ │ 138          │ │ 8            │  │
│               │ │  │ Animais      │ │ Chips        │ │ Tanques      │  │
│               │ │  │ ativos       │ │ vinculados   │ │ ativos       │  │
│               │ │  │ 4 sem chip   │ │ 12 livres    │ │              │  │
│               │ │  └──────────────┘ └──────────────┘ └──────────────┘  │
│               │ │                                                        │
│               │ │  ┌─ Consultar Chip ──────────────────────────────┐   │
│               │ │  │  [ChipCodeInput __________]   [Consultar]     │   │
│               │ │  │  ┌─ ResultCard ─────────────────────────────┐ │   │
│               │ │  │  │ ● Chip 007 · Animal #42 · Tanque T-03    │ │   │
│               │ │  │  │ Tilápia do Nilo · desde 12/04/2025       │ │   │
│               │ │  │  │ [Ver animal]  [Encerrar vínculo]         │ │   │
│               │ │  │  └──────────────────────────────────────────┘ │   │
│               │ │  └──────────────────────────────────────────────┘   │
│               │ │                                                        │
│               │ │  ┌─ Animais sem chip (4) ─────────────────────────┐  │
│               │ │  │  #38 Tambaqui · T-01 · desde 20/05/2025        │  │
│               │ │  │  #41 Tilápia  · T-02 · desde 22/05/2025        │  │
│               │ │  │  #44 Pacu     · T-01 · desde 01/06/2025        │  │
│               │ │  │  #47 Tilápia  · T-03 · desde 05/06/2025        │  │
│               │ │  │                         [Ver todos os animais]  │  │
│               │ │  └──────────────────────────────────────────────────┘  │
└───────────────┘ └──────────────────────────────────────────────────────┘
```

#### Wireframe Mobile

```
┌────────────────────────┐
│ [≡] Fisher Control     │
├────────────────────────┤
│ Consultar chip         │
│ [___________] [Buscar] │
│ ┌──────────────────────┐│
│ │ ● Chip 007           ││
│ │ Animal #42 Tilápia   ││
│ │ Tanque T-03          ││
│ │ [Ver] [Encerrar]     ││
│ └──────────────────────┘│
├────────────────────────┤
│ 142 animais · 4 sem chip│
│ 138 vinculados · 12 livres│
├────────────────────────┤
│ ⚠ 4 animais sem chip   │
│ #38 Tambaqui · T-01    │
│ #41 Tilápia  · T-02    │
│ + 2 mais  [Ver todos →]│
└────────────────────────┘
```

#### ResultCard — 5 estados

| Estado      | Borda            | Fundo               | Conteúdo                               |
| ----------- | ---------------- | ------------------- | -------------------------------------- |
| Vinculado   | `border-success` | `bg-success-subtle` | animal + tanque + ações                |
| Disponível  | `border-info`    | `bg-info-subtle`    | "Disponível" + link para criar vínculo |
| Recuperado  | `border-warning` | `bg-warning-subtle` | "Recuperado recentemente, disponível"  |
| Danificado  | `border-danger`  | `bg-danger-subtle`  | "Inoperante — não pode ser vinculado"  |
| Inexistente | `border-danger`  | `bg-danger-subtle`  | "Código [X] não cadastrado"            |

#### Critérios de Aceite

- [ ] Consulta de chip cobre todos os 5 estados com visual distinto
- [ ] Zeros à esquerda e letras maiúsculas preservados na exibição
- [ ] Animais sem chip: máximo 4 linhas + link "ver todos"
- [ ] Mobile: widget de consulta de chip está acima das métricas

---

### Tela 03 — Vínculos Chip-Animal

**Rota:** `/vinculos` | **Usuário-alvo:** Técnico, Admin

#### Wireframe Desktop

```
┌──────────────────────────────────────────────────────────────────────┐
│  Vínculos Chip-Animal                            [+ Nova Associação] │
│                                                                       │
│  Tanque: [Todos ▼]  Espécie: [Todas ▼]  Status: [Ativos ▼]          │
│  Código do chip: [ChipCodeInput _______]  [Buscar]  [Limpar]         │
│                                                                       │
│  138 vínculos · Página 1 de 7                                        │
│                                                                       │
│  ┌────────────┬──────────────────┬──────────┬────────────┬────────┐  │
│  │ Chip       │ Animal           │ Espécie  │ Tanque     │ Ações  │  │
│  ├────────────┼──────────────────┼──────────┼────────────┼────────┤  │
│  │ 007        │ #42 Tilápia      │ Tilápia  │ T-03       │ [Ver][⊘]│ │
│  │ 0963       │ #38 Tambaqui     │ Tambaqui │ T-01       │ [Ver][⊘]│ │
│  │ A12        │ #55 Pacu         │ Pacu     │ T-02       │ [Ver][⊘]│ │
│  └────────────┴──────────────────┴──────────┴────────────┴────────┘  │
│  ← Anterior  1 2 3 ... 7  Próxima →                                 │
└──────────────────────────────────────────────────────────────────────┘
```

#### Wireframe Mobile — Cards

```
┌──────────────────────────────┐
│ Vínculos              [+ Novo]│
│ [ChipCodeInput] [Buscar]      │
├──────────────────────────────┤
│ ┌────────────────────────────┐│
│ │ Chip: 007           (ativo)││
│ │ #42 · Tilápia · T-03       ││
│ │ Desde: 12/04/2025          ││
│ │ [Ver animal]  [Encerrar ⊘] ││
│ └────────────────────────────┘│
└──────────────────────────────┘
```

#### Modal: Nova Associação

```
┌─── Nova Associação ───────────────────────────────────────────────┐
│  Código do chip *                                                  │
│  [ChipCodeInput ____________]                                      │
│  Aceita letras, números e hífen. Ex: 007, A12, B-04               │
│                                                                    │
│  ┌─ Resultado ao sair do campo ──────────────────────────────────┐│
│  │ ✓ Chip 007 — disponível. Último uso: Abate em 10/05/2025     ││
│  └───────────────────────────────────────────────────────────────┘│
│                                                                    │
│  Animal *  [Selecionar ▼]  (busca por ID, somente sem chip ativo) │
│  Data de início *  [dd/mm/aaaa]  (padrão: hoje)                   │
│  Observações  [________________________________]                  │
│                                                                    │
│  [Cancelar]                          [Confirmar Associação]      │
└────────────────────────────────────────────────────────────────────┘
```

#### Modal: Encerrar Vínculo

```
┌─── Encerrar Vínculo ─────────────────────────────────────────────┐
│  ⚠️ Você está encerrando um vínculo ativo.                       │
│                                                                   │
│  Chip: 007  |  Animal: #42 Tilápia do Nilo                       │
│  Tanque: T-03  |  Vinculado desde: 12/04/2025 (57 dias)          │
│                                                                   │
│  Motivo *                                                         │
│  ○ Abate         ○ Morte natural                                  │
│  ○ Perda         ○ Dano no chip                                   │
│  ○ Correção manual                                                │
│                                                                   │
│  Observações  [____________________________________________]      │
│  ⚠️ Esta ação não pode ser desfeita.                              │
│                                                                   │
│  [Cancelar]                        [Encerrar Vínculo]            │
└───────────────────────────────────────────────────────────────────┘
```

#### Regras de Negócio / Validação

- Chip inexistente → bloquear: "Código [X] não encontrado. Cadastre o chip antes de associar."
- Chip já vinculado → bloquear: "Chip [X] já está vinculado ao animal #Y."
- Animal com chip ativo → bloquear: "Animal #Y já possui chip [Z] ativo."
- Motivo de encerramento obrigatório
- Data de início não pode ser futura

#### Paginação

- 20 itens por página
- Filtros enviados ao servidor a cada mudança (nunca filtrar na memória)
- URL reflete filtros ativos (query params) para suportar bookmark/compartilhamento

#### Critérios de Aceite

- [ ] Nenhuma regra de bloqueio pode ser contornada
- [ ] Código exibido exatamente como normalizado (uppercase, sem espaços)
- [ ] Filtros são server-side, não client-side
- [ ] Paginação de 20 itens com contagem total do servidor
- [ ] Mobile mostra cards em vez de tabela

---

### Tela 04 — Espécies

**Rota:** `/especies` | **Usuário-alvo:** Admin, Técnico

#### Wireframe Desktop

```
┌──────────────────────────────────────────────────────────────────┐
│  Espécies                                  [+ Nova Espécie]      │
│  [Buscar por nome ________________]                               │
│  12 espécies · Página 1 de 1                                     │
│                                                                   │
│  ┌──────────────────────┬──────────────────────┬──────────────┐  │
│  │ Nome popular         │ Nome científico       │ Ações        │  │
│  ├──────────────────────┼──────────────────────┼──────────────┤  │
│  │ Tilápia do Nilo      │ Oreochromis niloticus │ [Ed] [⊘]     │  │
│  │ Tambaqui             │ Colossoma macropomum  │ [Ed] [⊘]     │  │
│  └──────────────────────┴──────────────────────┴──────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

#### Formulário (Modal)

```
Nome popular *          [___________________________]  (obrigatório, único)
Nome científico         [___________________________]  (opcional)
Descrição / observações [___________________________]  (opcional, textarea)
```

#### Regras de Validação

- Nome popular: obrigatório, único no sistema, máx 100 chars
- Exclusão bloqueada com animais associados — mostrar contagem no modal de bloqueio
- Busca server-side com debounce 300ms

#### Estados

| Estado             | Comportamento                                                  |
| ------------------ | -------------------------------------------------------------- |
| Lista vazia        | Ícone + "Nenhuma espécie cadastrada" + botão criar             |
| Exclusão bloqueada | Modal: "Espécie associada a X animais — não pode ser excluída" |
| Erro de unicidade  | Inline: "Já existe uma espécie com este nome popular"          |

#### Critérios de Aceite

- [ ] Nome popular único com feedback inline
- [ ] Exclusão bloqueada com animais associados
- [ ] Busca server-side, não client-side

---

### Tela 05 — Tanques

**Rota:** `/tanques` | **Usuário-alvo:** Admin, Técnico

#### Wireframe Desktop

```
┌──────────────────────────────────────────────────────────────────┐
│  Tanques                                        [+ Novo Tanque]  │
│  [Buscar ________]  Status: [Todos ▼]                            │
│                                                                   │
│  ┌────────┬──────────┬──────────┬──────────┬──────────┬────────┐ │
│  │ Nome   │ Cap.     │ L × A    │ Animais  │ Status   │ Ações  │ │
│  ├────────┼──────────┼──────────┼──────────┼──────────┼────────┤ │
│  │ T-01   │ 200 un.  │ 10m×1.2m │ 48       │ ● Ativo  │[Ed][⊘]│ │
│  │ T-02   │ 150 un.  │ 8m×1.0m  │ 32       │ ● Ativo  │[Ed][⊘]│ │
│  │ T-03   │ 100 un.  │ 6m×0.8m  │ 0        │ ○ Inativo│[Ed][⊘]│ │
│  └────────┴──────────┴──────────┴──────────┴──────────┴────────┘ │
└──────────────────────────────────────────────────────────────────┘
```

#### Formulário (Modal)

```
Nome / código *    [_______________]   (obrigatório, único)
Capacidade *       [_______] unidades  (obrigatório, inteiro positivo)
Largura            [_______] metros    (opcional)
Altura             [_______] metros    (opcional)
Status             ● Ativo  ○ Inativo
Observações        [_______________________________]
```

#### Regras de Negócio

- Tanque inativo não aparece nos selects de animal, vínculo ou eventos
- Exclusão bloqueada se houver animais (ativos ou históricos)

#### Critérios de Aceite

- [ ] Tanque inativo ausente em todos os formulários de seleção
- [ ] Exclusão bloqueada com animais associados
- [ ] Coluna "Animais" exibe contagem real (não local)

---

### Tela 06 — Chips

**Rota:** `/chips` | **Usuário-alvo:** Admin, Técnico

#### Wireframe Desktop

```
┌──────────────────────────────────────────────────────────────────┐
│  Chips de Identificação                          [+ Novo Chip]   │
│  [ChipCodeInput ______]  Status: [Todos ▼]  [Buscar]            │
│  248 chips · Página 1 de 13                                      │
│                                                                   │
│  ┌──────────┬──────────────────┬────────────────────┬──────────┐ │
│  │ Código   │ Status           │ Animal atual       │ Ações    │ │
│  ├──────────┼──────────────────┼────────────────────┼──────────┤ │
│  │ 007      │ ● Vinculado      │ #42 Tilápia · T-03 │ [Ver]    │ │
│  │ 0963     │ ○ Disponível     │ —                  │ [Ver]    │ │
│  │ A12      │ ⊘ Danificado     │ —                  │ [Ver][Ed]│ │
│  │ B-04     │ ◎ Recuperado     │ —                  │ [Ver]    │ │
│  └──────────┴──────────────────┴────────────────────┴──────────┘ │
│  ← Anterior  1 2 3 ... 13  Próxima →                            │
└──────────────────────────────────────────────────────────────────┘
```

#### Formulário: Novo Chip

```
Código *    [ChipCodeInput ___________________]
            Aceita A–Z, 0–9 e hífen · máx 64 caracteres
            Ex: 007, A12, B-04, 0963
Observações [________________________________]
```

#### Status de Chip

| Status     | Dot        | Significado            | Na lista de nova associação? |
| ---------- | ---------- | ---------------------- | ---------------------------- |
| Vinculado  | ● verde    | Ativo em um animal     | Não                          |
| Disponível | ○ azul     | Livre para novo uso    | Sim                          |
| Recuperado | ◎ amarelo  | Devolvido recentemente | Sim                          |
| Danificado | ⊘ vermelho | Inoperante             | **Não**                      |

#### Regras de Negócio

- Código é string, armazenada e exibida exatamente como normalizada
- Código único no sistema (normalização aplicada antes da comparação)
- Exclusão física bloqueada se houver histórico — marcar como Danificado
- `maxLength={64}` no input

#### Critérios de Aceite

- [ ] `007` e `7` são chips distintos se cadastrados separadamente
- [ ] Chip danificado não aparece no select de nova associação
- [ ] Código exibido em `font-mono` em todas as telas sem exceção
- [ ] `maxLength=64` aplicado no ChipCodeInput

---

### Tela 07 — Animais (Lista)

**Rota:** `/animais` | **Usuário-alvo:** Todos

#### Wireframe Desktop

```
┌──────────────────────────────────────────────────────────────────┐
│  Animais                                        [+ Novo Animal]  │
│  [Buscar por ID ou observação ________]                          │
│  Espécie: [Todas ▼]  Tanque: [Todos ▼]  Chip: [Com chip ▼]      │
│  142 animais · Página 1 de 8                                     │
│                                                                   │
│  ┌──────┬──────────┬───────────┬──────────┬──────────┬────────┐  │
│  │ ID   │ Espécie  │ Tanque    │ Chip     │ Cadastro │ Ações  │  │
│  ├──────┼──────────┼───────────┼──────────┼──────────┼────────┤  │
│  │ #42  │ Tilápia  │ T-03      │ 007      │ 12/04/25 │ [Ver]  │  │
│  │ #44  │ Pacu     │ T-01      │ — ⚠️     │ 01/06/25 │ [Ver]  │  │
│  └──────┴──────────┴───────────┴──────────┴──────────┴────────┘  │
│  ← Anterior  1 2 3 ... 8  Próxima →                             │
└──────────────────────────────────────────────────────────────────┘
```

#### Wireframe Mobile — Cards

```
┌───────────────────────────────┐
│ #42 · Tilápia do Nilo         │
│ Tanque T-03 · Chip 007        │
│ Desde 12/04/2025    [Ver →]   │
└───────────────────────────────┘
┌───────────────────────────────┐
│ #44 · Pacu            ⚠️      │
│ Tanque T-01 · SEM CHIP        │
│ Desde 01/06/2025    [Ver →]   │
└───────────────────────────────┘
```

#### Paginação e Filtros

- 20 itens/página, server-side
- Filtros cumulativos enviados ao servidor
- Opções de Chip: Todos / Com chip / Sem chip
- Busca por texto: ID numérico ou texto livre em observações

#### Critérios de Aceite

- [ ] Animal sem chip tem badge ⚠️ + tooltip explicativo
- [ ] Todos os filtros são server-side
- [ ] Mobile usa cards em vez de tabela (sem scroll horizontal)

---

### Tela 08 — Detalhe do Animal

**Rota:** `/animais/[id]` | **Usuário-alvo:** Todos

#### Wireframe Desktop

```
┌──────────────────────────────────────────────────────────────────┐
│  ← Voltar   Animal #42              [Editar] [Registrar Abate]   │
│                                                                   │
│  ┌─ Dados ─────────────────────────────────────────────────────┐ │
│  │  Espécie:  Tilápia do Nilo (Oreochromis niloticus)           │ │
│  │  Tanque:   T-03                                              │ │
│  │  Cadastro: 12/04/2025                                        │ │
│  │  Observações: Animal de reposição, lote B.                   │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌─ Chip Atual ────────────────────────────────────────────────┐ │
│  │  ● Vinculado: Chip 007 · desde 12/04/2025 (57 dias)         │ │
│  │  [Encerrar vínculo]                                         │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌─ Histórico de Chips ────────────────────────────────────────┐ │
│  │  3 registros · Página 1 de 1                                │ │
│  │  ┌─────────┬────────────┬────────────┬─────────────────┐    │ │
│  │  │ Chip    │ Início     │ Fim        │ Motivo          │    │ │
│  │  ├─────────┼────────────┼────────────┼─────────────────┤    │ │
│  │  │ 007     │ 12/04/2025 │ —          │ (ativo)         │    │ │
│  │  │ 003     │ 10/01/2025 │ 11/04/2025 │ Dano no chip    │    │ │
│  │  │ 001     │ 05/10/2024 │ 09/01/2025 │ Correção manual │    │ │
│  │  └─────────┴────────────┴────────────┴─────────────────┘    │ │
│  └─────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────┘
```

#### Histórico — Paginação e Agrupamento

- 20 registros por página (histórico pode ser longo — L3)
- Ordenado por data desc (mais recente primeiro)
- Agrupamento opcional por ano (toggle visual)
- Cada linha do histórico tem o código em `font-mono`
- Histórico é sempre somente leitura

#### Estados Especiais

| Estado         | Comportamento                                                      |
| -------------- | ------------------------------------------------------------------ |
| Sem chip ativo | Seção "Chip Atual": aviso azul + botão "Associar chip"             |
| Animal abatido | Banner vermelho no topo + campos readonly + sem ações operacionais |
| Carregando     | Skeleton por seção, não spinner de página                          |

#### Mobile

- Seções empilhadas verticalmente full-width
- Botões de ação no rodapé fixo (sticky bottom bar)

#### Critérios de Aceite

- [ ] Histórico paginado, ordenado desc, somente leitura
- [ ] Animal abatido: sem nenhuma ação operacional disponível
- [ ] Código do chip em `font-mono` em todas as linhas do histórico
- [ ] Estado "sem chip" mostra caminho claro para criar vínculo

---

### Tela 09 — Histórico do Chip

**Rota:** `/chips/[id]` | **Usuário-alvo:** Todos

#### Wireframe

```
┌──────────────────────────────────────────────────────────────────┐
│  ← Voltar   Chip 007                                  [Editar]  │
│                                                                   │
│  ┌─ Status Atual ───────────────────────────────────────────┐    │
│  │  ● Vinculado ao Animal #42 · Tilápia do Nilo · T-03      │    │
│  │  Desde: 12/04/2025                                        │    │
│  └───────────────────────────────────────────────────────────┘    │
│                                                                   │
│  ┌─ Histórico de Associações ─────────────────────────────────┐  │
│  │  Utilizado em 3 animais · Página 1 de 1                    │  │
│  │  ┌────────┬────────────┬────────────┬──────────┬─────────┐  │  │
│  │  │ Animal │ Espécie    │ Início     │ Fim      │ Motivo  │  │  │
│  │  ├────────┼────────────┼────────────┼──────────┼─────────┤  │  │
│  │  │ #42    │ Tilápia    │ 12/04/2025 │ —        │ (ativo) │  │  │
│  │  │ #31    │ Tambaqui   │ 01/01/2025 │ 11/04/25 │ Abate   │  │  │
│  │  │ #18    │ Tilápia    │ 05/06/2024 │ 31/12/24 │ Morte   │  │  │
│  │  └────────┴────────────┴────────────┴──────────┴─────────┘  │  │
│  └─────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

#### Critérios de Aceite

- [ ] Histórico paginado (20/página), ordenado desc
- [ ] Cada linha clicável → detalhe do animal
- [ ] Chip sem histórico: "Chip recém-cadastrado, sem histórico de uso."
- [ ] Código do chip no cabeçalho e na tabela sempre em `font-mono`

---

### Tela 10 — Abate Individual

**Rota:** `/abate/individual` | **Usuário-alvo:** Técnico, Admin

#### Stepper — 3 Etapas

```
[1. Identificar] ──── [2. Dados] ──── [3. Confirmar]
```

**Etapa 1 — Identificar animal**

```
┌──────────────────────────────────────────────────────────────────┐
│  Abate Individual  ── 1 de 3: Identificar animal ─────────────  │
│                                                                   │
│  Código do chip *                                                 │
│  [ChipCodeInput ________]  [Buscar]                               │
│  Aceita A–Z, 0–9 e hífen · ex: 007, A12, B-04                   │
│                                                                   │
│  ┌─ ResultCard (após lookup) ──────────────────────────────────┐ │
│  │ ✓ Chip 007 — Animal #42 · Tilápia do Nilo · Tanque T-03    │ │
│  │   Vinculado em 12/04/2025 (57 dias)                         │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  [Avançar →]  (habilitado somente após lookup bem-sucedido)      │
└──────────────────────────────────────────────────────────────────┘
```

**Etapa 2 — Dados do abate**

```
┌──────────────────────────────────────────────────────────────────┐
│  ── 2 de 3: Dados do abate ──────────────────────────────────── │
│                                                                   │
│  Animal:  #42 Tilápia do Nilo · T-03 · Chip 007  (readonly)     │
│                                                                   │
│  Data do abate *  [dd/mm/aaaa]  (padrão: hoje)                   │
│  Peso (g)         [_______]     (opcional)                       │
│  Observações      [________________________________]             │
│                                                                   │
│  ┌─ Recuperação do chip ───────────────────────────────────────┐ │
│  │  Chip 007 — O que fazer após o abate?                       │ │
│  │  ○ Recuperar — marcar como disponível para novo uso         │ │
│  │  ○ Não recuperar                                            │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  [← Voltar]                                     [Avançar →]     │
└──────────────────────────────────────────────────────────────────┘
```

**Etapa 3 — Confirmação final**

```
┌──────────────────────────────────────────────────────────────────┐
│  ── 3 de 3: Confirmar abate ─────────────────────────────────── │
│                                                                   │
│  ⚠️ ATENÇÃO — Esta ação não pode ser desfeita.                   │
│                                                                   │
│  Animal:    #42 Tilápia do Nilo · Tanque T-03                    │
│  Chip:      007 → será recuperado ✓                              │
│  Data:      08/06/2026                                           │
│  Peso:      450g                                                 │
│                                                                   │
│  [← Voltar]                         [Confirmar Abate]            │
└──────────────────────────────────────────────────────────────────┘
```

> Botão "Confirmar Abate" usa estilo `danger`. Enter **não** confirma.  
> Após confirmação → redirect para detalhe do animal (somente leitura) + toast de sucesso.

#### Bloqueios na Etapa 1

| Caso               | Mensagem                                                        |
| ------------------ | --------------------------------------------------------------- |
| Chip inexistente   | "Código [X] não encontrado no sistema."                         |
| Chip não vinculado | "Chip [X] não está vinculado a nenhum animal."                  |
| Animal já abatido  | "Animal vinculado ao chip [X] já foi abatido em [data]."        |
| Chip danificado    | "Chip [X] está marcado como danificado e não pode ser operado." |

#### Mobile

- Stepper vertical, cada etapa ocupa tela inteira com scroll
- Botões de navegação fixos no rodapé

#### Critérios de Aceite

- [ ] Etapa 1 bloqueia todos os casos inválidos com mensagem específica
- [ ] Pergunta sobre recuperação do chip é obrigatória
- [ ] Enter não confirma na etapa 3
- [ ] Etapa 3 exibe resumo completo antes do commit
- [ ] Nenhuma rota permite pular etapas

---

### Tela 11 — Abate em Lote

**Rota:** `/abate/lote` | **Usuário-alvo:** Técnico, Admin  
**Status:** Escopo futuro — especificado aqui para orientar implementação futura sem retrabalho.

#### Stepper — 4 Etapas

**Etapa 1 — Filtro e seleção**

```
┌──────────────────────────────────────────────────────────────────┐
│  Abate em Lote  ── 1 de 4: Selecionar animais ─────────────── │
│                                                                   │
│  Filtrar por tanque:  [Todos ▼]    Espécie: [Todas ▼]           │
│  [Selecionar todos os filtrados]   (atalho — não default)        │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │ ☑ #42 Tilápia do Nilo · T-03 · Chip 007                    │ │
│  │ ☑ #38 Tambaqui       · T-01 · Chip 0963                    │ │
│  │ □ #44 Pacu           · T-01 · Sem chip ⚠️                  │ │
│  │ ☑ #55 Pacu           · T-02 · Chip A12                     │ │
│  └─────────────────────────────────────────────────────────────┘ │
│  3 animais selecionados                                          │
│  [Avançar →]                                                     │
└──────────────────────────────────────────────────────────────────┘
```

**Etapa 2 — Dados gerais**

```
Data do abate *   [dd/mm/aaaa]  (padrão: hoje)
Observações       [________________________________]
[← Voltar]                                [Avançar →]
```

**Etapa 3 — Revisão e chips**

```
┌──────────────────────────────────────────────────────────────────┐
│  ── 3 de 4: Revisão e recuperação de chips ─────────────────── │
│  [Recuperar todos]  [Não recuperar nenhum]  (atalhos globais)   │
│                                                                   │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │ #42 Tilápia · Chip 007                                     │  │
│  │   Recuperar chip?  ○ Sim  ○ Não                            │  │
│  ├────────────────────────────────────────────────────────────┤  │
│  │ #38 Tambaqui · Chip 0963                                   │  │
│  │   Recuperar chip?  ○ Sim  ○ Não                            │  │
│  ├────────────────────────────────────────────────────────────┤  │
│  │ #55 Pacu · Chip A12                                        │  │
│  │   Recuperar chip?  ○ Sim  ○ Não                            │  │
│  └────────────────────────────────────────────────────────────┘  │
│  [← Voltar]                                     [Avançar →]     │
└──────────────────────────────────────────────────────────────────┘
```

**Etapa 4 — Confirmação final**

```
┌──────────────────────────────────────────────────────────────────┐
│  ── 4 de 4: Confirmar abate em lote ────────────────────────── │
│                                                                   │
│  ⚠️ ATENÇÃO — Esta ação afeta 3 animais. Não pode ser desfeita.  │
│                                                                   │
│  Data: 08/06/2026                                                │
│  · #42 Tilápia · Chip 007  → recuperado                         │
│  · #38 Tambaqui · Chip 0963 → não recuperado                    │
│  · #55 Pacu · Chip A12     → recuperado                         │
│                                                                   │
│  [← Voltar]              [Confirmar Abate de 3 Animais]         │
└──────────────────────────────────────────────────────────────────┘
```

Após confirmação: página de resultado com contagem (processados, chips recuperados, erros).

#### No MVP

- Criar rota com `PlaceholderPage` ("Em desenvolvimento")
- **Não** incluir no menu lateral até estar pronto
- A especificação acima orienta implementação futura sem retrabalho arquitetural

---

### Tela 12 — Desova

**Rota:** `/desova` | **Status:** Escopo futuro (placeholder no MVP)

**Arquitetura de dados:** `SpawningEvent` → animal → tanque + espécie como contexto.  
`MonitoringRecord` (temperatura, hora-grau) pertence ao `SpawningEvent`.

---

### Tela 13 — Monitoramento

**Rota:** `/monitoramento` | **Status:** Escopo futuro (placeholder no MVP)

---

### Tela 14 — Relatórios

**Rota:** `/relatorios` | **Usuário-alvo:** Admin, Técnico

#### MVP: Página de relatórios básica

```
┌──────────────────────────────────────────────────────────────────┐
│  Relatórios                                                       │
│                                                                   │
│  ┌─ Exportação CSV ───────────────────────────────────────────┐  │
│  │  Exportar animais     Filtros: Espécie [Todas ▼] Tanque [Todos ▼]│
│  │  Status: [Ativos ▼]   [Exportar CSV ↓]                    │  │
│  ├────────────────────────────────────────────────────────────┤  │
│  │  Exportar histórico de vínculos                            │  │
│  │  Período: [dd/mm/aaaa] a [dd/mm/aaaa]   [Exportar CSV ↓]  │  │
│  ├────────────────────────────────────────────────────────────┤  │
│  │  Exportar chips        Status: [Todos ▼]  [Exportar CSV ↓]│  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                   │
│  🚧 Relatórios analíticos em desenvolvimento (v1.1)              │
└──────────────────────────────────────────────────────────────────┘
```

#### Regras de Export

- Botão de exportar reflete **exatamente os filtros ativos** — sem surpresas no arquivo
- Antes de baixar, exibir: "X registros serão exportados com os filtros atuais. Continuar?"
- Formato: CSV com cabeçalho em português, datas em `dd/mm/aaaa`, códigos de chip sem coerção numérica

#### Critérios de Aceite

- [ ] Exportação respeita filtros ativos
- [ ] Preview de quantidade antes do download
- [ ] Código de chip nunca exportado como número (colunas CSV entre aspas se necessário)

---

### Tela 15 — Usuários e Permissões

**Rota:** `/usuarios` | **Usuário-alvo:** Admin exclusivamente

#### Wireframe

```
┌──────────────────────────────────────────────────────────────────┐
│  Usuários                                       [+ Novo Usuário] │
│  [Buscar ________]  Perfil: [Todos ▼]                            │
│                                                                   │
│  ┌──────────────┬────────────────────┬──────────┬──────────┬───┐ │
│  │ Nome         │ E-mail             │ Perfil   │ Status   │Ed.│ │
│  ├──────────────┼────────────────────┼──────────┼──────────┼───┤ │
│  │ João Silva   │ joao@...           │ Admin    │ ● Ativo  │[Ed]│ │
│  │ Maria Souza  │ maria@...          │ Técnico  │ ● Ativo  │[Ed]│ │
│  │ Carlos Lima  │ carlos@...         │ Visitante│ ○ Inativo│[Ed]│ │
│  └──────────────┴────────────────────┴──────────┴──────────┴───┘ │
└──────────────────────────────────────────────────────────────────┘
```

#### Perfis de Acesso

| Perfil    | Leitura | Criar/Editar               | Encerrar/Abater | Exportar           | Gerenciar usuários |
| --------- | ------- | -------------------------- | --------------- | ------------------ | ------------------ |
| Admin     | Tudo    | Tudo                       | Tudo            | Tudo               | ✓                  |
| Técnico   | Tudo    | Animais, vínculos, eventos | ✓               | Animais e vínculos | ✗                  |
| Visitante | Tudo    | ✗                          | ✗               | ✗                  | ✗                  |

#### Regras de Negócio

- Admin não pode remover o próprio perfil de Admin
- Deve existir ao menos 1 Admin ativo no sistema
- Alterar perfil exige confirmação em modal

#### Critérios de Aceite

- [ ] Rota retorna 403 para Técnico e Visitante
- [ ] Último Admin ativo: não pode ser desativado nem rebaixado
- [ ] Alteração de perfil requer modal de confirmação

---

## Parte 5 — Design System

### 5.1 Filosofia Visual

Ferramenta operacional de manejo profissional. Não é um dashboard SaaS, não é uma landing page.

Princípios:

- **Legibilidade em uso repetitivo** — texto limpo, contraste alto, sem animações sem propósito
- **Semântica de cor funcional** — cores comunicam estado operacional, não decoração
- **Densidade adequada** — compacto mas legível; menos scroll por ação
- **Escalabilidade controlada** — Tailwind com tokens, não classes improvisadas

---

### 5.2 Tailwind Config — Fonte Única de Tokens

```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss';

const config: Config = {
  content: ['./src/**/*.{ts,tsx}'],
  theme: {
    extend: {
      colors: {
        base: '#09090B',
        surface: '#111113',
        elevated: '#18181B',
        input: '#101012',

        border: {
          DEFAULT: '#27272A',
          subtle: '#1A1A1D',
        },

        text: {
          primary: '#FAFAFA',
          secondary: '#A1A1AA',
          muted: '#71717A',
          inverse: '#09090B',
        },

        brand: {
          DEFAULT: '#7C3AED',
          hover: '#8B5CF6',
          subtle: '#2E1065',
        },

        success: {
          DEFAULT: '#22C55E',
          subtle: '#052E16',
        },

        warning: {
          DEFAULT: '#F59E0B',
          subtle: '#451A03',
        },

        danger: {
          DEFAULT: '#EF4444',
          subtle: '#450A0A',
        },

        info: {
          DEFAULT: '#3B82F6',
          subtle: '#172554',
        },
      },

      fontFamily: {
        sans: ['var(--font-inter)', 'system-ui', 'sans-serif'],
        mono: ['var(--font-jetbrains)', 'Courier New', 'monospace'],
      },

      fontSize: {
        xs: ['0.75rem', { lineHeight: '1rem' }],
        sm: ['0.875rem', { lineHeight: '1.25rem' }],
        base: ['1rem', { lineHeight: '1.5rem' }],
        lg: ['1.125rem', { lineHeight: '1.75rem' }],
        xl: ['1.25rem', { lineHeight: '1.75rem' }],
        '2xl': ['1.5rem', { lineHeight: '2rem' }],
        '3xl': ['1.875rem', { lineHeight: '2.25rem' }],
      },

      borderRadius: {
        sm: '6px',
        md: '10px',
        lg: '14px',
        xl: '18px',
        full: '9999px',
      },

      boxShadow: {
        sm: '0 2px 8px rgba(0,0,0,0.25)',
        md: '0 8px 24px rgba(0,0,0,0.35)',
        lg: '0 16px 48px rgba(0,0,0,0.45)',
        glow: '0 0 32px rgba(124,58,237,0.25)',
      },

      screens: {
        sm: '640px',
        md: '768px',
        lg: '1024px',
        xl: '1280px',
        '2xl': '1536px',
      },
    },
  },
  plugins: [],
};

export default config;
```

> **Regra absoluta:** Nenhuma cor, fonte ou espaçamento como valor literal no JSX/TSX.  
> Toda variação via classes Tailwind mapeadas no config acima.

---

### 5.3 Tipografia

```
Fontes (via next/font):
  Inter        → font-sans → todo o texto da interface
  JetBrains Mono → font-mono → exclusivamente para códigos de chip

Regra crítica:
  Código de chip: SEMPRE font-mono, em TODOS os contextos:
  - Inputs de busca e cadastro
  - Tabelas e listas
  - Histórico
  - ResultCards
  - Modais de confirmação
  Motivo: 007 vs 07 vs 0007 devem ser visualmente distintos.

Escala de uso típico:
  text-3xl font-bold    → números em StatCards
  text-xl font-semibold → h1 de página
  text-lg font-semibold → h2 de seção
  text-base             → formulários, corpo
  text-sm               → tabelas, labels
  text-xs text-muted    → metadados, timestamps secundários
```

---

### 5.4 Layout e Grid

```
AppLayout:
  Desktop (≥ 1024px):  sidebar fixa w-60 + main flex-1
  Tablet (< 1024px):   drawer deslizável + botão hambúrguer no header
  Mobile (< 640px):    drawer ou bottom nav com 5 itens principais

AuthLayout:
  Full screen, bg-base, flex items-center justify-center
  max-w-md mx-auto para o card de formulário

Content:
  Padding de página: p-6 (desktop), p-4 (tablet/mobile)
  max-w-5xl mx-auto para listas e tabelas
  max-w-4xl mx-auto para detalhes de entidade
```

---

### 5.5 Componentes — Variantes e Regras

#### Botões

```
Variantes:
  primary   → bg-brand text-text-inverse hover:bg-brand-hover
  secondary → bg-surface border border-border text-text-primary hover:bg-elevated
  danger    → bg-danger-subtle border border-danger text-danger
              hover:bg-danger hover:text-text-inverse
  ghost     → bg-transparent text-text-secondary hover:bg-elevated

Tamanhos:
  sm → h-8  px-3 text-sm  (ações em tabela)
  md → h-10 px-4 text-base  (padrão)
  lg → h-12 px-6 text-lg  (CTAs em mobile)

Regras:
  · Máx 1 botão primary por contexto
  · Danger EXCLUSIVAMENTE em ações destrutivas
  · Cancelar à esquerda, ação principal à direita
  · Enter NÃO confirma ações danger
  · Loading: spinner inline left + texto "Carregando..." + disabled
  · Icon-only: sempre aria-label + tooltip via title
```

#### Inputs

```
Base:
  border border-border bg-input text-text-primary
  rounded-md h-10 px-3 text-base
  placeholder:text-text-muted

Focus:
  focus:outline-none focus:border-brand focus:ring-2 focus:ring-brand/20

Erro:
  border-danger focus:ring-danger/20
  + mensagem abaixo: text-danger text-sm mt-1

Select com muitas opções (> 10):
  Implementar combobox pesquisável em vez de select nativo

Textarea:
  Mesma base, min-h-[80px] resize-y
```

#### ChipCodeInput (componente crítico)

```typescript
// Especificação completa do componente

interface ChipCodeInputProps {
  value: string;
  onChange: (normalized: string) => void;
  onResolve: (code: string, source: ChipInputSource) => void;
  disabled?: boolean;
  // Extensibilidade futura:
  inputSource?: 'manual' | 'nfc' | 'bluetooth' | 'keyboard_emulation';
}

// Atributos HTML obrigatórios:
// inputMode="text"       → teclado genérico (aceita alfanumérico)
// autoCapitalize="characters" → maiúsculas automáticas em mobile
// autoCorrect="off"
// spellCheck={false}
// maxLength={64}         → limite do domínio (não 80 do contrato)
// className inclui: font-mono text-lg text-center tracking-widest

// Normalização em tempo real (onChange):
// 1. Remover espaços em branco
// 2. Converter para UPPERCASE
// 3. Filtrar: manter apenas A-Z, 0-9, hífen
// 4. NUNCA parseInt(), parseFloat() ou Number()
// 5. NUNCA remover zeros à esquerda

// Lookup (ao blur ou Enter):
// 1. Disparar busca na API com código normalizado
// 2. Exibir ResultCard abaixo com um dos 5 estados
// 3. Não avançar fluxo sem lookup bem-sucedido

// Placeholder: "ex: 007, A12, B-04"
// Hint abaixo: "Chips atuais costumam ter 3–4 dígitos"
```

#### Tabelas

```
thead: bg-elevated text-text-secondary text-xs uppercase tracking-wide
tbody tr: border-b border-border-subtle hover:bg-elevated transition-colors
td: py-3 px-4 text-sm
Primeira coluna: font-medium text-text-primary
Coluna de código de chip: font-mono — sem truncamento
Coluna de status: StatusBadge (dot + texto) — nunca só cor
Coluna de ações: text-right, gap-2

Tabela compacta (histórico): py-2 no td, text-xs no thead

Paginação:
  Sempre server-side, nunca filtrar/paginar no frontend
  Padrão: 20 itens/página
  Componente: mostra "← Anterior  1 2 3 ... N  Próxima →"
  + "X de Y registros" acima ou abaixo

Mobile (< 640px):
  Tabelas viram MobileCardList
  Cada card: bg-surface rounded-lg p-4 border border-border
  Exibe os mesmos dados em layout vertical label → valor
```

#### Paginação

```
Padrão: 20 itens/página
Server-side sempre (nunca filtrar/paginar no frontend)
URL query params refletem filtros e página (para bookmark)
Componente visual: ← Anterior · números de página · Próxima →
Contagem: "142 registros · Página 1 de 8"
```

#### StatCard

```
bg-surface rounded-xl border border-border p-5
Número: text-3xl font-bold text-text-primary
Rótulo: text-sm text-text-secondary mt-1
Detalhe: text-xs text-text-muted mt-1
Sem cor de destaque no número — cor apenas no detalhe semântico
```

#### ResultCard (resposta do ChipCodeInput)

```
Estrutura base:
  rounded-lg border-l-4 p-4
  ícone de status (lucide) + texto principal bold + metadados + ações inline

Estados:
  VINCULADO:    border-success bg-success-subtle → ícone: Link2
  DISPONÍVEL:   border-info    bg-info-subtle    → ícone: CheckCircle
  RECUPERADO:   border-warning bg-warning-subtle → ícone: RefreshCw
  DANIFICADO:   border-danger  bg-danger-subtle  → ícone: XCircle
  INEXISTENTE:  border-danger  bg-danger-subtle  → ícone: AlertCircle

Animação: fade-in 150ms ao aparecer
aria-live="polite" para anúncio de screen reader
```

#### Modais

```
Overlay: bg-black/70 backdrop-blur-sm

Dialog (desktop):
  bg-surface rounded-xl border border-border p-6
  max-w-lg (padrão) / max-w-2xl (largo)
  max-h-[90vh] overflow-y-auto

Mobile: bottom sheet
  rounded-t-2xl fixo na base, max-h-[85vh]
  Mesmo conteúdo do modal desktop

Modal danger:
  Header: bg-danger-subtle border-b border-danger/30
  Ícone ⚠️ no header
  Resumo dos dados ANTES do botão confirmar
  Botão confirmar: variant danger
  Foco inicial: botão Cancelar
  Enter: não confirma
  Escape: fecha = cancela

role="dialog" aria-modal="true" aria-labelledby=[id do título]
Trap de foco dentro do modal (Tab/Shift+Tab cicla internamente)
```

#### Toasts

```
Desktop: bottom-right, 16px de margem
Mobile: top-center

Duração: success 3s · warning 4.5s · error 6s

Tipos:
  success → border-l-4 border-success + ícone Check
  error   → border-l-4 border-danger  + ícone X
  warning → border-l-4 border-warning + ícone Triangle
  info    → border-l-4 border-info    + ícone Info

USAR para: confirmação de ação concluída, avisos não bloqueantes
NÃO usar para: resultado de consulta de chip, erros de formulário, ações que precisam de decisão
```

#### StatusBadge

```typescript
// SEMPRE dot + texto — NUNCA só cor
// "● Vinculado"     → text-success  + dot success
// "○ Disponível"    → text-info     + dot info
// "◎ Recuperado"    → text-warning  + dot warning
// "⊘ Danificado"    → text-danger   + dot danger
// "● Ativo"         → text-success  + dot success
// "○ Inativo"       → text-muted    + dot muted
// "✕ Abatido"       → text-danger   + bg-danger-subtle rounded-full px-2
```

---

### 5.6 Padrões Especiais

#### Padrão: Ações Perigosas

1. Botão de trigger: estilo `danger`
2. Modal obrigatório com resumo dos dados afetados
3. Texto "Esta ação não pode ser desfeita" visível
4. Botão de confirmar no modal: `danger`
5. Foco inicial: botão Cancelar
6. Escape = Cancelar
7. Enter **não** confirma
8. Mobile: bottom sheet

#### Padrão: Dados Históricos

1. Ordem cronológica decrescente (mais recente primeiro)
2. Registro ativo: destaque leve (texto mais brilhante + dot verde)
3. Registros encerrados: `text-text-secondary`
4. Motivo de encerramento sempre exibido
5. Datas: `dd/mm/aaaa` — sem formato relativo em contexto operacional
6. Paginação de 20 itens — sem exceção para históricos longos
7. Agrupamento temporal por ano: toggle opcional
8. Somente leitura — sem edição de registros passados

#### Padrão: Exportação CSV

1. Botão de exportar sempre visível junto aos filtros ativos
2. Preview: "X registros serão exportados. Continuar?" antes do download
3. Código de chip exportado como string (sem coerção numérica no CSV)
4. Datas em `dd/mm/aaaa`
5. Cabeçalho em português

---

## Parte 6 — Prompt para o Codex

> Copie exatamente o bloco abaixo e envie ao Codex como primeira mensagem da sessão de implementação.

---

```
Você está implementando o frontend do Fisher Control, sistema web profissional
para gestão de piscicultura.

═══════════════════════════════════════
DOMÍNIO — REGRAS INVIOLÁVEIS
═══════════════════════════════════════

Animal e chip são entidades SEPARADAS.
O chip é um identificador físico reutilizável.
Após abate, o chip pode ser recolhido e implantado em outro animal.
O chip NÃO é a identidade permanente do animal.

Chips são digitados manualmente pelo operador após leitura em aparelho externo.
O sistema assume que o leitor pode mostrar o código, mas NÃO depende disso.
A confirmação operacional é SEMPRE do Fisher Control.

═══════════════════════════════════════
STACK
═══════════════════════════════════════

- Next.js 14+ com App Router
- React + TypeScript strict
- Tailwind CSS — tailwind.config.ts é a ÚNICA fonte de tokens
  Sem valores literais (#hex, px avulsos) no JSX/TSX
- lucide-react para ícones
- Google Fonts via next/font: Inter (sans) + JetBrains Mono (mono)
- Componentes organizados por feature em /features/[feature]/components/
- Cliente HTTP tipado com tipos locais derivados do contrato REST da API

═══════════════════════════════════════
TAILWIND CONFIG
═══════════════════════════════════════

Copie integralmente o tailwind.config.ts da Seção 5.2 do documento de especificação.
NÃO adicione tokens fora do config.

═══════════════════════════════════════
ROTAS
═══════════════════════════════════════

/login               → AuthLayout, sem sidebar
/dashboard           → AppLayout
/vinculos            → AppLayout
/animais             → AppLayout
/animais/[id]        → AppLayout
/especies            → AppLayout
/tanques             → AppLayout
/chips               → AppLayout
/chips/[id]          → AppLayout
/abate/individual    → AppLayout
/abate/lote          → PlaceholderPage ("Em desenvolvimento") — SEM link no menu
/desova              → PlaceholderPage — SEM link no menu
/monitoramento       → PlaceholderPage — SEM link no menu
/relatorios          → AppLayout (exportação CSV básica no MVP)
/usuarios            → AppLayout, Admin only

Proteção de rotas:
  Não autenticado → redirect /login
  Admin-only com outro perfil → 403 page

═══════════════════════════════════════
LAYOUTS
═══════════════════════════════════════

AppLayout:
  ≥ 1024px → sidebar fixa w-60 + main flex-1 overflow-y-auto
  < 1024px → drawer deslizável (hambúrguer no header)
  < 640px  → drawer ou bottom nav com itens principais

AuthLayout:
  Full screen bg-base, sem sidebar, card centralizado max-w-md

═══════════════════════════════════════
PAGINAÇÃO — REGRA GLOBAL
═══════════════════════════════════════

TODOS os listagens usam:
  - 20 itens por página (backend suporta max 100, padrão é 20)
  - Paginação e filtros SEMPRE server-side — nunca filtrar na memória
  - URL query params refletem filtros e página ativos
  - Componente de paginação: "← Anterior · 1 2 3 ... N · Próxima →"
  - Contagem: "X registros · Página Y de Z"

═══════════════════════════════════════
ORDEM DE IMPLEMENTAÇÃO
═══════════════════════════════════════

1.  tailwind.config.ts — tokens completos
2.  globals: next/font Inter + JetBrains Mono, variáveis CSS, reset
3.  AppLayout + AuthLayout
4.  Sidebar (grupos de nav, perfil, items "em breve" não-clicáveis)
5.  ChipCodeInput (componente crítico — ver spec abaixo)
6.  StatusBadge (dot + texto, nunca só cor)
7.  DataTable + MobileCardList (paginação, empty/loading/error states)
8.  Paginação component (server-side, query params)
9.  Modal base + BottomSheet mobile (trap de foco, Escape fecha)
10. Toast system
11. StatCard
12. ResultCard (5 estados, aria-live)
13. PlaceholderPage
14. Telas por prioridade:
    a. /login
    b. /dashboard
    c. /vinculos
    d. /animais + /animais/[id]
    e. /chips + /chips/[id]
    f. /especies + /tanques
    g. /abate/individual
    h. /relatorios (exportação CSV)
    i. /usuarios

═══════════════════════════════════════
ChipCodeInput — ESPECIFICAÇÃO COMPLETA
═══════════════════════════════════════

REGRAS ABSOLUTAS (violá-las = defeito crítico):

1.  O código do chip é SEMPRE uma string. Nunca: parseInt, parseFloat, Number().
2.  Zeros à esquerda NUNCA removidos. "007" e "7" são chips distintos.
3.  Formato aceito: A–Z, 0–9, hífen. Máximo 64 caracteres.
4.  Normalização em tempo real: remover espaços + uppercase + filtrar chars inválidos.
5.  Exibir código normalizado enquanto o usuário digita.
6.  maxLength={64} no input (limite do domínio, não 80 do contrato — divergência conhecida).
7.  Ao blur ou Enter: disparar lookup na API.
8.  Exibir ResultCard abaixo com um dos 5 estados (ver Design System).
9.  Não avançar nenhum fluxo sem lookup bem-sucedido.
10. Hint visível abaixo do campo: "Chips atuais costumam ter 3–4 dígitos"
11. Placeholder: "ex: 007, A12, B-04"

Atributos HTML:
  inputMode="text"
  autoCapitalize="characters"
  autoCorrect="off"
  spellCheck={false}
  maxLength={64}
  className inclui: font-mono text-lg text-center tracking-widest

Interface para extensibilidade futura (preparar mas não implementar no MVP):
  inputSource: 'manual' | 'nfc' | 'bluetooth' | 'keyboard_emulation'
  onChipCodeResolved: (code: string, source: ChipInputSource) => void
  MVP: inputSource sempre 'manual'

═══════════════════════════════════════
CAMPOS DE ENTIDADES
═══════════════════════════════════════

Espécie:
  nome popular (obrigatório, único, máx 100)
  nome científico (opcional, máx 150)
  descrição/observações (opcional, textarea)

Tanque:
  nome/código (obrigatório, único)
  capacidade (inteiro positivo + unidade texto)
  largura em metros (opcional, decimal)
  altura em metros (opcional, decimal)
  status: Ativo | Inativo
  observações (opcional)
  → Tanque inativo: oculto em todos os selects de outros formulários

Chip:
  código: string A-Z0-9- ≤ 64 chars, obrigatório, único após normalização
  status: Vinculado | Disponível | Danificado | Recuperado
  → Chip danificado: NÃO aparece no select de nova associação

Animal:
  espécie (select de espécies ativas)
  tanque (select de tanques ativos)
  observações (opcional)
  data de cadastro (automática)

Vínculo Chip-Animal:
  chip (ChipCodeInput + lookup)
  animal (select de animais sem chip ativo)
  data de início (padrão: hoje, não pode ser futura)
  observações (opcional)
  → Motivo de encerramento: Abate | Morte natural | Perda | Dano no chip | Correção manual

═══════════════════════════════════════
REGRAS ABSOLUTAS DE IMPLEMENTAÇÃO
═══════════════════════════════════════

1.  Código de chip: string em todo o ciclo (input → state → API → exibição → CSV).
2.  ChipCodeInput: maxLength=64, normalizar, nunca coerção numérica.
3.  Ações perigosas (abate, encerrar vínculo, excluir): SEMPRE modal com resumo.
     Botão de confirmar: variant danger. Enter NÃO confirma. Foco inicial: Cancelar.
4.  Histórico: sempre read-only, cronológico decrescente, paginado (20/página).
5.  Bloqueio de chip já vinculado: mensagem específica que nomeia o animal atual.
6.  Bloqueio de animal com chip ativo: mensagem específica que nomeia o chip atual.
7.  Telas "em breve": PlaceholderPage sem lógica, SEM link no menu.
8.  Validação: erros visíveis apenas após blur ou primeiro submit.
9.  Estados obrigatórios em TODAS as telas: empty (ícone + texto), loading (skeleton),
     error (mensagem legível + retry), success (toast ou estado atualizado).
10. Tabelas viram MobileCardList em < 640px. Nunca scroll horizontal forçado.
11. Acessibilidade mínima:
     - label associado a todo input
     - aria-live="polite" nos ResultCards
     - role="dialog" aria-modal="true" em modais com trap de foco
     - aria-label em botões icon-only
     - Contraste mínimo 4.5:1 texto/fundo
12. Sem dados mock em produção. Loading states reais.
13. Tailwind: sem valores literais no JSX. Classes apenas do config.
14. Exportação CSV: código de chip como string (sem coerção). Preview de quantidade antes do download.
15. Filtros e busca: SEMPRE server-side. URL query params refletem estado.

═══════════════════════════════════════
COMPORTAMENTOS UX CRÍTICOS
═══════════════════════════════════════

Painel (/dashboard):
  Widget de consulta de chip é o elemento PRINCIPAL.
  Em mobile, aparece ACIMA das métricas.
  ResultCard mostra estado do chip ANTES de qualquer ação.

Abate individual (3 etapas obrigatórias):
  Etapa 1: identificar → ChipCodeInput → lookup → confirmar animal
  Etapa 2: dados → data, peso, pergunta sobre chip (obrigatória)
  Etapa 3: resumo → botão danger → confirm → redirect readonly
  Não pode pular etapas. Enter não confirma na etapa 3.

Encerramento de vínculo:
  Modal com: chip, animal, tanque, dias ativos
  Motivo obrigatório (enum de 5 opções)
  Botão danger. Enter não confirma.

Animal abatido (/animais/[id]):
  Banner vermelho no topo
  Todos os campos readonly
  Nenhum botão de ação operacional visível

Histórico (chip e animal):
  Read-only sem exceções
  Código de chip em font-mono
  Datas em dd/mm/aaaa (sem formato relativo)
  Paginado (20/página), agrupamento por ano como toggle opcional

Exportação CSV (/relatorios):
  Preview: "X registros com os filtros atuais. Exportar?"
  Código de chip nunca exportado como número

═══════════════════════════════════════
AMBIGUIDADES
═══════════════════════════════════════

Ao encontrar comportamento não especificado:
  · Implemente o caminho mais conservador (menos destrutivo)
  · Marque com: // TODO(spec): [descreva a dúvida]
  · Não implemente funcionalidades não documentadas

Referência completa: fisher_control_spec.md (documento de especificação v1.2)
```
