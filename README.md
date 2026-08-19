# mtk-claude-plugins

Repositório de **plugins do Claude Code** da MTK. Funciona como um _marketplace_: qualquer pessoa pode adicionar este repositório no Claude Code e instalar os plugins/skills daqui.

## Conceitos

### Plugin

Um **plugin** é um pacote autocontido que estende o Claude Code com funcionalidades: skills, MCP servers, agents, hooks, etc. Cada plugin vive em sua própria pasta, com um manifesto `.claude-plugin/plugin.json` descrevendo nome, versão, descrição e autor.

### Marketplace

Um **marketplace** é um catálogo (`marketplace.json`) que lista os plugins disponíveis e onde encontrá-los. Este repositório é um marketplace: o arquivo `.claude-plugin/marketplace.json` na raiz lista todos os plugins publicados aqui.

### Skill

Uma **skill** é uma instrução reutilizável que o Claude aprende a executar. É definida em um arquivo `SKILL.md` com um frontmatter YAML (pelo menos o campo `description`, que diz ao Claude quando invocá-la automaticamente) seguido do corpo em markdown com as instruções. Skills ficam em `<plugin>/skills/<nome-da-skill>/SKILL.md` e são descobertas automaticamente quando o plugin é instalado. Depois de instalada, uma skill pode ser chamada explicitamente como `/nome-do-plugin:nome-da-skill`.

### Tools

**Tools** são as capacidades que o Claude usa para agir (ler/editar arquivos, rodar comandos, chamar uma API, etc.). Um plugin pode trazer tools próprias via MCP servers, além das tools nativas do Claude Code.

### MCP (Model Context Protocol)

**MCP** é o protocolo usado para conectar o Claude a serviços/ferramentas externas (APIs, bancos de dados, etc.), expondo novas tools. Um plugin registra seus MCP servers em um arquivo `.mcp.json` na raiz do plugin (chave `mcpServers`). Ao instalar/habilitar o plugin, o Claude Code inicia esses servidores automaticamente.

## Estrutura do repositório

```
mtk-claude-plugins/
├── .claude-plugin/
│   └── marketplace.json              # catálogo do marketplace
├── publicar-app-play-store/          # um plugin
│   ├── .claude-plugin/
│   │   └── plugin.json               # manifesto do plugin
│   ├── .mcp.json                     # MCP servers do plugin (se houver)
│   └── skills/
│       └── publicar-app-play-store/
│           └── SKILL.md              # skill do plugin
└── README.md
```

Cada novo plugin entra como uma nova pasta na raiz, seguindo esse mesmo padrão, e precisa ser adicionado à lista `plugins` em `.claude-plugin/marketplace.json`.

## Configuração inicial (uma vez só)

Antes de instalar qualquer plugin, adicione este repositório como marketplace. Isso só precisa ser feito **uma vez** por pessoa/máquina:

```
/plugin marketplace add Mobiltracker/mtk-claude-plugins
```

Ou via CLI:

```bash
claude plugin marketplace add Mobiltracker/mtk-claude-plugins
```

## Instalar um plugin

Repita este passo para cada plugin do marketplace que você quiser usar:

```
/plugin install publicar-app-play-store@mtk-claude-plugins
```

Ou via CLI:

```bash
claude plugin install publicar-app-play-store@mtk-claude-plugins --scope user
```

O comando de instalação abre um menu para escolher o escopo (`user`, `project` ou `local`).

### Publicando alterações

1. Commitar as mudanças:
   ```bash
   git add <arquivos>
   git commit -m "mensagem do commit"
   ```
2. Dar push:
   ```bash
   git push -u origin master
   ```

O Claude Code **não** detecta automaticamente novos commits. Quem já tem o marketplace adicionado precisa:

- Atualizar o catálogo: `/plugin marketplace update mtk-claude-plugins` (só atualiza a lista disponível — não reinstala o que já está instalado)
- Instalar o que for novo: `/plugin install <novo-plugin>@mtk-claude-plugins`
- Se só mudou uma skill de um plugin já instalado (sem bump de `version`): `/reload-plugins`

> Dica: com _auto-update_ habilitado (aba **Marketplaces** em `/plugin`), o catálogo se atualiza sozinho em segundo plano.

## Usando no Claude Desktop

O marketplace e o `/plugin install` são exclusivos do Claude Code — não funcionam no Claude Desktop, claude.ai (web) ou mobile. Para usar a skill `publicar-app-play-store` no Desktop, replique manualmente:

1. **Copiar a skill**: abra `publicar-app-play-store/skills/publicar-app-play-store/SKILL.md` e cole o conteúdo como uma Skill customizada nas configurações do Claude Desktop.
2. **Configurar o MCP**: edite o config do Desktop e adicione o mesmo servidor registrado em `publicar-app-play-store/.mcp.json`:
   - macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
   - Windows: `%APPDATA%\Claude\claude_desktop_config.json`
   - Linux: `~/.config/Claude/claude_desktop_config.json`
   ```json
   {
     "mcpServers": {
       "playwright": {
         "command": "npx",
         "args": ["@playwright/mcp@latest"]
       }
     }
   }
   ```
3. Reiniciar o Claude Desktop.

> Não sincroniza automaticamente: qualquer atualização feita aqui no repositório precisa ser copiada manualmente de novo para o Desktop.

## Comandos úteis

| Comando                                             | O que faz                                                                         |
| --------------------------------------------------- | --------------------------------------------------------------------------------- |
| `/plugin`                                           | Abre o gerenciador de plugins (Discover, Installed, Marketplaces, Errors)         |
| `/plugin marketplace add <fonte>`                   | Adiciona este repositório como marketplace                                        |
| `/plugin marketplace update <nome>`                 | Atualiza o catálogo do marketplace (busca novos plugins/versões)                  |
| `/plugin marketplace list`                          | Lista os marketplaces adicionados                                                 |
| `/plugin marketplace remove <nome>`                 | Remove um marketplace                                                             |
| `/plugin install <plugin>@<marketplace>`            | Instala um plugin                                                                 |
| `/plugin list`                                      | Lista plugins instalados                                                          |
| `/plugin enable` / `disable <plugin>@<marketplace>` | Habilita/desabilita um plugin sem desinstalar                                     |
| `/plugin uninstall <plugin>@<marketplace>`          | Remove um plugin                                                                  |
| `/plugin details <plugin>@<marketplace>`            | Mostra o que o plugin traz (skills, agents, hooks, MCP servers)                   |
| `/reload-plugins`                                   | Recarrega plugins/skills/hooks/MCP servers sem reiniciar a sessão                 |
| `/skills`                                           | Lista as skills disponíveis (nativas, do projeto, de plugins) e permite favoritar |
| `/mcp`                                              | Lista os MCP servers configurados, status de conexão e permite autenticar         |

Equivalentes via CLI: `claude plugin marketplace add|list|update|remove`, `claude plugin install|uninstall|enable|disable|list|details`.
