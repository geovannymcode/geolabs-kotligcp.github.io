# **Fase 2: Creación de la Entidad, Repositorio, Servicio y Controlador**

En esta sección implementaremos los componentes principales para nuestra API REST. Adicionalmente, configuraremos **Flyway** para las migraciones de base de datos y utilizaremos un archivo `compose.yaml` para ejecutar PostgreSQL localmente.

## **Creación de la API REST y Pruebas Locales**

### **Estructura del Proyecto**

El proyecto está organizado en una estructura clara y modular, siguiendo las mejores prácticas de desarrollo con Spring Boot y Kotlin. La siguiente jerarquía de carpetas y archivos refleja cómo se distribuyen las responsabilidades dentro del proyecto:

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

- **Paquete principal** (`com.geovannycode`): Aquí se encuentran las clases esenciales de la API:
    - `KotlinGcpApplication`: Es el punto de entrada de la aplicación. Inicializa Spring Boot y configura el servidor embebido (Tomcat). 
    - `Speaker`: Representa la entidad `Speaker` almacenada en la base de datos. Este modelo mapea los datos relacionales de la tabla PostgreSQL a un objeto Kotlin.
    - `SpeakerRepository`: Define las operaciones necesarias para interactuar con la base de datos. Hereda métodos predefinidos de Spring Data JPA como `findAll`, `save`, `findById`, y `deleteById`.
    - `SpeakerService`: Gestiona la lógica de negocio y las validaciones necesarias antes de interactuar con la base de datos.
    - `SpeakerController`: Maneja las solicitudes HTTP y expone los endpoints REST para interactuar con los servicios de la API.
- **Directorio de recursos** (`resources`):
    - Este directorio contiene configuraciones y scripts utilizados por la aplicación.
        - `db/migration`: Contiene los archivos SQL que Flyway utiliza para crear y modificar el esquema de la base de datos.
            - `V1__Create_Speaker_Table.sql`: Crea la tabla speakers.
            - `V2__Add_Data_Speaker.sql`: Inserta datos iniciales en la tabla para pruebas.
        - `application.properties `:
            - Archivo de configuración con datos de conexión a la base de datos y otras configuraciones de la aplicación.

### **1. Implementación Técnica**

**Punto de Entrada**: `KotlinGcpApplication.kt`

El archivo `KotlinGcpApplication.kt` define el punto de entrada principal de nuestra aplicación. Es aquí donde Spring Boot inicia todo el proceso de configuración, creación de beans y levantamiento del servidor embebido (Tomcat).

```kotlin title="KotlinGcpApplication.kt" linenums="1"
package com.geovannycode

import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

@SpringBootApplication
class KotlinGcpApplication

fun main(args: Array<String>) {
    runApplication<KotlinGcpApplication>(*args)
}
```

Esta clase define el punto de entrada de la aplicación:

- **Línea 2 y 3** Estas líneas importan las funcionalidades necesarias desde las bibliotecas de Spring Boot:
    - `SpringBootApplication`: Anotación que activa la configuración automática de Spring Boot.
    - `runApplication`: Función de utilidad que inicia la aplicación con todas las configuraciones proporcionadas por Spring Boot.
- **Línea 5** `@SpringBootApplication`: 
Esta anotación combina tres anotaciones clave de Spring Boot:
    - `@Configuration`: Marca la clase como una fuente de configuraciones de Spring.
    - `@EnableAutoConfiguration`: Activa la configuración automática de Spring Boot, cargando los componentes según las dependencias declaradas.
    - `@ComponentScan`: Escanea automáticamente el paquete actual y sus subpaquetes en busca de clases anotadas como `@Component`, `@Service`, `@Repository`, etc., para registrarlas como beans en el contexto de la aplicación.
- **Línea 6** `class KotlinGcpApplication`: 
    - Define una clase vacía con el nombre de la aplicación. Aunque no contiene lógica adicional, Spring Boot la utiliza como punto central para identificar el contexto principal de la aplicación.
    - **Nota**: El nombre de la clase es arbitrario, pero debe ser único dentro del paquete y representativo de la funcionalidad de la aplicación.
- **Línea 8**: `fun main(args: Array<String>)`
    - Define la función `main`, que es el punto de entrada estándar para cualquier aplicación escrita en Kotlin. Aquí es donde comienza la ejecución del programa.
- **Línea 9** `rrunApplication<KotlinGcpApplication>(*args)`:
Esta línea ejecuta la función `runApplication`, que realiza las siguientes tareas:
    - **Inicia Spring Boot**: Configura el contexto de la aplicación y todos los beans necesarios.
    - **Levanta el Servidor Embebido**: En este caso, inicia un servidor Tomcat que se ejecuta en el puerto `8080` por defecto.
    - **Carga los Recursos de Configuración**: Como `application.properties` o `application.yml`, que contienen configuraciones clave como la conexión a la base de datos.
    - **Procesa los Argumentos**: Los parámetros pasados en `args` se interpretan como argumentos de línea de comandos para personalizar el comportamiento de la aplicación en tiempo de ejecución.

**Nota**:
El operador `*` delante de `args` se llama spread operator en Kotlin. Descompone el arreglo de argumentos para que puedan ser pasados individualmente como parámetros a la función `runApplication`.

### **Modelo de Datos**: `Speaker.kt`

La clase `Speaker.kt` define el modelo de datos que representa la tabla `speakers` en la base de datos. Es una entidad JPA (Java Persistence API) que Spring Data utiliza para mapear automáticamente los registros de la base de datos a objetos Kotlin.

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

- **Línea 1** `@Entity`: 
    - Marca la clase como una entidad JPA, lo que indica que está vinculada a una tabla en la base de datos.
    - `Spring Boot`: Detecta esta anotación durante el escaneo de componentes (gracias a `@SpringBootApplication`) y registra automáticamente la clase como una entidad administrada.
- **Línea 2**: `@Table(name = "speakers")`:
    - Especifica el nombre de la tabla en la base de datos asociada con esta entidad.
- **Línea 3** `data class`: 
    - La clase `Speaker` está definida como una **data class**, lo que aporta las siguientes ventajas:
        - **Inmutabilidad**:
            - Los valores de las propiedades son constantes (`val`) y no pueden ser modificados después de la creación del objeto.
            - Esto es útil en aplicaciones donde los datos no deben cambiarse accidentalmente.
        - **Métodos Automáticos Generados**:
            - `toString`: Proporciona una representación de texto para imprimir fácilmente los objetos.
            - `equals` y `hashCode`: Facilitan la comparación entre objetos.
            - `copy`: Permite crear una nueva instancia basada en un objeto existente con cambios en algunas propiedades.
        - **Compatibilidad con Kotlin/JPA**:
            - Las `data class` se integran bien con JPA, siempre que cumplan las reglas básicas (como constructor primario completo).
- **Línea 4** `@Id`: Define este campo como la clave primaria de la tabla.
- **Línea 5** `@GeneratedValue`: Especifica que el valor de este campo será generado automáticamente.
    - `strategy = GenerationType.SEQUENCE`: Usa una estrategia de secuencia, que es eficiente para PostgreSQL.
    - `generator = "speaker_id_generator"`: Asocia este campo a un generador de secuencias específico.
- **Línea 6** `SequenceGenerator`: Configura el generador de secuencias.
    - `name = "speaker_id_generator"`: Nombre del generador asociado.
    - `sequenceName = "speaker_id_seq"`: Nombre de la secuencia en la base de datos que generará los valores del campo `id`.
- **Propósito**: Permite que PostgreSQL genere automáticamente un valor único para cada registro en el campo `id`.
- **Nullable**: Se define como `Long? = null` porque el valor inicial es `null` hasta que se genera automáticamente al guardar el registro.
- **Campos**:
    - `id`: Clave primaria generada automáticamente usando una secuencia PostgreSQL.
    - `name`: Almacena el nombre del speaker.
    - `country`: Almacena el país de origen del speaker.

### **Repositorio**: `SpeakerRepository.kt`

El repositorio es una interfaz que permite interactuar con la base de datos de manera directa, manejando operaciones CRUD (Crear, Leer, Actualizar y Eliminar) sin necesidad de escribir código SQL manualmente. En este caso, `SpeakerRepository` hereda de `CrudRepository`, una clase proporcionada por Spring Data JPA.

```kotlin title="SpeakerRepository.kt" linenums="1"
@Repository
interface SpeakerRepository : CrudRepository<Speaker, Long>
```

- **Línea 2**: `interface SpeakerRepository : CrudRepository<Speaker, Long>`
    - `interface`: Define que `SpeakerRepository` es una interfaz, no una clase.
    - `: CrudRepository<Speaker, Long>`:
        - `CrudRepository` es una interfaz genérica que requiere dos parámetros:
        - `Speaker`: Es la entidad sobre la que este repositorio operará.
        - `Long`: Es el tipo de dato de la clave primaria de la entidad Speaker.

**Nota Importante**:
Spring implementa automáticamente esta interfaz al detectar su definición durante el escaneo de componentes. No es necesario escribir código adicional para implementar esta interfaz, lo que ahorra tiempo y esfuerzo.

- Este repositorio hereda de `CrudRepository`, proporcionando métodos como:
    - **`findById`**: Busca un registro por su ID.
    - **`findAll`**: Devuelve todos los registros.
    - **`save`**: Guarda o actualiza un registro.
    - **`deleteById`**: Elimina un registro por ID.

### **Servicio**: `SpeakerService.kt`

El archivo `SpeakerService.kt` implementa la lógica de negocio para la API REST. Actúa como un intermediario entre el controlador y el repositorio, asegurando que las operaciones CRUD (Crear, Leer, Actualizar y Eliminar) se realicen de manera controlada.

```kotlin title="SpeakerRepository.kt" linenums="1"
@Service
@Transactional
class SpeakerService(val repo: SpeakerRepository) {
    fun getSpeakers(): List<Speaker> = repo.findAll().toList()
    fun getSpeaker(id: Long) = repo.findById(id)
    fun createSpeaker(speaker: Speaker) = repo.save(speaker)
    fun deleteSpeaker(id: Long) = repo.deleteById(id)
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

- **Línea 1**: `@Service`
    - **Propósito**: Marca esta clase como un servicio administrado por **Spring**.
    - **Spring Boot**: Detecta automáticamente esta anotación durante el escaneo de componentes y la registra como un bean disponible en el contenedor de la aplicación.
    - **Ventaja**: Permite inyectar esta clase en otros componentes (como el controlador) sin necesidad de inicializarla manualmente.
- **Línea 2**: `@Transactional`
    - **Propósito**: Garantiza que todas las operaciones dentro de este servicio se ejecuten dentro de una transacción.
    - **Ventajas**:
        - Si ocurre un error en cualquier parte del método, todas las operaciones realizadas dentro de la transacción serán revertidas.
        - Asegura consistencia en la base de datos.
- **Línea 3** `Constructor: private val repo: SpeakerRepository`
    - Inyecta automáticamente el repositorio `SpeakerRepository` en el servicio.
    - Esto permite acceder a los métodos predefinidos en `CrudRepository` (como `findAll`, `save`, etc.) para interactuar con la base de datos.
- **Línea 4** `getSpeakers`: Devuelve una lista de todos los registros en la tabla speakers.
    - **Detalle**:
        - `repo.findAll()`: Devuelve un `Iterable<Speaker>`.
        - `.toList()`: Convierte el iterable en una lista, que es más fácil de manejar.
- **Línea 5** `getSpeaker`: Busca un registro por su clave primaria (`id`) y lo devuelve. Si no existe, devuelve `null`.
    - **Detalle**:
        - `repo.findById(id)`: Devuelve un `Optional<Speaker>`.
        - `.orElse(null)`: Extrae el valor si existe, o devuelve `null` si no lo encuentra.
- **Línea 6** `createSpeaker`: Inserta un nuevo registro en la base de datos.
    - **Detalle**:
        - `repo.save(speaker)`: Guarda el objeto `Speaker` en la base de datos y devuelve la entidad guardada con su clave primaria (`id`) asignada.
- **Línea 7** `deleteSpeaker`: Elimina un registro por su clave primaria (id).
    - **Detalle**:
        - `repo.deleteById(id)`: Elimina el registro correspondiente en la base de datos.
- **Línea 8** `updateSpeaker`: Actualiza un registro existente en la base de datos. Si no existe, devuelve null.
    - **Detalle**:
        - `repo.existsById(id)`: Verifica si el registro con el `id` especificado existe.
        - `updatedSpeaker.copy(id = id)`: Crea una nueva instancia del objeto `Speaker` con el id correcto.
        - `repo.save(speakerToUpdate)`: Guarda el registro actualizado.

### **Controlador REST**: `SpeakerController.kt`

El controlador se encarga de manejar las solicitudes HTTP entrantes y delegarlas al servicio correspondiente. En este caso, proporciona una interfaz para las operaciones CRUD sobre la entidad `Speaker`.

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
