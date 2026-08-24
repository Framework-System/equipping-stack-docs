# Changelog

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
