
# 📊 Netflix Data Pipeline

![image](https://github.com/user-attachments/assets/82981dad-da4d-4b56-b2f9-b63ecfebab4b)

Este repositório contém um pipeline de dados completo para ingestão, processamento e transformação de informações da Netflix utilizando **Azure Data Factory, Databricks, Delta Live Tables (DLT) e Unity Catalog**.

## 📌 Arquitetura do Projeto

O pipeline segue a arquitetura **Bronze - Silver - Gold**, utilizando ferramentas modernas como **Azure Data Factory (ADF)** para ingestão, **AutoLoader para processamento incremental**, **Jobs no Databricks** para orquestração e **DLT para automação e controle de qualidade**.

📷 **Fluxo Geral do Pipeline:**



### 🔄 Fluxo de Dados

1. **Azure Data Factory (ADF)** coleta dados do GitHub e os armazena no **Azure Data Lake Gen2**.
2. **AutoLoader no Databricks** lê e processa dados novos de forma incremental para a camada **Bronze**.
3. **Transformação na camada Silver**: limpeza, padronização e enriquecimento dos dados.
4. **Delta Live Tables (DLT)** estrutura e valida os dados na camada **Gold**, garantindo qualidade e conformidade.
5. **Unity Catalog** gerencia as tabelas e os acessos para controle e governança de dados centralizada.
6. **Os dados processados são disponibilizados para análise** em **Power BI** e **Azure Synapse Analytics**.

---

## 🔄 **Integração com Azure Data Factory**

Arquivo: `pipeline_adf.json`

- **Ingestão:** O ADF busca arquivos CSV do GitHub e carrega para o **Data Lake Gen2**.
- **Orquestração:** Dispara os Jobs do Databricks para processar os dados.
- **Monitoramento:** Configurado para alertas de erro via email.
- **Automação do pipeline**, garantindo execução otimizada.

---

## 🚀 Notebooks e Processamento de Dados

### 1️⃣ **Camada Bronze - AutoLoader**

Arquivo: `1_autoloader.ipynb`

- O **AutoLoader** faz a ingestão automática e incremental de arquivos CSV.
- Permite escalabilidade para grandes volumes de dados e reduz custos operacionais.
- Detecta automaticamente novos arquivos sem necessidade de monitoramento manual.
- Os dados são armazenados na camada **Bronze** no Data Lake Gen2.

```python
checkpoint_location = "abfss://container@storageaccountl.dfs.core.windows.net/checkpoint"
df = spark.readStream\
    .format('cloudFiles')\
    .option('cloudFiles.format', 'csv')\
    .option('cloudFiles.schemaLocation', checkpoint_location)\
    .load('abfss://raw@storageaccountl.dfs.core.windows.net')
```

### 2️⃣ **Camada Silver - Transformações**

Arquivo: `2_silver.ipynb`

- Realiza limpeza, tratamento de valores nulos e ajuste de tipos de dados.
- Criação de colunas derivadas (`Shorttitle`, `type_flag`, etc.).
- Armazena os dados refinados na camada **Silver** no formato **Delta**.

```python
df = df.withColumn('Shorttitle',split(col('title'),':')[0])
df = df.withColumn('type_flag',when(col('type') == 'Movie',1)\
        .when(col('type') == 'TV Show',2).otherwise(0))
df.write.format('delta')\
     .mode('overwrite')\
     .option('path', 'abfss://container@storageaccountl.dfs.core.windows.net/silver/netflix_titles')\
     .save()
```

### 3️⃣ **Camada Gold - Delta Live Tables (DLT)**

![image](https://github.com/user-attachments/assets/c810bd87-00bf-47a6-bc81-3e2ff1652239)


Arquivo: `7_DLT_Notebook.ipynb`

- **DLT transforma a camada Silver na Gold, aplicando validações e garantindo Data Quality.**
- **Regras de qualidade são aplicadas automaticamente**, rejeitando dados inválidos e garantindo consistência.
- **Escalabilidade automática** para grandes volumes de dados.
- **Governança de dados** com rastreamento de mudanças.

#### **Vantagens do Delta Live Tables (DLT):**
- ✅ **Automação total do pipeline de dados** – não precisa gerenciar tarefas manualmente.
- ✅ **Verificação e validação de qualidade embutida** com `@dlt.expect_all_or_drop()`.
- ✅ **Histórico completo das alterações** – facilita auditorias e conformidade.
- ✅ **Execução otimizada e escalável**, reduzindo custos operacionais.

```python
@dlt.table(
  name = "gold_netflixdirectors"
)
@dlt.expect_all_or_drop({'rule1': 'show_id is NOT NULL'})
def gold_netflixdirectors():
    df = spark.readStream.format('delta').load('abfss://container@storageaccountl.dfs.core.windows.net/silver/netflix_directors')
    return df
```

---

## 🏗️ **Unity Catalog e Localizações Externas**

- O **Unity Catalog** centraliza a governança de dados e fornece controle unificado de acessos.
- Todas as tabelas são gerenciadas dentro do metastore `netflix_unity_metastore`.
- As localizações externas foram configuradas para armazenar os dados no **Azure Data Lake Gen2**, garantindo segurança e rastreabilidade.

📷 **Print do Unity Catalog:**

![image](https://github.com/user-attachments/assets/da428e91-2694-4991-ac8d-82378e3e628d)


---

## 🏗️ **Jobs do Databricks**

Os **Jobs** no Databricks garantem a automação do pipeline de dados.

### 🔹 **Job 1 - Processamento Silver** (`job_silver.json`)
- Executa `3_lookupnotebook.ipynb` para buscar metadados.
- Usa `2_silver.ipynb` para processar diferentes tabelas **Silver** com parâmetros dinâmicos.
- Executa todas as tabelas do array `my_arr`.
- ![image](https://github.com/user-attachments/assets/e1e5c00b-5880-4568-872f-48517bd75789)

  

### 🔹 **Job 2 - Verificação Condicional**
- Executa `5_lookupNotebook.ipynb` para verificar a data.
- Dependendo do dia da semana, decide qual notebook executar (**4_Silver.ipynb** ou **6_false_notebook.ipynb**).
- ![image](https://github.com/user-attachments/assets/39d092bb-1fdc-471e-a4e9-2ade7e638d86)


---


