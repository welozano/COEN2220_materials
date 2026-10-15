# Lab 9 — Linked Lists: Nodes, Traversal, and Linear Search
## COEN 2220 — Programming 2

**Instructor:** Wilson Lozano

**Duración:** 110 min (sesión de 2 horas crédito nominales)  
**Precede a:** Semana 10 — Stacks & Queues  
**Requisitos:** Lecture de Semana 9, Partes 1–4; Labs 1–8 completados  
**Basado en:** Gaddis, *Starting Out with C++*, 8va ed., Cap. 17 — secciones 17.1–17.2 (Linked List ADT y Linked List Operations). Este lab no usa STL.

---

## Objetivos

Al finalizar este laboratorio, el estudiante podrá:

1. Representar una colección de tamaño desconocido mediante nodes conectados y un pointer `head`.
2. Crear un node dinámico, inicializar sus data members y agregarlo al final de una singly linked list.
3. Recorrer una linked list con un pointer de trabajo sin modificar `head`.
4. Implementar una búsqueda lineal que devuelva si un valor está almacenado en la lista.
5. Distinguir el propósito de `head`, `currentNode` y un contador de elementos de la clase.

> **Alcance de este lab:** practicarán únicamente lo impartido hasta la Parte 4 de la lecture: nodes, `head`, creación dinámica, agregar al final, traversal y búsqueda lineal. **No** implementarán inserción por prioridad, eliminación, destructores, listas doubly/circular ni `std::list`.

## Parte A — Preparar la carpeta y el repositorio (5 min)

> ⚠️ **Este paso es el que más se presta a error — léelo con cuidado antes de hacer clic en nada.** Cada lab tiene su propio repositorio. No inicialices Git en la carpeta padre `COEN2220` ni dentro de la carpeta de otro lab.

1. **A1.** Abre VS Code o VSCodium y selecciona `File → Open Folder...`.
2. **A2.** Navega a tu carpeta padre del curso, por ejemplo `COEN2220`. Si todavía no existe, créala **desde este diálogo** en un lugar de fácil acceso.
3. **A3.** Entra dentro de la carpeta padre. Sin salir del diálogo, crea una carpeta nueva llamada `lab9-linked-lists`, hermana de las carpetas de los labs anteriores.
4. **A4.** Abre específicamente `lab9-linked-lists` con **Open** / **Select Folder**.

**Cómo saber si abriste la carpeta correcta:** el Explorer debe mostrar `LAB9-LINKED-LISTS` como raíz. Si muestra `COEN2220` o el nombre de otro lab, usa `File → Close Folder` y repite los pasos.

> **¿Por qué importa tanto esto?** Inicializar el repositorio en la carpeta padre mezcla todos los labs en un solo historial. Inicializarlo dentro de un lab anterior mezcla archivos y entregas. Un repositorio separado permite evaluar esta práctica como una unidad de trabajo independiente.

Abre Source Control (`Ctrl+Shift+G` / `Cmd+Shift+G`) y selecciona **Initialize Repository**. Crea `.gitignore` en la raíz con este contenido:

```gitignore
# Compiled executables (Windows)
*.exe

# Compiled executables (Mac/Linux)
lost_item_list_example
study_group_list

# Intermediate object files
*.o

# Editor configuration
.vscode/
```

Haz el commit inicial y publica el repositorio con el flujo ya practicado: stage → commit → **Publish Branch**. Usa el mensaje `"Initial setup and gitignore"`.

💾 **Por qué este commit importa:** guarda una base limpia antes de escribir código. Si aparece un problema después, el historial muestra que la carpeta, el repositorio y `.gitignore` ya estaban correctos.

## Parte B — Ejemplo guiado: `LostItemList` (30 min)

### Contexto

La oficina de Objetos Perdidos del campus recibe artículos durante el día, pero no conoce de antemano cuántos artículos registrará. Cada artículo recibe un código, como `LF-104`. El personal necesita registrar los códigos en el orden en que llegan, mostrar todos los códigos registrados y confirmar si un código particular aparece en el registro.

La lógica de este ejemplo refuerza las operaciones vistas en la lecture con un contexto distinto. La colección se llamará `LostItemList`; cada node guardará un código y la dirección del node siguiente. La clase conservará `head` para recordar dónde comienza la lista.

> **Nota de alcance:** el ejemplo usa `new` para crear nodes, pero no implementa un destructor. Esa operación pertenece a una parte posterior de la lecture y queda fuera de este lab. No agregues `delete` ni un destructor por tu cuenta todavía.

Crea un archivo llamado `lost_item_list_example.cpp`. Construirás el ejemplo en tres pasos. El programa inicial tiene bloques comentados para los Pasos B2 y B3. En cada paso, quita únicamente los comentarios `/*` y `*/` que mantienen inactivo el bloque indicado; la etiqueta `// --- STEP ... ---` permanece activa para que puedas ubicar el código correcto.

### B1 — Crear una lista vacía

Una linked list necesita primero dos cosas: un tipo para representar **un** node y un pointer que recuerde el primer node de la colección. `head` no es un node; guarda una dirección. Cuando no existe ningún artículo registrado, `head` debe ser `nullptr`.

Escribe el siguiente starter code y ejecútalo antes de activar los otros pasos:

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 9 - Linked Lists
 * Description: Guided example - a linked list of lost-item codes
 * Due date: [Date]
 */

#include <iostream>
#include <string>
using namespace std;

class LostItemList
{
private:
    struct Node
    {
        string itemCode; // Store one lost-item code.
        Node *next;      // Store the address of the next node.
    };

    Node *head;          // Store the first node address or nullptr.

public:
    LostItemList()
    {
        head = nullptr;  // A new list begins with no nodes.
    }

    bool isEmpty() const
    {
        return head == nullptr; // No first node means the list is empty.
    }

    // --- STEP B2: Add one item at the end ---
    /*
    void addItem(const string& newItemCode)
    {
        Node *newNode = new Node;
        newNode->itemCode = newItemCode;
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

        currentNode->next = newNode;
    }
    */

    // --- STEP B3: Traverse and search the list ---
    /*
    void display() const
    {
        Node *currentNode = head;

        while (currentNode != nullptr)
        {
            cout << "Lost item: " << currentNode->itemCode << endl;
            currentNode = currentNode->next;
        }
    }

    bool contains(const string& targetItemCode) const
    {
        Node *currentNode = head;

        while (currentNode != nullptr)
        {
            if (currentNode->itemCode == targetItemCode)
            {
                return true;
            }

            currentNode = currentNode->next;
        }

        return false;
    }
    */
};

int main()
{
    cout << boolalpha; // Print bool results as true or false.

    LostItemList lostItems;

    cout << "List starts empty: " << lostItems.isEmpty() << endl;

    // --- STEP B2: Add dummy lost-item codes ---
    /*
    lostItems.addItem("LF-104");
    lostItems.addItem("LF-215");
    lostItems.addItem("LF-330");

    cout << "List starts empty after additions: "
         << lostItems.isEmpty() << endl;
    */

    // --- STEP B3: Display and search stored codes ---
    /*
    lostItems.display();
    cout << "Contains LF-215: " << lostItems.contains("LF-215") << endl;
    cout << "Contains LF-999: " << lostItems.contains("LF-999") << endl;
    */

    return 0;
}

/*
 * Analysis Answers
 *
 * B4 — Appending LF-330
 * When addItem("LF-330") reaches its final assignment, which node does head point to,
 * which node does currentNode point to, and what link is created? Explain why head does
 * not change in this case.
 * Answer:
 *
 * B5 — Traversal and linear search
 * What traversal step do display and contains have in common? What is different about
 * the action each function performs at a node, and why may contains stop earlier?
 * Answer:
 *
 */
```

Compila y ejecuta:

```bash
g++ lost_item_list_example.cpp -o lost_item_list_example
./lost_item_list_example
```

**Qué deberías ver en B1:**

```text
List starts empty: true
```

<details>
<summary>Pregunta de verificación: ¿por qué <code>head</code> es <code>nullptr</code> en una lista vacía?</summary>

No existe un primer node. `head` necesita indicar explícitamente que no guarda la dirección de ningún node; `nullptr` representa esa ausencia de dirección.

</details>

### B2 — Crear y enlazar un node al final

Ahora quita únicamente los comentarios `/*` y `*/` que rodean el bloque `STEP B2` dentro de la clase y el bloque `STEP B2` dentro de `main`. Al hacerlo, esos dos bloques pasan a formar parte activa del programa.

La función sigue el proceso de la lecture: primero crea `newNode` fuera de la lista, inicializa sus data members y deja su `next` en `nullptr`. Luego distingue dos casos. Si no existe un primer node, `newNode` se convierte en `head`. Si ya hay nodes, `currentNode` comienza en `head` y avanza hasta el único node cuyo `next` es `nullptr`; ese node se enlaza con `newNode`.

Compila y ejecuta de nuevo.

**Qué deberías ver adicionalmente:**

```text
List starts empty after additions: false
```

<details>
<summary>Pregunta de verificación: después de agregar <code>LF-330</code>, ¿qué pointer sigue señalando el primer node y qué pointer termina señalando el último?</summary>

`head` sigue señalando el primer node, `LF-104`. Dentro de `addItem`, `currentNode` termina señalando `LF-330`, el último node. Cuando la función termina, `currentNode` deja de existir porque era un pointer local; `head` conserva la entrada a la lista.

</details>

### B3 — Recorrer y buscar sin mover `head`

Ahora quita únicamente los comentarios `/*` y `*/` que rodean los dos bloques `STEP B3`. Al hacerlo, se activan `display`, `contains` y sus pruebas dentro de `main`. `display` y `contains` comienzan ambos con `Node *currentNode = head;`. La diferencia es la acción que realizan durante el traversal: `display` imprime cada código y `contains` compara cada código con el valor solicitado.

Compila y ejecuta una última vez.

**Qué deberías ver adicionalmente:**

```text
Lost item: LF-104
Lost item: LF-215
Lost item: LF-330
Contains LF-215: true
Contains LF-999: false
```

<details>
<summary>Pregunta de verificación: ¿por qué <code>contains</code> puede ejecutar <code>return true</code> antes de llegar a <code>nullptr</code>?</summary>

Una coincidencia ya responde la pregunta de la función. No es necesario visitar los nodes restantes una vez que el código solicitado fue encontrado.

</details>

### B4 — Analizar el recorrido y los pointers

Al final de `lost_item_list_example.cpp` ya aparece un bloque de comentarios llamado `Analysis Answers`. Responde allí B4 y B5 después de ejecutar el programa completo. Estas preguntas no requieren código nuevo: requieren usar el ejemplo para explicar qué pointers existen durante una inserción y qué comparten `display` y `contains`.

Escribe tus respuestas en inglés dentro de los espacios que siguen a `Answer:`. No borres las preguntas ni respondas solamente con una palabra; justifica cada respuesta con los nodes y el flujo del programa.

💾 **Buen momento para el commit** — completaste un ejemplo guiado que construye, recorre y busca una linked list. Commit: `"Complete LostItemList guided example"`. Este mensaje identifica una unidad funcional completa y permite distinguir este avance del setup inicial.

## Parte C — TODO: representar `StudyGroupList` (20 min)

### Contexto

El Centro de Éxito Estudiantil confirma grupos de estudio durante la semana. No sabe cuántos grupos terminará registrando. De cada grupo necesita conservar un código, como `MATH-201-A`, y el salón donde se reunirá, como `315`.

Esta práctica usa la misma base de linked list, pero añade una decisión de diseño: la clase conservará `groupCount`, la cantidad de groups actualmente registrados. `groupCount` no reemplaza `head` ni un traversal; solo permite responder cuántos groups hay sin recorrer la lista. Por eso `groupCount` pertenece a la clase, no a cada node.

Crea un segundo archivo llamado `study_group_list.cpp` y copia este template. Resuelve **solo** los TODO marcados `(Part C)` por ahora. Ignora los TODOs de las Partes D y E.

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 9 - Linked Lists
 * Description: Practice - a linked list of study groups
 * Due date: [Date]
 */

#include <iostream>
#include <string>
using namespace std;

class StudyGroupList
{
private:
    // ===== Resolve these TODOs now (Part C) =====

    // TODO (Part C): Define a Node with string groupCode, int meetingRoom, and Node *next.
    // TODO (Part C): Declare a Node *head data member for the first node address.
    // TODO (Part C): Declare an int groupCount data member for the number of stored groups.

public:
    // TODO (Part C): Write a constructor that creates an empty list and sets groupCount to zero.

    // ===== Resolve these TODOs later (Part D) =====

    // TODO (Part D): Write addGroup. It receives a group code and room number, adds one node at the end, and updates groupCount.
    // TODO (Part D): Write displayGroups as a const member function.
    // TODO (Part D): Write size as a const member function that returns groupCount.

    // ===== Resolve these TODOs later (Part E) =====

    // TODO (Part E): Write containsGroup as a const member function that searches by group code.
};

int main()
{
    cout << boolalpha;

    // ===== Resolve these TODOs later (Part E) =====

    // TODO (Part E): Create a StudyGroupList object.
    // TODO (Part E): Add three dummy groups with different codes and room numbers.
    // TODO (Part E): Display every stored group and print the result of size().
    // TODO (Part E): Search for one stored group code and one missing group code; print both bool results.

    return 0;
}
```

### Antes de escribir

1. **C1.** ¿Qué información describe a un solo group y por eso debe estar dentro de `Node`?
2. **C2.** ¿Qué información describe la colección completa y por eso debe ser un data member de `StudyGroupList`, no de `Node`?
3. **C3.** ¿Qué valores deben tener `head` y `groupCount` en una lista recién creada?
4. **C4.** ¿Por qué `groupCount` permite saber la cantidad de groups sin sustituir el trabajo de `displayGroups` o `containsGroup`?

<details>
<summary>Ver orientación</summary>

Cada `Node` representa un group individual, así que debe guardar su código, salón y enlace `next`. `head` representa la entrada a toda la lista y `groupCount` representa una propiedad de toda la colección. Una lista nueva no tiene primer node y no contiene groups, por lo que necesita `head == nullptr` y `groupCount == 0`.

</details>

Después de completar la Parte C, compila. `main` todavía no produce resultados, pero el archivo debe compilar correctamente.

```bash
g++ study_group_list.cpp -o study_group_list
./study_group_list
```

💾 **Buen momento para el commit** — definiste la representación y el estado inicial de la nueva colección. Commit: `"Define StudyGroupList representation"`. Este commit separa el diseño de los nodes y el estado de la clase de las operaciones que se añadirán después.

## Parte D — TODO: agregar y mostrar grupos (25 min)

Ahora resuelve **solo** los TODO marcados `(Part D)` dentro de `StudyGroupList`. No resuelvas todavía los TODOs de la Parte E en `main` ni `containsGroup`.

`addGroup` recibe `newGroupCode` y `newMeetingRoom`. Debe crear un node dinámico, inicializar los tres data members del node y agregarlo al final de la lista. Cada vez que un node queda correctamente conectado a la lista, debe aumentar `groupCount` en uno.

`displayGroups` debe recorrer la lista con un pointer local, sin cambiar `head`. Para cada group, debe imprimir exactamente una línea descriptiva con su código y salón. `size` no necesita traversal: debe devolver el contador de la clase.

### Antes de escribir

1. **D1.** ¿En qué momento es seguro aumentar `groupCount`: antes de crear el node, después de inicializarlo o después de conectarlo a la lista? Explica por qué.
2. **D2.** Si la lista está vacía, ¿qué pointer debe recibir la dirección de `newNode`?
3. **D3.** Si la lista ya tiene groups, ¿qué condición identifica al último node durante el traversal?
4. **D4.** ¿Por qué `displayGroups` usa un pointer local en vez de avanzar `head`?
5. **D5.** ¿Por qué `size` puede devolver `groupCount` sin usar un loop?

<details>
<summary>Ver orientación</summary>

El contador debe cambiar después de que el nuevo node se conecte a la lista. Para una lista vacía, `head` recibe la dirección de `newNode`. Para una lista no vacía, el último node es aquel cuyo `next` es `nullptr`. `displayGroups` necesita preservar `head`, porque la clase debe recordar el primer node después de mostrar los groups.

</details>

Después de completar los métodos, compila otra vez. `main` sigue sin llamar las operaciones; por eso todavía no se espera output adicional. La meta de esta parte es que la clase compile con sus operaciones de agregado, traversal y conteo correctamente declaradas.

💾 **Buen momento para el commit** — agregaste las operaciones centrales para modificar y recorrer la colección. Commit: `"Add and display study groups"`. Es un punto útil del historial porque las operaciones ya compilan antes de añadir las pruebas del cliente.

## Parte E — TODO: buscar y probar `StudyGroupList` (15 min)

Ahora resuelve **solo** los TODO marcados `(Part E)`: `containsGroup` dentro de la clase y las pruebas de `main`.

`containsGroup` debe usar el mismo patrón de traversal que `displayGroups`, pero su acción es comparar `currentNode->groupCode` con el código solicitado. Si encuentra una coincidencia, puede devolver `true` inmediatamente. Si llega a `nullptr`, debe devolver `false`.

En `main`, usa datos dummy y demuestra estos resultados:

1. **E1.** Crear un objeto `StudyGroupList`.
2. **E2.** Agregar tres groups con códigos y salones distintos.
3. **E3.** Mostrar los tres groups y el valor que devuelve `size()`.
4. **E4.** Buscar un código existente y uno que no existe; ambos resultados deben imprimirse con etiquetas descriptivas en inglés.

### Antes de escribir

1. **E5.** ¿Qué condición debe controlar el `while` de `containsGroup` para evitar usar `->` con `nullptr`?
2. **E6.** ¿Qué ocurre con `head` mientras `containsGroup` avanza `currentNode`?
3. **E7.** ¿Por qué `containsGroup` puede terminar antes de visitar todos los nodes, pero `displayGroups` normalmente no?
4. **E8.** ¿Qué diferencia existe entre `groupCount` y el número de nodes que un traversal ya visitó?

<details>
<summary>Ver orientación</summary>

El loop debe continuar mientras `currentNode != nullptr`. `head` no cambia porque el traversal usa otro pointer. Una coincidencia responde la pregunta de búsqueda, mientras que mostrar la lista requiere visitar cada node. `groupCount` es el total que la clase mantiene; un traversal solo conoce cuántos nodes ha visitado si se programa un contador para ello.

</details>

Una posible salida, usando datos dummy de tu elección, debe tener esta forma:

```text
Study group: MATH-201-A | Room: 315
Study group: COEN-2220-B | Room: 204
Study group: CHEM-105-C | Room: 118
Registered groups: 3
Contains COEN-2220-B: true
Contains PHYS-110-A: false
```

Compila y ejecuta antes de continuar:

```bash
g++ study_group_list.cpp -o study_group_list
./study_group_list
```

💾 **Buen momento para el commit** — las pruebas de `main` demuestran que la clase agrega, recorre, cuenta y busca groups correctamente. Commit: `"Search and test study groups"`. Este mensaje permite identificar en el historial cuándo se añadió evidencia de uso desde el código cliente.

## Parte F — Revisión, evaluación y entrega (15 min)

Antes de entregar, verifica:

- `lost_item_list_example.cpp` compila con `g++ lost_item_list_example.cpp -o lost_item_list_example`.
- En el ejemplo guiado, los bloques B2 y B3 ya no están comentados y producen el output esperado.
- El bloque final `Analysis Answers` de `lost_item_list_example.cpp` responde B4 y B5 en inglés con explicaciones propias.
- `study_group_list.cpp` compila con `g++ study_group_list.cpp -o study_group_list`.
- `StudyGroupList` usa `Node *head` y cada `Node` guarda código, salón y `next`.
- `addGroup` funciona cuando la lista está vacía y cuando ya contiene nodes.
- `groupCount` empieza en cero y aumenta una vez por cada group conectado correctamente.
- `displayGroups` y `containsGroup` usan un pointer local y no modifican `head`.
- La búsqueda demuestra un resultado `true` y otro `false`.
- Todos los datos son dummy; los identificadores, comentarios y mensajes del programa están en inglés.
- `.gitignore` evita que los ejecutables y archivos objeto aparezcan en Source Control.

### Criterios de evaluación (100 puntos)

| Criterio | Puntos |
| --- | ---: |
| Setup, `.gitignore`, headers y commits claros | 10 |
| Ejemplo guiado `LostItemList` completo, ejecutable y con respuestas B4–B5 | 20 |
| Representación de `StudyGroupList`: `Node`, `head`, `groupCount` y constructor | 20 |
| `addGroup` correcto para lista vacía y no vacía; actualización correcta de `groupCount` | 25 |
| `displayGroups`, `size` y `containsGroup` correctos | 20 |
| Pruebas con datos dummy y output descriptivo | 5 |

## Entregable del laboratorio

Envía al profesor el enlace de tu repositorio `lab9-linked-lists` en GitHub, con:

- `lost_item_list_example.cpp`, con los tres pasos activados y ejecutables.
- `lost_item_list_example.cpp`, con el bloque final `Analysis Answers` completado para B4–B5.
- `study_group_list.cpp`, con todos los TODOs de las Partes C, D y E resueltos.
- Un `.gitignore` funcionando, sin ejecutables ni archivos objeto en Source Control.
- Al menos **4 commits** visibles: setup, ejemplo guiado, operaciones de `StudyGroupList` y pruebas/búsqueda.
- Headers estándar completos con tu nombre y fecha reales en ambos archivos fuente.

## Próxima sesión

En la Semana 10 estudiarán Stacks & Queues como ADTs. Las ideas de `head`, nodes y recorrido seguirán siendo útiles, pero una queue agregará reglas formales sobre el orden en que se insertan y atienden los elementos.
