# Semana 3 — Object-Oriented Programming, Parte 1: Introduction to Classes
## COEN 2220 — Programming 2

**Instructor:** Wilson Lozano

**Duración:** 170 min (sesión de 3 horas crédito nominales)
**Precede a:** Lab 3 — OOP parte 1 (clases, objetos, constructores)
**Lectura complementaria:** Gaddis, *Starting Out with C++*, 8va ed., Cap. 13 — secciones 13.1 a 13.9 (Identifying Classes, Objects, and Members; Class Declarations; Public/Private; Objects; Constructors; Destructors)

---

## Objetivos

Al finalizar esta sesión, el estudiante podrá:
1. Explicar la diferencia entre programación procedural (lo visto hasta ahora) y orientada a objetos.
2. Declarar una clase con member variables y member functions, usando `private`/`public` correctamente.
3. Escribir constructores (incluyendo sobrecargados) y destructores.
4. Explicar la diferencia entre un `struct` (ya conocido) y una `class`.
5. Aplicar el proceso básico de identificar clases y responsabilidades a partir de la descripción de un problema.

---

## Parte 0 — Puente con lo que ya saben: de Structs a Clases (10 min)

En el repaso de la Semana 1 trabajaron con este struct:

```cpp
struct Student {
    string name;
    int    id;
    double gpa;
};
```

Y una función separada para trabajar con él:

```cpp
void printStudent(Student s) {
    cout << s.name << " - GPA: " << s.gpa << endl;
}
```

Esto funciona, pero hay una desconexión: **los datos (`Student`) y la función que opera sobre ellos (`printStudent`) viven en lugares completamente separados**. Nada en el lenguaje los mantiene juntos — cualquiera podría escribir `printStudent` en otro archivo, con otro nombre, y no habría forma de saber que "pertenece" a `Student`.

Una **clase** resuelve exactamente esto: empaqueta los datos *y* las funciones que operan sobre ellos en una sola unidad.

```cpp
class Student {
    private:
        string name;
        int    id;
        double gpa;
    public:
        void print() {
            cout << name << " - GPA: " << gpa << endl;
        }
};
```

Note que `print()` ya no recibe un `Student` como parámetro — **vive dentro de la clase**, y accede directamente a `name` y `gpa` sin que se los pasen. Esa es la primera diferencia real que van a sentir hoy.

---

## Parte 1 — Procedural vs. Orientado a Objetos (10 min)

Todo el curso de Intro, y las primeras dos semanas de este curso, siguieron el estilo **procedural**: el programa es una secuencia de funciones que reciben datos, los procesan, y devuelven resultados. Los datos y las funciones son cosas separadas.

La **programación orientada a objetos (OOP)** organiza el programa distinto: en vez de "funciones que operan sobre datos", pensamos en **objetos** — unidades que combinan datos (**atributos**) y comportamiento (**métodos**) — y el programa es una colección de objetos que interactúan entre sí.

Ninguno de los dos paradigmas es "mejor" en abstracto — pero a medida que los programas crecen (como el proyecto que van a construir este semestre), OOP ayuda a mantener el código organizado: cada objeto es responsable de sus propios datos, y el resto del programa no necesita saber *cómo* los maneja internamente, solo *qué* puede pedirle que haga.

**Para discutir en clase:** en el struct `Student` de arriba, cualquier parte del programa puede hacer `s.gpa = -5.0;` — un GPA negativo no tiene sentido, pero nada lo impide. ¿Cómo creen que una clase podría prevenir esto? *(Guarden la respuesta — la Parte 2 responde exactamente esto.)*

---

## Parte 2 — Anatomía de una Clase: `private` y `public` (30 min)

### Formato general

```cpp
class ClassName
{
    private:
        // private member declarations
    public:
        // public member declarations
};
```

`private` y `public` son **especificadores de acceso** — controlan quién puede tocar cada miembro de la clase.

- **`private`**: solo las funciones miembro de la *misma* clase pueden acceder. Nada fuera de la clase puede leer ni modificar directamente.
- **`public`**: cualquier parte del programa puede acceder.

**No hay una regla que obligue a poner `private` antes que `public`**, ni a agrupar todos los miembros de un mismo tipo de acceso juntos — pero es la convención más común y la que vamos a seguir: datos privados primero, funciones públicas después.

### Ejemplo completo: la clase `Circle`

```cpp
class Circle
{
    private:
        double radius;
    public:
        void setRadius(double r);
        double getRadius() const;
        double getArea() const;
};
```

Esto responde la pregunta de la Parte 1: como `radius` es `private`, código fuera de la clase **no puede** hacer `c.radius = -5.0;` directamente — el compilador da error. La única forma de modificarlo es a través de `setRadius`, y ahí *sí* podemos poner una validación:

```cpp
void Circle::setRadius(double r)
{
    if (r >= 0)
        radius = r;
    else
    {
        cout << "Invalid radius\n";
        radius = 0;
    }
}
```

**Nota de sintaxis:** `Circle::setRadius` — el `::` (operador de resolución de ámbito) le dice al compilador "esta función que estoy definiendo aquí afuera es la que pertenece a la clase `Circle`". El código de la clase declara *qué* funciones existen; las definiciones (el cuerpo real) normalmente se escriben aparte.

### Funciones que solo leen: `const`

```cpp
double getRadius() const;
```

El `const` después de los paréntesis le dice al compilador: *"esta función no va a modificar ningún dato del objeto"*. Si por error escriben código adentro que sí modifica algo, el compilador da error — es una protección extra, no solo documentación. Van a ver esto constantemente en funciones tipo `get`.

### Las funciones públicas son la "interfaz"

Piensen en `setRadius`/`getRadius`/`getArea` como el **panel de control** del objeto — la única forma en que el resto del programa puede interactuar con un `Circle`, sin necesidad de saber que por dentro guarda un `double radius`. Este concepto — separar *cómo se usa* algo de *cómo está implementado por dentro* — es el mismo que van a ver formalizado más adelante en el curso como **interfaces** (funciones virtuales puras), y otra vez cuando lleguemos a ADT. Guarden esta idea, va a reaparecer.

**Ejercicio en clase:** declaren (sin implementar todavía) una clase `BankAccount` con un miembro privado `double balance;` y funciones públicas `deposit`, `withdraw`, y `getBalance`. Solo la declaración de la clase, con los prototipos.

<details>
<summary>Ver una posible solución</summary>

```cpp
class BankAccount
{
    private:
        double balance;
    public:
        void deposit(double amount);
        void withdraw(double amount);
        double getBalance() const;
};
```

</details>

---

## Parte 3 — Creando y Usando Objetos (20 min)

Una vez declarada la clase, crear un objeto se ve casi igual que declarar cualquier otra variable:

```cpp
int main() {
    Circle c1;
    Circle c2;

    c1.setRadius(5.0);
    c2.setRadius(2.5);

    cout << "Area of c1: " << c1.getArea() << endl;
    cout << "Area of c2: " << c2.getArea() << endl;

    return 0;
}
```

**Punto clave:** `c1` y `c2` son objetos **independientes** — cada uno tiene su propia copia de `radius`. Cambiar `c1` no afecta a `c2` en absoluto, exactamente como pasaba con dos variables `int` normales, o con dos elementos de un array de structs (Semana 1).

La sintaxis `objeto.funcion(argumentos)` debería sentirse familiar — es la misma notación de punto que ya usaban para acceder a miembros de un struct (`s.gpa`), solo que ahora en vez de acceder a un dato directamente, están llamando a una función.

**Ejercicio en clase:** usando la clase `BankAccount` que declararon en la Parte 2 (asuman que las funciones ya están implementadas correctamente), escriban código en `main` que cree dos cuentas, deposite `100.0` en la primera y `50.0` en la segunda, y luego imprima el balance de ambas.

<details>
<summary>Ver una posible solución</summary>

```cpp
BankAccount account1, account2;
account1.deposit(100.0);
account2.deposit(50.0);
cout << "Account 1: " << account1.getBalance() << endl;
cout << "Account 2: " << account2.getBalance() << endl;
```

</details>

---

## Parte 4 — Constructores (30 min)

### El problema que resuelven

Con la clase `Circle` tal como está, este código compila perfectamente:

```cpp
Circle c3;
cout << c3.getArea();   // area of what radius? never initialized
```

`radius` nunca se inicializó — el valor es basura. Un **constructor** es una función miembro especial que se ejecuta automáticamente cuando el objeto se crea, diseñada exactamente para evitar este problema.

### Sintaxis: mismo nombre que la clase, sin tipo de retorno

```cpp
class Circle
{
    private:
        double radius;
    public:
        Circle();               // constructor (no arguments)
        void setRadius(double r);
        double getRadius() const;
        double getArea() const;
};

Circle::Circle()
{
    radius = 0.0;
}
```

Ahora `Circle c3;` inicializa `radius` a `0.0` automáticamente — ya no hay basura.

### Constructor "default"

Un constructor que **no requiere argumentos** (ya sea porque no tiene parámetros, o porque todos sus parámetros tienen valor por default) se llama **constructor default**. Es el que se usa cuando escriben `Circle c3;` sin nada entre paréntesis.

**Regla importante:** si escriben una clase **sin ningún constructor**, C++ genera uno automáticamente, pero que no hace nada (los miembros quedan sin inicializar, con basura). Por eso la buena práctica es **siempre escribir al menos un constructor**, aunque sea trivial.

### Constructores con parámetros (y sobrecarga)

```cpp
class Circle
{
    private:
        double radius;
    public:
        Circle();                  // default constructor
        Circle(double r);          // constructor with parameter
        void setRadius(double r);
        double getRadius() const;
        double getArea() const;
};

Circle::Circle() {
    radius = 0.0;
}

Circle::Circle(double r) {
    radius = r;
}
```

Esto es **sobrecarga de funciones**, aplicada a constructores — dos constructores con el mismo nombre (`Circle`), distinguidos por su lista de parámetros. El compilador elige cuál usar según cómo llamen al constructor:

```cpp
Circle c1;          // calls Circle()
Circle c2(4.0);     // calls Circle(double)
```

**Regla dura:** solo puede haber **un** constructor default por clase — si escriben `Circle()` y además un constructor con *todos* sus parámetros con valor por default (ej. `Circle(double r = 0.0)`), ambos calificarían como "constructor sin argumentos requeridos", y el compilador no sabría cuál usar. Es un error de compilación.

### Objetos creados dinámicamente también llaman al constructor

```cpp
Circle *ptr = new Circle(3.0);   // the Circle(double) constructor runs here
```

Esto conecta directamente con lo que vieron en Semana 1-2: `new` no solo reserva memoria, también dispara el constructor correspondiente automáticamente.

**Ejercicio en clase:** agreguen a `BankAccount` un constructor con parámetro `double initialBalance` que inicialice `balance`, y un constructor default que inicialice `balance` a `0.0`.

<details>
<summary>Ver una posible solución</summary>

```cpp
class BankAccount
{
    private:
        double balance;
    public:
        BankAccount();
        BankAccount(double initialBalance);
        void deposit(double amount);
        void withdraw(double amount);
        double getBalance() const;
};

BankAccount::BankAccount() {
    balance = 0.0;
}

BankAccount::BankAccount(double initialBalance) {
    balance = initialBalance;
}
```

</details>

---

## Parte 5 — Destructores (15 min)

Un **destructor** es lo opuesto conceptual al constructor: una función miembro que se ejecuta automáticamente cuando el objeto **deja de existir** (sale de scope, o se hace `delete` sobre un pointer que lo apunta).

### Sintaxis: el nombre de la clase, precedido de `~`

```cpp
class Circle
{
    private:
        double radius;
    public:
        Circle();
        ~Circle();      // destructor
        // ... rest of members
};

Circle::~Circle() {
    cout << "Destroying circle with radius " << radius << endl;
}
```

**¿Para qué sirve en la práctica?** El uso más común (que van a necesitar pronto, cuando construyan sus propias estructuras de datos) es **liberar memoria dinámica** que el objeto haya reservado con `new` durante su vida — el destructor es el lugar natural para el `delete` correspondiente, garantizando que se ejecute automáticamente sin que ustedes tengan que acordarse de llamarlo a mano.

**Regla dura, igual que con el constructor default:** una clase solo puede tener **un** destructor — como no reciben argumentos, no hay forma de sobrecargarlos.

**Traza a mano:** ¿en qué orden se van a imprimir los mensajes de este programa?

```cpp
class Demo {
    public:
        Demo() { cout << "Constructor\n"; }
        ~Demo() { cout << "Destructor\n"; }
};

int main() {
    cout << "Start of main\n";
    Demo d;
    cout << "End of main\n";
    return 0;
}
```

<details>
<summary>Ver respuesta</summary>

```
Start of main
Constructor
End of main
Destructor
```

El constructor se ejecuta apenas se declara `d` (línea 2), y el destructor se ejecuta automáticamente al final de `main`, cuando `d` sale de scope — no en el orden en que aparecen escritas las clases, sino en el orden real de ejecución del programa.

</details>

---

## Parte 6 — Struct vs. Class: Comparación Directa (15 min)

Técnicamente, en C++ un `struct` puede tener funciones miembro, constructores, y hasta `private`/`public` — la diferencia real entre `struct` y `class` en el lenguaje es una sola:

| | `struct` | `class` |
|---|---|---|
| Acceso por default (si no escriben `private`/`public`) | `public` | `private` |

Eso es todo, a nivel de lenguaje. **La diferencia real es de convención**, no de capacidad técnica:

- `struct` se usa quan solo necesitas **agrupar datos** relacionados, sin comportamiento asociado (como los que ya usaron: `Point`, `Student` básico, `Date`).
- `class` se usa cuando necesitas **encapsulación real** — datos protegidos, funciones que garantizan que esos datos siempre sean válidos, y una interfaz pública clara.

**Para discutir en clase:** ¿el `Student` struct de la Semana 1 (con `name`, `id`, `gpa` públicos) sería mejor como `class`? ¿Qué ganarían exactamente?

<details>
<summary>Ver una posible respuesta</summary>

Sí sería mejor como `class` si quieren *garantizar* invariantes — por ejemplo, que `gpa` siempre esté entre 0.0 y 4.0, o que `id` nunca sea negativo. Con el struct actual, cualquier parte del programa puede asignar `s.gpa = 99.0;` sin ninguna validación. Con una clase (miembros privados + `setGpa` con validación), esa garantía queda protegida en un solo lugar del código, no dispersa por todo el programa.

</details>

---

## Parte 7 — Identificando Clases y Responsabilidades (25 min)

Cuando enfrenten un problema nuevo (como el proyecto de este semestre), rara vez alguien les va a decir explícitamente "hagan una clase `X`". Hay un método simple para identificar candidatos a partir de una descripción en lenguaje natural:

### Paso 1 — Subrayen los sustantivos

Los **sustantivos** en la descripción del problema son candidatos a clases (o a atributos, si son valores simples). Los **verbos** asociados a esos sustantivos son candidatos a métodos.

### Paso 2 — Eliminen los que son solo valores simples

Un sustantivo que representa un solo dato (un nombre, una fecha, un precio) normalmente es un **atributo** de otra clase, no una clase en sí misma.

### Ejemplo guiado

> *"Una biblioteca presta libros a estudiantes. Cada libro tiene un título, un autor, y un ISBN. Cada estudiante tiene un nombre y un número de estudiante. La biblioteca debe poder registrar cuándo un libro fue prestado y a quién."*

Sustantivos candidatos: *biblioteca, libro, estudiante, título, autor, ISBN, nombre, número de estudiante*.

Filtrando los que son solo valores simples (título, autor, ISBN, nombre, número de estudiante — todos strings/números que van a vivir *dentro* de otra clase, no como clase propia), quedan tres candidatos reales a clase: **Library, Book, Student**.

**Responsabilidades de `Book`** (lo que debe *saber* y lo que debe *hacer*):
- Saber: título, autor, ISBN, si está prestado o no.
- Hacer: marcar como prestado, marcar como disponible.

**Ejercicio en clase:** apliquen el mismo proceso a esta descripción y propongan las clases candidatas (no hace falta escribir código, solo identificar):

> *"Un restaurante recibe pedidos de mesas. Cada pedido tiene una lista de platos, cada uno con nombre y precio. Al final, el restaurante calcula el total del pedido, incluyendo un cargo de servicio del 10%."*

<details>
<summary>Ver una posible respuesta</summary>

Candidatos a clase: **Restaurant, Order, Dish, Table** (quizás). *Nombre* y *precio* son atributos simples de `Dish`, no clases propias. `Order` sería responsable de saber qué platos contiene y calcular el total (incluyendo el cargo de servicio) — una buena pregunta de seguimiento: ¿el cálculo del 10% debería ser responsabilidad de `Order` o de `Restaurant`? No hay una única respuesta correcta — es exactamente el tipo de decisión de diseño que van a enfrentar en el proyecto.

</details>

---

## Resumen de la sesión

- Una clase combina datos (member variables) y comportamiento (member functions) en una sola unidad — a diferencia de structs + funciones separadas.
- `private` protege los datos; `public` expone una interfaz controlada para acceder a ellos.
- Los constructores inicializan objetos automáticamente al crearse; los destructores limpian automáticamente al destruirse. Ambos pueden ser sobrecargados (constructores) o son únicos (destructor).
- `struct` y `class` son técnicamente casi idénticos — la diferencia es el acceso por default (`public` vs. `private`) y, sobre todo, la convención de uso.
- Identificar clases candidatas a partir de una descripción de problema es un proceso sistemático: sustantivos → clases/atributos, verbos → métodos.

## Próxima sesión

**Semana 4 — OOP Parte 2 (Cap. 14, selección): static members, copy constructors, operator overloading, y aggregation.** Van a profundizar en la mecánica interna de las clases que empezaron a construir hoy.
