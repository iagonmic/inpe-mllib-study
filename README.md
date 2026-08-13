# Classificação de focos intensos de queimadas com PySpark MLlib

Estudo acadêmico de processamento distribuído e Machine Learning que combina focos de queimadas do Instituto Nacional de Pesquisas Espaciais (INPE) com observações meteorológicas do Instituto Nacional de Meteorologia (INMET). O recorte analisado é a região Nordeste do Brasil, entre 2020 e 2024.

## Sobre o projeto

O projeto investiga como precipitação, umidade, duração de períodos sem chuva e velocidade do vento se relacionam com a intensidade dos focos de queimadas. Os dados das duas fontes são unidos por hora e pela estação meteorológica válida mais próxima, calculada com a distância de Haversine.

A variável-alvo é `FOCO_INTENSO`: um foco recebe valor `1` quando sua Potência Radiativa do Fogo (`FRP`) está acima da mediana dos dados válidos e `0` caso contrário. Sobre essa base, o estudo:

- limpa e integra dados do INPE e do INMET com Spark;
- cria atributos meteorológicos, temporais e geográficos;
- testa Regressão Logística e Random Forest com PySpark MLlib;
- avalia AUC-ROC, acurácia, precisão, recall e diferentes limiares de classificação;
- discute a utilidade do modelo, suas limitações e possíveis vieses dos sensores e da cobertura das estações.

O objetivo é apoiar a priorização de monitoramento e de recursos de prevenção e combate, não substituir alertas operacionais nem demonstrar relações causais.

## Estrutura do repositório

```text
.
├── data/
│   ├── README.md                 # download, organização e dicionário dos dados
│   ├── bronze/
│   │   ├── inpe/                 # CSVs brutos do INPE
│   │   ├── inmet/                # CSVs brutos do INMET
│   │   └── inpe_inmet/           # Parquet integrado, gerado na etapa 1
│   └── silver/                   # Parquet tratado, gerado na etapa 2
├── notebooks/
│   ├── 1_data_preparation.ipynb
│   ├── 2_feature_engineering.ipynb
│   └── 3_model_pipeline.ipynb
├── pyproject.toml
├── uv.lock
└── README.md
```

Os arquivos de dados são grandes e não fazem parte do versionamento. Consulte o [README da pasta `data`](data/README.md) antes da primeira execução: ele explica como obter as duas bases, onde colocar cada arquivo, o formato esperado e o significado das colunas.

## Pré-requisitos

Antes de executar, tenha instalado:

- Git;
- [uv](https://docs.astral.sh/uv/) para criar e sincronizar o ambiente Python;
- Python 3.13, conforme `.python-version` e `pyproject.toml` — o `uv` pode instalar a versão necessária;
- JDK 17 ou superior, com a variável `JAVA_HOME` configurada;
- no Windows, os binários do Hadoop descritos em [Solução de problemas](#solução-de-problemas);
- espaço em disco para os CSVs e Parquets gerados;
- preferencialmente, pelo menos 8 GB de memória disponível para o driver do Spark na preparação dos dados.

O projeto utiliza PySpark 4.2.0, JupyterLab, pandas, NumPy e Matplotlib. As versões exatas ficam registradas em `uv.lock`.

Para conferir as instalações no PowerShell:

```powershell
git --version
uv --version
java -version
$env:JAVA_HOME
```

## Instalação

Clone o repositório e entre na pasta:

```powershell
git clone https://github.com/iagonmic/inpe-mllib-study.git
cd inpe-mllib-study
```

Sincronize o ambiente reproduzível definido em `uv.lock`:

```powershell
uv sync
```

Não é necessário ativar o ambiente virtual para usar os comandos abaixo: `uv run` executa tudo no ambiente do projeto.

## Preparação dos dados

Baixe os dados públicos do INPE e do INMET antes de abrir o primeiro notebook. Ao terminar, esta estrutura mínima deve existir:

```text
data/
└── bronze/
    ├── inpe/
    │   └── *.csv
    └── inmet/
        └── *.csv
```

As instruções detalhadas de download, os períodos, filtros, formatos e dicionários de colunas estão em [`data/README.md`](data/README.md).

## Execução dos notebooks

Execute os notebooks pela ordem numérica dos nomes, pois cada um consome o resultado produzido pelo anterior:

1. `1_data_preparation.ipynb`: lê os CSVs brutos, trata tipos e valores inválidos, associa cada foco à estação meteorológica válida mais próxima na mesma hora e grava `data/bronze/inpe_inmet/` em Parquet particionado por ano.
2. `2_feature_engineering.ipynb`: filtra o Nordeste, limita a distância da estação a 100 km, trata ausências, cria a variável-alvo e novas features e grava `data/silver/` em Parquet particionado por ano.
3. `3_model_pipeline.ipynb`: testa as hipóteses, monta os pipelines do MLlib, treina e compara os modelos e interpreta métricas e coeficientes.

Abra o primeiro notebook com:

```powershell
uv run jupyter notebook notebooks/1_data_preparation.ipynb
```

No Jupyter, execute todas as células do notebook e confirme que ele terminou sem erros antes de abrir o próximo. Depois, repita com:

```powershell
uv run jupyter notebook notebooks/2_feature_engineering.ipynb
uv run jupyter notebook notebooks/3_model_pipeline.ipynb
```

Também é possível iniciar o JupyterLab na raiz do projeto e abrir os arquivos pela interface:

```powershell
uv run jupyter lab
```

## Solução de problemas

### `JAVA_HOME` não está configurado

Confirme que um JDK 17 ou superior está instalado e que `JAVA_HOME` aponta para a pasta do JDK, não para a pasta `bin`. Feche e abra o terminal novamente após alterar a variável.

Em um novo PowerShell, confira:

```powershell
java -version
$env:JAVA_HOME
```

### Erros do Hadoop no Windows

Se o Spark apresentar erros relacionados ao Hadoop, à biblioteca nativa ou ao `winutils.exe`, crie a pasta necessária no PowerShell:

```powershell
New-Item -ItemType Directory -Force C:\hadoop\bin
```

Baixe os arquivos `hadoop.dll` e `winutils.exe` da pasta [`hadoop-3.3.6/bin` do repositório winutils](https://github.com/cdarlint/winutils/tree/master/hadoop-3.3.6/bin) e coloque ambos em:

```text
C:\hadoop\bin\
```

A versão 3.3.6 do Hadoop é compatível com o Spark 4.2.0 utilizado neste estudo. Em seguida, nas variáveis de ambiente do Windows:

1. crie a variável `HADOOP_HOME` com o valor `C:\hadoop`;
2. adicione `%HADOOP_HOME%\bin` à variável `Path`;
3. feche e abra novamente o terminal e o Jupyter para aplicar as alterações.

Para conferir a configuração em um novo PowerShell, execute:

```powershell
winutils.exe
```

### `Path does not exist: .../data/bronze/RESULTADOS_2025.csv`

Confira o nome e a localização do arquivo indicado no erro. Se uma versão modificada do notebook referenciar `RESULTADOS_2025.csv`, o caminho relativo dessa referência deve ser exatamente:

```text
data/bronze/RESULTADOS_2025.csv
```

Os notebooks atuais deste repositório não usam esse nome único: eles leem todos os CSVs das duas fontes nos caminhos abaixo. Prefira manter os caminhos definidos no código atual:

```text
data/bronze/inpe/*.csv
data/bronze/inmet/*.csv
```

Não altere a estrutura de pastas e extraia os arquivos dos pacotes ZIP antes de executar o notebook. Veja a preparação completa em [`data/README.md`](data/README.md).

### Kernel ou dependências não aparecem no Jupyter

Feche o Jupyter, sincronize novamente o ambiente e reabra o notebook:

```powershell
uv sync
uv run jupyter notebook notebooks/1_data_preparation.ipynb
```

### Spark fica sem memória (`OutOfMemoryError`)

Feche outros programas que estejam consumindo memória e confirme que o primeiro notebook mantém `spark.driver.memory` em `8g`. A integração espaço-temporal cria um conjunto intermediário muito maior do que a saída final; por isso, uma máquina com pouca memória pode não concluir essa etapa. Não execute várias sessões Spark ao mesmo tempo e encerre a sessão anterior antes de reiniciar um notebook.

### O notebook seguinte não encontra o Parquet

Confirme que todas as células do notebook anterior terminaram com sucesso. A sequência de dependências é:

```text
CSVs brutos → data/bronze/inpe_inmet/ → data/silver/ → modelos e avaliações
```

## Fontes

- [Programa Queimadas do INPE](https://terrabrasilis.dpi.inpe.br/queimadas/)
- [Banco de Dados Meteorológicos do INMET](https://bdmep.inmet.gov.br/)
- [Documentação do Apache Spark](https://spark.apache.org/docs/latest/)

## Licença

O código deste projeto é disponibilizado sob os termos da [GNU General Public License v3.0](LICENSE). Os dados pertencem às respectivas instituições de origem e estão sujeitos aos termos de uso delas.
