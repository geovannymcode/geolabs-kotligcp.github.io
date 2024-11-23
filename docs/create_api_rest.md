# **Creación de una API REST con Kotlin y Spring Boot**

En esta primera fase, desarrollaremos una API REST desde cero utilizando **Kotlin, Spring Boot, y PostgreSQL**. Nos enfocaremos en el desarrollo local, incluyendo la configuración de la base de datos con **Docker Compose** y la gestión de migraciones con **Flyway**. Despues realizaremos pruebas locales para validar el correcto funcionamiento de los endpoints. En la siguiente fase abordaremos el despliegue en **Google Cloud** utilizando **App Engine y Cloud SQL**.

## **Fase 1: Creación de la API REST y Pruebas Locales**

### **Estructura del Proyecto**

El proyecto está organizado de la siguiente manera:

```bash
src
 ├── main
 │   ├── kotlin
 │   │   └── com.geovannycode
 │   │       ├── 📄 KotlinGcpApplication.kt
 │   │       ├── 📄 Speaker.kt
 │   │       ├── 📄 SpeakerController.kt
 │   │       ├── 📄 SpeakerRepository.kt
 │   │       └── 📄 SpeakerService.kt
 │   └── resources
 │       ├── 📁 db
 │       │   └── migration
 │       │       ├── 📄 V1__Create_Speaker_Table.sql
 │       │       └── 📄 V2__Add_Data_Speaker.sql
 │       ├── 📁 static
 │       ├── 📁 templates
 │       └── 📁 application.properties
```

- **Paquete principal** (`com.geovannycode`): Contiene todas las clases de la aplicación: el punto de entrada (`KotlinGcpApplication`), la entidad (`Speaker`), la lógica de negocio (`SpeakerService`), el repositorio (`SpeakerRepository`) y el controlador REST (`SpeakerController`).
- **Directorio de recursos** (`resources`):
    - `db/migration`: Archivos SQL utilizados por Flyway para la creación y migración de datos.
    - `application.properties`: Configuración de conexión a la base de datos.

### **1. Implementación Técnica**

Punto de Entrada: `KotlinGcpApplication.kt`

```kotlin title="KotlinGcpApplication.kt" linenums="1"
@SpringBootApplication
class KotlinGcpApplication

fun main(args: Array<String>) {
    runApplication<KotlinGcpApplication>(*args)
}
```

Esta clase define el punto de entrada de la aplicación:

- **Línea 1** `@SpringBootApplication`: Configura automáticamente los componentes de Spring (escaneo de clases, configuración de beans, y más).
- **Línea 5** `runApplication`: Inicia la aplicación y el servidor embebido (Tomcat).

### **Modelo de Datos**: `Speaker.kt`

```kotlin title="Speaker.kt" linenums="1"
@Entity
@Table(name = "speakers")
data class Speaker(
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "speaker_id_generator")
    @SequenceGenerator(name = "speaker_id_generator", sequenceName = "speaker_id_seq")
    val id: Long? = null,
    val name: String,
    val country: String
)
```

Este archivo define la estructura de la tabla `speakers` en la base de datos:

- **Línea 1** `@Entity`: Marca la clase como una entidad JPA, que mapea un registro de la tabla.
- **Línea 2** `@Table`: Define el nombre de la tabla como `speakers`.
- **Campos**:
    - `id`: Clave primaria generada automáticamente usando una secuencia PostgreSQL.
    - `name`: Almacena el nombre del speaker.
    - `country`: Almacena el país de origen del speaker.

El uso de una `data class` de Kotlin facilita la inmutabilidad y proporciona métodos como `copy()`.

### **Repositorio**: `SpeakerRepository.kt`

```kotlin title="SpeakerRepository.kt" linenums="1"
@Repository
interface SpeakerRepository : CrudRepository<Speaker, Long>
```

- Este repositorio hereda de `CrudRepository`, proporcionando métodos como:
    - **`findById`**: Busca un registro por su ID.
    - **`findAll`**: Devuelve todos los registros.
    - **`save`**: Guarda o actualiza un registro.
    - **`deleteById`**: Elimina un registro por ID.
- Spring Data JPA se encarga de la implementación del repositorio, permitiéndonos enfocarnos en la lógica de negocio.

### **Servicio**: `SpeakerService.kt`

```kotlin title="SpeakerRepository.kt" linenums="1"
@Service
@Transactional
class SpeakerService(val repo: SpeakerRepository) {
    fun getSpeaker(id: Long) = repo.findById(id)
    fun createSpeaker(speaker: Speaker) = repo.save(speaker)
    fun deleteSpeaker(id: Long) = repo.deleteById(id)
    fun getSpeakers(): List<Speaker> = repo.findAll().toList()
    fun updateSpeaker(id: Long, updatedSpeaker: Speaker): Speaker? {
        return if (repo.existsById(id)) {
            val speakerToUpdate = updatedSpeaker.copy(id = id)
            repo.save(speakerToUpdate)
        } else {
            null
        }
    }
}
```

- **Línea 4** `getSpeakers`: Recupera todos los registros como una lista.
- **Línea 5** `createSpeaker`: Inserta un nuevo speaker en la base de datos.
- **Línea 6** `deleteSpeaker`: Elimina un registro.
- **Línea 7** `getSpeaker`: Busca un speaker por ID.
- **Línea 8** `updateSpeaker`:
    - Comprueba si el ID existe.
    - Usa el método `copy()` para mantener la inmutabilidad de la entidad.
    - Guarda el registro actualizado.

### **Controlador REST**: `SpeakerController.kt`

```kotlin title="SpeakerController.kt" linenums="1"
@RestController
@RequestMapping("/api/speakers")
class SpeakerController(private val service: SpeakerService) {

    @GetMapping
    fun getSpeakers(): ResponseEntity<List<Speaker>> =
        ResponseEntity.ok(service.getSpeakers())

    @GetMapping("/{id}")
    fun getSpeaker(@PathVariable id: Long): ResponseEntity<Speaker> =
        service.getSpeaker(id).map { ResponseEntity.ok(it) }
            .orElse(ResponseEntity.notFound().build())

    @PostMapping
    fun createSpeaker(@RequestBody speaker: Speaker): ResponseEntity<Speaker> =
        ResponseEntity.ok(service.createSpeaker(speaker))

    @DeleteMapping("/{id}")
    fun deleteSpeaker(@PathVariable id: Long): ResponseEntity<Void> {
        service.deleteSpeaker(id)
        return ResponseEntity.noContent().build()
    }

    @PutMapping("/{id}")
    fun updateSpeaker(@PathVariable id: Long, @RequestBody updatedSpeaker: Speaker): ResponseEntity<Speaker> {
        val updated = service.updateSpeaker(id, updatedSpeaker)
        return if (updated != null) {
            ResponseEntity.ok(updated)
        } else {
            ResponseEntity.notFound().build()
        }
    }
}
```

Este archivo expone los endpoints para interactuar con la API:

- **Línea 4** `GET /api/speakers`: Recupera todos los speakers.
- **Línea 4** `POST /api/speakers`: Crea un nuevo registro.
- **Línea 4** `GET /api/speakers/{id}`: Devuelve un speaker por ID.
- **Línea 4** `PUT /api/speakers/{id}`: Actualiza un registro existente.
- **Línea 4** `DELETE /api/speakers/{id}`: Elimina un registro por ID.

## **2. Configuración Local**

### **Base de Datos con Docker Compose**

Archivo `docker-compose.yaml`

```yaml title="docker-compose.yaml" linenums="1"
services:
  postgres_gcp:
    container_name: "postgres_gcp"
    image: 'postgres:16'
    env_file: ./.env
    ports:
      - ${DB_LOCAL_PORT}:${DB_DOCKER_PORT}
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
```

- Asegúrate de configurar las variables en el archivo .env para personalizar los valores.
- Con la incorporacion de Docker Support no hay necesidad de ejecutar el comando de docker para levantar el servicio, con ejecutar la aplicacion automaticamente se levanta el servicio de docker.

## **Migraciones con Flyway**

Flyway asegura la creación y migración de la base de datos.

Archivo `V1__Create_Speaker_Table.sql`

```sql title="V1__Create_Speaker_Table.sql" linenums="1"
CREATE SEQUENCE speaker_id_seq START 1 INCREMENT BY 50;

CREATE TABLE speakers (
    id BIGINT DEFAULT nextval('speaker_id_seq') NOT NULL,
    name VARCHAR(255) NOT NULL,
    country VARCHAR(255) NOT NULL,
    PRIMARY KEY (id)
);
```

Archivo `V2__Add_Data_Speaker.sql`

```sql title="V2__Add_Data_Speaker.sql" linenums="1"
INSERT INTO speakers (id, name, country) VALUES (nextval('speaker_id_seq'), 'Geovanny Mendoza', 'Colombia');
```

## **Pruebas Locales con Postman**

Utiliza Postman para probar los endpoints como se puede mostrar en la `Figura #1`. 

![GeoLabs Kotlin GCP](./files/Imagen02.png "GeoLabs Kotlin GCP")
<p align="center">
  <strong>Figura # 1:</strong> Representación del Postman
</p>


- Ejemplo:
    - `GET /api/speakers`: Recupera todos los speakers.
    - `POST /api/speakers`: Crea un nuevo speaker con este JSON:
        ```json
        {
            "name": "John Doe",
            "country": "USA"
        }
        ```
    - `PUT /api/speakers/{id}`: Actualiza un speaker.
    - `DELETE /api/speakers/{id}`: Elimina un speaker.
