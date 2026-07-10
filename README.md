# equipping-stack-docs

Plugin standalone para Claude Code que gera, na hora certa, **skills de documentação locais ao projeto** com docs na versão que o projeto realmente usa — via [Context7](https://context7.com) (API REST anônima, sem MCP, sem API key).

O conhecimento de treino do modelo sobre frameworks desatualiza rápido — e falha mais forte em **sistemas legados**, onde o modelo escreve os idiomas de hoje contra as APIs de ontem. Este plugin resolve os dois lados: stacks novas (docs atuais da versão exata) e legados (docs da versão antiga, com anotações de delta baseadas em evidência).

## Como funciona

1. **Descobre a stack** em cascata: design aprovado → manifestos (`package.json`, `pom.xml`, `go.mod`...) → inferência do código (imports, jars, `web.xml`, script tags — versão marcada como aproximada).
2. **Confirma com você** a lista (tecnologia, versão real do projeto, fonte) antes de gastar qualquer chamada.
3. **Despacha um subagente por tecnologia**, cada um consultando a API anônima do Context7 e destilando uma skill de ~150 linhas em `.claude/skills/<tech>-<major>-docs/` do seu projeto.
4. **Versões antigas**: escada de resolução (versão exata → vizinha de mesmo major → docs latest em modo delta). Anotações `[not in 2.x]` só entram com evidência nas próprias docs baixadas (guias de migração, notas "since v") — nunca da memória de treino. Distância grande demais → variante query-only honesta (`version-gap`).
5. As skills geradas são versionadas no git do seu projeto e disparam automaticamente quando qualquer agente futuro tocar código daquela stack.

Falha do Context7 nunca bloqueia seu fluxo — degradação é sempre para variantes honestas, nunca para silêncio ou docs enganosas.

## Instalação

```
/plugin marketplace add Framework-System/frwk-plugins
/plugin install equipping-stack-docs@frwk-plugins
```

Ou direto deste repositório:

```
/plugin marketplace add Framework-System/equipping-stack-docs
/plugin install equipping-stack-docs@equipping-stack-docs
```

## Quando dispara

- **Com design/spec aprovado** (funciona ao lado do [Superpowers](https://github.com/obra/superpowers): após o brainstorming, antes do writing-plans).
- **Autônomo em codebase existente**: ao começar trabalho substancial (feature, mudança multi-arquivo) num projeto cuja stack não tem doc skills locais. Hotfix trivial não interrompe — guarda anti-ruído explícita.

## Compatibilidade

- Pensado para uso junto com o Superpowers oficial (o passo final invoca `superpowers:writing-plans` quando disponível), mas funciona sozinho — sem planning skill, ele reporta as skills geradas e devolve o controle.
- **Não instale junto com o [frameworkpowers](https://github.com/Framework-System/frameworkpowers)**: aquele fork já embute esta skill; os dois juntos duplicam o gatilho.

## Créditos

A estrutura de skills segue o padrão do [Superpowers](https://github.com/obra/superpowers) de Jesse Vincent (MIT).
