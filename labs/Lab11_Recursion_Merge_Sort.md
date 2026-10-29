# Lab 11 — Recursion, Merge Sort, and Project Application
## COEN 2220 — Programming 2

**Instructor:** Wilson Lozano

**Duración:** 110 min (sesión de 2 horas crédito nominales)  
**Precede a:** Semana 12 — Quicksort, searching e introducción a Binary Trees  
**Requisitos:** Lecture de Semana 11; Labs 1–10 completados  
**Basado en:** Gaddis, *Starting Out with C++*, 8va ed., Cap. 19 — secciones 19.1–19.2 y 19.10 para recursión; Cap. 17 para nodes y traversal. Merge sort es contenido complementario de la lecture de Semana 11; no se atribuye al capítulo 19.

---

## Objetivos

Al finalizar este laboratorio, el estudiante podrá:

1. Identificar el caso base y el avance de una función recursiva que procesa una linked list.
2. Explicar cómo merge sort separa una lista, ordena sus mitades y combina los nodes al regresar.
3. Definir un criterio de ordenamiento con dos campos y una regla explícita para los empates.
4. Implementar merge sort sobre un array temporal de pointers, conservando los registros originales y su orden de almacenamiento.
5. Distinguir el tiempo de ordenamiento del espacio auxiliar de las implementaciones con nodes y con arrays.
6. Construir y probar una vista temporal de datos del proyecto sin modificar sus estructuras operacionales.

**Distribución:** A: 5 min; B: 30 min; C: 15 min; D: 25 min; E: 20 min; F: 15 min. Total: **110 min**. Los últimos 10 minutos de F quedan reservados para depuración, dudas y cierre.

> **Alcance:** el ejemplo guiado está completamente resuelto; las Partes C y D son práctica TODO. La Parte E prepara una técnica para el proyecto mediante un programa independiente dentro del mismo repositorio del lab. No uses STL para almacenar u ordenar colecciones, `std::sort`, exceptions ni datos personales reales.

## Parte A — Preparar la carpeta y el repositorio (5 min)

> ⚠️ **Este paso es el que más se presta a error — léelo con cuidado antes de hacer clic en nada.** Cada lab tiene su propio repositorio. No inicialices Git en la carpeta padre `COEN2220` ni dentro de la carpeta de otro lab.

1. **A1.** Abre VS Code o VSCodium y selecciona `File → Open Folder...`.
2. **A2.** Navega a tu carpeta padre del curso, por ejemplo `COEN2220`. Si todavía no existe, créala **desde este diálogo** en un lugar de fácil acceso.
3. **A3.** Sin salir del diálogo, crea dentro una carpeta llamada `lab11-recursion-merge-sort`, hermana de las carpetas de los labs anteriores.
4. **A4.** Abre específicamente `lab11-recursion-merge-sort` con **Open** / **Select Folder**.

**Cómo saber si abriste la carpeta correcta:** el Explorer debe mostrar `LAB11-RECURSION-MERGE-SORT` como raíz. Si muestra `COEN2220` u otro lab, usa `File → Close Folder` y repite los pasos.

> **¿Por qué importa tanto esto?** Inicializar Git en la carpeta padre mezcla todos los labs en un historial. Hacerlo dentro de un lab anterior mezcla archivos y entregas. Cada práctica debe poder revisarse como una unidad independiente.

Abre Source Control (`Ctrl+Shift+G` / `Cmd+Shift+G`) y selecciona **Initialize Repository** para iniciar el historial de esta carpeta. Crea `.gitignore` con:

```gitignore
# Compiled executables (Windows)
*.exe

# Compiled executables (Mac/Linux)
book_repair_list_example
repair_task_view
delivery_view_practice

# Intermediate object files
*.o

# Editor configuration
.vscode/
```

Haz stage → commit → **Publish Branch**, con el flujo ya practicado. Mensaje: `"Initial setup and gitignore"`.

💾 **Por qué este commit importa:** conserva la estructura inicial y evita confundir problemas de organización del repo con cambios posteriores del programa.

## Parte B — Ejemplo guiado: contar trabajo y ordenar una lista de reparaciones (30 min)

### Contexto: qué necesita la biblioteca

El taller de conservación de una biblioteca recibe libros con páginas dañadas. Cada libro recibe un código dummy, como `BK-101`, y se registra cuántas páginas necesitan reparación. El personal necesita conocer el total de páginas pendientes y preparar un listado de menor a mayor cantidad de páginas para estimar qué trabajos son pequeños y cuáles son grandes.

La cantidad de libros recibidos varía. Para almacenarlos, este programa declarará una clase C++ llamada `BookRepairList`, que representa **la colección completa de reparaciones**. Su tipo privado `Node` representa **un libro registrado**: contiene su código, las páginas pendientes y el enlace `next`. El data member `head` conserva la dirección del primer node.

En esta colección sí se permite cambiar el orden de los nodes para producir el listado. Si dos libros tienen igual cantidad de páginas, deben conservar su orden original entre sí. Esta es una condición del ejemplo de biblioteca; en la Parte C conocerás otro escenario que exige conservar intacto el orden de la colección original.

La clase es dueña de los nodes que crea con `new` y los libera en su destructor. Como en la Semana 9, no copies objetos `BookRepairList` mediante assignment ni los pases por valor: esta versión no implementa copias independientes de sus nodes.

Primero observaremos la lista inicial. Después calcularemos el total mediante recursión y finalmente ordenaremos sus nodes con merge sort. Así podremos verificar que ordenar cambia los enlaces, pero conserva todos los libros y el total de páginas.

### B1 — Crear y mostrar los datos antes de procesarlos

Crea `book_repair_list_example.cpp` con este starter code. Todos los métodos están definidos **dentro de la clase** y por eso no llevan `BookRepairList::`. El constructor, `addBook`, `display` y destructor están activos desde el comienzo. Los métodos y pruebas de B2 y B3 están resueltos, pero inactivos entre comentarios `/*` y `*/`.

Cada etiqueta `// --- STEP B2 ... ---` o `// --- STEP B3 ... ---` permanece fuera del comentario. Al llegar a ese paso, quitarás solo los comentarios que rodean sus bloques y conservarás las etiquetas.

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 11 - Recursion and Merge Sort
 * Description: Guided example - recursive total and linked-list merge sort
 * Due date: [Date]
 */

#include <iostream>
#include <string>
using namespace std;

class BookRepairList
{
private:
    struct Node
    {
        string bookCode; // Keep the book identity attached to its page count.
        int pages;       // Store the number of pages still needing repair.
        Node *next;      // Link to the next registered book.
    };

    Node *head; // Remember the first node of the collection.

    // --- STEP B2: Sum pages recursively ---
    /*
    int totalFrom(Node *current) const
    {
        if (current == nullptr)
        {
            return 0; // Base case: no remaining books contribute zero pages.
        }

        int laterPages = totalFrom(current->next); // Solve a shorter remaining chain.
        return current->pages + laterPages; // Finish this call after the next returns.
    }
    */

    // --- STEP B3: Split, recursively sort, and merge existing nodes ---
    /*
    Node *splitList(Node *firstNode)
    {
        Node *slow = firstNode;
        Node *fast = firstNode;
        while (fast->next != nullptr && fast->next->next != nullptr)
        {
            slow = slow->next;       // Find the final node of the left half.
            fast = fast->next->next;
        }

        Node *rightHead = slow->next; // Keep the right half before cutting its link.
        slow->next = nullptr;         // Both halves now end independently.
        return rightHead;
    }

    Node *mergeLists(Node *left, Node *right)
    {
        if (left == nullptr)
        {
            return right;
        }
        if (right == nullptr)
        {
            return left;
        }

        Node *result;
        if (left->pages <= right->pages)
        {
            result = left; // Prefer the left node on ties to preserve arrival order.
            left = left->next;
        }
        else
        {
            result = right;
            right = right->next;
        }

        Node *tail = result; // Track the last node already selected.
        while (left != nullptr && right != nullptr)
        {
            if (left->pages <= right->pages)
            {
                tail->next = left;
                left = left->next; // Preserve access to the unused left chain.
            }
            else
            {
                tail->next = right;
                right = right->next;
            }
            tail = tail->next;
        }

        if (left != nullptr)
        {
            tail->next = left;
        }
        else
        {
            tail->next = right; // Attach the entire chain that has not been used.
        }
        return result;
    }

    Node *sortNodes(Node *firstNode)
    {
        if (firstNode == nullptr || firstNode->next == nullptr)
        {
            return firstNode; // Zero or one node is already sorted.
        }

        Node *rightHead = splitList(firstNode);
        Node *leftSorted = sortNodes(firstNode); // Finish the left half first.
        Node *rightSorted = sortNodes(rightHead);
        return mergeLists(leftSorted, rightSorted); // Both inputs are now sorted.
    }
    */

public:
    BookRepairList()
    {
        head = nullptr;
    }

    ~BookRepairList()
    {
        while (head != nullptr)
        {
            Node *nodeToDelete = head;
            head = head->next; // Preserve the next address before releasing this node.
            delete nodeToDelete;
        }
    }

    void addBook(const string& newBookCode, int newPages)
    {
        Node *newNode = new Node;
        newNode->bookCode = newBookCode;
        newNode->pages = newPages;
        newNode->next = nullptr;
        if (head == nullptr)
        {
            head = newNode;
            return;
        }

        Node *current = head;
        while (current->next != nullptr)
        {
            current = current->next;
        }
        current->next = newNode; // Register books in their original arrival order.
    }

    void display() const
    {
        Node *current = head;
        while (current != nullptr)
        {
            cout << "Book: " << current->bookCode
                 << " | Pages: " << current->pages << endl;
            current = current->next;
        }
    }

    // --- STEP B2: Offer a public operation without exposing Node ---
    /*
    int totalPages() const
    {
        return totalFrom(head); // Start the recursive traversal at the first book.
    }
    */

    // --- STEP B3: Preserve the new entry point after sorting ---
    /*
    void sortByPages()
    {
        head = sortNodes(head); // The smallest book may become the new first node.
    }
    */
};

int main()
{
    BookRepairList repairs;
    repairs.addBook("BK-101", 12);
    repairs.addBook("BK-102", 4);
    repairs.addBook("BK-103", 9);
    repairs.addBook("BK-104", 2);
    repairs.addBook("BK-105", 7);
    repairs.addBook("BK-106", 4);
    repairs.addBook("BK-107", 10);
    repairs.addBook("BK-108", 3);

    cout << "Arrival order:" << endl;
    repairs.display();

    // --- STEP B2: Observe the recursive total ---
    /*
    cout << "Total pages before sorting: " << repairs.totalPages() << endl;
    */

    // --- STEP B3: Observe the sorted list and verify the total ---
    /*
    repairs.sortByPages();
    cout << "Sorted by pages:" << endl;
    repairs.display();
    cout << "Total pages after sorting: " << repairs.totalPages() << endl;
    */

    return 0;
}

/*
 * Analysis Answers
 *
 * B4 - Recursive total
 * For the eight-book list, how many totalFrom calls occur, including the base case?
 * What value does the call for BK-108 return, and where does execution resume next?
 * Answer:
 *
 * B5 - Splitting and returning
 * Show the two-node sublist BK-101:12 -> BK-102:4, its base cases, and its merged
 * result. Which node address is returned? Why must sortByPages update head?
 * Answer:
 *
 * B6 - Time and auxiliary space
 * Compare totalPages and sortByPages. Give the time and auxiliary-space order of
 * each operation, including pending calls. Do not count list construction.
 * Answer:
 *
 */
```

Compila y ejecuta en cada paso:

```bash
g++ book_repair_list_example.cpp -o book_repair_list_example
./book_repair_list_example
```

**Qué deberías ver en B1:**

```text
Arrival order:
Book: BK-101 | Pages: 12
Book: BK-102 | Pages: 4
Book: BK-103 | Pages: 9
Book: BK-104 | Pages: 2
Book: BK-105 | Pages: 7
Book: BK-106 | Pages: 4
Book: BK-107 | Pages: 10
Book: BK-108 | Pages: 3
```

<details>
<summary>Verificación: ¿por qué cada código debe seguir asociado a su cantidad de páginas después de ordenar?</summary>

El código identifica el libro que necesita esa reparación. Cambiar solo las cantidades entre nodes sin preservar la asociación produciría un listado incorrecto. En este ejemplo se conservarán los datos de cada node y se modificarán sus enlaces.

</details>

### B2 — Recorrer mediante llamadas y terminar en `nullptr`

Quita los comentarios `/*` y `*/` de los **tres bloques `STEP B2`**: el helper privado `totalFrom`, el método público `totalPages` y su prueba en `main`. Conserva B3 comentado.

`totalPages` permite que el código cliente pida un total sin conocer `Node`. Inicia el trabajo en `head`. Cada llamada a `totalFrom` recibe un pointer local a un node, solicita el total de los que siguen y conserva pendiente su suma. El avance es `current->next`; el caso base no es la última página ni un número cero, sino la ausencia de otro node: `current == nullptr`.

En el final de la cadena, el proceso es:

```text
totalFrom(BK-108:3)
    totalFrom(nullptr) -> return 0
    resume at BK-108: return 3 + 0 -> 3
resume at BK-107: return 10 + 3 -> 13
```

Las llamadas anteriores siguen regresando del mismo modo. Mover un parámetro pointer local no cambia `head` ni los enlaces. Compila y ejecuta: el output de B1 permanece y se añade:

```text
Total pages before sorting: 51
```

<details>
<summary>Verificación: ¿por qué se evalúa el caso base antes de consultar <code>current->pages</code>?</summary>

La llamada final recibe `nullptr`. No existe allí un node cuyo member pueda leerse; primero se detecta ese estado y se devuelve cero. Las llamadas que reciben nodes válidos pueden consultar sus páginas y pedir el total de la cadena siguiente.

</details>

### B3 — Ordenar al regresar de las mitades

Quita los comentarios de los **tres bloques `STEP B3`**: helpers privados, método público y prueba en `main`. Mantén activos los bloques B2, porque la prueba final vuelve a usar `totalPages`.

El pedido público comienza en `sortByPages`. El helper `sortNodes` recibe la primera dirección de una sublista. Si tiene cero o un node, devuelve esa dirección inmediatamente. De lo contrario, `splitList` encuentra la mitad con `slow` y `fast`, guarda la dirección de la derecha y corta un `next` para que las mitades no sigan conectadas.

Luego **`sortNodes` se llama a sí mismo** sobre la izquierda. Cuando esa llamada termina, solicita ordenar la derecha. Solo al recibir ambas listas ordenadas ejecuta `mergeLists`. Este último usa un loop: selecciona el próximo node por páginas, reconecta el `next` de `tail` y conserva el acceso a los pendientes con `left` y `right`.

Esta traza sigue la primera rama y el primer merge; los códigos conservan la identidad de los nodes:

```text
sortNodes(BK-101:12 -> BK-102:4 -> BK-103:9 -> BK-104:2 -> ...)
    sortNodes(BK-101:12 -> BK-102:4 -> BK-103:9 -> BK-104:2)
        sortNodes(BK-101:12 -> BK-102:4)
            sortNodes(BK-101:12) -> return BK-101 (one node)
            sortNodes(BK-102:4)  -> return BK-102 (one node)
        mergeLists(BK-101:12, BK-102:4)
            -> BK-102:4 -> BK-101:12 -> nullptr
```

La sublista de dos nodes quedó ordenada al regresar; no lo estaba simplemente por haber sido separada. Las demás ramas repiten ese proceso hasta completar los ocho libros. Compila y ejecuta. Después del total de B2 debe aparecer:

```text
Sorted by pages:
Book: BK-104 | Pages: 2
Book: BK-108 | Pages: 3
Book: BK-102 | Pages: 4
Book: BK-106 | Pages: 4
Book: BK-105 | Pages: 7
Book: BK-103 | Pages: 9
Book: BK-107 | Pages: 10
Book: BK-101 | Pages: 12
Total pages after sorting: 51
```

<details>
<summary>Verificación: ¿qué demuestra que BK-102 siga antes de BK-106, y qué no demuestra por sí solo que el total siga siendo 51?</summary>

Ambos tienen cuatro páginas y conservan su orden original: es evidencia de estabilidad. El total sin cambios es una comprobación útil, pero por sí solo no demuestra que se conservaron todos los libros correctamente; también debemos revisar códigos, cantidades y número de registros.

</details>

### B4 — Explicar lo observado

Responde B4–B6 en el bloque `Analysis Answers` al final del mismo `.cpp`, en inglés. Usa los nombres de los nodes y las llamadas del programa para justificar. Recuerda: **tiempo** mide cómo crece el trabajo; **espacio auxiliar** mide la memoria adicional simultánea, incluidas las llamadas pendientes. Los nodes originales no se cuentan como espacio auxiliar del ordenamiento.

💾 **Buen momento para el commit** — el ejemplo ya demuestra recursión lineal, recursión por mitades y combinación por enlaces, con evidencia de ejecución. Commit: `"Complete recursive book repair example"`.

## Parte C — TODO: decidir el orden de una vista temporal de tareas (15 min)

### Contexto: ordenar un reporte sin alterar el registro de llegada

El mismo taller también registra tareas de conservación, como preparar una encuadernación o reparar una cubierta. Cada tarea tiene un ID, los días que faltan para su fecha límite y una estimación de minutos de trabajo. El supervisor quiere un reporte para planificar el día, pero debe conservar intacto el registro de llegada para auditoría.

El programa declarará un `struct` llamado `RepairTask` para representar **una tarea**, con `taskId`, `daysUntilDue` y `minutesNeeded`. El array `tasks` conservará los ocho registros en orden de llegada. El reporte tendrá estas reglas:

1. Menos días hasta la fecha límite primero: una tarea con un día restante aparece antes que otra con tres.
2. Si faltan los mismos días, menos minutos de trabajo primero, para identificar los trabajos breves de ese grupo.
3. Si coinciden ambos campos, conservar el orden de llegada. No usar el ID como tercer criterio.

Para satisfacer esas reglas sin cambiar `tasks`, crearemos una **vista temporal**: un array llamado `view` cuyas posiciones guardan direcciones de las tareas existentes. Ordenar la vista cambia la secuencia de esas direcciones; los registros originales permanecen en sus posiciones.

```text
Original records:   tasks[0]        tasks[1]        tasks[2] ...
                    RT-101          RT-102          RT-103

Before sorting:     view[0] -> tasks[0]
                    view[1] -> tasks[1]

After sorting:      each view position may point to a different original record.
                    tasks keeps its original records and arrival order.
```

`RepairTask *view[]` significa “array cuyos elementos son pointers a `RepairTask`”. `&tasks[index]` obtiene la dirección de un registro. Como `view[index]` es un pointer, se consulta un member con `->`, por ejemplo `view[index]->daysUntilDue`. La asignación entre posiciones de `view` copia una dirección; no copia ni modifica la tarea señalada. La vista no es dueña de esas tareas y no debe ejecutar `delete` sobre sus pointers.

Antes de ordenar, necesitas expresar el criterio en una función llamada `comesBeforeOrEqual`. Recibirá dos pointers a tareas y devolverá `true` si la izquierda debe aparecer antes que la derecha **o si ambas empatan en los dos campos**. Esa regla de empate permitirá que el merge elija la izquierda y conserve estabilidad.

En sus parámetros, `const RepairTask *` permite consultar los campos de la tarea señalada, pero impide modificarlos mediante ese pointer. Comparar dos tareas debe decidir su orden sin cambiar sus datos. Esta función supone que ambos pointers señalan tareas válidas; el merge solo la llamará mientras haya un elemento pendiente en cada mitad.

### C1 — Razonar antes de implementar

1. **C1.** Si la izquierda tiene un día restante y la derecha tres, ¿debe decidir el resultado la duración de la tarea?
2. **C2.** Si ambas tienen dos días restantes, ¿qué campo debe resolver el orden?
3. **C3.** Si coinciden ambos campos, ¿qué resultado debe dar `comesBeforeOrEqual` para permitir que el merge conserve la izquierda?
4. **C4.** ¿Por qué cambiar una dirección de `view` no mueve el registro dentro de `tasks`?

Crea `repair_task_view.cpp` con este template. Resuelve **solo** los TODO `(Part C)` ahora; ignora los de `(Part D)`. La regla completa del comparador debe ser tu implementación. La línea provisional `return false` solo permite que el starter code compile: debes reemplazarla.

Las funciones son libres: los prototipos están antes de `main` y las definiciones después. Por eso no llevan un nombre de clase seguido de `::`. Los arrays `view` y `temporary` contienen pointers; las tareas señaladas siguen viviendo en `tasks` durante toda la ejecución.

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 11 - Recursion and Merge Sort
 * Description: Practice - sort a temporary view without changing original tasks
 * Due date: [Date]
 */

#include <iostream>
#include <string>
using namespace std;

struct RepairTask
{
    string taskId;     // Preserve the identity of the original task.
    int daysUntilDue;  // Smaller values must appear first in the report.
    int minutesNeeded; // Break equal-deadline ties with smaller durations first.
};

bool comesBeforeOrEqual(const RepairTask *left, const RepairTask *right);
void mergeTaskView(RepairTask *view[], RepairTask *temporary[],
                   int first, int middle, int last);
void mergeSortView(RepairTask *view[], RepairTask *temporary[], int first, int last);
void displayOriginal(const RepairTask tasks[], int count);
void displayView(RepairTask *view[], int count);

int main()
{
    const int TASK_COUNT = 8;
    RepairTask tasks[TASK_COUNT] =
    {
        {"RT-101", 3, 30}, {"RT-102", 1, 20},
        {"RT-103", 2, 15}, {"RT-104", 1, 10},
        {"RT-105", 2, 15}, {"RT-106", 3, 10},
        {"RT-107", 1, 20}, {"RT-108", 2, 5}
    };
    RepairTask *view[TASK_COUNT];      // Borrow addresses of existing tasks.
    RepairTask *temporary[TASK_COUNT]; // Hold addresses while merging the view.
    for (int index = 0; index < TASK_COUNT; index++)
    {
        view[index] = &tasks[index]; // Preserve arrival order in the initial view.
    }

    cout << boolalpha;
    cout << "Original arrival order:" << endl;
    displayOriginal(tasks, TASK_COUNT);

    // ===== Resolve these TODOs now (Part C) =====
    // TODO (Part C): Activate the C test block after completing the comparator.
    /*
    cout << "Earlier deadline first: "
         << comesBeforeOrEqual(&tasks[1], &tasks[0]) << endl;
    cout << "Shorter duration within same deadline: "
         << comesBeforeOrEqual(&tasks[3], &tasks[1]) << endl;
    cout << "Exact tie keeps left: "
         << comesBeforeOrEqual(&tasks[2], &tasks[4]) << endl;
    cout << "Later deadline first: "
         << comesBeforeOrEqual(&tasks[0], &tasks[1]) << endl;
    */

    // ===== Resolve these TODOs later (Part D) =====
    // TODO (Part D): Call mergeSortView for the complete logical range.
    // TODO (Part D): Display the sorted view with a descriptive heading.
    // TODO (Part D): Display tasks again to prove that arrival order did not change.

    return 0;
}

bool comesBeforeOrEqual(const RepairTask *left, const RepairTask *right)
{
    // ===== Resolve this TODO now (Part C) =====
    // TODO (Part C): Implement the specified two-field order, including exact ties.
    return false; // Temporary placeholder; replace this line in Part C.
}

// ===== Resolve these TODOs later (Part D) =====
void mergeSortView(RepairTask *view[], RepairTask *temporary[], int first, int last)
{
    // TODO (Part D): Implement recursive merge sort using inclusive indices.
}

void mergeTaskView(RepairTask *view[], RepairTask *temporary[],
                   int first, int middle, int last)
{
    // TODO (Part D): Merge two sorted segments using comesBeforeOrEqual.
    // TODO (Part D): Preserve pending pointers and copy the merged segment back.
}

void displayOriginal(const RepairTask tasks[], int count)
{
    for (int index = 0; index < count; index++)
    {
        cout << "Task: " << tasks[index].taskId
             << " | Days until due: " << tasks[index].daysUntilDue
             << " | Minutes: " << tasks[index].minutesNeeded << endl;
    }
}

void displayView(RepairTask *view[], int count)
{
    for (int index = 0; index < count; index++)
    {
        cout << "Task: " << view[index]->taskId
             << " | Days until due: " << view[index]->daysUntilDue
             << " | Minutes: " << view[index]->minutesNeeded << endl;
    }
}

/*
 * Analysis Answers
 *
 * C5 - Two-field decision
 * Explain the result for RT-103 compared with RT-108. Which field decides it?
 * Answer:
 *
 * D1 - Recursive ranges
 * Show first, last, middle, and the base-case evaluations along the first branch
 * starting at 0..7. Identify where execution resumes after the call for 0..0.
 * Answer:
 *
 * D2 - Stable ties
 * Identify both pairs of tasks that tie in the two sort fields. Explain how the
 * comparator and the merge preserve their arrival order.
 * Answer:
 *
 * D3 - Time, space, and original data
 * Give the sorting time and auxiliary-space order of this pointer-array version.
 * Compare with the linked-list sort in Part B and explain why the original task
 * storage order is preserved here but the node order changes in Part B.
 * Answer:
 *
 */
```

### C2 — Probar decisiones antes de ordenar

Después de completar el comparador y activar su bloque en `main`, compila y ejecuta:

```bash
g++ repair_task_view.cpp -o repair_task_view
./repair_task_view
```

El listado inicial debe conservar los ocho IDs en el orden del template. Después debe aparecer:

```text
Earlier deadline first: true
Shorter duration within same deadline: true
Exact tie keeps left: true
Later deadline first: false
```

El resto del programa todavía no ordena: los cuerpos de D están vacíos y `main` aún no los llama. Es válido que el compilador advierta sobre parámetros o variables todavía sin usar durante esta etapa. Responde C5 en el bloque final de comentarios, en inglés.

<details>
<summary>Ver orientación sobre el orden de las decisiones</summary>

Primero determina si los días son diferentes. Solo cuando coinciden tiene sentido usar los minutos para resolver el empate. Revisa también el caso de igualdad completa: el merge necesita distinguir una preferencia por la derecha de un empate que debe conservar la izquierda. No uses el ID para resolver ese empate.

</details>

💾 **Buen momento para el commit** — probaste el criterio por separado antes de mezclarlo con la recursión y los movimientos del merge. Commit: `"Define and test repair task comparison"`.

## Parte D — TODO: ordenar pointers y comprobar los registros (25 min)

Continúa en `repair_task_view.cpp`. Resuelve **solo** los TODO `(Part D)` ahora; conserva la solución de C. Los helpers ya están declarados y ubicados: debes completar sus cuerpos y las llamadas de `main`, no crear otra clase ni mover funciones al interior de `main`.

`mergeSortView` debe ordenar el segmento inclusivo `first..last`. `mergeTaskView` recibirá dos segmentos ya ordenados, `first..middle` y `middle + 1..last`, y deberá combinarlos según `comesBeforeOrEqual`. El auxiliar almacena las direcciones seleccionadas y después esas direcciones vuelven al segmento de `view`. Durante el proceso no deben cambiar campos de `RepairTask`, posiciones de `tasks` ni objetos señalados.

### D1 — Pensar antes de modificar

1. **D1.** ¿Qué relación entre `first` y `last` identifica un segmento que ya no necesita dividirse?
2. **D2.** ¿Qué límites recibe la llamada izquierda y cuáles recibe la derecha? ¿Por qué no empiezan ambas en `middle`?
3. **D3.** ¿En qué momento se garantiza que los dos segmentos entregados a `mergeTaskView` están ordenados?
4. **D4.** Al elegir entre `view[leftIndex]` y `view[rightIndex]`, ¿qué tipo de dato se copia al auxiliar: una tarea o su dirección?
5. **D5.** Si una mitad se agota, ¿qué debe pasar con las direcciones pendientes de la otra?

Puedes consultar las Partes 2–4 de la lecture para los rangos y el merge. La diferencia de esta práctica está en el **tipo de elemento que ordenas** y en el **criterio de dos campos**; no copies la comparación por minutos del ejemplo de calibración.

### D2 — Ejecutar y verificar el reporte

Una vez implementados los helpers y las llamadas de `main`, el reporte ordenado debe contener exactamente estas líneas:

```text
Task: RT-104 | Days until due: 1 | Minutes: 10
Task: RT-102 | Days until due: 1 | Minutes: 20
Task: RT-107 | Days until due: 1 | Minutes: 20
Task: RT-108 | Days until due: 2 | Minutes: 5
Task: RT-103 | Days until due: 2 | Minutes: 15
Task: RT-105 | Days until due: 2 | Minutes: 15
Task: RT-106 | Days until due: 3 | Minutes: 10
Task: RT-101 | Days until due: 3 | Minutes: 30
```

Al volver a mostrar `tasks`, sus IDs deben seguir en el orden `RT-101` a `RT-108` y conservar sus campos originales. Ejecuta también la llamada de ordenamiento con un rango lógico vacío `0..-1` y con un solo elemento `0..0`, usando los arrays de capacidad ocho ya declarados. No declares arrays de tamaño cero. Repite el ordenamiento completo sobre la vista ya ordenada y comprueba que el resultado permanece igual.

<details>
<summary>Verificación: si la vista queda ordenada pero RT-101 cambia de posición dentro de <code>tasks</code>, ¿se cumplió el requisito?</summary>

No. El requisito incluye preservar el almacenamiento y el orden de los registros originales. La vista debe modificar únicamente sus pointers. Una secuencia final ascendente no demuestra por sí sola que el programa conservó correctamente los datos originales.

</details>

Responde D1–D3 en el bloque `Analysis Answers`, en inglés. Identifica qué representa `n`, separa el tiempo del espacio auxiliar y cuenta las llamadas activas cuando corresponda. No respondas solamente con una expresión Big-O.

💾 **Buen momento para el commit** — ya existe una vista ordenada y probada que conserva la colección original. Commit: `"Sort and verify temporary repair task view"`.

## Parte E — Aplicación al proyecto: construir una vista sin alterar las órdenes (20 min)

### Contexto: por qué la técnica importa en la cafetería

El proyecto de cafetería tiene dos necesidades diferentes: preparar órdenes en su secuencia FIFO y mostrar un reporte temporal de órdenes activas de `DELIVERY` organizado por edificio, prioridad e ID. Cambiar el orden de la queue para producir el reporte afectaría qué orden se prepara después. Cambiar la lista principal tampoco cumple el requisito del reporte temporal.

Por eso el proyecto pide una **vista o arreglo temporal de pointers a `Order`**. Es la misma separación que acabas de practicar entre `tasks` y `view`: el reporte conserva direcciones hacia objetos existentes y puede reorganizar esas direcciones sin reorganizar la colección operacional. Aunque la lista enlazada de la Parte B se pueda ordenar, reconectar la lista principal de órdenes no corresponde al requisito de este reporte.

### E1 — Preparar una prueba independiente dentro del lab

Continúa en `lab11-recursion-merge-sort` y crea `delivery_view_practice.cpp`. Este archivo será un programa independiente para practicar la construcción de la vista; no depende de que ya tengas implementadas las clases completas del proyecto.

Para representar las órdenes de prueba, define un `struct` llamado `OrderPreview`, con estos members: `int orderId`, `string orderType`, `string status`, `string building` e `int priority`. Es una representación reducida para este ejercicio: contiene los datos necesarios para seleccionar las órdenes y describir el criterio de reporte. Usa el header estándar de los otros archivos, con tu nombre, fecha y una descripción de esta práctica.

En `main`, un array `orders` conservará los cuatro registros dummy. Otro array, `view`, almacenará pointers de tipo `OrderPreview *` hacia las órdenes seleccionadas. Ambos pueden tener capacidad fija de cuatro posiciones, con una cantidad lógica para indicar cuántas direcciones válidas contiene la vista. Los objetos pertenecen al array `orders`; la vista solo conserva sus direcciones.

### E2 — Implementar y demostrar la vista

1. **E1.** Crea los cuatro registros dentro de `orders` en una secuencia conocida. Identifica qué array almacena las órdenes y cuál guarda solamente sus direcciones. Para construir el reporte, consulta los registros sin retirarlos de su colección original.
2. **E2.** Crea un array temporal de pointers con capacidad suficiente para las órdenes que puede contener tu prueba y lleva una cantidad lógica de elementos válidos. Copia a la vista únicamente direcciones de órdenes activas de tipo `DELIVERY`. Una orden activa es `PENDING` o `IN_PROGRESS`; una completada o cancelada no pertenece a esta vista.
3. **E3.** Usa por lo menos cuatro órdenes dummy: dos `DELIVERY` activas, una `DINE_IN` activa y una completada o cancelada. Muestra los IDs que quedaron en la vista para comprobar la selección. No clones los objetos ni transfieras su propiedad a la vista.
4. **E4.** Muestra los IDs y estados de `orders` antes y después de construir la vista. Comprueba que sus posiciones y datos no cambiaron. No necesitas implementar una queue en esta práctica: explica cómo preservar la colección original ayuda a mantener el orden FIFO cuando la técnica se integre al sistema.

La vista contiene pointers prestados: no debe liberar las órdenes. En esta prueba, los dos arrays viven en `main` y no requieren `new` ni `delete`. Utiliza solo las posiciones de `view` que indique su cantidad lógica; las posiciones restantes no contienen direcciones que debas consultar.

Compila y ejecuta:

```bash
g++ delivery_view_practice.cpp -o delivery_view_practice
./delivery_view_practice
```

### E3 — Preparar el criterio para el siguiente avance

En `project_application.md`, dentro del repo del lab, escribe las reglas que usará el comparador del reporte: edificio alfabético, prioridad descendente dentro del edificio y `orderId` ascendente en el empate. Ilustra las reglas con IDs y edificios dummy y anota dónde se conectará el ordenamiento con la vista.

El resultado exigido hoy es **la vista filtrada funcionando y probada** y el criterio documentado. Si ya la tienes, comienza a adaptar tu merge sort para ordenar sus pointers según esas tres reglas. La implementación completa de ese reporte podrá continuar en el siguiente lab; esta sección no añade otra estructura obligatoria ni cambia los requisitos del proyecto.

### E4 — Registrar la evidencia en el lab

Guarda el código en `delivery_view_practice.cpp` y el output de su ejecución en `delivery_view_output.txt`, ambos dentro del repo del lab. El output debe mostrar la colección original antes y después, además de los IDs seleccionados para la vista.

Completa `project_application.md` con esta estructura:

```md
# Lab 11 - Project Application

## E1 - Original collection and borrowed pointers
Describe orders, view, and which array stores the actual records.

## E2 - Implemented view and test
Describe the four dummy cases, the expected selected IDs, and the observed IDs.
Explain how you checked that original data and storage order were preserved.

## E3 - Sorting criterion and next integration step
State the three comparison rules and give a small example of their expected order.
Identify the remaining integration work, if any.

## E4 - Lab evidence
Source file: delivery_view_practice.cpp
Test output: delivery_view_output.txt
Explain how this practice prepares the project's temporary report.
```

Completa estas respuestas en inglés. La práctica, su evidencia y sus explicaciones forman parte de tu entrega del lab.

💾 **Buen momento para el commit** — construiste y probaste una vista filtrada, y documentaste cómo prepara el reporte del proyecto. Commit: `"Build and document delivery report view practice"`.

## Parte F — Revisión, evaluación y entrega (15 min)

Dedica los primeros cinco minutos a revisar esta lista. Los diez restantes se reservan para corregir problemas de compilación, aclarar trazas y completar la entrega.

- El ejemplo guiado compila en B1, B2 y B3; en la entrega final todos los bloques están activos.
- El total de páginas es 51 antes y después de ordenar; los ocho códigos siguen presentes y los dos libros de cuatro páginas conservan su orden.
- B4–B6 están respondidas al final de `book_repair_list_example.cpp`.
- `repair_task_view.cpp` implementa los TODO de C y D; el comparador provisional fue reemplazado.
- La vista respeta los dos campos y los empates; `tasks` conserva su orden y sus datos originales.
- Se probaron rangos vacío, de un elemento y una vista ya ordenada; los límites no causan accesos inválidos.
- C5 y D1–D3 están respondidas al final de `repair_task_view.cpp`.
- `delivery_view_practice.cpp` compila y `delivery_view_output.txt` muestra los casos de E; `project_application.md` explica selección, conservación de registros y criterio futuro.
- Los headers tienen nombre y fecha; identificadores, mensajes, comentarios y respuestas están en inglés.
- `.gitignore` mantiene ejecutables y archivos objeto fuera de Source Control.

💾 **Buen momento para el commit** — si corregiste detalles en esta revisión, guarda esas correcciones con un mensaje que identifique qué comprobación fallaba y qué quedó resuelto.

### Criterios de evaluación (100 puntos)

| Criterio | Puntos |
| --- | ---: |
| Setup, headers, `.gitignore` y commits claros | 5 |
| Ejemplo guiado ejecutable y análisis B4–B6 | 20 |
| Comparador de dos campos, prueba de C y respuesta C5 | 15 |
| Merge sort de pointers, estabilidad, integridad de datos y análisis D1–D3 | 25 |
| Pruebas de casos límite y repetición del ordenamiento | 10 |
| Práctica de vista de órdenes implementada y probada en el lab; criterio y evidencia documentados | 25 |

## Entregable del laboratorio

Entrega al profesor el enlace de tu repo `lab11-recursion-merge-sort`, con:

- `book_repair_list_example.cpp`, con B2–B3 activos y B4–B6 respondidas en comentarios al final.
- `repair_task_view.cpp`, con los TODO C–D resueltos y C5/D1–D3 respondidas en comentarios al final.
- `delivery_view_practice.cpp`, con la práctica de la Parte E, y `delivery_view_output.txt`, con su evidencia de ejecución.
- `project_application.md`, con E1–E4 completos sobre la práctica y su conexión con el proyecto.
- `.gitignore` funcionando, sin ejecutables ni archivos objeto.
- Al menos **cinco commits** en el repo del lab: setup, ejemplo guiado, comparador, ordenamiento y aplicación al proyecto.

## Próxima sesión

En la Semana 12 compararán merge sort con quicksort y retomarán searching. El campo usado para ordenar debe coincidir con el campo que requiere binary search. La próxima aplicación al proyecto podrá continuar el reporte temporal desde la vista que construiste hoy.
