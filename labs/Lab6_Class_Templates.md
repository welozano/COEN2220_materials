# Lab 6 — Class Templates
## COEN 2220 — Programming 2

**Instructor:** Wilson Lozano

**Duración:** 110 min (sesión de 2 horas crédito nominales)  
**Precede a:** Lab 7 — Abstract Data Types (ADT)  
**Requisitos:** Lecture de Semana 6; Labs 1–5 completados (Git/GitHub, clases, inheritance, polymorphism y templates)  
**Basado en:** Gaddis, *Starting Out with C++*, 8va ed., Cap. 16 — secciones 16.3–16.4 (Where to Start When Defining Templates; Class Templates). Exceptions y STL no forman parte de este lab.

---

## Objetivos

Al finalizar este laboratorio, el estudiante podrá:

1. Crear objetos de una class template con tipos concretos como `int` y `double`.
2. Implementar una class template de capacidad fija con una interfaz pública clara.
3. Definir una member function de class template fuera de la declaración de la clase.
4. Implementar y probar una operación genérica que requiere que el tipo `T` soporte `==`.
5. Documentar el contrato básico de un ADT y relacionarlo con una class template concreta.

## Parte A — Preparar la carpeta y el repositorio (5 min)

> ⚠️ **Este paso es el que más se presta a error — léelo con cuidado antes de hacer clic en nada.** Cada lab tiene su propio repositorio. No inicialices Git en la carpeta padre `COEN2220` ni dentro de la carpeta de otro lab.

1. Abre VS Code o VSCodium y selecciona `File → Open Folder...`.
2. Navega a tu carpeta padre del curso, por ejemplo `COEN2220`. Si todavía no existe, créala **desde este diálogo** en un lugar de fácil acceso.
3. Entra dentro de la carpeta padre. Sin salir del diálogo, crea una carpeta nueva llamada `lab6-class-templates`, hermana de las carpetas de los labs anteriores.
4. Abre específicamente `lab6-class-templates` con **Open**/ **Select Folder**.

**Cómo saber si abriste la carpeta correcta:** el Explorer debe mostrar `LAB6-CLASS-TEMPLATES` como raíz. Si muestra `COEN2220` o el nombre de otro lab, usa `File → Close Folder` y repite los pasos.

> **¿Por qué importa tanto esto?** Inicializar el repositorio en la carpeta padre mezcla todos los labs en un solo historial. Inicializarlo dentro de un lab anterior mezcla archivos de temas distintos. Un repositorio separado permite revisar y entregar esta unidad de trabajo por separado.

Abre Source Control (`Ctrl+Shift+G` / `Cmd+Shift+G`) y selecciona **Initialize Repository**. Crea `.gitignore` en la raíz con este contenido:

```gitignore
# Compiled executables (Windows)
*.exe

# Compiled executables (Mac/Linux)
workshop_log_example
metric_log

# Intermediate object files
*.o

# Editor configuration
.vscode/
```

Haz el commit inicial y publica el repositorio con el flujo ya practicado: stage → commit → **Publish Branch**. Usa el mensaje `"Initial setup and gitignore"`.

💾 **Por qué este commit importa:** guarda una base limpia antes de escribir código. Si aparece un problema después, el historial muestra que la carpeta, el repositorio y `.gitignore` ya estaban correctos.

## Parte B — Ejemplo guiado: `WorkshopLog<T>` (25 min)

### Contexto

La oficina de actividades estudiantiles organiza talleres durante el semestre. Al terminar cada taller, el personal quiere conservar dos historiales separados para consultar después:

- la cantidad de asistentes de cada taller, que se guarda como `int`;
- el *rating* promedio recibido por cada taller, que se guarda como `double`.

Primero, alguien podría crear una clase `AttendanceLog` para las cantidades enteras. Después necesitaría una clase `RatingLog` para los valores decimales. Ambas tendrían la misma responsabilidad: empezar vacías, agregar una medición, indicar cuántas mediciones guardan y verificar si no tienen datos. La única diferencia real sería el tipo del valor almacenado.

Antes de duplicar esas dos clases, el diseñador identifica el patrón común: ambas son registros de una métrica de talleres. Por eso crea `WorkshopLog<T>`. El nombre `WorkshopLog` comunica el propósito del registro; el type parameter `T` permite decidir después si el registro guarda asistentes (`int`) o *ratings* (`double`). La clase no representa un taller completo: representa una colección de valores asociados a talleres.

Crea un archivo llamado `workshop_log_example.cpp`. No pegues el código y lo ejecutes una sola vez: constrúyelo en los tres pasos indicados. El programa inicial contiene marcadores TODO para los Pasos 2 y 3. En cada paso recibirás el bloque resuelto que debes copiar exactamente donde aparece el marcador; no tendrás que descomentar bloques de código.

### Paso 1 — Declarar la class template y crear el primer objeto

El primer paso todavía no registra asistentes. Su propósito es construir la base de la clase y comprobar que el programa puede crear un registro vacío. La pregunta de diseño es: “si hoy el registro guarda asistentes y mañana guarda *ratings*, ¿qué parte debe cambiar?” La respuesta es el tipo de cada valor guardado, no la capacidad ni las operaciones del registro.

Por eso la clase declara `T values[CAPACITY]`. Cuando se cree `WorkshopLog<int>`, ese arreglo se convierte en un arreglo de `int`. El member variable `count` no usa `T` porque siempre representa una cantidad de posiciones utilizadas; por eso sigue siendo `int`.

El constructor establece el estado inicial del objeto: no hay valores almacenados, así que `count` comienza en cero. Las operaciones `size` e `isEmpty` permiten consultar ese estado sin modificarlo. Por ahora, deja los TODOs de los Pasos 2 y 3 como comentarios.

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 6 - Class Templates
 * Description: Guided example - a generic workshop metric log
 * Due date: [Date]
 */

#include <iostream>
using namespace std;

template <typename T>
class WorkshopLog
{
private:
    static const int CAPACITY = 4;
    T values[CAPACITY];       // Store values of the selected type.
    int count;                // Track how many positions contain data.

public:
    WorkshopLog()
    {
        count = 0;            // A new log starts with no stored values.
    }

    int size() const
    {
        return count;         // Report how many values are currently stored.
    }

    bool isEmpty() const
    {
        return count == 0;    // A log is empty only when it stores no values.
    }

    // --- STEP 2: Add a generic value ---
    // TODO (Step 2): Paste the add member function here.

    // --- STEP 3: Store a second metric type ---
    // TODO (Step 3): Paste the get member function here.
};

int main()
{
    cout << boolalpha;        // Print bool values as true or false.

    WorkshopLog<int> attendanceCounts;

    cout << "Attendance records: " << attendanceCounts.size() << endl;
    cout << "Log is empty: " << attendanceCounts.isEmpty() << endl;

    // --- STEP 2: Add attendance counts ---
    // TODO (Step 2): Paste the attendance test code here.

    // --- STEP 3: Create a log for ratings ---
    // TODO (Step 3): Paste the rating test code here.

    return 0;
}
```

Antes de compilar, identifica estas tres ideas dentro del código:

- `template <typename T>` crea el molde de la clase; todavía no elige un tipo concreto.
- `WorkshopLog<int> attendanceCounts;` crea una versión concreta del molde para guardar asistentes.
- `attendanceCounts.size()` y `attendanceCounts.isEmpty()` son llamadas de código cliente: usan la interfaz pública, sin tocar `values` ni `count`.

Compila y ejecuta:

```bash
g++ workshop_log_example.cpp -o workshop_log_example
./workshop_log_example
```

**Qué deberías ver:**

```text
Attendance records: 0
Log is empty: true
```

<details>
<summary>Pregunta de verificación: ¿qué tipo representa <code>T</code> para <code>attendanceCounts</code>?</summary>

Para el objeto `attendanceCounts`, `T` representa `int`. Por eso el arreglo `values` de ese objeto almacena cantidades enteras de asistentes.
</details>

### Paso 2 — Agregar valores del tipo seleccionado

Un registro vacío todavía no resuelve la necesidad de la oficina: debe poder guardar la cantidad de asistentes al terminar cada taller. Esa responsabilidad corresponde a una member function `add`, porque `add` necesita modificar los data members privados `values` y `count` de un objeto específico.

Antes de insertar el método, observa su lógica:

1. Verifica primero si ya se usaron las cuatro posiciones disponibles.
2. Si el registro está lleno, devuelve `false` y no modifica el objeto.
3. Si hay espacio, guarda el valor en la siguiente posición libre.
4. Aumenta `count` para que el objeto recuerde que ahora contiene un valor adicional.
5. Devuelve `true` para informar que la operación tuvo éxito.

Esta sería la implementación de la member function:

```cpp
bool add(const T& value)
{
    if (count == CAPACITY)
    {
        return false;     // Reject a value when the fixed array is full.
    }

    values[count] = value;
    count++;
    return true;
}
```

El parámetro es `const T& value`. `T` permite que el mismo método acepte un `int` para asistentes o un `double` para *ratings*. La referencia evita una copia innecesaria y `const` deja claro que `add` no debe cambiar el valor recibido.

Después, reemplaza el TODO de `STEP 2` dentro de `main` con estas llamadas. Cada llamada pide al objeto `attendanceCounts` que ejecute su propia versión de `add`; no escribe directamente en el arreglo privado.

```cpp
attendanceCounts.add(32);
attendanceCounts.add(47);

cout << "Attendance records: " << attendanceCounts.size() << endl;
```

Compila y ejecuta de nuevo.

**Qué deberías ver adicionalmente:**

```text
Attendance records: 2
```

Para el objeto `attendanceCounts`, el compilador trata `const T& value` como una referencia constante a `int`. Por eso ambos valores agregados son cantidades enteras y el resultado de `size()` cambia de 0 a 2.

<details>
<summary>Pregunta de verificación: ¿qué debe devolver <code>add</code> si el arreglo ya contiene cuatro valores?</summary>

Debe devolver `false`. La capacidad es fija y el método no puede escribir fuera de los límites del arreglo.
</details>

### Paso 3 — Crear otra versión de la misma class template

La oficina no solo necesita guardar una métrica; también debe poder consultar una medición específica. Esa es la responsabilidad de `get`. Como el método no cambia el registro, se declara `const`. Su precondición indica que el código cliente debe pedir una posición que realmente exista; por ejemplo, no puede pedir la posición 4 si solo se guardaron dos valores.

El return type de `get` es `T`. Esto es importante: para `WorkshopLog<int>`, `get` devuelve un `int`; para `WorkshopLog<double>`, devuelve un `double`. Reemplaza el TODO de `STEP 3` dentro de la clase con este método:

```cpp
T get(int index) const
{
    // Precondition: index is between 0 and count - 1.
    return values[index];
}
```

Ahora se puede demostrar por qué se creó una class template en vez de una clase solamente para asistentes. Reemplaza el TODO de `STEP 3` dentro de `main` con el siguiente código. Crea un segundo objeto, esta vez con `T` igual a `double`, pero usa exactamente las mismas operaciones `add` y `get`.

```cpp
WorkshopLog<double> workshopRatings;

workshopRatings.add(4.25);
workshopRatings.add(4.75);

cout << "First workshop rating: "
     << workshopRatings.get(0) << endl;
cout << "Second workshop rating: "
     << workshopRatings.get(1) << endl;
```

Compila y ejecuta de nuevo.

**Qué deberías ver adicionalmente:**

```text
First workshop rating: 4.25
Second workshop rating: 4.75
```

`WorkshopLog<int>` y `WorkshopLog<double>` son clases concretas distintas generadas desde la misma class template. La estructura y las operaciones son iguales; el tipo guardado cambia. Ese es el beneficio concreto de la template: la oficina obtiene dos registros adecuados para sus métricas sin que el programador tenga que mantener dos clases con lógica duplicada.

💾 **Buen momento para el commit** — terminaste un ejemplo guiado que demuestra dos instancias concretas de una class template. Commit: `"Complete WorkshopLog class template example"`.

## Parte C — TODO: implementar `MetricLog<T>` (25 min)

### Contexto

La oficina de actividades ahora quiere guardar otras métricas de talleres. Por ejemplo, puede registrar cuántos estudiantes quedaron en lista de espera usando `int` o cuántos minutos duró una sesión usando `double`. En ambos casos necesita una colección de capacidad fija con las mismas operaciones básicas.

Crea un segundo archivo llamado `metric_log.cpp`. Copia el template completo siguiente. No está resuelto: los TODOs de las Partes C, D y E están agrupados dentro del mismo archivo. Resuelve **solo** los TODOs marcados `(Part C)` ahora e ignora los demás.

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 6 - Class Templates
 * Description: Generic metric log practice and ADT contract
 * Due date: [Date]
 */

#include <iostream>
using namespace std;

/*
 * MetricLog ADT
 *
 * Data:
 * [Complete in Part E: describe the values managed by this log.]
 *
 * Operations:
 * [Complete in Part E: describe add(value).]
 * [Complete in Part E: describe get(index) and its precondition.]
 * [Complete in Part E: describe contains(target).]
 * [Complete in Part E: describe size() and isEmpty().]
 */

template <typename T>
class MetricLog
{
private:
    // ===== Resolve these TODOs now (Part C) =====

    // TODO (Part C): Add a constant for a fixed capacity of four values.
    // TODO (Part C): Add an array that stores values of the generic type.
    // TODO (Part C): Add an int that tracks how many values are stored.

public:
    // TODO (Part C): Write a constructor that creates an empty log.

    // TODO (Part C): Write add. It receives one generic value by const reference.
    // Return false when the log has no remaining capacity.

    // TODO (Part C): Write get. It receives a valid index and returns its stored value.

    // TODO (Part C): Write size as a const member function.

    // TODO (Part C): Write isEmpty as a const member function.

    bool contains(const T& target) const;
};

// ===== Do not resolve these TODOs yet (Part D) =====

// TODO (Part D): Define MetricLog<T>::contains outside the class.
// It receives a generic target by const reference.
// Return true when an equal stored value exists; otherwise return false.

int main()
{
    MetricLog<int> waitlistCounts;

    waitlistCounts.add(6);
    waitlistCounts.add(11);

    cout << "First waitlist count: "
         << waitlistCounts.get(0) << endl;
    cout << "Stored waitlist counts: "
         << waitlistCounts.size() << endl;

    // ===== Do not resolve these TODOs yet (Part E) =====

    // TODO (Part E): Create a MetricLog<double> for session durations.
    // TODO (Part E): Add two dummy duration values to that log.
    // TODO (Part E): Use contains with one value that exists and one that does not exist.
    // TODO (Part E): Print descriptive English labels for all results.

    return 0;
}
```

### Antes de escribir

1. ¿Cuál member variable debe usar `T` y cuáles deben seguir siendo `int`?
2. ¿Por qué `get` tiene la precondición de recibir un índice válido?
3. ¿Qué debe devolver `add` cuando no queda capacidad?
4. ¿Por qué `size` e `isEmpty` no deben modificar el objeto?
5. ¿Por qué una class template puede usar un array de `T` aunque no sepa todavía si `T` será `int` o `double`?

Después de terminar la Parte C, compila y ejecuta. No agregues todavía `contains` ni las pruebas de duración.

**Qué deberías ver:**

```text
First waitlist count: 6
Stored waitlist counts: 2
```

<details>
<summary>Ver respuestas a las preguntas guía</summary>

El arreglo de valores usa `T` porque ese es el tipo que cambia entre objetos. La capacidad y el contador usan `int` porque representan cantidades y posiciones. `get` necesita un índice válido para no acceder fuera del arreglo. `add` debe devolver `false` si no queda espacio. `size` e `isEmpty` solo consultan el estado; por eso son `const`. Cuando se crea un objeto como `MetricLog<int>`, el compilador usa `int` donde aparece `T`; para `MetricLog<double>`, usa `double`.
</details>

💾 **Buen momento para el commit** — implementaste y probaste una class template con una interfaz básica. Commit: `"Implement MetricLog class template"`.

## Parte D — TODO: agregar `contains` (20 min)

### Contexto

El coordinador no quiere registrar dos veces la misma cantidad de estudiantes en lista de espera. Antes de agregar un nuevo valor, puede consultar si ese valor ya existe en `MetricLog<int>`. Esa necesidad añade una nueva operación a la misma class template: `contains`.

Resuelve **solo** el TODO marcado `(Part D)`. No resuelvas todavía los TODOs de la Parte E.

### Antes de escribir

1. ¿Qué operación necesita soportar el tipo `T` para que `contains` pueda comparar un valor almacenado con el target?
2. ¿Por qué `contains` debe ser una member function `const`?
3. Si mañana se crea `MetricLog<Book>`, ¿qué tendría que definir la clase `Book` para que `contains` compile?
4. ¿Por qué `contains` pertenece a `MetricLog<T>` y no a una función separada en este escenario?

Debes definir `contains` fuera de la clase, usando el template prefix y la sintaxis `MetricLog<T>::contains`. Así practicas la misma forma de definición externa presentada en la Parte 3 de la lecture. Después de implementarla, agrega temporalmente una llamada de prueba en `main` solo si necesitas verificarla. El trabajo permanente de pruebas con `double` se realizará en la Parte E.

<details>
<summary>Ver respuestas a las preguntas guía</summary>

`T` debe soportar `==`. `contains` solo consulta el log, por eso debe ser `const`. Una clase `Book` necesitaría una comparación válida con `operator==`. La operación pertenece a `MetricLog<T>` porque consulta los valores y el contador privados que pertenecen a cada objeto del log.
</details>

💾 **Buen momento para el commit** — agregaste una operación genérica que depende de una capacidad específica del tipo `T`. Commit: `"Add MetricLog contains operation"`.

## Parte E — Aplicar la template y documentar su ADT (20 min)

### Contexto

Ahora usarás la misma class template para una segunda métrica: duración de sesiones. Las duraciones usan `double`, pero el log ofrece las mismas operaciones que para la lista de espera.

Resuelve **solo** los TODOs marcados `(Part E)` en `main`. Usa datos dummy y demuestra:

- un objeto `MetricLog<int>` para lista de espera;
- un objeto `MetricLog<double>` para duración de sesiones;
- una búsqueda que devuelve `true`;
- una búsqueda que devuelve `false`.

Una salida posible es:

```text
First waitlist count: 6
Stored waitlist counts: 2
First session duration: 42.5
Duration exists: true
Other duration exists: false
```

### El contrato del ADT

Antes de la declaración de `MetricLog<T>` en `metric_log.cpp`, el template ya incluye una especificación en comentarios para el **MetricLog ADT**. Completa los textos entre corchetes con tus propias palabras. No es una tercera clase ni una palabra reservada: es la documentación del contrato que tu class template implementa.

Las operaciones públicas de `MetricLog<T>` son la implementación en C++ de ese contrato. El código dentro de `main` es el código cliente: usa las operaciones públicas, pero no accede directamente al array ni al contador privados.

<details>
<summary>Pregunta de verificación: ¿por qué <code>MetricLog&lt;int&gt;</code> y <code>MetricLog&lt;double&gt;</code> pueden implementar el mismo ADT?</summary>

Porque el ADT define las operaciones y reglas de una colección de métricas, no un único tipo de valor. La class template reutiliza esa implementación con `int`, `double` u otro tipo que soporte las operaciones necesarias.
</details>

💾 **Buen momento para el commit** — demostraste dos versiones concretas de la template y documentaste el ADT que ambas implementan. Commit: `"Test MetricLog types and document ADT"`.

## Parte F — Verificación, commit final y entrega (15 min)

Antes de entregar, verifica:

- `workshop_log_example.cpp` compila y contiene los tres pasos activos.
- `metric_log.cpp` compila y todos los TODOs de las Partes C, D y E están resueltos o eliminados.
- Los nombres, comentarios y mensajes del código están en inglés.
- Los datos de prueba son dummy; no incluyas información personal real.
- `.gitignore` evita que los ejecutables aparezcan en Source Control.
- Los comentarios al inicio de `metric_log.cpp` incluyen la especificación completa del MetricLog ADT.

Haz un commit final si necesitaste hacer correcciones después de la Parte E. Usa un mensaje que describa claramente la corrección realizada.

## Entregable del laboratorio

Envía al profesor el enlace de tu repositorio `lab6-class-templates` en GitHub, con:

- Al menos **5 commits** visibles: setup, ejemplo guiado, template básica, contains y pruebas/contrato ADT.
- Un `.gitignore` funcionando, sin ejecutables ni archivos objeto en el repositorio.
- `workshop_log_example.cpp` con los tres pasos del ejemplo guiado activos.
- `metric_log.cpp` con todos los TODOs resueltos y la especificación del MetricLog ADT.
- El encabezado estándar completo en ambos archivos, con tu nombre y fecha reales.
- Una ejecución verificable de `metric_log.cpp` que use `MetricLog<int>` y `MetricLog<double>`, y demuestre `contains` con resultados `true` y `false`.

## Próxima sesión

En el Lab 7 profundizarás en Abstract Data Types (ADT): diseñarás contratos más completos y analizarás cómo una misma interfaz puede tener implementaciones distintas antes de estudiar linked lists.
