# Lab 3 — Object-Oriented Programming, Parte 1
## COEN 2220 — Programming 2

**Instructor:** Wilson Lozano

**Duración:** 110 min (sesión de 2 horas crédito nominales)
**Precede a:** N/A — este lab se basa en la lecture de Semana 3 (Introduction to Classes)
**Requisitos:** Lab 1 y Lab 2 completados (Git/GitHub, `.gitignore`)
**Basado en:** Gaddis, *Starting Out with C++*, 8va ed., Cap. 13

---

## Objetivos

Al finalizar este laboratorio, el estudiante podrá:
1. Inicializar correctamente un repositorio Git para un lab nuevo, sin mezclarlo con la carpeta del curso.
2. Declarar e implementar una clase completa (private members, public interface, constructores, destructor) siguiendo un ejemplo guiado.
3. Convertir un `struct` existente a una `class` con encapsulación real.
4. Declarar esqueletos de clase a partir de un análisis de responsabilidades.

---

## Parte A — Preparar la Carpeta y el Repositorio (10 min)

> ⚠️ **Este paso es el que más se presta a error — léelo con cuidado antes de hacer clic en nada.**

Ya tienes una carpeta padre de curso (la creaste en el Lab 1), algo como `COEN2220` o `COEN2220_programming_2` — el nombre exacto depende de lo que tú le hayas puesto — y dentro de ella, carpetas separadas para cada lab (`lab1-git`, `lab2-pointers`, etc.). **Cada lab es su propio repositorio independiente** — nunca conviertas la carpeta padre del curso en un repositorio.

> 💻 **Vas a usar VS Code o VSCodium** — los pasos son idénticos en ambos.

1. Abre tu editor.
2. `File → Open Folder...`
3. Navega hasta tu carpeta `COEN2220` (o como la hayas nombrado) — **pero no la selecciones a ella misma.** Entra a esa carpeta primero (doble clic para navegar adentro en el diálogo).
4. Dentro del mismo diálogo, crea una carpeta nueva llamada `lab3-oop` (la mayoría de los diálogos de "Open Folder" tienen un botón o clic derecho para "Nueva carpeta") — **hermana** de `lab1-git` y `lab2-pointers`, al mismo nivel, no una dentro de otra.
5. Selecciona esa carpeta `lab3-oop` recién creada (entra a ella) y confirma con **Open** / **Select Folder**.

**Cómo saber si abriste la carpeta correcta:** mira el panel Explorer a la izquierda en tu editor — el nombre en la parte superior debe decir `LAB3-OOP`, no `COEN2220`. Si ves el nombre de la carpeta del curso ahí, cierra la carpeta (`File → Close Folder`) y repite los pasos 2-5 con más cuidado.

> **¿Por qué importa tanto esto?** Si inicializas el repositorio de Git en la carpeta padre por accidente, terminas con un solo repositorio gigante que incluye *todos* tus labs anteriores mezclados — imposible de separar después sin trabajo extra. Cada lab debe vivir en su propio repositorio, exactamente como hiciste en el Lab 1 y el Lab 2.

Con `lab3-oop` abierta como carpeta raíz, abre el panel de Source Control (`Ctrl+Shift+G` / `Cmd+Shift+G`) y haz clic en **"Initialize Repository"**.

**Qué deberías ver:** el panel de Source Control pasa de mostrar un botón de inicializar a mostrar "No changes" (o similar) — confirma que ahora `lab3-oop` es un repositorio Git válido.

### Recordatorio rápido: `.gitignore`

Crea un archivo `.gitignore` en la raíz de `lab3-oop` (repaso del Lab 2 — si no recuerdas por qué hace falta, revisa esa sección ahí):

```gitignore
# Ejecutables compilados (Windows)
*.exe

# Ejecutables (Mac/Linux) - agrega aqui cada nombre que generes:
book
student
struct_vs_class
restaurant_design

# Archivos objeto intermedios
*.o

# Configuracion del editor
.vscode/
```

💾 **Buen momento para el commit** — tienes un punto de partida limpio (carpeta correcta, repo inicializado, `.gitignore` funcionando) antes de escribir ninguna clase todavía. Commit: `"Setup inicial + gitignore"`, luego **Publish Branch**.

---

## Parte B — Ejemplo Guiado, Completo: la clase `Book` (20 min)

### Contexto

Recuerda el escenario de la biblioteca que trabajamos en la lecture (Parte 7 — identificar clases): una biblioteca presta libros a estudiantes, cada libro tiene título/autor/ISBN, y hay que saber si está prestado o no. `Book` es exactamente la clase que identificamos ahí — hoy la van a construir de verdad, y en el resto del semestre va a reaparecer como pieza base cuando trabajen con colecciones de objetos (arrays de objetos, y más adelante, estructuras de datos).

Este bloque está **completamente resuelto** — es el patrón modelo que van a replicar en la Parte C con `Student`. Pero **no lo peguen y corran de una vez**: lo vamos a construir en 3 pasos, compilando y observando el resultado en cada uno, para que entiendan qué hace cada pieza antes de verla toda junta.

### Paso 1 — Datos, constructores, y getters

Crea `book.cpp` con este contenido. Fíjate que las secciones marcadas `STEP 2` y `STEP 3` están **comentadas** — todavía no las actives.

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 3 - Object-Oriented Programming, Part 1
 * Description: Guided example - the Book class
 * Due date: [Date]
 */

#include <iostream>
#include <string>
using namespace std;

class Book
{
    private:
        // Everything a Book needs to "know" about itself.
        // All private: nothing outside this class can touch these
        // directly - that's the point of encapsulation.
        string title;
        string author;
        string isbn;
        bool   isCheckedOut;

    public:
        // --- STEP 1: constructors + getters ---

        // Default constructor: runs automatically when you write
        // `Book b;` with no arguments. Strings default to "" on their
        // own, but isCheckedOut (a bool) would start as unpredictable
        // garbage without this - so we set it explicitly to false.
        Book() {
            title = "";
            author = "";
            isbn = "";
            isCheckedOut = false;
        }

        // Parameterized constructor: lets us create a fully-formed
        // Book in one line, instead of creating an empty one and then
        // calling setters one at a time. This is the one we'll
        // actually use in main() below.
        Book(string t, string a, string i) {
            title = t;
            author = a;
            isbn = i;
            isCheckedOut = false;   // a brand-new book always starts available
        }

        // Getters: marked const because reading a value should never
        // change the object. The compiler enforces this - if you
        // accidentally wrote code in here that modified a member, it
        // wouldn't compile.
        string getTitle() const { return title; }
        string getAuthor() const { return author; }
        string getIsbn() const { return isbn; }
        bool   getIsCheckedOut() const { return isCheckedOut; }

        void printInfo() const {
            cout << "Title:  " << title << "\n"
                 << "Author: " << author << "\n"
                 << "ISBN:   " << isbn << "\n"
                 << "Status: " << (isCheckedOut ? "Checked out" : "Available") << "\n";
        }

        // --- STEP 2: setters (uncomment the /* ... */ block below when instructed) ---
        /*
        void setTitle(string t) {
            if (!t.empty())
                title = t;
        }

        void setAuthor(string a) {
            if (!a.empty())
                author = a;
        }
        */

        // --- STEP 3: behavior (uncomment the /* ... */ block below when instructed) ---
        /*
        void checkOut() {
            if (!isCheckedOut) {
                isCheckedOut = true;
                cout << "\"" << title << "\" is now checked out.\n";
            } else {
                cout << "\"" << title << "\" is already checked out.\n";
            }
        }

        void returnBook() {
            if (isCheckedOut) {
                isCheckedOut = false;
                cout << "\"" << title << "\" has been returned.\n";
            } else {
                cout << "\"" << title << "\" was not checked out.\n";
            }
        }
        */
};

int main() {
    // Using the parameterized constructor - real-looking but dummy data.
    Book b1("Clean Code", "Robert C. Martin", "978-0132350884");

    cout << "--- Initial state ---\n";
    b1.printInfo();

    // --- STEP 2 main() code goes here later ---

    // --- STEP 3 main() code goes here later ---

    return 0;
}
```

Compila y corre:
```bash
g++ book.cpp -o book
./book
```

**Qué deberías ver:**
```
--- Initial state ---
Title:  Clean Code
Author: Robert C. Martin
ISBN:   978-0132350884
Status: Available
```

**Antes de seguir, revisa:** ¿por qué `Status` dice "Available" si nunca lo asignamos explícitamente en `main()`?

<details>
<summary>Ver respuesta</summary>

El constructor parametrizado lo fija a `false` internamente — `isCheckedOut = false;` — nunca queda a la suerte.

</details>

### Paso 2 — Activar los setters

Descomenta el bloque `STEP 2`: quita únicamente las líneas `/*` y `*/` que envuelven el código (deja la línea `// --- STEP 2: setters ---` tal cual, esa sí es un comentario válido por sí sola y no necesita tocarse). Luego, dentro de `main()`, reemplaza la línea `// --- STEP 2 main() code goes here later ---` con:

```cpp
    cout << "\n--- After setTitle/setAuthor ---\n";
    b1.setTitle("Clean Code (2nd Edition)");
    b1.setAuthor("");   // empty string - should be REJECTED
    b1.printInfo();
```

Compila y corre otra vez.

**Qué deberías ver (agregado al final de la salida anterior):**
```
--- After setTitle/setAuthor ---
Title:  Clean Code (2nd Edition)
Author: Robert C. Martin
ISBN:   978-0132350884
Status: Available
```

**Antes de seguir, revisa:** el `Author` **no cambió**, aunque llamamos `setAuthor("")`. ¿Por qué?

<details>
<summary>Ver respuesta</summary>

La validación `if (!a.empty())` rechaza el string vacío — el setter simplemente no hace nada cuando el argumento no es válido, sin necesidad de un mensaje de error para este caso.

</details>

### Paso 3 — Activar el comportamiento (`checkOut`/`returnBook`)

Descomenta el bloque `STEP 3` (mismo procedimiento: solo quita `/*` y `*/`, deja la etiqueta `// --- STEP 3 ---` intacta). Luego, reemplaza `// --- STEP 3 main() code goes here later ---` en `main()` con:

```cpp
    cout << "\n--- Checking out ---\n";
    b1.checkOut();
    b1.checkOut();   // try again - should say it's already out

    cout << "\n--- Returning ---\n";
    b1.returnBook();
    b1.returnBook(); // try again - should say it wasn't out
```

Compila y corre una última vez.

**Qué deberías ver (agregado al final):**
```
--- Checking out ---
"Clean Code (2nd Edition)" is now checked out.
"Clean Code (2nd Edition)" is already checked out.

--- Returning ---
"Clean Code (2nd Edition)" has been returned.
"Clean Code (2nd Edition)" was not checked out.
```

**Observen el patrón completo que van a replicar en la Parte C:**
1. Member variables privadas, agrupadas arriba, con un comentario explicando qué representan como grupo.
2. Constructor default + constructor con parámetros (sobrecarga) — cada uno con un comentario explicando *cuándo* se usaría.
3. Getters marcados `const`, con un comentario explicando por qué.
4. Setters con validación — el comentario explica *qué* se está validando y *por qué*.
5. Métodos de comportamiento que cambian el estado interno y dan retroalimentación con `cout`.
6. Un método `printInfo()` para mostrar el objeto completo de forma consistente.

💾 **Buen momento para el commit** — acabas de terminar una clase completa y funcional. Este es el patrón que vas a repetir el resto del semestre: **haz commit cuando termines una unidad de trabajo coherente que compila y corre**, no a mitad de algo roto. El mensaje `"Guided example: Book class"` ya sigue una buena práctica: describe *qué* se agregó, en pocas palabras, no "cambios" o "avance" (mensajes así no dicen nada útil si alguien —incluyéndote a ti mismo en el futuro— revisa el historial del repo buscando cuándo se agregó algo específico).

---

## Parte C — Ejercicio TODO: la clase `Student` (25 min)

### Contexto

`Student` es la otra clase que identificamos en el mismo escenario de biblioteca de la lecture (Parte 7), junto a `Book`. En un sistema de biblioteca real, un `Student` es quien pide prestado un `Book` — esa conexión entre las dos clases (un estudiante "tiene" libros prestados) es exactamente el tipo de relación que van a formalizar más adelante en el curso (aggregation, Semana 4). Por ahora, `Student` vive de forma independiente, con el mismo patrón interno que `Book`: datos privados, constructores, getters/setters validados, y un destructor.

Ahora te toca a ti, siguiendo el mismo patrón de `Book`, pero **sin la lógica ya resuelta en los comentarios** — solo la estructura.

El archivo `student.cpp` de abajo tiene TODOs marcados con tres etiquetas distintas: `(Parte C)`, `(Parte D)`, y `(Parte E)` — corresponden a las tres secciones de este lab en las que vas a completar `Student` por partes. **Ahora mismo, en esta sección, resuelve únicamente los TODO marcados `(Parte C)`.** Ignora por completo los marcados `(Parte D)` y `(Parte E)` — no los borres ni los completes todavía, simplemente déjalos ahí; les toca su propia sección más adelante.

Crea `student.cpp`:

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 3 - Object-Oriented Programming, Part 1
 * Description: Student class - exercise with TODOs
 * Due date: [Date]
 */

#include <iostream>
#include <string>
using namespace std;

class Student
{
    private:
        string name;
        int    id;
        double gpa;

    public:
        // ===== Resuelve estos TODO ahora (Parte C) =====

        // TODO (Parte C): Default constructor.
        // Inicializa name a "", id a 0, gpa a 0.0

        // TODO (Parte C): setName(string n)
        // Solo asigna si n no esta vacio.

        // TODO (Parte C): setGpa(double g)
        // Solo asigna si g esta entre 0.0 y 4.0 (inclusive).
        // Si no es valido, imprime un mensaje de error y no cambia el valor.

        // TODO (Parte C): Getters con const: getName(), getId(), getGpa()

        // TODO (Parte C): printInfo() const
        // Imprime name, id, y gpa con formato similar a Book::printInfo()

        // ===== No los toques todavia - les toca mas adelante =====

        // TODO (Parte D): Constructor with parameters (name, id, gpa).

        // TODO (Parte E): Destructor.
        // Imprime un mensaje indicando que el objeto Student con ese 'name' fue destruido.
};

int main() {
    // ===== Resuelve estos TODO ahora (Parte C) =====

    // TODO (Parte C): Crea un Student usando el constructor default,
    // asigna valores con los setters, e imprime con printInfo().

    // TODO (Parte C): Prueba setGpa() con un valor invalido (ej. 5.0)
    // y confirma que el mensaje de error aparece y el gpa no cambia.

    // ===== No lo toques todavia - le toca mas adelante =====

    // TODO (Parte D): Crea un segundo Student usando el constructor con
    // parametros, con datos dummy (ej. "Alice Smith", 1001, 3.7).

    return 0;
}
```

**Esto es lo que debes entregar en esta parte:** el constructor default, `setName`, `setGpa` (con la validación de rango), los tres getters, y `printInfo()` — todos implementados y probados en `main()` con al menos un objeto `Student` usando **datos dummy** (nunca información personal real).

**Ejemplo de salida esperada** (con datos dummy):
```
Name: John Doe
ID:   1000
GPA:  0

Invalid GPA: 5 (must be between 0.0 and 4.0)
```

<details>
<summary>Para practicar por tu cuenta: ¿qué pasa si no validas setGpa()?</summary>

Sin la validación, `student.setGpa(5.0);` asignaría 5.0 sin problema — un GPA que no existe en ningún sistema de calificación real. Este es exactamente el problema que discutimos en la lecture (Parte 1) sobre por qué un `struct` con miembros públicos no puede *garantizar* que sus datos sean válidos, y una `class` con encapsulación sí puede.

</details>

💾 **Buen momento para el commit** — `Student` ya tiene una primera versión funcional (compila, corre, y ya viste el rechazo de `setGpa` en acción), aunque todavía le falten el constructor con parámetros y el destructor. Commit: `"Student: constructor default, setters, getters, printInfo"`.

---

## Parte D — Constructor con Parámetros (20 min)

Vuelve a `student.cpp`. Ahora sí le toca al TODO marcado `(Parte D)`: el constructor que recibe `name`, `id`, y `gpa` directamente. (Deja el TODO `(Parte E)` sin tocar todavía — es lo último que falta.)

**Importante:** este constructor también debe validar el `gpa` — no le den un pase libre solo porque viene del constructor. Piensen cómo reusar la lógica de validación que ya escribieron en `setGpa` (pista: pueden llamar `setGpa(g);` *dentro* del constructor, en vez de repetir el `if`).

Agrega a `main()` el segundo objeto `Student` (marcado en el TODO), usando el constructor nuevo con datos dummy.

**Esto es lo que debes entregar en esta parte:** el constructor con parámetros implementado, reusando la validación de `setGpa` (no duplicada), y probado con al menos un objeto en `main()`.

<details>
<summary>Para practicar por tu cuenta: ¿por qué reusar setGpa() es mejor que repetir el if?</summary>

Si en algún momento cambian la regla de validación (por ejemplo, si el rango de GPA cambiara), solo tendrían que actualizarla en **un** lugar (`setGpa`) en vez de buscar cada copia del mismo `if` repartida por la clase. Es el mismo principio detrás de "no te repitas" (DRY) que van a ver una y otra vez el resto del semestre.

</details>

💾 **Buen momento para el commit** — agregaste una funcionalidad completa y ya probada (el constructor con parámetros) sobre una base que ya funcionaba. Commit: `"Student: constructor con parametros"`.

---

## Parte E — Destructor (15 min)

**Recordatorio rápido antes de escribir código** (de la lecture, Parte 5): un destructor se ejecuta automáticamente cuando el objeto se destruye — sale de scope, o se hace `delete` sobre un pointer que lo apunta. Si no escriben ninguno, C++ genera uno automáticamente que no hace nada.

Para `Student` tal como está hoy (sin memoria dinámica adentro), no tener destructor no rompe nada — no hay nada que limpiar todavía. **Entonces, ¿por qué molestarse en escribirlo ahora?** Por costumbre deliberada: dentro de pocas semanas (Linked Lists, Semana 9) van a construir clases que sí reservan memoria dinámica internamente, y ahí un destructor faltante o mal escrito sí causa memory leaks reales. Practicar el patrón ahora, con una clase simple donde el riesgo es bajo, es más fácil que aprenderlo por primera vez en una clase compleja donde un error es más difícil de rastrear.

Completa el `TODO (Parte E)`: el destructor de `Student`, que imprime un mensaje indicando qué objeto se está destruyendo (usando su `name`). Con esto, los tres TODO de `student.cpp` quedan completos.

**Antes de correr el programa, traza a mano:** con los dos objetos `Student` que ya tienes en `main()` (uno del constructor default, otro con parámetros), ¿en qué orden esperas ver los mensajes del destructor al final del programa?

<details>
<summary>Ver la respuesta esperada</summary>

Los destructores se llaman en **orden inverso** a como los objetos fueron creados — el último objeto declarado es el primero en destruirse, cuando `main()` termina. Si declaraste primero el objeto con constructor default y después el de parámetros, el destructor del segundo debería imprimir su mensaje primero.

</details>

Corre el programa y confirma que el orden real coincide con tu predicción.

**Esto es lo que debes entregar en esta parte:** el destructor implementado e imprimiendo un mensaje identificable por objeto, con la traza a mano confirmada contra la salida real.

💾 **Buen momento para el commit** — `Student` queda completo (los tres TODO resueltos), un buen punto de cierre para este archivo antes de pasar a algo distinto. Commit: `"Student: destructor"`.

---

## Parte F — De Struct a Class (10 min)

Crea `struct_vs_class.cpp` con este struct ya dado:

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 3 - Object-Oriented Programming, Part 1
 * Description: Converting a struct to a class with encapsulation
 * Due date: [Date]
 */

#include <iostream>
using namespace std;

struct RectangleStruct
{
    double width;
    double height;
};

// TODO (Parte F): Declara e implementa RectangleClass aqui abajo,
// convirtiendo RectangleStruct a una class con encapsulacion real:
//   - width y height como miembros privados
//   - setWidth(double) y setHeight(double) que solo acepten valores > 0
//     (si el valor no es valido, no lo asignes, y no hace falta mensaje de error aqui)
//   - getWidth() const y getHeight() const
//   - getArea() const

int main() {
    // Esto compila hoy, sin ningun problema - y ese es el problema:
    RectangleStruct r;
    r.width = -5.0;    // no tiene sentido, pero nada lo impide
    r.height = 3.0;
    cout << "Struct area (con ancho invalido): " << (r.width * r.height) << endl;

    // TODO (Parte F): Crea un objeto de tipo RectangleClass, intenta asignarle un
    // width negativo con setWidth(), y confirma que NO se acepta
    // (el area calculada con datos validos previos, si los hubo,
    // deberia quedar intacta).

    return 0;
}
```

**Esto es lo que debes entregar en esta parte:** `RectangleClass` completa, y en `main()` la prueba explícita de que un valor inválido es rechazado — no basta con que compile, tiene que demostrarse con `cout` que el rechazo realmente ocurre.

💾 **Buen momento para el commit** — completaste la conversión y la demostraste con un caso de prueba real (el rechazo del valor negativo), no solo con código que compila. Commit: `"RectangleClass: conversion de struct a class"`.

---

## Parte G — Diseño: Identificando Clases (10 min)

### Contexto

Este bloque es de **diseño**, no de lógica completa — recuerda el ejercicio de la lecture (Parte 7) sobre el restaurante, donde ya identificamos las clases candidatas usando el método de "sustantivos → clases, verbos → métodos". Aquí lo llevan a código real, pero solo como **esqueletos** (declaraciones de clase, sin implementación) — el objetivo es practicar el *proceso de decidir qué construir*, no la mecánica de escribirlo (eso ya lo practicaste a fondo en las Partes B-E).

El escenario completo, con un poco más de detalle que en la lecture:

> *Un restaurante recibe pedidos de mesas. Cada pedido tiene una lista de platos, cada uno con nombre y precio. Al final, el restaurante calcula el total del pedido, incluyendo un cargo de servicio del 10%.*
>
> Por ejemplo: la Mesa 5 pide una Ensalada César (\$8.50) y un Salmón a la Parrilla (\$22.00). El subtotal es \$30.50. El pedido completo, con el 10% de servicio, sale a \$33.55. Ese pedido tiene que poder crecer (agregar más platos mientras la mesa sigue pidiendo), y en algún momento alguien tiene que poder preguntarle "¿cuánto llevas hasta ahora?".

### El proceso mental, paso a paso (sin resolverlo por ti)

Antes de tocar el código, contesta estas preguntas en tu cabeza (o en un papel) — son el mismo tipo de pregunta que vas a tener que hacerte solo cuando diseñes el proyecto final:

1. **¿Qué necesita *saber* un `Dish`, además de nombre y precio?** Piensa en el ejemplo de arriba: ¿hace falta que un `Dish` sepa a qué mesa pertenece, o eso es responsabilidad de otra clase? ¿Dos platos con el mismo nombre y precio en dos pedidos distintos son "el mismo" `Dish`, o son dos objetos independientes?

2. **¿Cómo guarda `Order` "una lista de platos" si todavía no han visto estructuras de datos de tamaño variable?** (Esto lo van a resolver formalmente más adelante en el curso — por ahora, un array de tamaño fijo con un contador de cuántos espacios están realmente en uso es una solución razonable y suficiente.)

3. **¿Quién calcula el 10% de servicio — `Order` o `Restaurant`?** Piénsalo así: si mañana el restaurante decide subir el cargo a 15%, o cobrar un porcentaje distinto según el tipo de mesa (interior vs. terraza, por ejemplo), ¿ese cambio debería tocar el código de `Order`, o el de `Restaurant`? No hay una única respuesta correcta — pero la pregunta de "qué cambia junto y qué cambia por separado" es exactamente el criterio que se usa en diseño real para decidir dónde vive una responsabilidad.

4. **¿`Order` necesita saber a qué mesa pertenece?** Si la respuesta es sí, ¿ese dato vive en `Order`, o es más apropiado en una clase `Table` que ni siquiera pedimos que declaren hoy? (No hace falta que declaren `Table` — solo que noten que la decisión de "hasta dónde crece el diseño" también es parte del ejercicio.)

Ahora sí, a los esqueletos:

Crea `restaurant_design.cpp`:

```cpp
/*
 * Course: COEN 2220 - Programming 2
 * Name: [Your Name]
 * Lab: Lab 3 - Object-Oriented Programming, Part 1
 * Description: Class skeletons - restaurant case design
 * Due date: [Date]
 */

#include <iostream>
#include <string>
using namespace std;

// Reminder of the scenario (see "Contexto" above for the full version):
// "A restaurant takes orders from tables. Each order has a list of
//  dishes, each with a name and a price. At the end, the restaurant
//  calculates the order total, including a 10% service charge."

// TODO (Parte G): Declara la clase Dish.
//   - Miembros privados que necesita saber un plato (piensa en tu
//     respuesta a la pregunta 1 de arriba).
//   - Constructor(es) que consideres necesarios.
//   - Getters para sus atributos.
//   No hace falta implementar el cuerpo de cada funcion - un prototipo
//   dentro de la clase es suficiente para este ejercicio.

// TODO (Parte G): Declara la clase Order.
//   - Como guarda la lista de platos (ver tu respuesta a la pregunta 2).
//   - Necesita un metodo para calcular el total, incluyendo el 10%
//     de cargo de servicio - decide tu si ese calculo vive aqui o
//     en Restaurant (ver tu respuesta a la pregunta 3).

// TODO (Parte G, opcional): Declara Restaurant si decidiste que el
// calculo del cargo de servicio le pertenece a esta clase en vez de
// a Order.

int main() {
    cout << "Esqueletos de diseno - sin logica de ejecucion en este bloque.\n";
    return 0;
}
```

**Esto es lo que debes entregar en esta parte:** las declaraciones de `Dish` y `Order` (con sus prototipos, no implementación completa), y un comentario corto explicando tu decisión sobre dónde vive el cálculo del cargo de servicio — específicamente, *por qué* elegiste esa opción, no solo cuál elegiste.

<details>
<summary>Para practicar por tu cuenta: ¿por que no se pide implementacion completa aqui?</summary>

Porque el objetivo de este bloque es practicar el *proceso de diseño* (identificar clases, decidir responsabilidades) — no la mecánica de escribir código, que ya practicaste a fondo en las Partes B-E. Esta misma pregunta de diseño (¿dónde vive esta responsabilidad?) va a reaparecer, en una forma más compleja, cuando definan la arquitectura de su proyecto final.

</details>

💾 **Buen momento para el commit final** — cierra el lab con los esqueletos de diseño y tu decisión documentada. Commit: `"Restaurant design: esqueletos de Dish y Order"`.

---

## Entregable del laboratorio

Envía al profesor el enlace de tu repositorio `lab3-oop` en GitHub, con:
- Al menos **6 commits** visibles (uno por cada parte con contenido: A/B combinados y C a G).
- Un `.gitignore` funcionando (ningún ejecutable visible en el repo).
- Los archivos: `book.cpp` (dado, sin modificar), `student.cpp` (completo, todos los TODO resueltos), `struct_vs_class.cpp` (completo), y `restaurant_design.cpp` (esqueletos + comentario de decisión).
- Cada archivo con su encabezado estándar completado (nombre y fecha reales, no los placeholders).

---

## Próxima sesión

**Semana 4 — OOP Parte 2, lab correspondiente.** Van a profundizar en static members, copy constructors, y operator overloading — construyendo sobre las clases `Book` y `Student` que ya tienen funcionando.
