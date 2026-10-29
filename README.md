# Tech Challenge POSTECH — E-commerce Olist

Repositório do Tech Challenge da Fase 1 da Pós-Tech em Data Analytics. Um trabalho colaborativo.

## Objetivo

Construir um relatório executivo para investidores e acionistas a partir do [dataset público da Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (~100 mil pedidos de 2016 a 2018). A ideia é transformar dados de vendas em uma narrativa clara sobre desempenho comercial, logística e satisfação do cliente, terminando em recomendações e previsões.

📊 **Análises e descobertas:** [docs/analises-e-descobertas.md](docs/analises-e-descobertas.md)

## Requisitos

Definidos no enunciado (`data/challenge-description/`).

**Entregáveis**
- Repositório no GitHub com os códigos utilizados.
- Apresentação executiva com storytelling.
- Vídeo executivo de até 5 minutos, em linguagem de negócio (sem termos técnicos), apresentado por pelo menos um integrante.

**Critérios de avaliação**
- Relatório estruturado: sumário, contexto, análise e conclusões.
- Capacidade analítica: profundidade, métricas relevantes, previsões e recomendações.
- Storytelling: clareza da jornada e conexão com decisões de investimento.
- Governança: documentação dos dados, qualidade e reprodutibilidade.

## Divisão do time

| Frente | Escopo |
|---|---|
| **1. Crescimento e Receita + Base de Dados** | Base tratada única, dicionário de dados, qualidade, organização do repositório e agregações de receita |
| **2. Logística e SLA + Dados Externos** | Prazos de entrega, atrasos x avaliação, desempenho por região e dados do IBGE |
| **3. Clientes e Pagamentos + Previsão** | Meios de pagamento, RFM, recompra por coorte e previsão de receita |
| **4. Satisfação do Cliente + Padrão Visual** | Notas, fatores de satisfação, risco de churn e padrão visual dos gráficos |
| **5. Recomendações + Storytelling** | Recomendações, relatório executivo, apresentação e roteiro do vídeo |

## Estrutura

```
Tech-challenge-POSTECH/
├── data/
│   ├── challenge-description/    # Enunciado do Tech Challenge (PDF)
│   ├── raw/                      # Dados originais da Olist (não alterar)
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
│       ├── agregacoes-crescimento-receita.ipynb  # Frente 1
│       ├── analise_logistica.ipynb               # Frente 2
│       ├── pagamentos-clientes.ipynb             # Frente 3
│       └── diagnostico-problemas-comportamento.ipynb  # Cruzamento logística x pagamentos x satisfação
├── entregaveis/
│   ├── Relatorio_Executivo_Olist.pdf        # Relatório executivo
│   └── Apresentacao_Executiva_Olist.pptx    # Apresentação de 5 minutos
├── src/
│   └── librarys.ipynb            # Bibliotecas usadas em todos os notebooks
└── requirements.txt
```

## Base de dados

Os dados brutos passam por três etapas, em `notebooks/tratamento/`, até a base que todo o time usa. As análises de cada frente ficam em `notebooks/analises/` e só leem de `data/processed/`.

| Etapa | Notebook | O que faz |
|---|---|---|
| Inventário | `inventario-de-dados.ipynb` | Diagnostica as 9 tabelas: tamanho, tipos, nulos, duplicados e conexões. Não altera nada |
| Normalização | `normalizacao-dados.ipynb` | Converte datas, padroniza textos (maiúsculas, sem acentos e sem espaços extras), remove duplicatas da geolocalização e cria a tabela `orders_enriched` com indicadores por pedido (valores, atraso, avaliação, UF) |
| Categorias | `tabelas-categorias.ipynb` | Relaciona itens vendidos com categorias, avaliações e atrasos |

- [Dicionário de dados](docs/dicionario-de-dados.md): o que significa cada tabela e coluna.
- [Relatório de qualidade](docs/relatorio-de-qualidade.md): problemas encontrados e como foram tratados.

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
3. Para recriar a pasta `data/processed/`, rode na ordem: `tratamento/normalizacao-dados` → `tratamento/tabelas-categorias` → `analises/agregacoes-crescimento-receita`.

Todo notebook começa com `%run ../../src/librarys.ipynb` para carregar as bibliotecas. Novos notebooks devem ficar em `notebooks/tratamento/` (se alteram a base) ou `notebooks/analises/` (se só analisam) e usar caminhos relativos (`../../data/processed/...`).
