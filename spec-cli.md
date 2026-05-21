# SPEC.md — myUniverse

## 1. Resumen

myUniverse es una aplicación desarrollada en **Java** con una arquitectura **Clean Architecture / Hexagonal ligera**, orientada a la exploración de un edificio de ejemplo mediante una **interfaz de terminal** (CLI/TUI), con navegación por plantas, consulta de espacios, búsqueda, filtrado y un **dashboard de administración en consola**. La aplicación se ejecuta desde terminal y permite alternar entre **modo 2D** y **modo 3D** mediante comandos o parámetros de arranque [web:48][web:49].

La primera versión no pretende modelar toda la universidad visualmente, pero sí debe cubrir **todos los casos de uso funcionales del proyecto**, incluyendo el **CRUD de Universidad**, el CRUD de Edificios, Plantas, Espacios y Recorridos, así como los flujos de visitante y administración [web:50][web:57].

---

## 2. Objetivos

- Construir una aplicación de terminal en Java para explorar un edificio por plantas.
- Mostrar planos ASCII o textuales de forma clara y navegable.
- Consultar espacios, puntos de interés y detalles contextuales.
- Incluir un dashboard de administración por consola.
- Permitir ejecutar el sistema en modo 2D o modo 3D desde terminal.
- Mantener una arquitectura limpia, modular y testeable.
- Garantizar cobertura funcional de todos los casos de uso del proyecto.

---

## 3. Alcance funcional

### 3.1 Funcionalidad de visitante

- Ver edificio disponible.
- Ver plantas del edificio.
- Seleccionar una planta.
- Ver espacios de la planta.
- Buscar espacios por nombre o tipo.
- Filtrar puntos de interés.
- Ver detalle de un espacio.
- Ver recorridos si están definidos.
- Navegar por menús de texto y pantallas de consola.

### 3.2 Funcionalidad de administración

- CRUD de Universidad.
- CRUD de Edificio.
- CRUD de Planta.
- CRUD de Espacio.
- CRUD de Recorrido.
- Importar y exportar datos.
- Consultar estadísticas básicas.
- Gestionar contenido del sistema desde consola.

### 3.3 Evolución técnica

- Modo 2D inicial en terminal.
- Modo 3D activado por parámetro CLI.
- Posible uso de TUI avanzada si se requiere.
- Posible backend o persistencia remota futura.

---

## 4. Decisión tecnológica

- Lenguaje principal: **Java**
- Construcción: **Maven** o **Gradle**
- Arquitectura: **Clean Architecture / Hexagonal ligera**
- Interfaz: **CLI/TUI**
- Librerías recomendadas:
  - **Spring Shell** para comandos interactivos [web:48][web:51]
  - **Lanterna** para TUI en terminal con cajas, paneles y navegación textual [web:49][web:52]
  - **JLine** opcional para mejorar entrada/salida y experiencia de terminal [web:57]
- Render visual inicial: plano 2D en ASCII o caracteres
- Evolución 3D: adaptador de infraestructura que calcula una proyección textual o pseudo-3D
- Persistencia inicial: JSON local
- Persistencia futura: API o base de datos

---

## 5. Estructura del repositorio

```text
myUniverse/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/myuniverse/
│   │   │       ├── domain/
│   │   │       ├── application/
│   │   │       ├── infrastructure/
│   │   │       ├── presentation/
│   │   │       ├── admin/
│   │   │       └── shared/
│   │   └── resources/
│   │       ├── data/
│   │       └── config/
│   └── test/
│       └── java/
├── docs/
│   ├── SPEC.md
│   └── diagrams/
├── README.md
└── pom.xml
```

Si más adelante se separa backend y frontend, la solución puede evolucionar a:

```text
myUniverse/
├── frontend/   (si hubiera una UI externa futura)
├── backend/
└── docs/
```

---

## 6. Arquitectura de la interfaz de terminal

La capa de presentación se divide en dos bloques: **visitor** y **admin**, ambos ejecutados desde consola.

```text
presentation/
└── cli/
    ├── CliController.java
    ├── visitor/
    │   ├── VisitorConsole.java
    │   ├── MenuVisitor.java
    │   ├── FloorMapConsole.java
    │   └── SpaceDetailConsole.java
    └── admin/
        ├── AdminDashboardConsole.java
        ├── UniversityCrudConsole.java
        ├── BuildingCrudConsole.java
        ├── FloorCrudConsole.java
        ├── SpaceCrudConsole.java
        └── RouteCrudConsole.java
```

### Responsabilidades

#### `CliController`
- Enrutador principal de comandos.
- Decide si se abre modo visitante, admin, 2D o 3D.
- Interpreta parámetros de arranque y comandos iniciales.

#### `VisitorConsole`
- Muestra menús de navegación.
- Renderiza el plano textual del edificio.
- Permite seleccionar planta, espacio o recorrido.

#### `AdminDashboardConsole`
- Gestiona el panel de administración.
- Presenta menús numerados o guiados.
- Orquesta los CRUD mediante formularios de texto.

#### `UniversityCrudConsole`
- Implementa el flujo guiado de creación, edición y eliminación.
- Hace preguntas una por una.
- Valida entradas antes de enviar al caso de uso.

---

## 7. Flujo del dashboard de administración

El dashboard de administración funciona como un conjunto de pantallas de consola guiadas.

### Ejemplo de creación

```text
=== CREAR NUEVA UNIVERSIDAD ===
> Ingrese el nombre: Universidad de Cantabria
> Ingrese las siglas: UC
> Ingrese la ciudad: Santander
 Guardar   Cancelar[3][4]
```

### Reglas
- El flujo debe ser claro y secuencial.
- Cada formulario debe validar campos obligatorios.
- El usuario debe poder cancelar en cualquier momento.
- La vista de consola no debe contener lógica de negocio.
- Toda operación debe ejecutarse mediante casos de uso.

---

## 8. Modo 3D en terminal

En esta versión, el “modo 3D” no usa WebGL ni motor gráfico real. Se implementa como una **evolución de renderizado textual**.

### Modo 2D
- Vista plana del edificio en ASCII.
- Representación por caracteres.
- Leyenda simple:
  - `#` paredes
  - `.` pasillo
  - `D` despacho
  - `A` aula
  - `S` servicio
  - `E` entrada

### Modo 3D
- Vista pseudo-3D en texto.
- Puede mostrar:
  - apilado por capas,
  - alzado/isometría básica,
  - proyección con coordenadas \((X, Y, Z)\),
  - distancias relativas al punto de vista.
- El objetivo es simular profundidad visual sin salir de la consola.

### Activación
```bash
java -jar myUniverse.jar --mode=2d
java -jar myUniverse.jar --mode=3d
java -jar myUniverse.jar --mode=admin
```

### Regla clave
El dominio no debe conocer el modo de ejecución. La selección solo afecta a presentación e infraestructura.

---

## 9. Casos de uso

### Visitante
- verEdificio()
- listarPlantas()
- seleccionarPlanta()
- verEspaciosPlanta()
- verEspacio()
- buscarEspacio()
- verDetallesEspacio()
- filtrarPuntosInteres()
- verRecorridos()

### Administrador
- crearUniversidad()
- listarUniversidades()
- actualizarUniversidad()
- eliminarUniversidad()
- crearEdificio()
- actualizarEdificio()
- eliminarEdificio()
- crearPlanta()
- actualizarPlanta()
- eliminarPlanta()
- crearEspacio()
- actualizarEspacio()
- eliminarEspacio()
- crearRecorrido()
- actualizarRecorrido()
- eliminarRecorrido()
- importarDatos()
- exportarDatos()

---

## 10. Estructura interna por capas

### 10.1 Domain

```text
domain/
├── entities/
│   ├── University.java
│   ├── Building.java
│   ├── Floor.java
│   ├── Space.java
│   ├── Route.java
│   └── PointOfInterest.java
├── valueobjects/
│   ├── UniversityId.java
│   ├── BuildingId.java
│   ├── FloorNumber.java
│   ├── SpaceId.java
│   ├── Coordinate2D.java
│   ├── Coordinate3D.java
│   └── SpaceType.java
├── repositories/
│   ├── UniversityRepository.java
│   ├── BuildingRepository.java
│   ├── SpaceRepository.java
│   └── RouteRepository.java
└── services/
    ├── SearchService.java
    └── RouteValidationService.java
```

### 10.2 Application

```text
application/
├── usecases/
│   ├── university/
│   │   ├── CreateUniversityUseCase.java
│   │   ├── UpdateUniversityUseCase.java
│   │   ├── DeleteUniversityUseCase.java
│   │   └── ListUniversitiesUseCase.java
│   ├── building/
│   │   ├── GetBuildingFloorsUseCase.java
│   │   ├── GetFloorPlanUseCase.java
│   │   └── SelectFloorUseCase.java
│   ├── space/
│   │   ├── GetSpaceByIdUseCase.java
│   │   ├── SearchSpacesUseCase.java
│   │   ├── FilterPointsOfInterestUseCase.java
│   │   └── ListSpacesByTypeUseCase.java
│   ├── route/
│   │   ├── CreateRouteUseCase.java
│   │   ├── UpdateRouteUseCase.java
│   │   └── DeleteRouteUseCase.java
│   └── render/
│       ├── RenderFloor2DUseCase.java
│       └── RenderFloor3DUseCase.java
├── dto/
│   ├── UniversityDTO.java
│   ├── BuildingDTO.java
│   ├── FloorDTO.java
│   ├── SpaceDTO.java
│   └── RouteDTO.java
└── ports/
    ├── input/
    └── output/
```

### 10.3 Infrastructure

```text
infrastructure/
├── persistence/
│   ├── json/
│   │   ├── UniversityJsonRepository.java
│   │   ├── BuildingJsonRepository.java
│   │   ├── SpaceJsonRepository.java
│   │   └── RouteJsonRepository.java
│   └── mapper/
│       ├── UniversityMapper.java
│       ├── BuildingMapper.java
│       ├── FloorMapper.java
│       └── SpaceMapper.java
├── visualization/
│   ├── terminal2d/
│   │   └── AsciiFloorPlanAdapter.java
│   ├── terminal3d/
│   │   └── Pseudo3DFloorPlanAdapter.java
│   └── renderer/
│       ├── FloorRenderer.java
│       └── RouteRenderer.java
├── cli/
│   ├── CommandParser.java
│   ├── AppModeResolver.java
│   └── TerminalSession.java
└── config/
```

### 10.4 Presentation

```text
presentation/
└── cli/
    ├── CliController.java
    ├── visitor/
    │   ├── VisitorConsole.java
    │   ├── MenuVisitor.java
    │   ├── FloorMapConsole.java
    │   └── SpaceDetailConsole.java
    └── admin/
        ├── AdminDashboardConsole.java
        ├── UniversityCrudConsole.java
        ├── BuildingCrudConsole.java
        ├── FloorCrudConsole.java
        ├── SpaceCrudConsole.java
        └── RouteCrudConsole.java
```

---

## 11. Contratos principales

```java
public interface UniversityRepository {
    List<University> findAll();
    Optional<University> findById(UniversityId id);
    void save(University university);
    void update(University university);
    void deleteById(UniversityId id);
}
```

```java
public interface BuildingRepository {
    Optional<Building> findById(BuildingId id);
    List<Floor> findFloorsByBuildingId(BuildingId id);
    Optional<Space> findSpaceById(SpaceId id);
    List<Space> searchSpaces(String term);
}
```

```java
public interface RouteRepository {
    List<Route> findAll();
    Optional<Route> findById(String id);
    void save(Route route);
    void update(Route route);
    void deleteById(String id);
}
```

---

## 12. Persistencia inicial

La persistencia inicial será mediante JSON local.

```text
src/main/resources/data/
├── universities.json
├── buildings.json
├── floors.json
├── spaces.json
└── routes.json
```

### Objetivos
- Reducir complejidad inicial.
- Facilitar pruebas.
- Permitir evolución progresiva.
- Evitar depender de una base de datos al principio.

---

## 13. Comandos de terminal

### Inicio de aplicación
```bash
java -jar myUniverse.jar
```

### Modo visitante
```bash
java -jar myUniverse.jar --mode=visitor
```

### Modo administración
```bash
java -jar myUniverse.jar --mode=admin
```

### Modo 2D
```bash
java -jar myUniverse.jar --mode=2d
```

### Modo 3D
```bash
java -jar myUniverse.jar --mode=3d
```

### Ayuda
```bash
java -jar myUniverse.jar --help
```

Si se usa Spring Shell, también puede exponerse una shell interactiva con comandos como:
```bash
university create --name "Universidad de Cantabria" --code "UC"
space list --floor 1
route show --id r-001
```

---

## 14. Testing

### Qué se prueba
- Entidades de dominio.
- Value Objects.
- Casos de uso.
- Validaciones.
- Mapeadores.
- Repositorios mockeados.
- Flujos del dashboard admin.
- Renderizadores 2D y 3D en consola, al menos a nivel funcional.

### Herramientas
- JUnit 5
- Mockito
- AssertJ
- Tests de integración ligeros si hacen falta

### Criterio
Cada caso de uso debe tener tests unitarios que verifiquen su comportamiento principal.

---

## 15. Implementación incremental

### Fase 1
- Definir entidades y value objects.
- Definir contratos de repositorio.
- Definir casos de uso de visitante y administración.
- Crear JSON de ejemplo.
- Montar CLI básica.

### Fase 2
- Implementar menús de visitante.
- Implementar dashboard admin.
- Implementar CRUD de Universidad.
- Implementar CRUD de Edificios, Plantas, Espacios y Recorridos.

### Fase 3
- Implementar render 2D ASCII.
- Añadir búsqueda y filtros.
- Añadir validaciones de consola.

### Fase 4
- Implementar modo 3D pseudo-3D.
- Añadir proyección o apilado por capas.
- Optimizar navegación y experiencia de usuario.

### Fase 5
- Tests completos.
- Refactorización y limpieza.
- Preparar migración a backend o persistencia avanzada.

---

## 16. Principios de diseño

- Alta cohesión.
- Bajo acoplamiento.
- Dependencias hacia adentro.
- Separación entre lógica y presentación.
- Código orientado al dominio.
- Trazabilidad caso de uso → módulo.
- Extensibilidad sin romper el núcleo.

---

## 17. Criterio académico

La reducción del escenario visual a un edificio de ejemplo no reduce el alcance funcional del proyecto. El sistema debe seguir soportando todos los casos de uso definidos, incluyendo el CRUD de Universidad y el dashboard administrativo. El modo 3D se considera una evolución técnica de la capa de terminal, no un motor gráfico externo [web:48][web:49][web:57].