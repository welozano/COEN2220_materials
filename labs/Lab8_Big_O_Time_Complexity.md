# Lab 8 — Big-O / Time Complexity
## COEN 2220 — Programming 2

**Instructor:** Wilson Lozano

**Duración:** 110 min (sesión de 2 horas crédito nominales)  
**Precede a:** Semana 9 — Linked Lists  
**Requisitos:** Lecture de Semana 8; Labs 1–7 completados  
**Basado en:** Gaddis, *Starting Out with C++*, 8va ed., Cap. 8 — secciones 8.1 y 8.3 (searching and sorting). Exceptions y STL no forman parte de este lab.

---

## Objetivos

Al finalizar este laboratorio, el estudiante podrá:

1. Identificar qué representa `n` y cuál es la operación representativa en un fragmento de código.
2. Justificar las clasificaciones `O(1)`, `O(log n)`, `O(n)` y `O(n²)` con el patrón de crecimiento observado.
3. Comparar la cantidad de comparaciones de linear search y binary search para una misma búsqueda.
4. Explicar por qué binary search requiere datos ordenados y cuándo ordenar puede o no justificar su costo.
5. Implementar y probar patrones simples de reducción a la mitad y comparación de pares.

## Parte A — Preparar la carpeta y el repositorio (5 min)

> ⚠️ **Este paso es el que más se presta a error — léelo con cuidado antes de hacer clic en nada.** Cada lab tiene su propio repositorio. No inicialices Git en la carpeta padre `COEN2220` ni dentro de la carpeta de otro lab.

1. **A1.** Abre VS Code o VSCodium y selecciona `File → Open Folder...`.
2. **A2.** Navega a tu carpeta padre del curso, por ejemplo `COEN2220`. Si todavía no existe, créala **desde este diálogo** en un lugar de fácil acceso.
3. **A3.** Entra dentro de la carpeta padre. Sin salir del diálogo, crea una carpeta nueva llamada `lab8-big-o`, hermana de las carpetas de los labs anteriores.
4. **A4.** Abre específicamente `lab8-big-o` con **Open** / **Select Folder**.

**Cómo saber si abriste la carpeta correcta:** el Explorer debe mostrar `LAB8-BIG-O` como raíz. Si muestra `COEN2220` o el nombre de otro lab, usa `File → Close Folder` y repite los pasos.

> **¿Por qué importa tanto esto?** Inicializar el repositorio en la carpeta padre mezcla todos los labs en un solo historial. Inicializarlo dentro de un lab anterior mezcla archivos y entregas. Un repositorio separado permite evaluar esta práctica como una unidad de trabajo independiente.

Abre Source Control (`Ctrl+Shift+G` / `Cmd+Shift+G`) y selecciona **Initialize Repository**. Crea `.gitignore` en la raíz con este contenido:

```gitignore
# Compiled executables (Windows)
*.exe

# Compiled executables (Mac/Linux)
appointment_analysis_example
campus_activity_analysis

# Intermediate object files
*.o

# Editor configuration
.vscode/
```

Haz el commit inicial y publica el repositorio con el flujo ya practicado: stage → commit → **Publish Branch**. Usa el mensaje `"Initial setup and gitignore"`.

💾 **Por qué este commit importa:** guarda una base limpia antes de analizar o escribir código. Si aparece un problema después, el historial muestra que la carpeta, el repositorio y `.gitignore` ya estaban correctos.

## Parte B — Ejemplo guiado: medir trabajo, no segundos (35 min)

### Contexto

El Centro de Servicios Estudiantiles mantiene IDs numéricos de citas. El personal necesita consultar una cita por posición, buscar un ID específico y estimar cuánto trabajo requiere cada estrategia cuando la lista de citas crece.

En este lab no usarán un cronómetro ni compararán milisegundos. En su lugar, las funciones contarán una operación representativa, como comparar un ID contra el valor buscado. Ese contador no representa el tiempo exacto de una computadora; sirve como evidencia concreta de cuántas veces el algoritmo tuvo que tomar una decisión.

Crea un archivo llamado `appointment_analysis_example.cpp`. En vez de construirlo por fragmentos, usa desde el inicio el **starter code completo** de esta parte. Las funciones ya están declaradas antes de `main` y definidas después de `main`; no tienes que implementarlas ni moverlas. Así podrás concentrarte en observar y explicar el trabajo que hace cada patrón.

El programa prepara tres listas ordenadas de IDs: una de 50 citas, otra de 100 y otra de 200. Para cada tamaño, mide el mismo tipo de trabajo: un acceso directo, el promedio de comparaciones de linear search, el promedio de comparaciones de binary search y la cantidad de pares distintos. Los bloques `B1`, `B2`, `B3` y `B4` comienzan comentados dentro de `main`. Removerás los delimitadores de comentario de un bloque a la vez, lo ejecutarás y observarás el resultado antes de continuar con el siguiente.

### Starter code — Ejecutar las cuatro observaciones

El sistema ya conoce que necesita una cita guardada en una posición válida, puede buscar un ID y, en otra tarea, puede comparar pares de citas. El código siguiente representa esas situaciones para los tres tamaños. `fillSortedAppointmentIds` prepara datos ordenados y controlados; no es la parte que deben analizar. La atención del ejemplo está en las cuatro secciones marcadas dentro de `main`.

No se escriben literalmente 50, 100 y 200 IDs en el código. La función de preparación genera siempre la misma secuencia ascendente: por eso los primeros 50 IDs de la lista de 100 son exactamente los de la lista de 50, y los primeros 100 de la lista de 200 coinciden con los de la lista de 100. Al crecer la lista, cambia `n`, no los datos anteriores.

Para evitar que una sola búsqueda produzca una conclusión accidental, B2 contiene directamente diez IDs objetivo válidos por cada tamaño. Están repartidos entre el inicio, el medio y el final de cada lista. Son valores controlados, no valores generados con `rand()`: todos los estudiantes ejecutarán las mismas búsquedas y podrán comparar resultados. B2 usa esos diez objetivos en un loop de linear search; B3 reutiliza **los mismos** objetivos en un loop separado de binary search. Así, la diferencia observada se debe al algoritmo y no a objetivos distintos.

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 8 - Big-O / Time Complexity
 * Description: Guided example - compare algorithm work patterns
 * Due date: [Date]
 */

#include <iostream>
using namespace std;

// Function prototypes let main use these functions before their definitions.
int getAppointmentAt(const int appointmentIds[], int appointmentIndex);
int countLinearComparisons(const int appointmentIds[], int appointmentCount, int targetId);
int countBinaryComparisons(const int appointmentIds[], int appointmentCount, int targetId);
int countPossiblePairs(int appointmentCount);
void fillSortedAppointmentIds(int appointmentIds[], int appointmentCount);

int main()
{
    const int SMALL_COUNT = 50;
    const int MEDIUM_COUNT = 100;
    const int LARGE_COUNT = 200;

    int appointments50[SMALL_COUNT];
    int appointments100[MEDIUM_COUNT];
    int appointments200[LARGE_COUNT];

    // The helper creates ascending IDs so binary search is valid.
    fillSortedAppointmentIds(appointments50, SMALL_COUNT);
    fillSortedAppointmentIds(appointments100, MEDIUM_COUNT);
    fillSortedAppointmentIds(appointments200, LARGE_COUNT);

    /* --- B1: Direct access to one known position ---
    cout << "Appointment ID at a known index for 50 appointments: "
         << getAppointmentAt(appointments50, 25)
         << " (Direct accesses: 1)" << endl;
    cout << "Appointment ID at a known index for 100 appointments: "
         << getAppointmentAt(appointments100, 50)
         << " (Direct accesses: 1)" << endl;
    cout << "Appointment ID at a known index for 200 appointments: "
         << getAppointmentAt(appointments200, 100)
         << " (Direct accesses: 1)" << endl;
    */

    /* --- B2: Average linear-search comparisons for ten controlled targets ---
    const int SEARCH_COUNT = 10;

    // Every value below appears in its corresponding ordered appointment list.
    // The values are distributed from the beginning to the end of each list.
    const int searchTargets50[SEARCH_COUNT] =
        {1000, 1008, 1018, 1028, 1038, 1048, 1058, 1068, 1078, 1098};
    const int searchTargets100[SEARCH_COUNT] =
        {1000, 1018, 1038, 1058, 1078, 1098, 1118, 1138, 1158, 1198};
    const int searchTargets200[SEARCH_COUNT] =
        {1000, 1038, 1078, 1118, 1158, 1198, 1238, 1278, 1318, 1398};

    int linearTotal50 = 0;
    int linearTotal100 = 0;
    int linearTotal200 = 0;

    // Use every target once and add its linear-search comparison count.
    for (int query = 0; query < SEARCH_COUNT; query++)
    {
        linearTotal50 += countLinearComparisons(appointments50, SMALL_COUNT,
                                                 searchTargets50[query]);
        linearTotal100 += countLinearComparisons(appointments100, MEDIUM_COUNT,
                                                  searchTargets100[query]);
        linearTotal200 += countLinearComparisons(appointments200, LARGE_COUNT,
                                                  searchTargets200[query]);
    }

    cout << "Average linear comparisons for 50 appointments: "
         << static_cast<double>(linearTotal50) / SEARCH_COUNT << endl;
    cout << "Average linear comparisons for 100 appointments: "
         << static_cast<double>(linearTotal100) / SEARCH_COUNT << endl;
    cout << "Average linear comparisons for 200 appointments: "
         << static_cast<double>(linearTotal200) / SEARCH_COUNT << endl;
    */

    /* --- B3: Average binary-search comparisons for the same ten targets ---
    // Keep B2 active: it declares the target ID lists used below.
    int binaryTotal50 = 0;
    int binaryTotal100 = 0;
    int binaryTotal200 = 0;

    // Reuse the B2 target lists; only the search algorithm changes here.
    for (int query = 0; query < SEARCH_COUNT; query++)
    {
        binaryTotal50 += countBinaryComparisons(appointments50, SMALL_COUNT,
                                                 searchTargets50[query]);
        binaryTotal100 += countBinaryComparisons(appointments100, MEDIUM_COUNT,
                                                  searchTargets100[query]);
        binaryTotal200 += countBinaryComparisons(appointments200, LARGE_COUNT,
                                                  searchTargets200[query]);
    }

    cout << "Average binary comparisons for 50 appointments: "
         << static_cast<double>(binaryTotal50) / SEARCH_COUNT << endl;
    cout << "Average binary comparisons for 100 appointments: "
         << static_cast<double>(binaryTotal100) / SEARCH_COUNT << endl;
    cout << "Average binary comparisons for 200 appointments: "
         << static_cast<double>(binaryTotal200) / SEARCH_COUNT << endl;
    */

    /* --- B4: Distinct appointment pairs that two nested loops would visit ---
    cout << "Possible pairs for 50 appointments: "
         << countPossiblePairs(SMALL_COUNT) << endl;
    cout << "Possible pairs for 100 appointments: "
         << countPossiblePairs(MEDIUM_COUNT) << endl;
    cout << "Possible pairs for 200 appointments: "
         << countPossiblePairs(LARGE_COUNT) << endl;
    */

    return 0;
}

/* Returns the ID stored at a valid position already known by the caller. */
int getAppointmentAt(const int appointmentIds[], int appointmentIndex)
{
    return appointmentIds[appointmentIndex]; // Read one known array position.
}

/* Returns how many IDs linear search compares before finding targetId. */
int countLinearComparisons(const int appointmentIds[], int appointmentCount, int targetId)
{
    int comparisons = 0;

    for (int index = 0; index < appointmentCount; index++)
    {
        comparisons++; // One ID is compared with the target.

        if (appointmentIds[index] == targetId)
        {
            break;     // Stop after the matching ID is found.
        }
    }

    return comparisons;
}

/*
 * Returns how many middle IDs binary search compares before finding targetId.
 * Precondition: appointmentIds is sorted in ascending order and contains targetId.
 */
int countBinaryComparisons(const int appointmentIds[], int appointmentCount, int targetId)
{
    int first = 0;
    int last = appointmentCount - 1;
    int comparisons = 0;

    while (first <= last)
    {
        int middle = (first + last) / 2;
        comparisons++; // Compare the middle ID with the target.

        if (appointmentIds[middle] == targetId)
        {
            break;
        }
        else if (appointmentIds[middle] < targetId)
        {
            first = middle + 1; // Discard the lower half.
        }
        else
        {
            last = middle - 1;  // Discard the upper half.
        }
    }

    return comparisons;
}

/* Returns the number of distinct appointment pairs visited by two nested loops. */
int countPossiblePairs(int appointmentCount)
{
    int pairCount = 0;

    for (int first = 0; first < appointmentCount; first++)
    {
        for (int second = first + 1; second < appointmentCount; second++)
        {
            pairCount++; // One distinct pair would be compared here.
        }
    }

    return pairCount;
}

/* Fills the array with ascending dummy IDs so binary search is valid. */
void fillSortedAppointmentIds(int appointmentIds[], int appointmentCount)
{
    for (int index = 0; index < appointmentCount; index++)
    {
        appointmentIds[index] = 1000 + (index * 2); // Keep IDs in ascending order.
    }
}
```

### B1 — Observar acceso directo

En `B1`, el programa conoce de antemano los índices 25, 50 y 100. Cada llamada lee una sola posición, aunque la lista tenga 50, 100 o 200 citas. No hay un loop que dependa del tamaño de la lista. El mensaje incluye el ID consultado y la cantidad de accesos directos para que puedas registrarla en la tabla.

Dentro de `main`, elimina solamente el `/*` que aparece antes de `B1` y el `*/` que termina ese bloque. Compila y ejecuta antes de activar otro bloque. Para la última columna de la tabla, pregúntate: **al duplicar la cantidad de citas, ¿cambió el número de posiciones que la función tuvo que leer?**

### B2 — Observar linear search

En `B2`, no se mide una sola búsqueda. Hay tres listas de diez IDs objetivo: `searchTargets50`, `searchTargets100` y `searchTargets200`. Sus valores ya están escritos en el código porque sabemos que aparecen en las listas de citas correspondientes. Cada lista de objetivos incluye IDs cercanos al inicio, al medio y al final, de modo que la muestra sea variada y repetible.

El loop de B2 llama a `countLinearComparisons` una vez por cada objetivo y acumula el resultado. Al dividir el total entre `SEARCH_COUNT`, el programa muestra el promedio de diez búsquedas. Para la última columna de la tabla, compara cada promedio con el siguiente: **cuando `n` se duplica, ¿el promedio se mantiene, se duplica aproximadamente o cambia de otra forma?**

Después de observar B1, elimina los dos delimitadores `/*` y `*/` que rodean solamente B2. Vuelve a compilar y ejecutar. B1 permanece activo, así que podrás ver su resultado junto al nuevo patrón lineal.

### B3 — Observar binary search

En `B3`, no se declaran objetivos nuevos. El loop vuelve a recorrer exactamente `searchTargets50`, `searchTargets100` y `searchTargets200`, declarados en B2. La única diferencia es que ahora llama a `countBinaryComparisons`. Reutilizar los mismos diez IDs permite comparar los promedios de ambos algoritmos de forma justa.

Los arrays están ordenados porque `fillSortedAppointmentIds` los preparó en orden ascendente. Cada comparación del elemento central elimina aproximadamente la mitad de los IDs restantes. Para la última columna de la tabla, compara los tres promedios: **al duplicar `n`, ¿cuántas comparaciones adicionales aparecen y cómo se compara ese cambio con B2?**

Cuando hayas registrado B2, elimina los delimitadores que rodean B3 y ejecuta de nuevo. **No vuelvas a comentar B2:** B3 depende de las listas de objetivos que B2 declaró. La función `countBinaryComparisons` ya está implementada y documentada debajo de `main`; úsala para analizar el resultado, no para reescribirla.

> ⚠️ **El orden es indispensable para B3.** Si los IDs no estuvieran ordenados, comparar el valor central no permitiría descartar correctamente una mitad.

### B4 — Observar comparación de pares

En `B4`, no se busca un ID específico. Se cuenta cuántos pares distintos visitarían dos loops anidados si el sistema necesitara comparar citas entre sí. Para la última columna de la tabla, compara cada número con el siguiente: **al duplicar la cantidad de citas, ¿por cuánto se multiplica aproximadamente la cantidad de pares?**

Por último, elimina los delimitadores que rodean B4 y vuelve a ejecutar. `countPossiblePairs` también está completamente implementada y documentada debajo de `main`; concéntrate en relacionar sus dos loops con el crecimiento de los números producidos.

### Ejecutar, registrar y comparar

Después de activar B1 y, progresivamente, B2, B3 y B4, compila y ejecuta el programa cada vez:

```bash
g++ appointment_analysis_example.cpp -o appointment_analysis_example
./appointment_analysis_example
```

Luego completa esta tabla con los números que produjo tu ejecución y una observación breve para cada patrón. Puedes crearla digitalmente o escribirla a mano; en ambos casos, guarda una imagen clara llamada `complexity_growth_table.png` o `complexity_growth_table.jpg` en la raíz de tu repositorio.

| Patrón / operación contada | `n = 50` | `n = 100` | `n = 200` | ¿Qué ocurrió al duplicar `n`? |
| --- | ---: | ---: | ---: | --- |
| B1 — accesos directos |  |  |  |  |
| B2 — promedio de comparaciones de linear search |  |  |  |  |
| B3 — promedio de comparaciones de binary search |  |  |  |  |
| B4 — pares posibles |  |  |  |  |

No escribas una clasificación sin justificarla. Usa los números y las preguntas de B1–B4 para describir el comportamiento que observaste en la última columna.

> **Sugerencia de visualización:** crea una gráfica opcional con `n` en el eje horizontal y la cantidad de operaciones en el eje vertical. Una gráfica para B1, B2 y B3 permite comparar sus cambios sin que una escala muy grande los oculte. Haz una segunda gráfica separada para B4 y observa la forma de su crecimiento. No necesitas entregar las gráficas, pero hacerlas puede ayudarte a interpretar la tabla.

💾 **Buen momento para el commit** — completaste un ejemplo guiado que cuenta evidencia de trabajo para `O(1)`, `O(n)`, `O(log n)` y `O(n²)`. Commit: `"Complete appointment complexity example"`.

## Parte C — Analizar código resuelto (35 min)

### Contexto

La oficina de actividades estudiantiles necesita justificar decisiones sobre cómo consulta y compara información de actividades. En esta parte no implementarás funciones nuevas: el objetivo es leer código que ya funciona, ejecutarlo y explicar qué trabajo repite cada función cuando la colección crece.

Crea un segundo archivo llamado `campus_activity_analysis.cpp` y copia el programa completo siguiente. Todas las funciones y las pruebas ya están resueltas deliberadamente. No cambies su lógica: úsalo como evidencia para responder las preguntas de análisis.

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 8 - Big-O / Time Complexity
 * Description: Resolved code for time-complexity analysis
 * Due date: [Date]
 */

#include <iostream>
using namespace std;

/* Returns the ID stored at a valid position already known by the caller. */
int getActivityIdAt(const int activityIds[], int activityIndex)
{
    return activityIds[activityIndex]; // Read one known array position.
}

/* Returns how many activities have an active status. */
int countActiveActivities(const bool isActive[], int activityCount)
{
    int activeCount = 0;

    for (int index = 0; index < activityCount; index++)
    {
        if (isActive[index])
        {
            activeCount++;
        }
    }

    return activeCount;
}

/*
 * Returns how many middle IDs binary search compares before finding targetId.
 * Precondition: activityIds is sorted in ascending order and contains targetId.
 */
int countBinaryComparisons(const int activityIds[], int activityCount, int targetId)
{
    int first = 0;
    int last = activityCount - 1;
    int comparisons = 0;

    while (first <= last)
    {
        int middle = (first + last) / 2;
        comparisons++;

        if (activityIds[middle] == targetId)
        {
            break;
        }
        else if (activityIds[middle] < targetId)
        {
            first = middle + 1;
        }
        else
        {
            last = middle - 1;
        }
    }

    return comparisons;
}

/* Returns how many times a positive count can be reduced by half before reaching 1. */
int countHalvingSteps(int itemCount)
{
    int steps = 0;

    while (itemCount > 1)
    {
        itemCount = itemCount / 2; // Remove approximately half of the remaining items.
        steps++;
    }

    return steps;
}

/* Returns the number of distinct activity pairs visited by two nested loops. */
int countPossiblePairs(int activityCount)
{
    int pairCount = 0;

    for (int first = 0; first < activityCount; first++)
    {
        for (int second = first + 1; second < activityCount; second++)
        {
            pairCount++; // One distinct pair would be compared here.
        }
    }

    return pairCount;
}

int main()
{
    // These activity IDs are sorted so binary search is valid.
    int activityIds[] = {101, 104, 108, 115, 130, 136, 140, 155};
    bool isActive[] = {true, false, true, true, false, false, true, false};
    int activityCount = 8;

    cout << "Activity at index 2: "
         << getActivityIdAt(activityIds, 2) << endl;
    cout << "Active activities: "
         << countActiveActivities(isActive, activityCount) << endl;
    cout << "Binary comparisons for 140: "
         << countBinaryComparisons(activityIds, activityCount, 140) << endl;
    cout << "Halving steps for 8: " << countHalvingSteps(8) << endl;
    cout << "Halving steps for 16: " << countHalvingSteps(16) << endl;
    cout << "Halving steps for 32: " << countHalvingSteps(32) << endl;
    cout << "Possible pairs for 4 activities: " << countPossiblePairs(4) << endl;
    cout << "Possible pairs for 8 activities: " << countPossiblePairs(8) << endl;

    return 0;
}

/*
 * Analysis Answers
 *
 * C1 — Direct access
 * Answer:
 *
 * C2 — Count active activities
 * Answer:
 *
 * C3 — Binary search
 * Answer:
 *
 * C4 — Halving steps
 * Answer:
 *
 * C5 — Distinct pairs
 * Answer:
 *
 * C6 — Compare the observed patterns
 * Answer:
 *
 * C7 — Sorting decision
 * Answer:
 *
 */
```

Compila y ejecuta el programa antes de responder:

```bash
g++ campus_activity_analysis.cpp -o campus_activity_analysis
./campus_activity_analysis
```

### Respuestas de análisis

Al final de `campus_activity_analysis.cpp` ya aparece un bloque de comentarios llamado `Analysis Answers`. Escribe allí tus respuestas a C1–C7, debajo de cada `Answer:`. Esa sección de comentarios es la evidencia que se evaluará para la Parte C; no crees un archivo adicional para las respuestas C1–C7.

Puedes responder en inglés o en español. No copies las preguntas sin contestarlas y no respondas solo con una notación como `O(n)`: justifica cada respuesta usando el código o el output observado.

Responde las siguientes preguntas después de leer el código y observar su ejecución:

1. **C1 — Direct access.** Para `getActivityIdAt`, identifica qué representa `n`, la operación representativa, la clasificación Big-O y una justificación basada en el código.
2. **C2 — Count active activities.** Para `countActiveActivities`, identifica qué representa `n`, la operación representativa, la clasificación Big-O y una justificación basada en el código.
3. **C3 — Binary search.** Para `countBinaryComparisons`, identifica qué representa `n`, la operación representativa, la clasificación Big-O y el requisito que deben cumplir los IDs antes de llamar la función. Explica por qué ese requisito importa.
4. **C4 — Halving steps.** Traza los valores de `itemCount` cuando inicia en 16. Luego indica qué operación se repite, cómo cambia el número de pasos cuando se pasa de 16 a 32 y qué clasificación Big-O justifica ese comportamiento.
5. **C5 — Distinct pairs.** Explica por qué el segundo loop de `countPossiblePairs` inicia en `first + 1`. Describe qué ocurriría si iniciara en 0 e identifica la clasificación Big-O de la función.
6. **C6 — Compare the observed patterns.** Usando el output de `main`, compara los resultados de acceso directo, conteo de actividades, reducción a la mitad y pares. ¿Qué evidencia del código y del output apoya cada comportamiento?
7. **C7 — Sorting decision.** Los IDs podrían llegar desordenados. Explica cuándo preferirías linear search sin ordenar primero y cuándo podría justificarse ordenar una vez para usar binary search muchas veces. Menciona el costo de sorting que discutieron en la lecture, aunque no lo implementes aquí.

> **Nota sobre sorting:** selection sort busca repetidamente el menor valor de la parte que todavía no está ordenada y lo coloca en su posición final. Por esos recorridos repetidos, su crecimiento es cuadrático. No implementarás selection sort ni usarás `std::sort` en este laboratorio; la idea se usa solo para analizar el costo que podría existir antes de aplicar binary search.

💾 **Buen momento para el commit** — agregaste respuestas razonadas sobre el código ya ejecutado. Commit: `"Add time complexity analysis answers"`.

## Parte D — Analizar pseudocódigo combinado (20 min)

Esta parte tampoco requiere escribir código. Crea un archivo llamado `pseudocode_analysis_answers.md` y responde allí D1 y D2. Para ambos, identifica qué representa `n`, cuál es la operación representativa, la clasificación Big-O de todo el procedimiento y una justificación. Analiza el caso que puede requerir más trabajo.

### D1 — Buscar y luego recorrer

El sistema recibe IDs ordenados y el estado de actividad de cada evento. Primero busca un ID específico; después necesita contar todas las actividades activas para mostrar un resumen.

```text
PROCEDURE FindAndCountActive(sortedIds, isActive, count, targetId)
    first = 0
    last = count - 1
    found = false

    WHILE first <= last AND found is false
        middle = (first + last) / 2

        IF sortedIds[middle] = targetId
            found = true
        ELSE IF sortedIds[middle] < targetId
            first = middle + 1
        ELSE
            last = middle - 1
        END IF
    END WHILE

    activeCount = 0
    FOR index = 0 TO count - 1
        IF isActive[index] is true
            activeCount = activeCount + 1
        END IF
    END FOR

    RETURN found, activeCount
END PROCEDURE
```

**D1.** Explica el crecimiento de la sección de búsqueda y el de la sección que cuenta actividades. Luego clasifica el procedimiento completo. ¿Por qué no se multiplican automáticamente los costos de ambas secciones?

### D2 — Revisar conflictos potenciales

Antes de asignar los salones, el sistema compara pares distintos de actividades para identificar cuáles tienen la misma hora de inicio. Cada par se debe revisar solo una vez.

```text
PROCEDURE CountPotentialConflicts(startTimes, count)
    conflictCount = 0

    FOR first = 0 TO count - 1
        second = first + 1

        WHILE second < count
            IF startTimes[first] = startTimes[second]
                conflictCount = conflictCount + 1
            END IF

            second = second + 1
        END WHILE
    END FOR

    RETURN conflictCount
END PROCEDURE
```

**D2.** Explica por qué `second` comienza en `first + 1`, qué acción se cuenta como operación representativa y cuál es el orden de crecimiento en el caso de más trabajo. Apoya tu respuesta con lo que ocurriría al duplicar `count`.

Usa esta estructura en `pseudocode_analysis_answers.md`:

```md
# Lab 8 — Part D Pseudocode Analysis

## D1 — Search and scan pseudocode
**Answer:**

## D2 — Potential-conflict pseudocode
**Answer:**
```

Puedes responder en inglés o español, pero justifica cada clasificación con el flujo de control del pseudocódigo.

💾 **Buen momento para el commit** — analizaste pseudocódigo que combina patrones de búsqueda, recorrido y comparación. Commit: `"Analyze combined pseudocode"`.

## Parte E — Revisión, evaluación y entrega (15 min)

Antes de entregar, verifica:

- `appointment_analysis_example.cpp` compila con `g++ appointment_analysis_example.cpp -o appointment_analysis_example`.
- `complexity_growth_table.png` o `complexity_growth_table.jpg` muestra los resultados de 50, 100 y 200 citas y una observación sobre cada patrón.
- `campus_activity_analysis.cpp` compila con `g++ campus_activity_analysis.cpp -o campus_activity_analysis`.
- El bloque final `Analysis Answers` de `campus_activity_analysis.cpp` responde C1–C7 con justificaciones propias.
- `pseudocode_analysis_answers.md` responde D1–D2 con justificaciones propias.
- El programa de análisis se conserva resuelto; no tiene TODOs de implementación.
- Los nombres, comentarios y mensajes que forman parte del programa están en inglés; las respuestas dentro de `Analysis Answers` y `pseudocode_analysis_answers.md` pueden estar en inglés o español.
- Los datos de prueba son dummy y `.gitignore` evita que los ejecutables aparezcan en Source Control.

### Criterios de evaluación (100 puntos)

| Criterio | Puntos |
| --- | ---: |
| Ejemplo guiado, tabla de crecimiento y evidencia correcta para los cuatro patrones | 20 |
| Respuestas C1–C3: análisis de acceso directo, recorrido y binary search | 30 |
| Respuestas C4–C5: reducción a la mitad y pares distintos | 20 |
| Respuestas C6–C7: comparación de evidencia y decisión de sorting | 10 |
| Respuestas D1–D2: análisis de pseudocódigo combinado | 10 |
| Archivos compilables, encabezados, datos dummy, `.gitignore` y commits claros | 10 |

## Entregable del laboratorio

Envía al profesor el enlace de tu repositorio `lab8-big-o` en GitHub, con:

- `appointment_analysis_example.cpp`, completado durante el ejemplo guiado.
- `complexity_growth_table.png` o `complexity_growth_table.jpg`, con la tabla completada de los tamaños 50, 100 y 200. Puede ser una tabla digital o una foto clara de una tabla hecha a mano.
- `campus_activity_analysis.cpp`, copiado y ejecutado sin modificar su lógica.
- `campus_activity_analysis.cpp`, con el bloque final `Analysis Answers` completado para C1–C7.
- `pseudocode_analysis_answers.md`, con las respuestas completas D1–D2.
- Al menos **3 commits** visibles: setup, ejemplo guiado y respuestas de análisis.
- Un `.gitignore` funcionando, sin ejecutables ni archivos objeto en el repositorio.
- Encabezados estándar completos con tu nombre y fecha reales en ambos archivos fuente.

## Próxima sesión

En la Semana 9 comenzarán **Linked Lists**. Allí verán una nueva implementación para almacenar colecciones y podrán comparar los costos de sus operaciones con los arrays que han usado hasta ahora.
