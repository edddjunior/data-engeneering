📊 Plataforma de Análise de E-Commerce em Tempo Real
📌 Visão Geral do Projeto
Construção de uma pipeline de engenharia de dados escalável para uma plataforma de e-commerce. A solução processa interações de clientes (cliques, compras, adições ao carrinho), atualizações de estoque e dados históricos de vendas. A arquitetura suporta:

Processamento em tempo real e em lote
Armazenamento em Data Warehouse na nuvem
Análises via dashboards e APIs


💼 Caso de Uso
✨ Necessidades de Negócio

Monitorar o comportamento do cliente em tempo real para oferecer recomendações personalizadas.
Acompanhar níveis de estoque para evitar rupturas.
Analisar dados históricos de vendas para previsão de demanda.
Fornecer dashboards de inteligência de negócios para stakeholders.

🔗 Fontes de Dados

Eventos de clientes (cliques, compras) via Apache Kafka
Atualizações de estoque via APIs de armazéns
Dados históricos de vendas de bancos de dados transacionais


🧱 Pilha Tecnológica



Camada
Ferramenta



Nuvem
AWS (adaptável para Azure/GCP)


Ingestão
Apache Kafka, AWS S3


Processamento
Apache Spark (batch + streaming), Python


Orquestração
Apache Airflow


Armazenamento
Amazon Redshift (SQL), MongoDB (NoSQL)


Visualização
Tableau, AWS QuickSight


APIs
FastAPI


Contêineres
Docker


Controle de versão
Git



🏗️ Arquitetura do Projeto
1. Ingestão de Dados

Kafka:
eventos_clientes: cliques, compras, adições ao carrinho
atualizacoes_estoque: atualizações via APIs


Histórico:
Dados de vendas (CSV/Parquet) carregados no S3



2. Processamento de Dados
🔄 Streaming (Spark Streaming)

Consome tópicos Kafka
Enriquece com dados do MongoDB (ex.: perfil de usuário)
Calcula métricas em tempo real (ex.: top 5 produtos visualizados)
Grava no Redshift e MongoDB

📊 Batch (Spark Batch)

Lê do S3
Limpeza e transformação de dados
Agregações históricas (vendas por região, categoria)
Grava no Redshift

3. Armazenamento

Amazon Redshift:
vendas(id_pedido, id_produto, quantidade, preco, data)
eventos_clientes(id_usuario, tipo_evento, id_produto, timestamp)
estoque(id_produto, nivel_estoque, ultima_atualizacao)


MongoDB:
Perfis de usuários
Logs de cliques



4. Orquestração (Airflow)

DAGs para:
ETL diário (S3 → Spark → Redshift)
Atualizações de streaming por hora
Verificações de qualidade de dados



5. Visualização

Dashboards via Tableau / QuickSight
Métricas em tempo real (ex.: usuários ativos, vendas por minuto)
Tendências históricas (ex.: receita mensal, produtos mais vendidos)
Alertas de estoque



6. APIs com FastAPI

Endpoints:
/vendas/resumo: resumo de vendas por produto
/produtos/mais_vendidos: ranking dos mais vendidos



7. Implantação

Serviços conteinerizados com Docker
Deploy via ECS ou EKS (AWS)
Monitoramento com Airflow Logs + CloudWatch


⚙️ Etapas de Implementação
1. Configuração do Ambiente

Inicializar repositório Git
Docker Compose para:
Kafka
Spark
Airflow
FastAPI


Provisionar:
AWS S3
Amazon Redshift
ECS ou EKS



2. Ingestão
# Simulador de eventos (Kafka Producer)
from kafka import KafkaProducer
import json

producer = KafkaProducer(bootstrap_servers='localhost:9092',
                         value_serializer=lambda v: json.dumps(v).encode('utf-8'))

evento = {
    'id_usuario': 'u123',
    'tipo_evento': 'clique',
    'id_produto': 'p456',
    'timestamp': '2025-05-14T19:16:00'
}
producer.send('eventos_clientes', evento)
producer.flush()

3. Processamento em Tempo Real
# Spark Streaming (leitura do Kafka + agregação)
from pyspark.sql import SparkSession
from pyspark.sql.functions import from_json, col
from pyspark.sql.types import StructType, StructField, StringType, TimestampType

spark = SparkSession.builder.appName("ECommerceStreaming").getOrCreate()

schema = StructType([
    StructField("id_usuario", StringType()),
    StructField("tipo_evento", StringType()),
    StructField("id_produto", StringType()),
    StructField("timestamp", TimestampType())
])

df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "eventos_clientes") \
    .load()

eventos = df.select(from_json(col("value").cast("string"), schema).alias("data")).select("data.*")

produtos_mais_vistos = eventos.groupBy("id_produto").count()

query = produtos_mais_vistos.writeStream \
    .format("jdbc") \
    .option("url", "jdbc:redshift://<endpoint>/<banco>") \
    .option("dbtable", "produtos_mais_vistos") \
    .option("user", "<usuario>") \
    .option("password", "<senha>") \
    .start()

4. Orquestração com Airflow
# DAG Airflow para job de lote
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

def executar_spark_lote():
    # Execução de script Spark em lote
    pass

with DAG('pipeline_ecommerce', start_date=datetime(2025, 5, 14), schedule_interval='@daily') as dag:
    tarefa_spark = PythonOperator(
        task_id='spark_etl_lote',
        python_callable=executar_spark_lote
    )

5. API com FastAPI
from fastapi import FastAPI
import pandas as pd
import redshift_connector

app = FastAPI()

@app.get("/vendas/resumo")
def obter_resumo_vendas():
    conn = redshift_connector.connect(
        host="<endpoint-redshift>",
        database="<db>",
        user="<usuario>",
        password="<senha>"
    )
    consulta = """
    SELECT id_produto, SUM(quantidade) as total_vendido
    FROM vendas
    GROUP BY id_produto
    """
    df = pd.read_sql(consulta, conn)
    return df.to_dict(orient="records")


📦 Entregáveis
Repositório Git com:

Código dos produtores Kafka
Jobs Spark (streaming + batch)
DAGs Airflow
Código FastAPI
docker-compose.yml para desenvolvimento local
Schemas do Redshift
Mockups de dashboards (Tableau/QuickSight)
Documentação técnica


🎓 Resultados de Aprendizado

Pipelines de Dados: ETL e ELT em lote e streaming
Nuvem: Serviços AWS (S3, Redshift, ECS/EKS)
Orquestração: Apache Airflow
Streaming: Kafka + Spark Streaming
Armazenamento: Redshift (SQL) + MongoDB (NoSQL)
Visualização: Dashboards interativos e em tempo real
APIs: Endpoints analíticos com FastAPI
Contêineres: Deploy com Docker


🚀 Próximos Passos

Simular dados de entrada (Kafka e S3)
Implantar a solução na AWS
Monitorar e escalar conforme o volume de dados
Otimizar performance e custos
Adicionar modelos de machine learning para previsão de demanda e personalização

