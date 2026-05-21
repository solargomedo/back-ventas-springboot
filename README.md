# Backend Innovatech - Ventas y Despachos

Repositorio backend compuesto por dos microservicios Spring Boot:

- `Springboot-API-REST`: API de ventas, expuesta en el puerto `8080`.
- `back-Despachos_SpringBoot/Springboot-API-REST-DESPACHO`: API de despachos, expuesta en el puerto `8081`.

Ambos servicios usan MySQL y se despliegan en contenedores Docker.

## Estructura DevOps

```text
back-ventas-springboot/
|-- Springboot-API-REST/
|   |-- Dockerfile
|   |-- .dockerignore
|   |-- pom.xml
|   `-- src/
|-- back-Despachos_SpringBoot/
|   `-- Springboot-API-REST-DESPACHO/
|       |-- Dockerfile
|       |-- .dockerignore
|       |-- pom.xml
|       `-- src/
|-- docker-compose.yml
|-- docker-compose.prod.yml
|-- .env.example
`-- .github/workflows/deploy.yml
```

## Variables

Para ejecución local se puede copiar `.env.example` a `.env`:

```env
MYSQL_ROOT_PASSWORD=rootpass
DB_NAME=innovatechdb
MYSQL_PORT=3306
VENTAS_PORT=8080
DESPACHOS_PORT=8081
```

Las aplicaciones Spring Boot reciben estas variables:

| Variable | Descripción |
|---|---|
| `DB_ENDPOINT` | Host de MySQL. En Compose local es `mysql`. |
| `DB_PORT` | Puerto interno de MySQL, normalmente `3306`. |
| `DB_NAME` | Nombre de la base de datos. |
| `DB_USERNAME` | Usuario de base de datos. |
| `DB_PASSWORD` | Password de base de datos. |

## Ejecución Local Con Docker

Desde la raíz del repositorio:

```powershell
docker compose up --build -d
```

Servicios publicados:

| Servicio | URL local |
|---|---|
| Ventas | `http://localhost:8080/api/v1/ventas` |
| Despachos | `http://localhost:8081/api/v1/despachos` |
| Swagger ventas | `http://localhost:8080/swagger-ui.html` |
| Swagger despachos | `http://localhost:8081/swagger-ui.html` |

Ver logs:

```powershell
docker compose logs -f
```

Detener sin borrar datos:

```powershell
docker compose down
```

Detener y borrar persistencia:

```powershell
docker compose down -v
```

## Persistencia

El servicio `mysql` usa un volumen nombrado:

```yaml
volumes:
  - mysql-data:/var/lib/mysql
```

Se eligió un volumen nombrado porque Docker administra su ubicación, evita depender de rutas específicas del sistema operativo y conserva la información aunque los contenedores sean recreados. Esto cumple la persistencia requerida para que ventas y despachos no se pierdan tras reinicios.

## Dockerfiles

Cada microservicio usa un Dockerfile multi-stage:

1. Etapa `build`: usa Maven con Java 17 para compilar el proyecto y generar el `.jar`.
2. Etapa final: usa solo JRE Java 17 Alpine, copia el `.jar` y ejecuta con un usuario no root.

Esto reduce el tamaño de imagen final y evita ejecutar la aplicación como `root`.

## CI/CD

El workflow `.github/workflows/deploy.yml` se activa con push a la rama `deploy`.

Flujo:

1. Descarga el repositorio.
2. Inicia Docker Buildx.
3. Autentica en Docker Hub.
4. Construye y publica:
   - `${DOCKERHUB_USERNAME}/ventas-api:${GITHUB_SHA}`
   - `${DOCKERHUB_USERNAME}/despachos-api:${GITHUB_SHA}`
   - tags `latest`
5. Copia `docker-compose.prod.yml` a la instancia EC2 backend.
6. Crea un `.env` remoto con secrets.
7. Ejecuta `docker compose pull` y `docker compose up -d`.

## GitHub Secrets Requeridos

| Secret | Uso |
|---|---|
| `DOCKERHUB_USERNAME` | Usuario del registro Docker Hub. |
| `DOCKERHUB_TOKEN` | Token de acceso Docker Hub. |
| `BACKEND_EC2_HOST` | IP pública o DNS de la instancia EC2 backend. |
| `EC2_USER` | Usuario SSH de EC2, por ejemplo `ubuntu` o `ec2-user`. |
| `EC2_SSH_KEY` | Llave privada SSH para acceder a EC2. |
| `MYSQL_ROOT_PASSWORD` | Password root de MySQL en producción. |
| `DB_NAME` | Nombre de base de datos. |
| `VENTAS_PORT` | Puerto publicado para ventas, normalmente `8080`. |
| `DESPACHOS_PORT` | Puerto publicado para despachos, normalmente `8081`. |

## Pruebas

Ventas:

```powershell
cd Springboot-API-REST
.\mvnw.cmd test
```

Despachos:

```powershell
cd back-Despachos_SpringBoot\Springboot-API-REST-DESPACHO
.\mvnw.cmd test
```

Los tests usan perfil `test` con H2 para no depender de MySQL real.
