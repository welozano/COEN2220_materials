# Lab 10 — Stacks & Queues: Static and Dynamic Implementations
## COEN 2220 — Programming 2

**Instructor:** Wilson Lozano

**Duración:** 110 min (sesión de 2 horas crédito nominales)  
**Precede a:** Semana 11 — Recursion  
**Requisitos:** Lecture de Semana 10; Labs 1–9 completados  
**Basado en:** Gaddis, *Starting Out with C++*, 8va ed., Cap. 18 — secciones 18.1, 18.2, 18.4 y 18.5 (Stack ADT, Dynamic Stacks, Queue ADT y Dynamic Queues). Este lab no usa STL.

---

## Objetivos

Al finalizar este laboratorio, el estudiante podrá:

1. Explicar cuándo una queue de capacidad fija se beneficia de un circular array y cuándo una queue necesita crecer dinámicamente.
2. Trazar cómo `frontIndex`, `rearIndex` e `itemCount` preservan FIFO en una static circular queue.
3. Representar un dynamic stack mediante nodes y un pointer `top`.
4. Implementar `push`, `pop` y `peek` para un dynamic stack sin alterar enlaces que no corresponden a la operación.
5. Implementar `enqueue` y `dequeue` para una dynamic queue con pointers `front` y `rear`.
6. Justificar los casos especiales de la primera inserción y de retirar el último node de una queue dinámica.

> **Alcance de este lab:** practicarán static circular queues, dynamic stacks y dynamic queues. No usarán `std::stack`, `std::queue`, `std::deque`, `std::list`, `vector`, prioridades ni estructuras del proyecto final. Los datos usados serán dummy.

## Parte A — Preparar la carpeta y el repositorio (5 min)

> ⚠️ **Este paso es el que más se presta a error — léelo con cuidado antes de hacer clic en nada.** Cada lab tiene su propio repositorio. No inicialices Git en la carpeta padre `COEN2220` ni dentro de la carpeta de otro lab.

1. **A1.** Abre VS Code o VSCodium y selecciona `File → Open Folder...`.
2. **A2.** Navega a tu carpeta padre del curso, por ejemplo `COEN2220`. Si todavía no existe, créala **desde este diálogo** en un lugar de fácil acceso.
3. **A3.** Entra dentro de la carpeta padre. Sin salir del diálogo, crea una carpeta nueva llamada `lab10-stacks-queues`, hermana de las carpetas de los labs anteriores.
4. **A4.** Abre específicamente `lab10-stacks-queues` con **Open** / **Select Folder**.

**Cómo saber si abriste la carpeta correcta:** el Explorer debe mostrar `LAB10-STACKS-QUEUES` como raíz. Si muestra `COEN2220` o el nombre de otro lab, usa `File → Close Folder` y repite los pasos.

> **¿Por qué importa tanto esto?** Inicializar el repositorio en la carpeta padre mezcla todos los labs en un solo historial. Inicializarlo dentro de un lab anterior mezcla archivos y entregas. Un repositorio separado permite evaluar esta práctica como una unidad de trabajo independiente.

Abre Source Control (`Ctrl+Shift+G` / `Cmd+Shift+G`) y selecciona **Initialize Repository**. Crea `.gitignore` en la raíz con este contenido:

```gitignore
# Compiled executables (Windows)
*.exe

# Compiled executables (Mac/Linux)
technology_service_queues_example
function_call_stack
tutoring_queue

# Intermediate object files
*.o

# Editor configuration
.vscode/
```

Haz el commit inicial y publica el repositorio con el flujo ya practicado: stage → commit → **Publish Branch**. Usa el mensaje `"Initial setup and gitignore"`.

💾 **Por qué este commit importa:** guarda una base limpia antes de escribir código. Si aparece un problema después, el historial muestra que la carpeta, el repositorio y `.gitignore` ya estaban correctos.

## Parte B — Ejemplo guiado: dos queues para un Centro de Servicios Tecnológicos (30 min)

### Contexto

El Centro de Servicios Tecnológicos del campus gestiona dos filas distintas durante el día:

- Mantiene exactamente cuatro laptops de préstamo. Cuando una persona recibe una laptop, el centro entrega la que lleva más tiempo disponible. Cuando una laptop se devuelve, se agrega al final de la fila de equipos disponibles. Como solo existen cuatro espacios físicos de almacenamiento, esta colección tiene capacidad fija.
- También recibe solicitudes de soporte, como `"SR-204"`. No conoce cuántas solicitudes llegarán durante el día; cada una debe ser atendida en orden de llegada. Esta colección necesita crecer cuando llega una solicitud nueva.

Para representar esas dos colecciones en C++, el programa declarará dos clases. La clase `LoanerDeviceQueue` representará la fila de **laptops disponibles**: sus elementos son IDs de laptops, como `"LT-101"`. La clase `SupportRequestQueue` representará la fila de **solicitudes de soporte**: sus elementos son IDs de solicitudes, como `"SR-204"`. Los dos nombres son nombres de clases, no keywords de C++.

Ambas clases modelan una regla FIFO, pero no necesitan la misma representación interna. `LoanerDeviceQueue` usa un **circular array**: tiene cuatro posiciones fijas y sus índices vuelven al comienzo del array para reutilizar una posición liberada, sin mover todas las laptops restantes. `SupportRequestQueue` usa **nodes dinámicos**: crea un node por cada solicitud que llega, porque la cantidad de solicitudes no se conoce de antemano.

Crea un archivo llamado `technology_service_queues_example.cpp`. Construirás un solo programa que representa ambas filas. Los bloques de los Pasos B2 y B3 comienzan comentados. En cada paso, quita únicamente los comentarios `/*` y `*/` que mantienen inactivo el bloque indicado; conserva activa la etiqueta `// --- STEP ... ---` para ubicar el bloque correcto.

### B1 — Crear dos queues vacías

Primero observa el estado inicial. Una `LoanerDeviceQueue` vacía usa `itemCount == 0`; aunque sus índices ya tengan valores válidos, aún no existe ninguna laptop disponible. Una `SupportRequestQueue` vacía necesita que **ambos** pointers, `front` y `rear`, sean `nullptr`: todavía no existe ni primera ni última solicitud.

Escribe este starter code y ejecútalo antes de activar los otros pasos:

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 10 - Stacks & Queues
 * Description: Guided example - static and dynamic service queues
 * Due date: [Date]
 */

#include <iostream>
#include <string>
using namespace std;

class LoanerDeviceQueue
{
private:
    static const int CAPACITY = 4;
    string deviceIds[CAPACITY]; // Store available device IDs in fixed positions.
    int frontIndex;             // Store the index of the next device to check out.
    int rearIndex;              // Store the index of the most recently returned device.
    int itemCount;              // Distinguish an empty queue from a full queue.

public:
    LoanerDeviceQueue()
    {
        frontIndex = 0;
        rearIndex = CAPACITY - 1; // The first enqueue advances rearIndex to 0.
        itemCount = 0;
    }

    bool isEmpty() const
    {
        return itemCount == 0; // No device is waiting to be checked out.
    }

    // --- STEP B2: Circular-array queue operations ---
    /*
    bool enqueue(const string& returnedDeviceId)
    {
        if (itemCount == CAPACITY)
        {
            return false; // No fixed array position is free.
        }

        rearIndex = (rearIndex + 1) % CAPACITY; // Advance and wrap when needed.
        deviceIds[rearIndex] = returnedDeviceId;
        itemCount++;
        return true;
    }

    bool dequeue(string& checkedOutDeviceId)
    {
        if (isEmpty())
        {
            return false; // No device can be checked out from an empty queue.
        }

        checkedOutDeviceId = deviceIds[frontIndex];
        frontIndex = (frontIndex + 1) % CAPACITY; // Advance and wrap when needed.
        itemCount--;
        return true;
    }

    void displayAvailableDevices() const
    {
        for (int offset = 0; offset < itemCount; offset++)
        {
            int currentIndex = (frontIndex + offset) % CAPACITY;
            cout << "Available device: " << deviceIds[currentIndex] << endl;
        }
    }
    */
};

class SupportRequestQueue
{
private:
    struct Node
    {
        string requestId; // Store one support-request ID.
        Node *next;       // Point to the request immediately behind this one.
    };

    Node *front; // Point to the request that must be handled next.
    Node *rear;  // Point to the request most recently received.

public:
    SupportRequestQueue()
    {
        front = nullptr;
        rear = nullptr;
    }

    ~SupportRequestQueue()
    {
        while (front != nullptr)
        {
            Node *nodeToDelete = front;
            front = front->next; // Save the next request through front before delete.
            delete nodeToDelete;
        }

        rear = nullptr; // The destructor leaves no pointer to a released node.
    }

    bool isEmpty() const
    {
        return front == nullptr; // An empty dynamic queue has no first node.
    }

    // --- STEP B3: Dynamic queue operations ---
    /*
    void enqueue(const string& newRequestId)
    {
        Node *newNode = new Node;
        newNode->requestId = newRequestId;
        newNode->next = nullptr; // The new request begins as the last node.

        if (rear == nullptr)
        {
            front = newNode; // The first node is also the front.
            rear = newNode;  // The first node is also the rear.
            return;
        }

        rear->next = newNode; // Connect the old rear to the new node.
        rear = newNode;       // The new node becomes the rear.
    }

    bool dequeue(string& handledRequestId)
    {
        if (front == nullptr)
        {
            return false; // An empty queue has no request to handle.
        }

        Node *nodeToDelete = front;
        handledRequestId = front->requestId;
        front = front->next; // The next request becomes the front.

        if (front == nullptr)
        {
            rear = nullptr; // Removing the last node leaves both pointers empty.
        }

        delete nodeToDelete;
        return true;
    }
    */
};

int main()
{
    LoanerDeviceQueue availableDevices;
    SupportRequestQueue supportRequests;
    string deviceId;
    string requestId;

    cout << boolalpha;
    cout << "Loaner device queue empty: "
         << availableDevices.isEmpty() << endl;
    cout << "Support request queue empty: "
         << supportRequests.isEmpty() << endl;

    // --- STEP B2: Reuse array positions after checkouts ---
    /*
    availableDevices.enqueue("LT-101");
    availableDevices.enqueue("LT-102");
    availableDevices.enqueue("LT-103");
    availableDevices.enqueue("LT-104");

    availableDevices.dequeue(deviceId);
    cout << "Checked out device: " << deviceId << endl;
    availableDevices.dequeue(deviceId);
    cout << "Checked out device: " << deviceId << endl;

    availableDevices.enqueue("LT-105"); // rearIndex wraps from 3 to 0.
    availableDevices.enqueue("LT-106");

    availableDevices.displayAvailableDevices();
    */

    // --- STEP B3: Handle requests in arrival order ---
    /*
    supportRequests.enqueue("SR-204");
    supportRequests.enqueue("SR-219");
    supportRequests.enqueue("SR-231");

    supportRequests.dequeue(requestId);
    cout << "Handled request: " << requestId << endl;
    supportRequests.dequeue(requestId);
    cout << "Handled request: " << requestId << endl;
    supportRequests.dequeue(requestId);
    cout << "Handled request: " << requestId << endl;

    cout << "Support request queue empty after handling all requests: "
         << supportRequests.isEmpty() << endl;

    supportRequests.enqueue("SR-245");
    supportRequests.dequeue(requestId);
    cout << "Handled new request after empty queue: " << requestId << endl;
    */

    return 0;
}

/*
 * Analysis Answers
 *
 * B4 — Circular queue state
 * After LT-101 and LT-102 are checked out, why can LT-105 use an array position
 * near the beginning even though LT-103 and LT-104 are still available? Explain
 * the roles of frontIndex, rearIndex, and itemCount.
 * Answer:
 *
 * B5 — Dynamic queue empty state
 * After SR-231 is handled, why must rear become nullptr as well as front? Explain
 * what would be wrong if rear still pointed to the deleted node before SR-245 arrives.
 * Answer:
 *
 */
```

Compila y ejecuta:

```bash
g++ technology_service_queues_example.cpp -o technology_service_queues_example
./technology_service_queues_example
```

**Qué deberías ver en B1:**

```text
Loaner device queue empty: true
Support request queue empty: true
```

<details>
<summary>Pregunta de verificación: ¿por qué una queue vacía de laptops usa <code>itemCount == 0</code>, mientras que una queue dinámica vacía usa <code>front == nullptr</code>?</summary>

La queue de laptops siempre posee un array de cuatro posiciones, aun cuando ninguna contenga una laptop disponible. `itemCount` indica cuántas posiciones forman parte lógica de la queue. La queue dinámica no posee nodes hasta que llega una solicitud; `front == nullptr` indica que no existe primer node.

</details>

### B2 — Reutilizar posiciones en la static circular queue

Ahora quita únicamente los comentarios `/*` y `*/` que rodean los dos bloques `STEP B2`: uno dentro de `LoanerDeviceQueue` y otro dentro de `main`.

Primero se agregan cuatro laptops y se retiran `LT-101` y `LT-102`. En ese momento las posiciones iniciales del array ya no pertenecen a la queue lógica. Cuando se devuelve `LT-105`, `rearIndex` estaba en el último índice válido, `3`; la expresión `(3 + 1) % 4` produce `0`, por lo que reutiliza la posición liberada al comienzo. `displayAvailableDevices` no recorre el array en orden físico de índice; inicia en `frontIndex` y visita exactamente `itemCount` elementos en el orden FIFO.

Compila y ejecuta de nuevo.

**Qué deberías ver adicionalmente:**

```text
Checked out device: LT-101
Checked out device: LT-102
Available device: LT-103
Available device: LT-104
Available device: LT-105
Available device: LT-106
```

<details>
<summary>Pregunta de verificación: después de las dos devoluciones, ¿cuál laptop debe salir primero si se ejecuta otro <code>dequeue</code>?</summary>

Debe salir `LT-103`. Aunque `LT-105` pudo almacenarse en el índice físico `0`, llegó a la queue después de `LT-103` y `LT-104`; FIFO se determina por `frontIndex` y el orden lógico, no por cuál índice numérico es menor.

</details>

### B3 — Atender una cantidad desconocida de solicitudes

Ahora quita únicamente los comentarios `/*` y `*/` que rodean los bloques `STEP B3` dentro de `SupportRequestQueue` y de `main`.

Cada `enqueue` crea un `Node` nuevo. La primera solicitud es un caso especial: no existe un `rear` anterior al cual conectarla, por lo que el mismo node se convierte en `front` y `rear`. `dequeue` debe tratar el caso opuesto: al remover el último node, ambos pointers deben regresar a `nullptr` antes de que llegue una nueva solicitud.

Compila y ejecuta una última vez.

**Qué deberías ver adicionalmente:**

```text
Handled request: SR-204
Handled request: SR-219
Handled request: SR-231
Support request queue empty after handling all requests: true
Handled new request after empty queue: SR-245
```

<details>
<summary>Pregunta de verificación: ¿por qué <code>SR-245</code> se puede agregar correctamente después de que la queue quedó vacía?</summary>

Al retirar `SR-231`, la función dejó tanto `front` como `rear` en `nullptr`. Por eso el siguiente `enqueue` reconoce que está agregando el primer node de una queue nueva y asigna ese node a ambos pointers.

</details>

### B4 — Analizar los dos estados especiales

Al final de `technology_service_queues_example.cpp` aparece un bloque de comentarios llamado `Analysis Answers`. Después de ejecutar los tres pasos, responde B4 y B5 en inglés dentro de los espacios que siguen a `Answer:`. No borres las preguntas ni respondas solo con una palabra; explica los índices o pointers involucrados.

💾 **Buen momento para el commit** — completaste un ejemplo que usa dos representaciones distintas para dos queues FIFO del mismo sistema. Commit: `"Complete technology service queue example"`. Este mensaje identifica que comprobaste tanto circularidad como el manejo dinámico de nodes.

## Parte C — TODO: representar un dynamic stack de llamadas (15 min)

### Contexto

Mientras un programa se ejecuta, una función puede llamar otra función antes de terminar. Por ejemplo, `main` llama `loadSchedule`, y `loadSchedule` llama `formatSchedule`. El programa debe terminar `formatSchedule` antes de regresar a `loadSchedule`; después termina `loadSchedule` antes de regresar a `main`. La última llamada registrada debe ser la primera que sale: ese comportamiento es LIFO.

Para representar ese stack en C++, construirás una clase llamada `FunctionCallStack`. Cada node de esa clase guardará el nombre de una función activa. No se conoce de antemano cuántas llamadas anidadas tendrá un programa, por lo que el stack será dinámico y el data member `top` señalará el único node que puede salir primero.

Crea un archivo llamado `function_call_stack.cpp` y copia este template. Resuelve **solo** los TODO marcados `(Part C)` por ahora. Ignora los TODOs de la Parte D.

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 10 - Stacks & Queues
 * Description: Practice - a dynamic stack of active function calls
 * Due date: [Date]
 */

#include <iostream>
#include <string>
using namespace std;

class FunctionCallStack
{
private:
    // ===== Resolve these TODOs now (Part C) =====

    // TODO (Part C): Define Node with string functionName and Node *next.
    // TODO (Part C): Declare Node *top for the node that must leave next.

public:
    // TODO (Part C): Write a constructor that creates an empty stack.

    // ===== Resolve these TODOs later (Part D) =====

    // TODO (Part D): Write isEmpty as a const member function.
    // TODO (Part D): Write push to add a function name at top.
    // TODO (Part D): Write pop. It returns bool and uses string& removedFunctionName.
    // TODO (Part D): Write peek. It returns bool and uses string& topFunctionName.
    // TODO (Part D): Write a destructor that releases every dynamic node.
};

int main()
{
    cout << boolalpha;

    // ===== Resolve these TODOs later (Part D) =====

    // TODO (Part D): Create a FunctionCallStack object.
    // TODO (Part D): Push main, loadSchedule, and formatSchedule in that order.
    // TODO (Part D): Use peek and pop to demonstrate LIFO with descriptive output.
    // TODO (Part D): Pop remaining calls, then demonstrate an unsuccessful pop on an empty stack.

    return 0;
}

/*
 * Analysis Answers
 *
 * D1 — Adding formatSchedule
 * When push("formatSchedule") executes, what must newNode->next receive before top
 * changes? Explain why reversing those two assignments would lose the previous top.
 * Answer:
 *
 * D2 — Returning from a function
 * After main, loadSchedule, and formatSchedule are pushed in that order, list the
 * order in which three successful pop calls must return their names. Explain LIFO.
 * Answer:
 *
 * D3 — Destructor order
 * Why must the destructor save a next-node address before deleting the current node?
 * Answer:
 *
 */
```

### Antes de escribir

1. **C1.** ¿Qué datos describen una llamada individual y por eso pertenecen a `Node`?
2. **C2.** ¿Qué pointer representa el estado completo del stack y por eso pertenece a `FunctionCallStack`?
3. **C3.** ¿Qué valor debe tener `top` cuando todavía no se ha llamado ninguna función?
4. **C4.** ¿Por qué este escenario no necesita un pointer `rear`?

<details>
<summary>Ver orientación</summary>

Cada node representa una llamada individual, por lo que necesita el nombre de la función y el enlace `next`. `top` representa el acceso a todo el stack y debe iniciar en `nullptr`. Un stack solo agrega y retira desde `top`; a diferencia de una queue, no tiene un segundo extremo de inserción.

</details>

Después de completar la Parte C, compila. `main` todavía no usa la clase, pero el archivo debe compilar correctamente.

```bash
g++ function_call_stack.cpp -o function_call_stack
./function_call_stack
```

💾 **Buen momento para el commit** — definiste la representación de un stack dinámico antes de modificar sus enlaces. Commit: `"Define dynamic function call stack"`. Este punto del historial separa el diseño del estado de las operaciones LIFO.

## Parte D — TODO: agregar, retirar y liberar llamadas (20 min)

Ahora resuelve **solo** los TODO marcados `(Part D)` dentro de `FunctionCallStack` y de `main`.

`push` debe crear un node, guardar el nombre recibido y conectarlo delante del top anterior. `pop` debe verificar primero si `top` existe; si existe, debe copiar el nombre del top a `removedFunctionName`, mover `top` al node siguiente y liberar el node que salió. `peek` copia el nombre de `top` sin retirar ningún node. Como `FunctionCallStack` crea nodes con `new`, su destructor debe recorrer y liberar los nodes que permanezcan cuando el objeto termine.

### Antes de escribir

1. **D1.** En `push`, ¿por qué `newNode->next = top` debe ocurrir antes de `top = newNode`?
2. **D2.** En `pop`, ¿qué información necesitas conservar antes de ejecutar `delete`?
3. **D3.** ¿Qué member cambia en `peek`?
4. **D4.** En el destructor, ¿por qué no puedes leer `currentNode->next` después de `delete currentNode`?
5. **D5.** ¿Por qué un dynamic stack no necesita `isFull()` basado en una capacidad fija?

<details>
<summary>Ver orientación</summary>

Antes de reemplazar `top`, el node nuevo debe guardar la dirección del top anterior. Antes de liberar un node, debes conservar la dirección que permite continuar el recorrido. `peek` no cambia ningún member: solo consulta el top. Un dynamic stack puede agregar nodes mientras el programa pueda obtener memoria dinámica; no tiene un límite fijo de posiciones como un array.

</details>

Tu prueba debe demostrar este orden de salida:

```text
Current function: formatSchedule
Returned from function: formatSchedule
Returned from function: loadSchedule
Returned from function: main
Stack empty: true
Pop from empty stack succeeded: false
```

Compila y ejecuta antes de continuar:

```bash
g++ function_call_stack.cpp -o function_call_stack
./function_call_stack
```

Después, responde D1–D3 dentro del bloque `Analysis Answers` al final de `function_call_stack.cpp`. Las respuestas deben estar en inglés y justificar el orden de los pointers y nodes; no escribas solo `"LIFO"`.

💾 **Buen momento para el commit** — el stack ya agrega, consulta, retira y libera nodes dinámicos, y `main` demuestra la regla LIFO. Commit: `"Implement dynamic function call stack"`.

## Parte E — TODO: atender solicitudes con una dynamic queue (25 min)

### Contexto

El Centro de Tutorías recibe estudiantes durante el día. Cada estudiante obtiene un código dummy, como `"TU-301"`, y espera una sesión breve. No hay prioridades: quien llega primero debe ser atendido primero. La cantidad de estudiantes cambia cada día y no existe un límite fijo de lugares en la lista del programa; por eso la clase usará una dynamic queue.

Para representar esa fila en C++, el template declara una clase llamada `TutoringQueue`. La clase ya declara su tipo `Node`, sus data members `front` y `rear`, su constructor y su destructor. Esos elementos describen cómo la clase guarda la colección; tu trabajo será implementar las operaciones que respetan FIFO: agregar por `rear` y retirar por `front`.

Crea un archivo llamado `tutoring_queue.cpp` y copia este template. Resuelve **solo** los TODO marcados `(Part E)`.

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 10 - Stacks & Queues
 * Description: Practice - a dynamic queue of tutoring requests
 * Due date: [Date]
 */

#include <iostream>
#include <string>
using namespace std;

class TutoringQueue
{
private:
    struct Node
    {
        string studentId; // Store one student waiting for tutoring.
        Node *next;       // Point to the student immediately behind this one.
    };

    Node *front; // Point to the student who must be served next.
    Node *rear;  // Point to the student most recently added.

public:
    TutoringQueue()
    {
        front = nullptr;
        rear = nullptr;
    }

    ~TutoringQueue()
    {
        while (front != nullptr)
        {
            Node *nodeToDelete = front;
            front = front->next; // Advance before releasing the current node.
            delete nodeToDelete;
        }

        rear = nullptr;
    }

    bool isEmpty() const
    {
        return front == nullptr;
    }

    // ===== Resolve these TODOs now (Part E) =====

    // TODO (Part E): Write enqueue to add newStudentId at rear.
    // TODO (Part E): Write dequeue. It returns bool and uses string& servedStudentId.
};

int main()
{
    cout << boolalpha;

    // TODO (Part E): Create a TutoringQueue object.
    // TODO (Part E): Add TU-301, TU-318, and TU-327 in that arrival order.
    // TODO (Part E): Dequeue and print the first two served student IDs.
    // TODO (Part E): Dequeue the last student, then print whether the queue is empty.
    // TODO (Part E): Add TU-340 after the queue becomes empty, dequeue it, and print its ID.

    return 0;
}

/*
 * Analysis Answers
 *
 * E1 — First arrival
 * Why must the first call to enqueue assign newNode to both front and rear?
 * Answer:
 *
 * E2 — Last departure
 * After dequeue removes the only remaining node, what must happen to rear and why?
 * Answer:
 *
 * E3 — FIFO evidence
 * List the expected service order for TU-301, TU-318, and TU-327. Explain why this
 * differs from the return order of the function-call stack in Part D.
 * Answer:
 *
 */
```

### Antes de escribir

1. **E1.** ¿Qué caso especial existe cuando `rear == nullptr` durante `enqueue`?
2. **E2.** Durante un `enqueue` sobre una queue no vacía, ¿qué pointer cambia y qué pointer no cambia?
3. **E3.** Después de mover `front` en `dequeue`, ¿cómo sabes que también debes cambiar `rear`?
4. **E4.** ¿Qué node se debe liberar y en qué momento de la operación?
5. **E5.** ¿Por qué `TutoringQueue` usa dos pointers, mientras `FunctionCallStack` solo necesita `top`?

<details>
<summary>Ver orientación</summary>

Cuando no existe `rear`, el node nuevo es simultáneamente el primer y el último node. En una queue no vacía, `enqueue` conecta el node nuevo después de `rear` y actualiza solo `rear`; `front` conserva la próxima persona que será atendida. Después de retirar, si `front` queda en `nullptr`, se retiró el último node y `rear` también debe quedar en `nullptr`. La queue necesita ambos extremos porque agrega en uno y retira en el otro.

</details>

Tu programa debe demostrar esta forma de output:

```text
Served student: TU-301
Served student: TU-318
Served student: TU-327
Queue empty: true
Served student after empty queue: TU-340
```

Compila y ejecuta antes de entregar:

```bash
g++ tutoring_queue.cpp -o tutoring_queue
./tutoring_queue
```

Después, responde E1–E3 dentro del bloque `Analysis Answers` al final de `tutoring_queue.cpp`. Las respuestas deben estar en inglés y explicar los pointers involucrados.

💾 **Buen momento para el commit** — comprobaste la llegada, atención y reinicio de una dynamic queue, incluyendo los casos de primer y último node. Commit: `"Implement dynamic tutoring queue"`.

## Parte F — Revisión, evaluación y entrega (15 min)

Antes de entregar, verifica:

- `technology_service_queues_example.cpp` compila y los bloques B2 y B3 están activos.
- El ejemplo guiado demuestra que `LT-105` reutiliza una posición liberada mediante circularidad y que las solicitudes se atienden en orden FIFO.
- El bloque `Analysis Answers` de `technology_service_queues_example.cpp` responde B4 y B5 en inglés con explicaciones propias.
- `function_call_stack.cpp` compila e implementa un stack con `Node *top`, `push`, `pop`, `peek`, `isEmpty` y destructor.
- Las pruebas de `FunctionCallStack` demuestran LIFO y un `pop` fallido sobre un stack vacío.
- El bloque `Analysis Answers` de `function_call_stack.cpp` responde D1–D3 en inglés con explicaciones propias.
- `tutoring_queue.cpp` compila e implementa una queue con `front`, `rear`, `enqueue`, `dequeue`, `isEmpty` y destructor.
- Las pruebas de `TutoringQueue` demuestran FIFO, la primera llegada, la salida del último node y una nueva llegada después de quedar vacía.
- El bloque `Analysis Answers` de `tutoring_queue.cpp` responde E1–E3 en inglés con explicaciones propias.
- Los datos son dummy; los identificadores, comentarios y mensajes del programa están en inglés.
- `.gitignore` evita que ejecutables y archivos objeto aparezcan en Source Control.

### Criterios de evaluación (100 puntos)

| Criterio | Puntos |
| --- | ---: |
| Setup, `.gitignore`, headers y commits claros | 10 |
| Ejemplo guiado: static circular queue y dynamic queue activas; análisis B4–B5 | 25 |
| Representación correcta de `FunctionCallStack` y estado inicial | 15 |
| Operaciones LIFO, destructor, pruebas y análisis D1–D3 | 25 |
| Operaciones FIFO de `TutoringQueue`, destructor, pruebas y análisis E1–E3 | 25 |

## Entregable del laboratorio

Envía al profesor el enlace de tu repositorio `lab10-stacks-queues` en GitHub, con:

- `technology_service_queues_example.cpp`, con los tres pasos activos y las respuestas B4–B5 completas.
- `function_call_stack.cpp`, con todos los TODOs de las Partes C y D resueltos y las respuestas D1–D3 completas.
- `tutoring_queue.cpp`, con todos los TODOs de la Parte E resueltos y las respuestas E1–E3 completas.
- Un `.gitignore` funcionando, sin ejecutables ni archivos objeto en Source Control.
- Al menos **4 commits** visibles: setup, ejemplo guiado, dynamic stack y dynamic queue.
- Headers estándar completos con tu nombre y fecha reales en los tres archivos fuente.

## Próxima sesión

En la Semana 11 estudiarán recursion. El call stack de este lab ofrece un puente importante: cada llamada recursiva también debe terminar y regresar en el orden inverso en que fue iniciada.
