# Backend - Springboot-API-REST (Ventas)

[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.4.4-brightgreen?logo=springboot)](https://spring.io/projects/spring-boot) [![Java](https://img.shields.io/badge/Java-17-orange?logo=java)](https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html)

## 🎯 Propósito

Este servicio expone la API REST de ventas. Su responsabilidad principal es manejar la CRUD de ventas y permitir que el frontend consulte y actualice estados de venta.

## 📁 Estructura del proyecto

```
back-Ventas_SpringBoot/Springboot-API-REST/
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src/
    ├── main/java/com/citt/
    │   ├── SpringbootApiRestApplication.java
    │   ├── controller/
    │   │   └── VentaController.java
    │   ├── exceptions/
    │   │   └── RestResponseEntityExceptionHandler.java
    │   └── persistence/
    │       ├── entity/
    │       ├── repository/
    │       └── services/
    └── main/resources/
        ├── application.properties
        └── application-test.properties
```

## 🔌 Endpoints principales

| Método | Ruta | Descripción |
|---|---|---|
| POST | `/api/v1/ventas` | Crear nueva venta |
| PUT | `/api/v1/ventas/{idVenta}` | Actualizar venta |
| GET | `/api/v1/ventas` | Listar ventas |
| GET | `/api/v1/ventas/{idVenta}` | Obtener venta específica |
| DELETE | `/api/v1/ventas/{idVenta}` | Eliminar venta |

## 🛠️ Dependencias clave detectadas

- `spring-boot-starter-web`
- `spring-boot-starter-validation`
- `spring-boot-starter-data-jpa`
- `spring-boot-starter-test`
- `springdoc-openapi-starter-webmvc-ui`
- `mysql-connector-j`
- `org.projectlombok:lombok`

## 🌐 Configuración detectada

### `src/main/resources/application.properties`

```properties
spring.application.name=Springboot-API-REST
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

### `src/main/resources/application-test.properties`

- Usa H2 en memoria para pruebas.
- `spring.jpa.hibernate.ddl-auto=create-drop`.
- Consola H2 habilitada.

## 🚀 Cómo ejecutar

```powershell
cd "c:\Users\dell\Downloads\proyecto semestral\back-Ventas_SpringBoot\Springboot-API-REST"
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
| `DB_ENDPOINT` | Host de la DB | `localhost` |
| `DB_PORT` | Puerto MySQL | `3306` |
| `DB_NAME` | Nombre de la base | `ventasdb` |
| `DB_USERNAME` | Usuario DB | `root` |
| `DB_PASSWORD` | Contraseña DB | `secret` |

## 🔧 Consideraciones DevOps

- No se detectó `Dockerfile` en este módulo.
- No se detectó pipeline de GitHub Actions.
- Para producción en AWS EC2, se recomienda empaquetar con `./mvnw clean package` y ejecutar el JAR o contenedor Docker.
- El servicio expone API REST sin autenticación.

## 🧱 Recomendaciones de despliegue en AWS EC2

1. Construir imagen Docker o JAR.
2. Configurar `Security Group` con puerto `8080`.
3. Usar subred pública para backend si se expone directamente.
4. Mejor opción: usar RDS en subred privada para MySQL y no exponer 3306 públicamente.

## 📌 Nota

Este módulo está preparado para funcionar como microservicio independiente. En un pipeline DevOps, su build y deploy deben ejecutarse como unidades separadas para garantizar escalabilidad y despliegue autónomo.
