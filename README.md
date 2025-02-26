
# 📊 Netflix Data Pipeline
Este repositório contém um pipeline de dados altamente escalável, automatizado e governado, projetado para garantir ingestão, processamento e transformação eficiente dos dados da Netflix. Ele combina tecnologias modernas como Azure Data Factory, Databricks, Delta Live Tables (DLT) e Unity Catalog, proporcionando um ambiente de alta performance, confiabilidade e governança de dados.

🚀 Destaques e Benefícios do Projeto

- ✅ Governança de Dados Centralizada: Utiliza Unity Catalog para controle granular de permissões.
  
- ✅ Data Quality & Observability: Delta Live Tables (DLT) assegura a qualidade dos dados por meio de regras de validação embutidas e monitoramento contínuo.
  
- ✅ Automação Completa: Jobs no Databricks orquestram todas as etapas do pipeline.
  
- ✅ Escalabilidade e Eficiência: O uso de AutoLoader e Delta Lake otimiza a ingestão e o armazenamento, reduzindo custos e tempo de processamento.
  
- ✅ Flexibilidade e Confiabilidade: O pipeline suporta processamento batch e streaming, permitindo ingestão em tempo quase real.

## 📌 Arquitetura do Projeto
O pipeline segue a arquitetura **Bronze - Silver - Gold**, utilizando ferramentas modernas como **Azure Data Factory (ADF)** para ingestão, **AutoLoader para processamento incremental**, **Jobs no Databricks** para orquestração e **DLT para automação e controle de qualidade**.

![image](https://github.com/user-attachments/assets/82981dad-da4d-4b56-b2f9-b63ecfebab4b)

### 🔄 Fluxo de Dados

1. **Azure Data Factory (ADF)** coleta dados do GitHub e os armazena no **Azure Data Lake Gen2**.
2. **AutoLoader no Databricks** lê e processa dados novos de forma incremental para a camada **Bronze**.
3. **Transformação na camada Silver**: limpeza, padronização e enriquecimento dos dados.
4. **Delta Live Tables (DLT)** estrutura e valida os dados na camada **Gold**, garantindo qualidade e conformidade.
5. **Unity Catalog** gerencia as tabelas e os acessos para controle e governança de dados centralizada.
6. **Os dados processados são disponibilizados para análise** em **Power BI** e **Azure Synapse Analytics**.

---

## 🔄 **Integração com Azure Data Factory**

- **Ingestão:** O ADF busca arquivos CSV do GitHub e carrega para o **Data Lake Gen2**.
- **Orquestração:** Dispara os Jobs do Databricks para processar os dados.
- **Monitoramento:** Configurado para alertas de erro via email.
- **Automação do pipeline**, garantindo execução otimizada.

---
## 🏗️ **Unity Catalog e Localizações Externas**

- O **Unity Catalog** centraliza a governança de dados e fornece controle unificado de acessos.
- Todas as tabelas são gerenciadas dentro do metastore `netflix_unity_metastore`.
- As localizações externas foram configuradas para armazenar os dados no **Azure Data Lake Gen2**, garantindo segurança e rastreabilidade.

![image](https://github.com/user-attachments/assets/da428e91-2694-4991-ac8d-82378e3e628d)


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

### 3️⃣ **Camada Gold - Delta Live Tables (DLT)**

![image](https://github.com/user-attachments/assets/c810bd87-00bf-47a6-bc81-3e2ff1652239)

Arquivo: `7_DLT_Notebook.ipynb`

- **DLT transforma a camada Silver na Gold, aplicando validações e garantindo Data Quality.**
- **Regras de qualidade são aplicadas automaticamente**, rejeitando dados inválidos e garantindo consistência.
- **Escalabilidade automática** para grandes volumes de dados.
- **Governança de dados** com rastreamento de mudanças.


O uso de Delta Live Tables (DLT) torna este pipeline altamente eficiente, garantindo qualidade de dados desde a ingestão até a camada de consumo.

#### **Vantagens do Delta Live Tables (DLT):**
- ✅ **Automação total do pipeline de dados** – não precisa gerenciar tarefas manualmente.
- ✅ **Verificação e validação de qualidade embutida** com `@dlt.expect_all_or_drop()`.
- ✅ **Histórico completo das alterações** – facilita auditorias e conformidade.
- ✅ **Execução otimizada e escalável**, reduzindo custos operacionais.


A escalabilidade e o processamento otimizado do Databricks reduzem o tempo de execução e otimizam os custos de armazenamento e computação.Com rastreabilidade total das transformações, o pipeline mantém um histórico completo das mudanças nos dados, facilitando auditorias e análises.









