# Dicionário de dados

Descrição das tabelas e colunas da camada **processed** (`data/processed/`), base comum para todas as análises do projeto.

**Fonte:** [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) — cerca de 100 mil pedidos entre set/2016 e out/2018.

## Camadas

| Camada | Pasta | Conteúdo | Gerada por |
|---|---|---|---|
| raw | `data/raw/` | 9 CSVs originais da Olist, sem alteração | download do Kaggle |
| processed | `data/processed/` | Tabelas padronizadas, derivadas e agregadas | `notebooks/normalizacao-dados.ipynb`, `notebooks/tabelas-categorias.ipynb` e `notebooks/agregacoes-crescimento-receita.ipynb` |

## Padrões aplicados a todas as tabelas processed

- **Textos:** maiúsculas, sem acentos e cedilha, sem espaços nas pontas e sem espaços duplos (ex.: `são  paulo` → `SAO PAULO`).
- **Identificadores** (colunas `_id`): mantidos exatamente como na origem (hexadecimal minúsculo).
- **CEP:** texto com 5 dígitos, preservando o zero à esquerda (`01037`). Leia com `dtype={"<coluna_cep>": str}`.
- **Datas:** formato `AAAA-MM-DD HH:MM:SS`. Por serem CSV, precisam ser lidas com `parse_dates` no `pd.read_csv`.
- **Valores monetários:** em reais (R$).

## Visão geral das tabelas

| Tabela | Uma linha por | Chave | Linhas | Origem |
|---|---|---|---|---|
| `customers` | cliente em um pedido | `customer_id` | 99.441 | raw padronizada |
| `orders` | pedido | `order_id` | 99.441 | raw padronizada |
| `order_items` | item de pedido | `order_id` + `order_item_id` | 112.650 | raw padronizada |
| `payments` | pagamento de um pedido | `order_id` + `payment_sequential` | 103.886 | raw padronizada |
| `reviews` | avaliação de um pedido | `review_id` + `order_id` | 99.224 | raw padronizada |
| `products` | produto | `product_id` | 32.951 | raw padronizada |
| `category_translation` | categoria | `product_category_name` | 71 | raw padronizada |
| `sellers` | vendedor | `seller_id` | 3.095 | raw padronizada |
| `geolocation` | coordenada de um CEP | — | 720.494 | raw padronizada, sem duplicatas |
| `orders_enriched` | pedido | `order_id` | 99.441 | derivada |
| `order_items_category` | item de pedido | `order_id` + `order_item_id` | 112.650 | derivada |
| `category_reviews` | categoria | `product_category_name` | 74 | derivada |
| `agg_mensal` | mês | `purchase_year_month` | 20 | agregada |
| `agg_categoria` | categoria | `product_category_name` | 74 | agregada |
| `agg_uf` | UF do cliente | `customer_state` | 27 | agregada |
| `agg_produto` | produto | `product_id` | 32.081 | agregada |
| `agg_seller` | seller | `seller_id` | 2.945 | agregada |

## Relacionamentos

```
customers ──customer_id── orders ──order_id── order_items ──product_id── products ──product_category_name── category_translation
                            │                      │
                            ├──order_id── payments └──seller_id── sellers
                            └──order_id── reviews

customers.customer_zip_code_prefix / sellers.seller_zip_code_prefix ── geolocation.geolocation_zip_code_prefix
```

> **Atenção nos joins:** `payments`, `reviews` e `geolocation` têm mais de uma linha por chave de ligação. Agregue antes de juntar (ex.: soma de pagamentos por pedido, média de review por pedido, média de coordenadas por CEP) para não multiplicar linhas.

---

## Tabelas padronizadas

### `customers`

Cadastro do cliente associado a cada pedido.

| Coluna | Tipo | Descrição | Nulos |
|---|---|---|---|
| `customer_id` | texto | Identificador do cliente **no pedido**. Cada pedido gera um `customer_id` novo | 0 |
| `customer_unique_id` | texto | Identificador da **pessoa**. Use para contar clientes e medir recompra (96.096 únicos) | 0 |
| `customer_zip_code_prefix` | texto | 5 primeiros dígitos do CEP | 0 |
| `customer_city` | texto | Cidade do cliente | 0 |
| `customer_state` | texto | UF do cliente (27 UFs) | 0 |

### `orders`

Pedido e as datas de cada etapa do fluxo.

| Coluna | Tipo | Descrição | Nulos |
|---|---|---|---|
| `order_id` | texto | Identificador do pedido | 0 |
| `customer_id` | texto | Cliente do pedido (liga com `customers`) | 0 |
| `order_status` | texto | Status: `DELIVERED`, `SHIPPED`, `CANCELED`, `UNAVAILABLE`, `INVOICED`, `PROCESSING`, `CREATED`, `APPROVED` | 0 |
| `order_purchase_timestamp` | data/hora | Data e hora da compra | 0 |
| `order_approved_at` | data/hora | Aprovação do pagamento | 160 (0,2%) |
| `order_delivered_carrier_date` | data/hora | Entrega do pedido à transportadora | 1.783 (1,8%) |
| `order_delivered_customer_date` | data/hora | Entrega ao cliente | 2.965 (3,0%) |
| `order_estimated_delivery_date` | data | Data de entrega prometida ao cliente (sem horário) | 0 |

Os nulos de data são, em sua maioria, pedidos que não chegaram àquela etapa (ver [relatório de qualidade](relatorio-de-qualidade.md)).

### `order_items`

Itens de cada pedido. Um pedido pode ter vários itens, inclusive do mesmo produto.

| Coluna | Tipo | Descrição | Nulos |
|---|---|---|---|
| `order_id` | texto | Pedido do item | 0 |
| `order_item_id` | inteiro | Número sequencial do item dentro do pedido (1 a 21) | 0 |
| `product_id` | texto | Produto (liga com `products`) | 0 |
| `seller_id` | texto | Vendedor (liga com `sellers`) | 0 |
| `shipping_limit_date` | data/hora | Prazo para o seller entregar o item à transportadora | 0 |
| `price` | decimal | Preço do item (R$ 0,85 a R$ 6.735,00) | 0 |
| `freight_value` | decimal | Frete do item (R$ 0,00 a R$ 409,68) | 0 |

### `payments`

Pagamentos de cada pedido. Um pedido pode ter vários pagamentos (ex.: cartão + voucher).

| Coluna | Tipo | Descrição | Nulos |
|---|---|---|---|
| `order_id` | texto | Pedido pago | 0 |
| `payment_sequential` | inteiro | Sequência do pagamento dentro do pedido | 0 |
| `payment_type` | texto | `CREDIT_CARD`, `BOLETO`, `VOUCHER`, `DEBIT_CARD`, `NOT_DEFINED` | 0 |
| `payment_installments` | inteiro | Número de parcelas (0 a 24) | 0 |
| `payment_value` | decimal | Valor pago nesse pagamento | 0 |

### `reviews`

Avaliações dos clientes.

| Coluna | Tipo | Descrição | Nulos |
|---|---|---|---|
| `review_id` | texto | Identificador da avaliação. Pode se repetir em pedidos diferentes | 0 |
| `order_id` | texto | Pedido avaliado. Pode ter mais de uma avaliação | 0 |
| `review_score` | inteiro | Nota de 1 a 5 | 0 |
| `review_comment_title` | texto | Título do comentário (opcional) | 87.666 (88,4%) |
| `review_comment_message` | texto | Comentário (opcional) | 58.293 (58,7%) |
| `review_creation_date` | data | Envio do formulário de avaliação ao cliente | 0 |
| `review_answer_timestamp` | data/hora | Resposta do cliente | 0 |

### `products`

Cadastro de produtos. Os nomes com `lenght` estão grafados assim na origem (o correto seria `length`).

| Coluna | Tipo | Descrição | Nulos |
|---|---|---|---|
| `product_id` | texto | Identificador do produto | 0 |
| `product_category_name` | texto | Categoria em português (73 categorias) | 610 (1,9%) |
| `product_name_lenght` | decimal | Quantidade de caracteres do nome | 610 (1,9%) |
| `product_description_lenght` | decimal | Quantidade de caracteres da descrição | 610 (1,9%) |
| `product_photos_qty` | decimal | Quantidade de fotos no anúncio | 610 (1,9%) |
| `product_weight_g` | decimal | Peso em gramas | 2 |
| `product_length_cm` | decimal | Comprimento em cm | 2 |
| `product_height_cm` | decimal | Altura em cm | 2 |
| `product_width_cm` | decimal | Largura em cm | 2 |

### `category_translation`

| Coluna | Tipo | Descrição | Nulos |
|---|---|---|---|
| `product_category_name` | texto | Categoria em português | 0 |
| `product_category_name_english` | texto | Categoria em inglês | 0 |

### `sellers`

| Coluna | Tipo | Descrição | Nulos |
|---|---|---|---|
| `seller_id` | texto | Identificador do vendedor | 0 |
| `seller_zip_code_prefix` | texto | 5 primeiros dígitos do CEP | 0 |
| `seller_city` | texto | Cidade do vendedor | 0 |
| `seller_state` | texto | UF do vendedor (23 UFs) | 0 |

### `geolocation`

Coordenadas por prefixo de CEP. Um mesmo CEP tem várias coordenadas (19.015 CEPs distintos).

| Coluna | Tipo | Descrição | Nulos |
|---|---|---|---|
| `geolocation_zip_code_prefix` | texto | 5 primeiros dígitos do CEP | 0 |
| `geolocation_lat` | decimal | Latitude | 0 |
| `geolocation_lng` | decimal | Longitude | 0 |
| `geolocation_city` | texto | Cidade | 0 |
| `geolocation_state` | texto | UF | 0 |

---

## Tabelas derivadas

### `orders_enriched`

Uma linha por pedido, com indicadores prontos para análise. Contém **todos** os pedidos, de todos os status — filtre `order_status == "DELIVERED"` para análises de receita.

Inclui todas as colunas de `orders`, mais:

| Coluna | Tipo | Descrição | Regra de cálculo | Nulos |
|---|---|---|---|---|
| `order_price` | decimal | Valor dos produtos do pedido | Soma de `price` dos itens | 775 (pedidos sem itens) |
| `order_freight` | decimal | Valor do frete do pedido | Soma de `freight_value` dos itens | 775 |
| `order_total` | decimal | Valor total do pedido | `order_price` + `order_freight` | 775 |
| `review_score` | decimal | Avaliação do pedido | Média de `review_score` das avaliações do pedido | 768 (pedidos sem avaliação) |
| `customer_state` | texto | UF do cliente | De `customers` | 0 |
| `customer_city` | texto | Cidade do cliente | De `customers` | 0 |
| `purchase_year_month` | texto | Mês da compra (`AAAA-MM`) | De `order_purchase_timestamp` | 0 |
| `dt_diff` | inteiro | Dias entre a entrega e a data estimada. Positivo = atraso; negativo = antecipação | Data de `order_delivered_customer_date` − `order_estimated_delivery_date`, sem horário | 2.965 (pedidos sem entrega) |
| `is_late` | 0/1 | Pedido entregue com atraso | 1 se `dt_diff` > 0; 0 caso contrário | 2.965 (pedidos sem entrega) |

> `is_late` fica **vazio** (e não 0) quando o pedido não tem data de entrega: não é possível dizer se um pedido cancelado ou em trânsito atrasou.

### `order_items_category`

Uma linha por item de pedido, com a categoria do produto e informações do pedido. Base para análises de receita, volume e participação por categoria.

| Coluna | Tipo | Descrição | Origem |
|---|---|---|---|
| `order_id`, `order_item_id`, `product_id`, `seller_id`, `shipping_limit_date`, `price`, `freight_value` | — | Colunas do item | `order_items` |
| `product_category_name` | texto | Categoria em português. `SEM CATEGORIA` quando o produto não tem categoria | `products` |
| `product_category_name_english` | texto | Categoria em inglês. Usa o nome em português quando não há tradução | `category_translation` |
| `order_status` | texto | Status do pedido | `orders_enriched` |
| `order_purchase_timestamp` | data/hora | Data da compra | `orders_enriched` |
| `customer_state` | texto | UF do cliente | `orders_enriched` |
| `review_score` | decimal | Avaliação média do pedido | `orders_enriched` |
| `is_late` | 0/1 | Pedido atrasado | `orders_enriched` |

> `review_score` e `is_late` são do **pedido**: um pedido com 3 itens repete o mesmo valor nas 3 linhas. Para médias por categoria, deduplique por `order_id` + `product_category_name` (como em `category_reviews`).

### `category_reviews`

Uma linha por categoria, com indicadores agregados.

| Coluna | Tipo | Descrição | Regra de cálculo |
|---|---|---|---|
| `product_category_name` | texto | Categoria em português | — |
| `product_category_name_english` | texto | Categoria em inglês | — |
| `pedidos` | inteiro | Pedidos com ao menos um item da categoria | Contagem distinta de `order_id` |
| `review_medio` | decimal | Avaliação média | Média de `review_score`, um registro por pedido e categoria |
| `pct_atraso` | decimal | % de pedidos atrasados (0 a 100) | Média de `is_late` × 100, um registro por pedido e categoria |
| `receita` | decimal | Soma do preço dos itens da categoria (sem frete) | Soma de `price` |

> `receita` considera pedidos de todos os status e só o preço (sem frete). Para a receita realizada no padrão do projeto, use `agg_categoria`.


---

## Tabelas agregadas (crescimento e receita)

Geradas por `notebooks/agregacoes-crescimento-receita.ipynb`. Todas seguem as mesmas regras:

- **Somente pedidos `DELIVERED`**, com compra entre **jan/2017 e ago/2018**.
- **Receita = preço + frete** (valor total pago pelo cliente).
- **Ticket médio = receita ÷ pedidos.**
- **`participacao_receita_pct`** = receita da linha ÷ receita total do período × 100 (0 a 100).

A receita total é a mesma em todas as tabelas: **R$ 15.373.120,01**.

### `agg_mensal`

Uma linha por mês (20 meses).

| Coluna | Tipo | Descrição |
|---|---|---|
| `purchase_year_month` | texto | Mês da compra (`AAAA-MM`) |
| `pedidos` | inteiro | Quantidade de pedidos |
| `receita` | decimal | Receita do mês |
| `ticket_medio` | decimal | Receita ÷ pedidos |
| `crescimento_receita_pct` | decimal | Variação da receita em relação ao mês anterior (%). Vazio no primeiro mês |

### `agg_categoria`

Uma linha por categoria (74), ordenada pela receita.

| Coluna | Tipo | Descrição |
|---|---|---|
| `product_category_name` | texto | Categoria em português (`SEM CATEGORIA` para produtos sem cadastro) |
| `product_category_name_english` | texto | Categoria em inglês |
| `pedidos` | inteiro | Pedidos com ao menos um item da categoria |
| `itens` | inteiro | Itens vendidos |
| `receita` | decimal | Soma de preço + frete dos itens da categoria |
| `participacao_receita_pct` | decimal | % da receita total |

> A soma de `pedidos` entre categorias é maior que o total de pedidos, porque um pedido com itens de duas categorias conta nas duas.

### `agg_uf`

Uma linha por UF do cliente (27), ordenada pela receita.

| Coluna | Tipo | Descrição |
|---|---|---|
| `customer_state` | texto | UF do cliente |
| `pedidos` | inteiro | Quantidade de pedidos |
| `receita` | decimal | Receita da UF |
| `ticket_medio` | decimal | Receita ÷ pedidos |
| `participacao_receita_pct` | decimal | % da receita total |

### `agg_produto`

Uma linha por produto vendido no período (32.081). Para o top 10, ordene por `receita` ou `itens`.

| Coluna | Tipo | Descrição |
|---|---|---|
| `product_id` | texto | Identificador do produto |
| `product_category_name` | texto | Categoria em português |
| `pedidos` | inteiro | Pedidos que incluíram o produto |
| `itens` | inteiro | Unidades vendidas |
| `receita` | decimal | Soma de preço + frete |
| `preco_medio` | decimal | Preço médio de venda (sem frete) |
| `participacao_receita_pct` | decimal | % da receita total |

### `agg_seller`

Uma linha por seller com vendas no período (2.945). Para o top 10, ordene por `receita`.

| Coluna | Tipo | Descrição |
|---|---|---|
| `seller_id` | texto | Identificador do vendedor |
| `pedidos` | inteiro | Pedidos com itens do seller |
| `itens` | inteiro | Itens vendidos |
| `receita` | decimal | Soma de preço + frete dos itens do seller |
| `review_medio` | decimal | Avaliação média, um registro por pedido e seller |
| `pct_atraso` | decimal | % de pedidos atrasados (0 a 100), um registro por pedido e seller |
| `seller_city` | texto | Cidade do seller |
| `seller_state` | texto | UF do seller |
| `participacao_receita_pct` | decimal | % da receita total |
