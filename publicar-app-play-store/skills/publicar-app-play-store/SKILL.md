---
name: publicar-app-play-store
description: Use when publicando um novo app parceiro white-label (Monitor/Tracker/Visits) na Google Play Console, guiando o fluxo completo de criação e configuração via MCP de browser, sem clicar no botão final de publicação
---

# Publicar App na Play Store

## Overview

Substitui `partner/publicar_app/publish_play_store.py` (que usava `browser-use` + uma `ANTHROPIC_API_KEY` própria, cobrada por token). Aqui, o próprio agente da sessão atual do Claude Code navega a Play Console usando as tools do MCP `playwright` (registrado em `.mcp.json`) — sem instanciar nenhum LLM externo, sem custo de API separado.

Também existe uma automação sem IA em `node/partner/app-creation/` (Puppeteer com seletores fixos `debug-id`) — está **deprecada**, quebra a cada mudança de layout da Play Console. Não usar.

Cada clique/preenchimento passa pelo sistema de permissão normal do Claude Code — isso já funciona como "modo híbrido" (o colaborador vê e aprova cada ação). Não é necessário simular pausas extras.

## Passo 0 — Coletar dados

Perguntar ao colaborador, antes de começar:

- Nome do app (`APP_NAME`)
- Nome da marca/cliente (`BRAND_NAME`)
- Email de contato do cliente (`CLIENT_EMAIL`)
- Site do cliente (`CLIENT_WEBSITE`)
- URL da política de privacidade (S3, `mobiltracker-partner-prefs`)
- Caminho local do `.aab` gerado
- Caminho local do ícone (512x512) e do recurso gráfico (1024x500)
- Caminhos das 4 screenshots (Login, Mapa last location, Mapa histórico, Comandos)
- Qual variante do app é (Monitor / Tracker / Visits) — define qual `data_safety_export.csv` usar:
  - `partner/manual_scripts/app_android_monitor_privacy/data_safety_export.csv`
  - `partner/manual_scripts/app_android_tracker_privacy/data_safety_export_app_tracker.csv`
  - `partner/manual_scripts/app_android_visits_privacy/data_safety_export_app_visits.csv`

Não avançar sem todos esses dados.

## REGRA ABSOLUTA

**Nunca clicar em "Iniciar lançamento para produção", "Publicar" ou qualquer botão de envio final do app.** Ao chegar nesse ponto, parar e avisar:

> PRONTO PARA PUBLICAR. Revise a versão e clique em "Iniciar lançamento para produção" para finalizar.

A tarefa termina aí — o colaborador faz o clique final manualmente.

## Passo 1 — Criar o app

1. Navegar para `https://play.google.com/console/`
2. Login com `mobiltracker.brazil@gmail.com` (se pedir verificação extra do Google por detectar navegação automatizada, avisar o colaborador e deixar ele resolver na janela aberta)
3. Selecionar a conta de desenvolvedor
4. Clicar em "Criar app"
5. Preencher: nome = `APP_NAME`, idioma padrão = Português (Brasil), tipo = App (não Jogo), preço = Gratuito, aceitar declarações de Políticas do programa e Leis de exportação dos EUA
6. Clicar em "Criar app"

## Passo 2 — Configuração inicial

Painel > Configurar o app > Ver etapas. Preencher cada subseção e salvar antes de ir para a próxima:

- **Política de privacidade**: inserir a URL da política, salvar
- **Acesso de apps**: "Recursos são restritos", credencial de teste `android@teste.com` / `123456`, salvar
- **Anúncios**: "O app não tem anúncios", salvar
- **Classificação de conteúdo**: email `mobiltrackerbrazil@gmail.com`, categoria "Todos os Outros Tipos de Aplicações", questionário: "Conteúdo Online" = Sim, todas as demais perguntas = Não (ler cada uma antes de confirmar que faz sentido), enviar
- **Público-alvo e conteúdo**: faixa etária "Maiores de 18 anos", salvar
- **Segurança dos dados**: importar o CSV correto da variante do app (Passo 0), avançar para "Coleta de dados e segurança":
  - Coleta dados obrigatórios: Sim
  - Dados criptografados em trânsito: Sim
  - Métodos de criação de contas: selecionar as opções relevantes mostradas
  - Oferece exclusão de dados: Não
  - Confirmar que os tipos de dados foram marcados pelo CSV (IDs de usuário, interações no app, registros de falhas/diagnóstico/desempenho, identificadores de dispositivo); se algum não vier marcado, marcar manualmente e preencher o questionário de cada tipo:
    - Interações no app: Coletado, não efêmero, coleta obrigatória, motivo = Funcionalidade do app / Análise / Segurança-conformidade-prevenção de fraudes
    - Registros de falhas/diagnóstico/desempenho: Coletado, não efêmero, coleta obrigatória, motivo = Análise / Segurança-conformidade-prevenção de fraudes
    - Identificadores de dispositivo: Coletado E Compartilhado, não efêmero, coleta obrigatória, motivo coleta = Funcionalidade do app / Análise / Mensagens do desenvolvedor / Segurança-conformidade, motivo compartilhado = Análise / Mensagens do desenvolvedor
  - Salvar e avançar
  - Apps governamentais: Não · Recursos financeiros: "Meu app não oferece recursos financeiros" · Apps de saúde: "Meu app não tem recursos de saúde"
- **ID de publicidade**: (em "Monitorar e aprimorar") "O app usa ID de publicidade?" = Não, salvar
- **Categoria e contato**: App, categoria "Empresa", email = `CLIENT_EMAIL`, site = `CLIENT_WEBSITE`, marcar "Marketing externo", salvar
- **Detalhes do app**:
  - Nome = `APP_NAME`
  - Breve descrição: `Exclusivo para usuários registrados {BRAND_NAME}.`
  - Descrição completa:

    ```
    Monitore seus rastreadores {BRAND_NAME} em tempo real.

    Esse aplicativo é destinado exclusivamente aos usuários registrados {BRAND_NAME} para fornecer serviços de monitoramento e controle remoto de rastreadores, permitindo visualização das localizações e status desses dispositivos.
    O aplicativo não funciona sem a infraestrutura necessária da plataforma {BRAND_NAME}.

    Obs.: se você não é um usuário registrado desse serviço, não instale esse aplicativo.
    ```

  - Upload do ícone 512x512, gráfico 1024x500 e das 4 screenshots
  - Salvar

## Passo 3 — Publicar

- **Países e regiões**: Testar e lançar > Produção > Países/regiões, adicionar todos, salvar
- **Teste fechado** (obrigatório antes de produção): Testes > Teste fechado, adicionar todos os países, criar nova versão:
  - Upload do `.aab`
  - Assinatura de apps: "Gerenciar Preferências" > baixar chave de assinatura > copiar valor hex > rodar a workflow `export_signing_key_output_zip.yml` em `github.com/mobiltracker/mobiltracker-app-monitor/actions` com esse hex > baixar o `.zip` gerado > fazer upload do `.zip` no campo de assinatura > aceitar os termos
  - Verificar fingerprints em Configuração > Assinatura de apps:
    - MD5: `FA:59:F1:46:48:6A:22:4B:7E:CD:B6:FF:6F:63:14:8A`
    - SHA1: `E7:4E:D4:69:CA:63:13:A5:5E:AD:02:DA:B8:ED:75:C8:6E:36:35:15`
    - SHA256: `6B:82:48:C8:1C:EA:AB:CF:D5:54:F1:70:56:A9:D8:5F:2A:3E:99:04:64:9C:8E:2F:C2:3B:B1:DC:3A:D4:A3:FA`
  - Notas da versão: "Lançamento."
  - Salvar e enviar para revisão
- **Versão de produção**: Painel > Testar e lançar > Produção > Versões > Editar versão, upload do mesmo `.aab`, notas "Lançamento.", salvar
- **Publicar**: Versões > Produção > Avaliar Versão, navegar até "Iniciar lançamento para produção" — aplicar a REGRA ABSOLUTA e parar aqui

## Regras

- Nunca pular a REGRA ABSOLUTA, em nenhum modo de execução
- Nunca inventar resposta para uma pergunta do questionário que não esteja listada aqui — se a Play Console mostrar uma pergunta nova/diferente, parar e perguntar ao colaborador
- Se uma tool do MCP falhar em encontrar/interpretar um elemento da tela, parar e mostrar o que está vendo — não adivinhar clique
- Nunca commitar nada relacionado a esse fluxo sem aprovação explícita (regra do projeto)

## Erros comuns

| Erro                                                      | Causa                                                                                                      |
| --------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| Tool do MCP `playwright` não aparece disponível           | `.mcp.json` não configurado ou Node/`npx` não instalado                                                    |
| Login pede verificação extra do Google                    | Google detectou navegação automatizada — resolver manualmente na janela aberta                             |
| CSV de segurança dos dados rejeitado ou com dados errados | Upload do arquivo da variante errada do app (Monitor vs Tracker vs Visits)                                 |
| Passo trava sem avançar                                   | Play Console mudou o texto/estrutura da tela — parar e confirmar com o colaborador antes de tentar de novo |
