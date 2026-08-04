# inpe-mllib-study

Estudo de Machine Learning distribuído com PySpark MLlib utilizando dados de queimadas do INPE.

## 📖 Sobre

Este repositório documenta o desenvolvimento de um pipeline de Machine Learning distribuído utilizando Apache Spark (PySpark MLlib) sobre dados públicos do INPE.

O projeto faz parte de um estudo acadêmico com foco em:

- Processamento distribuído de dados;
- Engenharia de atributos;
- Construção de pipelines de Machine Learning;
- Treinamento e avaliação de modelos utilizando MLlib.

## 🎯 Objetivo

Desenvolver um pipeline completo de Machine Learning para analisar dados de queimadas fornecidos pelo INPE, aplicando boas práticas de engenharia de dados e aprendizado de máquina distribuído.

## 🛠️ Tecnologias

- Python
- Apache Spark
- PySpark
- PySpark MLlib
- Jupyter Notebook

## 📂 Estrutura

```text
.
├── notebooks/
├── data/
├── silver/
├── models/
├── README.md
└── requirements.txt
```

## 📊 Dataset

Fonte oficial:

- INPE – Programa Queimadas
- https://terrabrasilis.dpi.inpe.br/queimadas/

> Os dados não são versionados neste repositório.

## 🚀 Como executar

Clone o projeto:

```bash
git clone https://github.com/<seu-usuario>/inpe-mllib-study.git
cd inpe-mllib-study
```

Crie o ambiente:

```bash
python -m venv .venv

# Linux/macOS
source .venv/bin/activate

# Windows
.venv\Scripts\activate
```

Instale as dependências:

```bash
pip install -r requirements.txt
```

Inicie o Jupyter:

```bash
jupyter lab
```

## 📌 Etapas do estudo

- [ ] Entendimento do problema
- [ ] Coleta dos dados
- [ ] Análise exploratória
- [ ] Limpeza dos dados
- [ ] Engenharia de atributos
- [ ] Construção do Pipeline MLlib
- [ ] Treinamento dos modelos
- [ ] Avaliação
- [ ] Ajuste de hiperparâmetros
- [ ] Conclusões

## 📚 Referências

- Apache Spark
- PySpark MLlib
- INPE - Programa Queimadas

## 📄 Licença

Projeto desenvolvido para fins acadêmicos.
