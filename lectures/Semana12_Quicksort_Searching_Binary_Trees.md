# Semana 12 — Quicksort, Searching e Introducción a Binary Trees
## COEN 2220 — Programming 2

**Instructor:** Wilson Lozano

**Duración:** 170 min (sesión de 3 horas crédito nominales)  
**Precede a:** Lab 12 — Sorting y Searching + aplicación al proyecto  
**Lectura complementaria:** Gaddis, *Starting Out with C++: From Control Structures through Objects*, 8va ed., Cap. 19.8 (QuickSort), Cap. 8.1 y 8.3 (searching y sorting), Cap. 19.6 (binary search recursiva) y Cap. 20.1 (definición y aplicaciones de binary trees). Quicksort sigue el esquema del libro: seleccionar inicialmente el elemento central, moverlo al inicio del segmento y particionar con comparación estricta respecto al pivot.

---

## Objetivos

Al finalizar esta sesión, el estudiante podrá:

1. Definir pivot y partición, y trazar qué región del array representa cada índice de trabajo.
2. Implementar quicksort sobre registros completos y justificar sus límites recursivos y su caso base.
3. Comparar quicksort, merge sort y los métodos básicos considerando tiempo, espacio auxiliar y estabilidad.
4. Implementar y trazar linear search y binary search, distinguiendo sus requisitos, resultados y costos.
5. Explicar qué es un binary tree y reconocer root, children, parent, leaf y subtree.
6. Distinguir un binary tree general de un BST y relacionar su regla de orden con la selección de una rama de búsqueda.

## Parte 0 — Puente: otra forma de dividir antes de ordenar (10 min)

En la [Lecture de Semana 11](Semana11_Recursion_Merge_Sort.md) el laboratorio de calibración necesitaba ordenar registros de pruebas por duración, manteniendo cada ID asociado a sus minutos. Merge sort dividía los rangos por la mitad, ordenaba ambas partes y las combinaba al regresar. La recursión conservaba los rangos pendientes mientras el programa resolvía problemas más pequeños.

Ahora necesitamos comparar esa estrategia con otra forma de ordenar. En arrays, la versión de merge sort estudiada usa una colección auxiliar para combinar registros. Quicksort reorganiza el segmento alrededor de un elemento de referencia **antes** de ordenar sus dos lados. Estudiaremos qué garantiza esa reorganización y después cómo la recursión completa el ordenamiento. Finalmente usaremos los datos ordenados para buscar y conectaremos la selección de una mitad con una nueva estructura enlazada: el árbol binario.

### Qué debemos recordar de los rangos

Un rango inclusivo `first..last` contiene ambas posiciones. `0..7` incluye ocho elementos; `2..2` contiene uno; `3..2` está vacío. Cada llamada recursiva recibe sus propias copias de los límites y debe acercarse a un caso base.

Para recuperar el hilo de merge sort, la siguiente secuencia resume sus responsabilidades; las flechas indican el orden del trabajo, no enlaces entre nodes:

```text
Merge sort: split by index -> sort both halves -> merge sorted results
```

En quicksort también ordenaremos dos subproblemas, pero primero moveremos registros para establecer qué lado les corresponde. Dividir por posición y dividir según el valor de un elemento no son la misma decisión.

<details>
<summary>Verificación: ¿dividir un array por la mitad garantiza que todos los valores de la izquierda sean menores que los de la derecha?</summary>

No. En merge sort la división determina rangos, no el orden de sus valores. Por eso, después de ordenar las mitades, se necesita una operación de combinación que compare sus elementos.

</details>

## Parte 1 — Quicksort: el problema, el pivot y la partición (20 min)

Ya sabemos reducir un rango mediante recursión. Falta entender qué trabajo puede hacerse **antes** de las llamadas para que sus resultados no necesiten un merge posterior. Introduciremos quicksort mediante un catálogo concreto, definiremos qué garantiza su partición y prepararemos los índices que luego seguiremos en una traza.

### Contexto: un catálogo de clips audiovisuales

El Centro de Medios de una universidad conserva clips para preparar presentaciones. Cada clip tiene un ID y una duración en segundos. El personal necesita un catálogo de menor a mayor duración para revisar qué materiales pueden ajustarse a un bloque de tiempo. También quiere consultar después si existe un clip con una duración determinada.

La colección del ejemplo tendrá ocho clips dummy. Fijamos estos datos para que todos puedan seguir las mismas comparaciones; después podrán cambiar las duraciones y repetir las pruebas. Las letras A–H abrevian sus IDs en las trazas. La tabla presenta la colección original y el campo que decide el orden:

| Índice original | Letra de la traza | ID | Duración en segundos |
| ---: | --- | --- | ---: |
| 0 | A | VC-101 | 42 |
| 1 | B | VC-102 | 18 |
| 2 | C | VC-103 | 35 |
| 3 | D | VC-104 | 30 |
| 4 | E | VC-105 | 27 |
| 5 | F | VC-106 | 18 |
| 6 | G | VC-107 | 50 |
| 7 | H | VC-108 | 12 |

Ordenaremos por duración, moviendo cada registro completo. B y F tienen la misma duración, pero siguen siendo clips diferentes. Su identidad permitirá comprobar más adelante si el algoritmo conserva el orden original entre empates.

### Qué es quicksort

**Quicksort** es un algoritmo de ordenamiento que selecciona un elemento de referencia, reorganiza los demás respecto a él y ordena recursivamente los segmentos resultantes. El elemento de referencia se llama **pivot**. La operación que reorganiza el segmento se llama **partición**.

Usaremos esta regla: al terminar la partición, los registros con duración **menor** que la del pivot estarán a su izquierda y los de duración **mayor o igual** estarán a su derecha. El pivot quedará entre ambos grupos, en una posición válida para el orden final por duración. Dentro de cada grupo todavía puede haber desorden.

Hay distintas formas de seleccionar el pivot y de implementar la partición. Seguiremos el esquema del libro: calcular el índice central del segmento, seleccionar el registro que está allí y moverlo al inicio mediante un intercambio. Durante el recorrido, el pivot permanecerá en `first`; al terminar se colocará entre los dos grupos.

Seleccionar el elemento central se refiere a su **posición**, no a conocer de antemano el valor mediano de las duraciones. En una entrada ya ordenada con valores distintos, esa selección evita escoger sistemáticamente un extremo por valor y produce divisiones equilibradas. En una entrada arbitraria, sin embargo, el registro central podría ser el menor o el mayor. Esta selección no garantiza el equilibrio ni elimina el peor caso de quicksort.

### Qué produce una partición y qué deja pendiente

Para los ocho clips, el índice central se calcula como `0 + (7 - 0) / 2 = 3`, usando división entera. El pivot inicial es D, de 30 segundos, que está en ese índice. La siguiente forma representa la **condición al terminar** la primera partición, no el catálogo completamente ordenado:

```text
Durations smaller than 30 | pivot D:30 | durations greater than or equal to 30
```

La partición devuelve el índice donde quedó el pivot. Después quicksort ordena ambos lados sin incluir ese índice otra vez. La diferencia con merge sort es el momento del trabajo: aquí la partición organiza los lados antes de la recursión; no habrá un merge después de ella.

<details>
<summary>Verificación: ¿un clip de 18 segundos tiene que terminar necesariamente en el primer índice después de particionar con pivot 30?</summary>

No. Debe quedar a la izquierda del pivot, pero podría haber otros clips menores, como el de 12 segundos. La partición establece grupos respecto al pivot; las llamadas recursivas ordenarán el interior de esos grupos.

</details>

## Parte 2 — Partición paso a paso con ocho registros (30 min)

La regla respecto al pivot ya está definida. Ahora debemos mantenerla mientras recorremos el array, sin perder registros ni separar sus IDs de sus duraciones. Seguiremos los índices y cada intercambio de la primera partición; ese seguimiento permitirá leer después la función del programa completo.

### Cómo interpretar la traza y las regiones

La notación `A:42` significa “clip A, ID VC-101, duración 42 segundos”. Los espacios separan posiciones consecutivas del array; no representan una linked list. Esta es la entrada antes de particionar `0..7`:

```text
Index:  0     1     2     3     4     5     6     7
Clip:   A:42  B:18  C:35  D:30  E:27  F:18  G:50  H:12
                          pivot selected at middle = 3
```

La función que llamaremos `partitionClips` usará estos nombres:

- `first` y `last`: límites inclusivos del segmento actual.
- `middle`: índice central, calculado como `first + (last - first) / 2`.
- `pivotSeconds`: duración del registro elegido en `middle`, que luego se mueve a `first`; será 30 durante esta primera partición.
- `boundary`: última posición del grupo menor que el pivot; comienza en `first` para indicar que todavía no hay registros en ese grupo.
- `scan`: índice del registro que estamos examinando; recorre desde `first + 1` hasta `last`, porque el pivot se queda en `first`.

Antes del recorrido se intercambian los registros de `first` y `middle`, no sus duraciones aisladas. El primer intercambio del ejemplo deja el array así:

```text
Index:  0     1     2     3     4     5     6     7
Clip:   D:30  B:18  C:35  A:42  E:27  F:18  G:50  H:12
        pivot
```

Antes de cada comparación, las regiones tienen este significado. Si un inicio supera a su final, esa región está vacía:

| Región | Qué sabemos de sus elementos |
| --- | --- |
| `first` | Conserva el pivot durante el recorrido. |
| `first + 1..boundary` | Todos tienen duración menor que el pivot. |
| `boundary + 1..scan - 1` | Ya fueron examinados y tienen duración mayor o igual al pivot. |
| `scan..last` | Todavía no han sido examinados. |

**Estado inicial: por qué las variables comienzan allí.** La llamada actual recibió `first = 0` y `last = 7`, porque debe particionar las ocho posiciones. Esos dos límites **no cambian durante esta partición**. `middle` se calculó como 3 y sirvió para seleccionar D:30; después del intercambio inicial, ese registro está en el índice 0. `middle` no avanza ni sigue al pivot cuando se mueve: solo recuerda el índice que usamos para seleccionarlo.

`boundary` y `scan` son variables enteras que guardan índices, no pointers a los clips. Tienen tareas distintas: `scan` indica **qué posición debemos examinar ahora**; `boundary` indica **hasta dónde llega el grupo de clips menores que ya aceptamos**. Como el pivot ocupa `first`, el grupo menor solo puede comenzar en `first + 1`, que aquí es 1. Inicializamos `boundary = first`, es decir, 0: el rango del grupo menor sería `1..0`, que está vacío. Eso no significa que el pivot pertenezca al grupo menor; significa que todavía no aceptamos ningún clip en ese grupo.

Inicializamos `scan = first + 1`, es decir, 1, para examinar B:18. No comenzamos en 0 porque allí está el pivot reservado, y no necesitamos compararlo consigo mismo. El siguiente diagrama muestra el array y, debajo, los índices que guardan las variables en ese instante. Cada número de las filas de variables está alineado con su posición del array; `pivotSeconds = 30` es una duración, no un índice:

```text
Index:    0     1     2     3     4     5     6     7
Clip:     D:30  B:18  C:35  A:42  E:27  F:18  G:50  H:12
first:    0
last:                                               7
boundary: 0
scan:           1

pivotSeconds = 30
Smaller group: empty (indices 1..0)
Next record to examine: B:18, at index 1
```

**Primera comparación: aceptar B:18 y después avanzar.** Con `scan = 1`, leemos la duración de B y comprobamos `18 < 30`. Como es verdadero, B debe incorporarse al grupo menor. La próxima posición de ese grupo es `boundary + 1`, es decir, `0 + 1 = 1`. Por eso el orden de las acciones es:

1. Aumentar `boundary` de 0 a 1 para reservar esa posición.
2. Intercambiar los registros de `boundary` y `scan`, ahora índices 1 y 1. Es un intercambio consigo mismo: B ya está donde corresponde y no se mueve.
3. Al terminar esa vuelta, aumentar `scan` de 1 a 2 para examinar el siguiente registro. `boundary` permanece en 1: solo lo aumentaremos si encontramos otro clip menor que 30.

Después de esas tres acciones, el array sigue igual, pero **lo que sabemos de sus regiones cambió**: B ya fue examinado y aceptado; C todavía no. El estado antes de la siguiente comparación es:

```text
Index:    0     1     2     3     4     5     6     7
Clip:     D:30  B:18  C:35  A:42  E:27  F:18  G:50  H:12
first:    0
last:                                               7
boundary:       1
scan:                 2

pivotSeconds = 30
Smaller group: indices 1..1 -> B:18
Next record to examine: C:35, at index 2
```

El incremento de `boundary` debe ocurrir **antes** del intercambio. Si intercambiáramos usando su valor inicial 0, moveríamos B al lugar reservado para el pivot y sacaríamos D:30 de allí antes de tiempo. En el procedimiento elegido, el pivot debe quedarse en `first` hasta terminar el recorrido.

**Dos comparaciones falsas: `scan` avanza, pero `boundary` no.** Ahora se evalúa C:35 en el índice 2. Como `35 < 30` es falso, no aceptamos ese clip en el grupo menor: no aumentamos `boundary` ni intercambiamos registros. Solo avanzamos `scan` de 2 a 3. En el índice 3 está A:42; `42 < 30` también es falso, de modo que `boundary` sigue en 1 y `scan` avanza de 3 a 4.

El estado siguiente muestra por qué las dos variables no avanzan juntas. Ya examinamos B, C y A, pero solo B cumplió la condición. C y A forman el grupo ya examinado que no es menor que el pivot; E todavía espera su comparación:

```text
Index:    0     1     2     3     4     5     6     7
Clip:     D:30  B:18  C:35  A:42  E:27  F:18  G:50  H:12
first:    0
last:                                               7
boundary:       1
scan:                             4

pivotSeconds = 30
Smaller group: indices 1..1 -> B:18
Next record to examine: E:27, at index 4
Examined, not smaller: indices 2..3 -> C:35, A:42
Not examined yet:     indices 4..7 -> E:27, F:18, G:50, H:12
```

**Siguiente comparación verdadera: incorporar E:27 junto a B:18.** Con `scan = 4`, la comparación `27 < 30` es verdadera. E debe quedar dentro del grupo menor, pero no basta con aumentar `boundary` y dejar E en el índice 4: los índices `1..boundary` deben formar un grupo continuo de menores, sin C:35 ni A:42 en medio. Ejecutamos estas acciones, en este orden:

1. Aumentar `boundary` de 1 a 2. El índice 2 es la posición que incorporaremos al grupo menor; todavía contiene C:35, así que falta completar el intercambio.
2. Intercambiar los registros de los índices 2 y 4. E:27 pasa al índice 2 y C:35 al índice 4. Ahora `1..2` sí contiene únicamente clips menores que 30: B:18 y E:27. C:35 queda con A:42 entre los registros examinados que no son menores.
3. Aumentar `scan` de 4 a 5. La comparación siguiente será con F:18, no con C:35, aunque C acaba de llegar al índice 4: C ya fue examinado y sabemos que no es menor que el pivot.

Después del intercambio y del avance de `scan`, la distribución y las variables quedan así:

```text
Index:    0     1     2     3     4     5     6     7
Clip:     D:30  B:18  E:27  A:42  C:35  F:18  G:50  H:12
first:    0
last:                                               7
boundary:             2
scan:                                   5

pivotSeconds = 30
Smaller group: indices 1..2 -> B:18, E:27
Next record to examine: F:18, at index 5
Examined, not smaller: indices 3..4 -> A:42, C:35
Not examined yet:     indices 5..7 -> F:18, G:50, H:12
```

**Incorporar F:18: el grupo menor gana otra posición.** El estado anterior dejó `scan = 5` y `boundary = 2`. En el índice 5 está F:18, y `18 < 30` es verdadero. El grupo menor ocupa `1..2`; para extenderlo sin dejar huecos, la posición que debemos incorporar es la 3:

1. Aumentar `boundary` de 2 a 3. Esa posición todavía contiene A:42, por lo que falta colocar allí el clip menor que acabamos de encontrar.
2. Intercambiar los registros de los índices 3 y 5. F:18 pasa al índice 3 y A:42 al índice 5. El grupo `1..3` queda formado por B:18, E:27 y F:18, todos menores que 30. C:35 y A:42 quedan fuera de ese grupo, en `4..5`.
3. Aumentar `scan` de 5 a 6 para examinar G:50. No volvemos a comparar A:42: ya sabemos que no es menor que 30.

Después de completar esa vuelta, este es el estado antes de examinar G:

```text
Index:    0     1     2     3     4     5     6     7
Clip:     D:30  B:18  E:27  F:18  C:35  A:42  G:50  H:12
first:    0
last:                                               7
boundary:                   3
scan:                                         6

pivotSeconds = 30
Smaller group: indices 1..3 -> B:18, E:27, F:18
Next record to examine: G:50, at index 6
Examined, not smaller: indices 4..5 -> C:35, A:42
Not examined yet:     indices 6..7 -> G:50, H:12
```

**Examinar G:50: el grupo menor no cambia.** Con `scan = 6`, comprobamos `50 < 30`. Como es falso, G no debe incorporarse al grupo menor. `boundary` permanece en 3 y no se hace ningún intercambio. Al terminar esta vuelta, `scan` aumenta de 6 a 7. El array sigue igual, pero G ahora forma parte de los registros ya examinados que no son menores que el pivot.

El siguiente estado muestra que solo queda H por examinar. Todavía debemos procesarlo porque `scan = 7` cumple la condición `scan <= last`, con `last = 7`:

```text
Index:    0     1     2     3     4     5     6     7
Clip:     D:30  B:18  E:27  F:18  C:35  A:42  G:50  H:12
first:    0
last:                                               7
boundary:                   3
scan:                                               7

pivotSeconds = 30
Smaller group: indices 1..3 -> B:18, E:27, F:18
Next record to examine: H:12, at index 7
Examined, not smaller: indices 4..6 -> C:35, A:42, G:50
Not examined yet:     index 7 -> H:12
```

**Incorporar H:12 y terminar las comparaciones.** Con `scan = 7`, leemos H:12 y comprobamos `12 < 30`. Es verdadero, así que debemos añadir H al grupo menor:

1. Aumentar `boundary` de 3 a 4. Es la posición siguiente al grupo `1..3`, pero todavía contiene C:35.
2. Intercambiar los registros de los índices 4 y 7. H:12 pasa al índice 4 y C:35 al índice 7. El grupo menor ahora ocupa `1..4`; los registros de `5..7` ya fueron examinados y no son menores que 30.
3. Aumentar `scan` de 7 a 8 y volver a comprobar la condición del loop: `8 <= 7` es falso. El recorrido termina **antes** de intentar leer un registro en el índice 8.

El siguiente estado está tomado después de esa última vuelta y antes del intercambio final del pivot. La columna 8 solo muestra el valor de `scan` fuera del rango; no representa una posición existente del array:

```text
Index:    0     1     2     3     4     5     6     7     (8)
Clip:     D:30  B:18  E:27  F:18  H:12  A:42  G:50  C:35
first:    0
last:                                               7
boundary:                         4
scan:                                                     8

pivotSeconds = 30
Smaller group: indices 1..4 -> B:18, E:27, F:18, H:12
Next record to examine: none; scan = 8 is greater than last = 7
Examined, not smaller: indices 5..7 -> A:42, G:50, C:35
Not examined yet:     empty (indices 8..7)
```

Se completaron las siete comparaciones, correspondientes a los índices 1 a 7. `first` siguió en 0 y `last` en 7 durante todo el recorrido; el pivot D:30 continúa reservado en `first`. El avance de `scan` indica que no quedan registros por examinar, mientras que `boundary = 4` indica dónde termina el grupo menor. Con ese estado ya podemos colocar el pivot entre los dos grupos.

Cuando termina el recorrido, `boundary` vale 4. Las posiciones `1..4` contienen los cuatro clips menores que 30, pero no están ordenadas. El pivot todavía está en el índice 0, antes de ese grupo.

Ahora intercambiamos el pivot D:30, ubicado en `first`, con el registro de `boundary`. El registro menor que estaba allí pasa al inicio y el pivot queda después de todos los menores. Esta traza muestra el cambio final:

```text
Before final swap:
0     1     2     3     4     5     6     7
D:30  B:18  E:27  F:18  H:12  A:42  G:50  C:35
pivot                 boundary

After swapping indices 0 and 4:
[H:12 B:18 E:27 F:18] [D:30] [A:42 G:50 C:35]
       left 0..3       index 4       right 5..7
```

La función devuelve `4`. El segmento izquierdo aún necesita ordenar `12, 18, 27, 18`; el derecho, `42, 50, 35`. El pivot ya separa correctamente ambos grupos. El índice central donde se seleccionó era 3, pero su índice final es 4: son dos posiciones con funciones diferentes.

### Conectar las acciones con el cuerpo de la función

Este fragmento corresponde al cuerpo de `partitionClips` y aparecerá integrado en el programa completo de la Parte 3. `clips` será el array de registros y `swapClips` una función que intercambia dos registros completos. No es un programa independiente:

```cpp
int middle = first + (last - first) / 2; // Select the central array position.
swapClips(clips[first], clips[middle]);  // Keep the selected pivot at first.
int pivotSeconds = clips[first].seconds;
int boundary = first; // The smaller-record region is still empty.

for (int scan = first + 1; scan <= last; scan++)
{
    if (clips[scan].seconds < pivotSeconds)
    {
        boundary++; // Reserve the next position after the smaller-record region.
        swapClips(clips[boundary], clips[scan]);
    }
}

swapClips(clips[first], clips[boundary]); // Place the pivot between both groups.
return boundary; // Report the pivot's final index for this partition.
```

Comenzar en `first + 1` evita comparar el pivot consigo mismo y usar `<= last` incluye el último registro, que ya no es el pivot reservado. Si `boundary` y `scan` coinciden, intercambiar una posición consigo misma es válido y no cambia sus datos. Al final, `boundary` puede ser `first` si ningún registro era menor, o `last` si todos lo eran.

<details>
<summary>Verificación: con pivot de 18 segundos, ¿a qué grupo pertenece otro clip que también dura 18?</summary>

Con la comparación estricta `<`, ese otro clip no entra al grupo menor. Queda en el grupo mayor o igual, a la derecha del pivot cuando termina la partición. Eso no elimina el empate: quicksort todavía debe conservar y ordenar todos los registros.

</details>

## Parte 3 — Quicksort completo: límites, llamadas y retornos (30 min)

Ya trazamos una partición y sabemos que devuelve un índice de pivot. Todavía falta ordenar el interior de sus grupos. Conectaremos ese resultado con las llamadas recursivas, mostraremos el programa completo y seguiremos la ejecución hasta los casos base para verificar que ningún rango se repite indefinidamente.

### Qué representa cada pieza del programa

Declararemos un `struct` llamado `VideoClip` para representar **un clip**, con `clipId` y `seconds`. El array `clips` contendrá los ocho registros del Centro de Medios. Las funciones serán libres, con prototipos antes de `main` y definiciones después; ninguna pertenece a una clase ni requiere `::`.

`main` será el punto de inicio del programa: creará el array, pedirá mostrar su contenido, solicitará ordenarlo y pedirá mostrarlo otra vez. Para repartir esas tareas, definiremos las cuatro funciones de la tabla. La columna de quién las llama permite distinguir las operaciones solicitadas desde `main` de los pasos internos del algoritmo:

| Función | Quién la llama y para qué | Qué recibe y qué resultado produce |
| --- | --- | --- |
| `displayClips` | `main`, para comprobar el antes y el después. | Recibe el array y su cantidad lógica; imprime IDs y duraciones sin modificar los registros. |
| `quickSortClips` | Primero `main`, para ordenar todo el catálogo; después ella misma, para ordenar cada segmento pendiente. | Recibe el array y los límites inclusivos `first` y `last`. Modifica ese segmento hasta dejarlo ordenado; no devuelve un valor. |
| `partitionClips` | `quickSortClips`, cuando el segmento tiene al menos dos registros y necesita separar sus grupos. | Recibe el array y esos límites. Reorganiza el segmento respecto al pivot y devuelve su índice final como un entero. |
| `swapClips` | `partitionClips`, para mover el pivot o colocar un registro menor en su grupo. | Recibe dos registros por referencia e intercambia sus IDs y duraciones como unidades completas; no devuelve un valor. |

Sigamos el pedido inicial de ordenamiento. `main` ejecutará `quickSortClips(clips, 0, CLIP_COUNT - 1)`. Como hay ocho clips, la función recibirá `first = 0` y `last = 7`. Esos parámetros le indican **qué parte del catálogo debe dejar ordenada**; no significan que ya conozca el orden de sus registros.

Al entrar en **cada llamada a `quickSortClips`**, la función comprueba `first >= last`. Si es verdadero, recibió cero o un registro y regresa sin particionar. Si es falso, entonces `first < last`: hay por lo menos dos registros y todavía necesita trabajar. En ese caso, **`quickSortClips` llama a `partitionClips` con el mismo array y los límites de su propio segmento**.

Para el pedido inicial, `partitionClips(clips, 0, 7)` realiza el recorrido que acabamos de trazar: selecciona D:30 en el índice central 3, lo mueve al inicio, organiza los grupos y finalmente coloca D:30 en el índice 4. Después devuelve el entero `4`. Al regresar a `quickSortClips`, ese resultado se guarda en una variable local llamada `pivotIndex`.

Ese retorno no significa que `0..7` ya esté ordenado. Significa que el pivot está colocado y que conocemos los límites de los dos grupos que aún debemos ordenar. Por eso **`quickSortClips` se llama a sí misma** una vez para ordenar el grupo izquierdo y otra para ordenar el derecho. Sus nuevos límites se calculan con `pivotIndex`, la posición final del pivot, **no con `middle`, la posición donde se seleccionó inicialmente**.

La siguiente tabla describe los argumentos de esas dos llamadas recursivas a `quickSortClips`. Los valores de ejemplo corresponden al retorno `pivotIndex = 4` dentro de la llamada original `0..7`:

| Llamada recursiva a `quickSortClips` | Su nuevo `first` | Su nuevo `last` | Pedido concreto en este ejemplo |
| --- | --- | --- | --- |
| Para ordenar el grupo izquierdo | `first` de quien la llama | `pivotIndex - 1` | Recibir `0..3` y ordenar `[H:12 B:18 E:27 F:18]`. |
| Para ordenar el grupo derecho | `pivotIndex + 1` | `last` de quien la llama | Recibir `5..7` y ordenar `[A:42 G:50 C:35]`. |

La llamada original a `quickSortClips` sobre `0..7` espera a que termine por completo la llamada sobre `0..3`; luego inicia la llamada sobre `5..7`. Cada una repite el mismo procedimiento con sus propios límites: comprobar el caso base, solicitar una partición si hace falta y ordenar sus grupos. El índice 4 no entra en ninguna de las dos llamadas porque D:30 ya separa correctamente los valores menores de los mayores o iguales.

Cada segmento nuevo contiene menos registros que el de la llamada que lo originó. Si un grupo queda vacío, sus límites también se pasan a `quickSortClips`, cuya comprobación inicial lo detecta y regresa. Cuando las dos llamadas recursivas terminan, la llamada original también termina y devuelve el control a `main`; en ese momento `main` puede mostrar el catálogo completo ya ordenado.

### Programa ejecutable

Guarda este programa completo como `video_clip_quicksort.cpp`. Usa los datos controlados de la traza para comprobar los mismos movimientos. El programa modifica el array `clips`; si una aplicación necesitara preservar su orden original, podría ordenar una vista temporal como plantea [Lab 11](../labs/Lab11_Recursion_Merge_Sort.md), donde la práctica del taller de conservación propone ordenar pointers a tareas sin mover sus registros originales.

```cpp
#include <iostream>
#include <string>
using namespace std;

struct VideoClip
{
    string clipId; // Preserve the identity of each video clip.
    int seconds;   // Use duration as the ascending sort key.
};

void swapClips(VideoClip& left, VideoClip& right);
int partitionClips(VideoClip clips[], int first, int last);
void quickSortClips(VideoClip clips[], int first, int last);
void displayClips(const VideoClip clips[], int count);

int main()
{
    const int CLIP_COUNT = 8;
    VideoClip clips[CLIP_COUNT] =
    {
        {"VC-101", 42}, {"VC-102", 18},
        {"VC-103", 35}, {"VC-104", 30},
        {"VC-105", 27}, {"VC-106", 18},
        {"VC-107", 50}, {"VC-108", 12}
    };

    cout << "Before sorting:" << endl;
    displayClips(clips, CLIP_COUNT);
    quickSortClips(clips, 0, CLIP_COUNT - 1); // Include all eight valid positions.
    cout << "After sorting by duration:" << endl;
    displayClips(clips, CLIP_COUNT);
    return 0;
}

void swapClips(VideoClip& left, VideoClip& right)
{
    VideoClip temporary = left; // Save the entire record before overwriting it.
    left = right;
    right = temporary;
}

// Precondition: first..last is a valid nonempty segment of clips.
int partitionClips(VideoClip clips[], int first, int last)
{
    int middle = first + (last - first) / 2;
    swapClips(clips[first], clips[middle]); // Move the selected central record to first.
    int pivotSeconds = clips[first].seconds;
    int boundary = first;
    for (int scan = first + 1; scan <= last; scan++)
    {
        if (clips[scan].seconds < pivotSeconds)
        {
            boundary++; // Extend the smaller-record region before placing this clip.
            swapClips(clips[boundary], clips[scan]);
        }
    }
    swapClips(clips[first], clips[boundary]); // The pivot now separates both regions.
    return boundary;
}

void quickSortClips(VideoClip clips[], int first, int last)
{
    if (first >= last)
    {
        return; // Base case: zero or one clip needs no partition.
    }

    int pivotIndex = partitionClips(clips, first, last);
    quickSortClips(clips, first, pivotIndex - 1); // Finish the left region first.
    quickSortClips(clips, pivotIndex + 1, last);  // Then finish the right region.
}

void displayClips(const VideoClip clips[], int count)
{
    for (int index = 0; index < count; index++)
    {
        cout << "Clip: " << clips[index].clipId
             << " | Seconds: " << clips[index].seconds << endl;
    }
}
```

`VideoClip&` permite modificar los registros originales recibidos por `swapClips`; no se intercambian simples copias locales. La variable `temporary` conserva los dos members mientras se sobrescribe una posición. Es una sola variable auxiliar para un registro, no un array proporcional al número de clips.

Para compilar, usa este comando; `-o` indica el nombre del ejecutable:

```bash
g++ video_clip_quicksort.cpp -o video_clip_quicksort
```

Para ejecutarlo desde la misma carpeta:

```bash
./video_clip_quicksort
```

El output permite comprobar tanto el criterio de duración como la identidad de los clips:

```text
Before sorting:
Clip: VC-101 | Seconds: 42
Clip: VC-102 | Seconds: 18
Clip: VC-103 | Seconds: 35
Clip: VC-104 | Seconds: 30
Clip: VC-105 | Seconds: 27
Clip: VC-106 | Seconds: 18
Clip: VC-107 | Seconds: 50
Clip: VC-108 | Seconds: 12
After sorting by duration:
Clip: VC-108 | Seconds: 12
Clip: VC-102 | Seconds: 18
Clip: VC-106 | Seconds: 18
Clip: VC-105 | Seconds: 27
Clip: VC-104 | Seconds: 30
Clip: VC-103 | Seconds: 35
Clip: VC-101 | Seconds: 42
Clip: VC-107 | Seconds: 50
```

El orden es correcto por duración y cada ID conserva sus datos. En esta entrada B/VC-102 permanece antes que F/VC-106 entre los clips de 18 segundos. Eso no basta para afirmar que el algoritmo es estable: una garantía debe cumplirse para cualquier entrada válida. En la Parte 4 mostraremos una entrada pequeña donde esta misma implementación invierte un empate.

### Trazar las llamadas y evaluar el caso base

La primera partición dejó D:30 en el índice 4, pero eso solo completó el trabajo inicial de `quickSortClips(clips, 0, 7)`. Para seguir lo que falta, debemos distinguir **entrar en una llamada nueva** de **reanudar una llamada que estaba esperando**. Ambas acciones aparecen en la ejecución, aunque solo la primera evalúa de nuevo el caso base.

En la tabla, `Q(first,last)` abrevia `quickSortClips(clips, first, last)`. Todas las llamadas trabajan sobre el **mismo array `clips`**. La indentación de la columna de llamada indica su profundidad: una llamada más adentro fue solicitada por la del nivel anterior. Una fila que dice “Reanudar” conserva esa profundidad porque continúa una llamada existente; no crea otra llamada ni vuelve a particionar su rango.

La tabla incluye las once llamadas a `quickSortClips`, incluso las de un elemento y las vacías, y las diez reanudaciones posteriores a sus llamadas hijas. Cada partición se describe como una operación completa: sus comparaciones e intercambios internos siguen el recorrido desarrollado en la Parte 2. No se repiten esas instrucciones en esta tabla, cuyo propósito es mostrar **el orden de llamadas, casos base y retornos**.

| Paso de ejecución | Llamada y profundidad | Qué se ejecuta y qué ocurre después |
| ---: | --- | --- |
| 1 | `Q(0,7)` — entrar desde `main` | Recibe `A:42 B:18 C:35 D:30 E:27 F:18 G:50 H:12`. `0 >= 7` es falso. Particiona con D:30, seleccionado en el centro 3, y recibe `pivotIndex = 4`. El array queda `H:12 B:18 E:27 F:18 D:30 A:42 G:50 C:35`. Solicita su izquierda `Q(0,3)` y espera. |
| 2 | &emsp;↳ `Q(0,3)` — entrar | Recibe `H:12 B:18 E:27 F:18`. `0 >= 3` es falso. Particiona con B:18, del centro 1, y recibe `pivotIndex = 1`; ese segmento queda `H:12 B:18 E:27 F:18`. Solicita `Q(0,0)` y espera. |
| 3 | &emsp;&emsp;↳ `Q(0,0)` — entrar | Contiene H:12. `0 >= 0` es verdadero: caso base de un elemento. No particiona ni cambia el array; ejecuta `return` y devuelve el control a `Q(0,3)`. |
| 4 | &emsp;`Q(0,3)` — reanudar | Continúa justo después de su llamada izquierda. Su pivot B:18 sigue en el índice 1. Ahora solicita la derecha `Q(2,3)` y vuelve a esperar. |
| 5 | &emsp;&emsp;↳ `Q(2,3)` — entrar | Recibe `E:27 F:18`. `2 >= 3` es falso. Particiona con E:27, del centro 2, y recibe `pivotIndex = 3`. El segmento queda `F:18 E:27`. Solicita `Q(2,2)` y espera. |
| 6 | &emsp;&emsp;&emsp;↳ `Q(2,2)` — entrar | Contiene F:18. `2 >= 2` es verdadero: regresa sin modificarlo y devuelve el control a `Q(2,3)`. |
| 7 | &emsp;&emsp;`Q(2,3)` — reanudar | Continúa después de su izquierda. Su pivot E:27 sigue en el índice 3. Solicita la derecha `Q(4,3)` y espera. |
| 8 | &emsp;&emsp;&emsp;↳ `Q(4,3)` — entrar | El rango está vacío. `4 >= 3` es verdadero: regresa sin acceder al array y devuelve el control a `Q(2,3)`. |
| 9 | &emsp;&emsp;`Q(2,3)` — reanudar | Ya terminaron sus dos llamadas hijas. Las posiciones `2..3` contienen `F:18 E:27`. No mueve ni combina registros al regresar: llega al final de la función y devuelve el control a `Q(0,3)`. |
| 10 | &emsp;`Q(0,3)` — reanudar | Ya terminó su derecha. Las posiciones `0..3` contienen `H:12 B:18 F:18 E:27`. No hace otro intercambio; llega al final y devuelve el control a `Q(0,7)`. |
| 11 | `Q(0,7)` — reanudar | Su izquierda está ordenada en el mismo array y D:30 sigue en el índice 4. Todavía falta su derecha: solicita `Q(5,7)` y espera. |
| 12 | &emsp;↳ `Q(5,7)` — entrar | Recibe `A:42 G:50 C:35`. `5 >= 7` es falso. Particiona con G:50, del centro 6, y recibe `pivotIndex = 7`. El segmento queda `C:35 A:42 G:50`. Solicita `Q(5,6)` y espera. |
| 13 | &emsp;&emsp;↳ `Q(5,6)` — entrar | Recibe `C:35 A:42`. `5 >= 6` es falso. Particiona con C:35, del centro 5, y recibe `pivotIndex = 5`. El segmento sigue `C:35 A:42`. Solicita `Q(5,4)` y espera. |
| 14 | &emsp;&emsp;&emsp;↳ `Q(5,4)` — entrar | El rango está vacío. `5 >= 4` es verdadero: regresa sin acceder al array y devuelve el control a `Q(5,6)`. |
| 15 | &emsp;&emsp;`Q(5,6)` — reanudar | Continúa después de su izquierda vacía. Su pivot C:35 sigue en el índice 5. Solicita la derecha `Q(6,6)` y espera. |
| 16 | &emsp;&emsp;&emsp;↳ `Q(6,6)` — entrar | Contiene A:42. `6 >= 6` es verdadero: regresa sin modificarlo y devuelve el control a `Q(5,6)`. |
| 17 | &emsp;&emsp;`Q(5,6)` — reanudar | Ya terminaron sus dos llamadas hijas. `5..6` contiene `C:35 A:42`; no necesita otra operación. Llega al final y devuelve el control a `Q(5,7)`. |
| 18 | &emsp;`Q(5,7)` — reanudar | Ya terminó su izquierda; G:50 sigue en el índice 7. Solicita su derecha `Q(8,7)` y espera. |
| 19 | &emsp;&emsp;↳ `Q(8,7)` — entrar | El rango está vacío. `8 >= 7` es verdadero: regresa sin acceder al array y devuelve el control a `Q(5,7)`. |
| 20 | &emsp;`Q(5,7)` — reanudar | Ya terminaron sus dos llamadas hijas. `5..7` contiene `C:35 A:42 G:50`; llega al final y devuelve el control a `Q(0,7)`, sin copiar ni unir segmentos. |
| 21 | `Q(0,7)` — reanudar | Ya terminó también su derecha. El array completo está ordenado. Llega al final y devuelve el control a `main`, que continúa con el encabezado y la llamada a `displayClips`. |

### Qué pasa con el array cuando hay un return

`partitionClips` devuelve un **valor**, el índice final del pivot. `quickSortClips`, en cambio, es `void`: cuando ejecuta `return;` o llega al final de su cuerpo, devuelve **el control de ejecución**, sin entregar otro array ni un segmento nuevo.

Las llamadas recursivas no construyeron arrays separados. Sus parámetros `first` y `last` describen regiones del array original, y sus intercambios ya modificaron ese almacenamiento. Por eso, cuando una llamada termina, los registros que ordenó **ya están en sus posiciones dentro de `clips`**. Al regresar desaparece su estado local, pero no se deshacen sus intercambios.

El siguiente esquema sigue las mismas llamadas de la tabla y añade los puntos donde se reanuda la llamada anterior. Las líneas `clips:` muestran el estado del array compartido; las barras `|` solo separan visualmente regiones para explicar su estado, no arrays distintos ni una operación que las una. En los casos base se indica qué registro permanece en su posición o si el rango está vacío; esos registros no son valores devueltos por la función:

```text
Q(0,7): central pivot D:30 -> final index 4
    clips: [H:12 B:18 E:27 F:18] | D:30 | [A:42 G:50 C:35]
    Q(0,3): central pivot B:18 -> final index 1
        Q(0,0): 0 >= 0 is true; H:12 stays at index 0
            return control to Q(0,3); no records move
        Q(0,3) resumes after its left call; now call its right side:
        Q(2,3): central pivot E:27 -> final index 3
            Q(2,2): 2 >= 2 is true; F:18 stays at index 2
                return control to Q(2,3); no records move
            Q(2,3) resumes after its left call; now call its right side:
            Q(4,3): 4 >= 3 is true; empty range
                return control to Q(2,3); no array access
            Q(2,3) resumes after its right call; both calls are finished
            indices 2..3 already contain F:18 E:27; no merge or copy
            finish Q(2,3); return control to Q(0,3)
        Q(0,3) resumes after its right call; both calls are finished
        indices 0..3 already contain H:12 B:18 F:18 E:27
        finish Q(0,3); return control to Q(0,7); no records move
    Q(0,7) resumes after its left call:
    clips: [H:12 B:18 F:18 E:27] | D:30 | [A:42 G:50 C:35]
           left side sorted       fixed  right side still pending
    Q(0,7) now calls its right side:
    Q(5,7): central pivot G:50 -> final index 7
        Q(5,6): central pivot C:35 -> final index 5
            Q(5,4): 5 >= 4 is true; empty range
                return control to Q(5,6); no array access
            Q(5,6) resumes after its left call; now call its right side:
            Q(6,6): 6 >= 6 is true; A:42 stays at index 6
                return control to Q(5,6); no records move
            Q(5,6) resumes after its right call; both calls are finished
            finish Q(5,6); return control to Q(5,7); no merge or copy
        Q(5,7) resumes after its left call; now call its right side:
        Q(8,7): 8 >= 7 is true; empty range
            return control to Q(5,7); no array access
        Q(5,7) resumes after its right call; both calls are finished
        indices 5..7 already contain C:35 A:42 G:50
        finish Q(5,7); return control to Q(0,7); no records move
    Q(0,7) resumes after its right call:
    clips: [H:12 B:18 F:18 E:27] | D:30 | [C:35 A:42 G:50]
           left side sorted       fixed  right side sorted
    both calls are finished; no merge, concatenation, or copy
finish Q(0,7); return control to main
main displays the array that has already been sorted
```

¿Por qué el array completo queda ordenado sin unir partes? La partición inicial garantizó que todos los registros de `0..3` fueran menores que D:30 y todos los de `5..7` fueran mayores o iguales. Las llamadas siguientes solo intercambian dentro de sus propios límites: la izquierda ordena sus cuatro registros sin mover el pivot del índice 4 ni tocar la derecha; la derecha hace lo mismo en `5..7`. Al terminar ambas, ya tenemos en posiciones consecutivas un grupo menor ordenado, el pivot y un grupo mayor o igual ordenado. Esa relación basta para que `0..7` esté ordenado.

En merge sort, separar por índices no garantizaba esa relación entre los valores de las mitades: por eso había que combinar comparándolas después de ordenar. En quicksort, la partición estableció esa relación **antes** de las llamadas recursivas. El `return` no hace la combinación: simplemente permite que la llamada anterior continúe o que termine si ya completó sus dos lados.

Si incluyéramos otra vez el pivot en un lado, podríamos repetir un rango completo cuando el pivot termina en un extremo. Excluirlo garantiza que el problema se reduzca incluso con entrada ordenada o con empates.

<details>
<summary>Verificación: para el rango <code>5..6</code> cuyo pivot queda en el índice 5, ¿qué recibe cada llamada y cuál regresa inmediatamente?</summary>

La izquierda recibe `5..4`, un segmento vacío: `5 >= 4` es verdadero y regresa sin acceder al array. La derecha recibe `6..6`, con un solo registro: `6 >= 6` también es verdadero y regresa sin particionar. La llamada original conserva `5..6` mientras espera esos resultados.

</details>

## Parte 4 — Comparar algoritmos: tiempo, memoria y estabilidad (20 min)

El programa de quicksort ya produce el catálogo ordenado. Ahora debemos decidir cuándo conviene esa estrategia y qué propiedades perdimos respecto al merge sort de la semana anterior. Analizaremos el trabajo de la partición, el efecto del pivot y la memoria de las llamadas antes de resumir los métodos en una tabla.

### Particiones equilibradas frente a particiones extremas

Aquí `n` es el número de clips que se ordenan. Particionar un segmento de `m` clips examina `m - 1` registros y realiza intercambios de costo acotado para estos datos: su trabajo es `O(m)`. Este análisis trata la comparación de segundos y la copia de los registros con IDs de tamaño acotado como operaciones de costo constante.

Si el pivot produce grupos de tamaños parecidos, la profundidad de división crece como `log n` y el trabajo total por nivel es proporcional a `n`. En ese comportamiento equilibrado, quicksort toma `O(n log n)` tiempo. También tiene tiempo esperado `O(n log n)` bajo supuestos adecuados sobre la distribución de la entrada o una selección aleatoria del pivot; seleccionar siempre la posición central no asegura esos supuestos para cualquier entrada.

Si el pivot queda siempre en un extremo, los tamaños pendientes pueden ser `n - 1`, `n - 2`, y así sucesivamente. El trabajo se acumula como `n + (n - 1) + ... + 1`, cuyo crecimiento es `O(n²)`. Elegir el índice central evita seleccionar un extremo por valor en una entrada ascendente de valores distintos, pero no elimina entradas desfavorables. Una colección de valores todos iguales sigue siendo un ejemplo: ningún elemento cumple `< pivot` y el pivot queda al comienzo en cada llamada, aunque originalmente se haya seleccionado en el centro.

### Memoria adicional no equivale a cero

La partición usa unos pocos índices y una variable temporal para intercambiar registros. Puede decirse que reorganiza **in-place**, es decir, sobre el mismo almacenamiento del array, sin un segundo array de `n` registros. Sin embargo, la implementación recursiva mantiene llamadas activas: estas también cuentan como espacio auxiliar.

En divisiones equilibradas, las llamadas requieren `O(log n)` espacio. En el peor caso de la versión presentada pueden formar una cadena de profundidad `O(n)`. Por tanto, describir esta implementación completa como de espacio auxiliar `O(1)` sería incorrecto.

### Estabilidad y recordatorio de los métodos básicos

Un ordenamiento **estable** conserva el orden original entre registros que empatan en el criterio. Que los empates se hayan conservado en la entrada de ocho clips no garantiza estabilidad. Para comprobarlo, consideremos una entrada más pequeña con los mismos clips B:18, F:18 y H:12, en ese orden. Esta traza aplica exactamente la misma selección central y partición:

```text
Input:                         [B:18 F:18 H:12]
Select central F:18; swap 0,1: [F:18 B:18 H:12]
Scan B:18: 18 < 18? false.
Scan H:12: 12 < 18? true; swap indices 1 and 2:
                               [F:18 H:12 B:18]
Place pivot; swap 0,1:         [H:12 F:18 B:18]
```

Las dos llamadas siguientes reciben un solo registro cada una y regresan. F:18 terminó antes que B:18, invirtiendo el empate original. Por tanto, esta implementación no es estable; eso importa si el orden de llegada debe conservarse entre empates.

> **Recordatorio de merge sort:** divide el segmento, ordena las mitades y las combina. La versión de arrays de Semana 11 usa un auxiliar de `n` registros y elige la izquierda en los empates; es estable y toma `O(n log n)` tiempo incluso en su peor caso. La versión con linked lists reconecta nodes, con merge iterativo y espacio auxiliar `O(log n)` por la recursión.

> **Recordatorio de selection sort:** busca el menor de la región pendiente y lo intercambia con su primera posición. Repite esas búsquedas, de ahí su tiempo `O(n²)`. La versión básica con intercambios puede invertir empates; por ejemplo, `[A:2, B:2, C:1]` pasa a `[C:1, B:2, A:2]` al mover el mínimo al inicio.

> **Recordatorio de bubble sort:** compara vecinos y los intercambia si están fuera de orden. Su peor caso es `O(n²)`. Si se detiene al completar una pasada sin intercambios, una entrada ya ordenada toma `O(n)` tiempo. Es estable si intercambia solo cuando el vecino izquierdo es estrictamente mayor, conservando los empates.

La tabla compara las implementaciones descritas para arrays. En todas las columnas de memoria se cuenta el call stack, y se excluye la colección original:

| Método | Mejor caso de tiempo | Peor caso de tiempo | Espacio auxiliar en el peor caso | ¿Conserva el orden original entre empates? |
| --- | --- | --- | --- | --- |
| Selection sort básico | `O(n²)` | `O(n²)` | `O(1)` | No. |
| Bubble sort con salida anticipada | `O(n)` | `O(n²)` | `O(1)` | Sí, con intercambio estricto. |
| Merge sort de arrays, Semana 11 | `O(n log n)` | `O(n log n)` | `O(n)` | Sí, eligiendo izquierda en empates. |
| Quicksort presentado | `O(n log n)` | `O(n²)` | `O(n)` | No. |

### Cuál conviene elegir, en qué casos y por qué

Para responder cuál es mejor, primero debemos precisar **qué queremos mejorar: el tiempo de ordenamiento, la memoria adicional o la cantidad de intercambios**. En el catálogo de clips no es igual ordenar miles de registros con un tiempo predecible que revisar una colección pequeña usando muy poca memoria. La elección debe relacionarse con ese objetivo concreto.

La siguiente tabla convierte los valores de complejidad en una decisión. Cada fila indica qué método conviene para una necesidad, qué ventaja ofrece frente a las otras opciones y qué costo aceptamos. Se refiere a las implementaciones estudiadas, sin garantizar que un método tarde menos segundos en toda entrada:

| Si la prioridad es… | ¿Cuál conviene elegir? | Ventaja que justifica la elección | Costo o condición que aceptamos |
| --- | --- | --- | --- |
| Ordenar muchos registros manteniendo buen crecimiento de tiempo incluso en el peor caso. | **Merge sort.** | Garantiza `O(n log n)` tiempo, frente al peor caso `O(n²)` de los otros tres métodos de la tabla anterior. | En arrays requiere espacio auxiliar `O(n)`. |
| Obtener tiempo `O(n log n)` y usar menos memoria que merge sort de arrays cuando las particiones son equilibradas. | **Quicksort.** | En ese comportamiento combina tiempo `O(n log n)` con espacio auxiliar `O(log n)`, frente al espacio `O(n)` de merge sort de arrays. | El equilibrio no está garantizado: puede llegar a tiempo `O(n²)` y espacio `O(n)`. No se exige conservar empates. |
| Terminar pronto cuando los datos ya llegan ordenados. | **Bubble sort con salida anticipada.** | Una pasada sin intercambios permite terminar en tiempo `O(n)` y espacio `O(1)`; selection sort sigue haciendo `O(n²)` comparaciones y merge sort sigue dividiendo en `O(n log n)` tiempo. | Esa ventaja es para la entrada ya ordenada. Otras entradas pueden requerir `O(n²)` tiempo; “casi ordenado” no garantiza una sola pasada. |
| Reducir intercambios en una colección pequeña cuando mover registros resulta costoso. | **Selection sort básico.** | Realiza como máximo `n - 1` intercambios efectivos: `O(n)` intercambios, frente a los `O(n²)` que puede hacer bubble sort. | Su tiempo total sigue siendo `O(n²)` por las comparaciones y usa espacio `O(1)`. Menos intercambios no equivale a menos trabajo total. |
| Garantizar memoria adicional constante y aceptar tiempo cuadrático para una colección pequeña. | **Selection sort o bubble sort.** | Ambos garantizan espacio auxiliar `O(1)` en estas versiones iterativas. Merge sort de arrays requiere `O(n)` y quicksort recursivo puede requerir `O(n)` en el peor caso. | Aceptamos peor caso de tiempo `O(n²)`. Entre ambos, selection limita intercambios y bubble con salida anticipada aprovecha la entrada ya ordenada. |
| Ordenar una linked list con buen crecimiento de tiempo y sin reservar otro conjunto de `n` registros. | **Merge sort con nodes**, como en Semana 11. | Ofrece tiempo `O(n log n)` y espacio auxiliar `O(log n)` con combinación iterativa; reutiliza los nodes y reconecta sus enlaces. | Se permite cambiar el orden de los nodes originales. La función de quicksort con índices de esta guía no se aplica directamente a un `Node *`. |

Por ejemplo, si el Centro de Medios pregunta qué elegir para un catálogo grande y quiere evitar el peor caso cuadrático, responderíamos: **merge sort, porque garantiza tiempo `O(n log n)`, a cambio de memoria auxiliar `O(n)` en arrays**. Si la restricción principal fuera memoria auxiliar estrictamente constante para ocho registros, selection o bubble sort serían alternativas compatibles, aceptando su peor caso `O(n²)`.

Quicksort y merge sort pueden compartir tiempo `O(n log n)` cuando quicksort divide de forma equilibrada; ese dato no demuestra que quicksort sea siempre más rápido. En esa comparación, quicksort puede ahorrar memoria auxiliar, mientras merge sort conserva una garantía de tiempo ante entradas desfavorables. Para elegir por velocidad real entre opciones que cumplen los requisitos, podemos medirlas con datos representativos.

La respuesta útil es, por tanto, **“este método conviene para esta prioridad, por este costo de tiempo o memoria, a cambio de esta limitación”**. Compara siempre la misma medida: el `O(log n)` de memoria no debe presentarse como si fuera un tiempo de ordenamiento mejor que `O(n log n)`.

<details>
<summary>Pregunta de decisión: si un reporte exige conservar el orden de llegada entre empates, ¿basta que quicksort entregue duraciones ascendentes?</summary>

No. Debe satisfacerse también la estabilidad o definirse explícitamente un criterio adicional que reproduzca el orden deseado. La versión de merge sort estudiada conserva los empates; la de quicksort presentada no lo garantiza.

</details>

## Parte 5 — Searching: localizar datos y justificar la estrategia (25 min)

Ya podemos ordenar registros y comparar los costos de hacerlo. El Centro de Medios todavía necesita localizar clips, y ordenar no responde por sí mismo una consulta. Retomaremos linear search y binary search, vistas en el análisis de Big-O, para implementar sus decisiones, trazar sus límites y explicar cuándo el costo previo de ordenar se justifica. La idea de descartar una región preparará la introducción a los árboles.

### Qué se busca y qué devuelve una búsqueda

Supongan que el personal necesita saber si existe un clip de exactamente cierta cantidad de segundos. Para concentrarnos en los algoritmos de búsqueda, este programa usará solo las ocho duraciones ya ordenadas del catálogo: `[12, 18, 18, 27, 30, 35, 42, 50]`. Es una versión reducida del ejemplo de clips, no una nueva asociación entre IDs y tiempos.

La duración buscada se solicitará por consola para probar tanto coincidencias como valores ausentes. Las funciones devolverán **un índice válido** si encuentran el valor y `-1` si no existe. `-1` es una señal de resultado, no una posición que pueda leerse. Antes de usar un índice devuelto, el programa cliente debe comprobarlo.

**Linear search** compara el objetivo con cada elemento desde el comienzo y termina cuando encuentra una coincidencia o agota la colección. No exige datos ordenados. La versión que mostraremos devuelve la primera coincidencia del array.

**Binary search** compara el objetivo con el elemento central de un rango ordenado. Si no coincide, descarta una mitad que no puede contenerlo. Requiere que los datos estén ordenados por el mismo campo y en el sentido que espera el algoritmo; aquí, duración ascendente. La versión básica puede devolver cualquiera de las coincidencias si hay repetidos.

### Cómo se obtienen los siguientes límites

En una búsqueda binaria, `first..last` delimita las posiciones que todavía pueden contener el objetivo. Si `first > last`, no queda ninguna: el valor no fue encontrado. Si el segmento tiene un elemento, todavía debemos compararlo; no podemos darlo por encontrado solo porque `first == last`.

La tabla muestra qué información aporta comparar con `durations[middle]` y cómo modifica el rango:

| Comparación | Qué se puede concluir por el orden ascendente | Próximo rango |
| --- | --- | --- |
| El valor central coincide con el objetivo. | Encontramos una posición válida. | Devolver `middle`. |
| El valor central es menor que el objetivo. | El centro y todo lo anterior son demasiado pequeños. | `middle + 1..last`. |
| El valor central es mayor que el objetivo. | El centro y todo lo posterior son demasiado grandes. | `first..middle - 1`. |

El centro se excluye porque ya fue comparado. La versión iterativa cambia sus índices locales dentro de un loop; la recursiva pasa esos nuevos límites a otra llamada y devuelve su resultado.

### Programa completo con tres búsquedas

Guarda este programa como `clip_duration_search.cpp`. Las funciones libres se declaran antes de `main` y se definen después. `linearSearch` acepta un array y una cantidad; `binarySearchIterative` también recibe una cantidad y establece sus límites; `binarySearchRecursive` recibe directamente los límites inclusivos.

Antes de buscar, `main` comprueba la lectura por consola. La expresión `cin >> target` intenta extraer un entero y su resultado permite comprobar si la lectura fue válida; el operador `!` invierte esa condición. Por eso `if (!(cin >> target))` entra al bloque de error si no pudo leerse el objetivo, evitando usar un valor sin inicializar.

```cpp
#include <iostream>
using namespace std;

int linearSearch(const int durations[], int count, int target);
int binarySearchIterative(const int durations[], int count, int target);
int binarySearchRecursive(const int durations[], int first, int last, int target);

int main()
{
    const int CLIP_COUNT = 8;
    int durations[CLIP_COUNT] = {12, 18, 18, 27, 30, 35, 42, 50};
    int target;

    cout << "Enter a duration in seconds: ";
    if (!(cin >> target))
    {
        cout << "Invalid duration input." << endl;
        return 1; // Do not search with an unread target value.
    }

    cout << "Linear search index: " << linearSearch(durations, CLIP_COUNT, target) << endl;
    cout << "Iterative binary search index: "
         << binarySearchIterative(durations, CLIP_COUNT, target) << endl;
    cout << "Recursive binary search index: "
         << binarySearchRecursive(durations, 0, CLIP_COUNT - 1, target) << endl;
    return 0;
}

int linearSearch(const int durations[], int count, int target)
{
    for (int index = 0; index < count; index++)
    {
        if (durations[index] == target)
        {
            return index; // Stop at the first match in array order.
        }
    }
    return -1; // No array element matched the target.
}

// Precondition: durations[0..count-1] is sorted in ascending order.
int binarySearchIterative(const int durations[], int count, int target)
{
    int first = 0;
    int last = count - 1;
    while (first <= last)
    {
        int middle = first + (last - first) / 2;
        if (durations[middle] == target)
        {
            return middle;
        }
        if (durations[middle] < target)
        {
            first = middle + 1; // Discard the center and the lower half.
        }
        else
        {
            last = middle - 1; // Discard the center and the upper half.
        }
    }
    return -1; // The remaining range became empty.
}

// Precondition: first..last is a valid sorted segment, or an empty range.
int binarySearchRecursive(const int durations[], int first, int last, int target)
{
    if (first > last)
    {
        return -1; // Base case: no possible position remains.
    }

    int middle = first + (last - first) / 2;
    if (durations[middle] == target)
    {
        return middle; // Base case: a matching position has been found.
    }
    if (durations[middle] < target)
    {
        return binarySearchRecursive(durations, middle + 1, last, target);
    }
    return binarySearchRecursive(durations, first, middle - 1, target);
}
```

`cin >> target` intenta leer el entero. La condición `!(cin >> target)` detecta que la lectura no pudo completarse; el programa muestra un mensaje y termina con `return 1` desde `main`. Las funciones de búsqueda usan `return -1` con otro propósito: informar que un valor leído correctamente no está en la colección.

Compila el programa con:

```bash
g++ clip_duration_search.cpp -o clip_duration_search
```

Ejecuta y escribe `30` cuando se solicite:

```bash
./clip_duration_search
```

La siguiente ejecución muestra la entrada escrita por el usuario en la misma línea del prompt y los índices resultantes:

```text
Enter a duration in seconds: 30
Linear search index: 4
Iterative binary search index: 4
Recursive binary search index: 4
```

El programa imprime índices para poder compararlos; en un catálogo con registros completos, ese índice permitiría consultar el registro encontrado después de descartar `-1`.

### Traza de una coincidencia y de un valor ausente

Para buscar 30, la versión recursiva hace estas comparaciones. Los límites de la tabla son los de cada llamada antes de decidir:

| `first` | `last` | `middle` | Valor central | Decisión |
| ---: | ---: | ---: | ---: | --- |
| 0 | 7 | 3 | 27 | `27 < 30`: buscar en `4..7`. |
| 4 | 7 | 5 | 35 | `35 > 30`: buscar en `4..4`. |
| 4 | 4 | 4 | 30 | Coincide: devolver 4. |

Las llamadas anteriores devuelven ese mismo índice al regresar. No suman ni combinan resultados: en cada nivel solo se necesitaba explorar una mitad. Linear search, en cambio, consulta las posiciones `0`, `1`, `2`, `3` y `4` antes de encontrar 30.

Para buscar 31, las dos primeras decisiones son las mismas, pero en `4..4` se compara 30 con 31. Como es menor, se solicita `5..4`. Allí `first > last` es verdadero y se devuelve `-1` sin calcular un nuevo centro ni acceder al array. El `-1` se propaga al regresar. En otra ejecución con entrada 31, las tres funciones deben imprimir `-1`.

### Tiempo, espacio y costo previo de ordenar

La tabla separa el peor caso de tiempo y el espacio auxiliar de las versiones presentadas. Los datos de entrada no cuentan como espacio auxiliar:

| Algoritmo | Requisito de orden | Peor caso de tiempo | Espacio auxiliar |
| --- | --- | --- | --- |
| Linear search iterativa | Ninguno. | `O(n)` | `O(1)` |
| Binary search iterativa | Orden por el campo buscado. | `O(log n)` | `O(1)` |
| Binary search recursiva | El mismo orden. | `O(log n)` | `O(log n)` por llamadas activas. |

Si el array no está ordenado y solo haremos una consulta, linear search evita ordenar toda la colección previamente. Si habrá muchas consultas sobre datos que se mantienen ordenados, puede justificarse ese trabajo inicial y aprovechar búsquedas logarítmicas. Para un ordenamiento `O(n log n)` seguido de `q` búsquedas binarias, el trabajo conjunto es `O(n log n + q log n)`; ordenar una vez no significa ordenar antes de cada consulta.

Nuestro catálogo se ordenó por duración, no por ID. No podemos aplicar binary search por `clipId` usando ese mismo orden. Para buscar un ID mediante binary search necesitaríamos otra vista ordenada por ID, o elegir una estructura adecuada para esa consulta.

En una linked list, acceder a un supuesto índice central requiere recorrer enlaces. Por tanto, no se traslada directamente la ventaja de acceso de un array a una lista. Esta limitación motiva preguntar si una estructura enlazada puede conservar ramas que permitan decidir hacia dónde buscar.

<details>
<summary>Verificación: ¿el caso base de merge sort y el de búsqueda binaria pueden tratar igual el rango <code>4..4</code>?</summary>

No. Para ordenar, un elemento ya está ordenado y puede regresarse sin comparar con otro. Para buscar, todavía hay que comprobar si ese único elemento coincide con el objetivo. Solo un rango vacío demuestra ausencia sin revisar otra posición.

</details>

## Parte 6 — Introducción a Binary Trees y BST (20 min)

La búsqueda binaria en un array ordenado descarta una mitad después de comparar un valor central. En una linked list solo tenemos una secuencia de enlaces y no podemos saltar al centro directamente. Introduciremos una estructura que conserva dos posibles ramas por node, definiremos su vocabulario y veremos qué regla adicional permite elegir una rama para buscar. La implementación completa de sus operaciones continuará en Semana 13.

### Qué es un binary tree

Un **binary tree**, o árbol binario, es una estructura enlazada en la que cada node puede tener **como máximo dos hijos**, identificados como izquierdo y derecho. No exige que todos tengan dos. Un árbol no vacío tiene un único node inicial; cada node restante tiene exactamente un padre, y los enlaces hacia hijos no forman ciclos.

En una singly linked list, cada node conserva el siguiente de una cadena. En un árbol binario, un node puede conservar dos direcciones, normalmente llamadas `left` y `right`, hacia el inicio de sus ramas. Un pointer sin hijo vale `nullptr`.

El vocabulario identifica posiciones y relaciones, no keywords de C++:

- **Root:** node inicial del árbol. El programa conserva su dirección en un pointer que aquí llamaremos `root`. En un árbol vacío ese pointer vale `nullptr`.
- **Parent:** padre de un node, el que tiene un enlace directo hacia él.
- **Child:** hijo, el node alcanzado por uno de esos enlaces directos.
- **Leaf:** hoja, un node sin hijos; ambos pointers valen `nullptr`.
- **Subtree:** subárbol, formado por un node y todos los que descienden de él.

### Contexto: organizar la consulta por ID del catálogo

El Centro de Medios también necesita encontrar fichas por ID. Para introducir la estructura, usaremos siete IDs numéricos dummy y únicos: 10, 20, 30, 40, 50, 60 y 70. Estos números representan **IDs**, no las duraciones del ejemplo anterior. En esta primera demostración cada node guardará solo el ID; un catálogo completo podría asociarlo con la ficha del clip.

El diagrama muestra un árbol cuya entrada es 40. Las ramas etiquetadas son enlaces `left` y `right`, no posiciones de un array. Se omiten las flechas a `nullptr` de las hojas:

```text
                         root
                           |
                          [40]
                    left /    \ right
                       [20]   [60]
                 left /  \   /  \ right
                    [10] [30] [50] [70]
```

40 es la root y padre de 20 y 60. El subárbol izquierdo de 40 incluye 20, 10 y 30. Los nodes 10, 30, 50 y 70 son hojas. El node 30 no es hijo directo de 40: su padre es 20.

### Qué distingue un Binary Search Tree

Un **Binary Search Tree (BST)** es un árbol binario que además respeta una regla de orden. Con IDs únicos, para cada node, **todos** los IDs de su subárbol izquierdo son menores que su ID y **todos** los de su subárbol derecho son mayores. La regla debe cumplirse en cada subárbol, no solamente al comparar la root con sus dos hijos inmediatos.

El árbol mostrado es un BST. Para buscar 30, comparamos con 40: como 30 es menor, seguimos `left` hasta 20. Como 30 es mayor que 20, seguimos `right` hasta 30. La ruta es `40 → 20 → 30`; no necesitamos visitar 60, 50 ni 70.

Un binary tree general no promete esa regla y no permite descartar ramas por el valor. Asimismo, el valor de la root de un BST no tiene que ser la mediana: la forma depende de cómo se construya. Veremos inserción y búsqueda programadas en la siguiente lecture.

### Representar los dos enlaces en C++

Declararemos un `struct TreeNode` para **un node**, con `clipId`, `left` y `right`. El programa completo siguiente, `clip_tree_structure.cpp`, crea los siete nodes como variables locales y conecta sus direcciones explícitamente para representar el diagrama. Sirve para observar la estructura; no implementa todavía inserción automática ni balanceo.

```cpp
#include <iostream>
using namespace std;

struct TreeNode
{
    int clipId;      // Store one unique catalog ID.
    TreeNode *left;  // Point to the left child or nullptr.
    TreeNode *right; // Point to the right child or nullptr.
};

int main()
{
    TreeNode node40 = {40, nullptr, nullptr};
    TreeNode node20 = {20, nullptr, nullptr};
    TreeNode node60 = {60, nullptr, nullptr};
    TreeNode node10 = {10, nullptr, nullptr};
    TreeNode node30 = {30, nullptr, nullptr};
    TreeNode node50 = {50, nullptr, nullptr};
    TreeNode node70 = {70, nullptr, nullptr};

    TreeNode *root = &node40; // Keep access to the starting node.
    node40.left = &node20;
    node40.right = &node60;
    node20.left = &node10;
    node20.right = &node30;
    node60.left = &node50;
    node60.right = &node70;

    cout << "Root ID: " << root->clipId << endl;
    cout << "Left child ID: " << root->left->clipId << endl;
    cout << "Right child ID: " << root->right->clipId << endl;
    cout << "ID reached by left, then right: "
         << root->left->right->clipId << endl;
    return 0; // Local nodes end their lifetime here; no delete is needed.
}
```

Las llaves inicializan los members en su orden declarado: ID, dirección izquierda y dirección derecha. `node40` es un objeto directo, por eso se usa `node40.left`; `root` es un pointer, por eso se usa `root->left`. La expresión `root->left->right->clipId` sigue primero la dirección izquierda, luego la derecha y finalmente consulta el ID. Los enlaces utilizados existen en esta demostración; en una búsqueda general habría que verificar `nullptr` antes de avanzar.

Compila el ejemplo con:

```bash
g++ clip_tree_structure.cpp -o clip_tree_structure
```

Para ejecutarlo:

```bash
./clip_tree_structure
```

El output verifica cuatro accesos concretos del diagrama:

```text
Root ID: 40
Left child ID: 20
Right child ID: 60
ID reached by left, then right: 30
```

Los nodes son variables locales que siguen vivos durante esos accesos. No se usó `new`, por lo que no corresponde ejecutar `delete`. En la Semana 13 una clase administrará nodes dinámicos y deberá definir también cómo libera su memoria.

### La forma del árbol también afecta el costo

La **altura** es la cantidad de enlaces de la ruta más larga desde la root hasta una hoja; usaremos esa convención al contar. El árbol del diagrama tiene altura 2: por ejemplo, de 40 a 20 y de 20 a 10.

Buscar siguiendo una rama depende de esa altura. Si el árbol mantiene ramas de tamaños comparables, su altura crece como `log n` y la búsqueda puede ser `O(log n)`. Si se forma una cadena, la altura crece como `n` y el peor caso es `O(n)`. Ser BST garantiza una regla de orden, no una forma equilibrada ni tiempo logarítmico en toda situación.

<details>
<summary>Verificación: si cambiamos el ID de la hoja 30 a 45, manteniendo los mismos enlaces, ¿seguimos teniendo un binary tree y un BST?</summary>

Sigue siendo un binary tree: no cambiaron sus relaciones y nadie tiene más de dos hijos. Ya no es un BST, porque 45 queda dentro del subárbol izquierdo de 40 y es mayor que 40. Compararlo solo con su padre 20 no basta para validar la regla global.

</details>

## Parte 7 — Verificar decisiones y cerrar (15 min)

Ya vimos dos formas de dividir para ordenar, dos estrategias de búsqueda y una estructura que permite conservar ramas enlazadas. Necesitamos comprobar que distinguimos sus condiciones antes de aplicarlas a registros del proyecto. Usaremos cinco minutos para estas decisiones y reservaremos los diez restantes para dudas y repetición de trazas.

### Práctica razonada

Las siguientes preguntas recuperan los escenarios de esta sesión. Justifica cada decisión por el requisito o el estado de los datos:

1. Esta versión elige inicialmente la posición central como pivot. ¿Eso garantiza que haya la misma cantidad de duraciones menores y mayores?
2. El Centro de Medios recibe una lista sin ordenar y solo necesita buscar un ID una vez. ¿Es obligatorio ordenar primero?
3. Un reporte está ordenado por duración, pero se quiere buscar por ID mediante binary search. ¿Se cumple su requisito?
4. Un árbol tiene como máximo dos hijos por node. ¿Eso basta para descartar una rama al comparar IDs?

<details>
<summary>Ver respuestas razonadas</summary>

1. No. El centro es una posición del segmento, no necesariamente su mediana por duración. En una entrada arbitraria podría contener el menor, el mayor o un valor repetido muchas veces; seleccionar el centro no elimina el peor caso `O(n²)`.
2. No. Linear search puede buscar directamente; ordenar agrega un costo que quizá no se justifique para una sola consulta.
3. No. Se requiere orden por ID; puede crearse otra vista ordenada por ese campo o elegirse una búsqueda que no lo requiera.
4. No. Esa condición define parte de la forma de un binary tree; descartar por valor requiere además la regla de un BST.

</details>

### Qué comprobar al ejecutar

Prueba quicksort con los ocho clips, un segmento lógico vacío `0..-1` usando capacidad positiva, un registro, entrada ordenada, orden inverso y duraciones iguales. Confirma que cada ID conserva su duración y que ningún registro desaparece. Con el programa de búsqueda, prueba 30, 31, 12, 50 y 18: distingue una coincidencia repetida de la ausencia y nunca uses `-1` como índice.

En el proyecto de cafetería se necesita preparar órdenes en FIFO y mostrar por separado un reporte temporal de delivery. Lab 11 propone construir una vista de pointers que permita consultar registros sin cambiar su colección original. Lab 12 continuará con sorting y searching sobre datos dummy dentro del repo del lab: elegir el algoritmo y el campo adecuado debe preservar la secuencia operacional y las reglas del reporte.

## Resumen de la sesión

- Quicksort selecciona un pivot, particiona y ordena sus dos lados. La partición no deja necesariamente ordenado el interior de esos lados.
- Los rangos recursivos excluyen el índice del pivot; el caso base evita procesar segmentos vacíos o de un elemento.
- El quicksort presentado toma `O(n log n)` con divisiones equilibradas y puede tomar `O(n²)` en el peor caso. Su espacio recursivo también depende de la forma de esas divisiones.
- Estabilidad significa conservar el orden original entre empates; esta versión de quicksort no la garantiza.
- Linear search no exige orden. Binary search requiere orden por el campo buscado y reduce el rango a una sola mitad por paso.
- Un binary tree tiene hasta dos hijos por node. Un BST añade una regla de orden sobre subárboles completos; su altura determina el costo de seguir una ruta de búsqueda.

## Próxima sesión

El Lab 12 practicará sorting y searching con una sección de aplicación al proyecto dentro del propio repositorio del lab. En la Semana 13 se desarrollarán las operaciones de un BST: inserción, búsqueda y recorridos, junto con una práctica breve de árboles y repaso para el Examen 2 de Semana 14.
