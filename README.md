# 🌳 ETL - Multas Ambientais no Brasil

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/Pandas-2.0+-green.svg)](https://pandas.pydata.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

> Sistema de ETL (Extract, Transform, Load) para análise de multas ambientais aplicadas no Brasil, implementando um Data Warehouse dimensional seguindo o modelo Star Schema.

## 📋 Sobre o Projeto

Este projeto implementa um processo completo de ETL para transformar dados brutos de multas ambientais aplicadas em todos os estados brasileiros em um modelo dimensional (Data Warehouse), permitindo análises multidimensionais avançadas sobre infrações ambientais no Brasil.

### 🎯 Objetivos

- **Consolidar** dados de 27 estados brasileiros em um único modelo dimensional
- **Normalizar** valores de multas considerando 10 moedas históricas do Brasil
- **Padronizar** datas e corrigir inconsistências nos dados originais
- **Implementar** um Star Schema otimizado para análises OLAP
- **Facilitar** consultas analíticas sobre infrações ambientais

---

## 🏗️ Arquitetura do Data Warehouse

### Star Schema

```
                    ┌──────────────────┐
                    │     DTempo       │
                    ├──────────────────┤
                    │ pk_tempo PK      │
                    │ data             │
                    │ ano              │
                    │ mes              │
                    │ dia              │
                    │ trimestre        │
                    │ diaSemana        │
                    └────────┬─────────┘
                             │
       ┌─────────────────────┼─────────────────────┐
       │                     │                     │
┌──────┴────────┐     ┌──────┴──────────────┐  ┌──┴────────────┐
│  DInfrator    │     │  FAutoInfracao      │  │    DLocal     │
├───────────────┤     ├─────────────────────┤  ├───────────────┤
│pk_infrator PK │─────│ pk_fato PK          │──│ pk_local PK   │
│documento      │     │ numAuto             │  │ uf            │
│nome           │     │ fk_tempo FK         │  │ nome_estado   │
│tipoPessoa     │     │ fk_infrator FK      │  │ municipio     │
└───────────────┘     │ fk_local FK         │  │ regiao        │
                      │ fk_infracao FK      │  └───────────────┘
       ┌──────────────│ fk_debito FK        │──────────┐
       │              │ valorOriginal       │          │
       │              │ moedaOriginal       │          │
       │              │ valorPadrao         │          │
       │              └─────────────────────┘          │
       │                                               │
┌──────┴────────┐                            ┌─────────┴──────┐
│  DInfracao    │                            │    DDebito     │
├───────────────┤                            ├────────────────┤
│pk_infracao PK │                            │ pk_debito PK   │
│tipoAuto       │                            │ situacao       │
│tipoInfracao   │                            │ moeda          │
│enquadramento  │                            └────────────────┘
│Legal          │
└───────────────┘
```

### 📊 Dimensões

| Dimensão | Chave Primária | Registros | Descrição |
|----------|---------------|-----------|-----------|
| **DTempo** | `pk_tempo` | 14.767 | Datas das infrações (1977-2025) |
| **DInfrator** | `pk_infrator` | 462.105 | Infratores (PF/PJ) |
| **DLocal** | `pk_local` | 5.463 | Localização geográfica (27 UFs) |
| **DInfracao** | `pk_infracao` | 48.944 | Tipos de infrações |
| **DDebito** | `pk_debito` | 354 | Situações de débito e moedas |

### 📈 Tabela Fato

| Tabela | Chave Primária | Registros | Valor Total |
|--------|---------------|-----------|-------------|
| **FAutoInfracao** | `pk_fato` | 707.247 | R$ 99+ bilhões |

---

## 🚀 Funcionalidades

### ✨ Principais Features

- ✅ **Carga de Dados**: Importação automática de 27 arquivos CSV (um por estado)
- ✅ **Limpeza de Dados**: Correção de datas inválidas e inconsistências
- ✅ **Conversão Monetária**: Normalização de 10 moedas históricas brasileiras para Real
- ✅ **Modelo Dimensional**: Implementação completa de Star Schema
- ✅ **Nomenclatura Padronizada**: 
  - `pk_*` para Primary Keys nas dimensões
  - `fk_*` para Foreign Keys na tabela fato
- ✅ **Integridade Referencial**: 100% de relacionamentos válidos
- ✅ **Exportação**: Geração de CSVs para todas as tabelas dimensionais

### 💱 Conversão de Moedas Suportadas

O sistema converte automaticamente valores das seguintes moedas históricas para Real:

| Moeda | Período | Taxa de Conversão |
|-------|---------|-------------------|
| Real | 1994-atual | 1:1 |
| Cruzeiro Real | 1993-1994 | 2.750:1 |
| Cruzeiro (90-93) | 1990-1993 | 2.750.000:1 |
| Cruzado Novo | 1989-1990 | 2.750.000:1 |
| Cruzado | 1986-1989 | 2.750.000.000:1 |
| Cruzeiro (70-86) | 1970-1986 | 2.750.000.000.000:1 |
| UFIR | 1991-2000 | ~1,06:1 |
| BTN | 1986-1991 | Calculado |
| MVR | 1989-1991 | Calculado |
| OTN | 1986-1989 | Calculado |

---

## 📁 Estrutura do Projeto

```
etl-multas-ambientais-brasil/
│
├── ETL.ipynb                           # Notebook principal com todo o processo ETL
├── README.md                           # Este arquivo
│
├── Dados/                              # Dados de entrada (CSVs por estado)
│   ├── multasDistribuidasBensTuteladosAC.csv
│   ├── multasDistribuidasBensTuteladosAL.csv
│   ├── ... (27 arquivos, um por estado)
│   └── multasDistribuidasBensTuteladosTO.csv
│
└── Modelo/                             # Data Warehouse (saída)
    ├── DW_DTempo.csv                   # Dimensão Tempo
    ├── DW_DInfrator.csv                # Dimensão Infrator
    ├── DW_DLocal.csv                   # Dimensão Local
    ├── DW_DInfracao.csv                # Dimensão Infração
    ├── DW_DDebito.csv                  # Dimensão Débito
    └── DW_FAutoInfracao.csv            # Tabela Fato
```

---

## 🛠️ Tecnologias Utilizadas

- **Python 3.8+**
- **Pandas** - Manipulação e transformação de dados
- **Jupyter Notebook** - Ambiente de desenvolvimento interativo
- **NumPy** - Operações numéricas

---

## 📦 Instalação

### Pré-requisitos

- Python 3.8 ou superior
- pip (gerenciador de pacotes Python)

### Passos

1. **Clone o repositório**
```bash
git clone https://github.com/seu-usuario/etl-multas-ambientais-brasil.git
cd etl-multas-ambientais-brasil
```

2. **Crie um ambiente virtual (recomendado)**
```bash
python -m venv venv
source venv/bin/activate  # Linux/Mac
# ou
venv\Scripts\activate     # Windows
```

3. **Instale as dependências**
```bash
pip install pandas jupyter numpy
```

4. **Inicie o Jupyter Notebook**
```bash
jupyter notebook
```

5. **Abra o arquivo `ETL.ipynb` e execute as células sequencialmente**

---

## 💻 Como Usar

### Execução Completa do ETL

1. **Carregamento dos Dados**
   - Execute as células iniciais para carregar os 27 arquivos CSV
   - Total esperado: 707.247 registros

2. **Correção de Dados**
   - Correção automática de datas inválidas
   - Normalização de formatos

3. **Criação das Dimensões**
   - DTempo, DInfrator, DLocal, DInfracao, DDebito
   - Geração automática de chaves primárias (pk_*)

4. **Criação da Tabela Fato**
   - Merge com todas as dimensões
   - Conversão de moedas para Real
   - Geração de chaves estrangeiras (fk_*)

5. **Exportação**
   - CSVs gerados na pasta `Modelo/`

### Exemplos de Consultas Analíticas

```python
# Total de multas por região
analise_regiao = FAutoInfracao_final.merge(
    DLocal, left_on='fk_local', right_on='pk_local'
).groupby('regiao').agg({
    'pk_fato': 'count',
    'valorPadrao': 'sum'
})
```

```python
# Multas por ano
analise_ano = FAutoInfracao_final.merge(
    DTempo, left_on='fk_tempo', right_on='pk_tempo'
).groupby('ano').agg({
    'pk_fato': 'count',
    'valorPadrao': 'sum'
})
```

```python
# Top 5 tipos de infração
analise_infracao = FAutoInfracao_final.merge(
    DInfracao, left_on='fk_infracao', right_on='pk_infracao'
).groupby('tipoInfracao').agg({
    'pk_fato': 'count',
    'valorPadrao': 'sum'
}).sort_values('count', ascending=False).head(5)
```

---

## 📊 Estatísticas do Projeto

### Volume de Dados

- **📄 Arquivos processados**: 27 CSVs (um por estado)
- **📝 Total de registros**: 707.247 multas
- **📅 Período coberto**: 1977-2025 (48 anos)
- **💰 Valor total normalizado**: R$ 99+ bilhões
- **🌍 Cobertura geográfica**: 27 estados + 5.185 municípios

### Qualidade dos Dados

- **✅ Integridade referencial**: 100%
- **✅ Datas corrigidas**: 3 registros (0,0004%)
- **✅ Valores convertidos**: 100%
- **✅ Registros sem FK NULL**: 0

---

## 🎓 Conceitos Aplicados

### Data Warehousing
- ✅ Modelagem Dimensional (Star Schema)
- ✅ Tabelas de Dimensão e Fato
- ✅ Surrogate Keys (pk_*)
- ✅ Foreign Keys (fk_*)
- ✅ Slowly Changing Dimensions (Tipo 1)

### ETL (Extract, Transform, Load)
- ✅ Extração de múltiplas fontes
- ✅ Limpeza e validação de dados
- ✅ Transformações complexas (moedas, datas)
- ✅ Carga em modelo dimensional

### Boas Práticas
- ✅ Nomenclatura padronizada e semântica
- ✅ Documentação inline
- ✅ Validação de integridade
- ✅ Logging e rastreabilidade

---

## 📚 Referências

- [Banco Central do Brasil - Histórico de Moedas](https://www.bcb.gov.br/)
- [IBAMA - Sistema de Multas Ambientais](https://www.ibama.gov.br/)
- [Kimball, R. - The Data Warehouse Toolkit](https://www.kimballgroup.com/)

