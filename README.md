# equipping-stack-docs

Plugin/skill para agentes de código que gera, na hora certa, **skills de documentação locais ao projeto** com docs na versão que o projeto realmente usa — via [Context7](https://context7.com) (API REST anônima, sem MCP, sem API key).

O conhecimento de treino do modelo sobre frameworks desatualiza rápido — e falha mais forte em **sistemas legados**, onde o modelo escreve os idiomas de hoje contra as APIs de ontem. Este plugin resolve os dois lados: stacks novas (docs atuais da versão exata) e legados (docs da versão antiga, com anotações de delta baseadas em evidência).

## Sumário

- [Instalação](#instalacao)
- [O que funciona em cada agente](#o-que-funciona-em-cada-agente)
- [Como funciona](#como-funciona)
- [Quando dispara](#quando-dispara)
- [Uso como skill avulsa (qualquer agente)](#uso-como-skill-avulsa-qualquer-agente)
- [Compatibilidade e requisitos](#compatibilidade-e-requisitos)

## Instalação

A instalação muda conforme o agente. Se você usa mais de um, instale em cada um separadamente.

> **Repositórios privados.** Todos os repos da Framework System são privados. Qualquer comando abaixo
> que busque pela URL exige que você já esteja autenticado no GitHub (`gh auth login`, credential
> helper ou chave SSH no `ssh-agent`). Sem isso o download falha com erro de autenticação, não de
> "não encontrado".

### Claude Code

Entrega completa: skill, comandos de barra e templates.

- Registre o marketplace:

  ```bash
  /plugin marketplace add Framework-System/frwk-plugins
  ```

- Instale o plugin:

  ```bash
  /plugin install equipping-stack-docs@frwk-plugins
  ```

### GitHub Copilot CLI

O Copilot CLI lê o mesmo `marketplace.json` do Claude Code.

- Registre o marketplace:

  ```bash
  copilot plugin marketplace add Framework-System/frwk-plugins
  ```

- Instale o plugin:

  ```bash
  copilot plugin install equipping-stack-docs@frwk-plugins
  ```

### GitHub Copilot (VS Code, JetBrains, cloud agent, code review)

O Copilot descobre skills por diretório. Copie a skill para o repositório onde vai trabalhar:

```bash
mkdir -p .agents/skills
cp -R /caminho/para/equipping-stack-docs/skills/equipping-stack-docs .agents/skills/
```

Ou para o diretório pessoal, valendo em todos os projetos: `~/.copilot/skills/`.

### Codex (app e CLI)

O repositório traz `.codex-plugin/plugin.json` com a skill declarada. O plugin não está publicado no
marketplace oficial do Codex — instale a partir do repositório clonado, ou copie a skill:

```bash
mkdir -p ~/.agents/skills
cp -R /caminho/para/equipping-stack-docs/skills/equipping-stack-docs ~/.agents/skills/
```

No Codex a skill é acionada por `$equipping-stack-docs` ou pela descrição dela.

### Cursor

**Testado com o Cursor Agent 2026.08.11.** O Cursor lê o mesmo `marketplace.json` do Claude Code,
mas exige a URL completa — o atalho `owner/repo` é recusado com `Invalid URL format`.

```bash
cursor-agent login
cursor-agent plugin marketplace add github.com/Framework-System/frwk-plugins
```

O registro vale para a conta e o Cursor já clona os repositórios no ato. A ativação de cada plugin
é feita no app: o CLI só tem `plugin marketplace`.


### Factory Droid

- Registre o marketplace:

  ```bash
  droid plugin marketplace add https://github.com/Framework-System/frwk-plugins
  ```

- Instale o plugin:

  ```bash
  droid plugin install equipping-stack-docs@frwk-plugins
  ```

### Gemini CLI

- Instale a extensão:

  ```bash
  gemini extensions install https://github.com/Framework-System/equipping-stack-docs
  ```

- Atualize depois:

  ```bash
  gemini extensions update equipping-stack-docs
  ```

A extensão injeta o `GEMINI.md`, que inclui o conteúdo da skill no contexto da sessão.

### Kimi Code

O manifesto `.kimi-plugin/plugin.json` declara a skill.

```text
/plugins install https://github.com/Framework-System/equipping-stack-docs
```

### Antigravity

```bash
agy plugin install https://github.com/Framework-System/equipping-stack-docs
```

### OpenCode

O OpenCode descobre skills por diretório. **Testado com o OpenCode 1.18.21.** Clone o repositório e
declare o caminho no `opencode.json` do seu projeto — note que `skills` é um **objeto com `paths`**,
não uma lista:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "skills": { "paths": ["/caminho/para/equipping-stack-docs/skills"] }
}
```

Confira com `opencode debug skill`. O OpenCode não recarrega config a quente: reinicie depois de editar.


### pi

```bash
pi install git:github.com/Framework-System/equipping-stack-docs
```

### Qualquer outro agente com suporte a Agent Skills

A skill segue o formato aberto [Agent Skills](https://agentskills.io). Copie
`skills/equipping-stack-docs/` para o diretório de skills do seu agente — os caminhos mais comuns são
`.agents/skills/` (no repositório) e `~/.agents/skills/` (pessoal).

Sem descoberta automática de skills, cole o conteúdo de
[`skills/equipping-stack-docs/SKILL.md`](skills/equipping-stack-docs/SKILL.md) no contexto do agente
(`AGENTS.md`, arquivo de instruções do projeto, system prompt) e peça: *"siga a skill equipping-stack-docs"*.

## O que funciona em cada agente

Testado de verdade com o GitHub Copilot CLI 1.0.80 e com o validador de referência da spec
(`skills-ref`), não inferido da documentação.

| Recurso | Claude Code | Copilot CLI | Demais agentes |
|---|---|---|---|
| Skill (`SKILL.md`) | sim | sim | sim |
| Scripts, templates e referências | sim | sim | sim |
| Comandos (`commands/`) | sim, como `/equipping-stack-docs:<cmd>` | sim, viram skills soltas | não |
| Subagentes em paralelo | — | — | — |
| Hooks de ciclo de vida | — | — | — |

O padrão aberto [Agent Skills](https://agentskills.io) cobre `SKILL.md` + `scripts/` +
`references/` + `assets/`. O Copilot CLI vai além e também importa `commands/` — mas **num espaço
de nomes plano, sem o prefixo do plugin**.

Este plugin não tem comandos de barra: ele é só a skill. Em qualquer agente, incluindo o Claude
Code, o acionamento é o mesmo — descreva o que quer e o agente carrega a skill pela descrição dela.

## Como funciona

1. **Descobre a stack** em cascata: design aprovado → manifestos (`package.json`, `pom.xml`, `go.mod`...) → inferência do código (imports, jars, `web.xml`, script tags — versão marcada como aproximada).
2. **Confirma com você** a lista (tecnologia, versão real do projeto, fonte) antes de gastar qualquer chamada.
3. **Despacha um subagente por tecnologia**, cada um consultando a API anônima do Context7 e destilando uma skill de ~150 linhas em `.claude/skills/<tech>-<major>-docs/` do seu projeto.
4. **Versões antigas**: escada de resolução (versão exata → vizinha de mesmo major → docs latest em modo delta). Anotações `[not in 2.x]` só entram com evidência nas próprias docs baixadas (guias de migração, notas "since v") — nunca da memória de treino. Distância grande demais → variante query-only honesta (`version-gap`).
5. As skills geradas são versionadas no git do seu projeto e disparam automaticamente quando qualquer agente futuro tocar código daquela stack.

Falha do Context7 nunca bloqueia seu fluxo — degradação é sempre para variantes honestas, nunca para silêncio ou docs enganosas.

## Quando dispara

- **Com design/spec aprovado**: após o refinamento do design, antes do plano de implementação — o plano já nasce referenciando as doc skills geradas.
- **Autônomo em codebase existente**: ao começar trabalho substancial (feature, mudança multi-arquivo) num projeto cuja stack não tem doc skills locais. Hotfix trivial não interrompe — guarda anti-ruído explícita.


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
- **Fluxos de planejamento**: se o ambiente tiver uma skill de planos de implementação (ex.: writing-plans), o passo final a invoca informando quais doc skills existem; sem ela, o plugin reporta as skills geradas e devolve o controle. Funciona sozinho.
- **Testado de ponta a ponta** contra a API real do Context7 nos cenários: stack nova com versão exata, legado com versão vizinha (Spring Boot 2.1), legado sem manifesto (inferência de Struts por `web.xml`/jar), e AngularJS 1.x (resolução do repo arquivado correto, sem vazamento de Angular moderno).
