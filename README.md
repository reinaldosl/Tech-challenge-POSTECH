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
│   └── tabelas-categorias.ipynb  # Fase 3
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

### Fase 5 — Agregações de crescimento e receita *(próxima)*

Evolução mensal de pedidos, receita e ticket médio; participação por categoria e UF; top produtos e sellers.

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
3. Rode os notebooks na ordem: `normalizacao-dados` → `tabelas-categorias`. Eles recriam toda a pasta `data/processed/`.

Todo notebook começa com `%run ../src/librarys.ipynb` para carregar as bibliotecas.
