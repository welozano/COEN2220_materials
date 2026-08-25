# Semana 1-2 — Pointers y Memoria Dinámica
## COEN 2220 — Programming 2

**Duración:** 4 horas (lectura) — 1h en semana 1, 3h en semana 2
**Precede a:** Lab 2 — IDE (VS Code + GitHub) y ejercicios de pointers/arrays dinámicos
**Basado en:** Gaddis, *Starting Out with C++*, 11th ed., Cap. 9 (Pointers)

---

## Objetivos

Al finalizar esta sesión, el estudiante podrá:
1. Explicar qué es una dirección de memoria y obtenerla con el operador `&`.
2. Declarar, inicializar y usar variables tipo pointer.
3. Usar el operador de indirección `*` para acceder al valor apuntado.
4. Explicar la relación entre arrays y pointers, y usar aritmética de punteros correctamente.
5. Pasar pointers como parámetros de función para modificar datos del llamador.
6. Asignar y liberar memoria dinámica con `new` y `delete`.
7. Identificar y evitar el error común de devolver un puntero a una variable local.
8. Decidir cuándo usar un `unique_ptr` en vez de un raw pointer.

---

# SEMANA 1 (60 min)

## Parte 0 — Puente con lo que ya saben: Reference Variables (10 min)

En Intro (COEN 2210) ya usaron esto:

```cpp
void getOrder(int &donuts)
{
    cout << "How many doughnuts do you want? ";
    cin >> donuts;
}

int main() {
    int jellyDonuts;
    getOrder(jellyDonuts);   // se pasa la variable directamente
}
```

El `&` en `int &donuts` le dice a C++: "conecta `donuts` directamente con la variable que me pasaron — lo que yo modifique adentro, se modifica afuera". Eso ya es, en el fondo, una forma de trabajar con **direcciones de memoria** — solo que C++ maneja la dirección por ustedes, de forma automática e invisible.

**Lo que viene ahora (pointers) es la versión explícita y manual de la misma idea.** Van a ver el operador `&` otra vez, pero con un significado relacionado — no idéntico — y van a tener que manejar la dirección ustedes mismos en vez de que el compilador lo haga por debajo.

> ⚠️ **Fuente clásica de confusión:** `&` en la declaración de un parámetro (`int &donuts`) es sintaxis de *reference variable*. `&` puesto antes de una variable normal (`&num`) es el operador de dirección. Se ven igual mecánicamente, pero **no son el mismo `&`** — el contexto donde aparecen es lo que determina qué hacen.

---

## Parte 1 — ¿Qué es una dirección de memoria? (15 min)

Cada variable en un programa vive en algún lugar físico de la memoria RAM. Ese lugar tiene una **dirección** — igual que cada casa en una calle tiene una dirección postal distinta.

- La **variable** es como la casa: tiene contenido (lo que vive adentro).
- La **dirección** es como la dirección postal: identifica *dónde* está esa casa, no *qué* hay adentro.

Hasta ahora, cuando escriben `int num = -99;`, nunca les ha importado *dónde* vive `num` en memoria — solo su valor. El operador de dirección `&` les permite, por primera vez, preguntarle al programa: "¿dónde exactamente vive esta variable?"

```cpp
int num = -99;
cout << &num;   // imprime la dirección de num (en hexadecimal)
                // ej: 0x7ffee2a1c9ac
```

**Nota:** la dirección se imprime en hexadecimal — justo lo que repasaron la semana pasada. No es casualidad: las direcciones de memoria son el ejemplo más común de por qué hex importa en programación real.

**Para discutir en clase:** si corren este mismo código dos veces seguidas, ¿esperan que la dirección impresa sea la misma? *(Respuesta: no necesariamente — el sistema operativo decide dónde ubicar el programa en memoria cada vez que corre.)*

---

## Parte 2 — Variables tipo Pointer (30 min)

Una **variable pointer** (o simplemente "pointer") es una variable que no guarda un valor normal — guarda una **dirección de memoria**. Como esa dirección "apunta" hacia otra variable, decimos que el pointer "apunta a" esa variable.

### Declaración

```cpp
int *intptr;   // intptr puede guardar la dirección de un int
```

Se lee: *"intptr puede apuntar a un int"*. El espaciado del asterisco no importa — estas tres formas son idénticas:

```cpp
int *intptr;
int* intptr;
int * intptr;
```

### Asignar una dirección a un pointer

```cpp
int num = 25;
int *intptr;
intptr = &num;   // ahora intptr guarda la dirección de num
```

| Variable | Contenido |
|---|---|
| `num` | `25` |
| `intptr` | `0x4a00` (la dirección de `num`) |

### Inicializar con `nullptr`

Un pointer que no apunta a nada todavía es peligroso — puede contener basura y apuntar a cualquier lugar de memoria. La convención es inicializarlo con `nullptr` (dirección "nula", introducida en C++11):

```cpp
int *ptr = nullptr;
```

Más adelante van a ver por qué esto es importante: pueden preguntar `if (!ptr)` para saber si un pointer todavía no apunta a nada válido.

**Ejercicio en clase:** declaren un `double *rate` y un `int val = 10;`, y hagan que `rate`... esperen, `rate` es de tipo `double*` — ¿pueden apuntarlo a `val`, que es `int`? Discutan por qué sí o por qué no antes de intentarlo en el compilador.

<details>
<summary>Ver respuesta</summary>

**No** — C++ no permite mezclar tipos. Un pointer declarado como `double*` solo puede apuntar a variables `double`. Esto no es una regla arbitraria: el compilador necesita saber el tipo para calcular correctamente cuántos bytes ocupa lo que está "del otro lado" del pointer.

</details>

---

# SEMANA 2 (180 min)

## Parte 3 — El Operador de Indirección `*` (20 min)

Ya usaron `&` para obtener una dirección. El operador `*` (indirección o "dereferencing") hace lo contrario: dado un pointer, les da acceso al **valor que está en esa dirección**.

```cpp
int x = 25;
int *intptr = &x;

cout << intptr;    // imprime la DIRECCIÓN de x
cout << *intptr;    // imprime 25 — el VALOR al que apunta intptr
```

**El mismo asterisco, dos trabajos distintos:**
- En una **declaración** (`int *intptr;`), el `*` dice "esta variable es un pointer".
- En una **expresión** (`*intptr`), el `*` dice "dame el valor apuntado" (dereferencing).

Y funciona en ambas direcciones — pueden usar `*intptr` para *modificar* el valor original:

```cpp
int x = 25;
int *ptr = &x;

*ptr = 100;          // esto cambia x, no solo ptr
cout << x;           // imprime 100
```

**Ejercicio en clase:** con `int a = 7, b = 3; int *p = &a;`, ¿qué imprime `cout << *p + b;`? ¿Y `*p = *p + b;` seguido de `cout << a;`?

<details>
<summary>Ver respuesta</summary>

`*p + b` → `7 + 3` = **10** (no modifica nada, solo lee).
Después de `*p = *p + b;`, `a` pasa a valer **10** — porque `*p` es literalmente `a`, escribir en `*p` es escribir en `a`.

</details>

---

## Parte 4 — La Relación Entre Arrays y Pointers (30 min)

Esto conecta con algo que ya saben de Intro: el nombre de un array, usado solo, ya es una dirección.

```cpp
int vals[] = {4, 7, 11};

cout << vals;      // imprime la dirección donde empieza el array
cout << vals[0];   // imprime 4 (el primer valor)
cout << *vals;      // imprime 4 también — dereferenciar el nombre del array
```

**El nombre de un array se comporta como un pointer constante** que apunta a su primer elemento. Y al revés: **un pointer puede usarse como si fuera un array**, con `[]` incluido:

```cpp
int *valptr = vals;      // valptr apunta al primer elemento de vals
cout << valptr[1];       // imprime 7 — igual que vals[1]
```

### La equivalencia central

```
vals[i]   es exactamente lo mismo que   *(vals + i)
```

Esto es literal en C++ — el compilador convierte `vals[i]` internamente a `*(vals + i)`. Por eso funciona con cualquier combinación de nombre de array / pointer, y con `[]` / aritmética:

| Método de acceso | Ejemplo |
|---|---|
| nombre de array + `[]` | `vals[2] = 17;` |
| pointer + `[]` | `valptr[2] = 17;` |
| nombre de array + aritmética | `*(vals + 2) = 17;` |
| pointer + aritmética | `*(valptr + 2) = 17;` |

**Ojo — advertencia importante:** ni el nombre de un array ni un pointer hacen *bounds checking*. Si escriben `vals[10]` en un array de tamaño 3, el compilador no los va a detener — van a leer o escribir memoria que no les pertenece. Este es uno de los errores más comunes (y más peligrosos) al trabajar con arrays y pointers en C++.

**Ejercicio en clase:** dado `int nums[] = {2, 4, 6, 8}; int *p = nums;`, escriban dos formas distintas de imprimir el valor `6` (el tercer elemento) — una con `[]` y otra con aritmética de punteros.

<details>
<summary>Ver una posible respuesta</summary>

```cpp
cout << p[2];         // usando []
cout << *(p + 2);     // usando aritmética
```

</details>

---

## Parte 5 — Aritmética de Punteros (30 min)

Cuando "suman 1" a un pointer, C++ no suma literalmente 1 byte — suma **el tamaño del tipo apuntado**. `valptr + 1` significa: *la dirección de valptr, más 1 vez el tamaño de un `int`*.

| Operación | Ejemplo | Efecto |
|---|---|---|
| `++`, `--` | `valptr++;` | avanza al siguiente elemento |
| `+`, `-` (pointer y entero) | `*(valptr + 2)` | accede 2 posiciones adelante, sin mover el pointer |
| `+=`, `-=` | `valptr += 2;` | mueve el pointer 2 posiciones |
| `-` (pointer menos pointer) | `valptr - vals` | cuántos elementos de diferencia hay entre ambos |

```cpp
int vals[] = {4, 7, 11};
int *valptr = vals;      // apunta a 4

valptr++;                 // ahora apunta a 7
valptr--;                 // vuelve a apuntar a 4
```

### Comparar pointers vs. comparar contenido

Un error conceptual común: `==` entre dos pointers compara **direcciones**, no lo que apuntan.

```cpp
if (ptr1 == ptr2)     // ¿apuntan al MISMO lugar?
if (*ptr1 == *ptr2)   // ¿tienen el MISMO valor?
```

Dos pointers pueden apuntar a variables distintas que casualmente tienen el mismo valor — `*ptr1 == *ptr2` sería verdadero, pero `ptr1 == ptr2` sería falso.

**Ejercicio en clase — traza a mano:**

```cpp
int x = 50, y = 60, z = 70;
int *ptr = nullptr;

ptr = &x;
*ptr *= 10;
ptr = &y;
*ptr *= 5;
ptr = &z;
*ptr *= 2;

cout << x << " " << y << " " << z;
```

¿Qué imprime?

<details>
<summary>Ver respuesta</summary>

**500 300 140** — cada vez que `ptr` se reasigna, las operaciones `*ptr *= N` afectan a la variable a la que apunta en ese momento: primero `x` (50×10=500), luego `y` (60×5=300), luego `z` (70×2=140).

</details>

---

## Parte 6 — Pointers como Parámetros de Función (25 min)

Ya vieron en la Parte 0 que reference variables permiten que una función modifique la variable original del llamador. Pointers logran lo mismo, pero de forma explícita — ustedes controlan manualmente el `*` y el `&`.

**Requiere tres piezas:**

1. Asterisco en el parámetro (prototipo y encabezado):
   ```cpp
   void getNum(int *ptr);
   ```
2. Asterisco en el cuerpo para dereferenciar:
   ```cpp
   cin >> *ptr;
   ```
3. Pasar la dirección como argumento al llamar:
   ```cpp
   getNum(&num);
   ```

### Ejemplo clásico: `swap`

```cpp
void swap(int *x, int *y)
{
    int temp;
    temp = *x;
    *x = *y;
    *y = temp;
}

int main() {
    int num1 = 2, num2 = -3;
    swap(&num1, &num2);
    // num1 ahora vale -3, num2 vale 2
}
```

Sin pointers (pasando por valor normal), `swap` solo intercambiaría copias locales — `num1` y `num2` en `main` quedarían intactos. Es exactamente el mismo problema que reference variables resuelve en Intro, pero aquí lo hacen ustedes explícitamente con `*` y `&`.

**Para discutir en clase:** ¿por qué `swap` *no puede* escribirse recibiendo `int x, int y` (sin pointers) y lograr el mismo efecto?

<details>
<summary>Ver una posible respuesta</summary>

Porque al pasar por valor, la función recibe **copias** de `num1` y `num2`. Cualquier cambio dentro de `swap` solo afecta esas copias locales — cuando la función termina, las copias desaparecen y `num1`/`num2` en `main` nunca se tocaron. Se necesita la dirección (pointer o reference) para que la función pueda alcanzar la variable original.

</details>

---

## Parte 7 — Pointers a Constantes y Constant Pointers (10 min)

Solo el vocabulario esencial — van a verlo con más profundidad cuando trabajen con parámetros de función que no deben modificar datos.

```cpp
const int SIZE = 6;
const double payRates[SIZE] = {18.55, 17.45, 12.85, 14.97, 10.35, 18.89};

// pointer a constante: el VALOR apuntado no se puede modificar
const double *rates = payRates;
```

| Declaración | Significa |
|---|---|
| `const double *rates` | `rates` puede cambiar a qué apunta, pero no puede modificar el valor apuntado |
| `double * const rates` | `rates` siempre apunta al mismo lugar, pero el valor apuntado sí se puede modificar |
| `const double * const rates` | ninguna de las dos cosas se puede modificar |

**Por qué importa ahora:** si una función solo necesita *leer* un array grande (no modificarlo), declarar el parámetro como `const double *` es una señal clara — para el compilador y para cualquiera que lea el código — de que esa función no va a alterar los datos originales.

---

## Parte 8 — Memoria Dinámica: `new` y `delete` (30 min)

Todo lo que han visto hasta ahora apunta a variables que **ya existían** (declaradas normalmente). La memoria dinámica es distinta: le piden al programa que les **cree** una variable nueva mientras el programa está corriendo, sin haberla declarado de antemano.

### Asignar memoria dinámica

```cpp
double *dptr = nullptr;
dptr = new double;      // crea un double nuevo en memoria; dptr apunta a él
```

`new` devuelve la dirección de la memoria recién creada. También funciona con arrays — y aquí es donde de verdad se vuelve útil, porque el tamaño puede depender de una variable calculada en tiempo de ejecución (algo que un array normal declarado en el código no puede hacer):

```cpp
int numDays;
cin >> numDays;

double *sales = new double[numDays];   // el tamaño lo decide el usuario, no el código
```

### Liberar memoria dinámica

Toda memoria pedida con `new` tiene que devolverse explícitamente con `delete` — si no, el programa la sigue "reservando" aunque ya no la use (esto se llama **memory leak**).

```cpp
delete dptr;         // libera una sola variable
delete [] sales;      // libera un array (nota los corchetes)

sales = nullptr;       // buena práctica: evita que quede un "dangling pointer"
```

> ⚠️ **Regla dura:** solo usen `delete` sobre memoria que ustedes mismos pidieron con `new`. Nunca sobre una variable normal ni sobre un pointer que apunta a una variable local — eso corrompe el programa.

---

## Parte 9 — ⚠️ Error Común: Devolver Pointers de Funciones (15 min)

Esta es, casi con seguridad, la fuente #1 de bugs difíciles de rastrear cuando empiecen a combinar funciones y pointers. Merece su propio momento, separado de todo lo demás.

### La regla

> **Una función solo puede devolver un pointer a:**
> **(a) datos que le pasaron como argumento, o**
> **(b) memoria asignada dinámicamente (con `new`) dentro de la función.**
>
> **Nunca a una variable local declarada dentro de la función.**

### El error, en código — se ve razonable, pero está mal

```cpp
int* crearNumero()
{
    int local = 42;
    return &local;      // ⚠️ PELIGRO — compila, pero es un error grave
}
```

**¿Por qué está mal?** `local` vive en el *stack* de la función `crearNumero`. En el momento en que la función termina (`return`), esa memoria del stack se libera y queda disponible para lo próximo que la use — el pointer que devolvieron apunta a una dirección que técnicamente ya "no es de nadie". El programa puede parecer que funciona (a veces el valor viejo sigue ahí por pura suerte), pero es comportamiento indefinido: puede fallar de forma impredecible, especialmente cuando el programa crece.

### La versión correcta, usando memoria dinámica

```cpp
int* crearNumero()
{
    int *ptr = new int;
    *ptr = 42;
    return ptr;          // ✓ memoria dinámica sobrevive fuera de la función
}
```

Aquí sí funciona, porque la memoria pedida con `new` **no** vive en el stack de la función — vive en el *heap*, y sigue existiendo hasta que alguien la libere explícitamente con `delete` (responsabilidad de quien reciba el pointer).

**Ejercicio en clase — encuentren el error:**

```cpp
int* duplicar(int valor)
{
    int resultado = valor * 2;
    return &resultado;
}
```

<details>
<summary>Ver respuesta</summary>

`resultado` es una variable local — vive en el stack de `duplicar` y desaparece cuando la función termina. Devolver `&resultado` es exactamente el error de esta sección. La corrección: usar `new int` para que la memoria sobreviva, o (mejor, cuando es posible) devolver el valor directamente por valor (`return resultado;`) en vez de devolver un pointer, si no hay una razón real para necesitar un pointer aquí.

</details>

---

## Parte 10 — Smart Pointers: `unique_ptr` (20 min)

El problema de `new`/`delete`: es fácil olvidarse del `delete`, o hacer `delete` dos veces por error, o tener una función que sale temprano (por un `return` anticipado o una excepción) y nunca llega a su `delete`. Cada uno de estos es un bug real y común en código C++ tradicional.

**`unique_ptr`** (de la librería `<memory>`, C++11) resuelve esto: es un pointer que **se borra solo** cuando sale de scope — nunca tienen que escribir `delete` ustedes mismos.

```cpp
#include <memory>

unique_ptr<int> ptr(new int);   // ptr administra la memoria automáticamente
*ptr = 99;
cout << *ptr << endl;           // 99

// al terminar el programa (o la función), la memoria se libera SOLA
```

| Parte de la sintaxis | Qué significa |
|---|---|
| `unique_ptr<int>` | un smart pointer que puede apuntar a un `int` |
| `ptr` | el nombre del pointer |
| `(new int)` | la expresión que crea la memoria dinámica |

### ¿Cuándo usar `unique_ptr` y cuándo un raw pointer?

Este es el criterio que van a aplicar el resto del curso:

| Situación | Usa |
|---|---|
| Vas a pedir memoria dinámica (`new`) y quieres que se libere sola, sin arriesgarte a un memory leak | `unique_ptr` |
| Solo necesitas apuntar temporalmente a una variable que **ya existe** (no la creaste con `new`) | raw pointer normal |
| Vas a implementar tú mismo una estructura de datos enlazada (linked list, tree) más adelante en el curso, donde cada nodo apunta al siguiente | raw pointer (por ahora — es el enfoque que usa el libro en los capítulos 22-25) |
| No estás seguro | por defecto, si hiciste `new`, usa `unique_ptr` |

**Regla práctica corta:** *si escribiste `new`, probablemente deberías estar escribiendo `unique_ptr`, no un raw pointer con `delete` manual.* La excepción son las estructuras de datos que van a construir más adelante en el curso, donde van a manejar los pointers de los nodos directamente — eso lo van a ver con detalle cuando lleguemos a Linked Lists.

---

## Resumen de la sesión

- Un pointer guarda una dirección de memoria; `&` obtiene la dirección, `*` accede al valor apuntado.
- Un array y un pointer están profundamente relacionados: `vals[i]` es literalmente `*(vals + i)`.
- Pasar pointers a funciones permite modificar variables del llamador — el mecanismo explícito detrás de lo que reference variables hacían automáticamente.
- La memoria dinámica (`new`/`delete`) permite crear datos cuyo tamaño se decide en tiempo de ejecución — pero toda memoria pedida con `new` debe liberarse explícitamente.
- **Nunca devuelvan un pointer a una variable local** — solo a argumentos recibidos o a memoria dinámica.
- `unique_ptr` automatiza el `delete` y es la opción por defecto cuando ustedes mismos piden memoria dinámica.

## Próxima sesión

**Semana 3 — Object-Oriented Programming (parte 1).** Van a ver cómo C++ permite combinar datos (como los structs que repasaron) con las funciones que operan sobre esos datos, en una sola unidad: la clase.
