# Diseño Orientado a Objetos — Guía de estudio

Este documento recoge y explica todos los temas de la carpeta DOO, organizados en orden pedagógico.

## Índice

1. [OCP — Principio Abierto/Cerrado](#ocp--principio-abiertocerado)
2. [DD — Doble Despacho y Patrón Visitor](#dd--doble-despacho-y-patrón-visitor)
3. [LSP — Principio de Sustitución de Liskov](#lsp--principio-de-sustitución-de-liskov)
4. [pMP — Patrón Método Plantilla](#pmp--patrón-método-plantilla)
5. [csHR — Código Sucio por Herencia Rechazada](#cshr--código-sucio-por-herencia-rechazada)
6. [Relación entre todos los temas](#relación-entre-todos-los-temas)

---

## OCP — Principio Abierto/Cerrado

### ¿Qué dice el principio?

> Un módulo debe estar **abierto a extensión** pero **cerrado a modificación**.

Cuando llegan nuevos requisitos, deberías poder añadir comportamiento sin tocar el código que ya funciona.

### El hilo del repositorio (OCP01 → OCP07)

El tema se desarrolla a través de un sistema de matrícula universitaria que crece con el tiempo:

#### OCP01 — Modelo base

`Alumno` + `Universidad` + `Cliente`. Diseño simple, cohesionado y con jerarquías bien formadas. El cliente no necesita cambiar si se extiende el sistema.

#### OCP02 y OCP03 — Primeras extensiones

Aparecen `AlumnoErasmus` y `AlumnoVirtual`. El sistema se extiende correctamente mediante herencia: cada nuevo tipo añade su propia lógica sin modificar lo existente. El cliente sigue sin cambiar (salvo para crear los nuevos objetos).

> Hasta aquí: cohesión, acoplamiento y tamaño bien gestionados.

#### OCP04 — El problema: el evaluador de becas

Llega un nuevo requisito: un evaluador de becas que debe comportarse de forma diferente según el tipo de alumno. La solución más directa es esta:

```java
void evaluar(Alumno a) {
    if (a instanceof AlumnoErasmus)  { /* lógica Erasmus */ }
    else if (a instanceof AlumnoVirtual) { /* lógica Virtual */ }
    else { /* lógica alumno normal */ }
}
```

**Problema:** cada nuevo tipo de alumno obliga a abrir y modificar este método.

#### OCP04Alternativa — Jerarquía de evaluadores

Se especializa el evaluador en subclases (`EvaluadorErasmus`, `EvaluadorVirtual`...) para mejorar la cohesión. Pero el problema de fondo sigue: si añadimos un tipo de alumno nuevo, hay que tocar **todos** los evaluadores existentes.

#### OCP05 — Doble despacho

En lugar de que el evaluador identifique el tipo del alumno, el alumno se pasa a sí mismo al evaluador:

```java
// El alumno delega: "recíbeme, soy yo"
class AlumnoErasmus extends Alumno {
    void solicitarBeca(EvaluadorBecas e) {
        e.evaluar(this);   // this tiene tipo estático AlumnoErasmus
    }
}
```

El evaluador recibe el tipo exacto sin preguntar. El polimorfismo resuelve el despacho.

#### OCP05Extendido — La solución escala

Con 5 tipos de alumno y 4 evaluadores: **cero `instanceof`**. Añadir un nuevo tipo de alumno o un nuevo evaluador no requiere tocar lo existente.

#### OCP06 — El contrato formal

Con el sistema estable, se formaliza el contrato del método `solicitarBeca(evaluador)`:

| | Descripción |
|-|-|
| **Precondición** | `evaluador` no es nulo y es válido |
| **Postcondición** | se invoca exactamente una vez `evaluador.evaluar(this)` con el tipo concreto |
| **Invariante** | cualquier subtipo de `Alumno` puede procesarse en `Universidad` sin condiciones adicionales |

> Este mecanismo tiene nombre formal: **patrón Visitor**. El contrato que acabamos de definir es exactamente lo que el patrón exige.

#### OCP07 — La fractura: AlumnoHonorario

Un nuevo tipo: `AlumnoHonorario`, que no debería solicitar beca. La implementación "inocente":

```java
class AlumnoHonorario extends Alumno {
    @Override
    public void solicitarBeca(EvaluadorBecas evaluador) {
        // los alumnos honorarios no solicitan becas
    }
}
```

Compila. No explota. Pero **viola el contrato**: la postcondición dice que `evaluar` debe invocarse una vez, y aquí no se invoca nunca. El evaluador nunca sabe que ese alumno existió.

Esto conecta directamente con **LSP**: no basta con compilar. El subtipo debe respetar las garantías observables del tipo base.

**Dos síntomas posibles:**
- **Silencioso:** no hace nada, el fallo es invisible sin instrumentación.
- **Visible:** lanza `UnsupportedOperationException`, y alguien añade un `instanceof` en `Universidad`, rompiendo el desacoplamiento logrado.

**Tres salidas:**

| Opción | Descripción |
|-|-|
| Sacar `AlumnoHonorario` de la jerarquía | Si no cumple el contrato, quizá no pertenece a ella |
| Renegociar el contrato base | `solicitarBeca()` pasa a ser no-op por defecto, pero debilita la garantía para todos |
| Composición | La capacidad de solicitar beca se extrae como pieza componible; `AlumnoHonorario` recibe una implementación nula sin violar el contrato |

---

## DD — Doble Despacho y Patrón Visitor

### ¿Qué problema resuelve?

Cuando el comportamiento depende del tipo de **dos** objetos que interactúan, el polimorfismo simple no es suficiente:

```java
// El evaluador no sabe qué tipo concreto recibe
void evaluar(Alumno a) { ??? }
```

Java solo resuelve el tipo del objeto en el que se llama el método (primer objeto). Para resolver el tipo del argumento hay que usar `instanceof`... o doble despacho.

### Cómo funciona el doble despacho

```java
// Primer despacho: polimorfismo sobre el objeto receptor
class AlumnoErasmus extends Alumno {
    void solicitarBeca(EvaluadorBecas e) {
        e.evaluar(this);   // segundo despacho: this es AlumnoErasmus
    }
}

// Segundo despacho: sobrecarga resuelta en tiempo de compilación
class Evaluador implements EvaluadorBecas {
    void evaluar(AlumnoErasmus a) { /* específico Erasmus */ }
    void evaluar(AlumnoVirtual a) { /* específico Virtual */ }
}
```

Hay **dos saltos**: el primero elige qué `solicitarBeca()` invocar según el subtipo de alumno; el segundo elige qué sobrecarga de `evaluar()` invocar porque `this` ya tiene tipo estático concreto.

### Los cuatro ejemplos del repositorio

#### DD00 — El restaurante

Un `Recepcionista` saluda y un `Camarero` sirve a personas (`Hombre`, `Mujer`, `Androide`).

| Versión | Técnica | Problema que queda |
|-|-|-|
| [v000mal](DD/DD00/v000mal) | `instanceof` + casting | Cada nuevo tipo obliga a tocar Recepcionista y Camarero |
| [v001basico](DD/DD00/v001basico) | Doble despacho manual | `Persona` acoplada a `Recepcionista` concreto |
| [v002extensible](DD/DD00/v002extensible) | Interfaz `VisitadorPersona` (Visitor) | Diseño limpio y extensible |
| [v003malExtendida](DD/DD00/v003malExtendida) | Vuelta al `instanceof` al añadir `Androide` | Demuestra la fragilidad al no usar el patrón |

**El patrón Visitor aplicado:**
```java
// Interfaz común para todos los que interactúan con personas
public interface VisitadorPersona {
    void visitar(Hombre h);
    void visitar(Mujer m);
}

// Recepcionista y Camarero implementan la interfaz
public class Recepcionista implements VisitadorPersona { ... }
public class Camarero      implements VisitadorPersona { ... }

// Persona solo conoce la interfaz, no clases concretas
public abstract void aceptar(VisitadorPersona v);
```

#### DD01 — El zoológico

Un `Cuidador` alimenta y un `Veterinario` examina animales (`Perro`, `Pajaro`). Misma evolución de tres versiones. El patrón Visitor generaliza el mecanismo para que múltiples visitantes (`Cuidador`, `Veterinario`, `Entrenador`...) coexistan sin tocar la jerarquía `Animal`.

#### DD02 — La aspiradora

Una aspiradora autónoma y un gato interactúan con celdas de tres tipos (`CeldaLimpia`, `CeldaSucia`, `CeldaObstaculo`).

| Versión | Técnica | Aspectos destacados |
|-|-|-|
| [v000mal](DD/DD02/v000mal) | `instanceof` en `Aspiradora.limpiar()` y `Gato.ensuciar()` | Acoplamiento a todos los tipos de celda |
| [v001basico](DD/DD02/v001basico) | `celda.aceptarAspiradora(this)` | Doble despacho sin interfaz común |
| [v002extensible](DD/DD02/v002extensible) | `VisitanteCelda` + `celda.aceptar(visitante)` | `Aspiradora` y `Gato` implementan la misma interfaz |

#### DD03 — Sistema de pedidos

Tres tipos de pedido (`PedidoFisico`, `PedidoDigital`, `PedidoSuscripcion`) con tres procesadores distintos (`ProcesadorEnvios`, `ProcesadorFacturacion`, `ProcesadorNotificaciones`).

```java
// v002extensible: interfaz VisitantePedido
public interface VisitantePedido {
    void visitar(PedidoFisico pedido);
    void visitar(PedidoDigital pedido);
    void visitar(PedidoSuscripcion pedido);
}
```

Añadir `ProcesadorAnalytics` no requiere modificar nada existente. Añadir `PedidoTarjetaRegalo` sí requiere añadir un método a la interfaz e implementarlo en cada procesador existente.

> **Regla del Visitor:** es la herramienta correcta cuando las **operaciones crecen** y los **tipos son estables**. Si los tipos cambian frecuentemente, el coste se invierte.

### Comparativa de versiones (aplica a todos los ejemplos DD)

| Aspecto | v000mal | v001basico | v002extensible |
|:-:|:-:|:-:|:-:|
| `instanceof` | Sí | No | No |
| Extensible a nuevos tipos | No | Parcial | Sí* |
| Extensible a nuevos visitantes | No | Difícil | Fácil |
| Acoplamiento | Alto | Medio | Bajo |
| Complejidad | Baja | Media | Media-Alta |

*Con coste lineal en número de visitantes al añadir un tipo nuevo.

---

## LSP — Principio de Sustitución de Liskov

### ¿Qué dice el principio?

> Si `S` es subtipo de `T`, cualquier objeto de tipo `T` puede reemplazarse por un objeto de tipo `S` **sin alterar el comportamiento observable del programa**.

No basta con que el código compile. El subtipo debe respetar el **contrato completo** del tipo base: precondiciones, postcondiciones e invariantes.

### Las tres formas de violarlo

| Tipo de violación | Qué se rompe | Consecuencia para el cliente |
|-|-|-|
| **Precondición más estricta** | La subclase rechaza entradas que la base aceptaba | El cliente deja de funcionar sin saber por qué |
| **Postcondición más débil** | La subclase entrega menos de lo prometido | El cliente recibe resultados inesperados |
| **Invariante alterada** | La subclase rompe una propiedad siempre válida | El estado del objeto queda incoherente |

### Regla de oro

> **Pre:** un subtipo puede **debilitar** precondiciones (aceptar más entradas), nunca reforzarlas.  
> **Post:** un subtipo puede **reforzar** postcondiciones (entregar más), nunca debilitarlas.

### LSP00 — Sistema bancario

El contrato de `CuentaBancaria`:
- Depósitos positivos aceptados.
- Retiros positivos reducen el saldo exactamente en ese monto.
- El saldo nunca queda negativo.

| Subclase | Violación | Descripción |
|-|-|-|
| `CuentaLimitada` | Precondición más estricta | Rechaza retiros tras alcanzar un límite, aunque el saldo alcance |
| `CuentaComision` | Postcondición más débil | Deduce más del monto indicado (añade comisión silenciosamente) |
| `CuentaSobregiro` | Invariante alterada | Permite saldo negativo |

Un cliente que asuma el contrato de `CuentaBancaria` se rompe con cualquiera de estas subclases:

```java
// El cliente asume que tras retirar 100, el saldo bajó exactamente 100
cuenta.retirar(100);
assert saldoAntes - 100 == cuenta.getSaldo(); // falla con CuentaComision
```

**Solución:** extraer las políticas como interfaz independiente:

```java
interface PoliticaRetiro {
    void retirar(CuentaBancaria cuenta, double monto);
}

class PoliticaConComision implements PoliticaRetiro { ... }
class PoliticaConLimite    implements PoliticaRetiro { ... }
```

Cada política encapsula su comportamiento sin corromper el contrato de `CuentaBancaria`.

### LSP01 — Refactorización con políticas

Versión refactorizada del banco usando `PoliticaRetiro`. El cliente opera sobre `CuentaBancaria` y las variaciones de comportamiento se inyectan como políticas, sin herencia problemática.

### LSP03 — Procesador de pedidos

El contrato base: 10% de descuento, garantía de que el resultado será mínimo el 50% del importe original.

| Procesador | ¿Cumple LSP? | Por qué |
|:-:|-|-|
| `ProcesadorPremium` | ✅ | Postcondición más cerrada: da más descuento del mínimo garantizado |
| `ProcesadorPromocion` | ✅ | Precondición más abierta: acepta importe 0, más permisivo que la base |
| `ProcesadorDeudores` | ❌ | Postcondición más débil: devuelve `importe + recargo 20%`, menos de lo prometido |
| `ProcesadorMayorista` | ❌ | Precondición más estricta: rechaza importes < 75€, que la base aceptaría |

### LSP04 — El contrato roto invisible (conexión con OCP07)

`AlumnoHonorario` no invoca al evaluador. Sin instrumentación, nadie lo detecta. Si se cuenta cuántos alumnos entran en `procesarSolicitudBeca` y cuántas veces se invoca al evaluador, los números no cuadran. Es la unión entre LSP y el doble despacho de OCP.

### Diferencia entre violación visible y silenciosa

| Tipo | ¿Cómo se manifiesta? | ¿Cómo detectarla? |
|-|-|-|
| **Visible** | Excepción (`UnsupportedOperationException`) | En la primera ejecución afectada |
| **Silenciosa** | El sistema funciona pero produce resultados incorrectos | Solo con tests o instrumentación |

---

## pMP — Patrón Método Plantilla

### ¿Qué problema resuelve?

Varias clases derivadas cuyos métodos siguen el mismo esquema general, pero con algunos pasos internos distintos. Si cada una implementa el método completo, hay duplicación y la estructura común desaparece.

### La solución

Definir en la clase padre el **esqueleto del algoritmo** (el "método plantilla"), y delegar a las hijas únicamente los pasos que varían.

```
ClasePadre
│
├── metodoPlantilla()  ← final: define el esqueleto, no se sobreescribe
│     paso1Comun()
│     pasoVariable()   ← abstracto o con default
│     paso2Comun()
│
└── pasoVariable()     ← abstracto: cada hija lo implementa

ClaseHija extends ClasePadre
└── pasoVariable()     ← implementación concreta
```

### Ejemplo: proyecto pyKlondike

| | Sin método plantilla | Con método plantilla |
|:-:|:-:|:-:|
| Código | [ver](https://github.com/mmasias/23-24-pyKlondike/tree/449affaad73a3e49b27783e1c24488011c03a1ec/src) | [ver](https://github.com/mmasias/23-24-pyKlondike/tree/d66842e18ebf1473c12323aed6c4869dbb99e4da/src) |
| `mostrar()` | en cada clase derivada (duplicado) | en la clase padre (una sola vez) |
| `mostrarContenido()` | mezclado dentro de `mostrar()` | en cada clase derivada |

**Antes — sin patrón:**
```java
class CartaRoja {
    void mostrar() {
        // cabecera...  (duplicada en cada clase)
        System.out.println("♥ " + valor);
        // pie...  (duplicado en cada clase)
    }
}
class CartaNegra {
    void mostrar() {
        // cabecera...  (misma que arriba)
        System.out.println("♠ " + valor);
        // pie...  (mismo que arriba)
    }
}
```

**Después — con patrón:**
```java
abstract class Carta {
    final void mostrar() {           // esqueleto en el padre, no sobreescribible
        cabeceraComun();
        mostrarContenido();          // paso variable
        pieComun();
    }
    abstract void mostrarContenido(); // cada hija define esto
}

class CartaRoja extends Carta {
    void mostrarContenido() { System.out.println("♥ " + valor); }
}
class CartaNegra extends Carta {
    void mostrarContenido() { System.out.println("♠ " + valor); }
}
```

### Claves para recordar

| Aspecto | Detalle |
|-|-|
| **Dónde vive el algoritmo** | En la clase padre |
| **Qué sobreescriben las hijas** | Solo los pasos variables, nunca el esqueleto |
| **`final` en Java** | Evita que una hija sobreescriba el método plantilla |
| **Relación usada** | Herencia (no composición) |
| **Patrón relacionado** | Strategy — si los pasos variables cambian en tiempo de ejecución |

### Cuándo usarlo vs. cuándo no

| Cuándo usarlo | Cuándo no usarlo |
|-|-|
| El flujo es fijo, solo varían algunos pasos | Los pasos variables cambian en tiempo de ejecución → **Strategy** |
| Quieres eliminar duplicación en subclases | Las clases no comparten una jerarquía natural → **composición** |
| El número de variantes es estable | Añadir herencia solo para esto añade rigidez innecesaria |

### Pregunta de reflexión

> ¿Qué pasa si `metodoPlantilla()` no se marca `final`?

Una hija puede sobreescribir el esqueleto completo, rompiendo la garantía de estructura compartida. Volvemos al punto de partida: código duplicado sin estructura común garantizada.

---

## csHR — Código Sucio por Herencia Rechazada

### ¿Qué es la herencia rechazada?

Ocurre cuando una subclase **hereda métodos que no debería tener** porque no aplican a su naturaleza, aunque estén en la clase base.

### El ejemplo

`Alumno` implementa:
- `registrarCalificaciones()`
- `consultarPromedioAcademico()`
- `inscribirAsignaturas()`

`AlumnoErasmus` y `AlumnoVirtual` los heredan automáticamente, aunque para ellos no tengan sentido.

### Las opciones (y sus problemas)

| Opción | Problema |
|-|-|
| Dejar el método vacío | Violación silenciosa del contrato. El cliente llama y no pasa nada |
| Lanzar `UnsupportedOperationException` | Violación visible. Rompe LSP: el tipo base prometía ese comportamiento |
| Crear clase intermedia `AlumnoRegular` | Puede complicar la jerarquía en exceso |

### La solución preferida: composición/delegación

En lugar de heredar lo que no se necesita, las clases consumen únicamente los servicios que les corresponden:

```java
class ServicioCalificaciones {
    void registrar(...) { ... }
    double consultarPromedio() { ... }
}

class ServicioInscripcion {
    void inscribir(...) { ... }
}

class AlumnoRegular {
    private ServicioCalificaciones calificaciones; // solo este tipo lo usa
    private ServicioInscripcion inscripcion;
}

class AlumnoErasmus {
    // no tiene ServicioCalificaciones → no hereda lo que no necesita
}
```

### Señal de alerta

Si una subclase sobreescribe un método de la base con cuerpo vacío o con `throw new UnsupportedOperationException()`, casi seguro es herencia rechazada.

### Compromiso documentado

Si la herencia rechazada es una implementación vacía de un método que **nunca se usará en esa subclase**, puede tolerarse frente a la complejidad de refactorizar la jerarquía.

Pero si son **métodos públicos implementados que no deberían estar en la interfaz pública de la subclase**, hay que refactorizar hacia composición para no corromper la interfaz.

---

## Relación entre todos los temas

```
OCP (abierto/cerrado)
│
├── evitar instanceof
│     └── DD (doble despacho) ──────→ formalizado como patrón Visitor
│
├── contratos de jerarquía correctos
│     └── LSP (sustitución de Liskov)
│               │
│               └── fractura silenciosa (OCP07 / LSP04)
│
└── la jerarquía puede crecer mal
      └── csHR (herencia rechazada)
                └── solución: composición / delegación

pMP (método plantilla) → complementa OCP al definir esqueletos extensibles
                         con puntos de extensión controlados
```

### Pregunta que responde cada tema

| Tema | Pregunta central |
|-|-|
| **OCP** | ¿Puedo añadir comportamiento sin modificar lo que ya existe? |
| **DD / Visitor** | ¿Cómo evito `instanceof` cuando el comportamiento depende del tipo de dos objetos? |
| **LSP** | ¿Mi subclase puede sustituir a la base sin romper el comportamiento observable? |
| **pMP** | ¿Cómo comparto la estructura de un algoritmo evitando duplicación en cada subclase? |
| **csHR** | ¿Mi jerarquía hereda cosas que no le corresponden? ¿Necesito composición? |

## Conclusión: qué es SOLID

SOLID no es un patrón ni una técnica concreta, sino un conjunto de cinco principios de diseño orientado a objetos que ayudan a construir sistemas más fáciles de extender, mantener y entender.

- **S**: Single Responsibility Principle. Cada clase debe tener una sola responsabilidad clara.
- **O**: Open/Closed Principle. El software debe poder extenderse sin modificar lo ya existente.
- **L**: Liskov Substitution Principle. Un subtipo debe poder sustituir a su tipo base sin romper el comportamiento esperado.
- **I**: Interface Segregation Principle. Es mejor tener interfaces pequeñas y específicas que una interfaz grande y obligatoria para todos.
- **D**: Dependency Inversion Principle. El código debe depender de abstracciones, no de clases concretas.

En los temas de esta carpeta se ve SOLID aplicado de forma práctica:

- **OCP** explica cómo crecer sin tocar lo que ya funciona.
- **DD** muestra cómo evitar `instanceof` y organizar interacciones entre objetos.
- **LSP** recuerda que no basta con heredar: hay que respetar contratos.
- **pMP** enseña a compartir estructura sin duplicar código.
- **csHR** advierte cuándo la herencia deja de ser una buena idea y conviene usar composición.

En resumen, SOLID es una guía para tomar decisiones de diseño que reduzcan acoplamiento, eviten rigidez y hagan que el código soporte cambios futuros con menos riesgo.
