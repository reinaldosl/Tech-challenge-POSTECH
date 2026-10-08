# Tech Challenge POSTECH — E-commerce Olist

Repositório do Tech Challenge da Fase 1 da Pós-Tech em Data Analytics. Um trabalho colaborativo.

## Objetivo

Construir um relatório executivo para investidores a partir do [dataset público da Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (~100 mil pedidos de 2016 a 2018), com análises de desempenho comercial, logística e satisfação do cliente, terminando em recomendações.

## Estrutura

```
Tech-challenge-POSTECH/
├── data/
│   ├── challenge-description/    # Enunciado do Tech Challenge (PDF)
│   ├── raw/                      # Dados originais da Olist (não alterar)
│   └── processed/                # Base tratada, usada por todas as análises
├── docs/
│   ├── dicionario-de-dados.md    # Tabelas e colunas da base tratada
│   └── relatorio-de-qualidade.md # Nulos, duplicados e o que foi removido
├── notebooks/
│   ├── inventario-de-dados.ipynb # Fase 1
│   ├── normalizacao-dados.ipynb  # Fase 2
│   ├── tabelas-categorias.ipynb  # Fase 3
│   ├── agregacoes-crescimento-receita.ipynb  # Fase 5
│   └── pagamentos-clientes.ipynb             # Frente 3
├── src/
│   └── librarys.ipynb            # Bibliotecas usadas em todos os notebooks
└── requirements.txt
```

## Fases

### Fase 1 — Inventário dos dados brutos
`notebooks/inventario-de-dados.ipynb`

Primeiro olhar sobre as 9 tabelas: tamanho, tipos, nulos, duplicados e se as tabelas se conectam. Nada é alterado aqui, só diagnosticado.

### Fase 2 — Normalização e camada processed
`notebooks/normalizacao-dados.ipynb`

- Converte as datas de texto para data.
- Padroniza os textos: maiúsculas, sem acentos e sem espaços extras (`são  paulo` → `SAO PAULO`).
- Remove as linhas duplicadas da geolocalização.
- Cria a tabela `orders_enriched` (uma linha por pedido) com os indicadores:

| Coluna | O que é |
|---|---|
| `order_price` / `order_freight` / `order_total` | Valor dos produtos, do frete e total do pedido |
| `review_score` | Avaliação média do pedido |
| `customer_state` / `customer_city` | UF e cidade do cliente |
| `purchase_year_month` | Mês da compra (`2017-11`) |
| `dt_diff` | Dias entre a entrega e a data prometida (positivo = atrasou) |
| `is_late` | 1 se atrasou, 0 se chegou no prazo |

### Fase 3 — Tabelas de categorias
`notebooks/tabelas-categorias.ipynb`

- `order_items_category`: cada item vendido com sua categoria e os dados do pedido.
- `category_reviews`: por categoria, quantidade de pedidos, avaliação média, % de atraso e receita.

### Fase 4 — Documentação
`docs/`

- [Dicionário de dados](docs/dicionario-de-dados.md): o que significa cada tabela e coluna.
- [Relatório de qualidade](docs/relatorio-de-qualidade.md): problemas encontrados e como foram tratados.

### Fase 5 — Agregações de crescimento e receita
`notebooks/agregacoes-crescimento-receita.ipynb`

Evolução mensal de pedidos, receita e ticket médio; participação por categoria e UF; top produtos e sellers. Considera só pedidos entregues de jan/2017 a ago/2018. Gera as tabelas `agg_mensal`, `agg_categoria`, `agg_uf`, `agg_produto` e `agg_seller` em `data/processed/`.

### Frente 3 — Clientes e Pagamentos
`notebooks/pagamentos-clientes.ipynb`

Base: `orders_enriched` + `customers` (via `customer_id`), apenas pedidos `DELIVERED` de jan/2017 a ago/2018. Clientes contados por `customer_unique_id`. Validação: receita total de R$ 15.373.120,01, igual às tabelas agregadas.

**RFM**
- Recência: dias entre a última compra e 29/08/2018. R e M divididos em quintis (nota 5 = melhor).
- Frequência em 3 faixas (1, 2 e 3+ pedidos): 97% dos clientes compraram uma única vez, o que impede a divisão em quintis.
- 5 segmentos: Recorrentes, Novos de alto valor, Novos de baixo valor, Inativos de alto valor e Inativos de baixo valor.

**Coorte**
- Coorte = mês da primeira compra; retenção = % de clientes da coorte que compraram em cada mês seguinte.

**Previsão**
- Teste com treino até mai/2018 e validação em jun–ago/2018: tendência linear (erro de 34,2%) x média dos 3 meses recentes (erro de 11,8%). Escolhida a média recente.

**Pagamentos**
- Participação por forma de pagamento (em pedidos e em valor) e perfil de parcelamento no cartão, agrupado em faixas (1x, 2-3x, 4-6x, 7-10x, 11x+). Pedidos com mais de um pagamento contados uma única vez (contagem distinta de `order_id`); 2 pagamentos com 0 parcelas desconsiderados.

**Principais achados**
- 97% dos clientes compraram uma única vez; apenas 8% dos que compraram duas vezes voltaram para uma terceira.
- Em média, 0,5% dos clientes voltam a comprar no mês seguinte à primeira compra, sem melhora de 2017 para 2018.
- Clientes de alto valor (38% da base) geraram 68% da receita.
- Prioridade: novos de alto valor (15,5% dos clientes, 28,4% da receita, última compra há 91 dias em média).
- Cartão de crédito em 77% dos pedidos. Dois terços dos pagamentos no cartão são parcelados; a faixa de 7 a 10x concentra 31% do valor pago no cartão com 15% dos pagamentos.
- Previsão de cerca de R$ 1,01 milhão/mês para set–nov/2018 (faixa de R$ 889 mil a R$ 1,13 milhão). Não incorpora sazonalidade: novembro tende a superar a faixa por efeito da Black Friday.

## Guia para o time

A base está pronta para as demais frentes. Por onde começar e quais cuidados tomar:

| Frente | Por onde começar | Cuidados |
|---|---|---|
| **2. Logística e SLA** | `orders_enriched`: datas de compra, aprovação, postagem e entrega; `dt_diff`; `is_late`; `customer_state` | 166 pedidos com postagem antes da compra e 23 com entrega antes da postagem distorcem o lead time. Na `geolocation`, 33 coordenadas estão fora do Brasil e cada CEP tem vários pontos (use a média) |
| **3. Clientes e Pagamentos** | `payments`, `customers` (`customer_unique_id`), `orders_enriched`. Para a previsão, `agg_mensal` | Um pedido pode ter vários pagamentos: agregue por pedido antes do join. Conte clientes por `customer_unique_id` |
| **4. Satisfação** | `reviews` (comentários em maiúsculas e sem acento), `review_score` e `is_late` em `orders_enriched`, `category_reviews` | Alguns pedidos têm mais de uma avaliação; em `orders_enriched` o `review_score` já é a média do pedido. O padrão visual do grupo pode ser aplicado em `src/librarys.ipynb`, que todos os notebooks carregam |
| **5. Recomendações e Storytelling** | Tabelas `agg_*` e os achados da Fase 5 | — |

**Principais achados de crescimento e receita** (jan/2017–ago/2018, pedidos entregues):
- 96 mil pedidos, R$ 15,4 milhões de receita e ticket médio de R$ 159,79.
- Receita de jan–ago cresceu **143%** de 2017 para 2018, puxada pelo volume de pedidos (+140%); o ticket médio ficou estável (+1,4%).
- Pico em **nov/2017** (Black Friday). Em 2018 a receita se estabiliza em torno de R$ 1 milhão por mês.
- **SP responde por 37% da receita** (SP, RJ e MG somam 63%). Estados mais distantes têm ticket médio maior, possivelmente pelo frete.
- Receita pulverizada: são precisas 18 categorias para chegar a 80% da receita, e o top 10 sellers responde por apenas 13%.

Antes de analisar, leia o [dicionário de dados](docs/dicionario-de-dados.md) e as [regras para as análises](docs/relatorio-de-qualidade.md#8-regras-para-as-análises).

## Regras para usar a base

1. **Leia sempre de `data/processed/`**, nunca de `data/raw/`.
2. **Receita e ticket médio:** só pedidos `DELIVERED`.
3. **Análises por mês:** use de jan/2017 a ago/2018 (os meses fora disso têm pouquíssimos pedidos).
4. **Contar clientes:** use `customer_unique_id`.
5. **`is_late` vazio** significa pedido não entregue — não conta como "no prazo".

Detalhes no [relatório de qualidade](docs/relatorio-de-qualidade.md#8-regras-para-as-análises).

## Como rodar

1. Instale as bibliotecas:
   ```bash
   python -m pip install -r requirements.txt
   ```
2. No VS Code, selecione o **kernel Python local** (não o Colab).
3. Rode os notebooks na ordem: `normalizacao-dados` → `tabelas-categorias` → `agregacoes-crescimento-receita`. Eles recriam toda a pasta `data/processed/`.

Todo notebook começa com `%run ../src/librarys.ipynb` para carregar as bibliotecas.
