# Análises e descobertas

Métodos e principais achados de cada frente do projeto. Para a ideia geral, os requisitos e o uso da base, veja o [README](../README.md).

Todas as análises partem da base tratada em `data/processed/` e seguem as [regras para as análises](relatorio-de-qualidade.md#8-regras-para-as-análises): receita só de pedidos `DELIVERED`, período de jan/2017 a ago/2018 e clientes contados por `customer_unique_id`.

| Frente | Notebook | Status |
|---|---|---|
| 1. Crescimento e Receita + Base de Dados | `agregacoes-crescimento-receita.ipynb` | Concluída |
| 2. Logística e SLA + Dados Externos | `analise_logistica.ipynb` | Entregue, documentação pendente |
| 3. Clientes e Pagamentos + Previsão | `pagamentos-clientes.ipynb` | Concluída |
| 4. Satisfação do Cliente + Padrão Visual | `satisfacao-do-cliente.ipynb` | Entregue, documentação pendente |
| 5. Recomendações + Storytelling | — | Em andamento |

---

## Frente 1 — Crescimento e Receita
`notebooks/analises/agregacoes-crescimento-receita.ipynb`

**Método**
- Pedidos `DELIVERED` de jan/2017 a ago/2018 (96.211 pedidos).
- Receita = preço + frete (valor pago pelo cliente). Ticket médio = receita ÷ pedidos.
- Gera as tabelas `agg_mensal`, `agg_categoria`, `agg_uf`, `agg_produto` e `agg_seller` em `data/processed/`.
- Validação: a receita total é a mesma em todas as agregações (R$ 15.373.120,01).

**Principais achados**
- 96 mil pedidos, R$ 15,4 milhões de receita e ticket médio de R$ 159,79.
- Receita de jan–ago cresceu **143%** de 2017 para 2018, puxada pelo volume de pedidos (+140%); o ticket médio ficou estável (+1,4%).
- Pico em **nov/2017** (Black Friday). Em 2018 a receita se estabiliza em torno de R$ 1 milhão por mês.
- **SP responde por 37% da receita** (SP, RJ e MG somam 63%). Estados mais distantes têm ticket médio maior, possivelmente pelo frete.
- Receita pulverizada: são precisas 18 categorias para chegar a 80% da receita, e o top 10 sellers responde por apenas 13%.

---

## Frente 2 — Logística e SLA
`notebooks/analises/analise_logistica.ipynb`

*Documentação de método e achados a ser incluída pela responsável pela frente.*

**Por onde começar:** `orders_enriched` — datas de compra, aprovação, postagem e entrega; `dt_diff`; `is_late`; `customer_state`.

**Cuidados:** 166 pedidos com postagem antes da compra e 23 com entrega antes da postagem distorcem o lead time. Na `geolocation`, 33 coordenadas estão fora do Brasil e cada CEP tem vários pontos (use a média).

---

## Frente 3 — Clientes e Pagamentos
`notebooks/analises/pagamentos-clientes.ipynb`

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

---


## Frente 4 — Satisfação do Cliente
`notebooks/analises/satisfacao-do-cliente.ipynb`

**Objetivo:** Analisar a satisfação dos clientes com base nas avaliações dos pedidos, identificando fatores relacionados à insatisfação e oportunidades de melhoria.

**Análises realizadas:**
- Distribuição das avaliações de 1 a 5 estrelas.
- Classificação das avaliações em satisfeitas, neutras e insatisfeitas.
- Relação entre atrasos nas entregas e notas recebidas.
- Análise da satisfação por faixa de valor dos pedidos.
- Identificação de categorias com maiores índices de insatisfação.
- Análise dos comentários e possíveis sinais de risco de abandono.

**Bases utilizadas:** `orders_enriched.csv`, `reviews.csv`, `order_items_category.csv` e `customers.csv`.

**Principais resultados:**
- Total de pedidos analisados: 99.441.
- Pedidos com avaliação: 98.673.
- Nota média das avaliações: 4,09.
- Predominância de avaliações positivas.

**Cuidados:** Alguns pedidos possuem mais de uma avaliação. Para evitar duplicidades, foi considerada a avaliação mais recente de cada pedido. As análises utilizam os dados tratados, sem alterar os arquivos originais.

---

## Frente 5 — Recomendações e Storytelling

*Em andamento.*

**Por onde começar:** as tabelas `agg_*` e os achados das frentes acima.
