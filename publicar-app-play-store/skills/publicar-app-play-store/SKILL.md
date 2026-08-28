---
name: publicar-app-play-store
description: Usar para publicar um novo app parceiro white-label (Monitor/Tracker/Visits) na Google Play Console, guiando o fluxo completo de criação e configuração via MCP de browser, sem clicar no botão final de publicação.
---

# Publicar App na Play Store

## Overview

Guia a criação e configuração completa de um app parceiro white-label (Monitor/Tracker/Visits) na Google Play Console, navegando a interface via MCP `playwright` (registrado em `.mcp.json`) — sem instanciar nenhum LLM externo, sem custo de API separado.

Cada clique/preenchimento passa pelo sistema de permissão normal do Claude Code — isso já funciona como "modo híbrido" (o colaborador vê e aprova cada ação). Não é necessário simular pausas extras.

**Execução em blocos**: preencher cada subseção usando `browser_fill_form` para múltiplos campos de uma vez, em vez de um `browser_click`/`browser_type` por campo. O operador acompanha a tela ao vivo, então não é preciso tirar `browser_snapshot` pra confirmar estado.

## Passo 0 — Coletar dados

Perguntar ao colaborador, antes de começar:

- Nome do app (`APP_NAME`)
- Nome da marca/cliente (`BRAND_NAME`)
- Email de contato do cliente (`CLIENT_EMAIL`)
- Site do cliente (`CLIENT_WEBSITE`)
- URL da política de privacidade (S3, `mobiltracker-partner-prefs`) — gerada rodando o script da variante correta, no repo `mobiltracker/mobiltracker-scripts`, em `partner/manual_scripts/app_android_monitor_privacy` / `app_android_tracker_privacy` / `app_android_visits_privacy`, com os dados dessa empresa (businessname, cnpj, email, ramo, appname, domain)
- Caminho local do `.aab` gerado
- Nome do pacote (`PACKAGE_NAME`) — precisa ser exatamente o mesmo pacote do `.aab` informado acima
- Caminho local do ícone (512x512) e do recurso gráfico (1024x500)
- Caminhos das 4 screenshots (Login, Mapa last location, Mapa histórico, Comandos)
- Qual variante do app é (Monitor / Tracker / Visits)
- Caminho local do CSV de segurança dos dados da variante correta — está junto com os scripts de política de privacidade, no mesmo repo `mobiltracker/mobiltracker-scripts`, em `partner/manual_scripts/app_android_monitor_privacy/data_safety_export.csv` / `app_android_tracker_privacy/data_safety_export_app_tracker.csv` / `app_android_visits_privacy/data_safety_export_app_visits.csv`

Não avançar sem todos esses dados.

**Pré-requisito**: confirmar que o JSON de service account do cliente já está salvo no bucket S3 `google-play-service-accounts` (renomeado como `<BRAND_NAME>-api-...`) — feito manualmente pelo colaborador, fora do escopo desta skill, mas não precisa esperar a publicação pra fazer isso.

**Aviso**: o upload de ícone, recurso gráfico, screenshots e `.aab` não é feito pela automação — na etapa de preparação de cada um (antes de chegar na tela de upload), deixar os arquivos prontos/conferidos e pedir ao colaborador para fazer o upload manualmente. Avisar isso logo no início, sem esperar chegar nesse ponto.

**Aviso**: pop-ups de onboarding/novidades do Play Console (ex. "Gerencie as mudanças...") podem aparecer sobre a tela a qualquer momento, bloqueando cliques. Se um clique falhar por causa disso, tirar um `browser_snapshot` pra confirmar, fechar o pop-up (clicar em "Avançar" até o fim do carrossel, ou fechar/Esc) e repetir a ação original.

## REGRA ABSOLUTA

**Nunca clicar em "Iniciar lançamento para produção", "Publicar" ou qualquer botão de envio final do app.** Ao chegar nesse ponto, parar e avisar:

> PRONTO PARA PUBLICAR. Revise a versão e clique em "Iniciar lançamento para produção" para finalizar.

A tarefa termina aí — o colaborador faz o clique final manualmente.

## Passo 1 — Criar o app

1. Navegar para `https://play.google.com/console/`
2. Login com `mobiltracker.brazil@gmail.com` (se pedir verificação extra do Google por detectar navegação automatizada, avisar o colaborador e deixar ele resolver na janela aberta)
3. Selecionar a conta de desenvolvedor
4. Clicar em "Criar app"
5. Preencher: nome = `APP_NAME`, nome do pacote = `PACKAGE_NAME`, idioma padrão = Português (Brasil), tipo = App (não Jogo), preço = Gratuito, aceitar declarações de Políticas do programa e Leis de exportação dos EUA
6. Clicar em "Criar app"

## Passo 2 — Configuração inicial

Painel > Configurar o app > Ver etapas. Preencher cada subseção em bloco (todos os campos dela em uma única chamada de `browser_fill_form` quando a tela permitir) e salvar antes de ir para a próxima:

- **Política de privacidade**: inserir a URL da política, salvar
- **Acesso de apps**: "Recursos são restritos", credencial de teste `android@teste.com` / `123456` — confirmar que a conta "TK VALIDADOR INFRA" está compartilhada e online nessa credencial antes de salvar
- **Anúncios**: "O app não tem anúncios", salvar
- **Classificação de conteúdo**: email `mobiltrackerbrazil@gmail.com`, categoria "Todos os Outros Tipos de Aplicações", questionário: "Conteúdo Online" = Sim, todas as demais perguntas = Não (ler cada uma antes de confirmar que faz sentido). Ao terminar, clicar em "Salvar" (o botão "Avançar" fica desabilitado até isso) antes de avançar para a etapa "Resumo" e enviar
- **Público-alvo e conteúdo**: faixa etária "Maiores de 18 anos", salvar
- **Segurança dos dados**: importar o CSV informado no Passo 0 (todas as respostas do questionário vêm do CSV e variam por variante — Monitor/Tracker/Visits têm respostas diferentes, inclusive em localização, exclusão de dados e compartilhamento de identificadores; nunca preencher esses campos de memória). Avançar para "Coleta de dados e segurança" e conferir que veio tudo preenchido pelo CSV (coleta de dados obrigatórios, criptografia em trânsito, métodos de criação de contas, exclusão de dados, tipos de dados e o questionário de cada tipo). Se algum campo não vier marcado pelo CSV, PARAR e perguntar ao colaborador a resposta correta — não inventar. Salvar
- **Apps governamentais**: seção própria — "Não", salvar
- **Recursos financeiros**: seção própria — "Meu app não oferece recursos financeiros", salvar
- **Apps de saúde**: seção própria — "Meu app não tem recursos de saúde", salvar
- **ID de publicidade**: seção própria dentro de "Monitorar e aprimorar". "O app usa ID de publicidade?" = "Não", salvar
- **Categoria e contato**: App, categoria "Empresa", marcar "Marketing externo". Email e site costumam falhar com `browser_fill_form` nessa tela — preencher `CLIENT_EMAIL` e `CLIENT_WEBSITE` campo a campo com `browser_click` no campo seguido de `browser_type`. Salvar
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

  - Upload do ícone 512x512, gráfico 1024x500 e das 4 screenshots: pedir ao colaborador para fazer esse upload manualmente (não tentar pela automação). Depois, tirar um `browser_snapshot` e confirmar que cada um foi de fato anexado ao campo certo — ícone, recurso gráfico e as 4 capturas de tela são slots separados, é preciso clicar "Adicionar recursos" em cada um
  - Declaração de recursos de IA: "Não rotular recursos"
  - Salvar

## Passo 3 — Publicar

- **Países e regiões**: Testar e lançar > Produção > Países/regiões, adicionar todos, salvar
- **Produção direta**: Testar e lançar > Produção > Versões > "Criar e publicar uma versão" — tentar publicar direto em produção, pulando o Teste fechado quando o Play Console permitir:
  - Upload do `.aab`: pedir ao colaborador para fazer esse upload manualmente (não tentar pela automação). Depois de qualquer upload, sempre conferir com `browser_snapshot` que foi salvo antes de seguir para outro campo
  - Assinatura de apps: "Gerenciar Preferências" > baixar chave de assinatura > copiar valor hex > rodar a workflow `_export_signing_key_output_zip.yml` em `github.com/mobiltracker/mobiltracker-app-monitor/actions` com esse hex > baixar o `.zip` gerado > fazer upload do `.zip` no campo de assinatura > aceitar os termos
  - Verificar fingerprints em Configuração > Assinatura de apps:
    - MD5: `FA:59:F1:46:48:6A:22:4B:7E:CD:B6:FF:6F:63:14:8A`
    - SHA1: `E7:4E:D4:69:CA:63:13:A5:5E:AD:02:DA:B8:ED:75:C8:6E:36:35:15`
    - SHA256: `6B:82:48:C8:1C:EA:AB:CF:D5:54:F1:70:56:A9:D8:5F:2A:3E:99:04:64:9C:8E:2F:C2:3B:B1:DC:3A:D4:A3:FA`
  - Notas da versão: "Lançamento."
  - Salvar
- **Fallback — Teste fechado**: se o Play Console bloquear a produção direta e exigir teste fechado antes (ex.: conta nova, política do Google), seguir por Testes > Teste fechado, adicionar todos os países, selecionar a lista de testadores já existente ("Testadores") — não criar uma lista nova com e-mails arbitrários, pois a Play Console exige que sejam contas Google reais e válidas; criar nova versão com o mesmo `.aab`/assinatura/notas acima, salvar e enviar para revisão. Depois, repetir o upload em Painel > Testar e lançar > Produção > Versões > Editar versão
- **Publicar**: Versões > Produção > Avaliar Versão, navegar até "Iniciar lançamento para produção" — aplicar a REGRA ABSOLUTA e parar aqui

## Regras

- Agrupar preenchimentos relacionados em uma única chamada de ferramenta (`browser_fill_form`) sempre que a tela permitir
- Nunca pular a REGRA ABSOLUTA, em nenhum modo de execução
- Apps governamentais, Recursos financeiros e Apps de saúde são telas separadas (não sub-passos de Segurança dos dados) — abrir e salvar cada uma individualmente
- Nunca inventar resposta para uma pergunta do questionário que não esteja listada aqui — se a Play Console mostrar uma pergunta nova/diferente, parar e perguntar ao colaborador
- Se uma tool do MCP falhar em encontrar/interpretar um elemento da tela, parar e mostrar o que está vendo — não adivinhar clique
- Nunca commitar nada relacionado a esse fluxo sem aprovação explícita (regra do projeto)
- Upload de arquivo (ícone, gráfico, screenshots, `.aab`): nunca tentar pela automação — pedir direto ao colaborador para fazer manualmente (`.zip` de assinatura é exceção, pequeno o suficiente pra automação)
- Depois de qualquer upload manual do colaborador ou preenchimento de campos (nome de versão, notas etc.), salvar imediatamente ("Salvar" / "Salvar como rascunho" / "Adicionar recursos") e conferir com `browser_snapshot` que a ação realmente persistiu antes de prosseguir para outro campo ou tela
- Ao concluir cada subseção (ex.: Política de privacidade, Acesso de apps, Segurança dos dados, ID de publicidade, Categoria e contato), reportar uma linha de status ao colaborador (ex.: `[Política de privacidade] preenchida`) antes de seguir para a próxima

## Erros comuns

| Erro                                                      | Causa                                                                                                      |
| --------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| Tool do MCP `playwright` não aparece disponível           | `.mcp.json` não configurado ou Node/`npx` não instalado                                                    |
| Login pede verificação extra do Google                    | Google detectou navegação automatizada — resolver manualmente na janela aberta                             |
| CSV de segurança dos dados rejeitado ou com dados errados | Upload do arquivo da variante errada do app (Monitor vs Tracker vs Visits)                                 |
| Passo trava sem avançar                                   | Play Console mudou o texto/estrutura da tela — parar e confirmar com o colaborador antes de tentar de novo |
