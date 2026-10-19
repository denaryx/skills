# Skills da Denaryx para agentes de IA

Skills para agentes de IA que operam lojas na [Denaryx](https://denaryx.com), plataforma de e-commerce brasileira. Elas ensinam o agente a ligar-se à loja pelo servidor MCP da Denaryx e a fazer o trabalho do jeito certo: auditar e melhorar o SEO do catálogo, escrever categorias e páginas institucionais, entender a medição e o feed para anúncios e desenvolver temas em Liquid.

As skills seguem o formato aberto [Agent Skills](https://agentskills.io): uma pasta por skill, com um `SKILL.md`. Funcionam no Claude Code, no Cursor, no GitHub Copilot, no Codex e em outros agentes que leem esse formato.

## As skills

| Skill                          | Para quê                                                                                              |
| ------------------------------ | ----------------------------------------------------------------------------------------------------- |
| `denaryx-conectar`             | Ligar o agente ao servidor MCP da loja com o token de uma integração e escolher os escopos.           |
| `denaryx-seo-do-catalogo`      | Auditar e melhorar títulos, descrições, SEO e o texto das fotos, com simulação, lote e desfazer.      |
| `denaryx-categorias-e-paginas` | Escrever o texto das categorias e das marcas e criar ou revisar as páginas institucionais.            |
| `denaryx-medicao-e-anuncios`   | Os eventos do dataLayer, a compra pelo servidor, o feed e o relatório de vendas (`get_sales_report`). |
| `denaryx-temas`                | Desenvolver temas em Liquid com o CLI `denaryx` e as convenções da plataforma.                        |

## Instalar

Com o GitHub CLI (versão 2.90 ou mais nova):

```bash
gh skill install denaryx/skills
```

Com o `npx`:

```bash
npx skills add denaryx/skills
```

Como plugin do Claude Code:

```text
/plugin marketplace add denaryx/skills
/plugin install denaryx@denaryx
```

Sem ferramenta: copie as pastas de `skills/` para a pasta de skills do seu agente (no Claude Code, `.claude/skills/` do projeto ou `~/.claude/skills/`).

## Antes de usar

O agente precisa do token de uma integração da loja, criado pelo lojista no painel, em **Configurações › Integrações**. A skill `denaryx-conectar` mostra o passo a passo. O servidor MCP fica em `https://api.denaryx.com/mcp`, e o guia completo dele está em [denaryx.com/desenvolvedores/mcp](https://denaryx.com/desenvolvedores/mcp).

## Versões

Este repositório é publicado a partir do repositório da plataforma, onde as skills são testadas contra o servidor MCP: os nomes das ferramentas e os limites citados são conferidos a cada versão. Não abra pull request aqui; para sugerir uma mudança, abra uma issue.

## Licença

[MIT](LICENSE).
