# Performance Testing Sandbox: Docker + InfluxDB + Grafana + JMeter

Este repositorio contiene una infraestructura completa como código utilizando **Docker Compose** para levantar un entorno de monitoreo de pruebas de carga en tiempo real con **InfluxDB (v2.x)** y **Grafana**, junto con un script de prueba parametrizado en **JMeter**.

## 🚀 Arquitectura del Entorno

El entorno levanta de forma local los siguientes servicios intercomunicados:
*   **InfluxDB (v2.7):** Base de datos de series temporales configurada para recibir las métricas de JMeter en tiempo real.
*   **Grafana (v11.4.0):** Dashboard preconfigurado para visualizar el rendimiento de las pruebas (RPS, Latencia, Errores).

---

## 🛠️ Requisitos Previos

Antes de empezar, asegúrate de tener instalado:
*   [Docker & Docker Compose](https://www.docker.com/)
*   [Apache JMeter](https://jmeter.apache.org/) (Se recomienda v5.6.3 o superior)

---

## 💻 Guía de Uso

### 1. Configurar las Variables de Entorno

Crea un archivo `.env` en la raíz del proyecto basado en el archivo de ejemplo:

```bash
cp .env.example .env
```

Edita el archivo `.env` con tus propios valores para usuario, contraseña, organización, bucket y token de InfluxDB.

### 2. Levantar el Stack de Monitoreo

```bash
docker compose up -d
```

Esto levanta InfluxDB y Grafana. El health check garantiza que InfluxDB esté listo antes de que Grafana inicie.

### 3. Acceder a los Servicios

| Servicio  | URL                        | Credenciales                     |
|-----------|----------------------------|----------------------------------|
| Grafana   | http://localhost:3000      | admin / admin (primer acceso)    |
| InfluxDB  | http://localhost:8086      | Las definidas en tu `.env`       |

### 4. Ejecutar Prueba de Carga en Local

```bash
jmeter -n \
  -t "./performance-tests/backend-jmeter/SC_apiPrueba.jmx" \
  -l "results.jtl" \
  -e -o "report/" \
  -JV_threads=10 \
  -JV_rampUp=10 \
  -JV_duration=15 \
  -JINFLUX_HOST="localhost" \
  -JINFLUX_TOKEN="<tu-token>"
```

| Parámetro      | Descripción                              | Default |
|----------------|------------------------------------------|---------|
| `V_threads`    | Número de usuarios concurrentes          | 10      |
| `V_rampUp`     | Tiempo de rampa en segundos              | 10      |
| `V_duration`   | Duración total de la prueba en segundos  | 15      |
| `INFLUX_HOST`  | Host de InfluxDB                         | localhost |
| `INFLUX_TOKEN` | Token de autenticación de InfluxDB       | —       |

### 5. Ejecutar vía GitHub Actions

El pipeline se ejecuta manualmente desde la pestaña **Actions** del repositorio:

1. Ir a **Actions** → **Pipeline de Performance Personal (Docker + JMeter)**
2. Click en **Run workflow**
3. Configurar los parámetros (concurrencia, ramp-up, duración, path del JMX)
4. El reporte HTML se descarga como artefacto al finalizar

> El pipeline requiere el secret `INFLUX_TOKEN` configurado en el repositorio.

### 6. Detener el Stack

```bash
docker compose down -v
```

---

## 📁 Estructura del Proyecto

```
.
├── .env.example                            # Variables de entorno (plantilla)
├── .github/workflows/
│   └── perf_test_apiPrueba.yml             # Pipeline de GitHub Actions
├── docker-compose.yml                      # Stack: InfluxDB + Grafana
├── performance-tests/
│   └── backend-jmeter/
│       └── SC_apiPrueba.jmx               # Script de prueba JMeter
└── README.md
```

---

## 🧰 Tecnologías

| Herramienta   | Versión | Propósito                          |
|---------------|---------|-------------------------------------|
| Docker Compose| V2      | Orquestación de contenedores        |
| InfluxDB      | 2.7     | Base de datos de series temporales  |
| Grafana       | 11.4.0  | Visualización de métricas           |
| Apache JMeter | 5.6.3   | Ejecución de pruebas de carga       |
| GitHub Actions| —       | CI/CD para ejecución automatizada   |
