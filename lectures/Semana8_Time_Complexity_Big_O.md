# Semana 8 — Time Complexity: Big-O
## COEN 2220 — Programming 2

**Instructor:** Wilson Lozano

**Duración:** 170 min (sesión de 3 horas crédito nominales)  
**Precede a:** Lab 8 — Big-O / análisis de time complexity  
**Lectura complementaria:** Gaddis, *Starting Out with C++*, 8va ed., Cap. 8 — secciones 8.1 y 8.3 (Introduction to Search Algorithms; Introduction to Sorting Algorithms). El capítulo compara cantidades de comparaciones en búsquedas y muestra algoritmos de sorting; esta lecture usa esas ideas para introducir la notación Big-O.

---

## Objetivos

Al finalizar esta sesión, el estudiante podrá:

1. Explicar por qué medir segundos no basta para comparar la eficiencia de dos algoritmos.
2. Identificar el tamaño de entrada `n` y una operación representativa al analizar código sencillo.
3. Interpretar `O(1)`, `O(log n)`, `O(n)` y `O(n²)` como órdenes de crecimiento.
4. Comparar linear search y binary search, incluyendo el requisito de que los datos estén ordenados para usar binary search.
5. Reconocer en código la diferencia entre loops consecutivos, loops anidados y reducción repetida de un problema a la mitad.
6. Explicar por qué bubble sort y selection sort tienen crecimiento cuadrático, aun cuando no hacen exactamente el mismo número de intercambios.

## Parte 0 — Puente: del contrato ADT al costo de una operación (10 min)

En el Lab 7 definieron un ADT antes de implementar una clase. Un contrato como `StudySessionLog ADT` puede prometer operaciones tales como `addSession`, `totalMinutes`, `longestSession`, `size` e `isEmpty`. El contrato responde una pregunta importante: **¿qué puede pedir el código cliente?**

Hoy añadimos otra pregunta: **¿cuánto trabajo necesita cada operación cuando aumenta la cantidad de datos?**

Por ejemplo, `StudySessionLog::size()` puede devolver directamente el valor de `count`. En cambio, `totalMinutes()` debe revisar las duraciones registradas para sumarlas. Ambas operaciones son válidas dentro del mismo ADT, pero no tienen el mismo costo cuando el registro tiene muchas sesiones.

Para hablar con precisión, llamaremos **`n`** al tamaño de la entrada. En este contexto, `n` podría ser la cantidad de sesiones almacenadas. En otro programa, `n` puede significar cantidad de libros, estudiantes, órdenes o nodos. Antes de decir que algo es rápido o lento, siempre debemos preguntar: “¿qué representa `n` aquí?”

> **Idea clave:** el ADT describe las operaciones disponibles; el análisis de complejidad compara cómo crece el trabajo de esas operaciones cuando crece `n`.

## Parte 1 — Por qué no basta con medir segundos (20 min)

Una primera reacción podría ser ejecutar un programa, mirar cuántos segundos tarda y declarar ganador al que termine primero. El problema es que ese resultado depende de factores que no forman parte del algoritmo: el procesador, otras aplicaciones abiertas, memoria disponible y el compilador usado.

En cambio, analizaremos una acción que el algoritmo repite. Esa acción se llama aquí **operación representativa**. No tiene que ser la única instrucción del programa; es una operación cuyo número de repeticiones nos permite entender cómo crece el trabajo.

### Ejemplo: contar salones disponibles

La oficina de actividades tiene un array donde cada posición indica si un salón está disponible. Para saber cuántos salones se pueden usar, el programa debe revisar cada posición.

```cpp
int countAvailableRooms(const bool rooms[], int roomCount)
{
    int availableCount = 0;

    for (int index = 0; index < roomCount; index++)
    {
        if (rooms[index])
        {
            availableCount++; // Count each room marked as available.
        }
    }

    return availableCount;
}
```

El tamaño de entrada es `n = roomCount`. La operación representativa es revisar `rooms[index]`. El loop debe pasar por cada posición una vez:

| Cantidad de salones (`n`) | Revisiones de `rooms[index]` |
| ---: | ---: |
| 4 | 4 |
| 100 | 100 |
| 10,000 | 10,000 |

No importa si algunos salones están disponibles y otros no: para obtener un conteo correcto, el algoritmo debe revisar todos. Decimos que el trabajo crece **linealmente** con `n`.

No buscaremos una fórmula exacta para cada línea del programa. Por ejemplo, también se evalúa la condición del `if`, se incrementa `index` y se compara `index < roomCount`. Para una primera clasificación, lo importante es que todas esas acciones ocurren un número de veces proporcional a `n`.

### Una operación que no crece con `n`

Ahora supongan que el programa ya conoce un índice válido y necesita consultar el salón almacenado allí:

```cpp
bool roomIsAvailable(const bool rooms[], int roomIndex)
{
    return rooms[roomIndex]; // Read one known array position directly.
}
```

Aunque el array tenga 4, 100 o 10,000 elementos, esta función consulta una sola posición. El tamaño total del array no cambia el número de accesos que hace la función. En la próxima parte asignaremos una notación a esta diferencia entre trabajo que permanece constante y trabajo que crece con la entrada.

<details>
<summary>Pregunta de verificación: si <code>roomCount</code> se duplica, ¿qué ocurre con las revisiones de <code>countAvailableRooms</code>?</summary>

También se duplican aproximadamente. Si pasa de 100 a 200 salones, la función pasa de revisar 100 a revisar 200 posiciones. Eso caracteriza el crecimiento lineal.

</details>

## Parte 2 — Big-O: describir cómo crece el trabajo (25 min)

En la Parte 1 vimos dos comportamientos sin ponerles nombre: una función consultaba una posición conocida sin importar el tamaño total del array; otra debía revisar cada posición para contar los salones disponibles. **Big-O notation** nos dará un nombre para esas formas de crecimiento.

Big-O describe cómo crece el trabajo de un algoritmo cuando `n` se hace grande. No promete segundos exactos ni intenta contar cada instrucción de forma aislada. Su pregunta es: si el tamaño de entrada se duplica, ¿el trabajo permanece parecido, se duplica, se cuadruplica o crece de otra forma?

### Primer caso: trabajo que permanece constante — `O(1)`

La letra `O` se lee “orden de”. La expresión `O(1)` significa que el número de operaciones relevantes no depende de `n`. El `1` no representa exactamente una instrucción ni un segundo: representa una cantidad constante.

Por ejemplo, esta función siempre consulta la primera posición válida. Puede requerir algunas instrucciones internas, pero esa cantidad no aumenta cuando el array pasa de 4 a 10,000 valores.

```cpp
int firstSessionMinutes(const int sessionMinutes[], int sessionCount)
{
    // Precondition: sessionCount is greater than 0.
    return sessionMinutes[0]; // Read one fixed position.
}
```

La función no recorre `sessionMinutes`. Por eso la clasificamos como `O(1)`: su trabajo permanece constante respecto a la cantidad de sesiones.

> **Importante:** `O(1)` no significa “instantáneo” ni “sin costo”. Solo significa que agregar más datos al array no hace que esta operación tenga que revisar más posiciones.

### Segundo caso: trabajo proporcional a la entrada — `O(n)`

La función `countAvailableRooms` de la Parte 1 revisa cada salón. Si `n` representa la cantidad de salones, hace aproximadamente `n` revisiones. Ese patrón se llama `O(n)`, leído “orden de n” o crecimiento lineal.

| `n` salones | Revisiones necesarias | Qué ocurre si `n` se duplica |
| ---: | ---: | --- |
| 10 | 10 | De 10 a 20 revisiones. |
| 100 | 100 | De 100 a 200 revisiones. |
| 10,000 | 10,000 | De 10,000 a 20,000 revisiones. |

No significa que todas las instrucciones del programa sean idénticas; significa que el trabajo dominante crece en la misma proporción que `n`.

### Loops consecutivos no se multiplican

Consideren una función que primero imprime los códigos de los talleres y después cuenta los talleres con cupo. Tiene dos loops, pero uno termina antes de que empiece el otro.

```cpp
void reviewWorkshops(const string workshopCodes[], const bool hasSpace[], int workshopCount)
{
    for (int index = 0; index < workshopCount; index++)
    {
        cout << workshopCodes[index] << endl; // First pass through the data.
    }

    int openCount = 0;
    for (int index = 0; index < workshopCount; index++)
    {
        if (hasSpace[index])
        {
            openCount++;
        }
    }

    cout << "Workshops with space: " << openCount << endl;
}
```

El primer loop realiza aproximadamente `n` iteraciones y el segundo otras `n`. Como uno termina antes de que comience el otro, el trabajo se **suma**: `n + n = 2n`.

¿Por qué escribimos `O(n)` y no `O(2n)`? Big-O agrupa algoritmos por su **ritmo de crecimiento**, no por su costo exacto. Veamos qué sucede al duplicar la entrada:

| `n` talleres | Iteraciones de un loop | Iteraciones de dos loops consecutivos |
| ---: | ---: | ---: |
| 10 | 10 | 20 |
| 20 | 20 | 40 |
| 100 | 100 | 200 |

El segundo programa realiza aproximadamente el doble de trabajo que el primero para el mismo `n`; esa diferencia sí puede importar en una medición real. Sin embargo, ambos duplican su trabajo cuando `n` se duplica. Ambos crecen linealmente, por eso pertenecen a la misma categoría `O(n)`.

La misma idea permite resumir expresiones como `3n + 4`. El `+ 4` no crece cuando aumenta `n`, y el factor 3 no cambia el hecho de que al duplicar `n` el trabajo principal aproximadamente se duplica. En este curso usaremos Big-O para reconocer esa forma de crecimiento; no para negar que los detalles constantes existen.

<details>
<summary>Práctica breve: clasifica cada caso</summary>

1. Leer `values[7]` cuando el índice es válido y conocido.
2. Un loop que imprime los `n` nombres de estudiantes.
3. Dos loops consecutivos, cada uno de `n` iteraciones.

<details>
<summary>Ver respuesta</summary>

1. `O(1)`: se lee una posición conocida.
2. `O(n)`: se visita cada nombre una vez.
3. `O(n)`: son aproximadamente `2n` iteraciones, y las constantes se omiten al resumir el crecimiento.

</details>

</details>

## Parte 3 — Caso central: linear search y binary search (25 min)

El Capítulo 8 usa búsquedas para comparar eficiencia. Una **search algorithm** busca una meta dentro de una colección. El resultado puede ser una posición o una indicación de que el valor no aparece.

Supongan que la oficina necesita encontrar el número de un salón dentro de una lista. En una **linear search**, el programa comienza en la primera posición y revisa una por una hasta encontrar el valor o terminar la lista.

```cpp
int linearSearch(const int roomNumbers[], int roomCount, int targetRoom)
{
    for (int index = 0; index < roomCount; index++)
    {
        if (roomNumbers[index] == targetRoom)
        {
            return index; // Stop when the requested room is found.
        }
    }

    return -1;            // Report that no matching room was found.
}
```

Si `targetRoom` está al final o no aparece, la función puede hacer `n` comparaciones. Si aparece al inicio, puede encontrarlo con una sola comparación. Para describir el caso que puede requerir más trabajo, usaremos `O(n)`.

### Binary search reduce el problema a la mitad

Binary search tiene una ventaja distinta: compara el valor buscado con el elemento central. Si el valor buscado es mayor, puede ignorar toda la mitad menor; si es menor, puede ignorar toda la mitad mayor. En cada paso queda aproximadamente la mitad del trabajo pendiente.

Pero esa decisión solo tiene sentido si los datos están **ordenados**. Si los números de salón están mezclados, saber que el valor central es menor que el objetivo no permite concluir que el objetivo está a la derecha.

```cpp
int binarySearch(const int roomNumbers[], int roomCount, int targetRoom)
{
    int first = 0;
    int last = roomCount - 1;

    while (first <= last)
    {
        int middle = (first + last) / 2;

        if (roomNumbers[middle] == targetRoom)
        {
            return middle; // The middle value matches the target.
        }
        else if (roomNumbers[middle] < targetRoom)
        {
            first = middle + 1; // Discard the lower half.
        }
        else
        {
            last = middle - 1;  // Discard the upper half.
        }
    }

    return -1;                 // The target does not appear in the sorted array.
}
```

### Traza concreta

Usen este array, que ya está ordenado:

```cpp
int roomNumbers[] = {101, 104, 108, 115, 130, 136, 140, 155};
```

Busquemos `140`.

| Intento | Posición central | Valor central | Decisión |
| ---: | ---: | ---: | --- |
| 1 | 3 | 115 | `140` es mayor; se descartan las posiciones 0–3. |
| 2 | 5 | 136 | `140` es mayor; se descartan las posiciones 4–5. |
| 3 | 6 | 140 | Se encuentra el valor. |

La linear search habría revisado siete valores antes de llegar a `140`; la binary search necesitó tres comparaciones. La diferencia se vuelve más importante cuando aumenta `n`.

### Nombrar el crecimiento que reduce a la mitad: `O(log n)`

En esta clase, no necesitan calcular logaritmos con una calculadora. Lean `log n` como esta pregunta: **¿cuántas veces puedo dividir aproximadamente a la mitad hasta llegar a un elemento?**

| Elementos iniciales | Mitades necesarias para llegar a 1 | Comparaciones máximas aproximadas de binary search |
| ---: | ---: | ---: |
| 8 | 8 → 4 → 2 → 1 | 3 |
| 16 | 16 → 8 → 4 → 2 → 1 | 4 |
| 1,024 | 1,024 → 512 → ... → 1 | 10 |

Cuando se duplica la cantidad de elementos, binary search no duplica sus pasos: normalmente añade solo una comparación más. El libro usa potencias de dos para expresar esa idea. Para un array de 1,000 elementos, binary search necesita como máximo alrededor de 10 comparaciones; para 50,000 elementos, alrededor de 16. Esa reducción repetida se resume como `O(log n)`.

> ⚠️ **No concluyan que siempre se debe ordenar primero.** Si el programa realizará una sola búsqueda en datos desordenados, ordenar puede agregar trabajo innecesario. Binary search se vuelve especialmente útil cuando los datos ya están ordenados o cuando habrá muchas búsquedas posteriores sobre la misma colección.

<details>
<summary>Pregunta de verificación: ¿por qué binary search no funciona correctamente con <code>{140, 101, 155, 104}</code>?</summary>

El array no está ordenado. Si el algoritmo compara el valor central con el objetivo, no puede decidir de manera confiable qué mitad descartar: valores menores y mayores pueden aparecer en cualquiera de las dos mitades.

</details>

## Parte 4 — Reconocer crecimiento al leer código (25 min)

Ahora conectemos la notación con patrones visibles en C++. La meta no es memorizar una lista de respuestas. Primero identifiquen `n`, luego observen cuántas veces se repite la operación importante en relación con `n`.

### Un loop: `O(n)`

```cpp
int totalMinutes(const int sessionMinutes[], int sessionCount)
{
    int total = 0;

    for (int index = 0; index < sessionCount; index++)
    {
        total += sessionMinutes[index]; // One addition for each stored session.
    }

    return total;
}
```

Si `n` es `sessionCount`, el `for` ejecuta una suma por cada sesión: `O(n)`.

### Loops anidados: `O(n²)`

La situación cambia cuando un loop completo ocurre dentro de cada iteración de otro loop. Este ejemplo compara cada par de sesiones para contar cuántas tienen la misma duración.

```cpp
int countMatchingPairs(const int sessionMinutes[], int sessionCount)
{
    int matchingPairs = 0;

    for (int first = 0; first < sessionCount; first++)
    {
        for (int second = first + 1; second < sessionCount; second++)
        {
            if (sessionMinutes[first] == sessionMinutes[second])
            {
                matchingPairs++; // Compare one pair of distinct sessions.
            }
        }
    }

    return matchingPairs;
}
```

No hace exactamente `n × n` comparaciones porque el segundo loop comienza después de `first`. Para cuatro sesiones, compara los pares posibles: 0 con 1, 2 y 3; luego 1 con 2 y 3; finalmente 2 con 3. Son seis comparaciones.

| Sesiones (`n`) | Pares distintos que se comparan | Qué ocurre al duplicar aproximadamente `n` |
| ---: | ---: | --- |
| 4 | 6 | — |
| 8 | 28 | Hay mucho más que el doble de comparaciones. |
| 16 | 120 | El trabajo se acerca a cuadruplicarse. |

Para cada valor de `first`, el segundo loop todavía puede revisar muchos otros valores. Por eso, al duplicar la colección, la cantidad de pares crece aproximadamente cuatro veces, no dos. Llamamos a ese patrón **crecimiento cuadrático** y lo resumimos como `O(n²)`.

### Reducir repetidamente: `O(log n)`

El patrón de binary search se puede reconocer sin hablar todavía de arrays. Cada iteración reduce el valor pendiente aproximadamente a la mitad.

```cpp
int countHalvingSteps(int itemCount)
{
    int steps = 0;

    while (itemCount > 1)
    {
        itemCount = itemCount / 2; // Keep only about half of the remaining items.
        steps++;
    }

    return steps;
}
```

Si comenzamos con 32 elementos, la secuencia es 32, 16, 8, 4, 2, 1: cinco pasos. Si comenzamos con 64, se añade solo un paso más. Ese crecimiento lento es la intuición de `O(log n)`.

<details>
<summary>Ejercicio en clase: clasifica cada función y justifica con una frase</summary>

```cpp
bool isFirstValuePositive(const int values[])
{
    return values[0] > 0;
}
```

```cpp
void printPairs(const int values[], int size)
{
    for (int left = 0; left < size; left++)
    {
        for (int right = 0; right < size; right++)
        {
            cout << values[left] << "," << values[right] << endl;
        }
    }
}
```

<details>
<summary>Ver respuesta</summary>

`isFirstValuePositive` es `O(1)` porque consulta una posición fija. `printPairs` es `O(n²)` porque para cada valor de `left`, el loop interno recorre todos los valores de `right`.

</details>

</details>

## Parte 5 — Sorting como ejemplo de `O(n²)` (20 min)

Los algoritmos de sorting organizan datos en un orden, por ejemplo de menor a mayor. El Capítulo 8 presenta bubble sort y selection sort. En esta sesión no los implementarán como una tarea nueva: los usaremos para observar por qué algunos algoritmos hacen muchas comparaciones al crecer la colección.

### Nota antes de usar bubble sort

**Bubble sort** recorre una colección comparando elementos vecinos. Si dos vecinos están fuera de orden, los intercambia. Debe repetir pasadas hasta que los valores queden ordenados; los valores grandes pueden avanzar gradualmente hacia el final, una posición a la vez.

Una versión resumida se ve así:

```cpp
void bubbleSort(int values[], int size)
{
    bool swapped;

    do
    {
        swapped = false;

        for (int index = 0; index < size - 1; index++)
        {
            if (values[index] > values[index + 1])
            {
                int temporary = values[index];
                values[index] = values[index + 1];
                values[index + 1] = temporary;
                swapped = true; // Another pass is needed after an exchange.
            }
        }
    } while (swapped);
}
```

En un caso desfavorable, el algoritmo realiza muchas pasadas y en cada una compara muchos vecinos. Por ejemplo, si los valores están en orden inverso, una primera pasada compara casi todos los vecinos, luego otra pasada vuelve a comparar casi todos, y así sucesivamente. Esa combinación de trabajo repetido se aproxima a `n²`, así que clasificamos bubble sort como `O(n²)` en el peor caso.

### Nota antes de usar selection sort

**Selection sort** trabaja de otra manera: para cada posición, busca el valor menor entre los elementos que todavía no están en su lugar y lo intercambia con la posición actual. Puede hacer menos intercambios que bubble sort, porque mueve un valor directamente a su posición final.

```cpp
void selectionSort(int values[], int size)
{
    for (int start = 0; start < size - 1; start++)
    {
        int smallestIndex = start;

        for (int index = start + 1; index < size; index++)
        {
            if (values[index] < values[smallestIndex])
            {
                smallestIndex = index; // Remember the smallest remaining value.
            }
        }

        int temporary = values[start];
        values[start] = values[smallestIndex];
        values[smallestIndex] = temporary;
    }
}
```

La diferencia importante es esta: selection sort puede hacer menos **intercambios**, pero para encontrar el menor valor todavía debe recorrer repetidamente la parte no ordenada. Con cuatro valores, busca entre tres valores restantes para la primera posición, luego entre dos para la segunda y después entre uno para la tercera: `3 + 2 + 1 = 6` comparaciones. Con ocho valores, el mismo patrón es `7 + 6 + ... + 1 = 28` comparaciones.

Al duplicar aproximadamente la cantidad de valores, ese total crece mucho más que el doble. Las comparaciones forman el mismo patrón de pares que vimos en la Parte 4, por lo que el crecimiento de selection sort también es `O(n²)`.

| Algoritmo | Idea principal | Razón del crecimiento `O(n²)` |
| --- | --- | --- |
| Bubble sort | Compara vecinos e intercambia los que están fuera de orden. | Puede hacer muchas pasadas; cada pasada compara muchos vecinos. |
| Selection sort | Busca el menor valor restante y lo coloca en la siguiente posición. | Para cada posición, busca entre muchos elementos restantes. |

> **Conexión con el proyecto final:** más adelante podrán implementar selection sort o insertion sort donde el proyecto lo requiera. Por ahora, la meta es reconocer que ordenar un conjunto grande tiene un costo y justificar por qué no se ordena sin necesidad.

## Parte 6 — Aplicar Big-O al diseño de un ADT (20 min)

Regresemos al `StudySessionLog` del Lab 7. Allí la implementación usa un array de capacidad fija de cuatro posiciones y un contador. Estrictamente, un límite fijo de cuatro no puede crecer indefinidamente. Para aprender Big-O, imaginen la misma clase y el mismo contrato con capacidad suficiente para almacenar `n` sesiones. Así podemos preguntar qué ocurriría cuando aumenta la cantidad de sesiones; no estamos cambiando la interfaz del ADT, solo estudiando cómo escalaría la implementación.

Con esa idea, podemos analizar algunas operaciones sin cambiar el contrato:

| Operación del ADT | Decisión de implementación actual | Orden de crecimiento |
| --- | --- | --- |
| `size()` | Devuelve el member variable `count`. | `O(1)` |
| `isEmpty()` | Compara `count` con cero. | `O(1)` |
| `addSession(minutes)` | Escribe en la siguiente posición libre y aumenta `count`. | `O(1)` |
| `totalMinutes()` | Recorre las sesiones almacenadas para sumarlas. | `O(n)` |
| `longestSession()` | Recorre las sesiones almacenadas para comparar duraciones. | `O(n)` |

Esto no significa que un ADT tenga una complejidad única. Cada operación puede tener un costo diferente. También importa la implementación elegida. El contrato puede seguir prometiendo `addSession`, `size` y `totalMinutes`, aunque en una semana posterior una linked list reemplace el array interno.

La próxima estructura de datos, una linked list, hará visible ese intercambio. Algunas operaciones que son directas en un array, como acceder a una posición por índice, pueden requerir recorrer nodos en una linked list. A cambio, otras operaciones de inserción pueden ser más convenientes según dónde ocurra la inserción. No necesitan memorizar esos resultados todavía; la idea es anticipar que **la estructura elegida afecta el costo de las operaciones del mismo ADT**.

<details>
<summary>Pregunta de diseño: para un registro que calcula totales muchas veces, ¿qué operación merece atención especial?</summary>

`totalMinutes()` merece atención porque actualmente recorre todos los valores cada vez que se llama. Si el programa necesita el total con mucha frecuencia, el diseñador puede evaluar si conviene guardar información adicional o usar otra estrategia. La respuesta concreta depende de qué operaciones necesita más el programa.

</details>

## Parte 7 — Práctica de cierre y resumen (25 min)

### Práctica de clasificación

Para cada escenario, identifiquen `n`, la operación representativa y la clasificación más adecuada entre `O(1)`, `O(log n)`, `O(n)` y `O(n²)`.

1. Un programa recibe un índice válido y muestra el estudiante almacenado en esa posición de un array.
2. Un programa busca un código de libro revisando cada código desde el primero hasta encontrarlo.
3. Un programa busca un ID en un array ordenado y descarta la mitad que no puede contenerlo en cada intento.
4. Un programa compara cada taller con todos los demás para detectar títulos duplicados.

<details>
<summary>Ver respuesta</summary>

1. `O(1)`: se consulta una posición conocida.
2. `O(n)`: en el caso que requiere más trabajo, se revisan todos los códigos.
3. `O(log n)`: cada intento descarta aproximadamente la mitad de los elementos, siempre que el array esté ordenado.
4. `O(n²)`: hay comparaciones repetidas entre muchos pares de talleres.

</details>

### Resumen de la sesión

- Big-O describe cómo crece el trabajo de un algoritmo cuando aumenta el tamaño de entrada `n`; no es una medición exacta de segundos.
- Un acceso directo conocido puede ser `O(1)`; recorrer todos los datos una vez suele ser `O(n)`.
- Binary search es `O(log n)` porque reduce el espacio de búsqueda a la mitad, pero exige datos ordenados.
- Loops consecutivos normalmente suman trabajo y siguen siendo `O(n)`; loops anidados suelen producir crecimiento `O(n²)`.
- Bubble sort y selection sort tienen `O(n²)` por sus comparaciones repetidas, aunque organicen e intercambien valores de manera diferente.
- El costo pertenece a una **operación de un ADT**, no al ADT completo de una sola vez; la implementación influye en ese costo.

Usen aproximadamente los últimos 5 minutos para preguntas, corregir clasificaciones ambiguas y conectar los ejemplos con el Lab 8.

## Próxima sesión

En el **Lab 8** analizarán fragmentos cortos de C++, identificarán `n` y justificarán órdenes de crecimiento con ejemplos de búsqueda, loops y sorting. Después, las **linked lists** mostrarán una nueva implementación posible para un ADT y permitirán comparar sus costos con los de los arrays.
