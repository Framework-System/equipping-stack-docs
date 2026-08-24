# Changelog

## 1.0.5

Seção **Cursor** do README atualizada com o resultado do teste real (Cursor Agent 2026.08.11).

Dois detalhes que só apareceram testando: o Cursor **exige a URL completa**
(`github.com/Framework-System/frwk-plugins`) e recusa o atalho `owner/repo` com `Invalid URL
format`; e o registro do marketplace vale para a **conta**, não para o projeto, já clonando os
repositórios no ato. A ativação de cada plugin é feita no app — o CLI só tem `plugin marketplace`.

Com isso o Cursor sai da lista de agentes não testados. Sem mudança de comportamento.

## 1.0.4 — README: comando inexistente

O README citava `/equipping-stack-docs:start`, comando que este plugin não tem — ele é só a skill,
sem `commands/`. A referência foi criada pelo gerador de README que eu rodei nesta mesma leva, com
um fallback que inventava um comando quando o plugin não tinha nenhum.

Achado pelo validador novo (`scripts/validar-plugin.py`, regra `readme-comandos`) na primeira
execução dele.

## 1.0.3

Correção de documentação, a partir de teste real com o **OpenCode 1.18.21** e o **pi 0.73.1**
instalados.

O README dizia para configurar o OpenCode com `"skills": ["/caminho"]` — formato copiado do README
do superpowers, que está desatualizado. O OpenCode **rejeita a config e não inicia**:
`Expected object | undefined, got [...]`. O correto é objeto com `paths`:
`"skills": { "paths": ["/caminho"] }`. Confirmado no schema oficial e testado: com a forma certa,
`opencode debug skill` lista a skill.

Somado: verificação com `opencode debug skill`, e o aviso de que o OpenCode não recarrega config a
quente.

Sem mudança de comportamento.

## 1.0.2

Correção de documentação, a partir de teste real com o **GitHub Copilot CLI 1.0.80** instalado.

A versão anterior afirmava que comandos de barra não funcionam fora do Claude Code. **Está errado
para o Copilot CLI**, que importa `commands/*.md` como skills — só que num espaço de nomes plano,
sem o prefixo do plugin.

Isso tem um custo que a documentação precisava registrar: com os 9 plugins da Framework instalados
no Copilot CLI, **58 componentes viram 46 nomes únicos — 12 ficam inacessíveis em silêncio**, porque
`status` (6 plugins), `start` (5), `ingest`, `review` e `verify` (2 cada) disputam o mesmo nome. No
Claude Code isso não acontece: lá cada comando é prefixado pelo plugin.

A tabela de compatibilidade do README agora tem três colunas (Claude Code, Copilot CLI, demais
agentes) e traz o aviso de colisão quando o plugin tem algum dos nomes disputados.

Sem mudança de comportamento.

## 1.0.1 — README no padrão multi-harness

README reescrito no padrão de mercado: sumário, uma seção por agente com o comando exato de
instalação, aviso de que repositório privado exige autenticação, e a matriz explícita do que
funciona em cada agente.

Os manifestos multi-harness (Codex, Cursor, Kimi, Gemini, pi) já existiam desde julho — este
release apenas documenta o que já era verdade. Sem mudança de comportamento.

## 1.0.0

Versão inicial publicada, já com suporte multi-harness.
