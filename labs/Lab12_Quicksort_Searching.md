# Lab 12 — Quicksort, Searching, and Project Application
## COEN 2220 — Programming 2

**Instructor:** Wilson Lozano

**Duración:** 110 min (sesión de 2 horas crédito nominales)  
**Precede a:** Semana 13 — Binary Trees y Binary Search Trees  
**Requisitos:** Lecture de Semana 12 impartida; Lab 11 completado  
**Basado en:** Gaddis, *Starting Out with C++*, 8va ed., Cap. 8, sección 8.1 (linear y binary search); Cap. 19, secciones 19.6 (recursive binary search) y 19.8 (quicksort). Los escenarios y datos de esta guía son originales del curso.

---

## Objetivos

Al finalizar este laboratorio, el estudiante podrá:

1. Explicar cómo una partición coloca un pivot sin ordenar todavía sus dos lados.
2. Implementar quicksort sobre una vista de pointers sin mover los registros originales.
3. Aplicar binary search iterativa al mismo campo por el cual está ordenada una vista.
4. Distinguir una posición de la vista, una posición original y un código de registro.
5. Justificar la elección entre linear y binary search, considerando el orden disponible y el costo de prepararlo.
6. Ordenar y consultar un reporte temporal de delivery sin alterar la colección operacional.

> **Alcance:** B es un ejemplo completamente resuelto y progresivo. C–D contienen los TODO evaluados; E aplica la técnica al proyecto dentro del repo de este lab. Usa arrays y pointers, sin contenedores STL, `std::sort` ni exceptions.

## Parte A — Preparar la carpeta y el repositorio (5 min)

> ⚠️ **Este paso es el que más se presta a error — léelo con cuidado antes de hacer clic en nada.** Cada lab tiene su propio repositorio. No inicialices Git en la carpeta padre `COEN2220` ni dentro de otro lab.

1. **A1.** Selecciona `File → Open Folder...` en VS Code o VSCodium.
2. **A2.** Navega a tu carpeta padre del curso, por ejemplo `COEN2220`. Si no existe, créala desde este diálogo en un lugar de fácil acceso.
3. **A3.** Dentro del diálogo, crea `lab12-quicksort-searching` como carpeta hermana de los labs anteriores y ábrela con **Open** / **Select Folder**.
4. **A4.** Confirma que el Explorer muestre `LAB12-QUICKSORT-SEARCHING` como raíz. Si muestra la carpeta padre u otro lab, usa `File → Close Folder` y vuelve a abrir la carpeta correcta.

> **¿Por qué importa tanto esto?** Un repo iniciado en la carpeta padre mezcla todas las prácticas en un historial; uno iniciado dentro de otro lab mezcla sus entregas. Esta práctica debe poder revisarse por separado.

En Source Control, selecciona **Initialize Repository** para iniciar el historial de esta carpeta. Crea `.gitignore` con los nombres de los ejecutables que usarás:

```gitignore
# Compiled executables and object files
*.exe
*.o
equipment_catalog_example
parcel_pickup_view
delivery_report_practice

# Editor configuration
.vscode/
```

Haz stage → commit → **Publish Branch**, con el flujo ya practicado. Usa `"Initial setup and gitignore"`.

💾 **Por qué este commit importa:** guarda la organización inicial antes de escribir programas y mantiene los ejecutables fuera de la entrega.

## Parte B — Ejemplo guiado: localizar equipos en un catálogo (25 min)

En la lecture aprendimos cómo quicksort ordena registros: selecciona un pivot, separa los valores mediante una partición y ordena recursivamente ambos lados. También estudiamos cómo buscar en una colección ya ordenada. En esta parte aplicaremos ambas técnicas al inventario de un laboratorio de electrónica: una colección de registros que identifica cada equipo y el gabinete donde está guardado. El personal necesita encontrar el gabinete de un equipo a partir de su código. Construiremos un programa que muestre los registros iniciales, los ordene por código y permita consultar la ubicación de un equipo. Así distinguiremos el trabajo que prepara la colección del que responde cada consulta.

### B1 — Entender los registros y ejecutar el estado inicial

Un laboratorio de electrónica mantiene un inventario con equipos registrados en orden de inspección. Cada registro tiene un código numérico único y la ubicación física donde está guardado. El encargado necesita consultar repetidamente una ubicación a partir del código del equipo; el orden de inspección no tiene que conservarse en este catálogo de consulta.

El programa declarará `Equipment`, un `struct` que representa **un equipo**. Su member `assetCode` identifica el equipo y `cabinet` identifica el gabinete donde está guardado. El array `equipment` contendrá ocho registros dummy. Son datos fijos para comparar las ejecuciones y seguir los intercambios; después de comprobar el ejemplo puedes sustituirlos, conservando códigos únicos y actualizando la cantidad.

Ordenaremos **registros completos** por `assetCode` ascendente. Intercambiar solo los códigos asociaría un equipo con el gabinete equivocado. Binary search consultará después ese mismo campo; el gabinete se recupera del registro encontrado, no participa en el ordenamiento.

El recorrido del programa, cuando todos los pasos estén activos, será:

1. `main` mostrará el catálogo inicial y llamará a `quickSortEquipment` con el rango inclusivo `0..7`.
2. `quickSortEquipment` pedirá a `partitionEquipment` que reorganice un segmento. Este helper recibirá el array y sus límites, modificará los registros mediante `swapEquipment` y devolverá el índice final del pivot.
3. `quickSortEquipment` usará ese índice para solicitar primero su lado izquierdo y después su lado derecho. Una llamada con `first >= last` terminará sin leer el array; al regresar de ambas ramas no habrá una operación merge.
4. De vuelta en `main`, se mostrará el catálogo ordenado y se leerá por consola el código solicitado. `binaryFindEquipment` recibirá el catálogo ordenado, los límites y el objetivo; devolverá una posición o `-1`, sin modificar registros.
5. `main` comprobará el resultado antes de acceder al registro para mostrar su gabinete.

Crea `equipment_catalog_example.cpp` con el programa siguiente. Los prototipos están antes de `main`; las definiciones de las funciones libres van después de `main`, fuera de sus llaves. B1 ya funciona. B2 y B3 están resueltos dentro de comentarios `/* ... */`; las etiquetas `// --- STEP ... ---` permanecen fuera de esos comentarios.

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 12 - Quicksort and Searching
 * Description: Guided equipment catalog and code lookup
 * Due date: [Date]
 */

#include <iostream>
#include <string>
using namespace std;

struct Equipment
{
    int assetCode;   // Identify one physical device; codes are unique.
    string cabinet; // Keep its storage location attached to the code.
};

void displayEquipment(const Equipment equipment[], int count);
void swapEquipment(Equipment& left, Equipment& right);
int partitionEquipment(Equipment equipment[], int first, int last);
void quickSortEquipment(Equipment equipment[], int first, int last);
int binaryFindEquipment(const Equipment equipment[], int first, int last, int target);

int main()
{
    const int COUNT = 8;
    Equipment equipment[COUNT] = {
        {48, "C-4"}, {21, "A-2"}, {63, "D-1"}, {35, "B-3"},
        {57, "C-2"}, {14, "A-1"}, {72, "D-4"}, {29, "B-1"}
    };

    cout << "Inspection order:" << endl;
    displayEquipment(equipment, COUNT);

    // --- STEP B2: Prepare the catalog for repeated code searches ---
    /*
    quickSortEquipment(equipment, 0, COUNT - 1);
    cout << "Sorted by asset code:" << endl;
    displayEquipment(equipment, COUNT);
    */

    // --- STEP B3: Read a code and retrieve its cabinet safely ---
    /*
    int target;
    cout << "Enter an asset code: ";
    if (!(cin >> target))
    {
        cout << "Invalid code input." << endl;
        return 1; // Stop before searching with an unread value.
    }

    int position = binaryFindEquipment(equipment, 0, COUNT - 1, target);
    if (position == -1)
    {
        cout << "Equipment not found." << endl;
    }
    else
    {
        cout << "Found code " << equipment[position].assetCode
             << " in cabinet " << equipment[position].cabinet << endl;
    }
    */
    return 0;
}

void displayEquipment(const Equipment equipment[], int count)
{
    for (int index = 0; index < count; index++)
    {
        cout << "Code: " << equipment[index].assetCode
             << " | Cabinet: " << equipment[index].cabinet << endl;
    }
}

// --- STEP B2: Define the sorting operations ---
/*
void swapEquipment(Equipment& left, Equipment& right)
{
    Equipment saved = left; // Preserve both fields during the exchange.
    left = right;
    right = saved;
}

int partitionEquipment(Equipment equipment[], int first, int last)
{
    int middle = first + (last - first) / 2;
    swapEquipment(equipment[first], equipment[middle]);
    int pivotCode = equipment[first].assetCode; // Keep this value fixed during the scan.
    int boundary = first; // The smaller group initially has no elements.

    for (int scan = first + 1; scan <= last; scan++)
    {
        if (equipment[scan].assetCode < pivotCode)
        {
            boundary++; // Reserve the next position in the smaller group.
            swapEquipment(equipment[boundary], equipment[scan]);
        }
    }
    swapEquipment(equipment[first], equipment[boundary]); // Complete the partition.
    return boundary; // This final position divides the next recursive ranges.
}

void quickSortEquipment(Equipment equipment[], int first, int last)
{
    if (first >= last)
    {
        return; // Empty and one-record ranges need no partition.
    }
    int pivotIndex = partitionEquipment(equipment, first, last);
    quickSortEquipment(equipment, first, pivotIndex - 1);
    quickSortEquipment(equipment, pivotIndex + 1, last);
}
*/

// --- STEP B3: Define recursive lookup on the sorted catalog ---
/*
int binaryFindEquipment(const Equipment equipment[], int first, int last, int target)
{
    if (first > last)
    {
        return -1; // An empty search range proves that the code is absent.
    }
    int middle = first + (last - first) / 2;
    if (equipment[middle].assetCode == target)
    {
        return middle; // A one-record range must still be checked for a match.
    }
    if (equipment[middle].assetCode < target)
    {
        return binaryFindEquipment(equipment, middle + 1, last, target);
    }
    return binaryFindEquipment(equipment, first, middle - 1, target);
}
*/

/*
 * Analysis Answers
 *
 * B4 - Partition and recursive control
 * After the first partition, identify the pivot code, its returned index, and
 * the two next ranges. Is the whole catalog already sorted? Explain using data.
 * Answer:
 *
 * B5 - A search that fails
 * Trace the recursive search for code 36. List each first, last, middle,
 * inspected code, and next range. At the empty range, explain which calls resume
 * and how -1 reaches main without an invalid array access.
 * Answer:
 */
```

Compila B1 con este comando; repítelo después de activar cada paso para que el ejecutable corresponda al código actual:

```bash
g++ equipment_catalog_example.cpp -o equipment_catalog_example
```

Ejecuta por separado:

```bash
./equipment_catalog_example
```

**Qué deberías ver en B1:** el encabezado `Inspection order:` y los códigos `48, 21, 63, 35, 57, 14, 72, 29`, cada uno con su gabinete del starter. Todavía no se solicita una entrada ni se ordena.

<details>
<summary>Verificación: ¿por qué los prototipos de B2 y B3 pueden estar activos aunque sus definiciones estén comentadas?</summary>

Un prototipo declara cómo se llamaría una función. En B1 nadie llama a las funciones inactivas y no se necesitan sus definiciones para ese ejecutable. Al activar una llamada debes activar también su definición y las de los helpers que utilice.

</details>

### B2 — Activar y seguir la partición antes de buscar

Quita los comentarios `/*` y `*/` de los **dos bloques `STEP B2`**, el de `main` y el de las definiciones de sorting. Conserva B3 comentado. Compila y ejecuta de nuevo.

**Recordatorio de quicksort:** una partición separa los menores que un pivot de los que no son menores y coloca ese pivot en su posición final. Las dos llamadas recursivas ordenan después el interior de ambos lados. Seguimos el esquema del libro y la lecture: elegir la posición central, mover su registro al inicio y recorrer desde `first + 1` hasta `last`.

Para la primera partición, `first = 0` y `last = 7` abarcan los ocho equipos. El centro es `0 + (7 - 0) / 2 = 3`, que contiene el equipo 35. El intercambio inicial coloca ese registro en 0 y el equipo 48 en 3. `pivotCode` conserva 35 hasta terminar esta partición.

`boundary` es el índice del final del grupo menor. Comienza en 0 porque el grupo todavía está vacío: su rango sería `1..0`. El pivot no pertenece a ese grupo. `scan` empieza en 1 para no volver a examinar el pivot. En cada coincidencia menor, primero se incrementa `boundary` para reservar una posición y después se intercambia el registro completo hacia ella; una comparación falsa solo deja avanzar `scan`. El helper devuelve el índice del pivot después del intercambio final, no el centro original.

Por ejemplo, después del intercambio inicial, examinar 21 en el índice 1 produce `21 < 35`, verdadero. `boundary` pasa de 0 a 1 y se intercambia el registro 1 consigo mismo. Al inspeccionar luego 63 en el índice 2, `63 < 35` es falso: el grupo menor sigue terminando en 1 y solamente avanza `scan`. Así se distinguen ampliar el grupo y avanzar la inspección. La traza completa de este mecanismo está en la Parte 2 de la lecture; aquí comprobamos su resultado con los nuevos datos.

La última inspección usa `scan = 7` y el código 29. Al terminar esa iteración, `boundary = 3`; el incremento del loop deja `scan = 8`. La condición `8 <= 7` es falsa y no se consulta `equipment[8]`. Solo después se intercambia el pivot de 0 con el registro de 3. Terminar las comparaciones no equivale a haber colocado ya el pivot.

El siguiente estado muestra la primera partición **después** de las siete comparaciones y de colocar el pivot. Los números son códigos de equipos; las posiciones superiores son índices del array. Los gabinetes se mueven con sus equipos aunque se omitan en este esquema:

```text
Index:       0   1   2   3   4   5   6   7
Asset code: 29  21  14  35  57  63  72  48
                         ^
                    pivotIndex = 3
Left range:  0..2 -> [29 21 14]
Right range: 4..7 -> [57 63 72 48]
```

Este es un punto intermedio del algoritmo, no el output final. La izquierda todavía contiene `29` antes de `14`. Cuando `partitionEquipment` devuelve 3, `quickSortEquipment` llama primero al rango `0..2`; solo cuando esa llamada termina solicita `4..7`. Los registros siguen en el mismo array al regresar; no se concatenan segmentos como una etapa adicional.

Para seguir el primer caso base, esta traza muestra el lado izquierdo. Cada línea indentada es una llamada de `quickSortEquipment` solicitada por la anterior; los códigos entre corchetes identifican los registros de ese segmento, no sus índices:

```text
quickSortEquipment(0, 7) [48 21 63 35 57 14 72 29]
    partitionEquipment -> pivot 35 at index 3
    quickSortEquipment(0, 2) [29 21 14]
        middle=1 -> pivot 21 -> final pivot index=1 -> segment [14 21 29]
        quickSortEquipment(0, 0) [14]
            first >= last: 0 >= 0 is true -> return
        resume quickSortEquipment(0, 2) -> request right range 2..2
        quickSortEquipment(2, 2) [29]
            first >= last: 2 >= 2 is true -> return
        resume quickSortEquipment(0, 2) -> return
    resume quickSortEquipment(0, 7) -> request right range 4..7 [57 63 72 48]
```

La llamada `0..0` no termina todo el algoritmo: devuelve el control a `0..2`, que aún debe solicitar `2..2`. Al concluir ambas, `0..2` devuelve el control a `0..7`, que todavía tiene pendiente `4..7`. La derecha se procesa después con la misma regla; el catálogo completo se muestra únicamente cuando la llamada inicial regresa a `main`.

**Qué deberías ver al terminar B2:** después del catálogo inicial aparece el siguiente catálogo ordenado. Verifica también los gabinetes para comprobar la asociación de los campos:

```text
Sorted by asset code:
Code: 14 | Cabinet: A-1
Code: 21 | Cabinet: A-2
Code: 29 | Cabinet: B-1
Code: 35 | Cabinet: B-3
Code: 48 | Cabinet: C-4
Code: 57 | Cabinet: C-2
Code: 63 | Cabinet: D-1
Code: 72 | Cabinet: D-4
```

<details>
<summary>Verificación: ¿el índice 3 siguió automáticamente al equipo 35 desde el principio hasta el final?</summary>

No. `middle` conserva el entero 3, aunque el equipo 35 se mueva primero a 0. Después `boundary` cambia durante el recorrido y termina en 3 en estos datos. Que el índice inicial y el final coincidan aquí no es una regla del algoritmo ni significa que un índice siga al objeto intercambiado.

</details>

### B3 — Consultar el catálogo ordenado

Mantén activos B1 y B2. Quita los comentarios de los **dos bloques `STEP B3`**, el de `main` y la definición de búsqueda. Compila y ejecuta. Escribe `57` cuando aparezca el prompt.

**Recordatorio de binary search:** sobre códigos ascendentes, un código central menor que el objetivo permite descartar el centro y toda la mitad inferior. Un código central mayor permite descartar el centro y la mitad superior. Por eso los nuevos límites excluyen el centro ya comparado. La función recursiva devuelve el resultado de la única mitad que todavía puede contener el código.

La siguiente traza sigue las llamadas para 57. La indentación significa una llamada iniciada por la línea anterior; las flechas de retorno representan control y resultados, no enlaces entre nodes:

```text
main calls binaryFindEquipment(0, 7, target=57)
    first=0 last=7 middle=3 inspected=35 -> 35 < 57 -> request 4..7
    binaryFindEquipment(4, 7, target=57)
        first=4 last=7 middle=5 inspected=57 -> match -> return 5
    resume binaryFindEquipment(0, 7) -> return 5
resume main -> position=5 -> read equipment[5].cabinet -> C-2
```

**Qué deberías ver:** después de los dos catálogos, el programa imprime:

```text
Enter an asset code: 57
Found code 57 in cabinet C-2
```

Ejecuta nuevamente con `36` para provocar una ausencia y con `14` y `72` para consultar los extremos. Ante una entrada no numérica debe aparecer `Invalid code input.`. El `-1` de una ausencia se comprueba antes de usar `position`; no representa una posición válida del catálogo.

<details>
<summary>Verificación: ¿por qué la búsqueda no usa el mismo caso base que quicksort?</summary>

Quicksort puede terminar con un elemento porque ese segmento ya está ordenado. Binary search todavía debe comparar ese único código con el objetivo. Solo cuando `first > last` no quedan posiciones por comprobar y puede declararse la ausencia sin otro acceso.

</details>

### B4 — Explicar lo observado

Responde B4 y B5 al final de `equipment_catalog_example.cpp`, en el bloque `Analysis Answers`. Las respuestas de análisis de este lab pueden estar en español o inglés; conserva en inglés los identificadores, comments explicativos del programa, prompts y mensajes.

💾 **Buen momento para el commit:** el ejemplo ya muestra el flujo desde el catálogo inicial hasta una consulta válida o ausente. Usa `"Complete equipment sorting and lookup example"`.

## Parte C — TODO: ordenar una vista de paquetes disponibles (25 min)

En B se permitió reorganizar el catálogo completo. Ahora cambia una condición del problema: el registro de recepción debe conservarse y algunas entradas no pueden aparecer en la consulta. Usaremos la vista de pointers practicada en el Lab 11 para separar los registros originales de la secuencia que necesita el usuario que busca.

### C1 — Contexto y reglas de la consulta

La oficina de correspondencia del campus recibe paquetes y registra su llegada para auditoría. Cada paquete recibe un código numérico único de recogido, un locker y un estado `ready`. El estado es `true` cuando el personal ya colocó el paquete en el locker y puede entregarlo; es `false` mientras todavía se está procesando. Una persona indica su código y la oficina debe informar el locker **solo si ese paquete está listo**.

El programa declarará `Parcel`, un `struct` para **un paquete**, con `pickupCode`, `locker` y `ready`. El array `parcels` conservará los registros completos en orden de recepción. El array `readyView` guardará únicamente direcciones de paquetes listos. Su cantidad lógica, `readyCount`, puede ser menor que la capacidad del array.

La selección ya está resuelta en el starter para concentrar tu trabajo en sorting y searching. Ordenarás `readyView` por `pickupCode` ascendente para responder muchas consultas durante la jornada. No moverás posiciones de `parcels` ni cambiarás sus campos. Esta diferencia exige intercambiar pointers y comparar el campo del objeto señalado, en vez de intercambiar objetos completos como en B.

El programa muestra los registros al inicio, construye la vista y, cuando completes C, muestra la vista ordenada y otra vez el registro original. En D añadirá la consulta. Los datos fijos permiten comprobar tanto el filtro como el orden; el código buscado será una entrada por consola.

Antes de completar el código, decide:

1. ¿Qué se pierde si intercambias solamente `pickupCode` entre dos objetos?
2. ¿Qué tipo debe tener la variable temporal que intercambia dos posiciones de `readyView`?
3. ¿Qué rango debes ordenar si la capacidad es ocho pero solo hay seis pointers seleccionados?
4. ¿En qué momento debes guardar el código del pivot para que no cambie mientras se intercambian posiciones?

### C2 — Crear el starter y resolver solamente C

Crea `parcel_pickup_view.cpp` con este template completo. Las funciones están declaradas antes de `main` y sus cuerpos están después. Los TODO de C están agrupados en las funciones de sorting; los TODO de D están agrupados en la función de consulta. **Resuelve solo `(Part C)` ahora e ignora D.** La llamada de D permanece comentada para evitar mezclar las etapas.

`quickSortReady` recibe una vista y sus límites inclusivos, reorganiza sus pointers y no devuelve un valor. Llama a `partitionReady`, que devuelve el índice final del pivot, y luego se llama para cada lado. `swapParcelPointers` recibe por referencia dos posiciones pointer y debe intercambiar sus direcciones. `binaryFindReady`, que completarás en D, recibirá la vista ya ordenada, su cantidad y un código; devolverá un índice de **esa vista** o `-1`.

La sintaxis `Parcel *&left` combina dos ideas ya usadas: `left` es una referencia a una variable cuyo valor es una dirección de `Parcel`. Por eso asignarle otra dirección modifica la posición pointer que el llamador entregó. En cambio, asignar a `left->pickupCode` modificaría el registro original, lo cual viola el requisito. Los elementos de `parcels` permanecen vivos mientras se usa la vista; no habrá `new` y no corresponderá `delete`.

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 12 - Quicksort and Searching
 * Description: Preserve parcel arrival records and search a ready-only view
 * Due date: [Date]
 */

#include <iostream>
#include <string>
using namespace std;

struct Parcel
{
    int pickupCode; // Unique lookup key supplied by the recipient.
    string locker; // Physical location, not a sorting key.
    bool ready;    // Only ready parcels belong in the lookup view.
};

void displayParcels(const Parcel parcels[], int count);
void displayReady(Parcel *view[], int count);
void swapParcelPointers(Parcel *&left, Parcel *&right);
int partitionReady(Parcel *view[], int first, int last);
void quickSortReady(Parcel *view[], int first, int last);
int binaryFindReady(Parcel *view[], int count, int target);

int main()
{
    const int CAPACITY = 8;
    Parcel parcels[CAPACITY] = {
        {640, "L-4", true},  {215, "L-8", false},
        {580, "L-2", true},  {325, "L-6", true},
        {710, "L-1", true},  {460, "L-5", false},
        {190, "L-3", true},  {505, "L-7", true}
    };
    Parcel *readyView[CAPACITY]; // Borrow addresses; do not delete these objects.
    int readyCount = 0;

    cout << boolalpha << "Original arrival records:" << endl;
    displayParcels(parcels, CAPACITY);
    for (int index = 0; index < CAPACITY; index++)
    {
        if (parcels[index].ready)
        {
            readyView[readyCount] = &parcels[index];
            readyCount++; // Count initialized view positions only.
        }
    }

    quickSortReady(readyView, 0, readyCount - 1);
    cout << "Ready parcels by pickup code:" << endl;
    displayReady(readyView, readyCount);
    cout << "Original arrival records after sorting:" << endl;
    displayParcels(parcels, CAPACITY);

    // --- STEP D1: Activate after completing the search function ---
    /*
    int target;
    cout << "Enter a pickup code: ";
    if (!(cin >> target))
    {
        cout << "Invalid code input." << endl;
        return 1;
    }
    int position = binaryFindReady(readyView, readyCount, target);
    if (position == -1)
    {
        cout << "No ready parcel with that code." << endl;
    }
    else
    {
        cout << "Pickup code " << readyView[position]->pickupCode
             << " | Locker: " << readyView[position]->locker << endl;
    }
    */
    return 0;
}

// ===== Resolve only these TODOs now (Part C) =====
void swapParcelPointers(Parcel *&left, Parcel *&right)
{
    // TODO (Part C): Exchange the two borrowed addresses without changing objects.
}

int partitionReady(Parcel *view[], int first, int last)
{
    // TODO (Part C): Select the central record and partition by ascending pickupCode.
    // TODO (Part C): Follow the lecture's pivot-at-first scheme and return its final index.
    return first; // Temporary placeholder; replace when implementing the partition.
}

void quickSortReady(Parcel *view[], int first, int last)
{
    // TODO (Part C): Handle base cases and request partitioning and recursive sorting.
}

// ===== Resolve this TODO later (Part D) =====
int binaryFindReady(Parcel *view[], int count, int target)
{
    // TODO (Part D): Implement iterative binary search by pickupCode on the sorted view.
    return -1; // Temporary placeholder; replace with the complete search.
}

void displayParcels(const Parcel parcels[], int count)
{
    for (int index = 0; index < count; index++)
    {
        cout << "Code: " << parcels[index].pickupCode
             << " | Locker: " << parcels[index].locker
             << " | Ready: " << parcels[index].ready << endl;
    }
}

void displayReady(Parcel *view[], int count)
{
    for (int index = 0; index < count; index++)
    {
        cout << "View index: " << index
             << " | Code: " << view[index]->pickupCode
             << " | Locker: " << view[index]->locker << endl;
    }
}

/*
 * Analysis Answers
 *
 * C3 - Partition decisions
 * Trace the first partition of the six-pointer ready view. Record first, last,
 * middle, pivotCode, boundary, scan, and codes before and after each comparison.
 * Include the loop's final false test and the final pivot exchange.
 * Answer:
 *
 * C4 - Data and memory
 * Explain what changes when swapping two Parcel pointers by reference.
 * Identify who owns the objects and why the view must not delete them.
 * Answer:
 *
 * D3 - Search positions and absence
 * Trace code 505 and the absent ready-view code 460. Give the checked ranges
 * and explain why a view index cannot be used as an index into parcels.
 * Answer:
 *
 * D4 - Time and auxiliary space
 * For n ready parcels and q queries, compare sorting once then using binary
 * search with scanning the unsorted ready view for every query. Give worst-case
 * bounds for this quicksort, balanced-partition bounds, and iterative-search
 * auxiliary space. Distinguish the view storage from recursive-call storage.
 * Answer:
 */
```

En el template, `readyCount` comienza en cero porque todavía no se ha seleccionado ningún paquete. Cada dirección se guarda en la primera posición libre y luego se incrementa la cantidad. Por eso solamente `0..readyCount - 1` contiene pointers válidos. El starter deja activos esos pasos de selección, pero completar sorting y searching requiere tu solución.

### C3 — Implementar, ejecutar y comprobar

Mantén la variante de partición de B: posición central, pivot trasladado al inicio, comparación estricta `<` y colocación final. Adapta el dato intercambiado y el acceso al código. No necesitas una clase nueva ni un array auxiliar para combinar segmentos.

Compila el archivo:

```bash
g++ parcel_pickup_view.cpp -o parcel_pickup_view
```

Ejecuta después de completar C:

```bash
./parcel_pickup_view
```

El template inicial compila, pero sus cuerpos provisionales no ordenan. El resultado requerido después de resolver C es una vista con los códigos `190, 325, 505, 580, 640, 710`, en ese orden, y con sus lockers originales. `215` y `460` no deben aparecer en la vista porque no están listos. Los listados originales, antes y después, deben coincidir en todos sus campos y posiciones.

Antes de D, prueba también el rango lógico vacío `0..-1`, un único pointer válido en `0..0` y el ordenamiento completo repetido. Usa arrays de capacidad positiva; para un rango vacío no leas ninguna posición. Responde C3 y C4 en los comentarios finales. Para C3 conserva los mismos campos de estado en cada paso y distingue el valor del código de su índice.

💾 **Buen momento para el commit:** ya probaste el filtro dado, tu partición y tu recursión, conservando los datos originales. Usa `"Sort ready parcel pointer view by pickup code"`.

## Parte D — TODO: buscar en la vista y explicar los resultados (20 min)

La vista de C ya reúne los paquetes listos y los organiza por su código. Ahora resolverás la consulta de la persona que viene a recoger uno. La búsqueda debe devolver una posición de `readyView`; esa posición permite obtener el locker a través del pointer seleccionado. No representa el código ni la posición original del paquete en `parcels`.

### D1 — Razonar los límites antes del loop

Continúa en `parcel_pickup_view.cpp` y resuelve **solo `(Part D)`**. Conserva tu solución de C. Implementa `binaryFindReady` mediante un loop, para contrastar con la versión recursiva resuelta en B. Después quita los comentarios del bloque `STEP D1` de `main`; ese bloque ya lee y valida el código, llama a tu función y protege el acceso al resultado.

Antes de escribir la búsqueda, determina:

1. ¿Qué valor inicial de `last` excluye las posiciones no seleccionadas de un array de capacidad ocho?
2. ¿Cómo reconocerás que el rango está vacío sin leer un pointer no inicializado?
3. Si el código central es menor que el objetivo, ¿qué posiciones puedes descartar y por qué? ¿Qué pasa si es mayor?
4. ¿Qué debes devolver si el último rango de un elemento tampoco contiene el código?

La función recibe `count`, que es la cantidad lógica, no el índice final. Debe aceptar `count == 0` y devolver `-1` sin consultar `view[0]`. Durante una consulta no se ordena nuevamente ni se cambian los registros o los pointers.

### D2 — Verificar consultas distintas

Vuelve a compilar y ejecutar con los comandos de C3. La tabla define los casos mínimos: cada fila es una ejecución con una entrada distinta. En esta aplicación, “no listo” y “no recibido” producen el mismo mensaje de consulta porque ninguno está disponible para recogido; una consulta administrativa sobre todos los registros sería otra operación.

| Entrada | Qué representa | Resultado que debe comprobarse |
| ---: | --- | --- |
| 505 | Paquete listo dentro de la vista. | Locker `L-7`. |
| 190 | Primer código de la vista ordenada. | Locker `L-3`. |
| 710 | Último código de la vista ordenada. | Locker `L-1`. |
| 460 | Paquete recibido, todavía no listo. | `No ready parcel with that code.` |
| 999 | Código inexistente. | El mismo mensaje de ausencia en la vista. |

Prueba además la función con cantidad cero y con una vista de un pointer: busca su código y luego otro. Para esas pruebas puedes agregar llamadas temporales en `main` usando los arrays existentes; guarda los resultados observados en los comentarios finales.

Responde D3 y D4 en `Analysis Answers`. Recuerda que **tiempo** cuenta el crecimiento del trabajo y **espacio auxiliar** cuenta memoria adicional simultánea. En D4, `n` es la cantidad de paquetes listos, no la capacidad; `q` es el número de consultas realizadas sin reconstruir la vista. No atribuyas a quicksort un tiempo garantizado `O(n log n)` solo por elegir el centro como pivot.

💾 **Buen momento para el commit:** la consulta ya distingue coincidencias, extremos y ausencias sin confundir índices o cantidades. Usa `"Implement and verify iterative parcel lookup"`.

## Parte E — Aplicación al proyecto: ordenar delivery y consultar por ID (20 min)

En C–D ordenaste y buscaste por un mismo campo. El proyecto de cafetería plantea una diferencia importante: su reporte agrupa órdenes por edificio y prioridad, pero el usuario puede preguntar por un ID. Retomaremos la vista de órdenes del Lab 11 para implementar el reporte y elegir una búsqueda que funcione con ese orden.

### E1 — Retomar el problema y preparar los datos

En el proyecto, la cafetería universitaria registra órdenes `DINE_IN` y `DELIVERY`. La preparación conserva una secuencia FIFO. Por separado, el personal de delivery necesita un reporte temporal de órdenes activas organizado por edificio, prioridad descendente e ID ascendente. Una orden activa está `PENDING` o `IN_PROGRESS`; las completadas o canceladas no pertenecen a ese reporte.

En la Parte E del Lab 11 construiste una vista filtrada con un `struct OrderPreview` y documentaste sus reglas de orden. Hoy completarás el ordenamiento y añadirás una consulta por ID **en un programa dentro del repo de este lab**. Copia tu `delivery_view_practice.cpp` del Lab 11 como `delivery_report_practice.cpp`, actualiza su header y conserva el archivo anterior en su repo. Si tu implementación de sorting ya estaba completa, úsala y dedica ese tiempo a las pruebas.

`OrderPreview` conserva `int orderId`, `string orderType`, `string status`, `string building` e `int priority`. En esta prueba, una prioridad 3 aparece en el reporte antes que una prioridad 1 del mismo edificio. No implementes otra queue: el array original permitirá verificar que producir el reporte no altera su secuencia.

Usa estos seis registros dummy en el orden de la tabla. Los nombres de edificios se escriben sin acentos en los datos para que la comparación lexicográfica de `string` sea reproducible. Una comparación lexicográfica compara el texto carácter por carácter; usa la misma escritura en todos los registros.

| `orderId` | `orderType` | `status` | `building` | `priority` |
| ---: | --- | --- | --- | ---: |
| 420 | DELIVERY | PENDING | Biblioteca | 1 |
| 180 | DELIVERY | IN_PROGRESS | Ingenieria | 3 |
| 650 | DINE_IN | PENDING | Cafeteria-Lunch | 1 |
| 310 | DELIVERY | PENDING | Biblioteca | 3 |
| 205 | DELIVERY | PENDING | Biblioteca | 3 |
| 900 | DELIVERY | COMPLETED | Ingenieria | 2 |

### E2 — Implementar el reporte y la consulta

Antes de programar, identifica qué comparación decide cuando los edificios difieren, cuándo necesitas consultar la prioridad y cuándo el ID resuelve un empate. Los IDs son únicos; no se exige que quicksort sea estable para obtener este orden completo porque el último criterio distingue órdenes diferentes.

1. **E1.** Construye la vista filtrada de órdenes activas `DELIVERY`, conservando su cantidad lógica. Mantén los objetos en `orders`; la vista solo toma prestadas sus direcciones.
2. **E2.** Implementa el comparador del reporte: edificio ascendente; dentro del mismo edificio, prioridad descendente; si ambos coinciden, ID ascendente. Utilízalo para ordenar la vista con tu quicksort de C adaptado a `OrderPreview`. Si ya completaste el merge sort de esta vista en Lab 11, puedes conservarlo y justificar esa elección.
3. **E3.** Solicita por consola un ID y busca **linealmente en la vista ordenada**. Para organizar el código, define `findOrderInView`, que recibe la vista, su cantidad y el ID solicitado, y devuelve una posición de la vista o `-1`. `main` la llama y comprueba el resultado antes de mostrar el edificio, estado y prioridad. La función no modifica datos.
4. **E4.** Muestra `orders` antes y después de producir el reporte y realizar la consulta. Comprueba que ningún campo ni posición original cambió; no ejecutes `delete` sobre los pointers prestados.

¿Por qué la consulta E3 es lineal? El reporte tiene que conservar su orden por edificio y prioridad, y ese orden no garantiza IDs ascendentes en toda la vista. No sería válido descartar la mitad de los registros comparando únicamente un ID central. Crear una segunda vista ordenada por ID sería otra solución posible, con trabajo y memoria adicionales; hoy basta recorrer la vista que ya necesita el reporte.

### E3 — Ejecutar y documentar la evidencia

Compila el programa de aplicación:

```bash
g++ delivery_report_practice.cpp -o delivery_report_practice
```

Ejecuta por separado:

```bash
./delivery_report_practice
```

El reporte debe contener cuatro órdenes. Comprueba sus tres reglas por separado y verifica que los dos registros excluidos no aparezcan. Realiza una consulta que encuentre una orden, otra por una orden excluida y otra por un ID inexistente. Guarda output representativo de las pruebas en `delivery_report_output.txt`, incluyendo el original antes y después y la vista del reporte. La evidencia puede copiarse desde la terminal.

Crea `project_application.md` dentro del mismo repo con esta estructura. Puedes escribir sus respuestas en español o inglés; reemplaza cada indicación por tu explicación y tus resultados:

```md
# Lab 12 - Project Application

## E1 - Filter and ownership
Identify selected and excluded IDs, their reasons, and who owns each object.

## E2 - Report order and algorithm
List the observed report IDs and explain how each comparison rule affects them.
State the sorting algorithm used and its time and auxiliary-space bounds.

## E3 - Lookup choice and tests
Explain why this report cannot support binary search by orderId as currently sorted.
Give one found, one excluded, and one nonexistent test with observed results.
Describe what an additional ID-sorted view would cost and enable.

## E4 - Evidence and project connection
Source: delivery_report_practice.cpp
Output: delivery_report_output.txt
Explain how you verified that original records kept their order and field values.
Explain how this separation protects the operational FIFO sequence in the project.
```

💾 **Buen momento para el commit:** completaste el reporte temporal y documentaste por qué su consulta requiere una estrategia diferente de la de paquetes. Usa `"Complete delivery report sorting and ID lookup practice"`.

## Parte F — Revisión, evaluación y entrega (15 min)

Las Partes B–E ya conectaron ordenamiento, condiciones de búsqueda y conservación de datos. Dedica cinco minutos a verificar que la entrega demuestra esas tres cosas. Los diez minutos restantes quedan disponibles para depurar, aclarar trazas y terminar la publicación del repo.

### F1 — Revisión final

- B2 y B3 están activos; el catálogo encuentra 57 en `C-2` y maneja la ausencia y la entrada inválida.
- B4–B5 están respondidas al final de `equipment_catalog_example.cpp`.
- Los TODO C–D están completos; el resultado ya no depende de los retornos provisionales.
- Solo los seis paquetes listos aparecen en la vista, ordenados por código y con su locker correcto.
- Los registros originales permanecen en sus posiciones con todos sus campos intactos.
- C3–C4 y D3–D4 están respondidas al final de `parcel_pickup_view.cpp`, con las pruebas adicionales solicitadas.
- No se accede a una posición fuera de la cantidad lógica ni se usa `-1` como índice.
- E contiene el reporte implementado, consulta por ID, evidencia y explicación de sus criterios.
- Cada `.cpp` tiene nombre, fecha y descripción; el código y sus mensajes están en inglés.
- `.gitignore` mantiene ejecutables fuera del historial; los análisis pueden estar en inglés o español.

💾 **Buen momento para el commit:** si esta revisión requiere correcciones, guárdalas con un mensaje que identifique el comportamiento corregido.

### F2 — Criterios de evaluación (100 puntos)

| Criterio | Puntos |
| --- | ---: |
| Organización, headers, `.gitignore` y commits | 5 |
| Ejemplo guiado funcional y análisis B4–B5 | 15 |
| Partición y quicksort de pointers; análisis C3–C4 | 25 |
| Binary search iterativa, consultas y análisis D3–D4 | 20 |
| Casos límite, cantidad lógica y conservación de registros | 10 |
| Reporte de delivery, consulta adecuada, explicación y evidencia | 25 |

## Entregable del laboratorio

Entrega el enlace del repo `lab12-quicksort-searching`, con:

- `equipment_catalog_example.cpp`, con todos los pasos activos y B4–B5 respondidas en comentarios al final.
- `parcel_pickup_view.cpp`, con C–D completos y C3–C4/D3–D4 respondidas en comentarios al final, incluidas las pruebas de rangos y cantidades límite.
- `delivery_report_practice.cpp`, `delivery_report_output.txt` y `project_application.md`, correspondientes a E.
- `.gitignore` funcionando y al menos **cinco commits**: setup, ejemplo guiado, quicksort, búsqueda y aplicación al proyecto.

## Próxima sesión

En la Semana 13 implementaremos inserción, búsqueda y recorridos de Binary Search Trees. Retomaremos la distinción presentada en la lecture de Semana 12: un BST agrega una regla de orden a la forma de un binary tree, y el costo de buscar depende de su altura. El próximo lab incluirá una práctica breve de árboles y repaso para el Examen 2.
