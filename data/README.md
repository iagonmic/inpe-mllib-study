# Dados do projeto

Esta pasta reúne os dados de focos de queimadas do INPE e as observações meteorológicas do INMET usadas no estudo. Os arquivos brutos cobrem o período de 2020 a 2024 e alimentam a integração espaço-temporal feita em `notebooks/1_data_preparation.ipynb`.

Os dados não são distribuídos com o repositório. Antes de executar os notebooks, baixe os arquivos nas fontes oficiais e mantenha exatamente a organização descrita abaixo.

## Organização esperada

```text
data/
├── README.md
├── bronze/
│   ├── inpe/                   # CSVs originais de focos de queimadas
│   ├── inmet/                  # CSVs originais das estações automáticas
│   └── inpe_inmet/             # Parquet integrado gerado pelo notebook 1
└── silver/                     # Parquet tratado gerado pelo notebook 2
```

- `bronze/inpe/` e `bronze/inmet/` são as únicas entradas que o usuário precisa baixar.
- `bronze/inpe_inmet/` é recriada pelo primeiro notebook com `mode("overwrite")`.
- `silver/` é recriada pelo segundo notebook com `mode("overwrite")`.
- As saídas Parquet são particionadas em diretórios `ano=2020` até `ano=2024`.

## 1. Dados de queimadas do INPE

### Como baixar

1. Acesse a página [Exportar dados do Banco de Queimadas do INPE](https://data.inpe.br/queimadas/bdqueimadas/#exportar-dados).
2. Selecione o período de 2020 a 2024. Para facilitar o download e a conferência, exporte um arquivo por ano.
3. Selecione os nove estados do Nordeste: Alagoas, Bahia, Ceará, Maranhão, Paraíba, Pernambuco, Piauí, Rio Grande do Norte e Sergipe.
4. Exporte em CSV e baixe os arquivos ZIP.
5. Extraia somente os CSVs para `data/bronze/inpe/`.

O notebook lê todos os arquivos que correspondem a `data/bronze/inpe/*.csv`; portanto, não deixe ZIPs ou CSVs de outro recorte nessa pasta. Como proteção adicional, a etapa de engenharia de atributos filtra novamente os nove estados do Nordeste.

### Formato e colunas esperadas

O CSV deve possuir cabeçalho, usar vírgula como separador de campos e conter estas colunas:

| Coluna | Tipo esperado | Significado |
|---|---|---|
| `DataHora` | data e hora | Instante da detecção do foco. O notebook aceita `yyyy/MM/dd HH:mm:ss` e `yyyy-MM-dd HH:mm:ss`. |
| `Satelite` | texto | Satélite ou sensor responsável pela detecção. |
| `Pais` | texto | País em que o foco foi detectado. |
| `Estado` | texto | Unidade federativa do foco. |
| `Municipio` | texto | Município do foco. |
| `Bioma` | texto | Bioma associado à localização. |
| `DiaSemChuva` | inteiro | Número de dias consecutivos sem chuva estimado pelo INPE. |
| `Precipitacao` | decimal, mm | Precipitação associada ao foco pelo INPE. |
| `RiscoFogo` | decimal | Indicador de risco de fogo fornecido pelo INPE. |
| `FRP` | decimal | *Fire Radiative Power* ou Potência Radiativa do Fogo, usada como medida de intensidade. |
| `Latitude` | decimal, graus | Latitude do foco em coordenadas geográficas. |
| `Longitude` | decimal, graus | Longitude do foco em coordenadas geográficas. |

O valor sentinela `-999`, encontrado em algumas variáveis do INPE, é convertido em nulo durante a preparação. Registros sem `FRP` são removidos posteriormente porque não é adequado imputar a variável que origina o alvo do modelo.

## 2. Dados meteorológicos do INMET

### Como baixar

1. Acesse o [BDMEP — Banco de Dados Meteorológicos do INMET](https://bdmep.inmet.gov.br/). Faça cadastro ou login, caso o portal solicite.
2. Escolha dados horários de estações automáticas.
3. Selecione as estações da região Nordeste e o intervalo de 2020 a 2024.
4. Selecione vírgula como separador decimal e faça o download.
5. Extraia os CSVs diretamente para `data/bronze/inmet/`.

O notebook lê `data/bronze/inmet/*.csv`. Cada arquivo precisa preservar o bloco de metadados que antecede o cabeçalho, pois dele são extraídos nome, código e coordenadas da estação.

### Metadados esperados no início de cada arquivo

| Campo | Significado |
|---|---|
| `Nome` | Nome da estação meteorológica. |
| `Codigo Estacao` | Identificador da estação, como `A203`. |
| `Latitude` | Latitude da estação em graus. |
| `Longitude` | Longitude da estação em graus. |
| `Altitude` | Altitude da estação em metros. |
| `Situacao` | Situação informada pelo INMET no momento da consulta. |
| `Data Inicial` | Início do intervalo exportado. |
| `Data Final` | Fim do intervalo exportado. |
| `Periodicidade da Medicao` | Periodicidade dos registros; deve ser horária. |

A situação atual da estação não é usada para descartar seu histórico: uma estação hoje marcada como “Pane” ainda pode possuir medições válidas no período estudado.

### Colunas horárias esperadas

O corpo do arquivo usa ponto e vírgula como separador de campos, vírgula decimal e valores ausentes representados por `null`. O notebook atribui os nomes normalizados abaixo, na mesma ordem da exportação do INMET:

| Coluna normalizada | Unidade / significado |
|---|---|
| `data_medicao` | Data da medição (`yyyy-MM-dd`). |
| `hora_medicao` | Hora da medição (`HHmm`). |
| `precipitacao_total_horario_mm` | Precipitação total horária, em mm. |
| `pressao_atmosferica_estacao_horaria_mb` | Pressão atmosférica horária no nível da estação, em mB. |
| `pressao_atmosferica_nivel_mar_mb` | Pressão atmosférica reduzida ao nível do mar, em mB. |
| `pressao_atmosferica_max_hora_ant_mb` | Pressão máxima na hora anterior, em mB. |
| `pressao_atmosferica_min_hora_ant_mb` | Pressão mínima na hora anterior, em mB. |
| `radiacao_global_kj_m2` | Radiação global, em kJ/m². |
| `temperatura_cpu_estacao_c` | Temperatura da CPU da estação, em °C. |
| `temperatura_ar_c` | Temperatura horária do ar, em °C. |
| `temperatura_ponto_orvalho_c` | Temperatura do ponto de orvalho, em °C. |
| `temperatura_max_hora_ant_c` | Temperatura máxima do ar na hora anterior, em °C. |
| `temperatura_min_hora_ant_c` | Temperatura mínima do ar na hora anterior, em °C. |
| `temperatura_orvalho_max_hora_ant_c` | Máxima do ponto de orvalho na hora anterior, em °C. |
| `temperatura_orvalho_min_hora_ant_c` | Mínima do ponto de orvalho na hora anterior, em °C. |
| `tensao_bateria_estacao_v` | Tensão da bateria da estação, em V. |
| `umidade_relativa_max_hora_ant_pct` | Umidade relativa máxima na hora anterior, em %. |
| `umidade_relativa_min_hora_ant_pct` | Umidade relativa mínima na hora anterior, em %. |
| `umidade_relativa_ar_pct` | Umidade relativa horária do ar, em %. |
| `vento_direcao_horaria_graus` | Direção horária do vento, em graus. |
| `vento_rajada_max_ms` | Rajada máxima do vento, em m/s. |
| `vento_velocidade_horaria_ms` | Velocidade horária do vento, em m/s. |

Não remova as linhas iniciais nem converta manualmente a codificação ou os separadores. O notebook lê o corpo com um schema explícito e decodifica o arquivo como `ISO-8859-1` para extrair os metadados.

## 3. Base integrada (`bronze/inpe_inmet`)

O primeiro notebook normaliza os instantes para a hora, mantém estações que tenham ao menos uma medição meteorológica útil naquele horário, calcula a distância de Haversine e associa a cada foco a estação válida mais próxima. A granularidade resultante é um foco de queimada com uma estação meteorológica associada.

As colunas gravadas são:

| Grupo | Colunas |
|---|---|
| Identificação e tempo | `id_foco`, `data_hora_foco`, `ano` |
| Foco do INPE | `satelite`, `estado`, `municipio`, `bioma`, `frp`, `dia_sem_chuva`, `latitude_foco`, `longitude_foco`, `precipitacao_inpe_mm` |
| Estação do INMET | `codigo_estacao`, `nome_estacao`, `latitude_estacao`, `longitude_estacao` |
| Relação espacial | `distancia_estacao_km` |
| Medições meteorológicas | `temperatura_ar_c`, `umidade_relativa_ar_pct`, `precipitacao_inmet_mm`, `pressao_atmosferica_nivel_mar_mb`, `vento_velocidade_horaria_ms`, `radiacao_global_kj_m2` |

Esta camada ainda é considerada bronze porque preserva variáveis e ausências que serão avaliadas e tratadas no notebook seguinte.

## 4. Base tratada (`silver`)

O segundo notebook limita os dados aos estados do Nordeste e a estações situadas em até 100 km do foco, remove `FRP` ausente ou inválido, trata valores nulos e cria atributos para o modelo. Entre as principais colunas estão:

| Coluna | Significado |
|---|---|
| `FOCO_INTENSO` | Alvo binário: `1` para `frp` acima da mediana e `0` para valor igual ou abaixo. |
| `dia_sem_chuva` | Dias consecutivos sem chuva, após tratamento. |
| `precipitacao_inpe_mm` | Precipitação do INPE usada como fonte principal. |
| `umidade_relativa_ar_pct` | Umidade relativa horária da estação associada. |
| `seca_prolongada` | Indicador de pelo menos 52 dias sem chuva. |
| `baixa_umidade` | Indicador de umidade no primeiro quartil da amostra. |
| `condicao_seca` | Indicador conjunto de seca prolongada e baixa umidade. |
| `mes` | Mês extraído do instante do foco. |
| `periodo_dia` | `manha`, `tarde`, `noite` ou `madrugada`. |
| `bioma`, `estado`, `satelite` | Variáveis categóricas contextuais. |
| `distancia_estacao_km` | Distância entre o foco e a estação associada. |
| `*_missing` | Flags que registram quais variáveis estavam ausentes antes da imputação. |
| `vento_velocidade_horaria_ms` | Velocidade do vento, preservada para a análise complementar dos registros disponíveis. |

Embora os dois conjuntos tenham medições de precipitação, o modelo prioriza `precipitacao_inpe_mm`. A precipitação do INMET não é usada para preencher automaticamente a do INPE porque a análise exploratória encontrou baixa concordância entre as fontes.

## Validação antes de executar

No PowerShell, a partir da raiz do repositório, confirme que há CSVs nas duas entradas:

```powershell
Get-ChildItem data\bronze\inpe\*.csv
Get-ChildItem data\bronze\inmet\*.csv
```

Confira também as primeiras linhas de um arquivo de cada fonte:

```powershell
Get-Content data\bronze\inpe\SEU_ARQUIVO.csv -TotalCount 2
Get-Content data\bronze\inmet\SEU_ARQUIVO.csv -TotalCount 12
```

Se os arquivos estiverem nos locais corretos, volte ao [README principal](../README.md#execução-dos-notebooks) e execute os notebooks na ordem indicada.

## Fontes e responsabilidade de uso

- [INPE — Programa Queimadas](https://terrabrasilis.dpi.inpe.br/queimadas/)
- [INMET — BDMEP](https://bdmep.inmet.gov.br/)

Consulte as páginas oficiais para conhecer metodologia, atualizações, licenças e termos de uso. As medições podem conter lacunas, diferenças entre sensores e variações na cobertura espacial; os resultados do estudo devem ser interpretados considerando essas limitações.
