# Tech Challenge POSTECH — E-commerce Olist

Tech Challenge da Fase 1 da Pós-Tech em Data Analytics (FIAP), desenvolvido em grupo.

Relatório executivo para investidores e acionistas construído a partir do [dataset público da Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (~100 mil pedidos de 2016 a 2018). O trabalho transforma os dados de vendas em uma narrativa sobre desempenho comercial, logística e satisfação do cliente, com previsão de receita e recomendações.

## Integrantes

| Nome | RM |
|---|---|
| Reinaldo Souza Lima Filho | RM378195 |
| Robinson de Freitas Castro Junior | RM378491 |
| Rodrigo de Castro Souza | RM378152 |
| Vinicius Jacomini Vaccari | RM377243 |
| Vitor Hugo Ramos Guerra | RM377472 |

## Entregáveis

| Entregável | Arquivo |
|---|---|
| 🎥 Vídeo executivo | [youtu.be/PilxiR41t3w](https://youtu.be/PilxiR41t3w) |
| Relatório executivo | [entregaveis/Relatorio_Executivo_Olist.pdf](entregaveis/Relatorio_Executivo_Olist.pdf) |
| Apresentação executiva | [entregaveis/Apresentacao_Executiva_Olist.pptx](entregaveis/Apresentacao_Executiva_Olist.pptx) |
| Métodos e achados de cada análise | [docs/analises-e-descobertas.md](docs/analises-e-descobertas.md) |
| Dicionário de dados | [docs/dicionario-de-dados.md](docs/dicionario-de-dados.md) |
| Relatório de qualidade dos dados | [docs/relatorio-de-qualidade.md](docs/relatorio-de-qualidade.md) |

## Pergunta de negócio

> Como a Olist pode transformar o crescimento das vendas em um negócio sustentável, aumentando a fidelização dos clientes e melhorando a experiência de compra?

## Principais resultados

- **Crescimento:** R$ 15,4 milhões em 96 mil pedidos entregues (jan/2017–ago/2018). A receita de jan–ago cresceu 143% de 2017 para 2018, mas se estabilizou em cerca de R$ 1 milhão por mês ao longo de 2018.
- **Fidelização:** 97% dos clientes compraram uma única vez e só 2,2% da receita vem de clientes recorrentes. O cliente que volta gasta o dobro, mas apenas 2,8% voltam.
- **Logística:** 74% do prazo de entrega está no transporte. A promessa de prazo é folgada no Norte e apertada no Nordeste e no RJ, onde estão os maiores índices de atraso.
- **Satisfação:** nota média de 4,09. Pedidos atrasados têm 54% de notas 1–2, contra 9% dos entregues no prazo. Pedidos com mais de um vendedor têm nota média de 2,86.
- **Previsão:** cerca de R$ 1,01 milhão/mês para set–nov/2018, sem considerar o efeito da Black Friday.
- **Recomendações:** calibrar o prazo prometido por rota, criar SLA de postagem para vendedores, programa de recompra entre 30 e 90 dias após a entrega e expansão no Nordeste com vendedores locais.

O detalhamento está no [relatório executivo](entregaveis/Relatorio_Executivo_Olist.pdf) e em [análises e descobertas](docs/analises-e-descobertas.md).

## Frentes de análise

| Frente | Escopo | Notebook |
|---|---|---|
| **1. Crescimento e Receita + Base de Dados** | Base tratada, dicionário de dados, qualidade e agregações de receita | [agregacoes-crescimento-receita](notebooks/analises/agregacoes-crescimento-receita.ipynb) |
| **2. Logística e SLA + Dados Externos** | Prazos de entrega, atrasos x avaliação, desempenho por região e dados do IBGE | [analise_logistica](notebooks/analises/analise_logistica.ipynb) |
| **3. Clientes e Pagamentos + Previsão** | Meios de pagamento, RFM, recompra por coorte e previsão de receita | [pagamentos-clientes](notebooks/analises/pagamentos-clientes.ipynb) |
| **4. Satisfação do Cliente** | Notas, fatores de satisfação, temas dos comentários e risco de churn | [satisfacao-do-cliente](notebooks/analises/satisfacao-do-cliente.ipynb) |
| **5. Recomendações + Storytelling** | Diagnóstico cruzado, resposta à pergunta de negócio, relatório e apresentação | [diagnostico-problemas-comportamento](notebooks/analises/diagnostico-problemas-comportamento.ipynb) |

## Estrutura

```
Tech-challenge-POSTECH/
├── data/
│   ├── challenge-description/    # Enunciado do Tech Challenge (PDF)
│   ├── raw/                      # Dados originais da Olist (sem alteração)
│   ├── external/                 # IBGE: população (Censo 2022) e renda per capita (PNAD 2025) por UF
│   └── processed/                # Base tratada, usada por todas as análises
├── docs/
│   ├── analises-e-descobertas.md # Métodos e achados de cada frente
│   ├── dicionario-de-dados.md    # Tabelas e colunas da base tratada
│   └── relatorio-de-qualidade.md # Nulos, duplicados e o que foi removido
├── notebooks/
│   ├── tratamento/                   # Construção da base (raw → processed)
│   │   ├── inventario-de-dados.ipynb     # Diagnóstico dos dados brutos
│   │   ├── normalizacao-dados.ipynb      # Padronização e camada processed
│   │   └── tabelas-categorias.ipynb      # Tabelas de categorias
│   └── analises/                     # Análises de cada frente
│       ├── agregacoes-crescimento-receita.ipynb       # Frente 1
│       ├── analise_logistica.ipynb                    # Frente 2
│       ├── pagamentos-clientes.ipynb                  # Frente 3
│       ├── satisfacao-do-cliente.ipynb                # Frente 4
│       └── diagnostico-problemas-comportamento.ipynb  # Frente 5
├── entregaveis/
│   ├── Relatorio_Executivo_Olist.pdf
│   └── Apresentacao_Executiva_Olist.pptx
├── src/
│   └── librarys.ipynb            # Bibliotecas carregadas por todos os notebooks
└── requirements.txt
```

## Base de dados

Os dados brutos passam por três etapas, em `notebooks/tratamento/`, até a base usada por todas as análises. Os notebooks de `notebooks/analises/` leem apenas de `data/processed/` (e de `data/external/`, no caso do IBGE).

| Etapa | Notebook | O que faz |
|---|---|---|
| Inventário | `inventario-de-dados.ipynb` | Diagnostica as 9 tabelas: tamanho, tipos, nulos, duplicados e conexões. Não altera nada |
| Normalização | `normalizacao-dados.ipynb` | Converte datas, padroniza textos (maiúsculas, sem acentos e sem espaços extras), remove duplicatas da geolocalização e cria a tabela `orders_enriched` com indicadores por pedido (valores, atraso, avaliação, UF) |
| Categorias | `tabelas-categorias.ipynb` | Relaciona itens vendidos com categorias, avaliações e atrasos |

### Premissas das análises

1. **Receita e ticket médio:** apenas pedidos `DELIVERED`; receita = preço + frete.
2. **Análises por mês:** de jan/2017 a ago/2018 (os meses fora desse período têm pouquíssimos pedidos).
3. **Clientes:** contados por `customer_unique_id`.
4. **Atraso:** `is_late` vazio indica pedido não entregue, que fica fora do cálculo da taxa de atraso.

Detalhes no [relatório de qualidade](docs/relatorio-de-qualidade.md#8-premissas-das-análises).

## Como reproduzir

Testado com Python 3.13.

1. Instale as bibliotecas:
   ```bash
   python -m pip install -r requirements.txt
   ```
2. Abra os notebooks com um kernel Python local (VS Code ou Jupyter). Cada notebook carrega as bibliotecas com `%run ../../src/librarys.ipynb` e usa caminhos relativos, então deve ser executado a partir da própria pasta.
3. A pasta `data/processed/` já está versionada. Para recriá-la a partir de `data/raw/`, execute na ordem:
   1. `tratamento/normalizacao-dados`
   2. `tratamento/tabelas-categorias`
   3. `analises/agregacoes-crescimento-receita` (gera as tabelas `agg_*`)
   4. `analises/analise_logistica` (gera `analise_regional_completa`)

   Os demais notebooks de análise apenas leem a base e podem ser executados em qualquer ordem.
