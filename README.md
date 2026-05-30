# Backend Despacho — Innovatech Chile

API REST desarrollada con **Spring Boot y Java 17** para la gestión de despachos de Innovatech Chile. Expone los endpoints necesarios para que el frontend pueda consultar, registrar y administrar los despachos de la empresa.

---

## Tecnologías

- Java 17
- Spring Boot 3.4.4
- Maven
- MySQL 8.0
- Docker

---

## Requisitos previos

- [Docker](https://www.docker.com/) y Docker Compose instalados

---

## Variables de entorno

Copia el archivo `.env.example` como `.env` en la raíz del proyecto y completa los valores:

```bash
cp .env.example .env
```

| Variable | Descripción |
|---|---|
| `MYSQL_ROOT_PASSWORD` | Contraseña del usuario root de MySQL |
| `MYSQL_DATABASE` | Nombre de la base de datos creada al iniciar el contenedor |
| `MYSQL_USER` | Usuario de MySQL para la aplicación |
| `MYSQL_PASSWORD` | Contraseña del usuario de MySQL |
| `DB_NAME` | Nombre de la base de datos usada por Spring Boot |
| `DB_USERNAME` | Usuario que Spring Boot usa para conectarse |
| `DB_PASSWORD` | Contraseña que Spring Boot usa para conectarse |

---

## Correr localmente

```bash
docker-compose up --build
```

La API estará disponible en [http://localhost:8081](http://localhost:8081).

> El servicio `backend` espera a que el contenedor `db` esté saludable antes de iniciar (healthcheck con `mysqladmin ping`).

---

## Estructura Docker

El proyecto usa un **Dockerfile multi-stage**:

- **Stage 1** — `maven:3.9-eclipse-temurin-17` compila el proyecto con `mvn package -DskipTests`
- **Stage 2** — `eclipse-temurin:17-jre-alpine` ejecuta el jar generado con un usuario sin privilegios (`appuser`)

---

## CI/CD — GitHub Actions

El pipeline se encuentra en `.github/workflows/deploy.yml` y se activa automáticamente con cada `push` a la rama `deploy`.

### Flujo del pipeline

```
Push a rama deploy
       │
       ▼
Checkout del código
       │
       ▼
Login a Docker Hub
       │
       ▼
Build y push de imagen → DOCKERHUB_USERNAME/backend-despacho:latest
       │
       ▼
SSH a EC2 privada
       │
       ▼
docker pull + docker stop/rm + docker run (con variables de entorno DB)
```

### Secrets requeridos en GitHub

| Secret | Descripción |
|---|---|
| `DOCKERHUB_USERNAME` | Usuario de Docker Hub |
| `DOCKERHUB_TOKEN` | Token de acceso de Docker Hub |
| `EC2_BACKEND_HOST` | IP de la instancia EC2 del backend |
| `EC2_SSH_KEY` | Clave privada SSH para conectarse a EC2 |
| `DB_ENDPOINT` | Host de la base de datos |
| `DB_PORT` | Puerto de la base de datos |
| `DB_NAME` | Nombre de la base de datos |
| `DB_USERNAME` | Usuario de la base de datos |
| `DB_PASSWORD` | Contraseña de la base de datos |

---

## Infraestructura

La aplicación se despliega en una instancia **AWS EC2 privada**, accesible únicamente desde el frontend a través del puerto `8081`. No está expuesta directamente a internet, lo que garantiza que solo el servicio frontend autorizado pueda consumir la API.
