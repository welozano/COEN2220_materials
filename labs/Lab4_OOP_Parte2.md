# Lab 4 — Object-Oriented Programming, Parte 2: More About Classes
## COEN 2220 — Programming 2

**Instructor:** Wilson Lozano

**Duración:** 110 min (sesión de 2 horas crédito nominales)
**Precede a:** Lecture de Semana 5 — Inheritance, Polymorphism, and Virtual Functions
**Requisitos:** Lecture de Semana 4; Labs 1–3 completados (Git/GitHub, `.gitignore`, clases y constructores)
**Basado en:** Gaddis, *Starting Out with C++*, 8va ed., Cap. 14 — selección: static members, memberwise assignment, copy constructors, operator overloading y aggregation.

---

## Objetivos

Al finalizar este laboratorio, el estudiante podrá:

1. Implementar y probar un `static` data member y una `static` member function.
2. Distinguir, mediante una traza, entre copy construction y assignment sobre un objeto existente.
3. Modelar una relación de aggregation usando objetos de una clase como members de otra.
4. Implementar y aplicar una sobrecarga de `operator==` cuyo significado sea claro para el dominio de la clase.
5. Reconocer, de forma introductoria, cómo inheritance y `virtual` permiten polymorphism; este último tema se estudiará formalmente en la Semana 5.

## Parte A — Preparar la Carpeta y el Repositorio (5 min)

> ⚠️ **Este paso es el que más se presta a error — léelo con cuidado antes de hacer clic en nada.** Cada lab tiene su propio repositorio. No inicialices Git en la carpeta padre `COEN2220` ni dentro de la carpeta de otro lab.

1. Abre VS Code o VSCodium y selecciona `File → Open Folder...`.
2. Navega a tu carpeta padre del curso, por ejemplo `COEN2220`. Si todavía no existe, créala **desde este diálogo** en un lugar de fácil acceso.
3. Entra dentro de la carpeta padre. Sin salir del diálogo, crea una carpeta nueva llamada `lab4-more-classes`, hermana de `lab1-git`, `lab2-pointers` y `lab3-oop`.
4. Abre específicamente `lab4-more-classes` con **Open** / **Select Folder**.

**Cómo saber si abriste la carpeta correcta:** el Explorer debe mostrar `LAB4-MORE-CLASSES` como raíz. Si muestra `COEN2220` o el nombre de otro lab, usa `File → Close Folder` y repite los pasos.

> **¿Por qué importa tanto esto?** Inicializar el repositorio en la carpeta padre mezcla todos tus labs en un solo historial. Inicializarlo dentro de un lab anterior mezcla archivos de temas distintos. Un repositorio por lab permite revisar y entregar cada unidad de trabajo por separado.

Abre Source Control (`Ctrl+Shift+G` / `Cmd+Shift+G`) y selecciona **Initialize Repository**. Crea entonces `.gitignore` en la raíz con este contenido:

```gitignore
# Compiled executables (Windows)
*.exe

# Compiled executables (Mac/Linux)
study_time
course_section
inheritance_preview

# Intermediate object files
*.o

# Editor configuration
.vscode/
```

Haz el commit inicial y publica el repositorio, usando el flujo ya practicado en los Labs 1–3: stage → commit → **Publish Branch**. Usa el mensaje `"Initial setup and gitignore"`.

💾 **Por qué este commit importa:** registra una base funcional y limpia antes de escribir código. Si algo sale mal después, el historial muestra que la carpeta correcta, el repositorio y `.gitignore` ya estaban configurados.

## Parte B — Ejemplo Guiado: `StudyTime` (30 min)

### Contexto

Imagina una aplicación sencilla para registrar tiempo de estudio. Cada objeto `StudyTime` guarda una duración en minutos. Además, la clase quiere saber cuántos objetos `StudyTime` se han creado durante la ejecución. Ese total pertenece a la **clase**, no a un objeto individual, por lo que es un buen uso de `static`.

El ejemplo también mostrará una copia de objeto y una sobrecarga de `operator+`. No pegues el código completo y lo ejecutes una sola vez: constrúyelo en tres pasos y observa qué cambia en cada uno.

### Paso 1 — Datos por objeto y contador compartido

Crea `study_time.cpp` con este contenido. Los bloques `STEP 2` y `STEP 3` están comentados y no deben activarse todavía.

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 4 - Object-Oriented Programming, Part 2
 * Description: Guided example - static members, copies, and operators
 * Due date: [Date]
 */

#include <iostream>
using namespace std;

class StudyTime
{
    private:
        int minutes;               // Each object stores its own duration.
        static int objectCount;    // One counter is shared by the entire class.

    public:
        StudyTime(int m = 0) {
            minutes = m;
            objectCount++;         // Every new object increases the shared count.
        }

        int getMinutes() const { return minutes; }

        static int getObjectCount() {
            return objectCount;    // A static function reads class-level data.
        }

        // --- STEP 2: copy constructor (uncomment the /* ... */ block later) ---
        /*
        StudyTime(const StudyTime &other) {
            minutes = other.minutes;  // Copy the source object's duration.
            objectCount++;            // The copy is a new object too.
        }
        */

        // --- STEP 3: operator+ (uncomment the /* ... */ block later) ---
        /*
        StudyTime operator+(const StudyTime &other) const {
            // Return a new duration without changing either operand.
            return StudyTime(minutes + other.minutes);
        }
        */
};

int StudyTime::objectCount = 0;    // Define the single shared variable.

int main() {
    StudyTime reading(45);          // The first StudyTime object.

    cout << "Reading: " << reading.getMinutes() << " minutes" << endl;
    cout << "Objects created: " << StudyTime::getObjectCount() << endl;

    // --- STEP 2 main() code goes here later ---

    // --- STEP 3 main() code goes here later ---

    return 0;
}
```

Compila y ejecuta:

```bash
g++ study_time.cpp -o study_time
./study_time
```

**Qué deberías ver:** `Reading: 45 minutes` y `Objects created: 1`. El contador no está dentro de `reading`; se consulta con `StudyTime::getObjectCount()` porque pertenece a la clase.

**Antes de seguir:** si creas un segundo objeto `StudyTime`, ¿cuál será el valor de `objectCount`? ¿Cambiarán los minutos de `reading`?

<details>
<summary>Ver respuesta</summary>

El contador pasará a `2`, porque existe una sola variable `objectCount` compartida. Los minutos de `reading` seguirán siendo `45`: `minutes` no es `static`, así que cada objeto tiene su propia copia.

</details>

### Paso 2 — Activar el copy constructor

Descomenta solo el bloque `STEP 2`: elimina los delimitadores `/*` y `*/`, pero deja la línea `// --- STEP 2: copy constructor ---` intacta. Después reemplaza `// --- STEP 2 main() code goes here later ---` por:

```cpp
    StudyTime readingCopy = reading;  // Create a new object from reading.
    cout << "Copy: " << readingCopy.getMinutes() << " minutes" << endl;
    cout << "Objects created: " << StudyTime::getObjectCount() << endl;
```

Compila y ejecuta otra vez.

**Qué deberías ver:** `readingCopy` también guarda `45` minutos y el contador ahora vale `2`.

**Antes de seguir:** ¿por qué `StudyTime readingCopy = reading;` incrementa el contador aunque los dos objetos tengan el mismo número de minutos?

<details>
<summary>Ver respuesta</summary>

`readingCopy` es un objeto nuevo. El copy constructor copia los datos de `reading`, pero también crea una segunda instancia de la clase; por eso incrementa `objectCount`.

</details>

### Paso 3 — Activar `operator+`

Descomenta el bloque `STEP 3`. Luego reemplaza `// --- STEP 3 main() code goes here later ---` por:

```cpp
    StudyTime practice(30);
    StudyTime total = reading + practice;  // Calls reading.operator+(practice).

    cout << "Total: " << total.getMinutes() << " minutes" << endl;
    cout << "Objects created: " << StudyTime::getObjectCount() << endl;
```

Compila y ejecuta por última vez.

**Qué deberías ver:** `Total: 75 minutes`. El contador termina en `4`: `reading`, `readingCopy`, `practice` y `total`. La función `operator+` devuelve un objeto nuevo; no modifica `reading` ni `practice`.

💾 **Buen momento para el commit** — el ejemplo ya es una unidad funcional: muestra un member `static`, una copia y un operador con un significado natural. Commit: `"Guided example: StudyTime class"`.

## Parte C — TODO: Aggregation con `CourseSection` (20 min)

### Contexto

En la lecture, `Course` tenía un `Instructor` y un `TextBook` como members. Ahora vas a construir una versión más pequeña: `CourseSection` representa una sección concreta y **tiene** objetos `Instructor` y `TextBook`. Cada clase conserva la responsabilidad de sus propios datos; `CourseSection` los combina para presentar la información de la sección.

Crea `course_section.cpp`:

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 4 - Object-Oriented Programming, Part 2
 * Description: Aggregation practice with CourseSection
 * Due date: [Date]
 */

#include <iostream>
#include <string>
using namespace std;

class Instructor
{
    private:
        string name;  // State owned by one Instructor object.

    public:
        Instructor() { name = ""; }  // Start with a predictable empty value.
        void setName(string n) { name = n; }  // The public interface controls changes.
        string getName() const { return name; }  // Reading should not modify the object.
};

class TextBook
{
    private:
        string title;  // State owned by one TextBook object.

    public:
        TextBook() { title = ""; }  // Start with a predictable empty value.
        void setTitle(string t) { title = t; }  // Keep title changes inside the class interface.
        string getTitle() const { return title; }  // Reading should not modify the object.
};

class CourseSection
{
    private:
        // ===== Resuelve estos TODO ahora (Parte C) =====

        // TODO (Parte C): Add a private string for the section identifier.
        // TODO (Parte C): Add private Instructor and TextBook member objects.
        // These members create the "has-a" relationship for this exercise.

    public:
        // TODO (Parte C): Write a constructor that receives a section identifier,
        // an instructor name, and a textbook title. Use the public interfaces of
        // Instructor and TextBook to initialize their data.

        // TODO (Parte C): Write printInfo() const. Display the section identifier,
        // instructor name, and textbook title by calling public getters.
};

int main() {
    // TODO (Parte C): Create one CourseSection with dummy data and call printInfo().
    // Do not use real student information.

    return 0;
}
```

Antes de escribir, responde estas preguntas guía:

1. ¿Qué members pertenecen a `CourseSection` y cuáles pertenecen a `Instructor` o `TextBook`?
2. ¿Por qué `CourseSection` debe llamar `instructor.setName(...)` en vez de acceder a `instructor.name`?
3. ¿Cuál objeto representa el todo en la relación “has-a”?

Resuelve **solo** los TODO marcados `(Parte C)` ahora. Compila y ejecuta. Tu salida debe mostrar los tres datos con información dummy.

<details>
<summary>Ver respuesta a las preguntas guía</summary>

`CourseSection` guarda el identificador de sección y contiene los objetos `Instructor` y `TextBook`. Cada objeto conserva sus propios datos privados. `CourseSection` debe usar setters/getters porque `name` y `title` son privados. El objeto que representa el todo es `CourseSection`.

</details>

💾 **Buen momento para el commit** — terminaste una clase con una relación “has-a” y una interfaz pública comprobable. Commit: `"CourseSection: aggregation with Instructor and TextBook"`.

## Parte D — Copia versus Assignment: traza con `StudyTime` (15 min)

Vuelve a `study_time.cpp`, con los tres pasos del ejemplo guiado activos. Agrega al final de `main()`, antes de `return 0`, tus propias instrucciones para realizar esta secuencia:

1. Crea un objeto nuevo inicializado desde `reading`.
2. Crea otro objeto nuevo con la duración por default.
3. Asigna `reading` sobre el segundo objeto ya existente.
4. Imprime el valor final de `StudyTime::getObjectCount()` y los minutos de ambos objetos.

No copies una respuesta sin pensar: primero predice el contador después de cada paso. Luego compila y compara la predicción con la salida.

**Preguntas guía:**

1. ¿En cuáles de los pasos se llama el copy constructor?
2. ¿En cuál se usa assignment?
3. ¿Por qué assignment no incrementa `objectCount`?

<details>
<summary>Ver respuesta</summary>

El copy constructor se llama al crear el primer objeto desde `reading`. Crear el segundo objeto por default también llama un constructor, así que aumenta el contador. La instrucción de assignment modifica el segundo objeto que ya existía; no construye una instancia nueva y por eso no aumenta `objectCount`.

</details>

💾 **Buen momento para el commit** — registraste una prueba que demuestra una distinción central de la lecture. Commit: `"Trace copy construction and assignment"`.

## Parte E — TODO: Comparar duraciones (15 min)

En `study_time.cpp`, agrega este bloque **dentro de la clase `StudyTime`**, después del bloque `STEP 3`:

```cpp
        // ===== Resuelve estos TODO ahora (Parte E) =====

        // TODO (Parte E): Define operator== so two StudyTime objects are equal
        // when they represent the same number of minutes. The function must not
        // modify either object and must return a bool.
```

Antes de implementar, responde:

1. ¿Qué dato representa el significado principal de un `StudyTime`?
2. ¿Qué tipo de valor debe devolver una comparación de igualdad?
3. ¿Por qué `operator==` no debe cambiar `minutes`?

Resuelve **solo** el TODO de la Parte E. Después, en `main()`, crea dos objetos con la misma cantidad de minutos y un tercero con una cantidad distinta. Imprime el resultado de al menos dos comparaciones, usando `boolalpha` para que la salida diga `true` o `false`.

<details>
<summary>Ver respuesta a las preguntas guía</summary>

El dato significativo es `minutes`. Una comparación de igualdad debe devolver `bool`. Comparar debe observar el estado de los objetos, no alterarlo; de otro modo, una expresión como `first == second` produciría un efecto inesperado.

</details>

💾 **Buen momento para el commit** — añadiste una operación clara a una clase existente y la validaste con casos iguales y distintos. Commit: `"StudyTime: add equality operator"`.

## Parte F — Vista previa: Inheritance y Polymorphism (10 min)

### Contexto

Este bloque es una **vista previa**, no contenido evaluable de este lab. La próxima lecture estudiará formalmente inheritance, `virtual` y polymorphism. Por ahora, observa que `Student` puede ser un tipo más específico de `Person`, y que un `Person*` puede referirse a objetos de ambos tipos.

Crea `inheritance_preview.cpp` con este código completamente resuelto. Léelo, compílalo y traza qué función se ejecuta en cada vuelta del `for`.

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 4 - Object-Oriented Programming, Part 2
 * Description: Preview of inheritance and runtime polymorphism
 * Due date: [Date]
 */

#include <iostream>
#include <string>
using namespace std;

class Person
{
    private:
        string name;  // Every Person object has its own name.

    public:
        Person(string n) { name = n; }  // Initialize the shared base-class state.

        string getName() const { return name; }  // Derived classes can read this through the public interface.

        virtual void introduce() const {
            // virtual allows a derived class to provide its own behavior.
            cout << "Person: " << name << endl;
        }
};

class Student : public Person
{
    private:
        string major;  // Data that exists only for Student objects.

    public:
        Student(string n, string m) : Person(n) {
            // Person(n) initializes the name before Student's constructor body runs.
            major = m;  // Student adds data beyond the base Person class.
        }

        void introduce() const override {
            // override asks the compiler to verify that this replaces a virtual base method.
            // This version runs when a Person pointer refers to a Student.
            cout << "Student: " << getName() << ", Major: " << major << endl;
        }
};

int main() {
    Person visitor("Jordan Lee");
    Student student("Taylor Morgan", "Computer Engineering");

    // Both addresses fit in Person pointers because Student is a Person.
    Person *people[] = {&visitor, &student};

    for (int i = 0; i < 2; i++) {
        people[i]->introduce();  // virtual selects behavior from the actual object.
    }

    return 0;
}
```

**Qué deberías ver:** una línea que comienza con `Person:` y otra que comienza con `Student:`. Aunque el array usa `Person*`, la llamada sobre `student` ejecuta `Student::introduce()` porque el método es `virtual`.

**Para discutir:** ¿qué crees que ocurriría si `virtual` se eliminara de `Person::introduce()`?

<details>
<summary>Ver respuesta</summary>

La llamada hecha mediante `Person*` usaría la versión de `Person`, incluso cuando el pointer apunta a un `Student`. La palabra `virtual` permite que C++ seleccione la versión correspondiente al tipo real del objeto durante la ejecución.

</details>

No necesitas modificar este archivo ni hacer un commit separado: úsalo como referencia para la Semana 5.

## Parte G — Verificación, commit final y colchón (15 min)

Verifica que los tres programas compilan y que los ejecutables no aparecen en Source Control. Revisa también que los nombres, datos y mensajes del código sean ficticios y estén en inglés.

Haz un commit final si completaste la Parte E: `"Complete Lab 4 operator practice"`. Usa el tiempo restante para corregir errores de compilación, revisar tu historial de commits o pedir ayuda con una de las trazas.

## Entregable del laboratorio

Envía al profesor el enlace de tu repositorio `lab4-more-classes` en GitHub, con:

- Al menos **5 commits** visibles: setup, ejemplo guiado, aggregation, traza de copias/assignment y operator de igualdad.
- Un `.gitignore` funcionando, sin ejecutables ni archivos objeto en el repositorio.
- `study_time.cpp` con los tres pasos activos, la traza de la Parte D y el TODO de la Parte E resuelto.
- `course_section.cpp` con todos los TODO de la Parte C resueltos.
- `inheritance_preview.cpp` incluido sin modificaciones obligatorias.
- El encabezado estándar completo en cada `.cpp`, con tu nombre y fecha reales.

## Próxima sesión

**Semana 5 — Inheritance, Polymorphism, and Virtual Functions.** El archivo de vista previa mostró el resultado de `virtual`; en la próxima lecture aprenderás cómo diseñar correctamente una base class, una derived class y una interfaz polimórfica.
