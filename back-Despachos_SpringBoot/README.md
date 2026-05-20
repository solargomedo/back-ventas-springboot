# Backend - Springboot-API-REST-DESPACHO

[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.4.4-brightgreen?logo=springboot)](https://spring.io/projects/spring-boot) [![Java](https://img.shields.io/badge/Java-17-orange?logo=java)](https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html)

## 🎯 Propósito

Este servicio administra la lógica de despachos. Permite crear, listar, actualizar y eliminar órdenes de despacho de forma independiente al servicio de ventas.

## 📁 Estructura del proyecto

```
back-Despachos_SpringBoot/Springboot-API-REST-DESPACHO/
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src/
    ├── main/java/com/citt/
    │   ├── SpringbootApiRestDespachoApplication.java
    │   ├── controller/
    │   │   └── DespachoController.java
    │   ├── exceptions/
    │   │   └── RestResponseEntityExceptionHandler.java
    │   └── persistence/
    │       ├── entity/
    │       ├── repository/
    │       └── services/
    └── main/resources/
        └── application.properties
```

## 🔌 Endpoints principales

| Método | Ruta | Descripción |
|---|---|---|
| POST | `/api/v1/despachos` | Crear un despacho |
| PUT | `/api/v1/despachos/{idDespacho}` | Actualizar despacho |
| GET | `/api/v1/despachos` | Listar despachos |
| GET | `/api/v1/despachos/{idDespacho}` | Obtener despacho por ID |
| DELETE | `/api/v1/despachos/{idDespacho}` | Eliminar despacho |

## 🌐 Configuración detectada

`src/main/resources/application.properties` contiene:

```properties
spring.application.name=Springboot-API-REST
server.port=8081
spring.web.allow-cors=true
spring.datasource.url=jdbc:mysql://${DB_ENDPOINT}:${DB_PORT}/${DB_NAME}?useSSL=false&serverTimezone=UTC&createDatabaseIfNotExist=true
spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
spring.datasource.platform=mysql
spring.jpa.database=mysql
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect
springdoc.swagger-ui.path=/swagger-ui.html
```

## 🚀 Cómo ejecutar

```powershell
cd "c:\Users\dell\Downloads\proyecto semestral\back-Despachos_SpringBoot\Springboot-API-REST-DESPACHO"
./mvnw clean package
./mvnw spring-boot:run
```

## 🧪 Pruebas

```powershell
./mvnw test
```

## ⚠️ Variables de entorno necesarias

| Variable | Uso | Ejemplo |
|---|---|---|
| `DB_ENDPOINT` | Host de MySQL | `localhost` |
| `DB_PORT` | Puerto MySQL | `3306` |
| `DB_NAME` | Nombre de la base | `despachosdb` |
| `DB_USERNAME` | Usuario DB | `root` |
| `DB_PASSWORD` | Contraseña DB | `secret` |

## 🔧 Consideraciones DevOps

- No se detectó `Dockerfile` para este servicio.
- No se encontró pipeline de GitHub Actions.
- Recomendado utilizar AWS EC2 con contenedor Docker o ejecutar JAR directamente.
- El puerto `8081` está fijado en el archivo de configuración.

## 🧱 Recomendaciones de despliegue en AWS EC2

1. Empaquetar con `./mvnw clean package`.
2. Configurar Security Group para permitir tráfico TCP `8081`.
3. Usar subred pública para el servicio si se expone directamente.
4. RDS privado para MySQL y no exponer `3306` a Internet.

## 📌 Nota

Este servicio también forma parte de la arquitectura de microservicios del proyecto. Debe ser desplegado y monitoreado de forma independiente para registrar estados de despacho con alta disponibilidad.
