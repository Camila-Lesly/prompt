# Propuesta técnica de arquitectura — myUniverse

Desarrollar **myUniverse** como una aplicación **web** orientada a la exploración de un **edificio de ejemplo** mediante planos interactivos por planta, consulta de espacios y localización de puntos de interés. La solución propuesta sigue una arquitectura **frontend por capas con enfoque Clean Architecture / Hexagonal ligera**, separando claramente presentación, aplicación, dominio e infraestructura para facilitar mantenibilidad, testing y evolución futura.

La primera versión no busca modelar toda la universidad ni construir un 3D real complejo, sino validar una solución técnicamente sólida, bien diseñada y extensible. La visualización se apoya en **React + TypeScript** y un renderizado inicial mediante **SVG interactivo**, dejando abierta una evolución futura hacia vistas pseudo-3D o integración con motores gráficos si el proyecto lo requiere.

Además, esta propuesta se formula con una condición esencial: la arquitectura debe estar preparada para **cumplir con todos los casos de uso definidos en el proyecto**. El hecho de trabajar inicialmente sobre un edificio de ejemplo reduce el alcance espacial, pero **no reduce el alcance funcional** del sistema.

***

## Objetivos y no objetivos

### Objetivos

- Construir una aplicación web que permita explorar un **edificio ejemplo** por dentro.
- Localizar aulas, despachos, servicios y otros puntos de interés.
- Consultar información contextual de los espacios del edificio.
- Mantener una arquitectura limpia, modular y fácil de justificar académicamente.
- Facilitar pruebas unitarias sobre la lógica del dominio y los casos de uso.
- Garantizar que la arquitectura puede soportar **todos los casos de uso especificados en el proyecto**.
- Permitir una evolución futura hacia navegación más avanzada o visualización 3D.

### No objetivos

- Modelar toda la universidad real en esta versión visual.
- Implementar desde el inicio un motor 3D completo o un gemelo digital realista.
- Resolver navegación indoor compleja con posicionamiento en tiempo real si no está requerida en la primera iteración.
- Introducir una infraestructura backend pesada si no es necesaria para la primera entrega.

***

## Restricción funcional clave

La arquitectura propuesta debe cumplir con **todos los casos de uso definidos en el proyecto**, independientemente de que la representación visual inicial se centre en un único edificio de ejemplo.

Esto implica que:

- el sistema debe respetar los actores definidos,
- los flujos funcionales deben mantenerse,
- los módulos deben alinearse con los comportamientos requeridos,
- y cualquier simplificación del entorno debe afectar solo a la escala del escenario, no a la lógica del sistema.

En otras palabras, el edificio de ejemplo actúa como un **contexto de validación**, no como una redefinición del problema original.

***

## Alcance funcional de esta versión

La aplicación representa un **edificio ficticio o de ejemplo** con varias plantas y espacios internos. El usuario puede recorrer visualmente cada planta, seleccionar espacios y consultar información asociada.

Este alcance se plantea como una forma de implementar y validar los casos de uso del proyecto sobre un entorno más manejable, sin alterar el comportamiento esperado del sistema.

### Funcionalidades principales

- Visualizar el edificio por plantas.
- Mostrar aulas, despachos, baños, ascensores, escaleras y salidas.
- Consultar detalles de un espacio seleccionado.
- Buscar espacios por nombre o tipo.
- Filtrar puntos de interés.
- Cambiar entre plantas del edificio.
- Soportar la evolución hacia funcionalidades adicionales exigidas por los casos de uso del proyecto.

### Evolución futura posible

- Rutas internas entre espacios.
- Vista isométrica o pseudo-3D.
- Integración con mapas reales o motores 3D.
- Soporte multi-edificio.
- Persistencia remota y panel de administración.

***

## Visión arquitectónica

La propuesta sigue una **arquitectura por capas** con una separación explícita entre lógica de negocio y detalles tecnológicos.

```text
Usuario
  ↕
Interfaz Web (React)
  ↕
Casos de Uso / Aplicación
  ↕
Dominio
  ↕
Infraestructura (JSON, repositorios, adaptadores de visualización)
```

### Decisiones arquitectónicas clave

- **Frontend web** como primera versión del sistema.
- **React + TypeScript** como stack principal.
- **Clean Architecture / Hexagonal ligera** para separar responsabilidades.
- **Dominio independiente del framework**.
- **Repositorio abstracto** para desacoplar la fuente de datos.
- **SVG interactivo** como mecanismo inicial de representación visual del edificio.
- **Tests unitarios** en dominio y casos de uso desde el principio.
- **Trazabilidad entre arquitectura y casos de uso** como criterio obligatorio de diseño.

***

## Justificación de la arquitectura

Esta arquitectura es adecuada porque el proyecto necesita demostrar diseño y mantenibilidad más que complejidad gráfica. Separar el dominio del resto de capas permite cambiar la forma de visualizar el edificio o la fuente de datos sin reescribir la lógica principal.

Además, esta propuesta es especialmente defendible en un contexto académico porque favorece:

- alta cohesión,
- bajo acoplamiento,
- facilidad de prueba,
- extensibilidad,
- alineación con principios SOLID,
- y trazabilidad directa entre requisitos funcionales y módulos técnicos.

***

## Cobertura de casos de uso

La arquitectura propuesta para **myUniverse** debe cumplir con **todos los casos de uso definidos en el proyecto**. Esto significa que la organización en capas, módulos y componentes no se plantea solo desde una perspectiva técnica, sino también como una forma de garantizar la cobertura funcional del sistema.

Aunque la primera implementación se centre en un **edificio de ejemplo** en lugar de modelar toda la universidad, la solución debe seguir siendo válida para ejecutar los casos de uso especificados en la documentación del proyecto.

### Implicaciones

- Todos los casos de uso del proyecto deben poder trazarse a módulos concretos de la arquitectura.
- La reducción del alcance visual o espacial no debe eliminar comportamiento funcional requerido.
- El edificio de ejemplo se utilizará como entorno de validación del sistema, no como cambio del objetivo funcional del proyecto.
- La arquitectura debe permitir evolucionar desde un edificio a varios sin rehacer el núcleo del sistema.
- Los actores, flujos y responsabilidades definidos en los casos de uso deben respetarse en la implementación.

### Criterio de diseño

Cada decisión técnica debe poder justificarse indicando:

- qué caso de uso soporta,
- qué módulo o capa interviene,
- y cómo se integra esa funcionalidad en la solución global.

***

## Trazabilidad funcional

A continuación se propone una trazabilidad genérica entre tipos de casos de uso y módulos arquitectónicos. La correspondencia exacta deberá ajustarse a los casos de uso concretos del repositorio.

| Tipo de caso de uso | Capa principal | Módulos implicados |
|---|---|---|
| Visualizar edificio o planta | Presentación + Aplicación | `BuildingExplorerPage`, `GetBuildingFloors`, `GetFloorPlan` |
| Consultar espacio | Presentación + Aplicación + Dominio | `SpaceDetailPanel`, `GetSpaceById`, `Space` |
| Buscar aulas o servicios | Aplicación + Infraestructura | `SearchSpaces`, `JsonBuildingRepository` |
| Filtrar puntos de interés | Aplicación + Presentación | `FilterPointsOfInterest`, `PointOfInterestFilter` |
| Cambiar de planta | Presentación + Aplicación | `FloorSelector`, `GetFloorPlan` |
| Gestionar roles o actores | Aplicación + Dominio + Infraestructura | Casos de uso específicos según actor |
| Evolución a administración | Aplicación + Infraestructura | Repositorios, validaciones, persistencia futura |

Esta tabla no sustituye al análisis real del proyecto: sirve como guía para asegurar que **cada caso de uso tenga reflejo arquitectónico explícito**.

***

## Capas del sistema

## 1. Presentación

Responsable de la interacción con el usuario y del renderizado visual del sistema.

### Responsabilidades

- Mostrar las plantas del edificio.
- Renderizar espacios seleccionables.
- Gestionar formularios, filtros y búsqueda.
- Cambiar entre vistas y estados de interfaz.
- Invocar casos de uso de aplicación.
- Adaptarse a los distintos actores y flujos definidos en los casos de uso.

### Tecnologías propuestas

- **React**
- **TypeScript**
- **CSS Modules**, **Tailwind** o CSS estándar
- **SVG interactivo** para representar las plantas

### Componentes típicos

- `BuildingView`
- `FloorSelector`
- `FloorPlan`
- `SpaceCard`
- `SearchBar`
- `PointOfInterestFilter`
- `SpaceDetailPanel`

***

## 2. Aplicación

Contiene los **casos de uso** del sistema. Orquesta operaciones, coordina entidades del dominio y usa repositorios abstractos.

### Responsabilidades

- Obtener plantas de un edificio.
- Buscar espacios.
- Filtrar puntos de interés.
- Obtener el detalle de un espacio.
- Resolver cambios de planta o selección.
- Implementar la lógica asociada a todos los casos de uso definidos en el proyecto.

### Ejemplos de casos de uso

- `GetBuildingFloors`
- `GetFloorPlan`
- `GetSpaceById`
- `SearchSpaces`
- `FilterPointsOfInterest`
- `ListSpacesByType`

### Regla importante

Los casos de uso **no deben depender de React ni del mecanismo visual**.

***

## 3. Dominio

Es el núcleo del sistema. Aquí se modelan las entidades, reglas y conceptos del problema.

### Responsabilidades

- Representar la estructura lógica del edificio.
- Definir relaciones entre plantas, espacios y puntos de interés.
- Encapsular reglas de negocio del sistema.
- Mantener invariantes del modelo.
- Servir como base para implementar correctamente los casos de uso.

### Entidades candidatas

- `Building`
- `Floor`
- `Space`
- `Classroom`
- `Office`
- `Service`
- `PointOfInterest`

### Value Objects posibles

- `SpaceId`
- `FloorNumber`
- `Coordinate2D`
- `SpaceType`
- `DisplayName`

### Posibles reglas del dominio

- Un espacio pertenece a una única planta.
- Una planta pertenece a un único edificio.
- Un punto de interés debe estar asociado a una localización válida.
- El tipo de espacio condiciona la información que puede mostrarse.

***

## 4. Infraestructura

Implementa detalles técnicos concretos sin contaminar la lógica de dominio.

### Responsabilidades

- Cargar datos del edificio desde JSON.
- Implementar repositorios concretos.
- Adaptar datos a la visualización.
- Gestionar servicios auxiliares de búsqueda o persistencia futura.
- Dar soporte técnico a los casos de uso sin mezclar detalles técnicos con reglas de negocio.

### Primera versión propuesta

- Datos almacenados en **JSON local**.
- Repositorios que leen esos datos y los transforman en entidades del dominio.
- Adaptador específico para representar la geometría del plano en SVG.

### Evolución futura

- Sustituir JSON por API REST.
- Incorporar base de datos.
- Añadir autenticación o administración.
- Integrar un motor 3D o mapas vectoriales.

***

## Estructura de carpetas propuesta

```text
src/
  domain/
    entities/
    value-objects/
    repositories/
    services/
  application/
    use-cases/
    dto/
  infrastructure/
    repositories/
    mappers/
    data/
    visualization/
  presentation/
    components/
    pages/
    hooks/
    state/
```

### Ejemplo más concreto

```text
src/
  domain/
    entities/
      Building.ts
      Floor.ts
      Space.ts
    value-objects/
      SpaceId.ts
      FloorNumber.ts
      Coordinate2D.ts
    repositories/
      BuildingRepository.ts
  application/
    use-cases/
      GetBuildingFloors.ts
      GetFloorPlan.ts
      GetSpaceById.ts
      SearchSpaces.ts
  infrastructure/
    data/
      building-example.json
    repositories/
      JsonBuildingRepository.ts
    mappers/
      BuildingMapper.ts
    visualization/
      SvgFloorPlanAdapter.ts
  presentation/
    pages/
      BuildingExplorerPage.tsx
    components/
      FloorSelector.tsx
      FloorPlan.tsx
      SpaceDetailPanel.tsx
      SearchBar.tsx
```

***

## Tecnologías propuestas

## Lenguaje principal

- **TypeScript**

## Framework principal

- **React**

## Renderizado visual inicial

- **SVG interactivo**

## Testing

- **Vitest** o **Jest**
- **Testing Library** para pruebas de interfaz cuando sea necesario

## Gestión de estado

- Estado local de React al inicio
- Opcionalmente **Zustand** si la interfaz crece

## Estilos

- CSS Modules, Tailwind o CSS estándar según el nivel de complejidad visual

***

## Modelo de datos inicial

La primera versión puede trabajar con un archivo JSON como este:

```json
{
  "building": {
    "id": "building-x",
    "name": "Edificio X",
    "floors": [
      {
        "number": 0,
        "name": "Planta Baja",
        "spaces": [
          {
            "id": "aula-001",
            "name": "Aula 001",
            "type": "classroom",
            "coordinates": { "x": 120, "y": 80 }
          }
        ]
      }
    ]
  }
}
```

Este enfoque facilita una entrega incremental y evita depender de backend en la primera versión.

***

## Representación visual del edificio

La recomendación para esta primera fase es **no implementar un 3D real**, sino una representación visual suficientemente clara y mantenible.

### Opción recomendada

- Plano 2D por planta mediante **SVG interactivo**.

### Ventajas

- Más simple de implementar.
- Más fácil de probar.
- Más fácil de mantener.
- Más coherente con el alcance del proyecto.
- Permite evolucionar después hacia pseudo-3D o 3D real.
- No compromete la implementación de los casos de uso funcionales.

### Alternativas futuras

- Vista isométrica.
- Three.js.
- MapLibre GL con capas personalizadas.

***

## Casos de uso iniciales

### Lectura y exploración

- Ver edificio disponible.
- Ver listado de plantas.
- Seleccionar una planta.
- Consultar espacios de una planta.
- Ver detalle de un espacio.
- Buscar un espacio por nombre.
- Filtrar espacios por tipo.

### Evolución futura

- Calcular ruta entre dos espacios.
- Marcar favoritos.
- Administrar contenido del edificio.

> Estos casos de uso iniciales deben complementarse con los casos de uso reales del proyecto. La arquitectura final debe responder a todos ellos, no solo a los aquí ejemplificados.

***

## Interfaces y contratos

La aplicación debe depender de abstracciones, no de implementaciones concretas.

### Ejemplo de contrato de repositorio

```ts
export interface BuildingRepository {
  getBuilding(): Promise<Building>;
  getFloorByNumber(floorNumber: number): Promise<Floor | null>;
  getSpaceById(spaceId: string): Promise<Space | null>;
  searchSpaces(term: string): Promise<Space[]>;
}
```

### Ventaja

Con este contrato, la lógica de aplicación puede funcionar igual si los datos vienen de JSON, una API o una base de datos.

***

## Testing

La propuesta debe incluir **tests unitarios** desde el principio.

### Qué se debe probar

- Entidades del dominio.
- Value Objects.
- Casos de uso.
- Reglas del dominio.
- Mapeo básico de datos si introduce lógica relevante.
- Comportamientos asociados a los casos de uso del proyecto.

### Qué no debe ser prioritario al inicio

- Tests complejos de renderizado visual.
- Tests end-to-end si aún no existe una versión estable.

### Ejemplos de tests unitarios

- Crear una planta con espacios válidos.
- Verificar que un espacio pertenece a una única planta.
- Buscar espacios por nombre.
- Filtrar por tipo de punto de interés.
- Obtener correctamente una planta desde el repositorio.
- Validar reglas necesarias para cumplir casos de uso concretos.

### Herramientas propuestas

- **Vitest** para tests unitarios.
- **Testing Library** para interacciones simples de interfaz.

***

## Calidad y diseño

La implementación debe respetar estos principios:

- **Alta cohesión** en cada módulo.
- **Bajo acoplamiento** entre capas.
- **Responsabilidad única** por clase o componente.
- **Composición sobre herencia** cuando aporte más flexibilidad.
- **Ocultación de información**.
- **Código orientado al dominio**.
- **Evitar code smells** como God Class, Long Method, Primitive Obsession o Duplicated Code.
- **Trazabilidad entre cada módulo y los casos de uso que soporta**.

***

## Implementación incremental propuesta

### Fase 1

- Definir entidades del dominio.
- Definir contratos de repositorio.
- Identificar y mapear todos los casos de uso del proyecto.
- Crear una matriz de trazabilidad caso de uso → módulo.
- Crear JSON de ejemplo.
- Implementar repositorio sobre JSON.
- Implementar casos de uso básicos.

### Fase 2

- Crear interfaz web en React.
- Renderizar plano SVG por planta.
- Implementar selector de planta.
- Mostrar detalle de espacio.
- Añadir búsqueda y filtros.
- Incorporar flujos necesarios para cubrir los casos de uso restantes.

### Fase 3

- Añadir tests unitarios completos.
- Verificar cobertura funcional de todos los casos de uso.
- Refactorizar según principios SOLID.
- Mejorar la estructura modular.
- Preparar posibles extensiones.

### Fase 4

- Evaluar evolución a pseudo-3D.
- Evaluar backend o persistencia remota.
- Añadir rutas internas si el proyecto lo requiere.
- Escalar del edificio ejemplo a múltiples edificios si fuera necesario.

***

## Decisiones arquitectónicas

- **Aplicación web** en lugar de móvil para esta primera versión.
- **Edificio de ejemplo** en lugar de modelar toda la universidad visualmente.
- **React + TypeScript** como stack principal.
- **Arquitectura por capas con enfoque Clean Architecture**.
- **Datos en JSON local** para reducir complejidad inicial.
- **SVG interactivo** en vez de 3D real en la primera fase.
- **Tests unitarios** sobre dominio y casos de uso.
- **Cobertura explícita de todos los casos de uso del proyecto** como criterio de validación.
- **Preparación para evolución futura** sin reescribir el núcleo del sistema.

***

## Conclusión técnica

La propuesta técnica para **myUniverse** debe centrarse en una solución **web, modular y evolutiva**, donde el valor principal esté en el diseño de software y en la claridad arquitectónica. En lugar de abordar desde el inicio la complejidad de un campus completo o de un renderizado 3D avanzado, esta arquitectura permite construir una base sólida sobre un edificio representativo, validando los conceptos del sistema con una implementación realista, mantenible y justificable.

El edificio de ejemplo debe entenderse únicamente como una simplificación del escenario visual, no del comportamiento esperado. La arquitectura final debe seguir siendo capaz de soportar **todos los casos de uso especificados en el proyecto**, manteniendo trazabilidad entre requisitos, módulos y decisiones técnicas.