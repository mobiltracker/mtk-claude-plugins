---
name: create-issue
description: Cria uma issue GitHub estruturada para o time de automação da mobiltracker a partir de qualquer entrada (imagem, texto, screenshot, conversa)
---

Quando esta skill for invocada, siga os passos abaixo sem desviar da sequência.

Todas as interações com o GitHub são feitas via `gh` CLI (Bash), não via MCP.

## 0. Pré-requisito de autenticação

Os comandos de Project (passos 8 e 9) exigem o scope `project` no token do `gh`.
Se um comando falhar com erro de scope faltando (`missing required scopes`),
avise o usuário para rodar `gh auth refresh -s project` uma vez e tentar de novo.

## 1. Processar a entrada

Aceite qualquer formato: imagem, screenshot, texto livre, conversa com cliente, visão geral.
Extraia o máximo possível de informação antes de fazer qualquer pergunta.

## 2. Consultar o banco de dados

Sempre que fizer sentido para a tarefa — seja para alterar o banco (nova coluna, tabela, migration)
ou apenas para entender dados/estruturas existentes — consulte o repositório `mobiltracker/local-db-manager`:

```
gh api repos/mobiltracker/local-db-manager/... (ex: gh api repos/mobiltracker/local-db-manager/contents/<path>)
ou gh search code --repo mobiltracker/local-db-manager <termo>
```

Identifique quais tabelas, schemas ou estruturas são relevantes para a tarefa descrita.

Sempre confirme com o usuário se a tabela identificada é a correta para o escopo da tarefa:
"A tarefa envolve a tabela <nome>, é essa mesmo?"

Use esse contexto para enriquecer a seção Solução com precisão técnica.

## 3. Inferir o repositório

Rode `gh repo list mobiltracker --limit 200` para listar os repositórios reais da org e
validar que o repo inferido existe com esse nome (evita apostar em nome desatualizado,
renomeado ou removido).

Com base no conteúdo da tarefa, infira o repo mais provável. Exemplos comuns:
- `mobiltracker/mobiltracker-scripts` — scripts e automações gerais
- `mobiltracker/mtkpay-automacao-api` — API de automação do MtkPay
- `mobiltracker/n8n-workflows` — workflows n8n
- `mobiltracker/vanescola-api` — API do VanEscola
- `mobiltracker/mtkcontract` — contratos
- `mobiltracker/mtkpay-webhooks` — webhooks do MtkPay
- `mobiltracker/mtkpay-web` — api e tela do MtkPay
- `mobiltracker/mobiltracker` — sistema principal
- `mobiltracker/cs-router` — roteamento CS
- `mobiltracker/vanescola-router` — roteamento VanEscola

Se a tarefa não se encaixar claramente em nenhum desses, procure na listagem completa por nome/descrição.

## 4. Perguntar apenas o necessário

Priorize criar a tarefa sem fricção: evite uma bateria de perguntas.
Pergunte quando a informação for bloqueante para construir a tarefa, ou quando uma inferência importante
(ex: repositório, tabela) tiver risco real de estar errada — nesses casos vale confirmar rápido em vez de seguir errado.
Seja específico: informe exatamente o que não ficou claro (ou o que está inferindo) e por que precisa confirmar.

## 5. Gerar o título

Formato: `[Área]: ação objetiva`
Máximo 60 caracteres. Autoexplicativo — quem ler no kanban já entende o escopo.

Exemplos:
- `CI: paralelizar steps de build`
- `MtkPay: corrigir webhook de estorno`
- `VanEscola: adicionar índice na tabela de rotas`

## 6. Montar o body

```
## Contexto
<cenário atual — por que isso importa, qual o impacto>

## Problema
<o que está bloqueando, falhando ou precisa mudar — seja específico>

## Solução
<abordagem proposta, tabelas/componentes envolvidos, critérios de aceite>
```

Tom: direto e técnico. Time focado em agilidade e entrega com qualidade — sem rodeios.

## 7. Criar a issue

```
gh issue create --repo mobiltracker/<repo> --title "<título>" --body "<body>"
```

O comando retorna a URL da issue criada — guarde para os próximos passos.

## 8. Adicionar ao projeto Automação

Adicione a issue ao Project #9 da org `mobiltracker` e capture o ID do item criado:

```
gh project item-add 9 --owner mobiltracker --url <issue-url> --format json
```

## 9. Definir status no projeto

Descubra o ID do campo `Status` e o ID da opção `Week Tasks`:

```
gh project field-list 9 --owner mobiltracker --format json
```

Em seguida, aplique o status ao item criado no passo 8:

```
gh project item-edit --id <item-id> --project-id <project-id> --field-id <status-field-id> --single-select-option-id <week-tasks-option-id>
```

Se qualquer comando dos passos 8/9 falhar por falta de scope, siga o passo 0 e avise o usuário.

## 10. Retornar

Informe:
- Título gerado
- Link da issue criada
- Confirmação de que foi adicionada ao Project Automação com status `Week Tasks`
  (ou aviso caso isso tenha falhado, com a instrução do passo 0)
