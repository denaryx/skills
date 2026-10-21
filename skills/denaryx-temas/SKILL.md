---
name: denaryx-temas
description: Desenvolve temas em Liquid para lojas Denaryx com o CLI denaryx (tema init, dev, check, push, publish) e segue as convenções da plataforma (o head é da plataforma, preços em centavos, chaves de ajustes, verificador de temas). Use quando o usuário cria ou altera o tema de uma loja Denaryx, roda o CLI denaryx, recebe erros do verificador de temas ou pergunta sobre objetos, seções e blocos do Liquid na Denaryx.
license: MIT
metadata:
  author: denaryx
  version: '0.2.1'
---

# Temas em Liquid na Denaryx

A vitrine de uma loja Denaryx é um tema em Liquid, renderizado pela plataforma. A agência escreve o tema; o checkout, a área do cliente, a medição e o `<head>` são da plataforma.

## O CLI

Instale com `npm install --global denaryx` e entre com `denaryx login`. Os comandos principais:

| Comando                | O que faz                                                                                          |
| ---------------------- | -------------------------------------------------------------------------------------------------- |
| `denaryx tema init`    | Cria um tema a partir do Pilar, o tema de referência, sem rede.                                    |
| `denaryx tema dev`     | Servidor local em `http://127.0.0.1:9292` com os dados reais da loja, num tema de desenvolvimento. |
| `denaryx tema check`   | O verificador de temas, sem rede: o mesmo que a loja roda e que impede a publicação.               |
| `denaryx tema push`    | Envia os arquivos ao tema do projeto.                                                              |
| `denaryx tema publish` | Confere e publica o tema na loja.                                                                  |

Rode `denaryx tema check` antes de todo `push`. Um erro do verificador impede a publicação.

## A estrutura

As pastas do mercado: `layout/`, `templates/*.json`, `sections/` (com `{% schema %}`), `blocks/`, `snippets/`, `assets/`, `config/` e `locales/`. Os `templates/*.json` são o arranjo de seções que o editor do painel grava: o lojista edita textos, imagens e ordem sem código.

## As convenções que não se quebram

- **O `<head>` é da plataforma.** Ponha `{{ content_for_header }}` no layout. Ele traz a descrição, o canônico, o `robots`, o Open Graph, os dados estruturados, a medição e o CSS das seções. O tema escreve só o `<title>` (com `page_title`). Não repita descrição, canônico nem Open Graph.
- **Preços em centavos.** Os objetos trazem os preços em centavos, como o filtro `money` espera.
- **Endereços:** use `routes` e as `url` dos objetos, em vez de escrever caminhos.
- **Checkout e área do cliente** ficam numa origem própria e não rodam código do tema. O botão de finalizar leva a `/checkout`.
- **Medição:** o tema não instala GA4, pixel nem Tag Manager. Eventos próprios saem por `Denaryx.analytics.publish(nome, dados)`. Ver a skill `denaryx-medicao-e-anuncios`.
- **Cookies:** ponha os scripts do tema depois do `{{ content_for_header }}` e não grave cookie com `Domain`.
- **Ajustes da plataforma:** as chaves `logo`, `favicon`, `corFundo`, `corTexto`, `corDestaque`, `fonteTitulos` e `fonteTexto` em `config/settings_schema.json` ligam o tema ao checkout e aos e-mails.

## Objetos

Os nomes do mercado, em inglês: `product`, `variant`, `collection`, `cart`, `shop`, `request`, `routes`, `linklists`, `search` e `page`. O que só a Denaryx tem vem ao lado, em inglês: `product.brand`, `product.specifications`, `product.condition`, `collection.children`.

## Onde ler mais

- Guia da agência (estrutura, objetos, limites, convenções): https://denaryx.com/desenvolvedores/temas/guia-da-agencia
- Verificador de temas (cada regra e como corrigir): https://denaryx.com/desenvolvedores/temas/verificador
- Comandos do CLI: https://denaryx.com/desenvolvedores/cli/comandos
