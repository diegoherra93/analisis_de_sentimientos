# Proyecto de Análisis de Sentimiento con Big Data y Deep Learning

Este proyecto realiza un análisis de sentimiento en tiempo real a partir de mensajes de usuarios simulados enviados a través de Kafka. Utiliza HBase para el almacenamiento temporal de mensajes, un modelo de Deep Learning para clasificarlos, y MySQL como data warehouse para su visualización en Metabase.

---

## Instalación de herramientas necesarias

Antes de comenzar, se tiene que instalar lo siguiente:

### Docker Desktop
- Se descarga e instala desde: [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
- Asegúrese de que Docker esté corriendo correctamente.

### Python 3.10+
- Se descarga desde: [https://www.python.org/downloads/](https://www.python.org/downloads/)
- Activar la opción "Add Python to PATH" durante la instalación.

---

## Requisitos

### Software necesario

- **Python 3.10 o superior**
- **Docker y Docker Compose**

### Instalación de Python y Virtualenv (si no están instalados)

```bash
pip install virtualenv
```

---

## 1. Clonar el repositorio (opcional)

```bash
cd <carpeta-del-proyecto>
```

---

## 2. Crear y activar ambiente virtual

```bash
python -m venv venv
# En Windows (CMD o PowerShell)
venv\Scripts\activate
# En Linux/macOS
source venv/bin/activate
```

---

## 3. Instalar dependencias Python

```bash
pip install kafka-python happybase mysql-connector-python torch transformers
```

---

## 4. Levantar los servicios con Docker Compose

Asegúrese de estar en el mismo directorio del archivo `docker-compose.yml`, luego:

```bash
docker compose up -d
```

Este comando iniciará:

- Kafka y Zookeeper
- HBase (con interfaz web en `http://localhost:16010`)
- MySQL (base de datos `sentiment_db` en puerto `3306`)
- Metabase (dashboard accesible en `http://localhost:3000`)

### Parámetros importantes en `docker-compose.yml`

- **`container_name`**: define un nombre para cada contenedor.
- **`ports`**: mapea los puertos del contenedor al host.
- **`volumes`**: permite persistir los datos de la base entre reinicios.
- **`environment`**: define variables de entorno esenciales.

---

## 5. Crear tabla en MySQL

Accede al contenedor de MySQL:

```bash
docker exec -it proyecto_luis_diego_herra-mysql-1 mysql -u root -ppassword
```

Dentro del prompt de MySQL:

```sql
USE sentiment_db;
CREATE TABLE resultados_sentimiento (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id VARCHAR(255),
    comentario TEXT,
    fecha DATE,
    sentimiento VARCHAR(50),
    probabilidad FLOAT,
    timestamp VARCHAR(255)
);
```

---

## 6. Ejecutar los scripts

### a. Consumidor Kafka a HBase

```bash
python consumer_hbase.py
```

### b. Pipeline HBase -> Deep Learning -> MySQL

```bash
python pipeline_deep_learning.py
```

### c. Productores

Ejecutar en terminales separadas:

```bash
python usuario_1.py
```

```bash
python usuario_2.py
```

---

## 7. Acceder a Metabase

- Navegar a `http://localhost:3000`
- Crear una cuenta temporal o usar una existente
- Conectar la base de datos MySQL:
  - **Nombre de la base**: sentiment_db
  - **Host**: host.docker.internal
  - **Puerto**: 3306
  - **Usuario**: root
  - **Contraseña**: password

---

## Estructura del Proyecto

| Archivo                     | Descripción                                                                     |
| --------------------------- | ------------------------------------------------------------------------------- |
| `producer.py`               | Envía tweets aleatorios a Kafka. Implementado con KafkaProducer y envía datos JSON a un tópico Kafka.                                                |
| `usuario_1.py`              | Igual que `producer.py` pero con usuario fijo `user_1`.                         |
| `usuario_2.py`              | Igual que `producer.py` pero con usuario fijo `user_2`.                         |
| `consumer_hbase.py`         | Consume mensajes desde Kafka y los almacena en HBase.                           |
| `pipeline_deep_learning.py` | Procesa los tweets desde HBase con un modelo de Hugging Face y los guarda en MySQL. |
| `docker-compose.yml`        | Ejecuta los servicios (Kafka, HBase, MySQL, Metabase).                         |

---

## Explicación de cada componente

- **`producer.py`**, **`usuario_1.py`**, **`usuario_2.py`**:
  - Simulan tweets multilingües enviados a Kafka con campos: `id`, `usuario`, `mensaje`, `fecha`, `hora`.
  - Implementados con `KafkaProducer` de `kafka-python`.

- **`consumer_hbase.py`**:
  - Se conecta a `tweets` como consumidor Kafka, toma cada mensaje y lo guarda en la base de datos NoSQL HBase bajo una tabla `tweets`. Usa `happybase` y se conecta al contenedor de HBase. Utiliza como clave combinaciones de id, usuario y una marca de tiempo.
  - Estructura de columnas tipo `cf:<campo>`.

- **`pipeline_deep_learning.py`**:
  - Lee mensajes desde HBase aún no procesados.
  - Usa `AutoTokenizer` y `AutoModelForSequenceClassification` para análisis de sentimiento.
  - Guarda los resultados en MySQL y marca los mensajes en HBase como procesados.

- **`docker-compose.yml`**:
  - Kafka + Zookeeper → Mensajería de alto rendimiento.
  - HBase → Almacenamiento NoSQL en columnas.
  - MySQL → Almacenamiento estructurado para visualización.
  - Metabase → Herramienta de BI para mostrar dashboards.
