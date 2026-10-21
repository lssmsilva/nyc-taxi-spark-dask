# Análise de 47 milhões de corridas de táxi de NYC (Spark e Dask)

Projeto da pós-graduação em Engenharia e Análise de Dados (CESAR School), módulo de Processamento de Dados em Larga Escala.

Pipeline em Python que lê ~7,5 GB de CSVs do **NYC Yellow Taxi** (jan/2015 e jan–mar/2016, ~47 milhões de linhas), limpa os dados uma única vez, grava em **Parquet** e responde a quatro perguntas de negócio usando **PySpark** e **Dask**, sem carregar o dataset inteiro na memória.

## Principais resultados

| Pergunta | Resultado |
|---|---|
| O volume de corridas caiu de jan/2015 para jan/2016? | Sim: média diária de 2,87 mi para 2,40 mi (**−16,1%**). A queda nos fins de semana (≈ −24% a −28%) foi cerca do dobro da dos dias úteis (≈ −10% a −13%). |
| O percentual de gorjeta muda por horário e distância? | Corridas curtas têm gorjeta maior (**24,5%** abaixo de 1 milha, contra ~19% a 21% nas demais faixas). Por horário a variação é pequena (20% a 22%), e dia útil é ligeiramente maior que fim de semana. |
| Quando o trânsito é mais lento e ele piorou? | Mais lento às 9h–11h em dias úteis (**9,8 mph** na terça às 9h). Velocidade agregada caiu de 13,59 para 13,21 mph (**−2,8%**), com a maior piora às 16h–17h. |
| A demanda é concentrada e as regiões mais demandadas são as mais rentáveis? | As 10 células de ~1 km mais demandadas concentram **43% a 50%** dos embarques por faixa do dia. As células de aeroporto rendem **US$ 1,47/min**, contra US$ 1,15 nas de maior demanda. |

Limpeza: 47.248.845 linhas brutas, 45.994.800 mantidas (97,3%).

> Os dados mostram padrões compatíveis com a expansão de aplicativos de transporte, mas não provam a causa. As janelas de madrugada de quinta e sexta mostram um padrão simétrico que pode ser efeito de calendário e merece investigação (detalhes no notebook).

## Decisões técnicas

- **Schema explícito** na leitura dos CSVs (evita ler o arquivo duas vezes).
- **Limpeza única** gravada em Parquet (colunar e comprimido). Spark e Dask leem o mesmo Parquet.
- **Nenhum `collect()` ou `toPandas()` sobre dados grandes**: apenas resultados já agregados vão para o pandas.
- Agregações distribuídas (`count`, `sum`, `avg`, `percentile_approx`), `broadcast join` para tabelas pequenas e funções de janela particionadas.
- Dask com `LocalCluster` e limite de memória por worker.
- Gorjetas analisadas só para pagamentos com cartão (em dinheiro não são registradas).

## Como executar

1. Instale as dependências: `pip install -r requirements.txt`
2. Os dados (~1,8 GB compactados) vêm do Kaggle: <https://www.kaggle.com/datasets/elemento/nyc-yellow-taxi-trip-data>. O notebook tenta baixar com `kagglehub`; se preferir, extraia manualmente e aponte a variável `DATA_DIR`.
3. Abra `nyc_taxi_spark_dask.ipynb` e execute as células em ordem. A limpeza leva alguns minutos (≈ 385 s no ambiente original).

Variáveis de ambiente opcionais: `SPARK_DRIVER_MEMORY`, `DASK_WORKER_MEMORY`, `DATA_DIR`, `PARQUET_DIR`.

## Autora

Laura Silva Soares de Melo · [LinkedIn](https://www.linkedin.com/in/laura-soares-313067195) · [GitHub](https://github.com/lssmsilva)
