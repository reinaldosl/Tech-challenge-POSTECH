# Relatório de qualidade dos dados

Avaliação da qualidade dos dados brutos da Olist (`data/raw/`) e registro de tudo o que foi alterado, removido ou mantido na camada processed (`data/processed/`).

- **Diagnóstico:** `notebooks/inventario-de-dados.ipynb`
- **Tratamento:** `notebooks/normalizacao-dados.ipynb` e `notebooks/tabelas-categorias.ipynb`
- **Descrição das colunas:** [dicionário de dados](dicionario-de-dados.md)

## Resumo

| Tabela | Linhas raw | Linhas processed | Removidas | Principais pontos |
|---|---|---|---|---|
| `customers` | 99.441 | 99.441 | 0 | 278 clientes com CEP sem coordenadas |
| `orders` | 99.441 | 99.441 | 0 | Datas nulas em pedidos não concluídos; 8 entregues sem data de entrega |
| `order_items` | 112.650 | 112.650 | 0 | 383 itens com frete zero; 4 prazos de envio fora do período |
| `payments` | 103.886 | 103.886 | 0 | 9 pagamentos de valor zero; 3 `NOT_DEFINED`; 1 pedido sem pagamento |
| `reviews` | 99.224 | 99.224 | 0 | Avaliações repetidas entre pedidos e pedidos com mais de uma avaliação |
| `products` | 32.951 | 32.951 | 0 | 610 produtos sem categoria; 2 categorias sem tradução |
| `category_translation` | 71 | 71 | 0 | — |
| `sellers` | 3.095 | 3.095 | 0 | 7 sellers com CEP sem coordenadas |
| `geolocation` | 1.000.163 | 720.494 | **279.669** | Duplicatas; CEPs com várias coordenadas; 33 pontos fora do Brasil |

**A única remoção de linhas é a de duplicatas da geolocalização.** Nenhum pedido, item, pagamento ou avaliação foi removido: as inconsistências foram mantidas e documentadas abaixo, e o tratamento de cada uma fica a cargo da análise que a utiliza.

---

## 1. Transformações aplicadas

| Transformação | Tabelas | Motivo |
|---|---|---|
| Datas convertidas de texto para data/hora | `orders`, `order_items`, `reviews` | Permitir cálculo de prazos e agrupamento por mês |
| Textos em maiúsculas, sem acentos, sem espaços nas pontas e sem espaços duplos | Todas (exceto colunas `_id`) | Grafias diferentes do mesmo valor eram tratadas como valores distintos |
| CEP lido como texto | `customers`, `sellers`, `geolocation` | Lido como número, perde o zero à esquerda e deixa de casar entre tabelas |
| Leitura com `utf-8-sig` | `category_translation` | Caractere invisível (BOM) no início do arquivo corrompia o nome da primeira coluna |
| Remoção de linhas duplicadas | `geolocation` | Linhas idênticas não trazem informação nova |
| Produtos sem categoria → `SEM CATEGORIA` | `order_items_category`, `category_reviews` | Evitar que sumam nos agrupamentos por categoria |
| Categorias sem tradução → nome em português | `order_items_category`, `category_reviews` | `PC_GAMER` e `PORTATEIS_COZINHA_E_PREPARADORES_DE_ALIMENTOS` não existem na tabela de tradução |

**Efeito da padronização de textos:** a geolocalização tinha 8.011 grafias de cidade; após a padronização, 5.965. Ex.: `sao paulo` (135.800 linhas) e `são paulo` (24.918) passaram a ser `SAO PAULO` (160.719).

---

## 2. Linhas removidas

### `geolocation` — 279.669 linhas

| Etapa | Linhas |
|---|---|
| Linhas no raw | 1.000.163 |
| Duplicatas exatas no raw | 261.831 |
| Duplicatas adicionais após a padronização de textos (diferiam só por acento ou maiúscula) | 17.838 |
| **Linhas no processed** | **720.494** |

Mesmo após a remoção, cada CEP continua com várias coordenadas (19.015 CEPs distintos). Para usar em mapas, consolide um ponto por CEP (ex.: média de latitude e longitude).

---

## 3. Valores nulos

### Nulos esperados (regra de negócio, não são erro)

| Tabela | Coluna | Nulos | Explicação |
|---|---|---|---|
| `reviews` | `review_comment_title` | 87.666 (88,4%) | Título do comentário é opcional |
| `reviews` | `review_comment_message` | 58.293 (58,7%) | Comentário é opcional |
| `orders` | `order_delivered_customer_date` | 2.965 (3,0%) | Pedidos em trânsito, cancelados ou indisponíveis ainda não foram (ou nunca serão) entregues |
| `orders` | `order_delivered_carrier_date` | 1.783 (1,8%) | Pedidos que não chegaram à transportadora |
| `orders` | `order_approved_at` | 160 (0,2%) | Em sua maioria, pedidos que não chegaram à aprovação |

### Nulos que indicam inconsistência

| Tabela | Caso | Quantidade | Tratamento |
|---|---|---|---|
| `orders` | Status `DELIVERED` sem data de entrega ao cliente | 8 | Mantidos. `dt_diff` e `is_late` ficam vazios |
| `orders` | Status `DELIVERED` sem data de envio à transportadora | 2 | Mantidos |
| `orders` | Status `DELIVERED` sem data de aprovação | 14 | Mantidos |
| `products` | Sem categoria, nome, descrição e fotos (cadastro incompleto) | 610 | Mantidos. Agrupados como `SEM CATEGORIA` |
| `products` | Sem peso e medidas | 2 | Mantidos |

### Nulos gerados nas tabelas derivadas

| Tabela | Coluna | Nulos | Motivo |
|---|---|---|---|
| `orders_enriched` | `order_price`, `order_freight`, `order_total` | 775 | Pedidos sem itens (ver seção 5) |
| `orders_enriched` | `review_score` | 768 | Pedidos sem avaliação |
| `orders_enriched` | `dt_diff`, `is_late` | 2.965 | Pedidos sem data de entrega: não é possível dizer se atrasaram |

---

## 4. Duplicidade

| Tabela | Verificação | Resultado | Impacto e tratamento |
|---|---|---|---|
| Todas, exceto `geolocation` | Linhas inteiramente duplicadas | 0 | — |
| Todas | Chave primária repetida (`order_id`, `customer_id`, `product_id`, `seller_id`) | 0 | — |
| `reviews` | Mesmo `review_id` em pedidos diferentes | 789 avaliações em 1.603 linhas | Cliente avaliou de uma vez vários pedidos feitos juntos. Mantido |
| `reviews` | Pedidos com mais de uma avaliação | 547 pedidos | Duplicaria linhas no join. Em `orders_enriched`, `review_score` é a **média** por pedido |
| `payments` | Pedidos com mais de um pagamento | 2.961 pedidos (4.446 linhas extras) | Duplicaria linhas no join. Agregar por pedido antes de juntar |
| `geolocation` | Linhas inteiramente duplicadas | 279.669 (após padronização) | Removidas (seção 2) |

**Verificação nos joins:** `orders_enriched` manteve 99.441 linhas (= `orders`) e `order_items_category` manteve 112.650 linhas (= `order_items`), confirmando que nenhum join multiplicou registros.

---

## 5. Integridade entre tabelas

| Relação | Sem correspondência | Detalhe |
|---|---|---|
| `orders` → `order_items` | 775 pedidos sem itens | 603 `UNAVAILABLE`, 164 `CANCELED`, 5 `CREATED`, 2 `INVOICED`, 1 `SHIPPED`. Nenhum entregue |
| `orders` → `payments` | 1 pedido sem pagamento | `bfbd0f9bdef84302105ad712db648a6c`, status `DELIVERED` |
| `orders` → `reviews` | 768 pedidos sem avaliação | 646 deles entregues |
| `products` → `category_translation` | 2 categorias (13 produtos) | `PORTATEIS_COZINHA_E_PREPARADORES_DE_ALIMENTOS` (10) e `PC_GAMER` (3) |
| `customers` → `geolocation` | 157 CEPs (278 clientes) | Sem coordenadas, mas com cidade e UF |
| `sellers` → `geolocation` | 7 CEPs (7 sellers) | Sem coordenadas, mas com cidade e UF |
| `order_items`, `payments`, `reviews` → `orders` | 0 | Todos os registros têm pedido correspondente |
| `order_items` → `products` / `sellers` | 0 | — |

---

## 6. Consistência de valores e datas

| Tabela | Caso | Quantidade | Tratamento |
|---|---|---|---|
| `order_items` | Frete igual a zero | 383 itens | Mantidos. Podem ser frete grátis |
| `order_items` | `shipping_limit_date` após 2019 (até abr/2020) | 4 itens | Mantidos. Erro de digitação na origem; não usar essa coluna sem filtro |
| `payments` | `payment_value` igual a zero | 9 (6 `VOUCHER`, 3 `NOT_DEFINED`) | Mantidos |
| `payments` | `payment_installments` igual a zero | 2 | Mantidos |
| `payments` | `payment_type` = `NOT_DEFINED` | 3 | Mantidos |
| `products` | Peso igual a zero | 4 | Mantidos |
| `orders` | Envio à transportadora **antes** da compra | 166 | Mantidos. Afeta cálculos de lead time de postagem |
| `orders` | Entrega ao cliente **antes** do envio à transportadora | 23 | Mantidos. Afeta cálculos de lead time de transporte |
| `orders` | Status `CANCELED` com data de entrega | 6 | Mantidos. Recebem `dt_diff` e `is_late` (1 deles atrasado) |
| `geolocation` | Coordenadas fora dos limites do Brasil | 33 | Mantidos. Excluir ao plotar mapas |

---

## 7. Cobertura temporal

As compras vão de **04/09/2016 a 17/10/2018**, mas as pontas da série têm volume muito baixo:

| Período | Pedidos | Motivo provável |
|---|---|---|
| set/2016 a dez/2016 | 329 | Início da operação na plataforma (com dezembro de 2016 tendo 1 pedido e novembro nenhum) |
| jan/2017 a ago/2018 | 99.092 | Período com volume regular |
| set/2018 a out/2018 | 20 | Corte da extração dos dados |

Incluir as pontas em análises de evolução mensal criaria quedas artificiais.

---

## 8. Regras para as análises

Decisões que não foram aplicadas na base (para não limitar outras análises), mas devem ser seguidas por quem a utiliza:

1. **Receita e ticket médio:** considerar apenas pedidos com `order_status == "DELIVERED"`.
2. **Evolução temporal:** usar o período de **jan/2017 a ago/2018**.
3. **Contagem de clientes e recompra:** usar `customer_unique_id`, não `customer_id`.
4. **Atraso:** `is_late` e `dt_diff` só existem para pedidos com data de entrega; pedidos sem entrega ficam fora do denominador.
5. **Joins com `payments`, `reviews` ou `geolocation`:** agregar por chave antes de juntar.
6. **Mapas:** consolidar um ponto por CEP e descartar as 33 coordenadas fora do Brasil.
