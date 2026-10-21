---
name: denaryx-medicao-e-anuncios
description: Explica como uma loja Denaryx mede visitas e vendas e alimenta campanhas, com os eventos do dataLayer no formato do GA4, a compra enviada pelo servidor para GA4, Meta e TikTok, como não contar a mesma compra duas vezes, o feed de produtos do Google Shopping e o relatório de vendas do servidor MCP (get_sales_report) para comparar a receita da loja com a das campanhas. Use quando o usuário configura Google Tag Manager, GA4, Google Ads, Meta Ads ou TikTok Ads para uma loja Denaryx, quando um agente de anúncios precisa do catálogo, das conversões ou das vendas por dia, produto ou categoria da loja, ou quando as compras aparecem em dobro ou faltando.
license: MIT
metadata:
  author: denaryx
  version: '0.3.0'
---

# Medição e anúncios numa loja Denaryx

A Denaryx mede pela própria plataforma: o tema não precisa de script de rastreamento. Esta skill diz o que já existe e como usar sem medir em dobro.

## Os quatro caminhos de medição

| Caminho                                         | Onde roda                       | O que recebe                        |
| ----------------------------------------------- | ------------------------------- | ----------------------------------- |
| Ferramentas (GA4, Tag Manager, Meta, TikTok)    | Na página da loja               | Os eventos da vitrine               |
| Compra pelo servidor (GA4, Meta, TikTok)        | No servidor da plataforma       | A compra                            |
| Pixels personalizados (código colado no painel) | Isolados, na loja e no checkout | Os eventos da vitrine e do checkout |
| Tag Manager no checkout (só o contêiner)        | Isolado, só no checkout         | Os eventos do checkout              |

O lojista liga tudo em **Painel › Medição**. O checkout fica em outro endereço (`checkout.denaryx.com`), sem `dataLayer` próprio e sem script de terceiros: a compra chega às ferramentas pelo servidor ou pelo contêiner isolado do checkout.

## Os eventos

Toda página da vitrine tem um `window.dataLayer` com os eventos de comércio eletrônico no formato do GA4, desde a abertura da página. Nada sai para fora antes do aceite do comprador no aviso de cookies.

- **Vitrine:** `page_view`, `view_item`, `view_item_list`, `select_item`, `search`, `add_to_cart`, `remove_from_cart`, `view_cart` e `begin_checkout`.
- **Checkout** (no contêiner isolado): `begin_checkout`, `add_shipping_info`, `add_payment_info` e `purchase`.
- **Consentimento:** o evento `denaryx_consent`, com `consent_analytics` e `consent_marketing` (`granted` ou `denied`).

Cada evento leva um `event_id`, e antes de cada um entra um `{ ecommerce: null }`.

## Qual compra vale em cada ferramenta

A mesma compra pode chegar por mais de um caminho. Para não contar duas vezes:

| Ferramenta                                   | A compra que vale                              |
| -------------------------------------------- | ---------------------------------------------- |
| Google Analytics 4                           | A do servidor (Measurement Protocol)           |
| Google Ads                                   | Importar a conversão `purchase` do GA4         |
| Meta e TikTok                                | A do servidor (API de Conversões e Events API) |
| Afiliados, mídia programática e outras redes | O `purchase` do Tag Manager no checkout        |

- O `transaction_id` é o número do pedido. O `event_id` da compra é o id do pedido, o mesmo que vai à API de Conversões da Meta e à Events API do TikTok: a ferramenta descarta a repetida.
- Por padrão, a compra pelo servidor sai com o pagamento aprovado. O `purchase` do contêiner sai com o pedido feito, pago ou não.
- Use um caminho só por ferramenta também na loja: o GA4 ligado em Ferramentas e uma tag do GA4 no Tag Manager contam cada evento duas vezes.

## O catálogo para anúncios

Cada loja publica o feed do Google Shopping em `https://<endereço da loja>/feeds/google.xml`: RSS 2.0 com os campos `g:`, um item por variante (`item_group_id` é o produto), com preço e promoção, disponibilidade, marca, GTIN, condição e o caminho da categoria. O título e a descrição do feed vêm do produto: melhorar esses textos (skill `denaryx-seo-do-catalogo`) também melhora os anúncios.

## As vendas da loja pelo servidor MCP (`get_sales_report`)

Com o escopo `read_reports`, a ferramenta `get_sales_report` soma as vendas de um período, e por dia, por produto ou por coleção (categoria). É o número da loja para comparar com o que a campanha gastou e com o que a ferramenta de anúncios atribui. Não traz nenhum dado de quem comprou.

```json
{ "from": "2026-09-01", "to": "2026-09-30", "group_by": "day" }
```

- **O que conta:** o pedido pago e não cancelado, pela data do pagamento, nos dias de Brasília (`America/Sao_Paulo`). Se a conta de anúncios usa outro fuso, os dias não batem exatamente nas bordas: compare semanas ou meses.
- **A receita** (`revenue`) é o valor dos produtos com os descontos e sem o frete, o mesmo valor da compra que a loja manda à Meta, ao TikTok e ao GA4, no mesmo momento (o pagamento aprovado). `revenue` dividido pelo que se gastou em todos os canais dá o retorno geral da loja.
- **Atribuída e total não são a mesma coisa:** o relatório conta todas as vendas da loja, de qualquer canal; a ferramenta de anúncios conta só as que atribui à campanha. O total da loja costuma ser maior. O pedido cancelado depois sai do relatório, mas a conversão continua na ferramenta.
- **Os campos:** `orders`, `items`, `revenue`, `average_order_value` (o ticket médio), `discount` (cupom e Pix) e `shipping` (o frete), todos em centavos. `totals` é o período inteiro. Sem pedidos, o `average_order_value` vem `null`: não há ticket médio, então não o some como 0.
- **Por dia** (`group_by: "day"`): só os dias com venda. O dia que não aparece teve zero, e os dias de antes da loja existir nem contam. Até 92 dias por chamada.
- **O período vai até hoje:** o `to` no futuro é recusado. Para "este mês", use do dia 1 até hoje.
- **Por produto ou por coleção** (`"product"`, `"collection"`): as que mais venderam em receita, 20 por padrão (até 50 com `limit`; `truncated: true` diz que há mais), num período de até 366 dias. Aqui a receita é a dos itens, antes do cupom e do desconto no Pix. A coleção soma as coleções de baixo dela, e o produto em duas coleções conta nas duas: as linhas não somam o total. Use para saber quais produtos e categorias anunciar, e os títulos e endereços deles com `search_products` e `list_collections` (essas pedem `read_content` ou `read_products`).
- **Sem dado pessoal:** não peça `read_orders` ao lojista para ver os pedidos um a um. O servidor MCP não lê pedidos nem clientes, e as conversões por pessoa ficam nas ferramentas de anúncios, que já as recebem.

O guia do relatório, com cada campo, está em https://denaryx.com/desenvolvedores/recursos/relatorios. O guia completo do `dataLayer`, com os campos de cada evento e como testar, está em https://denaryx.com/desenvolvedores/medicao/datalayer.
