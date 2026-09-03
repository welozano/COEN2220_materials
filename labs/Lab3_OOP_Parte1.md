# Lab 3 — Object-Oriented Programming, Parte 1
## COEN 2220 — Programming 2

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

## Parte 0 — Preparar la Carpeta y el Repositorio (10 min)

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

💾 **Commit inicial:** `"Setup inicial + gitignore"`, luego **Publish Branch**.

---

## Parte 1 — Ejemplo Guiado, Completo: la clase `Book` (20 min)

Este bloque está **completamente resuelto** — es el patrón modelo que van a replicar en la Parte 2. Léanlo con cuidado antes de avanzar, no solo lo copien.

Crea `book.cpp`:

```cpp
/*
 * Curso: COEN 2220 - Programming 2
 * Nombre: [Tu Nombre]
 * Lab: Lab 3 - Object-Oriented Programming, Parte 1
 * Descripcion: Ejemplo guiado - clase Book completa
 * Fecha de entrega: [Fecha]
 */

#include <iostream>
#include <string>
using namespace std;

class Book
{
    private:
        string title;
        string author;
        string isbn;
        bool   isCheckedOut;

    public:
        // Default constructor
        Book() {
            title = "";
            author = "";
            isbn = "";
            isCheckedOut = false;
        }

        // Constructor with parameters
        Book(string t, string a, string i) {
            title = t;
            author = a;
            isbn = i;
            isCheckedOut = false;
        }

        // Setters (with light validation)
        void setTitle(string t) {
            if (!t.empty())
                title = t;
        }

        void setAuthor(string a) {
            if (!a.empty())
                author = a;
        }

        // Getters
        string getTitle() const { return title; }
        string getAuthor() const { return author; }
        string getIsbn() const { return isbn; }
        bool   getIsCheckedOut() const { return isCheckedOut; }

        // Behavior
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

        void printInfo() const {
            cout << "Title:  " << title << "\n"
                 << "Author: " << author << "\n"
                 << "ISBN:   " << isbn << "\n"
                 << "Status: " << (isCheckedOut ? "Checked out" : "Available") << "\n";
        }
};

int main() {
    Book b1("Clean Code", "Robert C. Martin", "978-0132350884");

    cout << "--- Initial state ---\n";
    b1.printInfo();

    cout << "\n--- Checking out ---\n";
    b1.checkOut();
    b1.checkOut();   // try again - should say it's already out

    cout << "\n--- Returning ---\n";
    b1.returnBook();
    b1.returnBook(); // try again - should say it wasn't out

    return 0;
}
```

**Qué deberías ver:**
```
--- Initial state ---
Title:  Clean Code
Author: Robert C. Martin
ISBN:   978-0132350884
Status: Available

--- Checking out ---
"Clean Code" is now checked out.
"Clean Code" is already checked out.

--- Returning ---
"Clean Code" has been returned.
"Clean Code" was not checked out.
```

**Observen el patrón que van a replicar:**
1. Member variables privadas, agrupadas arriba.
2. Constructor default + constructor con parámetros (sobrecarga).
3. Setters con validación mínima, getters marcados `const`.
4. Métodos de comportamiento (`checkOut`/`returnBook`) que cambian el estado interno y dan retroalimentación.
5. Un método `printInfo()` para mostrar el objeto completo de forma consistente.

💾 **Commit:** `"Ejemplo guiado: clase Book"`

---

## Parte 2 — Ejercicio TODO: la clase `Student` (25 min)

Ahora te toca a ti, siguiendo el mismo patrón de `Book`, pero **sin la lógica ya resuelta en los comentarios** — solo la estructura.

Crea `student.cpp`:

```cpp
/*
 * Curso: COEN 2220 - Programming 2
 * Nombre: [Tu Nombre]
 * Lab: Lab 3 - Object-Oriented Programming, Parte 1
 * Descripcion: Clase Student - ejercicio con TODOs
 * Fecha de entrega: [Fecha]
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
        // TODO (Parte 2): Default constructor.
        // Inicializa name a "", id a 0, gpa a 0.0

        // TODO (Parte 3): Constructor with parameters (name, id, gpa).

        // TODO (Parte 2): setName(string n)
        // Solo asigna si n no esta vacio.

        // TODO (Parte 2): setGpa(double g)
        // Solo asigna si g esta entre 0.0 y 4.0 (inclusive).
        // Si no es valido, imprime un mensaje de error y no cambia el valor.

        // TODO (Parte 2): Getters con const: getName(), getId(), getGpa()

        // TODO (Parte 2): printInfo() const
        // Imprime name, id, y gpa con formato similar a Book::printInfo()

        // TODO (Parte 4): Destructor.
        // Imprime un mensaje indicando que el objeto Student con ese 'name' fue destruido.
};

int main() {
    // TODO (Parte 2): Crea un Student usando el constructor default,
    // asigna valores con los setters, e imprime con printInfo().

    // TODO (Parte 3): Crea un segundo Student usando el constructor con
    // parametros, con datos dummy (ej. "Alice Smith", 1001, 3.7).

    // TODO (Parte 2): Prueba setGpa() con un valor invalido (ej. 5.0)
    // y confirma que el mensaje de error aparece y el gpa no cambia.

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

💾 **Commit:** `"Student: constructor default, setters, getters, printInfo"`

---

## Parte 3 — Constructor con Parámetros (20 min)

Vuelve a `student.cpp` y completa el `TODO (Parte 3)`: el constructor que recibe `name`, `id`, y `gpa` directamente.

**Importante:** este constructor también debe validar el `gpa` — no le den un pase libre solo porque viene del constructor. Piensen cómo reusar la lógica de validación que ya escribieron en `setGpa` (pista: pueden llamar `setGpa(g);` *dentro* del constructor, en vez de repetir el `if`).

Agrega a `main()` el segundo objeto `Student` (marcado en el TODO), usando el constructor nuevo con datos dummy.

**Esto es lo que debes entregar en esta parte:** el constructor con parámetros implementado, reusando la validación de `setGpa` (no duplicada), y probado con al menos un objeto en `main()`.

<details>
<summary>Para practicar por tu cuenta: ¿por qué reusar setGpa() es mejor que repetir el if?</summary>

Si en algún momento cambian la regla de validación (por ejemplo, si el rango de GPA cambiara), solo tendrían que actualizarla en **un** lugar (`setGpa`) en vez de buscar cada copia del mismo `if` repartida por la clase. Es el mismo principio detrás de "no te repitas" (DRY) que van a ver una y otra vez el resto del semestre.

</details>

💾 **Commit:** `"Student: constructor con parametros"`

---

## Parte 4 — Destructor (15 min)

Completa el `TODO (Parte 4)`: el destructor de `Student`, que imprime un mensaje indicando qué objeto se está destruyendo (usando su `name`).

**Antes de correr el programa, traza a mano:** con los dos objetos `Student` que ya tienes en `main()` (uno del constructor default, otro con parámetros), ¿en qué orden esperas ver los mensajes del destructor al final del programa?

<details>
<summary>Ver la respuesta esperada</summary>

Los destructores se llaman en **orden inverso** a como los objetos fueron creados — el último objeto declarado es el primero en destruirse, cuando `main()` termina. Si declaraste primero el objeto con constructor default y después el de parámetros, el destructor del segundo debería imprimir su mensaje primero.

</details>

Corre el programa y confirma que el orden real coincide con tu predicción.

**Esto es lo que debes entregar en esta parte:** el destructor implementado e imprimiendo un mensaje identificable por objeto, con la traza a mano confirmada contra la salida real.

💾 **Commit:** `"Student: destructor"`

---

## Parte 5 — De Struct a Class (10 min)

Crea `struct_vs_class.cpp` con este struct ya dado:

```cpp
/*
 * Curso: COEN 2220 - Programming 2
 * Nombre: [Tu Nombre]
 * Lab: Lab 3 - Object-Oriented Programming, Parte 1
 * Descripcion: Convertir un struct a class con encapsulacion
 * Fecha de entrega: [Fecha]
 */

#include <iostream>
using namespace std;

struct RectangleStruct
{
    double width;
    double height;
};

// TODO (Parte 5): Declara e implementa RectangleClass aqui abajo,
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

    // TODO (Parte 5): Crea un RectangleClass, intenta asignarle un
    // width negativo con setWidth(), y confirma que NO se acepta
    // (el area calculada con datos validos previos, si los hubo,
    // deberia quedar intacta).

    return 0;
}
```

**Esto es lo que debes entregar en esta parte:** `RectangleClass` completa, y en `main()` la prueba explícita de que un valor inválido es rechazado — no basta con que compile, tiene que demostrarse con `cout` que el rechazo realmente ocurre.

💾 **Commit:** `"RectangleClass: conversion de struct a class"`

---

## Parte 6 — Diseño: Identificando Clases (10 min)

Este bloque es de **diseño**, no de lógica completa — recuerda el ejercicio de la lecture (Parte 7) sobre el restaurante. Ahora lo llevan a código real, pero solo como **esqueletos** (declaraciones de clase, sin implementación).

Crea `restaurant_design.cpp`:

```cpp
/*
 * Curso: COEN 2220 - Programming 2
 * Nombre: [Tu Nombre]
 * Lab: Lab 3 - Object-Oriented Programming, Parte 1
 * Descripcion: Esqueletos de clase - diseno del caso del restaurante
 * Fecha de entrega: [Fecha]
 */

#include <iostream>
#include <string>
using namespace std;

// Recordatorio del escenario (de la lecture, Parte 7):
// "Un restaurante recibe pedidos de mesas. Cada pedido tiene una lista
//  de platos, cada uno con nombre y precio. Al final, el restaurante
//  calcula el total del pedido, incluyendo un cargo de servicio del 10%."

// TODO (Parte 6): Declara la clase Dish.
//   - Miembros privados que necesita saber un plato (piensa en la
//     descripcion: nombre, precio).
//   - Constructor(es) que consideres necesarios.
//   - Getters para sus atributos.
//   No hace falta implementar el cuerpo de cada funcion - un prototipo
//   dentro de la clase es suficiente para este ejercicio.

// TODO (Parte 6): Declara la clase Order.
//   - Piensa: ¿como va a guardar "una lista de platos"? (una pista:
//     un array de Dish de tamano fijo es aceptable por ahora - las
//     estructuras de datos que permiten listas de tamano variable
//     las van a ver mas adelante en el curso).
//   - Necesita un metodo para calcular el total, incluyendo el 10%
//     de cargo de servicio - decide tu si ese calculo vive aqui o
//     en Restaurant (no hay una unica respuesta correcta).

// TODO (Parte 6, opcional): Declara Restaurant si decidiste que el
// calculo del cargo de servicio le pertenece a esta clase en vez de
// a Order.

int main() {
    cout << "Esqueletos de diseno - sin logica de ejecucion en este bloque.\n";
    return 0;
}
```

**Esto es lo que debes entregar en esta parte:** las declaraciones de `Dish` y `Order` (con sus prototipos, no implementación completa), y un comentario corto explicando tu decisión sobre dónde vive el cálculo del cargo de servicio.

<details>
<summary>Para practicar por tu cuenta: ¿por que no se pide implementacion completa aqui?</summary>

Porque el objetivo de este bloque es practicar el *proceso de diseño* (identificar clases, decidir responsabilidades) — no la mecánica de escribir código, que ya practicaste a fondo en las Partes 1-4. Esta misma pregunta de diseño (¿dónde vive esta responsabilidad?) va a reaparecer, en una forma más compleja, cuando definan la arquitectura de su proyecto final.

</details>

💾 **Commit final:** `"Restaurant design: esqueletos de Dish y Order"`

---

## Entregable del laboratorio

Envía al profesor el enlace de tu repositorio `lab3-oop` en GitHub, con:
- Al menos **6 commits** visibles (uno por cada parte con contenido: 0/1 combinados y 1 a 6).
- Un `.gitignore` funcionando (ningún ejecutable visible en el repo).
- Los archivos: `book.cpp` (dado, sin modificar), `student.cpp` (completo, todos los TODO resueltos), `struct_vs_class.cpp` (completo), y `restaurant_design.cpp` (esqueletos + comentario de decisión).
- Cada archivo con su encabezado estándar completado (nombre y fecha reales, no los placeholders).

---

## Próxima sesión

**Semana 4 — OOP Parte 2, lab correspondiente.** Van a profundizar en static members, copy constructors, y operator overloading — construyendo sobre las clases `Book` y `Student` que ya tienen funcionando.
