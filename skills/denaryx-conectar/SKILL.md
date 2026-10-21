---
name: denaryx-conectar
description: Liga um agente de IA a uma loja da Denaryx pelo servidor MCP (https://api.denaryx.com/mcp) com o token de uma integração, e escolhe os escopos certos. Use quando o usuário quer conectar Claude Code, Cursor, um agente próprio ou um SDK a uma loja Denaryx, quando aparece um token dnxa_, ou quando uma ferramenta da Denaryx responde 401 ou 403.
license: MIT
metadata:
  author: denaryx
  version: '0.2.2'
---

# Conectar um agente à loja Denaryx

A Denaryx é uma plataforma de e-commerce brasileira. Cada loja tem um servidor MCP em `https://api.denaryx.com/mcp`. O agente entra com o token de uma integração da loja e só vê essa loja.

## 1. O lojista cria a integração

O token vem do painel da loja, e só o dono ou o gerente o criam. Peça ao usuário que:

1. Abra **Configurações › Integrações** no painel da loja e crie uma integração com um nome que diga quem vai usá-la (por exemplo, "Agente de SEO").
2. Marque só os escopos de que o agente precisa (tabela abaixo).
3. Copie o token, que começa com `dnxa_` e aparece uma vez só.

| O agente vai                                                       | Escopo          |
| ------------------------------------------------------------------ | --------------- |
| Ler o catálogo e as páginas, e auditar o SEO                       | `read_content`  |
| Também mudar títulos, descrições, SEO, endereços e textos de fotos | `write_content` |
| Ler as vendas somadas por dia, produto ou categoria, para anúncios | `read_reports`  |

O `write_content` muda **só os textos**. Ele não mexe em preço, estoque, variantes, fotos, nem exclui nada. Não peça `write_products` para um trabalho de conteúdo.

## 2. Ligue o cliente

O token é segredo. Não o escreva em arquivo versionado, em log nem em mensagem. Prefira uma variável de ambiente.

**Claude Code:**

```bash
claude mcp add --transport http denaryx https://api.denaryx.com/mcp \
  --header "Authorization: Bearer $DENARYX_TOKEN"
```

**Cliente com arquivo de configuração** (o `mcp.json` do Cursor, por exemplo):

```json
{
  "mcpServers": {
    "denaryx": {
      "url": "https://api.denaryx.com/mcp",
      "headers": { "Authorization": "Bearer dnxa_..." }
    }
  }
}
```

Se o cliente aceita variáveis de ambiente no arquivo, use uma no lugar do token. Senão, mantenha o arquivo fora do controle de versão.

**Agente próprio:** use o cliente do SDK do MCP com o transporte Streamable HTTP e o cabeçalho `Authorization: Bearer <token>` em toda requisição. O servidor não tem sessão: cada chamada é independente.

## 3. Confira com `get_shop`

A primeira chamada é sempre `get_shop`. Ela devolve a loja (nome, endereço da vitrine, moeda), a integração e os escopos do token. A lista de ferramentas mostra só as que esses escopos permitem. O cliente recebe essa lista quando se conecta: se o lojista marcar um escopo novo depois, o `get_shop` já mostra o escopo, mas a ferramenta só aparece depois de reconectar o servidor (no Claude Code, `/mcp`) ou de abrir outra conversa.

## Quando dá erro

| Resposta                                          | O que fazer                                                                                        |
| ------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| 401 sem token, `invalid_token`, `token_revoked`   | O token falta, está errado, foi revogado ou venceu. Peça um novo ao lojista.                       |
| 403 `insufficient_scope` (cabeçalho com `scope=`) | A integração não tem o escopo. Peça ao lojista para marcá-lo; não tente outra ferramenta.          |
| 429 com `Retry-After`                             | Limite de chamadas. Espere o tempo do cabeçalho e repita.                                          |
| 429 `daily_limit_exceeded`                        | O teto diário de mudanças acabou. Pare de escrever e retome no dia seguinte (horário de Brasília). |
| 503 com `Retry-After`                             | A API está ocupada. Espere e repita.                                                               |

## O que o servidor não faz

Nenhuma ferramenta lê pedidos um a um ou clientes, nem muda preço, estoque ou fotos, nem exclui nada, mesmo que o token tenha outros escopos: as vendas chegam só somadas (`read_reports`). Para o resto existe a API Admin REST (`https://api.denaryx.com/admin/2026-10`), que é para integrações de servidor como um ERP, e não para um agente de conteúdo ou de anúncios.

O guia completo do servidor está em https://denaryx.com/desenvolvedores/mcp.
