# Semana 10 — Stacks & Queues: reglas de acceso sobre colecciones
## COEN 2220 — Programming 2

**Instructor:** Wilson Lozano

**Duración:** 170 min (sesión de 3 horas crédito nominales)  
**Precede a:** Lab 10 — Stacks & Queues  
**Lectura complementaria:** Gaddis, *Starting Out with C++*, 8va ed., Cap. 18 — secciones 18.1, 18.2, 18.4 y 18.5 (Stack ADT, Dynamic Stacks, Queue ADT y Dynamic Queues). Las secciones 18.3 y 18.6 describen contenedores STL; STL no será el mecanismo de implementación ni el foco de evaluación de esta unidad.

---

## Objetivos

Al finalizar la sesión, el estudiante podrá:

1. Explicar que stack y queue son ADTs con reglas de acceso, no keywords de C++ ni sinónimos de linked list.
2. Diferenciar LIFO y FIFO, y justificar cuál regla corresponde a un escenario concreto.
3. Implementar y trazar las operaciones `push`, `pop`, `isEmpty` e `isFull` de un static stack basado en array.
4. Explicar cómo una singly linked list implementa un dynamic stack usando un pointer `top`.
5. Describir el problema de una static queue lineal y cómo `front`, `rear`, `itemCount` y un array circular lo resuelven.
6. Implementar y justificar `enqueue` y `dequeue` en una dynamic queue con pointers `front` y `rear`.

## Parte 0 — Puente: de la representación al contrato de acceso (10 min)

En la Semana 9 construyeron una singly linked list. Esa clase describía **cómo** se conectan los datos: cada node contiene datos y un pointer `next`; `head` conserva la dirección del primer node. La linked list es una **representación** que puede servir para muchas colecciones distintas.

Ahora estudiaremos dos **Abstract Data Types (ADTs)**. Un ADT es un contrato conceptual: define qué operaciones puede pedir el código cliente y qué reglas deben respetar esas operaciones. El contrato se decide antes de elegir si la clase usará un array, una linked list u otra representación interna.

Un **stack** y una **queue** pueden guardar una secuencia de valores, igual que un array o una linked list. Lo que los distingue es que no permiten retirar cualquier elemento arbitrariamente. Cada uno impone una regla sobre cuál elemento puede salir primero.

> **Idea clave:** una linked list responde “¿cómo están conectados los nodes?”. Un stack o una queue responde “¿en qué orden permite el contrato agregar y retirar valores?”. Una misma linked list puede usarse para implementar uno de esos ADTs.

La lista de turnos de Registraduría de la Semana 9 no era una queue estricta porque incluía una regla de prioridad. En una queue que estudiaremos hoy, sin prioridades, el orden de llegada determina exactamente el orden de atención.

## Parte 1 — Stack ADT: trabajar desde un solo extremo (20 min)

### Contexto y definición

Un asistente organiza expedientes de revisión sobre una bandeja. Cada expediente nuevo se coloca encima de los anteriores porque solo hay acceso cómodo a la parte superior. Cuando el asistente necesita revisar uno, toma el expediente que está arriba. El último expediente colocado es, por tanto, el primero que se retira.

Ese comportamiento se llama **LIFO**: *last in, first out* — el último que entra es el primero que sale. Un **stack** es un ADT que almacena elementos con esa regla. El único elemento accesible es el que está en la parte superior, llamada **top**.

Si se colocan los expedientes `101`, `104` y `110` en ese orden, `110` queda en `top`:

```text
top
 |
 v
[110]  <- Last in; first out
[104]
[101]  <- First in; last out
```

El stack no conoce todavía si internamente usará un array o nodes. Su contrato necesita operaciones como estas:

| Operación | Qué solicita el código cliente | Regla que debe respetar |
| --- | --- | --- |
| `push(value)` | Agregar un valor. | El valor nuevo se convierte en `top`. |
| `pop(...)` | Obtener y retirar el valor superior. | Solo puede salir el valor que está en `top`. |
| `peek(...)` | Consultar el valor superior sin retirarlo. | No modifica el stack. |
| `isEmpty()` | Consultar si existen valores. | Impide intentar un `pop` cuando no hay `top`. |
| `isFull()` | Consultar si queda espacio. | Solo aplica a una representación de capacidad fija. |

Dos condiciones reciben nombres útiles:

- **underflow:** intentar `pop` o `peek` cuando el stack está vacío;
- **overflow:** intentar `push` cuando un static stack ya alcanzó su capacidad.

No son errores que ocurren “por magia”. Son situaciones que la implementación debe comprobar para cumplir el contrato de manera segura. Por ahora basta saber que overflow depende de que exista un límite fijo; en la **Parte 2** se construirá precisamente un static stack con un array y una capacidad máxima para ver cómo se detecta ese caso en código.

### ¿Para qué se usa un stack?

LIFO es útil cuando la acción más reciente es la única que tiene sentido revertir o terminar primero. Por ejemplo:

- Un editor de texto puede guardar las acciones del usuario en un stack para implementar **Undo**: si primero se escribió una palabra y después se cambió su formato, la primera acción que debe deshacerse es el formato, porque fue la más reciente.
- Un navegador puede conservar las páginas visitadas para volver a la página anterior: la última página abandonada es la primera que se puede recuperar con el botón **Back**.
- Mientras un programa ejecuta una función que llama a otra función, el sistema conserva información sobre las llamadas pendientes. La función que terminó más recientemente debe devolver el control antes que las llamadas anteriores; ese orden también es LIFO.

En cada caso, un stack no se elige porque “usa un array” o porque “usa nodes”. Se elige porque la regla del problema exige recuperar primero lo último que se registró.

<details>
<summary>Pregunta de verificación: si primero se ejecuta <code>push(5)</code>, luego <code>push(8)</code> y finalmente <code>pop</code>, ¿qué valor debe salir?</summary>

Debe salir `8`. Fue el último valor agregado y por eso quedó en `top`.

</details>

## Parte 2 — Static stack: un array y un índice `topIndex` (25 min)

### De la regla LIFO a una representación concreta

Supongan que la oficina puede manejar como máximo cuatro expedientes a la vez. Esa capacidad fija permite implementar el stack con un array. La clase necesita tres piezas de estado:

1. `reviewIds`, el array donde se almacenan los IDs;
2. `CAPACITY`, el límite fijo del array;
3. `topIndex`, el índice del valor que está actualmente en `top`.

Cuando no hay valores, ningún índice del array representa el top. Usaremos `topIndex == -1` para expresar un stack vacío. Después de `push(101)`, `topIndex` vale `0`; después de `push(104)`, vale `1`.

```text
Empty stack                    After push(101)              After push(104)
topIndex = -1                  topIndex = 0                 topIndex = 1
[   ][   ][   ][   ]           [101][   ][   ][   ]          [101][104][   ][   ]
                               top -> 101                    top -> 104
```

Antes de leer el código, traduzcamos cada operación a pasos:

- `push`: verificar que no está lleno, aumentar `topIndex` y guardar el nuevo valor en esa posición.
- `pop`: verificar que no está vacío, copiar el valor de `topIndex` a una variable del cliente y disminuir `topIndex`.
- `peek`: verificar que no está vacío y copiar el valor de `topIndex`, pero no cambiar el índice.

El siguiente programa completo modela ese escenario. La función `pop` recibe `int& removedReviewId`: el `&` indica un reference parameter. Así puede copiar el valor retirado en una variable que `main` ya posee y, al mismo tiempo, devolver mediante `bool` si la operación fue posible.

```cpp
#include <iostream>
using namespace std;

class ReviewStack
{
private:
    static const int CAPACITY = 4;
    int reviewIds[CAPACITY]; // Store IDs in fixed array positions.
    int topIndex;            // Store the index of the current top value.

public:
    ReviewStack()
    {
        topIndex = -1;       // No array position is the top yet.
    }

    bool isEmpty() const
    {
        return topIndex == -1; // An empty stack has no top value.
    }

    bool isFull() const
    {
        return topIndex == CAPACITY - 1; // The last valid index is full.
    }

    bool push(int newReviewId)
    {
        if (isFull())
        {
            return false; // Do not write outside the fixed array.
        }

        topIndex++;                    // Reserve the next position as top.
        reviewIds[topIndex] = newReviewId; // Store the new top value.
        return true;
    }

    bool pop(int& removedReviewId)
    {
        if (isEmpty())
        {
            return false; // There is no top value to remove.
        }

        removedReviewId = reviewIds[topIndex]; // Copy the current top value.
        topIndex--;                             // The position is no longer in the stack.
        return true;
    }

    bool peek(int& topReviewId) const
    {
        if (isEmpty())
        {
            return false; // There is no top value to inspect.
        }

        topReviewId = reviewIds[topIndex]; // Copy without changing topIndex.
        return true;
    }
};

int main()
{
    ReviewStack reviews;
    int reviewId;

    reviews.push(101); // The stack bottom is now 101.
    reviews.push(104); // The top is now 104.
    reviews.push(110); // The top is now 110.

    if (reviews.peek(reviewId))
    {
        cout << "Current top review: " << reviewId << endl;
    }

    while (reviews.pop(reviewId))
    {
        cout << "Removed review: " << reviewId << endl;
    }

    cout << "Stack is empty: " << boolalpha << reviews.isEmpty() << endl;

    return 0;
}
```

La salida confirma LIFO:

```text
Current top review: 110
Removed review: 110
Removed review: 104
Removed review: 101
Stack is empty: true
```

En `push`, aumentar `topIndex` **antes** de guardar el valor es correcto porque una lista vacía comienza en `-1`; el primer elemento debe ocupar la posición `0`. En `pop`, se copia el valor **antes** de disminuir el índice para no perder cuál era el top.

> **Conexión con Big-O:** `push`, `pop` y `peek` consultan o cambian una sola posición conocida del array. Su trabajo no aumenta al crecer la cantidad de elementos almacenados: son operaciones `O(1)`.

## Parte 3 — Dynamic stack: una linked list cuyo `head` es `top` (20 min)

Un static stack tiene una capacidad fija. Si la oficina necesita conservar una cantidad de expedientes que no conoce de antemano, puede usar una linked list. En ese caso el stack crece y disminuye creando o eliminando nodes dinámicamente; se llama **dynamic stack**.

La regla LIFO sugiere una decisión importante: el `top` debe ser el primer node de la lista. En una clase de linked list general, agregar al final requería recorrer desde `head` hasta el último node. En cambio, un stack solo agrega y retira por un extremo. Si `top` apunta al primer node, `push` y `pop` pueden trabajar directamente allí sin traversal.

```text
top --> [110 | next] --> [104 | next] --> [101 | nullptr]
```

En este ejemplo, `top` cumple el mismo papel que `head` en la linked list de Semana 9, pero su nombre comunica una regla distinta: ese primer node es el único que puede salir.

Antes de definir las operaciones, la clase declara el tipo de node y el pointer que conserva el top. El `struct Node` representa un expediente enlazado; `top` guarda la dirección del node que el contrato permite retirar en este momento.

```cpp
class DynamicReviewStack
{
private:
    struct Node
    {
        int reviewId; // Store one review ID in this node.
        Node *next;   // Point to the node below this one in the stack.
    };

    Node *top; // Point to the only node that can be removed next.

public:
    DynamicReviewStack()
    {
        top = nullptr; // An empty stack has no top node.
    }

    void push(int newReviewId);
    bool pop(int& removedReviewId);
};
```

### `push`: convertir un node nuevo en el top

Los prototipos de `push` y `pop` ya aparecieron dentro de la declaración de `DynamicReviewStack`. Las definiciones siguientes se escriben **después** de cerrar la clase. Por eso usan `DynamicReviewStack::`: el operador `::` indica que esta función pertenece a esa clase y que puede acceder a sus members `Node` y `top`.

Para agregar `newReviewId`, se crea un node y se conecta delante del top actual:

```cpp
void DynamicReviewStack::push(int newReviewId)
{
    Node *newNode = new Node;
    newNode->reviewId = newReviewId;
    newNode->next = top; // Preserve the previous top below the new node.
    top = newNode;       // The new node becomes the only accessible top.
}
```

La línea `newNode->next = top` debe ocurrir antes de `top = newNode`. Primero se conserva la dirección del antiguo top; después se reemplaza `top` por la dirección del node nuevo.

### `pop`: retirar el top y conservar el resto

Para retirar un valor, primero se verifica si existe `top`. Luego se guarda su dato, se mueve `top` al siguiente node y se libera el node que antes estaba arriba:

```cpp
bool DynamicReviewStack::pop(int& removedReviewId)
{
    if (top == nullptr)
    {
        return false; // An empty dynamic stack has no top node.
    }

    Node *nodeToDelete = top;
    removedReviewId = top->reviewId;
    top = top->next;       // The node below becomes the new top.
    delete nodeToDelete;   // Release the removed dynamic node.
    return true;
}
```

Este orden reutiliza la lógica de eliminación que completarán de la Semana 9: actualizar el pointer que mantiene la colección antes de liberar el node. Una clase completa también necesita un destructor que libere los nodes restantes al terminar la vida del objeto. No se debe copiar superficialmente un stack que posee nodes dinámicos; aplican las mismas consideraciones de ownership y copies vistas para linked lists.

Un dynamic stack no tiene `isFull()` basado en capacidad: puede seguir creciendo mientras el programa pueda obtener memoria dinámica. Eso no significa que toda operación sea gratuita; significa que su límite no es un número fijo de posiciones de array.

## Parte 4 — Queue ADT: entrada por un extremo, salida por el otro (20 min)

### Contexto y definición

El Centro de Copiado del campus recibe solicitudes de impresión. No hay reglas de prioridad: la primera solicitud recibida debe ser la primera atendida. Si llegan las solicitudes `PR-101`, `PR-104` y `PR-110`, se atenderán exactamente en ese orden.

Ese comportamiento se llama **FIFO**: *first in, first out* — el primero que entra es el primero que sale. Una **queue** es un ADT que mantiene esa regla. Usa dos nombres para los extremos de la colección:

- **front:** extremo desde el que se retira la próxima solicitud;
- **rear:** extremo donde se agrega una solicitud nueva.

```text
front                                         rear
  |                                             |
  v                                             v
[PR-101] --> [PR-104] --> [PR-110]
  ^ first request to leave       ^ newest request added
```

El contrato de una queue usa estas operaciones:

| Operación | Qué hace | Extremo afectado |
| --- | --- | --- |
| `enqueue(value)` | Agrega una solicitud nueva. | `rear` |
| `dequeue(...)` | Obtiene y retira la solicitud más antigua. | `front` |
| `isEmpty()` | Indica si existe una próxima solicitud. | Consulta la colección. |
| `isFull()` | Indica si queda espacio. | Solo aplica a una queue fija. |

No confundan `enqueue` con `push`: ambos agregan datos, pero no prometen la misma regla de recuperación. Un stack agrega y retira por `top`; una queue agrega por `rear` y retira por `front`.

<details>
<summary>Pregunta de verificación: si una queue ejecuta <code>enqueue("A")</code>, <code>enqueue("B")</code> y luego <code>dequeue</code>, ¿qué valor debe salir?</summary>

Debe salir `A`, porque fue la primera solicitud que entró a la queue.

</details>

## Parte 5 — Static queue: el costo de mover y la idea de circularidad (20 min)

Una queue también puede implementarse con un array de capacidad fija. La clase necesita al menos:

1. `queueArray`, el array que guarda los valores;
2. `frontIndex`, el índice de la próxima solicitud que saldrá;
3. `rearIndex`, el índice donde se agregó la solicitud más reciente;
4. `itemCount`, la cantidad actual de solicitudes;
5. `CAPACITY`, el límite del array.

Una implementación ingenua podría retirar el primer valor y mover todos los demás una posición hacia la izquierda. Eso preserva FIFO, pero cada `dequeue` tendría que mover muchos valores: el trabajo puede crecer con el tamaño de la queue.

Otra opción es permitir que `frontIndex` avance sin mover valores. Sin embargo, después de varios `dequeue`, podría haber espacio libre al comienzo del array aunque `rearIndex` ya llegó al final. Para reutilizar esas posiciones se usa un **circular array**: después de la última posición, el índice vuelve a `0`.

El operador `%` devuelve el residuo de una división. Si `CAPACITY` es `5`, los únicos índices válidos son `0`, `1`, `2`, `3` y `4`. Supongan que `rearIndex` ya vale `4`, la última posición del array, y llega una solicitud nueva. Para avanzar una posición se calcula `rearIndex + 1`, es decir, `4 + 1`. Como `5 % 5` vale `0`, la expresión `(4 + 1) % 5` produce `0`: el siguiente valor se guarda al comienzo del array sin usar una posición inválida.

```text
Indexes:       0     1     2     3     4
queueArray:  [D]   [ ]   [ ]   [B]   [C]
              ^                 ^
              rear              front

Logical FIFO order: B (index 3) -> C (index 4) -> D (index 0)
```

En esa traza, `B` sigue siendo la próxima solicitud porque `frontIndex` vale `3`. Aunque `D` se almacenó en la posición `0`, se agregó después de `C`: `rearIndex` avanzó desde `4` y regresó a `0` mediante `% 5`.

`itemCount` evita una ambigüedad importante. En un array circular, `frontIndex == rearIndex` podría aparecer cuando solo hay un elemento, pero también después de dar una vuelta completa. El contador permite preguntar claramente si la queue está vacía (`itemCount == 0`) o llena (`itemCount == CAPACITY`).

Estos datos pertenecen a una clase que representa la queue; por eso son `private`. El programa cliente no cambia índices directamente: crea un objeto y solicita `enqueue` o `dequeue`.

```cpp
class StaticPrintRequestQueue
{
private:
    static const int CAPACITY = 5;
    int queueArray[CAPACITY]; // Store request IDs in fixed positions.
    int frontIndex;           // Store the index of the next request to leave.
    int rearIndex;            // Store the index of the most recently added request.
    int itemCount;            // Store how many requests are currently in the queue.

public:
    StaticPrintRequestQueue();
    bool enqueue(int newRequestId);
    bool dequeue(int& removedRequestId);
};
```

El constructor, `enqueue` y `dequeue` se definen después de cerrar la clase. El nombre `StaticPrintRequestQueue::` los conecta con esa clase y les permite usar sus members privados. El constructor establece el estado vacío; se usa `CAPACITY - 1` para que el primer `enqueue` avance `rearIndex` a la posición `0`.

```cpp
StaticPrintRequestQueue::StaticPrintRequestQueue()
{
    frontIndex = 0;
    rearIndex = CAPACITY - 1; // The first enqueue advances rearIndex to 0.
    itemCount = 0;
}

bool StaticPrintRequestQueue::enqueue(int newRequestId)
{
    if (itemCount == CAPACITY)
    {
        return false; // The fixed queue has no free position.
    }

    rearIndex = (rearIndex + 1) % CAPACITY; // Move or wrap to the next rear position.
    queueArray[rearIndex] = newRequestId;
    itemCount++;
    return true;
}

bool StaticPrintRequestQueue::dequeue(int& removedRequestId)
{
    if (itemCount == 0)
    {
        return false; // No front value exists.
    }

    removedRequestId = queueArray[frontIndex];
    frontIndex = (frontIndex + 1) % CAPACITY; // Move or wrap to the next front position.
    itemCount--;
    return true;
}
```

Esta representación logra `enqueue` y `dequeue` en `O(1)`, pero su lógica de índices es más delicada que la de un stack fijo. La queue sigue teniendo capacidad limitada; circularidad reutiliza posiciones, no crea capacidad nueva.

## Parte 6 — Dynamic queue: linked list con `front` y `rear` (30 min)

Una dynamic queue evita tanto la capacidad fija como el movimiento de valores de un array. Usa nodes enlazados y conserva dos pointers:

- `front` señala el primer node, la próxima solicitud que debe salir;
- `rear` señala el último node, donde debe conectarse una solicitud nueva.

Mantener ambos pointers es una decisión de diseño esencial. Si una clase solo tuviera `front`, tendría que hacer traversal hasta el último node cada vez que quisiera ejecutar `enqueue`. Con `rear`, puede conectar directamente el node nuevo al final.

Una queue vacía necesita una representación consistente: tanto `front` como `rear` valen `nullptr`. Cuando se agrega la primera solicitud, ambos deben señalar ese mismo node.

La siguiente declaración muestra dónde existen esos dos pointers y qué contiene cada node. Se llama `DynamicPrintRequestQueue` para distinguirla de `StaticPrintRequestQueue`, que usaba un array en la Parte 5. El código cliente no modifica enlaces directamente: solicita `enqueue` o `dequeue` a la clase.

```cpp
class DynamicPrintRequestQueue
{
private:
    struct Node
    {
        string requestId; // Store the ID of one print request.
        Node *next;       // Point to the request immediately behind it.
    };

    Node *front; // Point to the request that must leave next.
    Node *rear;  // Point to the request most recently added.

public:
    DynamicPrintRequestQueue()
    {
        front = nullptr; // An empty queue has no first request.
        rear = nullptr;  // An empty queue has no last request.
    }

    void enqueue(const string& newRequestId);
    bool dequeue(string& removedRequestId);
};
```

### `enqueue`: agregar en `rear`

Los prototipos de la clase anterior se definen después de cerrar `DynamicPrintRequestQueue`. El operador `DynamicPrintRequestQueue::` indica que cada definición es un member function de esa clase y le permite acceder a `Node`, `front` y `rear`. `newNode` se prepara antes de modificar los pointers de la queue:

```cpp
void DynamicPrintRequestQueue::enqueue(const string& newRequestId)
{
    Node *newNode = new Node;
    newNode->requestId = newRequestId;
    newNode->next = nullptr; // A new request will be the last node.

    if (rear == nullptr)
    {
        front = newNode; // The first node is also the front.
        rear = newNode;  // The first node is also the rear.
        return;
    }

    rear->next = newNode; // Link the old rear to the new last node.
    rear = newNode;       // Update rear to the new last node.
}
```

El `if` trata el único caso donde todavía no existe un rear. En una queue no vacía, `front` no cambia al agregar una solicitud: la solicitud más antigua continúa siendo la próxima en salir.

### `dequeue`: retirar desde `front`

Para retirar la solicitud más antigua, se guarda el node de `front`, se copia su ID, se mueve `front` y finalmente se libera el node antiguo. Hay un caso adicional: si se removió el último node, `rear` también debe convertirse en `nullptr`.

```cpp
bool DynamicPrintRequestQueue::dequeue(string& removedRequestId)
{
    if (front == nullptr)
    {
        return false; // An empty queue has no request to remove.
    }

    Node *nodeToDelete = front;
    removedRequestId = front->requestId;
    front = front->next; // The next request becomes the new front.

    if (front == nullptr)
    {
        rear = nullptr; // Removing the last node leaves both ends empty.
    }

    delete nodeToDelete;
    return true;
}
```

La condición `if (front == nullptr)` después de mover `front` no pregunta si la queue estaba vacía al inicio; pregunta si quedó vacía **después** de retirar su único node. Omitir esa actualización dejaría `rear` apuntando a memoria que ya fue liberada.

### Traza de la primera solicitud y de la última salida

Al ejecutar `enqueue("PR-101")` sobre una queue vacía, ambos pointers deben señalar el mismo node:

```text
front                                  rear
  |                                      |
  +--------------> [PR-101 | nullptr] <-+
```

Después de agregar `PR-104`, solo `rear` se mueve:

```text
front                                      rear
  |                                          |
  v                                          v
[PR-101 | next] -----------------> [PR-104 | nullptr]
```

Si luego se ejecuta `dequeue` dos veces, el segundo `dequeue` retira el último node. Por eso ambos pointers terminan en `nullptr`; ese estado permitirá que un `enqueue` futuro trate la nueva solicitud como la primera otra vez.

> **Conexión con Big-O:** con `front` y `rear`, `enqueue` conecta un node conocido y `dequeue` retira un node conocido. Ambas operaciones son `O(1)`. Una linked list general con solo `head` no ofrece ese `enqueue` constante al final, porque tendría que encontrar el último node mediante traversal.

## Parte 7 — Elegir la regla correcta y resumir (25 min)

Antes de escoger una implementación, el diseñador debe identificar la regla de acceso del problema. La siguiente tabla resume las diferencias principales:

| Necesidad | Stack | Queue | Linked list general |
| --- | --- | --- | --- |
| Regla de salida | LIFO | FIFO | La clase define la regla. |
| Punto de inserción | `top` | `rear` | Depende de la operación. |
| Punto de retiro | `top` | `front` | Depende de la operación. |
| Estado mínimo | Un `top` o `topIndex` | `front` y `rear`, o índices equivalentes | Usualmente `head`; puede tener más pointers. |
| Posible representación | Array o linked list | Array o linked list | Nodes enlazados. |

### Práctica de cierre

Para cada escenario, identifiquen el ADT apropiado y justifiquen su regla de salida antes de pensar en código:

1. Un historial de acciones de edición debe deshacer primero la acción más reciente.
2. Una oficina atiende solicitudes sin prioridad en el mismo orden en que fueron recibidas.
3. Un catálogo de recursos debe agregar, buscar y posiblemente insertar o remover en posiciones distintas según la regla del negocio.

<details>
<summary>Ver respuestas razonadas</summary>

1. Stack: deshacer requiere retirar primero la acción más reciente, por lo que sigue LIFO.
2. Queue: la primera solicitud recibida debe ser la primera atendida, por lo que sigue FIFO.
3. Una linked list general u otra colección diseñada para las operaciones necesarias: el escenario no impone por sí mismo LIFO ni FIFO.

</details>

## Resumen de la sesión

- Stack y queue son ADTs: sus contratos especifican reglas de acceso; un array o una linked list son posibles implementaciones.
- Un stack usa LIFO y trabaja desde `top`. Un static stack usa `topIndex`; un dynamic stack puede usar un pointer `top` al primer node.
- Una queue usa FIFO: `enqueue` agrega por `rear` y `dequeue` retira por `front`.
- Una static queue puede usar un array circular con `frontIndex`, `rearIndex` e `itemCount` para reutilizar posiciones sin mover todos los valores.
- Una dynamic queue mantiene `front` y `rear` para conectar y retirar nodes en `O(1)`.

Los últimos 10 minutos de esta parte se reservan para volver a trazar una operación, resolver dudas sobre pointers o completar la práctica de cierre.

## Próxima sesión

En el Lab 10 implementarán y probarán las operaciones fundamentales de stacks y queues. El objetivo será justificar qué pointer o índice cambia en cada operación, no memorizar líneas de código ni usar STL.
