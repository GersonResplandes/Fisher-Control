# Fundação De Frontend E UX Operacional

## Objetivo

Definir a base de interface antes de implementar telas definitivas.

O frontend deve ser criado depois da Fase 0 e deve seguir esta direção desde o primeiro layout.

## Direção De Produto

O Fisher Control deve parecer um sistema de operação e manejo, não uma landing page.

Prioridades:

- Leitura rápida.
- Repetição de tarefas sem atrito.
- Prevenção de erro humano.
- Feedback claro após ações.
- Separação entre cadastro, operação e consulta.
- Uso confortável em celular, tablet e desktop.
- Fluxos preparados para leitor externo atual e futuras fontes de leitura.

## Stack Visual

- Next.js.
- React.
- TypeScript.
- Tailwind CSS.
- Ícones com `lucide-react`.

## Estrutura Alvo

```text
web/src/
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

## Fluxo Principal Do Operador

1. Ler o código do chip no leitor externo.
2. Informar o código no sistema.
3. Localizar animal ativo associado ou selecionar animal sem chip ativo.
4. Associar chip quando necessário.
5. Registrar evento operacional.
6. Encerrar vínculo quando houver abate, morte, perda, dano ou correção.
7. Consultar histórico do animal ou do chip.

## Componentes Base

Criar componentes reutilizáveis para:

- Botões.
- Inputs.
- Selects.
- Textareas.
- Modal.
- Toast.
- Tabela responsiva.
- Paginação.
- Badge de status.
- Estado vazio.
- Estado de erro.
- Estado de carregamento.
- Confirmação de ação perigosa.

Componentes de UI não devem conhecer regra de negócio.

## Identidade Visual

Direção: aquática operacional.

Preferências:

- Modo claro como padrão.
- Teal, cyan e azul profundo como acentos.
- Fundo claro com sutileza aquática.
- Cards funcionais com leve profundidade.
- Botões vivos e legíveis.
- Estados de status contrastantes.

Evitar:

- Dashboard SaaS genérico.
- Landing page.
- Decoração sem função.
- Cards dentro de cards.
- Texto explicando a própria interface.

## Estados Obrigatórios

Toda tela de negócio deve prever:

- carregando;
- erro;
- sucesso;
- lista vazia;
- ação bloqueada;
- ação perigosa com confirmação.

## Acessibilidade E Responsividade

Obrigatório:

- Labels associados a inputs.
- Foco visível.
- Contraste adequado.
- Botões com texto ou tooltip quando usar só ícone.
- Layout sem sobreposição em mobile.
- Tabelas com alternativa em cards ou rolagem controlada.

## Critério Para Criar Tela

Uma tela só deve entrar quando:

- Usa o layout base existente.
- Reaproveita componentes de UI.
- Usa o cliente HTTP oficial.
- Trata carregamento, erro e vazio.
- Tem validação visível.
- Funciona em desktop e mobile básico.

