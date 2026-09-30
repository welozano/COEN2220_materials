# Lab 7 — Abstract Data Types (ADT)
## COEN 2220 — Programming 2

**Instructor:** Wilson Lozano

**Duración:** 110 min (sesión de 2 horas crédito nominales)  
**Precede a:** Semana 8 — Time Complexity (Big-O)  
**Requisitos:** Lecture de Semana 6; Labs 1–6 completados (clases, encapsulation, templates y class templates)  
**Basado en:** Gaddis, *Starting Out with C++*, 8va ed., conexión conceptual con Cap. 16. Exceptions y STL no forman parte de este lab.

---

## Objetivos

Al finalizar este laboratorio, el estudiante podrá:

1. Explicar que un Abstract Data Type (ADT) es un contrato conceptual y no una keyword de C++.
2. Describir los datos, operaciones, resultados y precondiciones de un ADT simple.
3. Implementar un ADT mediante una clase normal con data members `private` e interfaz `public`.
4. Distinguir entre el código que implementa un ADT y el código cliente que lo usa.
5. Explicar por qué una implementación interna puede cambiar sin obligar a cambiar el código cliente que usa la misma interfaz.

## Parte A — Preparar la carpeta y el repositorio (5 min)

> ⚠️ **Este paso es el que más se presta a error — léelo con cuidado antes de hacer clic en nada.** Cada lab tiene su propio repositorio. No inicialices Git en la carpeta padre `COEN2220` ni dentro de la carpeta de otro lab.

1. **A1.** Abre VS Code o VSCodium y selecciona `File → Open Folder...`.
2. **A2.** Navega a tu carpeta padre del curso, por ejemplo `COEN2220`. Si todavía no existe, créala **desde este diálogo** en un lugar de fácil acceso.
3. **A3.** Entra dentro de la carpeta padre. Sin salir del diálogo, crea una carpeta nueva llamada `lab7-adt`, hermana de las carpetas de los labs anteriores.
4. **A4.** Abre específicamente `lab7-adt` con **Open** / **Select Folder**.

**Cómo saber si abriste la carpeta correcta:** el Explorer debe mostrar `LAB7-ADT` como raíz. Si muestra `COEN2220` o el nombre de otro lab, usa `File → Close Folder` y repite los pasos.

> **¿Por qué importa tanto esto?** Inicializar el repositorio en la carpeta padre mezcla todos los labs en un solo historial. Inicializarlo dentro de un lab anterior mezcla archivos y entregas. Un repositorio separado permite evaluar esta práctica como una unidad de trabajo independiente.

Abre Source Control (`Ctrl+Shift+G` / `Cmd+Shift+G`) y selecciona **Initialize Repository**. Crea `.gitignore` en la raíz con este contenido:

```gitignore
# Compiled executables (Windows)
*.exe

# Compiled executables (Mac/Linux)
book_hold_log_example
study_session_log

# Intermediate object files
*.o

# Editor configuration
.vscode/
```

Haz el commit inicial y publica el repositorio con el flujo ya practicado: stage → commit → **Publish Branch**. Usa el mensaje `"Initial setup and gitignore"`.

💾 **Por qué este commit importa:** guarda una base limpia antes de escribir código. Si aparece un problema después, el historial muestra que la carpeta, el repositorio y `.gitignore` ya estaban correctos.

## Parte B — Ejemplo guiado: un ADT implementado por una clase (30 min)

### Contexto

La biblioteca universitaria recibe solicitudes para reservar libros. El personal necesita un programa pequeño que pueda registrar códigos de libros solicitados, indicar cuántas solicitudes se han registrado, saber si todavía no hay ninguna y verificar si un código particular ya aparece en el registro.

Antes de escribir C++, el diseñador puede definir qué debe ofrecer ese registro. Esa descripción se llama **BookHoldLog ADT**. El ADT no dice si los códigos se guardarán en un array, una linked list o algún otro mecanismo; solo promete los datos que el registro representa y las operaciones que el resto del programa podrá usar.

Después, un programador escribe una clase `BookHoldLog` que cumple ese contrato. En este ejemplo se usará un array de capacidad fija porque ya conocen arrays. Esa es una **decisión de implementación**, no la definición del ADT. Finalmente, `main` será el **código cliente**: usa las operaciones públicas sin acceder al array ni al contador internos.

La relación entre las tres ideas es:

```text
BookHoldLog ADT (contract) → BookHoldLog class (implementation) → main (client code)
```

Crea un archivo llamado `book_hold_log_example.cpp`. Construirás el ejemplo en cuatro pasos: primero el contrato, luego una implementación inicial, después la operación para agregar y finalmente la operación para buscar. En cada paso recibirás el bloque resuelto que debes copiar donde aparece el marcador.

### B1 — Definir el contrato del ADT

El primer paso todavía **no** implementa una clase. El diseñador debe comenzar por decidir qué representa el registro y qué operaciones prometerá al resto del programa. El comentario `BookHoldLog ADT` expresa ese contrato. Habla de “códigos de libros”, “agregar” y “verificar”, pero no revela todavía los nombres de data members ni cómo se recorrerán los datos. Esa separación permite que el cliente conozca qué puede pedir sin depender de detalles internos.

Crea el archivo con su header, los includes y el contrato siguiente. Por ahora no lo compiles como programa ejecutable: todavía contiene solo la especificación conceptual; aún no tiene una clase ni `main`.

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 7 - Abstract Data Types
 * Description: Guided example - ADT contract, implementation, and client code
 * Due date: [Date]
 */

#include <iostream>
#include <string>
using namespace std;

/*
 * BookHoldLog ADT
 *
 * Data:
 * A sequence of up to four book hold IDs.
 *
 * Operations:
 * addHold(id): Adds one book hold ID when space remains; returns whether it was added.
 * contains(id): Reports whether an equal book hold ID is stored.
 * size(): Returns the number of stored hold IDs.
 * isEmpty(): Reports whether no hold IDs are stored.
 */
```

<details>
<summary>Pregunta de verificación: ¿cuál parte del código es el ADT y cuál parte es la implementación?</summary>

En este paso, el comentario titulado `BookHoldLog ADT` es la única parte creada. Es la especificación conceptual: describe qué datos y operaciones promete el registro. La clase que implementará el contrato se añadirá en B2.

</details>

### B2 — Implementar el registro vacío mediante una clase

Ahora el programador debe escoger **cómo** cumplirá el contrato. Para este primer programa, usará un array de cuatro posiciones llamado `holdIds` y una variable `count`. El array, el contador y la constante `CAPACITY` son detalles de implementación; por eso se declaran `private`. Otra implementación podría usar una linked list más adelante, pero el cliente seguiría necesitando las mismas operaciones públicas.

Agrega el siguiente código **después** del comentario del ADT. Esta clase concreta implementa por ahora las operaciones que consultan un registro vacío: `size()` e `isEmpty()`. También agrega `main`, el código cliente que crea un objeto y usa esas operaciones públicas.

```cpp
class BookHoldLog
{
private:
    static const int CAPACITY = 4;
    string holdIds[CAPACITY]; // The implementation stores IDs in a fixed array.
    int count;                // The implementation tracks used array positions.

public:
    BookHoldLog()
    {
        count = 0;            // A new log begins with no stored hold IDs.
    }

    int size() const
    {
        return count;         // Client code may ask for the count, but cannot change it.
    }

    bool isEmpty() const
    {
        return count == 0;    // The log is empty exactly when no IDs are stored.
    }

    // --- B3: Add one book hold ID ---
    // TODO (B3): Paste addHold here.

    // --- B4: Search stored book hold IDs ---
    // TODO (B4): Paste contains here.
};

int main()
{
    cout << boolalpha;        // Print bool results as true or false.

    BookHoldLog holds;        // Client code creates one empty log object.

    cout << "Stored holds: " << holds.size() << endl;
    cout << "Log is empty: " << holds.isEmpty() << endl;

    // --- B3: Add book hold IDs through the public interface ---
    // TODO (B3): Paste addHold test calls here.

    // --- B4: Search through the public interface ---
    // TODO (B4): Paste contains test calls here.

    return 0;
}
```

Compila y ejecuta:

```bash
g++ book_hold_log_example.cpp -o book_hold_log_example
./book_hold_log_example
```

**Qué deberías ver:**

```text
Stored holds: 0
Log is empty: true
```

### B3 — Implementar la operación de agregar

El contrato promete una operación `addHold(id)`. Para cumplirla, la clase necesita decidir qué ocurre cuando el array ya está lleno y qué ocurre cuando queda espacio. Esa lógica corresponde a una member function porque debe leer y modificar el estado `private` de **un objeto específico**.

La operación devuelve `bool`. El código cliente necesita saber si el registro aceptó la solicitud; por ejemplo, una quinta solicitud no se puede guardar cuando la capacidad es cuatro. El parámetro es `const string& holdId`: recibe el código sin copiarlo ni modificarlo.

Esta sería la implementación de la member function:

```cpp
bool addHold(const string& holdId)
{
    if (count == CAPACITY)
    {
        return false;     // Do not write outside the fixed array capacity.
    }

    holdIds[count] = holdId;
    count++;
    return true;
}
```

Después, reemplaza el TODO de `B3` dentro de `main` con estas llamadas. El cliente solicita la operación mediante el objeto `holds`; no escribe directamente en `holdIds` ni altera `count`.

```cpp
holds.addHold("BK-104");
holds.addHold("BK-215");

cout << "Stored holds: " << holds.size() << endl;
```

Compila y ejecuta de nuevo.

**Qué deberías ver adicionalmente:**

```text
Stored holds: 2
```

<details>
<summary>Pregunta de verificación: ¿por qué <code>main</code> no debe escribir <code>holds.count++</code> directamente?</summary>

`count` es un detalle privado de la implementación. Si el cliente pudiera cambiarlo directamente, podría dejar el objeto en un estado inválido, por ejemplo indicando cinco valores cuando el array solo tiene cuatro posiciones. `addHold` controla esa regla antes de modificar el estado.

</details>

### B4 — Implementar la operación de búsqueda

Ahora el contrato promete `contains(id)`: el cliente puede preguntar si ya se registró un código. La clase debe revisar únicamente las posiciones que realmente contienen datos, desde la primera hasta la posición anterior a `count`. Si encuentra un código igual, puede responder `true` inmediatamente; si termina la búsqueda sin encontrarlo, responde `false`.

Como la búsqueda solo consulta el objeto y no cambia sus datos, la member function termina en `const`. Esta sería la implementación:

```cpp
bool contains(const string& holdId) const
{
    for (int index = 0; index < count; index++)
    {
        if (holdIds[index] == holdId)
        {
            return true;  // Stop as soon as one matching ID is found.
        }
    }

    return false;         // No stored ID matched the requested value.
}
```

Después, reemplaza el TODO de `B4` dentro de `main` con este código. Las dos preguntas usan la misma interfaz pública, pero producen resultados distintos porque solo uno de los códigos fue agregado.

```cpp
cout << "Contains BK-215: " << holds.contains("BK-215") << endl;
cout << "Contains BK-310: " << holds.contains("BK-310") << endl;
```

Compila y ejecuta otra vez.

**Qué deberías ver adicionalmente:**

```text
Contains BK-215: true
Contains BK-310: false
```

El cliente no necesita saber que la implementación usa un array ni que `contains` usa un `for`. Solo necesita saber que `contains` recibe un código y devuelve si está registrado. Más adelante, una linked list podría reemplazar el array; si la clase conserva las mismas operaciones y resultados, el código cliente puede seguir usando `holds.contains(...)` sin cambios.

💾 **Buen momento para el commit** — completaste un ejemplo que separa claramente contrato, implementación y código cliente. Commit: `"Complete BookHoldLog ADT example"`.

## Parte C — TODO: diseñar el contrato de `StudySessionLog` (15 min)

### Contexto

El Centro de Tutorías de la universidad quiere ofrecer a cada estudiante un resumen de sus sesiones de estudio durante la semana. El programa debe registrar la duración de cada sesión en minutos, indicar cuántas sesiones hay, calcular el total de minutos acumulados, identificar la sesión más larga y verificar si todavía no se ha registrado ninguna sesión.

Antes de decidir cómo almacenar las duraciones, el diseñador define el **StudySessionLog ADT**. Este contrato debe describir qué representa el registro y qué puede hacer el código cliente. No debe mencionar un array, `CAPACITY`, `count`, un `for` ni otro detalle de implementación: esos detalles pertenecen a la clase concreta que escribirás después.

Crea un segundo archivo llamado `study_session_log.cpp`. Copia el siguiente template. Resuelve **solo** los TODO marcados `(Part C)` dentro del comentario `StudySessionLog ADT`. Ignora los TODOs de las Partes D y E por ahora.

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 7 - Abstract Data Types
 * Description: ADT contract, implementation, and client code practice
 * Due date: [Date]
 */

#include <iostream>
using namespace std;

/*
 * StudySessionLog ADT
 *
 * Data:
 * TODO (Part C): Describe the study session durations managed by this ADT.
 *
 * Operations:
 * TODO (Part C): Describe addSession(minutes), including its result when the log cannot accept another session.
 * TODO (Part C): Describe totalMinutes().
 * TODO (Part C): Describe longestSession() and its precondition.
 * TODO (Part C): Describe size() and isEmpty().
 */
class StudySessionLog
{
private:
    // ===== Resolve these TODOs later (Part D) =====

    // TODO (Part D): Add a fixed capacity constant of four study sessions.
    // TODO (Part D): Add an int array named sessionMinutes for the stored session durations.
    // TODO (Part D): Add an int that tracks how many study sessions are stored.

public:
    // TODO (Part D): Write a constructor that creates an empty log.
    // TODO (Part D): Write addSession. It receives minutes and reports whether the session was stored.
    // TODO (Part D): Write totalMinutes as a const member function.
    // TODO (Part D): Write longestSession as a const member function.
    // TODO (Part D): Write size as a const member function.
    // TODO (Part D): Write isEmpty as a const member function.
};

int main()
{
    // ===== Resolve these TODOs later (Part E) =====

    // TODO (Part E): Create a StudySessionLog object and print whether it starts empty.
    // TODO (Part E): Add four dummy session durations and attempt to add a fifth.
    // TODO (Part E): Print the number of stored sessions and whether the fifth session was accepted.
    // TODO (Part E): Print the total minutes and the longest stored session.
    // TODO (Part E): Print descriptive English labels for all results.

    return 0;
}
```

### Antes de escribir

1. **C1.** ¿Qué representa la información del ADT para el Centro de Tutorías, sin describir cómo se almacenará?
2. **C2.** ¿Qué debe prometer `addSession(minutes)` al código cliente cuando ya no puede aceptar otra sesión?
3. **C3.** ¿Qué debe devolver `totalMinutes()` y qué información necesita considerar para producir ese resultado?
4. **C4.** ¿Qué precondición necesita `longestSession()` y por qué?
5. **C5.** ¿Por qué `size()` e `isEmpty()` no necesitan modificar el registro?

<details>
<summary>Ver orientación</summary>

El ADT debe hablar de duraciones de sesiones de estudio y de las operaciones disponibles para el cliente. `addSession` debe comunicar si pudo registrar una sesión; `totalMinutes` debe comunicar la suma de las duraciones almacenadas. `longestSession` necesita al menos una sesión para poder identificar una duración mayor. `size` e `isEmpty` observan el estado, por lo que no necesitan cambiarlo.

</details>

💾 **Buen momento para el commit** — definiste el contrato antes de escribir la implementación. Commit: `"Define StudySessionLog ADT contract"`.

## Parte D — TODO: implementar `StudySessionLog` (30 min)

El contrato ya indica **qué** debe poder hacer el registro. Ahora escribirás una implementación concreta con una clase normal llamada `StudySessionLog`. Para este lab, el Centro de Tutorías guardará internamente hasta cuatro duraciones de sesiones en un array de `int`.

Resuelve **solo** los TODO marcados `(Part D)` dentro de `StudySessionLog`. No escribas aún las pruebas de `main`; esas pertenecen a la Parte E.

### Antes de escribir

1. **D1.** ¿Qué data members son detalles de implementación y por qué deben ser `private`?
2. **D2.** ¿Qué estado debe establecer el constructor para que un registro nuevo esté vacío?
3. **D3.** ¿Qué debe ocurrir antes de guardar una duración nueva si la capacidad ya está llena?
4. **D4.** Para `totalMinutes`, ¿qué debe representar una variable acumuladora antes y después de recorrer las sesiones almacenadas?
5. **D5.** Para `longestSession`, ¿qué valor almacenado puede usarse como punto de partida de la comparación y por qué exige la precondición del contrato?
6. **D6.** ¿Por qué `totalMinutes`, `longestSession`, `size` e `isEmpty` deben ser `const`?

Tu implementación debe cumplir el contrato de la Parte C:

- `addSession` agrega una duración cuando queda espacio y comunica mediante `bool` si tuvo éxito;
- `totalMinutes` suma las duraciones de las sesiones almacenadas;
- `longestSession` devuelve la duración más alta entre las sesiones almacenadas;
- `size` devuelve cuántas sesiones se almacenaron;
- `isEmpty` indica si no se ha almacenado ninguna sesión.

Después de completar la clase, compila. Todavía no se espera output porque `main` sigue vacío, pero el archivo debe compilar correctamente.

```bash
g++ study_session_log.cpp -o study_session_log
./study_session_log
```

💾 **Buen momento para el commit** — implementaste una clase concreta que cumple el contrato del ADT. Commit: `"Implement StudySessionLog ADT"`.

## Parte E — TODO: usar el ADT como código cliente (20 min)

Ahora resuelve **solo** los TODO marcados `(Part E)` dentro de `main()`. El código cliente representa al programa que prepara un resumen para el Centro de Tutorías. Debe crear un objeto `StudySessionLog` y usar únicamente la interfaz pública; no debe tocar el array ni el contador directamente.

Usa datos dummy. Debes demostrar los siguientes resultados:

1. **E1.** Un registro nuevo comienza vacío.
2. **E2.** Se pueden agregar cuatro sesiones, por ejemplo de `45`, `60`, `35` y `90` minutos.
3. **E3.** Un intento de agregar una quinta sesión devuelve `false`.
4. **E4.** `totalMinutes()` devuelve la suma de las cuatro sesiones y `longestSession()` devuelve la duración más alta.

Antes de escribir, responde:

1. **E5.** ¿Por qué `main` puede llamar `sessionLog.addSession(...)` pero no debe acceder a `sessionLog.sessionMinutes`?
2. **E6.** Si en una semana futura cambias el array por una linked list, ¿qué llamadas de `main` deberían mantenerse si el ADT sigue siendo el mismo?
3. **E7.** ¿Qué evidencia de output demostrará que el contrato se cumplió?

Una salida posible, usando los datos sugeridos, es:

```text
Log starts empty: true
Fifth session accepted: false
Stored sessions: 4
Total study minutes: 230
Longest session: 90
```

<details>
<summary>Ver orientación</summary>

`sessionMinutes` es un detalle `private` de la implementación. El código cliente debe usar `addSession`, `totalMinutes`, `longestSession`, `size` e `isEmpty`. Si cambia la representación interna, las llamadas públicas como `sessionLog.addSession(45)` y `sessionLog.totalMinutes()` pueden mantenerse, porque expresan el contrato del ADT y no la implementación actual.

</details>

💾 **Buen momento para el commit** — demostraste que código cliente puede usar el ADT sin depender de cómo se almacenan las sesiones. Commit: `"Test StudySessionLog client code"`.

## Parte F — Revisión, evaluación y entrega (10 min)

Antes de entregar, verifica:

- `book_hold_log_example.cpp` compila con `g++ book_hold_log_example.cpp -o book_hold_log_example`.
- `study_session_log.cpp` compila con `g++ study_session_log.cpp -o study_session_log`.
- El comentario `StudySessionLog ADT` describe datos y operaciones sin exponer el array, `count` u otros detalles de implementación.
- Todos los TODOs de las Partes C, D y E están resueltos o eliminados.
- Los data members de `StudySessionLog` son `private` y el código de `main` usa solo member functions `public`.
- Los nombres, comentarios y mensajes del código están en inglés.
- Los datos de prueba son dummy y `.gitignore` evita que los ejecutables aparezcan en Source Control.

### Criterios de evaluación (100 puntos)

| Criterio | Puntos |
| --- | ---: |
| Contrato `StudySessionLog ADT` claro: datos, operaciones y precondición descritos sin detalles de implementación | 20 |
| Clase `StudySessionLog` con encapsulation, constructor y manejo correcto de capacidad | 30 |
| Implementación correcta de `totalMinutes`, `longestSession`, `size` e `isEmpty` | 20 |
| Código cliente y pruebas de registro vacío, capacidad completa, total y máximo | 20 |
| Dos archivos compilables, encabezados, datos dummy, `.gitignore` y commits claros | 10 |

## Entregable del laboratorio

Envía al profesor el enlace de tu repositorio `lab7-adt` en GitHub, con:

- `book_hold_log_example.cpp`, completado durante el ejemplo guiado.
- `study_session_log.cpp`, con todos los TODOs de las Partes C, D y E resueltos.
- Al menos **5 commits** visibles: setup, ejemplo guiado, contrato ADT, implementación y pruebas.
- Un `.gitignore` funcionando, sin ejecutables ni archivos objeto en el repositorio.
- Encabezados estándar completos con tu nombre y fecha reales en ambos archivos fuente.

## Próxima sesión

La próxima lecture introduce **Time Complexity (Big-O)**. El contrato de un ADT permite preguntarnos qué operaciones ofrece una estructura; Big-O permitirá comparar cuánto trabajo cuesta realizar esas operaciones cuando la cantidad de datos crece.
