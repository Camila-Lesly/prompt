# Hilo secuencial: doble despacho en OCP

Perdonar... no es sino hasta ahora que tengo un momento libre para poder escribir esto que les dije que lo publicaría, con el hilo secuencial del tema de doble despacho, que está en el repo pero no lo tengo aún enlazado en la secuencia que quiero que quede finalmente.

## Índice de contenidos

1. [Contexto e inicio](#contexto-e-inicio)
2. [OCP01: modelo base](#ocp01-modelo-base)
3. [OCP02 y OCP03: primeras extensiones](#ocp02-y-ocp03-primeras-extensiones)
4. [OCP04: aparece el evaluador de becas](#ocp04-aparece-el-evaluador-de-becas)
5. [OCP04Alternativa: jerarquía de evaluadores](#ocp04alternativa-jerarquía-de-evaluadores)
6. [OCP05 y OCP05Extendido: doble despacho](#ocp05-y-ocp05extendido-doble-despacho)
7. [OCP06: el contrato](#ocp06-el-contrato)
8. [OCP07: requisitos++ y fractura](#ocp07-requisitos-y-fractura)
9. [Cierre](#cierre)

## Resumen breve por apartado

### Contexto e inicio
Se presenta la evolución de una solución orientada a objetos para una universidad y cómo, al crecer los requisitos, surge la necesidad de aplicar doble despacho.

### OCP01: modelo base
En [OCP01](https://github.com/mmasias/idsw2/tree/main/src/DOO/OCP/OCP01) se modela la matrícula con `Alumno` y `Universidad`, y un cliente que consume ambas clases con una estructura simple y cohesionada.

### OCP02 y OCP03: primeras extensiones
En [OCP02](https://github.com/mmasias/idsw2/tree/main/src/DOO/OCP/OCP02) se añade `AlumnoErasmus` y en [OCP03](https://github.com/mmasias/idsw2/tree/main/src/DOO/OCP/OCP03) `AlumnoVirtual`, manteniendo un diseño modular y evitando cambios en el cliente más allá de la creación de objetos.

### OCP04: aparece el evaluador de becas
En [OCP04](https://github.com/mmasias/idsw2/tree/main/src/DOO/OCP/OCP04) se incorpora la evaluación de becas, pero el evaluador distingue tipos de alumno con `instanceof`, introduciendo rigidez y riesgo de cambios repetitivos.

### OCP04Alternativa: jerarquía de evaluadores
En [OCP04Alternativa](https://github.com/mmasias/idsw2/tree/main/src/DOO/OCP/OCP04Alternativa) se especializan evaluadores para mejorar cohesión, aunque sigue el problema central: cada nuevo tipo de alumno obliga a tocar múltiples métodos.

### OCP05 y OCP05Extendido: doble despacho
En [OCP05](https://github.com/mmasias/idsw2/tree/main/src/DOO/OCP/OCP05) y con mayor fuerza en [OCP05Extendido](https://github.com/mmasias/idsw2/tree/main/src/DOO/OCP/OCP05Extendido), el alumno delega en el evaluador su recepción según su tipo, eliminando `instanceof` y apoyándose en polimorfismo.

### OCP06: el contrato
En [OCP06](https://github.com/mmasias/idsw2/tree/main/src/DOO/OCP/OCP06) se formaliza el contrato de `Alumno.solicitarBeca(evaluador)`: precondiciones, postcondiciones e invariante de jerarquía que sostienen el doble despacho.

### OCP07: requisitos++ y fractura
En [OCP07](https://github.com/mmasias/idsw2/tree/main/src/DOO/OCP/OCP07) entra `AlumnoHonorario`, que no solicita beca y rompe el contrato heredado; aquí aparece explícitamente el problema de sustitución (LSP), incluso sin `instanceof` en la arquitectura original.

### Cierre
La secuencia muestra el paso de una extensión lineal a una arquitectura más abierta al cambio, donde el doble despacho reduce impacto ante nuevas variantes, pero también exige contratos de jerarquía correctos para no romper sustitución.

## Contexto e inicio

Todo empieza con una solución que aborda el modelado de una universidad que matricula alumnos.

## OCP01: modelo base

En [OCP01](https://github.com/mmasias/idsw2/tree/main/src/DOO/OCP/OCP01) hay una clase `Alumno` y una clase `Universidad`, clases que son usadas por un cliente que implementa a partir de ellas.

## OCP02 y OCP03: primeras extensiones

A partir de allí, dado que el sistema tiene acogida, empiezan a solicitarnos más requisitos y claro, como desarrolladores empezamos a extender el sistema:

- Nos dicen que además de alumnos, tenemos alumnos Erasmus ([OCP02](https://github.com/mmasias/idsw2/tree/main/src/DOO/OCP/OCP02)).
- Y tras los alumnos Erasmus, alumnos Virtuales ([OCP03](https://github.com/mmasias/idsw2/tree/main/src/DOO/OCP/OCP03)).

Todo esto, hasta este punto, resuelto con un exquisito apego a los principios de diseño modular: cohesión, acoplamiento, tamaño... y con un manejo de las jerarquías tal que hacen innecesario que el cliente tenga que cambiar (más allá del natural ajuste al momento de crear alumnos, como se puede ver en cada `Cliente.java` de estos directorios).

## OCP04: aparece el evaluador de becas

Pero a partir del siguiente requisito las cosas se complican.

Ahora lo que nos piden es la entrada de un evaluador de becas, que debe adecuar su comportamiento según si quien llega es alumno Erasmus, alumno virtual o alumno normal.

El primer ajuste ([OCP04](https://github.com/mmasias/idsw2/tree/main/src/DOO/OCP/OCP04)) involucra ajustar `Universidad` para que incluya este proceso de evaluación de beca e introducir un evaluador de becas. Y este evaluador de becas tendrá que atender a los alumnos. ¿Cómo lo hace? Con esta arquitectura no hay otra forma de hacerlo que usar `instanceof` para identificar el tipo de alumno que recibe.

## OCP04Alternativa: jerarquía de evaluadores

La segunda propuesta ([OCP04Alternativa](https://github.com/mmasias/idsw2/tree/main/src/DOO/OCP/OCP04Alternativa)) introduce una jerarquía de evaluadores, intentando mitigar el problema de cohesión que introducía una única clase abordando dos tipos de evaluador. Pero esto, aunque especializa el evaluador, no resuelve el verdadero problema: en caso de entrar un nuevo tipo de alumno, se tendrá que tocar un único (y además mismo) método de todas y cada una de las clases que tengan que desempeñar esa función.

## OCP05 y OCP05Extendido: doble despacho

El doble despacho es la técnica que hace que, en lugar de decirle al evaluador que atienda al alumno, lo que se hace es decirle al alumno que le diga al evaluador que le reciba, pasándose él mismo.

Con esto se evita el uso de `instanceof` y apoyamos la solución en el polimorfismo. Es la solución que se explora en [OCP05](https://github.com/mmasias/idsw2/tree/main/src/DOO/OCP/OCP05) y que brilla en [OCP05Extendido](https://github.com/mmasias/idsw2/tree/main/src/DOO/OCP/OCP05Extendido), donde se modelan 5 especializaciones de alumno y 4 evaluadores.

## OCP06: el contrato

Con el sistema ya estable, en [OCP06](https://github.com/mmasias/idsw2/tree/main/src/DOO/OCP/OCP06) se hace explícito el contrato del método `solicitarBeca(evaluador)` en la jerarquía `Alumno`.

Se definen precondiciones (evaluador válido), postcondiciones (invocar exactamente una evaluación con el tipo concreto del alumno) e invariante de jerarquía (cualquier subtipo debe poder procesarse sin condiciones adicionales en `Universidad`).

Este paso es clave porque hace visible lo que el diseño por doble despacho necesita para mantenerse correcto al crecer el sistema.

## OCP07: requisitos++ y fractura

En [OCP07](https://github.com/mmasias/idsw2/tree/main/src/DOO/OCP/OCP07) aparece un nuevo tipo: `AlumnoHonorario`, que no debería solicitar beca.

Si su implementación de `solicitarBeca` no hace nada (o lanza excepción), el contrato definido en OCP06 se rompe: el evaluador deja de ser invocado como el sistema espera. El resultado puede ser silencioso o visible, pero en ambos casos hay fractura de sustitución.

Aquí se hace explícita la conexión con LSP: no basta con compilar ni con evitar `instanceof`; cada subtipo debe respetar las garantías observables del tipo base.

## Cierre

El hilo completo deja clara la evolución: de una solución extensible para tipos de alumno, a una necesidad de variación en dos jerarquías (alumno y evaluador), resuelta con doble despacho y reforzada después por contratos formales.

OCP06 y OCP07 añaden una conclusión importante: un diseño abierto a extensión no es suficiente si los subtipos no preservan el contrato de comportamiento esperado.