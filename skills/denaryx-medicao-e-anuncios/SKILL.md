---
name: denaryx-medicao-e-anuncios
description: Explica como uma loja Denaryx mede visitas e vendas e alimenta campanhas: os eventos do dataLayer no formato do GA4, a compra enviada pelo servidor para GA4, Meta e TikTok, como não contar a mesma compra duas vezes, e o feed de produtos do Google Shopping. Use quando o usuário configura Google Tag Manager, GA4, Google Ads, Meta Ads ou TikTok Ads para uma loja Denaryx, quando um agente de anúncios precisa do catálogo ou das conversões da loja, ou quando as compras aparecem em dobro ou faltando.
license: MIT
metadata:
  author: denaryx
  version: "0.1.0"
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

## O que o servidor MCP ainda não faz

O servidor MCP da Denaryx ainda não tem relatório de vendas nem lê pedidos: um agente de IA manda o que lê ao provedor do modelo, e o servidor fica sem dados pessoais. Para conversões, use o que as próprias ferramentas recebem (GA4, Meta, TikTok). Para o catálogo, use o feed.

O guia completo do `dataLayer`, com os campos de cada evento e como testar, está em https://denaryx.com/desenvolvedores/medicao/datalayer.
