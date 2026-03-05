# Modern TIG Stack for InfluxDB 3

Con estas 3 tecnologias seremos capaces de recolectar, almacenar y analizar informacion en tiempo real para casi cualquier servicio, API, dispositivos IoT, etc.

1. **Telegraf** Recolecta la metricas del sistema donde esta ejecutandose y las escribe en la base de datos influxdb.
2. **InfluxDB 3** (Core o Enterprise) es una base de datos para series de tiempos.
3. **Grafana** Es una herramienta de visualizacion de datos (Dashboard) que grafica las metricas de las bases de datos como influxdb desde sus tablas.

## Pre-requisitos:

1. Docker instalado en la maquina. y el plugin de dockewr compose [docker-compose.yaml](docker-compose.yml) con este archivo configuraremos e instalaremos los 3 contenedores a utilizar, Telegraf, InfluxDB 3 Core/Enteprise y Grafana.
2. Git para control de versiones (opcional)
3. Un editor (vscode)

# Pasos:

## 1. Clonar el repositorio

```sh
git clone https://github.com/InfluxCommunity/TIG-Stack-using-InfluxDB-3.git
cd TIG-Stack-using-InfluxDB-3
```

## 2. Iniciar InfluxDB 3

InfluxDB 3 Core & Generar el Token

```sh
docker compose up -d influxdb3-core
docker compose exec influxdb3-core influxdb3 create token --admin
```

## 3. Update .env file

Open [.env](.env) file and paste the token string for "INFLUXDB_TOKEN" enviornment variable.

## 4. Start the remaining services of the TIG Stack in Docker

**First make sure Docker application is up and running on your local machine.
**

```sh
docker compose up -d telegraf
docker compose up -d grafana
```

## 5. Verify the Stack

Check Telegraf Logs

```sh
docker compose logs telegraf
```

Check InfluxDB 3 Logs & See Telegraf generated Tables

```sh
docker compose logs influxdb3-core
docker compose exec influxdb3-core influxdb3 query "SHOW TABLES" --database local_system --token REPLACE_WITH_YOUR_TOKEN_STRING
```

## 6. Setup & View Grafana Dashboard

- Open localhost:3000 from your browser
- Login with credentials from .env (default: admin/admin)
- Add Data Source :
  - Type: InfluxDB
  - Language : SQL
  - Database: Paste the string value for INFLUXDB_BUCKET enviornment variable from your .env file
  - URL: http://influxdb3-core:8181 for connecting to InfluxDB 3 Core
  - URL: http://influxdb3-enterprise:8181 for connecting to InfluxDB 3 Enterprise
  - Dataabse: Find this inside the .env file (bucker name is same as database name)
  - Token: Paste the string value for INFLUXDB_TOKEN enviornment variable from your .env file and toggle **Insecure Connection to ON**
- Add Data Visualization : Dashboards > Create Dashboard - Add Visualization > Select Data Source > InfluxDB_3_Core
- In the query 'builder' paste and run the following SQL query to see the visualization of the data collected via Telegraf, written to InfluxDB 3.

```sql
SELECT "cpu", "usage_user", "time" FROM "cpu" WHERE "time" >= $__timeFrom AND "time" <= $__timeTo AND "cpu" = 'cpu0'
```

![TIG Stack](https://github.com/InfluxCommunity/TIG-Stack-using-InfluxDB-3-Core/blob/main/Grafana_screenshot.png)

## 7. Stopping the TIG Stack & Removing Data

### Stop Services

```sh
docker compose down
```

### Stop and Remove Volumes (Destroys All Data)

```sh
docker compose down -v
```
