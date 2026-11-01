# Análises e descobertas

Métodos e principais achados de cada frente do projeto. Para a visão geral e os entregáveis, veja o [README](../README.md).

Todas as análises partem da base tratada em `data/processed/` e seguem as [premissas das análises](relatorio-de-qualidade.md#8-premissas-das-análises): receita só de pedidos `DELIVERED`, período de jan/2017 a ago/2018 e clientes contados por `customer_unique_id`.

| Frente | Notebook |
|---|---|
| 1. Crescimento e Receita + Base de Dados | `agregacoes-crescimento-receita.ipynb` |
| 2. Logística e SLA + Dados Externos | `analise_logistica.ipynb` |
| 3. Clientes e Pagamentos + Previsão | `pagamentos-clientes.ipynb` |
| 4. Satisfação do Cliente | `satisfacao-do-cliente.ipynb` |
| 5. Recomendações + Storytelling | `diagnostico-problemas-comportamento.ipynb` |

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

**Método**
- Base: `orders_enriched`. Para o lead time, só pedidos com datas em ordem cronológica (postagem depois da compra e entrega depois da postagem).
- Lead time por etapa e por UF: compra → aprovação → postagem → entrega.
- Atraso = entrega depois do prazo prometido (`order_estimated_delivery_date`).
- Dados externos: renda e população por UF do IBGE, juntadas aos indicadores logísticos na tabela `analise_regional_completa`, usada também pela Frente 5.

**Principais achados**
- A correlação entre atraso e nota de avaliação é de **-0,33** (negativa moderada): pedidos atrasados recebem notas menores.
- A etapa de transporte (postagem até a entrega) é a que mais varia entre as UFs, enquanto aprovação e postagem são parecidas em todo o país.

**Dados externos (IBGE)** — `data/external/ibge_uf.csv`
- `renda_media_ibge`: rendimento nominal mensal domiciliar per capita, 2025 — IBGE, PNAD Contínua (divulgado em 27/02/2026).
- `populacao_estimada`: população residente — IBGE, Censo Demográfico 2022 (primeira divulgação, jun/2023).
- Os valores de MG (20.869.101) e ES (3.885.049) diferem do Censo 2022 (20.539.989 e 3.833.712). Foram mantidos como usados nas análises. Com os valores do Censo, o único número afetado seria o de pedidos por 100 mil habitantes no Sudeste, de 77 para 78.

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

**Método**
- Unidade de análise: pedido. Pedidos com mais de uma avaliação ficam com a mais recente.
- Notas 1–2 = insatisfeito, 3 = neutro, 4–5 = satisfeito.
- Categoria do pedido = categoria de maior valor entre os itens; só categorias com 100+ pedidos avaliados.
- Temas dos comentários identificados por palavras-chave (entrega, produto, atendimento, preço, embalagem).
- Risco de churn (proxy): cliente cuja última compra avaliada recebeu nota 1–2. Não há rótulo de churn na base.

**Principais achados**
- 98.673 pedidos avaliados, nota média de **4,09**: 77% de notas 4–5 e 15% de notas 1–2.
- **Atraso é o principal fator de insatisfação:** 54% de notas 1–2 nos pedidos atrasados, contra 9% nos entregues no prazo. Acima de 30 dias de entrega, 63% de notas baixas.
- Pedidos de maior valor têm um pouco mais de insatisfação (18% de notas baixas no quartil mais caro, contra 12% no mais barato).
- Categorias com maior proporção de notas baixas: moda masculina (26%), móveis de escritório (23%) e áudio (22%).
- Nos comentários, os temas mais citados são produto/qualidade e entrega/atraso; menções a atendimento e embalagem têm a maior proporção de notas baixas (45% e 46%).
- 14,7% dos clientes têm sinal de risco de churn (última compra avaliada com nota 1–2).

**Cuidados:** as relações são descritivas e não provam causa; os comentários são uma amostra autosselecionada.

---

## Frente 5 — Recomendações e Storytelling
`notebooks/analises/diagnostico-problemas-comportamento.ipynb`

Cruza logística, pagamentos e satisfação para identificar os problemas da operação (Partes A a D) e responde à pergunta de negócio (Parte E):

> Como a Olist pode transformar o crescimento das vendas em um negócio sustentável, aumentando a fidelização dos clientes e melhorando a experiência de compra?

Resultado consolidado no [relatório executivo](../entregaveis/Relatorio_Executivo_Olist.pdf) e na [apresentação](../entregaveis/Apresentacao_Executiva_Olist.pptx).

**Principais achados**
- **O gargalo é o transporte:** 9,4 dos 12,6 dias médios de entrega (74%) ficam entre a postagem e a entrega.
- **A promessa de prazo é desigual:** Norte recebe 18 a 21 dias de folga e quase não atrasa; o Nordeste recebe 9 a 14 dias e lidera os atrasos (AL 21%, MA 18%). RJ tem 12% de atraso em 12,3 mil pedidos.
- **Crise de atraso em fev–mar/2018** (até 19% de atraso) sem pico de volume.
- **28% dos atrasos começam no seller**, que posta depois do prazo combinado.
- **Pedidos com mais de um seller** têm nota 2,86 (contra 4,17): o cliente recebe parte do pedido e avalia o que falta.
- **O boleto atrasa a aprovação** em cerca de 1 dia; pedidos com voucher são cancelados quase 3x mais.
- **O crescimento veio de clientes novos e parou em 2018:** só 2,2% da receita vem de clientes recorrentes. O cliente que volta gasta o dobro (R$ 321 x R$ 162), mas só 2,8% voltam.
- **Experiência ruim reduz a recompra** (atraso na 1ª compra: 2,0% x 2,9%), mas mesmo a melhor experiência leva a cerca de 3%. Um programa de fidelização que leve a recompra a 5% renderia cerca de R$ 258 mil em 12 meses.
- **O Nordeste tem 27% da população e 9% dos pedidos**, com pior experiência em atraso, frete, nota e recompra.

**Recomendações (hipóteses a testar)**
1. Entregar o que promete: calibrar o prazo por rota, SLA de postagem para sellers e avisar entregas divididas.
2. Criar motivos para voltar: contatos e cupom de segunda compra entre 30 e 90 dias após a entrega, com foco nas categorias de maior recompra.
3. Crescer no Nordeste depois de corrigir a experiência, atraindo sellers locais.
4. Cuidar da saúde financeira: revisar o parcelamento sem juros acima de 6x e incentivar pagamento instantâneo.

**Limitações:** comparações descritivas; a base não traz transportadora, custos de marketing nem dados de conversão.
