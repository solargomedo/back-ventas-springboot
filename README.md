# Backend Innovatech - Ventas y Despachos

Backend compuesto por dos microservicios Spring Boot separados y orquestados con Docker Compose.

- `ventas-api`: gestiona ventas y ordenes de compra.
- `despachos-api`: gestiona ordenes de despacho.
- `mysql`: base de datos compartida con persistencia mediante volumen Docker.

## Estructura

```text
back-ventas-springboot/
|-- ventas-api/
|   |-- Dockerfile
|   |-- .dockerignore
|   |-- pom.xml
|   |-- mvnw
|   |-- mvnw.cmd
|   `-- src/
|-- despachos-api/
|   |-- Dockerfile
|   |-- .dockerignore
|   |-- pom.xml
|   |-- mvnw
|   |-- mvnw.cmd
|   `-- src/
|-- docker-compose.yml
|-- docker-compose.prod.yml
|-- .env
|-- .env.example
|-- README.md
`-- .github/
    `-- workflows/
        `-- deploy.yml
```

Esta estructura deja ambos microservicios al mismo nivel, sin carpetas contenedoras innecesarias. Cada servicio conserva su propio `pom.xml`, wrapper Maven, `src/` y `Dockerfile`.

## Puertos Locales

| Servicio | Puerto contenedor | Puerto local | URL |
|---|---:|---:|---|
| MySQL | `3306` | `3307` | `localhost:3307` |
| Ventas API | `8080` | `8083` | `http://localhost:8083/api/v1/ventas` |
| Despachos API | `8081` | `8081` | `http://localhost:8081/api/v1/despachos` |

Ventas se publica localmente en `8083` porque el puerto `8080` suele estar ocupado en el equipo local. Dentro del contenedor sigue escuchando en `8080`.

## Variables

Archivo `.env` local:

```env
MYSQL_ROOT_PASSWORD=rootpass
DB_NAME=innovatechdb
MYSQL_PORT=3307
VENTAS_PORT=8083
DESPACHOS_PORT=8081
```

Variables usadas por los microservicios Spring Boot dentro de Compose:

| Variable | Valor local | Descripcion |
|---|---|---|
| `DB_ENDPOINT` | `mysql` | Nombre DNS del contenedor MySQL en la red Docker. |
| `DB_PORT` | `3306` | Puerto interno de MySQL dentro de Docker. |
| `DB_NAME` | `innovatechdb` | Base de datos creada al iniciar MySQL. |
| `DB_USERNAME` | `root` | Usuario de base de datos. |
| `DB_PASSWORD` | `rootpass` | Password definido en `.env`. |

## Ejecucion Con Docker

Desde la raiz del backend:

```powershell
docker compose down
docker compose up --build -d
```

Ver contenedores:

```powershell
docker compose ps
```

Ver logs:

```powershell
docker compose logs -f
```

Detener sin borrar datos:

```powershell
docker compose down
```

Detener y borrar la base persistida:

```powershell
docker compose down -v
```

## Pruebas Rapidas De API

Listar ventas:

```powershell
curl http://localhost:8083/api/v1/ventas
```

Crear una venta:

```powershell
curl -X POST http://localhost:8083/api/v1/ventas `
  -H "Content-Type: application/json" `
  -d "{\"fechaCompra\":\"2026-05-21\",\"direccionCompra\":\"Av Siempre Viva 123\",\"valorCompra\":25000,\"despachoGenerado\":false}"
```

Listar despachos:

```powershell
curl http://localhost:8081/api/v1/despachos
```

Swagger:

```text
http://localhost:8083/swagger-ui.html
http://localhost:8081/swagger-ui.html
```

## Integracion Con Frontend

El frontend debe apuntar a:

```env
VITE_API_VENTAS=http://localhost:8083
VITE_API_DESPACHOS=http://localhost:8081
```

Si se cambian estas variables, el frontend debe reconstruirse porque Vite inserta las variables durante el build.

## Despliegue Actual En AWS

Instancia EC2 backend:

```text
IP publica: 44.198.166.184
DNS publico: ec2-44-198-166-184.compute-1.amazonaws.com
IP privada: 172.31.8.186
```

URLs publicas de verificacion:

```text
Ventas API: http://44.198.166.184:8083/api/v1/ventas
Despachos API: http://44.198.166.184:8081/api/v1/despachos
```

Reglas de entrada requeridas en el Security Group backend:

| Tipo | Puerto | Origen |
|---|---:|---|
| SSH | `22` | `0.0.0.0/0` |
| TCP personalizado | `8083` | `0.0.0.0/0` |
| TCP personalizado | `8081` | `0.0.0.0/0` |

GitHub Secret que debe apuntar a esta instancia:

```text
BACKEND_EC2_HOST=44.198.166.184
```

Si AWS Academy detiene e inicia el laboratorio, la IP publica puede cambiar. En ese caso se debe actualizar `BACKEND_EC2_HOST` y tambien los secrets del frontend `VITE_API_VENTAS` y `VITE_API_DESPACHOS`.

## Persistencia

MySQL usa un volumen nombrado:

```yaml
volumes:
  - mysql-data:/var/lib/mysql
```

Se usa un volumen nombrado porque Docker administra su ubicacion y mantiene los datos aunque los contenedores se reinicien o se recreen. Solo se elimina si se ejecuta `docker compose down -v`.

## Dockerfiles

Cada microservicio usa Dockerfile multi-stage:

1. `maven:3.9.9-eclipse-temurin-17`: compila el proyecto y genera el `.jar`.
2. `eclipse-temurin:17-jre-alpine`: ejecuta solo el `.jar` con un usuario no root.

Esto reduce el peso de la imagen final y evita ejecutar la aplicacion con privilegios de root.

## Pruebas Maven

Ventas:

```powershell
cd ventas-api
.\mvnw.cmd test
```

Despachos:

```powershell
cd despachos-api
.\mvnw.cmd test
```

Los tests usan perfil `test` con H2, por lo que no dependen de MySQL real.

## CI/CD

El workflow `.github/workflows/deploy.yml` se activa con push a la rama `deploy`.

Flujo del pipeline:

1. Descarga el repositorio.
2. Autentica en Docker Hub.
3. Construye y publica:
   - `${DOCKERHUB_USERNAME}/ventas-api:${GITHUB_SHA}`
   - `${DOCKERHUB_USERNAME}/despachos-api:${GITHUB_SHA}`
   - tags `latest`
4. Copia `docker-compose.prod.yml` a la instancia EC2 backend.
5. Crea un `.env` remoto con GitHub Secrets.
6. Ejecuta `docker compose pull`.
7. Ejecuta `docker compose up -d`.

## GitHub Secrets

| Secret | Uso |
|---|---|
| `DOCKERHUB_USERNAME` | Usuario Docker Hub. |
| `DOCKERHUB_TOKEN` | Token Docker Hub. |
| `BACKEND_EC2_HOST` | IP publica o DNS de la instancia EC2 backend. |
| `EC2_USER` | Usuario SSH de EC2. |
| `EC2_SSH_KEY` | Llave privada SSH. |
| `MYSQL_ROOT_PASSWORD` | Password root de MySQL en produccion. |
| `DB_NAME` | Nombre de base de datos. |
| `VENTAS_PORT` | Puerto publicado para ventas en EC2. |
| `DESPACHOS_PORT` | Puerto publicado para despachos en EC2. |

Valores usados actualmente:

```text
BACKEND_EC2_HOST=44.198.166.184
VENTAS_PORT=8083
DESPACHOS_PORT=8081
```

## Estabilidad En EC2

La instancia backend ejecuta MySQL y dos aplicaciones Spring Boot. En instancias pequenas de AWS Academy se limita la memoria de Java desde `docker-compose.prod.yml`:

```yaml
JAVA_TOOL_OPTIONS: "-Xms128m -Xmx256m"
```

Adicionalmente, se recomienda configurar swap de 2 GB en la EC2 backend para evitar bloqueos durante el arranque:

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
free -h
```
