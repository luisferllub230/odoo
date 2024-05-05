# odoo

Instalación de Odoo en un contenedor Docker
===========================================

Resumen
-------

Este documento detalla los pasos para instalar Odoo en un contenedor Docker, incluyendo la configuración de un motor de base de datos PostgreSQL.

Requisitos previos
------------------

-   Docker instalado y en ejecución.
-   Conocimiento básico de Docker Compose.

Pasos
-----

### 1\. Configuración del motor de base de datos PostgreSQL

Cree un archivo `docker-compose.yml` con la siguiente configuración:

YAML

```
version: '3.4'
services:
  postgres:
    image: postgres:14.9
    restart: always
    container_name: postgres_v14
    environment:
      - POSTGRES_USER=odoo
      - POSTGRES_PASSWORD=odoo
      - POSTGRES_DB=postgres
      - PGDATA=/var/lib/postgresql/data/pgdata
    ports:
      - 5432:5432
    volumes:
      - pg1409:/var/lib/postgresql
      - pg1409_data:/var/lib/postgresql/data
volumes:
  pg1409:
  pg1409_data:

```


Este archivo crea un contenedor llamado `postgres` que ejecuta la imagen de PostgreSQL 14.9. El contenedor se reinicia automáticamente si se detiene y expone el puerto 5432 para conexiones externas.

### 2\. (Opcional) Instalación de un cliente de base de datos

Se recomienda instalar un cliente de base de datos como pgAdmin, pgAdmin Docker o DataGrip de JB para realizar consultas en la base de datos.

### 3\. Configuración del contenedor de Odoo

Cree un archivo `docker-compose.yml` adicional con la siguiente configuración:

YAML

```
version: '3.1'
services:
  odoo16:
    container_name: ${ODOO_DEVELOPER}_odoo
    image: odoo
    networks:
      - odoo-network
    ports:
      - "${ODOO_PORT}:8069"
      - "${ODOO_DEBUG_PORT}:3001"
    environment:
      - DB_PORT_5432_TCP_ADDR=${DB_PORT_5432_TCP_ADDR}
      - DB_PORT_5432_TCP_PORT=${DB_PORT_5432_TCP_PORT}
      - DB_ENV_POSTGRES_USER=${DB_ENV_POSTGRES_USER}
      - DB_ENV_POSTGRES_PASSWORD=${DB_ENV_POSTGRES_PASSWORD}
      - ODOO_DEBUG_PORT=${ODOO_DEBUG_PORT}
      - ODOO_AUTO_UPDATE_MODULES=${ODOO_AUTO_UPDATE_MODULES}
      - ODOO_MODULES=${ODOO_MODULES}
    volumes:
      - odoo_web_data:/var/lib/odoo
      - ./conf/:/etc/odoo
      - ./extra-addons:/mnt/extra-addons
      - ./log:/var/log/odoo
volumes:
  odoo_web_data:
networks:
  odoo-network:

```


Este archivo crea un contenedor llamado `odoo16` que ejecuta la imagen de Odoo. El contenedor se conecta a la red `odoo-network` (creada en el paso 1), expone los puertos 8069 y 3001 para la interfaz web y el modo de depuración, respectivamente, y define variables de entorno para la conexión a la base de datos PostgreSQL.

**Nota:** Se recomienda crear un archivo `.env` para almacenar las variables de entorno sensibles, como las credenciales de la base de datos.

Bash

```
# General Configuration paths
ODOO_EXTRA_ADDONS=/mnt/extra-addons
ODOO_PORT=9090
ODOO_DEBUG_PORT=9196

ODOO_DEVELOPER=lfernandez
ODOO_VOLUME=
ODOO_MODULES=
ODOO_AUTO_UPDATE_MODULES=

# PostgreSQL Configuration
DB_PORT_5432_TCP_ADDR=10.0.0.56
DB_PORT_5432_TCP_PORT=5432
DB_ENV_POSTGRES_USER=odoo
DB_ENV_POSTGRES_PASSWORD=odoo

```


### 4\. Ejecución de los contenedores

Ejecute los siguientes comandos para iniciar los contenedores:

Bash

```
docker-compose up -d

```


Esto iniciará tanto el contenedor de PostgreSQL como el contenedor de Odoo. Una vez que los contenedores estén
