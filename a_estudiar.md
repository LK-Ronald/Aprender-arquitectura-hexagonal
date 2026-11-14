# 🧭 Guía de Estudio: Ruta para Entender y Dominar la Arquitectura Hexagonal

> Este documento contiene la lista estructurada de temas, conceptos teóricos, patrones de diseño y buenas prácticas indispensables para comprender a fondo la **Arquitectura Hexagonal (Ports & Adapters)** y aplicarla con éxito en proyectos reales.

---

## 🗺️ Mapa de Ruta del Aprendizaje

```
[Nivel 0: Fundamentos POO & SOLID]
               │
               ▼
[Nivel 1: Principios de la Arquitectura Hexagonal]
               │
               ▼
[Nivel 2: Domain-Driven Design (DDD Táctico)]
               │
               ▼
[Nivel 3: Flujo de Información, DTOs y Mappers]
               │
               ▼
[Nivel 4: Ensamblado e Inyección de Dependencias]
               │
               ▼
[Nivel 5: Testing en Arquitectura Hexagonal]
               │
               ▼
[Nivel 6: Antipatrones y Errores Frecuentes]
```

---

## 📚 Módulo 0: Fundamentos de POO y Principios de Diseño

Antes de entender la arquitectura, es obligatorio dominar estos conceptos del paradigma orientado a objetos:

- [ ] **Interfaces vs. Clases Concretas:**
  - Cómo programar contra abstracciones y no contra implementaciones (`List` vs `ArrayList`).
  - Polimorfismo en tiempo de ejecución.
- [ ] **Principios SOLID (Especialmente DIP y SRP):**
  - **S (Single Responsibility Principle):** Cada clase tiene una sola razón para cambiar.
  - **O (Open/Closed Principle):** El sistema debe estar abierto a extensión pero cerrado a modificación.
  - **D (Dependency Inversion Principle):** Los módulos de alto nivel no deben depender de los de bajo nivel; ambos deben depender de abstracciones.
- [ ] **Patrones de Diseño Esenciales:**
  - **Adapter (Adaptador):** Permite que clases con interfaces incompatibles trabajen juntas.
  - **Dependency Injection (Inyección de Dependencias):** Pasar las dependencias por constructor en lugar de instanciarlas internamente con `new`.
  - **Factory / Builder:** Creación limpia de objetos complejos.

---

## 🏛️ Módulo 1: Fundamentos de la Arquitectura Hexagonal (Ports & Adapters)

Creada por **Alistair Cockburn** en 2005 para resolver el problema de la arquitectura en capas tradicional (donde la lógica quedaba acoplada a la base de datos o a la interfaz gráfica).

- [ ] **El Principio del Núcleo Aislado:**
  - El Dominio y la Lógica de Negocio viven en el centro.
  - El Dominio no debe conocer frameworks, bases de datos, librerías de UI ni bibliotecas de serialización (JSON/XML).
- [ ] **La Regla de Dependencia:**
  - Todas las flechas de dependencia de código fuente apuntan **hacia el interior** (hacia el Dominio).
  - La infraestructura conoce a la aplicación, pero la aplicación no conoce a la infraestructura.
- [ ] **¿Qué es un Puerto (Port)?:**
  - Es un **contrato formal** (una `interface` en Java).
  - Define una frontera de entrada o salida del hexágono.
- [ ] **Puertos de Entrada (Driving / Primary / Inbound Ports):**
  - Definen **lo que la aplicación ofrece al mundo exterior**.
  - Representan los **Casos de Uso** (ej: `CreateUserUseCase`, `LoginUseCase`, `GetNinoByIdUseCase`).
- [ ] **Puertos de Salida (Driven / Secondary / Outbound Ports):**
  - Definen **lo que la aplicación necesita del exterior para cumplir su trabajo**.
  - Declarados como interfaces dentro de la aplicación (ej: `SaveUserPort`, `EmailSenderPort`, `PasswordEncoderPort`).
- [ ] **¿Qué es un Adaptador (Adapter)?:**
  - Es la **implementación tecnológica concreta** de un puerto.
- [ ] **Adaptadores Primarios (Driving Adapters):**
  - Inician la acción desde el exterior hacia la aplicación.
  - Ejemplos: Terminal/CLI (`UserManagementCli`), Controladores REST (`UserController`), Consumidores de colas (Kafka/RabbitMQ), Controladores Desktop JavaFX/Swing.
- [ ] **Adaptadores Secundarios (Driven Adapters):**
  - Reaccionan ante los llamados de la aplicación interactuando con el exterior.
  - Ejemplos: Repositorios JDBC/JPA (`UserRepositoryMySQL`), Clientes HTTP externos, Adaptadores SMTP (`JavaMailEmailSenderAdapter`), Sistema de archivos.

---

## 🎯 Módulo 2: Domain-Driven Design (DDD) Táctico

La Arquitectura Hexagonal es el contenedor arquitectónico ideal para albergar un Dominio diseñado con DDD.

- [ ] **Entidades (Entities):**
  - Objetos con identidad única persistente a través del tiempo (ej: `UserModel` con su `id`).
  - Tienen comportamiento de negocio (métodos como `activate()`, `deactivate()`, `discharge()`).
- [ ] **Objetos de Valor (Value Objects - VO):**
  - Objetos inmutables definidos únicamente por el valor de sus atributos, sin identidad propia.
  - **Autorvalidación de invariantes:** Un `UserEmail` o un `NinoBirthDate` lanzan excepción en el constructor si los datos son inválidos, imposibilitando la existencia de objetos en estado inconsistente.
- [ ] **Modelo Anémico vs. Modelo Rico:**
  - **Antipatrón del Modelo Anémico:** Clases que son meras bolsas de getters y setters sin reglas de negocio.
  - **Modelo Rico:** Clases con métodos semánticos que expresan las reglas del negocio.
- [ ] **Agregados y Raíz de Agregado (Aggregate Root):**
  - Grupo de entidades y objetos de valor tratados como una unidad atómica de consistencia.
- [ ] **Eventos de Dominio (Domain Events):**
  - Hechos inmutables que ocurrieron en el negocio y que interesan a otras partes del sistema (ej: `UserCreatedDomainEvent`, `NinoUpdatedDomainEvent`).
- [ ] **Excepciones de Dominio:**
  - Excepciones que representan fallos de las reglas de negocio (ej: `UserAlreadyExistsException`, `InvalidNinoAdmissionDateException`), independientes de errores SQL o HTTP.

---

## 🔄 Módulo 3: Flujo de Información, DTOs y Mappers

Uno de los mayores desafíos es entender por qué no se usa la misma clase en todas las capas.

- [ ] **Segregación de Modelos:**
  - **DTO de Entrada/Presentación:** Lo que el usuario o cliente envía (`CreateUserRequest`).
  - **Comando / Consulta de Aplicación:** Objeto que representa la intención del caso de uso (`CreateUserCommand`, `GetUserByIdQuery`).
  - **Modelo de Dominio:** Entidad pura de negocio con Value Objects (`UserModel`).
  - **DTO / Entidad de Persistencia:** Estructura que mapea directamente con las tablas de base de datos (`UserEntity`, `UserPersistenceDto`).
  - **DTO de Respuesta:** Lo que se expone al cliente final (`UserResponse`).
- [ ] **Mappers (Transformadores de Datos):**
  - Por qué y cómo transformar datos entre fronteras (`DesktopMapper`, `ApplicationMapper`, `PersistenceMapper`).
  - Evitar que una columna o tipo de base de datos "contamine" la vista o el dominio.
- [ ] **Validaciones en Capas:**
  - **Validación Sintáctica / Formato:** En la entrada y en la aplicación (ej: anotaciones Jakarta `@NotBlank`, `@Email`, `@NotNull`).
  - **Validación de Negocio / Invariantes:** En los Value Objects y Entidades de Dominio (ej: fecha de admisión no puede ser anterior a la fecha de nacimiento).
  - **Validación de Estado / Unicidad:** En los Servicios de Aplicación consultando puertos de salida (ej: correo no duplicado).

---

## ⚙️ Módulo 4: Ensamblado y Contenedor de Dependencias

Cómo conectar todas las piezas sueltas sin violar las fronteras arquitectónicas.

- [ ] **Composition Root (Raíz de Composición):**
  - El único lugar donde se deben instanciar y cablear las dependencias concretas.
  - En este proyecto: `DependencyContainer.java`.
- [ ] **Inyección de Dependencias Pura (Pure Java DI):**
  - Construir todo el grafo de objetos manualmente sin frameworks mágicos ni reflexión.
  - Instanciar adaptadores de salida -> inyectarlos en servicios de aplicación -> inyectar servicios en controladores.
- [ ] **Inversión de Control (IoC):**
  - El flujo de ejecución es controlado por el contenedor y las interfaces, no por clases acopladas.

---

## 🧪 Módulo 5: Estrategias de Testing

La mayor ventaja práctica de la Arquitectura Hexagonal es su facilidad extrema para ser testeada:

- [ ] **Tests Unitarios del Dominio:**
  - Probar entidades y Value Objects sin mocks, sin base de datos y sin librerías externas.
  - Ejecución en milisegundos.
- [ ] **Tests Unitarios de la Capa de Aplicación:**
  - Probar los servicios (`CreateUserServiceTest`) simulando los puertos de salida (`SaveUserPort`, `GetUserByEmailPort`) usando herramientas como **Mockito**.
- [ ] **Tests de Integración de Adaptadores:**
  - Probar exclusivamente los adaptadores contra bases de datos reales o simuladas (ej: `UserRepositoryMySQLTest`, `JavaMailEmailSenderAdapterTest`).
- [ ] **Desacoplamiento de Pruebas:**
  - Poder cambiar MySQL por PostgreSQL sin que falle ni una sola prueba de dominio o aplicación.

---

## ⚠️ Módulo 6: Antipatrones y Errores Frecuentes

Cosas que debes evitar a toda costa:

- [ ] **Fuga de Infraestructura hacia el Dominio:**
  - Colocar anotaciones de JPA (`@Entity`, `@Table`, `@Id`) o de Jackson (`@JsonProperty`) en los modelos de dominio.
- [ ] **Puertos Acoplados a Tecnologías:**
  - Diseñar un puerto de entrada que reciba `HttpServletRequest` o un puerto de salida que devuelva `java.sql.ResultSet`. Los puertos solo deben hablar en términos de Dominio o DTOs agnósticos.
- [ ] **Servicios que se saltan el Dominio:**
  - Servicios de aplicación que hacen consultas SQL directamente o que manipulan datos crudos sin pasar por el modelo de dominio.
- [ ] **Dependencia Circular entre Capas:**
  - El paquete `domain` jamás debe importar nada de `application` ni de `infrastructure`.

---

## 📖 Lecturas y Referencias Recomendadas

1. **Alistair Cockburn:** *Hexagonal Architecture (Ports and Adapters Pattern)* - Artículo fundacional (2005).
2. **Eric Evans:** *Domain-Driven Design: Tackling Complexity in the Heart of Software* (El libro azul).
3. **Vaughn Vernon:** *Implementing Domain-Driven Design* (El libro rojo).
4. **Robert C. Martin (Uncle Bob):** *Clean Architecture: A Craftsman's Guide to Software Structure and Design*.
5. **Martin Fowler:** Artículos sobre *Anemic Domain Model*, *Value Objects* y *Separation of Concerns*.

---

## 🎯 Ejercicio Práctico Sugerido

Para verificar tu comprensión en este proyecto:
1. Inspecciona el flujo de la entidad `Nino`:
   - [NinoModel.java](file:///src/main/java/com/jcaa/usersmanagement/domain/model/NinoModel.java) (Dominio)
   - [CreateNinoUseCase.java](file:///src/main/java/com/jcaa/usersmanagement/application/port/in/CreateNinoUseCase.java) (Puerto In)
   - [CreateNinoService.java](file:///src/main/java/com/jcaa/usersmanagement/application/service/CreateNinoService.java) (Aplicación)
   - [SaveNinoPort.java](file:///src/main/java/com/jcaa/usersmanagement/application/port/out/SaveNinoPort.java) (Puerto Out)
   - [NinoRepositoryMySQL.java](file:///src/main/java/com/jcaa/usersmanagement/infrastructure/adapter/persistence/repository/NinoRepositoryMySQL.java) (Adaptador Out)
   - [NinoController.java](file:///src/main/java/com/jcaa/usersmanagement/infrastructure/entrypoint/desktop/controller/NinoController.java) (Adaptador In)
2. Intenta responder: *¿Qué cambios serían necesarios si cambiáramos la interfaz de consola por una API REST?*  
   *(Pista: Solo se crearían nuevos adaptadores de entrada en `infrastructure`, el dominio y la aplicación permanecerían intactos).*
