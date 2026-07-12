# equipping-stack-docs

Plugin/skill para agentes de código que gera, na hora certa, **skills de documentação locais ao projeto** com docs na versão que o projeto realmente usa — via [Context7](https://context7.com) (API REST anônima, sem MCP, sem API key).

O conhecimento de treino do modelo sobre frameworks desatualiza rápido — e falha mais forte em **sistemas legados**, onde o modelo escreve os idiomas de hoje contra as APIs de ontem. Este plugin resolve os dois lados: stacks novas (docs atuais da versão exata) e legados (docs da versão antiga, com anotações de delta baseadas em evidência).

## Como funciona

1. **Descobre a stack** em cascata: design aprovado → manifestos (`package.json`, `pom.xml`, `go.mod`...) → inferência do código (imports, jars, `web.xml`, script tags — versão marcada como aproximada).
2. **Confirma com você** a lista (tecnologia, versão real do projeto, fonte) antes de gastar qualquer chamada.
3. **Despacha um subagente por tecnologia**, cada um consultando a API anônima do Context7 e destilando uma skill de ~150 linhas em `.claude/skills/<tech>-<major>-docs/` do seu projeto.
4. **Versões antigas**: escada de resolução (versão exata → vizinha de mesmo major → docs latest em modo delta). Anotações `[not in 2.x]` só entram com evidência nas próprias docs baixadas (guias de migração, notas "since v") — nunca da memória de treino. Distância grande demais → variante query-only honesta (`version-gap`).
5. As skills geradas são versionadas no git do seu projeto e disparam automaticamente quando qualquer agente futuro tocar código daquela stack.

Falha do Context7 nunca bloqueia seu fluxo — degradação é sempre para variantes honestas, nunca para silêncio ou docs enganosas.

## Quando dispara

- **Com design/spec aprovado** (funciona ao lado do [Superpowers](https://github.com/obra/superpowers): após o brainstorming, antes do writing-plans).
- **Autônomo em codebase existente**: ao começar trabalho substancial (feature, mudança multi-arquivo) num projeto cuja stack não tem doc skills locais. Hotfix trivial não interrompe — guarda anti-ruído explícita.

## Instalação

A instalação varia conforme o harness. O coração do plugin é um único diretório de skill (`skills/equipping-stack-docs/`) — qualquer ambiente que leia o formato `SKILL.md` consegue usá-lo (veja [Uso como skill avulsa](#uso-como-skill-avulsa-qualquer-agente)).

### Claude Code

Via marketplace da Framework System (recomendado):

```
/plugin marketplace add Framework-System/frwk-plugins
/plugin install equipping-stack-docs@frwk-plugins
```

Ou direto deste repositório:

```
/plugin marketplace add Framework-System/equipping-stack-docs
/plugin install equipping-stack-docs@equipping-stack-docs
```

### Codex (app e CLI)

O repositório traz o manifesto `.codex-plugin/plugin.json` com a skill declarada. O plugin não está publicado no marketplace do Codex — instale a partir deste repositório clonado, apontando o Codex para o diretório do plugin, ou use o [uso como skill avulsa](#uso-como-skill-avulsa-qualquer-agente) copiando a skill para o diretório de skills do Codex.

### Cursor

O repositório traz `.cursor-plugin/plugin.json` com a skill declarada. Instale a partir deste repositório (não publicado no marketplace do Cursor) ou copie a skill para o diretório de skills do seu ambiente.

### Kimi Code

```
/plugins install https://github.com/Framework-System/equipping-stack-docs
```

O manifesto `.kimi-plugin/plugin.json` inclui o mapeamento de ferramentas do Kimi (AskUserQuestion para a confirmação da stack, Agent para os subagentes geradores).

### Pi

```
pi install git:github.com/Framework-System/equipping-stack-docs
```

Para desenvolvimento local: `pi -e /caminho/para/equipping-stack-docs`. O Pi tem skills nativas; o `package.json` declara `pi.skills`.

### Gemini CLI

```
gemini extensions install https://github.com/Framework-System/equipping-stack-docs
```

A extensão injeta o `GEMINI.md`, que inclui o conteúdo completo da skill no contexto da sessão.

### Antigravity

```
agy plugin install https://github.com/Framework-System/equipping-stack-docs
```

### OpenCode

O OpenCode descobre skills por diretório. Clone o repositório e adicione o caminho no seu `opencode.json`:

```json
{ "skills": { "paths": ["/caminho/para/equipping-stack-docs/skills"] } }
```

## Uso como skill avulsa (qualquer agente)

Não precisa de sistema de plugins. A skill é um único arquivo Markdown auto-suficiente — o template do subagente, a escada de versão e o tratamento de erros estão todos embutidos nele:

```bash
git clone https://github.com/Framework-System/equipping-stack-docs.git
# skill só deste projeto:
cp -r equipping-stack-docs/skills/equipping-stack-docs SEU_PROJETO/.claude/skills/
# ou pessoal, para todos os projetos (Claude Code):
cp -r equipping-stack-docs/skills/equipping-stack-docs ~/.claude/skills/
```

Em harnesses sem descoberta automática de skills, cole o conteúdo de [`skills/equipping-stack-docs/SKILL.md`](skills/equipping-stack-docs/SKILL.md) no contexto do agente (arquivo de instruções do projeto, `AGENTS.md`, system prompt) e peça: *"siga a skill equipping-stack-docs"*. Requisitos mínimos do ambiente: executar `curl`, criar arquivos e — idealmente — despachar subagentes; sem subagentes, o próprio agente executa o template de geração para uma tecnologia por vez.

## Compatibilidade e requisitos

- **Context7**: acesso anônimo, sem cadastro e sem API key — decisão de design. O rate limit anônimo é baixo; a skill economiza chamadas (confirmação antes de gerar, reaproveitamento de skills existentes, backoff em 429).
- **Superpowers**: pensado para conviver com o [Superpowers](https://github.com/obra/superpowers) oficial (o passo final invoca `superpowers:writing-plans` quando disponível), mas funciona sozinho — sem planning skill, ele reporta as skills geradas e devolve o controle.
- **Testado de ponta a ponta** contra a API real do Context7 nos cenários: stack nova com versão exata, legado com versão vizinha (Spring Boot 2.1), legado sem manifesto (inferência de Struts por `web.xml`/jar), e AngularJS 1.x (resolução do repo arquivado correto, sem vazamento de Angular moderno).

## Créditos

A estrutura de skills e o layout multi-harness seguem o padrão do [Superpowers](https://github.com/obra/superpowers) de Jesse Vincent (MIT).
