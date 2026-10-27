# Semana 11 — Recursion y Merge Sort
## COEN 2220 — Programming 2

**Instructor:** Wilson Lozano

**Duración:** 170 min (sesión de 3 horas crédito nominales)  
**Precede a:** Lab 11 — Recursion y Merge Sort + aplicación al proyecto  
**Lectura complementaria:** Gaddis, *Starting Out with C++: From Control Structures through Objects*, 8va ed., Cap. 19 — secciones 19.1–19.2 (Introduction to Recursion; Solving Problems with Recursion) y 19.10 (Recursion vs. Iteration). La sección 19.6 permite retomar binary search en la próxima sesión. **Merge sort se presenta como contenido complementario del curso; no aparece desarrollado en este capítulo de la 8va edición.**

---

## Objetivos

Al finalizar esta sesión, el estudiante podrá:

1. Definir recursión e identificar el caso base, el caso recursivo y el avance hacia la terminación.
2. Trazar llamadas y retornos, distinguiendo los parámetros locales de cada llamada y el trabajo pendiente de quien la llamó.
3. Explicar cómo merge sort divide un segmento, ordena sus mitades y combina dos segmentos ya ordenados.
4. Implementar y probar merge sort con arrays y explicar su adaptación a linked lists mediante separación y reconexión de nodes.
5. Justificar el tiempo `O(n log n)`, la estabilidad y el espacio auxiliar de las versiones con arrays y linked lists.
6. Relacionar el campo de ordenamiento con el requisito de binary search y reconocer que ordenar registros requiere moverlos completos.

## Parte 0 — Puente: el call stack conserva trabajo pendiente (10 min)

En la Semana 10 estudiaron un stack: una colección que sigue LIFO, *last in, first out*. El último elemento que entra es el primero que sale. También vieron que un programa usa un **call stack** para conservar información de las funciones cuya ejecución todavía no termina.

Supongan que un programa prepara un informe. `main` llama a una función `prepareReport`, y esta llama a `formatHeading`. Mientras se ejecuta `formatHeading`, `prepareReport` espera; cuando termina, el programa regresa al punto de `prepareReport` inmediatamente posterior a esa llamada. Después, cuando termina `prepareReport`, regresa a `main`.

Cada llamada activa necesita conservar información como sus parámetros, variables locales y el punto donde debe continuar al regresar. A ese conjunto de información lo llamaremos **stack frame**, o registro de una llamada. Dos llamadas a la misma función pueden tener registros distintos, igual que dos llamadas a funciones diferentes.

```text
Call order:   main -> prepareReport -> formatHeading
Return order: formatHeading -> prepareReport -> main
```

Hoy utilizaremos ese mismo mecanismo para entender una función que se llama a sí misma. No tendremos que crear una clase stack ni ejecutar `push` o `pop` manualmente: el mecanismo de llamadas del programa conserva ese trabajo pendiente.

## Parte 1 — Recursión: resolver una versión más pequeña del problema (25 min)

### Qué es y cuándo puede ayudar

La **recursión** es una técnica en la que una función resuelve un problema llamándose a sí misma para resolver una versión más pequeña del mismo problema. En esta sesión usaremos recursión directa: el nombre de la función aparece dentro de su propia definición.

Es útil cuando podemos describir una tarea mediante tareas similares más pequeñas. Por ejemplo, sumar cinco mediciones puede expresarse como “la quinta medición más la suma de las cuatro anteriores”. La suma de cuatro se reduce de la misma forma a la suma de tres. Para que esto termine, necesitamos una situación que pueda resolverse sin otra llamada.

Una solución recursiva necesita tres decisiones:

- **Caso base:** una situación que se resuelve directamente, sin volver a llamar la función.
- **Caso recursivo:** cómo se utiliza una llamada sobre un problema menor para resolver el actual.
- **Avance hacia el caso base:** qué cambia para que no se repita indefinidamente el mismo problema.

### Ejemplo: sumar las duraciones de cinco pruebas

Un laboratorio registra cuánto tardan sus pruebas de calibración. Antes de preparar el informe del día, necesita sumar las duraciones almacenadas en un array. Usaremos cinco valores pequeños, `5`, `8`, `3`, `6` y `4` minutos, para observar las llamadas y los resultados. La función `sumTestMinutes` recibirá el array y la cantidad de posiciones iniciales que debe sumar.

Si la cantidad es cero, la suma es cero: ese es el caso base. Si la cantidad es cinco, la función conserva la quinta duración, `4`, y solicita la suma de las cuatro anteriores. Al recibir ese resultado, agrega la duración que había conservado.

El siguiente programa es completo. Sus prototipos aparecen antes de `main`; las definiciones aparecen después de `main`. Son funciones libres, por lo que no pertenecen a una clase ni necesitan el operador `::`.

```cpp
#include <iostream>
using namespace std;

int sumTestMinutes(const int testMinutes[], int count);
int sumTestMinutesIterative(const int testMinutes[], int count);

int main()
{
    const int TEST_COUNT = 5;
    int testMinutes[TEST_COUNT] = {5, 8, 3, 6, 4};

    int recursiveTotal = sumTestMinutes(testMinutes, TEST_COUNT);
    cout << "Recursive total: " << recursiveTotal << endl;
    cout << "Iterative total: "
         << sumTestMinutesIterative(testMinutes, TEST_COUNT) << endl;

    return 0;
}

// Precondition: count is between 0 and the number of valid array elements.
int sumTestMinutes(const int testMinutes[], int count)
{
    if (count == 0)
    {
        return 0; // Base case: an empty group contributes no minutes.
    }

    int lastMinutes = testMinutes[count - 1]; // Keep this call's final duration.
    int earlierMinutes = sumTestMinutes(testMinutes, count - 1);
    return earlierMinutes + lastMinutes; // Resume here after the smaller call returns.
}

int sumTestMinutesIterative(const int testMinutes[], int count)
{
    int total = 0;

    for (int index = 0; index < count; index++)
    {
        total += testMinutes[index]; // Accumulate each duration in one call.
    }

    return total;
}
```

La salida es:

```text
Recursive total: 26
Iterative total: 26
```

### Qué ocurre durante las llamadas y los retornos

`count` es un parámetro pasado por valor: cada llamada tiene su propia copia. Cuando la llamada con `count == 5` llama con `count - 1`, la llamada nueva recibe `4`; el `count` de la llamada anterior sigue valiendo `5`. El array es compartido entre las llamadas y `const` impide modificar sus elementos mediante este parámetro.

| Llamada | Trabajo que conserva | Qué espera |
| --- | --- | --- |
| `sumTestMinutes(testMinutes, 5)` | `lastMinutes = 4` | La suma de las primeras cuatro posiciones. |
| `sumTestMinutes(testMinutes, 4)` | `lastMinutes = 6` | La suma de las primeras tres posiciones. |
| `sumTestMinutes(testMinutes, 3)` | `lastMinutes = 3` | La suma de las primeras dos posiciones. |
| `sumTestMinutes(testMinutes, 2)` | `lastMinutes = 8` | La suma de la primera posición. |
| `sumTestMinutes(testMinutes, 1)` | `lastMinutes = 5` | La suma de cero posiciones. |
| `sumTestMinutes(testMinutes, 0)` | Devuelve `0`. | No hace otra llamada. |

La llamada que recibe `0` devuelve primero. Luego se completan las sumas pendientes, en orden LIFO:

```text
count = 0: return 0
count = 1: return 0 + 5  -> 5
count = 2: return 5 + 8  -> 13
count = 3: return 13 + 3 -> 16
count = 4: return 16 + 6 -> 22
count = 5: return 22 + 4 -> 26
```

La línea `return earlierMinutes + lastMinutes` no se ejecuta hasta que la llamada más pequeña devuelve su resultado. Llegar al caso base termina la creación de llamadas nuevas; todavía falta regresar por las llamadas que estaban esperando.

### Terminación, errores y comparación con un loop

El caso base es `count == 0`; el avance es `count - 1`. La condición de entrada también importa: el ejemplo exige una cantidad entre cero y el número de posiciones válidas. Un valor negativo no se acerca a cero al restarle uno y puede provocar accesos inválidos. Tener un `if` no garantiza que la recursión sea correcta: cada llamada debe respetar las condiciones del problema.

Si la función volviera a llamarse con el mismo `count`, no habría reducción. Se acumularían llamadas pendientes hasta agotar el espacio disponible del call stack. Una llamada con `count + 1` tampoco avanzaría hacia este caso base.

Para esta suma, un loop suele ser la opción más directa. Ambas versiones revisan `n` duraciones y toman `O(n)` tiempo. La versión recursiva crea una cadena de `n + 1` llamadas contando el caso base; la iterativa realiza `n` vueltas de su loop. Esa diferencia no cambia el crecimiento lineal, pero sí cambia cómo se mantiene el trabajo pendiente. Después del ejemplo de factorial compararemos explícitamente el trabajo y la memoria que requieren ambas estrategias.

<details>
<summary>Pregunta de verificación: ¿hay cinco o seis llamadas a <code>sumTestMinutes</code> cuando <code>count</code> inicia en 5?</summary>

Hay seis llamadas en total: reciben `5`, `4`, `3`, `2`, `1` y `0`. Cinco son llamadas recursivas originadas dentro de la función; la llamada inicial proviene de `main`. La llamada con cero también existe, aunque su trabajo sea devolver inmediatamente.

</details>

### Segundo ejemplo: factorial y posibles órdenes de ejecución

El mismo laboratorio debe realizar cinco pruebas de calibración diferentes. Aquí, **distintas** significa que cada prueba es identificable y tiene una tarea propia:

- **A:** verificar el voltaje.
- **B:** verificar la temperatura.
- **C:** verificar la presión.
- **D:** verificar la velocidad.
- **E:** verificar la precisión.

Para este ejemplo, suponemos que ninguna prueba necesita el resultado de otra: **pueden ejecutarse en cualquier orden**. El laboratorio debe realizar las cinco, exactamente una vez cada una, y quiere saber cuántos órdenes de ejecución son posibles. La secuencia `A → B → C → D → E` es una posibilidad; `B → A → C → D → E` es otra. Contienen las mismas pruebas, pero cambia su orden.

Para elegir la primera prueba hay **5 opciones**. Después de elegirla, quedan **4** para la segunda posición, porque no podemos repetir la anterior. Luego quedan **3**, después **2** y finalmente **1**. Por cada elección inicial existen cuatro opciones para continuar, por cada una de esas existen tres, y así sucesivamente. Por eso multiplicamos: `5 × 4 × 3 × 2 × 1 = 120 órdenes diferentes`. No significa ejecutar 120 pruebas: significa que hay 120 maneras de ordenar las mismas cinco.

Este es un problema de **combinatoria**, el área de las matemáticas que estudia cómo contar selecciones y arreglos de elementos. Específicamente, contamos **permutaciones sin repetición de los cinco elementos**: usamos todos, ninguno se repite y el orden importa. Una **combinación**, en cambio, cuenta selecciones donde el orden no importa; seleccionar las cinco pruebas como un grupo sería una sola selección. Aquí necesitamos distinguir sus órdenes de ejecución, por lo que usamos permutaciones.

Ese producto se llama **factorial**. Para un entero no negativo `n`, se escribe `n!` y representa el producto de los enteros de `1` a `n`. Se define `0! = 1`: existe una forma de organizar una colección vacía, sin colocar ningún elemento. Aquí el signo `!` es notación matemática, no una operación factorial disponible en C++.

La definición se presta a recursión: `n! = n × (n - 1)!` para `n > 0`. El caso base es `0! = 1`, el caso recursivo multiplica por `n` y el avance reduce `n` en uno. La versión iterativa calcula el mismo producto mediante un loop.

El siguiente programa es independiente del de las duraciones. Guárdalo, si deseas probarlo, como `factorial_test_orders.cpp`. Incluye prototipos antes de `main` y definiciones después, igual que el ejemplo anterior.

```cpp
#include <iostream>
using namespace std;

long long factorialRecursive(int n);
long long factorialIterative(int n);

int main()
{
    const int DISTINCT_TESTS = 5;

    cout << "Recursive order count: " << factorialRecursive(DISTINCT_TESTS) << endl;
    cout << "Iterative order count: " << factorialIterative(DISTINCT_TESTS) << endl;
    cout << "Order count for no tests: " << factorialRecursive(0) << endl;

    return 0;
}

// Precondition: 0 <= n <= 20, so the result fits in long long.
long long factorialRecursive(int n)
{
    if (n == 0)
    {
        return 1; // Base case: the empty arrangement has one possible order.
    }

    long long smallerFactorial = factorialRecursive(n - 1);
    return n * smallerFactorial; // Multiply only after the smaller call returns.
}

// Precondition: 0 <= n <= 20, matching the recursive version.
long long factorialIterative(int n)
{
    long long product = 1; // Also gives the correct result when n is zero.

    for (int factor = 1; factor <= n; factor++)
    {
        product *= factor; // Build the same product without recursive calls.
    }

    return product;
}
```

Usamos `long long` para almacenar enteros mayores que los que permite un `int` típico. El factorial crece rápidamente; este ejemplo limita su entrada a `0..20` para que el resultado quepa en ese tipo. Cambiar el tipo no elimina el límite de representación. Los ejemplos usan entradas válidas; si se pidiera `n` al usuario, habría que comprobar ese rango antes de llamar las funciones.

La salida es:

```text
Recursive order count: 120
Iterative order count: 120
Order count for no tests: 1
```

Las llamadas y los retornos pueden leerse así:

```text
Calls:   factorialRecursive(5) -> (4) -> (3) -> (2) -> (1) -> (0)
Returns:
n = 0: return 1
n = 1: return 1 * 1  -> 1
n = 2: return 2 * 1  -> 2
n = 3: return 3 * 2  -> 6
n = 4: return 4 * 6  -> 24
n = 5: return 5 * 24 -> 120
```

En la llamada con `n == 5`, todavía no se conoce `smallerFactorial` cuando se solicita el factorial de cuatro. La multiplicación por cinco espera a que la llamada nueva termine. Cada llamada conserva su propio `n`, de modo que las multiplicaciones pendientes se completan al regresar.

<details>
<summary>Pregunta de verificación: ¿por qué el caso base del factorial devuelve 1 y el de la suma devuelve 0?</summary>

Cada caso base depende de la operación. Sumar cero valores aporta `0`, que no cambia una suma. Multiplicar cero factores aporta `1`, que no cambia un producto. Si el factorial devolviera `0` en su caso base, todas las multiplicaciones pendientes producirían cero.

</details>

### Comparar tiempo y memoria: recursión frente a iteración

Hasta ahora usamos Big-O para describir **tiempo**: cómo crece la cantidad de operaciones cuando aumenta la entrada. También podemos analizar la memoria necesaria. En ese análisis, **espacio** significa la cantidad de memoria que el algoritmo necesita mantener ocupada en su momento de mayor uso. Incluye, según qué se esté midiendo, arrays, variables y registros de llamadas activas. No se suman todas las asignaciones de memoria que ocurrieron a lo largo de la ejecución: se observa cuánta memoria se necesita simultáneamente.

El **espacio auxiliar** es la memoria adicional que utiliza el algoritmo para trabajar, aparte de los datos de entrada. Por ejemplo, en la suma, el array `testMinutes` ya contiene los datos recibidos y no se cuenta como auxiliar. Sí se cuentan el acumulador y el índice de la versión iterativa, o los parámetros, variables locales y demás información de las llamadas pendientes de la versión recursiva. Aunque no declaremos otro array ni usemos `new`, el call stack también consume memoria auxiliar.

Decir **espacio auxiliar `O(1)`** significa que esa memoria adicional permanece acotada al crecer `n`: un acumulador y un índice bastan tanto para cinco duraciones como para mil. No significa que se use cero memoria. Decir **espacio auxiliar `O(n)`** significa que la memoria adicional crece proporcionalmente a `n`: en la suma recursiva, más duraciones producen una cadena más larga de llamadas pendientes, cada una con una cantidad fija de información local.

Así, tiempo y espacio responden preguntas diferentes: “¿cuántas operaciones se realizan?” y “¿cuánta memoria debe mantenerse a la vez?”. Un algoritmo puede tomar `O(n)` tiempo y usar `O(1)` espacio auxiliar, como la suma iterativa. En la tabla siguiente, la columna de espacio mide siempre **espacio auxiliar**, incluyendo el call stack.

En la suma, `n` es la cantidad de duraciones. En el factorial, `n` es el entero cuyo factorial se calcula. Contamos cada suma o multiplicación de los tipos usados como una operación de costo constante; el análisis del factorial describe el patrón de trabajo, sujeto al rango numérico admitido por el programa.

| Problema y versión | Trabajo que se repite | Tiempo | Espacio auxiliar |
| --- | --- | --- | --- |
| Suma recursiva | Una suma por duración y una cadena de llamadas hasta cero. | `O(n)` | `O(n)` por las llamadas activas. |
| Suma iterativa | Una suma por vuelta del loop. | `O(n)` | `O(1)` para acumulador e índice. |
| Factorial recursivo | Una multiplicación por cada entero de `n` hasta `1`. | `O(n)` | `O(n)` por las llamadas activas. |
| Factorial iterativo | Una multiplicación por vuelta del loop. | `O(n)` | `O(1)` para producto e índice. |

En estos dos problemas, ambas estrategias realizan una cantidad de operaciones proporcional a `n`. Las llamadas recursivas añaden trabajo para conservar y recuperar el estado de cada llamada; Big-O no representa esa diferencia mediante un número exacto de segundos. El consumo de memoria sí tiene un crecimiento diferente en estas implementaciones: las llamadas pendientes crecen con `n`, mientras que las versiones con loops mantienen unas pocas variables.

No debe concluirse que toda recursión es `O(n)`. Para clasificar una función hay que observar cuántas llamadas genera, cuánto se reduce el problema y qué trabajo hace cada llamada. Merge sort, que estudiaremos a continuación, tendrá otro patrón.

> **Contraste con Fibonacci:** la sucesión de Fibonacci se define con `F(0) = 0`, `F(1) = 1` y `F(n) = F(n - 1) + F(n - 2)` para `n >= 2`. Una implementación recursiva directa genera dos llamadas y repite subproblemas: al calcular `F(5)`, por ejemplo, `F(3)` se calcula dentro de `F(4)` y de nuevo como la otra rama de `F(5)`. Esa versión tiene crecimiento de tiempo exponencial, que puede acotarse por `O(2^n)`, y profundidad de stack `O(n)`. Una versión iterativa puede calcular cada término una sola vez, conservando los dos anteriores, con tiempo `O(n)` y espacio `O(1)`. Su implementación queda como ampliación; este contraste muestra por qué recursión y tiempo lineal no son sinónimos.

## Parte 2 — Merge sort: dividir una colección para poder ordenarla (20 min)

### El problema del informe de calibración

Retomemos el laboratorio de calibración. Cada prueba tiene un ID y una duración. Las pruebas se registran en el orden en que terminan, pero el personal quiere un informe que muestre primero las más cortas y permita comparar duraciones. El programa debe **ordenar los registros por duración ascendente**, conservando el ID junto a la duración de su prueba.

Primero estudiaremos solo las duraciones para que los movimientos sean visibles. Usaremos ocho elementos para observar tres niveles de división y luego tres niveles de combinación. En el programa completo agregaremos los IDs y moveremos registros completos. Esta es la secuencia inicial:

```text
Index:       0   1   2   3   4   5   6   7
Minutes:    [8] [3] [6] [2] [7] [4] [6] [5]
```

**Sorting**, u ordenamiento, consiste en reorganizar elementos según un criterio. Aquí el criterio es duración de menor a mayor. El resultado debe ser `[2, 3, 4, 5, 6, 6, 7, 8]`; ordenar no debe perder mediciones ni cambiar sus valores. Las dos mediciones de seis minutos deben conservarse.

> **Recordatorio de selection sort:** busca el menor valor del segmento pendiente y lo coloca en la próxima posición de salida, repitiendo la búsqueda para cada posición. Esos recorridos repetidos producen tiempo `O(n²)`. Hoy estudiaremos una estrategia distinta: ordenar partes pequeñas y combinarlas.

### Qué es merge sort

**Merge sort** es un algoritmo de ordenamiento que divide una colección en dos partes, ordena cada parte y después combina ambas en una sola secuencia ordenada. Se llama **merge** a esa combinación. Sigue una estrategia llamada **divide and conquer**: dividir el problema, resolver los subproblemas y combinar sus soluciones.

La recursión permite repetir el mismo procedimiento sobre cada mitad. El caso base es un segmento con cero o un elemento: ya está ordenado porque no existe otro elemento que deba ir antes o después dentro de ese segmento.

### Cómo describir una parte del array sin crear arrays nuevos

Usaremos **segmentos con límites inclusivos**: `first` identifica el primer índice incluido y `last` el último. El segmento `0..7` incluye las ocho posiciones. El segmento `0..3` incluye cuatro y `0..1` incluye dos. Si `first > last`, el segmento está vacío.

La función que luego llamaremos `mergeSort` trabajará con esos límites. Para dividir, calcula:

```cpp
int middle = first + (last - first) / 2; // Locate the final index of the left half.
```

Esta línea irá dentro de `mergeSort`, después de comprobar su caso base. La división entre enteros descarta la parte fraccionaria. Para `first = 0` y `last = 7`, resulta `middle = 3`. La expresión calcula el punto medio sin sumar directamente dos índices potencialmente grandes.

Con ese punto medio se determinan los **nuevos límites que recibirá cada llamada**:

| Llamada siguiente | Su nuevo `first` | Su nuevo `last` | Razón |
| --- | --- | --- | --- |
| Ordenar la mitad izquierda | El `first` de la llamada actual. | El `middle` calculado. | Conserva el inicio y termina en el punto medio, incluido. |
| Ordenar la mitad derecha | `middle + 1`. | El `last` de la llamada actual. | Comienza después del punto medio y conserva el final. |

Por tanto, la llamada que recibe `0..7` solicita ordenar `0..3` y `4..7`. En la llamada izquierda, `first` vale `0` y `last` vale `3`; se vuelve a calcular `middle`, ahora `1`, para producir `0..1` y `2..3`. Cuando se ejecuta la llamada derecha sobre `4..7`, sus parámetros valen `first = 4` y `last = 7`; calcula `middle = 5` y produce `4..5` y `6..7`. Los límites siempre se obtienen de la llamada que se está ejecutando, no de los límites originales de todo el array.

Estas dos instrucciones aparecerán dentro de `mergeSort`, después de calcular `middle`. `tests` será el array de pruebas y `temporary` su array auxiliar; ambos se comparten entre las llamadas. Los dos argumentos finales son los límites que recibe la llamada nueva:

```cpp
mergeSort(tests, temporary, first, middle);    // Left call receives first..middle.
mergeSort(tests, temporary, middle + 1, last); // Right call receives middle+1..last.
```

No se asigna un valor nuevo al `first` o al `last` de la llamada actual. Cada llamada nueva recibe sus propias copias de esos enteros como parámetros. Por ejemplo, pasar `middle` como último argumento hace que **el parámetro `last` de la llamada nueva** reciba ese valor. La llamada original conserva sus límites para combinar después el segmento completo.

Usamos `middle + 1` porque los límites son inclusivos: el elemento en `middle` ya pertenece a la izquierda. Así ningún elemento se procesa en ambas mitades y ninguna posición queda fuera.

Dividir significa pasar límites diferentes a las llamadas siguientes; en esta etapa no se mueven valores ni se crean dos arrays nuevos.

```text
Split ranges by level:
0: [8 3 6 2 7 4 6 5] 0..7
1: [8 3 6 2] 0..3                    [7 4 6 5] 4..7
2: [8 3] 0..1   [6 2] 2..3           [7 4] 4..5   [6 5] 6..7
3: [8]0 [3]1    [6]2 [2]3            [7]4 [4]5    [6]6 [5]7

Merge sorted ranges on return:
2: [3 8] 0..1   [2 6] 2..3           [4 7] 4..5   [5 6] 6..7
1: [2 3 6 8] 0..3                    [4 5 6 7] 4..7
0: [2 3 4 5 6 6 7 8] 0..7
```

Estos son los rangos que se dividirán, no llamadas que corren simultáneamente. El programa completo terminará la rama izquierda antes de comenzar la derecha. Primero entenderemos cómo se combinan dos segmentos ordenados; después veremos qué llamadas producen esos segmentos.

### Cómo se evalúan los índices al llegar al caso base

El esquema anterior muestra los niveles de división y combinación. Para ver exactamente cuándo deja de dividirse una rama, necesitamos la condición del caso base de `mergeSort`: **`first >= last`**. Se comprueba al entrar en cada llamada, antes de calcular `middle` o solicitar nuevas llamadas.

- Si `first < last`, hay al menos dos elementos: la llamada continúa y divide su segmento.
- Si `first == last`, hay un solo elemento: ya está ordenado y la llamada ejecuta `return`.
- Si `first > last`, el segmento está vacío: también ejecuta `return`, sin acceder al array.

Este es el fragmento que aparecerá al inicio de la definición completa de `mergeSort` en la Parte 4:

```cpp
if (first >= last)
{
    return; // An empty segment or one element needs no further sorting.
}
```

Sigamos la primera rama del mismo ejemplo, sin cambiar sus valores ni sus rangos. La indentación muestra qué llamada está esperando a otra:

```text
mergeSort(..., 0, 7): 0 >= 7? false -> middle = 3
    mergeSort(..., 0, 3): 0 >= 3? false -> middle = 1
        mergeSort(..., 0, 1): 0 >= 1? false -> middle = 0
            mergeSort(..., 0, 0): 0 >= 0? true
                return; [8] is already sorted

        Resume the call for 0..1 after its left call:
            mergeSort(..., 1, 1): 1 >= 1? true
                return; [3] is already sorted

        Resume the call for 0..1 after its right call:
            merge(..., 0, 0, 1): combine [8] and [3] -> [3 8]
            the call for 0..1 finishes and returns

    Resume the call for 0..3 after its left call:
        next: mergeSort(..., 2, 3)
```

Los puntos suspensivos `...` abrevian los dos arrays que se pasan como primeros argumentos. En la línea de `merge`, los tres índices son `first`, `middle` y `last`; la función combina `0..0` con `1..1`.

Cuando `mergeSort(..., 0, 0)` ejecuta `return`, termina **solo esa llamada**; no termina todo el algoritmo. La llamada que recibió `0..1` sigue activa y continúa con su mitad derecha, `1..1`. Después de que esa segunda llamada también regresa, puede combinar ambos elementos. Por eso los resultados ordenados se construyen durante los retornos.

En el último nivel del esquema, todos los rangos tienen extremos iguales: `0..0`, `1..1`, hasta `7..7`. Cada uno satisface `first >= last`. No se calcula otro punto medio ni se llama a `merge` dentro de esas llamadas individuales; la combinación corresponde a la llamada anterior que las solicitó.

<details>
<summary>Pregunta de verificación: para cinco elementos en índices <code>0..4</code>, ¿cuáles son las mitades?</summary>

`middle = 0 + (4 - 0) / 2 = 2`. Las mitades son `0..2` y `3..4`: tres elementos y dos elementos. Las mitades no tienen que ser exactamente iguales, pero ambas deben ser más pequeñas que el segmento original y no deben solaparse ni omitir posiciones.

</details>

## Parte 3 — La operación merge: combinar dos segmentos ordenados (30 min)

### Qué recibe y qué debe producir

Supongan que ya terminamos de ordenar ambas mitades de las ocho duraciones. El array contiene ahora `[2, 3, 6, 8, 4, 5, 6, 7]`: la izquierda `[2, 3, 6, 8]` está ordenada y la derecha `[4, 5, 6, 7]` también. El array completo aún no lo está: por ejemplo, el `8` aparece antes del `4`.

La operación `merge` recibe los límites de esas dos mitades: `first = 0`, `middle = 3` y `last = 7`. Su tarea es producir `[2, 3, 4, 5, 6, 6, 7, 8]` dentro del segmento `0..7`.

No puede suponer que cualquier mitad está ordenada; esa es una **condición previa** de `merge`: quien la llama debe garantizar que `first..middle` y `middle + 1..last` ya estén ordenados según el mismo criterio. En la Parte 4 veremos cómo las llamadas recursivas garantizan esa condición.

### Por qué basta comparar el próximo valor de cada mitad

En una mitad ordenada, su menor elemento pendiente es el primero que aún no se ha usado. Por eso solo necesitamos comparar los dos primeros pendientes: el menor de ambos es el siguiente elemento del resultado. Después avanzamos únicamente en la mitad de donde salió el elemento elegido.

Necesitamos tres índices de trabajo:

- `leftIndex`: próximo elemento pendiente en la mitad izquierda; inicia en `first`.
- `rightIndex`: próximo elemento pendiente en la mitad derecha; inicia en `middle + 1`.
- `writeIndex`: posición donde guardar el próximo elemento del resultado; inicia en `first`.

El resultado se construye en un **array auxiliar**, llamado `temporary`. Es un segundo array del mismo tipo, con espacio suficiente para las posiciones del array original. Así conservamos los datos originales mientras todavía los estamos leyendo.

| Comparación o paso | Elemento elegido | Resultado acumulado en `temporary` |
| --- | --- | --- |
| `2` frente a `4` | `2`, de la izquierda | `[2]` |
| `3` frente a `4` | `3`, de la izquierda | `[2, 3]` |
| `6` frente a `4` | `4`, de la derecha | `[2, 3, 4]` |
| `6` frente a `5` | `5`, de la derecha | `[2, 3, 4, 5]` |
| `6` frente a `6` | `6`, de la izquierda en el empate | `[2, 3, 4, 5, 6]` |
| `8` frente a `6` | `6`, de la derecha | `[2, 3, 4, 5, 6, 6]` |
| `8` frente a `7` | `7`, de la derecha | `[2, 3, 4, 5, 6, 6, 7]` |
| La derecha se agotó | Copiar el `8` pendiente de la izquierda | `[2, 3, 4, 5, 6, 6, 7, 8]` |

En el último paso no hay dos elementos para comparar. Aun así, el elemento restante debe copiarse. Como su mitad ya estaba ordenada, los elementos que queden en ella pueden copiarse en su orden actual.

### Qué se perdería al escribir directamente en el array de entrada

En este ejemplo, copiar el primer `2` sobre la posición `0` no causa un conflicto porque ya lo consumimos. Pero el problema aparece al seleccionar el `4` de la derecha: su posición de salida es el índice `2`, donde todavía está el `6` pendiente de la izquierda. Escribir allí directamente destruiría ese `6` antes de compararlo. El array auxiliar evita sobrescribir datos pendientes. Después de terminar la combinación, copiamos el segmento auxiliar completo de regreso al original.

Para conectar el razonamiento con código, este fragmento muestra el loop principal de una versión que ordena solo enteros. Es un fragmento del cuerpo de `merge`, no un programa independiente; supone que los tres índices ya se inicializaron como se explicó y que `values` y `temporary` son arrays existentes. El programa completo de la Parte 4 define todas esas piezas para registros.

```cpp
while (leftIndex <= middle && rightIndex <= last)
{
    if (values[leftIndex] <= values[rightIndex])
    {
        temporary[writeIndex] = values[leftIndex]; // Preserve the selected left value.
        leftIndex++; // Only the selected half advances.
    }
    else
    {
        temporary[writeIndex] = values[rightIndex]; // Preserve the selected right value.
        rightIndex++;
    }

    writeIndex++; // One more output position has been filled.
}
```

`&&` exige que ambas mitades conserven al menos un elemento pendiente. Al salir del loop, puede quedar trabajo en una de ellas. La función completa necesitará dos loops adicionales para copiar los pendientes y un último loop para devolver el resultado al array original.

### Qué sucede cuando hay duraciones iguales

El operador `<=` elige el elemento izquierdo cuando los valores son iguales. Al ordenar registros, esa decisión permite conservar el orden original entre registros con la misma duración. Un algoritmo que conserva ese orden se llama **estable**.

Por ejemplo, si dos pruebas duran seis minutos y `CAL-103` estaba antes que `CAL-107` en la entrada, un ordenamiento estable mantiene ese orden relativo después de ordenar por duración. Estabilidad no significa que el array permanezca igual: significa que los empates conservan su orden original.

<details>
<summary>Práctica: combinen <code>[2, 7]</code> y <code>[3, 5, 9]</code>. ¿Qué mitad conserva elementos cuando termina el loop de comparaciones?</summary>

Se eligen `2`, `3`, `5` y `7`, en ese orden. En ese momento se agotó la izquierda y queda el `9` de la derecha, que se copia sin otra comparación entre mitades. El resultado es `[2, 3, 5, 7, 9]`.

</details>

## Parte 4 — Programa completo: ordenar registros con merge sort (35 min)

### Del dato aislado al registro de una prueba

El informe del laboratorio debe identificar cada prueba; una duración aislada no basta. Declararemos un `struct` llamado `CalibrationTest` para representar **una prueba**, con dos data members: `testId`, un ID dummy, y `minutes`, su duración entera en minutos. Un array de `CalibrationTest` representará la colección del informe.

La comparación usará `minutes`, pero cada asignación copiará **el registro completo**, incluido su ID. Si solo ordenáramos las duraciones y dejáramos los IDs donde estaban, el informe asociaría una duración con la prueba equivocada.

El programa tendrá tres funciones libres:

| Función | Responsabilidad |
| --- | --- |
| `mergeSort` | Determinar si un segmento necesita trabajo, ordenar recursivamente ambas mitades y llamar a `merge`. |
| `merge` | Combinar dos mitades ya ordenadas, usando el array auxiliar, y copiar el resultado al segmento original. |
| `displayTests` | Mostrar el ID y la duración de cada registro para verificar el antes y el después. |

`main` declara la colección y un array auxiliar con la misma capacidad. El auxiliar se crea una sola vez y se pasa a las funciones para reutilizarlo. Las funciones usan únicamente el segmento que les corresponde.

### Parámetros e índices antes de leer las definiciones

En `mergeSort(tests, temporary, first, last)`, `tests` es el array que se ordena; `temporary` es el espacio auxiliar; `first` y `last` delimitan las posiciones incluidas. En `merge`, `middle` indica además el último índice de la mitad izquierda. Estos parámetros son índices, no cantidades de elementos.

Los arrays se comparten entre las llamadas: modificar sus elementos afecta los arrays creados en `main`. En cambio, `first`, `middle` y `last` son enteros locales a cada llamada. No se pasan nuevas copias completas del array en cada llamada.

El código siguiente es un programa completo e independiente del ejemplo de la Parte 1. Para probarlo, guárdalo en un archivo llamado `calibration_merge_sort.cpp`. La declaración del `struct` y los prototipos van antes de `main`; las definiciones de las funciones van después. Todos los fragmentos de esta parte se integran en este único programa.

```cpp
#include <iostream>
#include <string>
using namespace std;

struct CalibrationTest
{
    string testId; // Keep the test identity attached to its duration.
    int minutes;   // Use this field as the ascending sort key.
};

void mergeSort(CalibrationTest tests[], CalibrationTest temporary[],
               int first, int last);
void merge(CalibrationTest tests[], CalibrationTest temporary[],
           int first, int middle, int last);
void displayTests(const CalibrationTest tests[], int count);

int main()
{
    const int TEST_COUNT = 8;
    CalibrationTest tests[TEST_COUNT] =
    {
        {"CAL-101", 8},
        {"CAL-102", 3},
        {"CAL-103", 6},
        {"CAL-104", 2},
        {"CAL-105", 7},
        {"CAL-106", 4},
        {"CAL-107", 6},
        {"CAL-108", 5}
    };
    CalibrationTest temporary[TEST_COUNT]; // Allocate one reusable auxiliary array.

    cout << "Before sorting:" << endl;
    displayTests(tests, TEST_COUNT);

    mergeSort(tests, temporary, 0, TEST_COUNT - 1); // Include every valid index.

    cout << "After sorting by duration:" << endl;
    displayTests(tests, TEST_COUNT);

    return 0;
}

// Sort the inclusive segment first..last; an empty segment needs no work.
void mergeSort(CalibrationTest tests[], CalibrationTest temporary[],
               int first, int last)
{
    if (first >= last)
    {
        return; // Base case: zero or one record is already sorted.
    }

    int middle = first + (last - first) / 2;

    mergeSort(tests, temporary, first, middle);    // Finish the entire left half first.
    mergeSort(tests, temporary, middle + 1, last); // Then finish the right half.
    merge(tests, temporary, first, middle, last); // Both halves are now sorted.
}

// Precondition: first..middle and middle+1..last are already sorted by minutes.
void merge(CalibrationTest tests[], CalibrationTest temporary[],
           int first, int middle, int last)
{
    int leftIndex = first;       // Next unused record from the left half.
    int rightIndex = middle + 1; // Next unused record from the right half.
    int writeIndex = first;      // Next output position in the auxiliary array.

    while (leftIndex <= middle && rightIndex <= last)
    {
        if (tests[leftIndex].minutes <= tests[rightIndex].minutes)
        {
            temporary[writeIndex] = tests[leftIndex]; // Copy the entire record.
            leftIndex++; // Prefer the left record on ties to preserve stability.
        }
        else
        {
            temporary[writeIndex] = tests[rightIndex];
            rightIndex++;
        }

        writeIndex++;
    }

    while (leftIndex <= middle)
    {
        temporary[writeIndex] = tests[leftIndex]; // Preserve every remaining left record.
        leftIndex++;
        writeIndex++;
    }

    while (rightIndex <= last)
    {
        temporary[writeIndex] = tests[rightIndex]; // Preserve every remaining right record.
        rightIndex++;
        writeIndex++;
    }

    for (int index = first; index <= last; index++)
    {
        tests[index] = temporary[index]; // Replace only this segment with its sorted result.
    }
}

void displayTests(const CalibrationTest tests[], int count)
{
    for (int index = 0; index < count; index++)
    {
        cout << "Test: " << tests[index].testId
             << " | Minutes: " << tests[index].minutes << endl;
    }
}
```

En `main`, las llaves de cada elemento, como `{"CAL-101", 8}`, inicializan los members del `struct` en el orden declarado: primero `testId`, después `minutes`. El operador `.` accede al member de un registro: `tests[leftIndex].minutes` consulta la duración de esa prueba. La asignación `temporary[writeIndex] = tests[leftIndex]` copia sus dos members, de acuerdo con el memberwise assignment visto en OOP.

Para compilar y ejecutar:

```bash
g++ calibration_merge_sort.cpp -o calibration_merge_sort
./calibration_merge_sort
```

El output completo es:

```text
Before sorting:
Test: CAL-101 | Minutes: 8
Test: CAL-102 | Minutes: 3
Test: CAL-103 | Minutes: 6
Test: CAL-104 | Minutes: 2
Test: CAL-105 | Minutes: 7
Test: CAL-106 | Minutes: 4
Test: CAL-107 | Minutes: 6
Test: CAL-108 | Minutes: 5
After sorting by duration:
Test: CAL-104 | Minutes: 2
Test: CAL-102 | Minutes: 3
Test: CAL-106 | Minutes: 4
Test: CAL-108 | Minutes: 5
Test: CAL-103 | Minutes: 6
Test: CAL-107 | Minutes: 6
Test: CAL-105 | Minutes: 7
Test: CAL-101 | Minutes: 8
```

### Seguir las dos llamadas recursivas sin saltarse pasos

El ejemplo completo conserva los ocho elementos de las Partes 2 y 3, en índices `0..7`. La primera llamada calcula `middle = 3`. La siguiente traza enumera las llamadas en el orden real de ejecución; se omiten los argumentos `tests` y `temporary` para concentrarnos en los límites.

| Paso | Operación | Resultado al regresar |
| ---: | --- | --- |
| 1 | `mergeSort(0, 7)` llama a `mergeSort(0, 3)`. | La llamada `0..7` queda esperando. |
| 2 | `mergeSort(0, 3)` llama a `mergeSort(0, 1)`. | La llamada `0..3` queda esperando. |
| 3 | `mergeSort(0, 1)` llama a `mergeSort(0, 0)` y luego a `mergeSort(1, 1)`. | Ambas alcanzan el caso base. |
| 4 | `merge(0, 0, 1)` combina las posiciones `0..0` y `1..1`. | `0..1` queda con duraciones `[3, 8]`. |
| 5 | `mergeSort(0, 3)` continúa con `mergeSort(2, 3)`, que visita los casos base `2..2` y `3..3`. | Ambas posiciones individuales están listas. |
| 6 | `merge(2, 2, 3)` combina esas posiciones. | `2..3` queda con `[2, 6]`. |
| 7 | `merge(0, 1, 3)` combina `0..1` con `2..3`. | `0..3` queda con `[2, 3, 6, 8]`. |
| 8 | `mergeSort(0, 7)` continúa con `mergeSort(4, 7)`, que llama a `mergeSort(4, 5)`. | Comienza el trabajo de la mitad derecha. |
| 9 | `mergeSort(4, 5)` visita los casos base `4..4` y `5..5`, y ejecuta `merge(4, 4, 5)`. | `4..5` queda con `[4, 7]`. |
| 10 | `mergeSort(4, 7)` continúa con `mergeSort(6, 7)`, que visita `6..6` y `7..7`, y ejecuta `merge(6, 6, 7)`. | `6..7` queda con `[5, 6]`. |
| 11 | `merge(4, 5, 7)` combina las dos partes derechas. | `4..7` queda con `[4, 5, 6, 7]`. |
| 12 | `merge(0, 3, 7)` combina las dos mitades finales. | El array contiene `[2, 3, 4, 5, 6, 6, 7, 8]`. |

La primera llamada no ejecuta inmediatamente sus tres líneas finales. Cada llamada a `mergeSort` debe terminar antes de pasar a la siguiente instrucción de quien la llamó. Por eso `merge` recibe mitades ordenadas: cuando llega su turno, ambas llamadas anteriores ya terminaron.

En el último merge, `CAL-103` y `CAL-107` empatan con duración `6`. El primero está en la mitad izquierda y se elige antes gracias a `<=`. Así el resultado conserva su orden original.

### Por qué termina y qué revisan los loops

Si `first < last`, el punto medio cumple `first <= middle < last`. La izquierda termina en `middle` y la derecha comienza en `middle + 1`: cada llamada recibe menos elementos. Usar `middle` como inicio derecho repetiría el mismo rango en un segmento de dos elementos y podría impedir terminar.

Los loops de pendientes también son necesarios. En cada posición del auxiliar debe quedar exactamente un registro del segmento original. La copia de regreso usa `first..last`, no todo el array: una combinación interna no debe sobrescribir posiciones de otros segmentos. `merge` solo lee las posiciones del auxiliar que acaba de escribir.

<details>
<summary>Pregunta de verificación: si una mitad contiene un solo registro, ¿por qué podemos usarla en <code>merge</code> sin ordenar más?</summary>

Un registro aislado ya satisface el criterio de orden dentro de su segmento. Al regresar del caso base, la condición previa de `merge` se cumple para esa mitad. No se requiere que el registro esté todavía en su posición final del array completo.

</details>

## Parte 5 — Cuánto trabajo hace y qué memoria necesita (25 min)

### Trabajo de una combinación

Aquí `n` representa el número de registros que se ordenan. Para combinar un segmento de `m` registros, `merge` copia esos `m` registros al auxiliar y luego los copia de regreso. Las comparaciones principales se detienen al agotarse una mitad; los loops de pendientes terminan de copiar lo que falta.

Esos loops son consecutivos. Los tres primeros, considerados juntos, escriben exactamente `m` registros; el último copia `m` de regreso. No hay un loop de `m` iteraciones dentro de otro de `m`. El trabajo total de combinar es proporcional a `m`, por lo que es `O(m)`.

### Por qué aparece el logaritmo

Para entender el crecimiento, pensemos en ocho registros. Al dividir repetidamente a la mitad:

```text
Range sizes along one path: 8 -> 4 -> 2 -> 1
```

Hay tres divisiones para llegar a un registro; `log2(8) = 3` expresa esa cantidad. Con dieciséis registros se necesita una división más. Este conteo de reducciones a la mitad es el mismo patrón logarítmico que vieron con binary search.

Sin embargo, merge sort debe resolver **ambas mitades**, mientras que binary search continúa buscando en una sola. Por eso sus costos completos son diferentes.

| Nivel de combinaciones para ocho registros | Segmentos que se combinan | Total de registros procesados en ese nivel |
| --- | --- | ---: |
| Desde los casos base | Cuatro segmentos de tamaño 2 | 8 |
| Siguiente nivel | Dos segmentos de tamaño 4 | 8 |
| Último nivel | Un segmento de tamaño 8 | 8 |

Cada nivel procesa en conjunto `n` registros y hay aproximadamente `log2(n)` niveles de combinaciones. El resultado es tiempo **`O(n log n)`**. Aquí sí se multiplican los factores: cada uno de esos niveles realiza trabajo proporcional a `n`.

Cuando `n` no es potencia de dos, algunas ramas tienen una división adicional respecto a otras. Eso no cambia el orden de crecimiento. Esta implementación siempre divide y combina los segmentos, incluso cuando la entrada ya estaba ordenada: su mejor, promedio y peor caso tienen crecimiento `O(n log n)` en este modelo.

| Cantidad de registros | `n log2(n)` como referencia de crecimiento | `n²` como referencia de crecimiento |
| ---: | ---: | ---: |
| 8 | 24 | 64 |
| 16 | 64 | 256 |
| 32 | 160 | 1,024 |

Estos números ilustran funciones de crecimiento; **no son cantidades exactas de comparaciones del programa**. Al duplicar `n`, el trabajo de merge sort crece algo más del doble, mientras que el crecimiento cuadrático se multiplica por cuatro.

### Memoria auxiliar y profundidad del call stack

El array `temporary` reserva `n` registros adicionales y se reutiliza durante toda la ejecución: cuesta `O(n)` espacio. El call stack mantiene las llamadas activas de una rama de divisiones, cuya profundidad es `O(log n)`.

Aunque cada llamada recursiva origina dos subproblemas, no están todos activos a la vez. Primero se termina la rama izquierda y luego se ejecuta la derecha. El total de llamadas durante toda la ejecución y la cantidad máxima de llamadas simultáneamente activas son medidas diferentes.

El espacio auxiliar total queda en `O(n)`, porque `n` domina a `log n`. El array original de registros no se cuenta como espacio auxiliar. Para este análisis tratamos la comparación de minutos y la copia de cada registro como operaciones de costo acotado; si los IDs fueran textos de longitud variable muy grande, habría que considerar también su costo de copia.

> **Recordatorio de bubble sort:** compara pares vecinos y los intercambia cuando están fuera de orden. Las pasadas repetidas pueden producir tiempo `O(n²)`; una versión que detecta una pasada sin intercambios puede terminar en `O(n)` para una entrada ya ordenada. Esa optimización no elimina su peor caso cuadrático.

| Propiedad | Merge sort presentado | Selection sort básico | Bubble sort con salida anticipada |
| --- | --- | --- | --- |
| Peor caso de tiempo | `O(n log n)` | `O(n²)` | `O(n²)` |
| Espacio auxiliar para ordenar el array | `O(n)` | `O(1)` | `O(1)` |
| Entrada ya ordenada | Sigue dividiendo y combinando. | Sigue buscando el mínimo pendiente. | Puede detenerse tras una pasada. |

Merge sort ofrece crecimiento favorable para colecciones grandes a cambio de memoria auxiliar en esta implementación con arrays. Elegir un algoritmo también exige considerar el tamaño real de los datos y las restricciones de memoria.

## Parte 6 — Merge sort con linked lists: ordenar cambiando enlaces (15 min)

### El mismo informe, almacenado en nodes

El laboratorio puede registrar sus pruebas conforme terminan sin conocer de antemano cuántas habrá. Supongamos ahora que el programa almacena las mismas ocho pruebas en una **singly linked list**, como las colecciones de la Semana 9. Cada node guarda el ID, la duración y un pointer `next`; la colección conserva un pointer `head` que señala su primer node.

El personal sigue necesitando un informe por duración ascendente. Merge sort conserva sus tres tareas —dividir, ordenar y combinar—, pero ahora trabaja con direcciones de nodes. Al ordenar, **los IDs y las duraciones permanecen dentro de sus nodes originales**. Lo que cambia es el orden de los enlaces `next` y, posiblemente, cuál node queda en `head`.

Para encapsular esa colección, declararemos una clase llamada `CalibrationTestList`. Su método público `sortByMinutes` solicitará el ordenamiento. Tres métodos privados realizarán el trabajo: `splitList` separará dos mitades, `mergeSortNodes` las ordenará recursivamente y `mergeSortedLists` las combinará con un loop.

### Cómo se aplica aquí la recursión

El problema que resuelve `mergeSortNodes` es **ordenar la lista que comienza en la dirección recibida**. Si esa lista tiene ocho nodes, la función no intenta elegir inmediatamente el menor de los ocho. La separa en dos listas de cuatro y se llama a sí misma para resolver exactamente el mismo problema —ordenar una lista—, pero ahora sobre menos nodes.

La llamada que recibe cuatro nodes hace lo mismo con dos listas de dos. La que recibe dos los separa en dos listas de un node. Al recibir un solo node, ya no necesita dividir: una lista de un elemento está ordenada. Ese es el **caso base**, junto con la lista vacía. El **caso recursivo** es separar una lista mayor, ordenar sus dos mitades mediante llamadas a la misma función y combinar los resultados. El **avance hacia el caso base** consiste en que cada mitad tiene menos nodes que la lista que la originó.

La función no necesita saber resolver directamente todos los tamaños. Solo necesita saber que una lista de un node está ordenada y saber combinar dos listas que ya lo están. Al regresar de los casos base, combina un node con otro para producir una lista ordenada de dos; esos resultados permiten ordenar cuatro y, finalmente, ocho. **La división reduce el problema; las combinaciones durante los retornos construyen la solución.**

El siguiente esquema muestra **qué lista recibe cada llamada al seguir la primera rama del ejemplo de calibración**. Cada número entre corchetes es la duración guardada en un node, y cada flecha `->` es su enlace `next`. Se omiten los IDs y el `nullptr` final para hacer visible la reducción. La indentación indica una llamada nueva: la de arriba queda esperando mientras se resuelve la de abajo.

```text
mergeSortNodes receives 8 nodes: [8] -> [3] -> [6] -> [2] -> [7] -> [4] -> [6] -> [5]
    After splitting, its left call receives 4 nodes: [8] -> [3] -> [6] -> [2]
        After splitting, its left call receives 2 nodes: [8] -> [3]
            After splitting, its left call receives 1 node: [8]
                Base case: return the address of this node.
```

La llamada que recibió `[8] -> [3]` no termina al recibir de vuelta el node `[8]`. Aún debe llamar con su otra mitad, `[3]`, que también satisface el caso base. Ahora sí tiene dos listas ordenadas de un node y puede combinar sus nodes como `[3] -> [8]`.

El próximo esquema muestra **tres combinaciones concretas que construyen el resultado al regresar**. Cada bloque presenta las dos listas que ya fueron ordenadas y la lista que devuelve su combinación. Entre un bloque y el siguiente deben completarse también las llamadas de las otras ramas: por ejemplo, la lista `[2] -> [6]` se obtiene al separar y combinar los nodes que inicialmente estaban como `[6] -> [2]`.

```text
First merge: each input has 1 node.
    Left sorted input:  [8]
    Right sorted input: [3]
    Returned list:      [3] -> [8]

Merge for the original left half: each input now has 2 nodes.
    Left sorted input:  [3] -> [8]
    Right sorted input: [2] -> [6]
    Returned list:      [2] -> [3] -> [6] -> [8]

Final merge: each input now has 4 nodes.
    Left sorted input:  [2] -> [3] -> [6] -> [8]
    Right sorted input: [4] -> [5] -> [6] -> [7]
    Returned list:      [2] -> [3] -> [4] -> [5] -> [6] -> [6] -> [7] -> [8]
```

Las listas de entrada de cada combinación están separadas; la función las une reconectando sus nodes existentes. No suma las duraciones ni crea más registros. El crecimiento de uno a dos, cuatro y ocho describe cuántos nodes contiene cada resultado ordenado.

La llamada de ocho nodes permanece pendiente en el call stack mientras se ordena su izquierda; cuando recibe ese resultado, solicita ordenar su derecha. Solo después de recibir ambos resultados ejecuta el merge final mostrado. No se ejecutan las dos ramas a la vez ni se pierde el trabajo de la llamada original: cada llamada conserva las direcciones y el punto donde debe continuar. En el Paso 3 seguiremos todas las ramas para completar los detalles omitidos en estos dos esquemas introductorios.

La recursión está en **`mergeSortNodes`, que se llama a sí misma dos veces sobre listas más pequeñas**. `splitList` encuentra y separa las mitades mediante un loop; `mergeSortedLists` combina mediante otro loop y no se llama a sí misma. Los pasos siguientes muestran cómo esos tres métodos cooperan para completar cada llamada recursiva.

### Paso 1 — Qué recibe y devuelve cada llamada

`mergeSortNodes` recibe la dirección del primer node de una lista y devuelve la dirección de su primer node **después de ordenarla**. El nombre de la función termina en `Nodes` porque ordena una cadena de nodes; no recibe índices. Su caso base es una lista vacía o de un solo node: esa lista ya está ordenada y puede devolver su dirección sin más trabajo.

Si recibe más de un node, la llamada debe completar estas tareas, en este orden:

1. Separar físicamente la cadena en dos listas con `splitList`.
2. Llamar a `mergeSortNodes` sobre la primera lista y **esperar a que devuelva una lista ordenada**.
3. Llamar a `mergeSortNodes` sobre la segunda lista y esperar su resultado ordenado.
4. Pasar esas dos listas a `mergeSortedLists` y devolver la dirección de la lista combinada.

La combinación no ordena por sí sola dos listas arbitrarias. Depende de que las llamadas del segundo y tercer paso ya hayan ordenado sus mitades. Para entender de dónde sale ese orden, seguiremos la división hasta los casos base y luego los retornos.

### Paso 2 — Separar: encontrar la mitad y cortar un enlace

Una lista no permite saltar a un índice central directamente. Usaremos dos pointers de trabajo que comienzan en el primer node: `slow` avanza un node por vuelta y `fast` avanza dos. Cuando `fast` ya no puede avanzar dos nodes, `slow` señala el último node de la primera mitad. En ocho nodes, termina en el cuarto; en cinco, termina en el tercero.

Antes de cortar, se guarda `slow->next` como dirección inicial de la mitad derecha. Después se asigna `slow->next = nullptr`, dejando dos listas independientes. Si se cortara antes de guardar esa dirección, se perdería el acceso a la segunda mitad.

Para distinguir incluso los dos nodes de seis minutos, asignaremos letras visuales a los registros: `A` es `CAL-101`, `B` es `CAL-102`, y así sucesivamente hasta `H`, que es `CAL-108`. La notación `[A:8]` representa el node de la prueba `CAL-101`, con duración `8`. Cada flecha `->` representa el enlace `next` del node anterior; las letras solo ayudan a seguir la identidad y no son nuevos data members.

```text
Before splitting:
head -> [A:8] -> [B:3] -> [C:6] -> [D:2] -> [E:7] -> [F:4] -> [G:6] -> [H:5] -> nullptr

Pointer positions while finding the middle:
                 slow     fast
Start:           A        A
After 1 step:    B        C
After 2 steps:   C        E
After 3 steps:   D        G
Stop: fast is G; G->next is H, but H->next is nullptr.

Save rightHead = E, then replace D->next with nullptr:
listHead  -> [A:8] -> [B:3] -> [C:6] -> [D:2] -> nullptr
rightHead -> [E:7] -> [F:4] -> [G:6] -> [H:5] -> nullptr
```

La separación todavía no ordenó ninguna mitad: la izquierda comienza con `8` antes de `3` y la derecha con `7` antes de `4`. Ahora ambas son problemas independientes y más pequeños. La llamada original conserva acceso a ambas mientras solicita ordenar primero la izquierda.

### Paso 3 — De dónde salen las mitades ordenadas: llamadas y retornos

Sigamos primero la cadena `A -> B -> C -> D`. `mergeSortNodes` la separa en `A -> B` y `C -> D`. A su vez, la llamada que recibe `A -> B` corta el enlace entre esos dos nodes y llama con `A` solo; esa llamada alcanza el caso base. Después llama con `B` solo, que también alcanza el caso base.

**Aquí aparece el primer merge con dos listas ordenadas:** cada una tiene un único node. `mergeSortedLists` compara `A:8` y `B:3`, selecciona `B` y conecta detrás a `A`. Devuelve la dirección de `B`, inicio de `B:3 -> A:8 -> nullptr`.

La misma operación sobre `C:6` y `D:2` devuelve `D:2 -> C:6 -> nullptr`. Solo después de recibir esos dos resultados, la llamada que tenía cuatro nodes combina `[B:3 -> A:8]` con `[D:2 -> C:6]` y devuelve `[D:2 -> B:3 -> C:6 -> A:8]`.

Este esquema representa las llamadas por indentación. Cada node se muestra como `letra:duración`, tanto al entrar como al regresar; por ejemplo, `A:8` identifica la prueba `CAL-101` y sus ocho minutos. Dentro de cada lista se omite el `nullptr` final para facilitar la lectura. Una línea más indentada es una llamada que debe terminar antes de que continúe la que está encima; los dos lados se ejecutan sucesivamente.

```text
mergeSortNodes(A:8 -> B:3 -> C:6 -> D:2 -> E:7 -> F:4 -> G:6 -> H:5)
    mergeSortNodes(A:8 -> B:3 -> C:6 -> D:2)
        mergeSortNodes(A:8 -> B:3)
            mergeSortNodes(A:8) -> return A:8  (one node)
            mergeSortNodes(B:3) -> return B:3  (one node)
            merge(A:8, B:3)    -> return B:3 -> A:8
        mergeSortNodes(C:6 -> D:2)
            mergeSortNodes(C:6) -> return C:6  (one node)
            mergeSortNodes(D:2) -> return D:2  (one node)
            merge(C:6, D:2)    -> return D:2 -> C:6
        merge(B:3 -> A:8, D:2 -> C:6)
            -> return D:2 -> B:3 -> C:6 -> A:8
    mergeSortNodes(E:7 -> F:4 -> G:6 -> H:5)
        mergeSortNodes(E:7 -> F:4)
            mergeSortNodes(E:7) -> return E:7  (one node)
            mergeSortNodes(F:4) -> return F:4  (one node)
            merge(E:7, F:4)    -> return F:4 -> E:7
        mergeSortNodes(G:6 -> H:5)
            mergeSortNodes(G:6) -> return G:6  (one node)
            mergeSortNodes(H:5) -> return H:5  (one node)
            merge(G:6, H:5)    -> return H:5 -> G:6
        merge(F:4 -> E:7, H:5 -> G:6)
            -> return F:4 -> H:5 -> G:6 -> E:7
    merge(D:2 -> B:3 -> C:6 -> A:8, F:4 -> H:5 -> G:6 -> E:7)
        -> return D:2 -> B:3 -> F:4 -> H:5 -> C:6 -> G:6 -> E:7 -> A:8
```

En el esquema, `merge(...)` abrevia el método `mergeSortedLists(...)`. Las letras representan direcciones iniciales y cadenas, no nuevos argumentos que deba recibir el código. Cada `return` entrega el primer node de la cadena reconstruida a la llamada que esperaba ese resultado.

La mitad derecha no empieza a ordenarse hasta que la mitad izquierda completa sus divisiones y combinaciones. Al terminar ambas, la llamada original por fin tiene estos dos resultados:

```text
sortedLeft  -> [D:2] -> [B:3] -> [C:6] -> [A:8] -> nullptr
sortedRight -> [F:4] -> [H:5] -> [G:6] -> [E:7] -> nullptr
```

### Paso 4 — Combinar al regresar: qué enlace se cambia realmente

Dentro de `mergeSortedLists`, `left` y `right` señalan el primer pendiente de cada mitad. `mergedHead` conservará el primer node del resultado y `tail` señalará el último que ya se eligió. Después de seleccionar el menor de los dos primeros, el loop conecta un node por vez mediante `tail->next`.

| Paso del merge final | Selección y enlace | `left` después | `right` después | `tail` después |
| --- | --- | --- | --- | --- |
| Elegir el inicio entre `D:2` y `F:4` | `mergedHead` recibe `D`. | `B:3` | `F:4` | `D:2` |
| Comparar `B:3` y `F:4` | Conservar `D->next = B`. | `C:6` | `F:4` | `B:3` |
| Comparar `C:6` y `F:4` | Cambiar `B->next` para que señale `F`. | `C:6` | `H:5` | `F:4` |
| Comparar `C:6` y `H:5` | Conectar `F->next = H`. | `C:6` | `G:6` | `H:5` |
| Comparar `C:6` y `G:6` | Conectar `H->next = C`; elegir la izquierda en el empate. | `A:8` | `G:6` | `C:6` |
| Comparar `A:8` y `G:6` | Cambiar `C->next` para que señale `G`. | `A:8` | `E:7` | `G:6` |
| Comparar `A:8` y `E:7` | Conectar `G->next = E`. | `A:8` | `nullptr` | `E:7` |
| La derecha se agotó | Conectar `E->next = A`, toda la cadena izquierda pendiente. | `A:8` | `nullptr` | `E:7` |

Miremos de cerca la tercera selección. Hasta entonces, `B` conservaba su antiguo enlace a `C`. Como `F:4` debe aparecer antes de `C:6`, el merge cambia ese enlace:

```text
Before selecting F:
tail = B                       left = C
[B:3 | next] -----------------> [C:6 | next] -> [A:8 | nullptr]

right = F
[F:4 | next] -> [H:5 | next] -> [G:6 | next] -> [E:7 | nullptr]

Change B->next to F; advance right to H and tail to F:
mergedHead = D
[D:2 | next] -> [B:3 | next] -> [F:4 | next] -> [H:5 | next] -> ...
                                tail          right

left still preserves the pending chain:
[C:6 | next] -> [A:8 | nullptr]
```

Los nodes `C` y `A` no desaparecen al sustituir el enlace de `B`: `left` conserva el acceso a ellos. El enlace que sale del `tail` todavía puede apuntar a un node no elegido; el resultado completo solo queda establecido cuando termina el loop y se conecta la cadena restante.

Al terminar, se devuelve `mergedHead`, que señala a `D`. El método público guarda esa dirección en `head`, y la colección queda así:

```text
head -> [D:2] -> [B:3] -> [F:4] -> [H:5] -> [C:6] -> [G:6] -> [E:7] -> [A:8] -> nullptr
```

Cada prueba sigue en el mismo node que al inicio. `C` continúa guardando `CAL-103` y seis minutos; `G` conserva `CAL-107` y seis minutos. Su orden relativo también se conservó porque los empates se resolvieron eligiendo la izquierda.

### Programa completo y ubicación de los métodos

Durante esta sección, concentren la lectura en los tres métodos privados de división, recursión y combinación. El constructor, agregado al final, destructor y traversal retoman operaciones ya vistas en la Semana 9; se incluyen para que puedan ejecutar el ejemplo completo después de clase.

El siguiente programa es independiente del ejemplo de arrays. Puede guardarse como `calibration_list_merge_sort.cpp`. Todos los métodos están definidos **dentro de la clase** `CalibrationTestList`; por eso no llevan `CalibrationTestList::`. `main` crea un objeto de esa clase y llama únicamente sus métodos públicos.

Los helpers devuelven `Node *`: una dirección, no un node nuevo. `mergeSortNodes` recibe la dirección inicial de una lista y devuelve la de su primer node después de ordenarla. Por eso `sortByMinutes` debe guardar ese resultado en `head`.

```cpp
#include <iostream>
#include <string>
using namespace std;

class CalibrationTestList
{
private:
    struct Node
    {
        string testId; // Keep the ID attached to the same test node.
        int minutes;   // Compare this field when selecting the next node.
        Node *next;    // Change this link to reorder the list.
    };

    Node *head; // Remember the first node of the entire collection.

    // Precondition: listHead begins a list containing at least two nodes.
    Node *splitList(Node *listHead)
    {
        Node *slow = listHead;
        Node *fast = listHead;

        while (fast->next != nullptr && fast->next->next != nullptr)
        {
            slow = slow->next;       // Move one node toward the middle.
            fast = fast->next->next; // Move twice as fast toward the end.
        }

        Node *rightHead = slow->next; // Preserve the second half before cutting.
        slow->next = nullptr;         // End the first half as an independent list.
        return rightHead;
    }

    // Precondition: left and right begin disjoint lists already sorted by minutes.
    Node *mergeSortedLists(Node *left, Node *right)
    {
        if (left == nullptr)
        {
            return right;
        }
        if (right == nullptr)
        {
            return left;
        }

        Node *mergedHead;
        if (left->minutes <= right->minutes)
        {
            mergedHead = left; // Prefer the left node on ties.
            left = left->next; // Preserve access to its remaining nodes.
        }
        else
        {
            mergedHead = right;
            right = right->next;
        }

        Node *tail = mergedHead; // The selected first node is also the current last.
        while (left != nullptr && right != nullptr)
        {
            if (left->minutes <= right->minutes)
            {
                tail->next = left; // Link an existing node into the result.
                left = left->next;
            }
            else
            {
                tail->next = right;
                right = right->next;
            }
            tail = tail->next; // Track the last node selected for the result.
        }

        if (left != nullptr)
        {
            tail->next = left; // Attach the entire remaining left chain.
        }
        else
        {
            tail->next = right; // Attach the remaining right chain, possibly empty.
        }
        return mergedHead;
    }

    Node *mergeSortNodes(Node *listHead)
    {
        if (listHead == nullptr || listHead->next == nullptr)
        {
            return listHead; // Base case: an empty list or one node is sorted.
        }

        Node *rightHead = splitList(listHead);
        Node *sortedLeft = mergeSortNodes(listHead);
        Node *sortedRight = mergeSortNodes(rightHead);
        return mergeSortedLists(sortedLeft, sortedRight);
    }

public:
    CalibrationTestList()
    {
        head = nullptr;
    }

    ~CalibrationTestList()
    {
        while (head != nullptr)
        {
            Node *nodeToDelete = head;
            head = head->next; // Preserve the next address before releasing this node.
            delete nodeToDelete;
        }
    }

    void addTest(const string& newTestId, int newMinutes)
    {
        Node *newNode = new Node;
        newNode->testId = newTestId;
        newNode->minutes = newMinutes;
        newNode->next = nullptr;

        if (head == nullptr)
        {
            head = newNode;
            return;
        }

        Node *currentNode = head;
        while (currentNode->next != nullptr)
        {
            currentNode = currentNode->next;
        }
        currentNode->next = newNode; // Append in the original registration order.
    }

    void sortByMinutes()
    {
        head = mergeSortNodes(head); // The first node may change after sorting.
    }

    void display() const
    {
        Node *currentNode = head;
        while (currentNode != nullptr)
        {
            cout << "Test: " << currentNode->testId
                 << " | Minutes: " << currentNode->minutes << endl;
            currentNode = currentNode->next;
        }
    }
};

int main()
{
    CalibrationTestList tests;
    tests.addTest("CAL-101", 8);
    tests.addTest("CAL-102", 3);
    tests.addTest("CAL-103", 6);
    tests.addTest("CAL-104", 2);
    tests.addTest("CAL-105", 7);
    tests.addTest("CAL-106", 4);
    tests.addTest("CAL-107", 6);
    tests.addTest("CAL-108", 5);

    cout << "Before sorting:" << endl;
    tests.display();
    tests.sortByMinutes();
    cout << "After sorting by duration:" << endl;
    tests.display();
    return 0;
}
```

Este programa produce el mismo output que el de la Parte 4, incluidos los IDs y el orden de los dos registros de seis minutos. Pueden compilarlo con `g++ calibration_list_merge_sort.cpp -o calibration_list_merge_sort` y ejecutarlo con `./calibration_list_merge_sort`.

El `new` de `addTest` crea los nodes al registrar las pruebas; el destructor los libera al terminar la vida de la colección. Las funciones que ordenan no crean ni eliminan nodes. El objeto es dueño de esos nodes y no debe copiarse superficialmente, por la misma razón explicada en la Semana 9.

### Leer los cambios de pointers y su costo

`left`, `right` y `listHead` son parámetros pointer pasados por valor: moverlos cambia la dirección que conserva la llamada local. En cambio, `tail->next = left` cambia un member de un node existente. Esa distinción permite recorrer pendientes con variables locales y modificar realmente la estructura con asignaciones a `next`.

El operador `&&` de `splitList` evalúa primero `fast->next != nullptr`; solo evalúa el siguiente acceso si el primero es verdadero. De forma similar, el `||` del caso base evita consultar `listHead->next` si `listHead` es `nullptr`. Así se protege cada acceso a members.

La división recorre cada sublista para encontrar su mitad y la combinación recorre sus nodes para elegir el orden. El trabajo total por nivel sigue siendo `O(n)` y hay `O(log n)` niveles, para tiempo `O(n log n)`. La combinación iterativa usa una cantidad fija de pointers; la recursión de división conserva `O(log n)` llamadas activas. Por tanto, **el espacio auxiliar de esta versión es `O(log n)`**. Este análisis corresponde al ordenamiento; el tiempo de construir inicialmente la lista con `addTest` se considera aparte.

### Comparar merge sort con arrays y con linked lists

Ahora podemos comparar las dos implementaciones de esta lecture. En ambas, `n` es la cantidad de pruebas almacenadas. Comparamos el tiempo necesario para ordenarlas y, por separado, la memoria adicional que utiliza ese ordenamiento; la colección original no se cuenta como espacio auxiliar.

| Medida o decisión | Versión con array de la Parte 4 | Versión con linked list de la Parte 6 |
| --- | --- | --- |
| Tiempo de ordenamiento | `O(n log n)` | `O(n log n)` |
| Espacio auxiliar total | `O(n)` | `O(log n)` |
| Cómo reorganiza las pruebas | Copia registros completos al auxiliar y luego de regreso. | Cambia los enlaces `next` de los nodes existentes. |
| Memoria adicional que determina el crecimiento | El array `temporary`, de `n` registros. | Las llamadas recursivas activas; el merge iterativo usa una cantidad fija de pointers. |
| Estabilidad | Conserva el orden de los empates al elegir la izquierda. | Conserva el orden de los empates al elegir la izquierda. |

**Ambas versiones tienen el mismo orden de crecimiento de tiempo.** Aunque la lista deba recorrerse para encontrar cada mitad, ese recorrido y el merge siguen sumando trabajo lineal por nivel. Repetir ese trabajo en los niveles de división produce `O(n log n)` en ambas representaciones.

La diferencia está en el **espacio auxiliar**: `O(log n)` crece más lentamente que `O(n)`. En la versión enlazada presentada, la memoria adicional aumenta con la profundidad de las llamadas, sin reservar otra colección de `n` registros. Esa conclusión depende de combinar mediante un loop; si el merge también fuera recursivo node por node, podría mantener una cadena de llamadas de profundidad `O(n)` y cambiar el costo de memoria.

No debemos comparar el `O(log n)` de memoria de la lista con el `O(n log n)` de tiempo del array para decidir cuál es más rápido: son medidas diferentes. Usar menos memoria adicional tampoco garantiza menor tiempo real de ejecución. Los arrays almacenan registros en posiciones consecutivas, lo que suele favorecer su acceso en memoria; los nodes pueden estar dispersos y requieren seguir pointers. Big-O describe el crecimiento, pero no determina por sí solo qué programa tardará menos segundos.

<details>
<summary>Verificación: al terminar, ¿a qué node debe apuntar <code>head</code> y por qué hace falta asignar el retorno de <code>mergeSortNodes</code>?</summary>

Debe apuntar al node de `CAL-104`, que dura dos minutos. Antes señalaba `CAL-101`, de ocho minutos. La función devuelve la nueva entrada a la cadena ordenada; si no se actualizara `head`, la clase conservaría la dirección del antiguo primer node y no podría recorrer la lista ordenada completa desde su inicio.

</details>

## Parte 7 — Verificar, conectar con searching y cerrar (10 min)

Estos 10 minutos se reservan para comprobar el resultado, resolver dudas y retomar una traza si hace falta. La práctica extensa de implementación y pruebas continuará en Lab 11.

### Pruebas que no deben faltar

Ejecuten el programa completo y luego cambien los datos de prueba, conservando consistencia entre la cantidad y la capacidad de los arrays. Revisen una sola prueba, una cantidad impar de pruebas, registros ya ordenados, duraciones en orden inverso y duraciones repetidas.

Para probar un segmento vacío en C++ estándar, pueden usar un array de capacidad positiva y cantidad lógica `0`. La llamada sería `mergeSort(tests, temporary, 0, -1)`; el caso base debe regresar sin acceder al array. No declaren un array de tamaño cero.

Además de revisar que las duraciones queden ordenadas, confirmen que todos los IDs siguen presentes, que cada uno conserva su duración original y que los empates conservan su orden. Una salida ascendente por sí sola no demuestra que se preservaron los registros.

### Ordenar y buscar son tareas diferentes

El personal del laboratorio también puede necesitar localizar una prueba concreta. **Linear search** revisa registros uno a uno hasta encontrar una coincidencia o agotar la colección; funciona aunque los datos estén desordenados. **Binary search** compara con el elemento central y descarta una mitad, lo que requiere que la colección esté ordenada por el campo que se está buscando.

Nuestro informe quedó ordenado por `minutes`. Eso permite aplicar binary search por duración, pero **no** por `testId`: los IDs no quedaron en orden ascendente. Además, como hay duraciones repetidas, una búsqueda binaria básica por duración puede devolver cualquiera de las coincidencias, no necesariamente la primera.

En la Semana 12 retomaremos la implementación y la elección de estrategias de búsqueda. Para muchas consultas por un mismo campo puede convenir ordenar una vez y después buscar; si solo hay una consulta, el costo de ordenar primero puede superar el de una búsqueda lineal directa. En una linked list, obtener el elemento central tampoco tiene el acceso directo `O(1)` de un array: la representación sigue siendo parte de la decisión.

### Conexión con el trabajo del proyecto

En el próximo lab practicarán recursión y merge sort, y reservarán tiempo para aplicar lo aprendido al proyecto. Antes de ordenar sus registros, deberán decidir qué campo define el orden, qué hacer con los empates y cómo verificar que se preservó la información completa. En arrays moverán registros completos; en linked lists podrán reconectar los nodes que los contienen. El criterio concreto de su aplicación debe responder a los requisitos del proyecto.

## Resumen de la sesión

- Una función recursiva resuelve un problema mediante llamadas a versiones más pequeñas. Necesita un caso base y un avance que garantice llegar a él.
- Cada llamada tiene su propio estado local y conserva trabajo pendiente. Los retornos completan ese trabajo en orden LIFO.
- Merge sort divide un segmento, ordena recursivamente sus mitades y las combina cuando ambas están listas.
- La versión con arrays usa un array auxiliar y copia registros completos; la versión con linked lists separa y reconecta nodes existentes mediante sus enlaces `next`.
- Ambas versiones ordenan por duración y conservan la identidad de cada prueba; elegir la izquierda en empates conserva estabilidad.
- Ambas toman tiempo `O(n log n)`. El espacio auxiliar es `O(n)` en la versión de arrays y `O(log n)` en la versión de linked lists con combinación iterativa.
- Binary search requiere orden por el mismo campo de búsqueda; ordenar por un campo no habilita búsquedas binarias por todos los demás.

## Próxima sesión

El Lab 11 reforzará recursión y merge sort, con una sección final de aplicación al proyecto. En la Lecture de Semana 12 estudiarán quicksort, compararán estrategias de sorting y searching y comenzarán la introducción a Binary Trees. La Semana 13 completará árboles, incluyendo BST, inserción, búsqueda y recorridos.
