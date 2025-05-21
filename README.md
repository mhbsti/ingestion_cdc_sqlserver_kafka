# ingestion_cdc_sqlserver_kafka
# Sistema de Captura de Dados em Tempo Real com SQL Server, Debezium e Kafka

Este projeto demonstra como configurar um ambiente para captura de dados em tempo real usando Microsoft SQL Server, Debezium e Kafka. O ambiente é criado usando Docker.

## Pré-requisitos

- Docker e Docker Compose instalados na máquina.
- Conhecimentos básicos de SQL e Docker.

## Estrutura do Projeto

O projeto consiste em um arquivo `docker-compose.yml` que configura os seguintes serviços:

- **SQL Server**: Para armazenar os dados.
- **Zookeeper**: Para gerenciar o Kafka.
- **Kafka**: Para gerenciar a transmissão das mensagens.
- **Debezium**: Para capturar as alterações no SQL Server.

## Passo a Passo

docker-compose up -d

verificar os logs do contêiner usando: docker-compose logs -f sqlserver

-- Habilitar CDC no banco de dados
USE master;
ALTER DATABASE seu_banco_de_dados SET RECOVERY SIMPLE;
USE seu_banco_de_dados;
EXEC sys.sp_cdc_enable_db;

-- Criar uma tabela de exemplo
CREATE TABLE dbo.sua_tabela (
    id INT PRIMARY KEY,
    nome NVARCHAR(100)
);

-- Habilitar CDC para a tabela criada
EXEC sys.sp_cdc_enable_table
    @source_schema = N'dbo',
    @source_name = N'sua_tabela',
    @role_name = NULL;


Use cURL ou Postman para enviar a configuração do conector Debezium:

curl -X POST -H "Content-Type: application/json" \
     --data '{
       "name": "sqlserver-connector",
       "config": {
         "connector.class": "io.debezium.connector.sqlserver.SqlServerConnector",
         "tasks.max": "1",
         "database.hostname": "sqlserver",
         "database.port": "1433",
         "database.user": "sa",
         "database.password": "YourStrong!Passw0rd",
         "database.dbname": "seu_banco_de_dados",
         "database.server.name": "sqlserver",
         "table.whitelist": "dbo.sua_tabela",
         "include.schema.changes": "true",
         "key.converter": "org.apache.kafka.connect.json.JsonConverter",
         "value.converter": "org.apache.kafka.connect.json.JsonConverter",
         "key.converter.schemas.enable": "false",
         "value.converter.schemas.enable": "false",
         "snapshot.mode": "schema_only"
       }
     }' \
     http://localhost:8083/connectors


Para consumir as mensagens do Kafka, execute o seguinte comando:
docker exec -it $(docker ps -qf "name=kafka") kafka-console-consumer --bootstrap-server localhost:9094 --topic sqlserver.dbo.sua_tabela --from-beginning

docker-compose down
