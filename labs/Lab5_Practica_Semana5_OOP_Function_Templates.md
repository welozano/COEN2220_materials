# Lab 5 — Práctica de Semana 5: OOP y Function Templates
## COEN 2220 — Programming 2

**Instructor:** Wilson Lozano

**Modalidad:** Práctica individual evaluada, para completar en casa
**Duración estimada:** 110 min de trabajo independiente
**Sesión presencial de la semana:** Project Kickoff — formación de equipos y discusión de propuesta
**Precede a:** Semana 6 — continuación de Templates e introducción a Abstract Data Types (ADT)
**Requisitos:** Lecture de Semana 5; Labs 1–4 completados (Git/GitHub, clases, inheritance, polymorphism y function templates)
**Basado en:** Gaddis, *Starting Out with C++*, 8va ed., Cap. 15 — secciones 15.1–15.7; Cap. 16 — secciones 16.2–16.3.

---

## Objetivos

Al finalizar esta práctica, el estudiante podrá:

1. Implementar una derived class mediante `public` inheritance a partir de una base class dada.
2. Sobrescribir una función `virtual` usando `override` y explicar el comportamiento resultante.
3. Usar una base pointer y el operador `->` para observar polymorphism dinámico.
4. Implementar una function template que opere con tipos compatibles.
5. Entregar un programa C++ compilable y probado en un único archivo fuente.

## Parte A — Preparar la carpeta y el repositorio (10 min)

> ⚠️ **Este paso es el que más se presta a error.** Cada práctica tiene su propio repositorio. No inicialices Git en la carpeta padre `COEN2220` ni dentro de la carpeta de otro lab.

1. Abre VS Code o VSCodium y selecciona `File → Open Folder...`.
2. Navega a tu carpeta padre del curso, por ejemplo `COEN2220`. Si todavía no existe, créala **desde este diálogo** en un lugar de fácil acceso.
3. Entra dentro de la carpeta padre. Sin salir del diálogo, crea una carpeta nueva llamada `lab5-week5-practice`, hermana de `lab1-git`, `lab2-pointers`, `lab3-oop` y `lab4-more-classes`.
4. Abre específicamente `lab5-week5-practice` con **Open** / **Select Folder**.

**Cómo saber si abriste la carpeta correcta:** el Explorer debe mostrar `LAB5-WEEK5-PRACTICE` como raíz. Si muestra `COEN2220` o el nombre de otro lab, usa `File → Close Folder` y repite los pasos.

> **¿Por qué importa tanto esto?** Inicializar el repositorio en la carpeta padre mezcla todos los labs en un solo historial. Inicializarlo dentro de un lab anterior mezcla temas y entregas. Un repositorio separado permite evaluar esta práctica como una unidad de trabajo independiente.

Abre Source Control (`Ctrl+Shift+G` / `Cmd+Shift+G`) y selecciona **Initialize Repository**. Crea `.gitignore` en la raíz con este contenido:

```gitignore
# Compiled executables (Windows)
*.exe

# Compiled executable (Mac/Linux)
week5_practice

# Intermediate object files
*.o

# Editor configuration
.vscode/
```

Haz el commit inicial y publica el repositorio con el flujo ya practicado: stage → commit → **Publish Branch**. Usa el mensaje `"Initial setup and gitignore"`.

💾 **Por qué este commit importa:** guarda una base limpia antes de escribir código. Si aparece un problema después, el historial muestra que la carpeta, el repositorio y `.gitignore` ya estaban correctos.

## Parte B — Contexto y starter template (10 min)

### Contexto

El campus publica información sobre actividades estudiantiles en un **Campus Event Board**. Todo evento tiene un nombre y puede imprimir sus detalles. Algunos eventos son abiertos; otros tienen capacidad limitada y deben informar cuántos asientos quedan disponibles.

El Event Board también ayuda a planificar futuras actividades. El organizador puede comparar los **estimated attendance** de dos eventos para decidir cuál necesita el salón más grande. Después de realizarlos, puede comparar los **event ratings** para decidir qué formato conviene repetir. Esas dos métricas son números de tipos distintos: los estimados son `int` y los ratings son `double`.

La clase base `CampusEvent` representa lo común. Tu trabajo será completar la derived class `LimitedEvent`, usar objetos mediante `CampusEvent *` y añadir una función genérica para comparar estimados de asistencia o ratings. El escenario es pequeño a propósito: permite practicar las decisiones de diseño de la lecture en un dominio nuevo, sin añadir archivos o estructuras de datos adicionales.

Crea **un único archivo** llamado `week5_practice.cpp`. Copia el template completo siguiente. El archivo no está resuelto: contiene TODOs para las Partes C, D y E. No intentes completar todos los TODOs de una vez.

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 5 - Week 5 Practice: OOP and Function Templates
 * Description: Inheritance, polymorphism, and function template practice
 * Due date: [Date]
 */

#include <iostream>
#include <string>
using namespace std;

class CampusEvent
{
    private:
        string name; // Each CampusEvent object keeps its own name.

    public:
        CampusEvent(string eventName) {
            name = eventName; // The base class initializes shared state.
        }

        string getName() const {
            return name; // Derived classes use this public interface to read the name.
        }

        virtual void printDetails() const {
            cout << name << ": open attendance" << endl;
        }
};

class LimitedEvent : public CampusEvent
{
    private:
        // ===== Resolve these TODOs now (Part C) =====

        // TODO (Part C): Add a private int member that stores available seats.

    public:
        // TODO (Part C): Write a constructor that receives an event name and an int.
        // Initialize the inherited CampusEvent part with the name, then store available seats.

        // TODO (Part C): Override printDetails() const.
        // Use getName() and print the number of available seats.
};

// ===== Resolve these TODOs later (Part D) =====

// TODO (Part D): Write a function template named largerValue.
// It receives two const references of the same generic type and returns the larger value.
// Do not write separate int and double versions.

int main() {
    CampusEvent openEvent("Coding Club Meeting");

    cout << "--- Base event ---" << endl;
    openEvent.printDetails(); // This call uses CampusEvent::printDetails directly.

    // ===== Resolve these TODOs later (Part E) =====

    // TODO (Part E): Create a LimitedEvent with dummy data.
    // TODO (Part E): Store the addresses of openEvent and your LimitedEvent
    // in two CampusEvent pointers. Call printDetails() through both pointers.
    // TODO (Part E): Use largerValue with two int values and with two double values.
    // Print each result with a descriptive English label.

    return 0;
}
```

Por ahora, el template solo demuestra la descripción de la base class. No compilará hasta que completes los TODOs de la Parte C. Cuando termines cada parte, compila con:

```bash
g++ week5_practice.cpp -o week5_practice
./week5_practice
```

**Qué deberías ver por ahora, después de completar la Parte C:**

```text
--- Base event ---
Coding Club Meeting: open attendance
```

No entregues otro `.cpp`. Todo el trabajo de esta práctica debe permanecer en `week5_practice.cpp`.

## Parte C — TODO: derived class y `override` (30 min)

### Antes de escribir

La relación de diseño es `LimitedEvent is a CampusEvent`: todo evento limitado tiene un nombre y puede imprimir detalles, pero añade el estado particular de asientos disponibles. Por eso `LimitedEvent` hereda públicamente de `CampusEvent`.

Resuelve **solo** los TODO marcados `(Part C)` dentro de `LimitedEvent`. Ignora los TODOs de las Partes D y E por ahora.

Preguntas guía:

1. ¿Qué member variable necesita `LimitedEvent` que no pertenece a todos los `CampusEvent`?
2. ¿Por qué el constructor de `LimitedEvent` necesita inicializar primero la parte `CampusEvent`?
3. ¿Qué firma debe tener `printDetails()` para realmente sobrescribir la función `virtual` de la base?
4. ¿Por qué `getName()` es preferible a intentar acceder directamente al `name` private de `CampusEvent`?

Después de implementar la clase, compila y ejecuta. En este momento solo verás la actividad base; eso confirma que tu derived class ya está declarada correctamente aunque todavía no la uses en `main`.

<details>
<summary>Ver respuestas a las preguntas guía</summary>

`LimitedEvent` necesita un `int` para guardar los asientos disponibles. Su constructor debe inicializar la parte base porque `name` pertenece a `CampusEvent` y es private. La función debe tener la firma `void printDetails() const` y usar `override`. La derived class puede pedir el nombre con `getName()`, pero no puede tocar el member private `name` directamente.

</details>

💾 **Buen momento para el commit** — completaste una clase derived con una responsabilidad clara y una versión propia de una operación virtual. Commit: `"LimitedEvent: inheritance and override"`.

## Parte D — TODO: function template `largerValue` (25 min)

### Contexto

Imagina que el organizador debe tomar dos decisiones concretas con los datos del Event Board:

| Decisión | Datos disponibles | Resultado que necesita y por qué |
| --- | --- | --- |
| Reservar un salón que pueda servir para cualquiera de los dos eventos | Coding Club Meeting: `45` asistentes estimados; Robotics Workshop: `80` asistentes estimados | `80`. El salón debe tener al menos 80 asientos para recibir el evento más grande. Si se eligiera 45, hasta 35 personas del Robotics Workshop no tendrían asiento. |
| Elegir qué formato repetir el próximo semestre | Coding Club Meeting: rating promedio `4.2` de 5; Robotics Workshop: rating promedio `4.7` de 5 | `4.7`. Una calificación mayor representa una evaluación más favorable de los asistentes; por eso el Robotics Workshop sería el formato que conviene repetir. |

El primer par usa `int`; el segundo usa `double`. Sin una template, habría que escribir una función para comparar dos `int` y otra casi idéntica para comparar dos `double`. Ambas recibirían dos valores, decidirían cuál es mayor y devolverían ese valor. La única diferencia sería el tipo escrito en la función.

`largerValue` representa esa operación común. No compara objetos `CampusEvent` completos; compara **una métrica a la vez**. Recibe dos values del mismo type parameter `T` y devuelve uno de esos values: el mayor. Cuando se llame con dos `int`, el compilador usará `T` como `int`; cuando se llame con dos `double`, usará `T` como `double`. Por eso el tipo debe soportar una comparación de mayor que.

Resuelve **solo** el TODO marcado `(Part D)`. Escribe una sola function template llamada `largerValue` con el contrato descrito arriba. No escribas dos funciones separadas y no uses STL.

Preguntas guía:

1. ¿Qué operador debe soportar el tipo genérico para que la función pueda decidir cuál valor es mayor?
2. ¿Por qué los parámetros se reciben como `const` references?
3. ¿Por qué una template con un solo type parameter no debe intentar comparar un `int` con un `string`?

En esta parte todavía no agregues llamadas a `largerValue` en `main`; esas pruebas pertenecen a la Parte E. Después de completar la template, compila para confirmar que la declaración es válida.

<details>
<summary>Ver respuestas a las preguntas guía</summary>

El tipo debe soportar una comparación como `>`. Las `const` references permiten leer los valores sin copiarlos ni modificarlos. Un solo type parameter representa un mismo tipo; además, comparar un `int` con un `string` no tiene un significado natural para esta función.

</details>

💾 **Buen momento para el commit** — agregaste una operación genérica sin duplicar lógica para cada tipo numérico. Commit: `"Add largerValue function template"`.

## Parte E — TODO: polymorphism y pruebas (20 min)

Ahora resuelve **solo** los TODOs marcados `(Part E)` dentro de `main()`. Usa datos dummy. Crea una `LimitedEvent`, guarda las direcciones del evento abierto y del evento limitado en variables de tipo `CampusEvent *`, y llama `printDetails()` mediante esas pointers.

Después prueba `largerValue` una vez con dos estimated attendance de tipo `int` y otra con dos event ratings de tipo `double`. Imprime los resultados con labels claros en inglés.

Antes de escribir, responde:

1. ¿Por qué una `CampusEvent *` puede guardar la dirección de un objeto `LimitedEvent`?
2. ¿Por qué se usa `->` y no `.` cuando llamas `printDetails()` mediante una pointer?
3. ¿Qué versión de `printDetails()` debe ejecutarse para la pointer que señala a tu `LimitedEvent`? ¿Por qué?
4. ¿Qué dos tipos usarás para probar `largerValue`?

Tu programa debe imprimir evidencia de los tres resultados: la descripción del evento abierto, la descripción especializada de `LimitedEvent` y las dos comparaciones de la template. Una salida posible, con tus propios datos dummy, es:

```text
--- Base event ---
Coding Club Meeting: open attendance
--- Polymorphism ---
Coding Club Meeting: open attendance
Robotics Workshop: 18 seats available
--- Function template ---
Higher attendance estimate: 91
Higher event rating: 4.75
```

<details>
<summary>Ver respuestas a las preguntas guía</summary>

Un objeto `LimitedEvent` también es un `CampusEvent`, por eso su dirección puede guardarse en una `CampusEvent *`. `->` accede a un member mediante una pointer; `.` se usa con un objeto directo. Como `printDetails()` es virtual, la pointer que señala al evento limitado ejecuta la versión overridden de `LimitedEvent`. La práctica debe usar `int` y `double` para comprobar que la template genera versiones para ambos tipos.

</details>

💾 **Buen momento para el commit** — ya demostraste inheritance, polymorphism y una function template en un programa que compila y produce resultados observables. Commit: `"Complete Week 5 OOP and template practice"`.

## Parte F — Revisión, evaluación y entrega (15 min)

Antes de entregar, verifica:

- `week5_practice.cpp` compila con `g++ week5_practice.cpp -o week5_practice`.
- Solo existe **un** archivo `.cpp` para esta práctica: `week5_practice.cpp`.
- Todos los TODOs de las Partes C, D y E están resueltos o eliminados.
- Los nombres, comentarios y mensajes del código están en inglés.
- Los datos de prueba son dummy; no incluyas información personal real.
- `.gitignore` evita que el ejecutable aparezca en Source Control.

### Criterios de evaluación (100 puntos)

| Criterio | Puntos |
| --- | ---: |
| `LimitedEvent` correctamente declarada con `public` inheritance, estado propio y constructor funcional | 30 |
| `printDetails() const override` correcto y salida de capacidad verificable | 20 |
| Uso correcto de base pointers y comportamiento polimórfico observable | 20 |
| `largerValue` como function template, probada con `int` y `double` | 20 |
| Archivo único, compilación, encabezado, datos dummy y commits claros | 10 |

## Entregable del laboratorio

Envía al profesor el enlace de tu repositorio `lab5-week5-practice` en GitHub, con:

- Al menos **4 commits** visibles: setup, derived class, function template y programa completo.
- Un `.gitignore` funcionando, sin ejecutables ni archivos objeto en el repositorio.
- Solo un archivo fuente de la práctica: `week5_practice.cpp`, con todos los TODOs resueltos.
- El encabezado estándar completo con tu nombre y fecha reales.
- Una ejecución verificable que muestre inheritance/polymorphism y las dos llamadas a `largerValue`.

## Próxima sesión

En la Semana 6 continuarán con templates y comenzarán Abstract Data Types (ADT).
