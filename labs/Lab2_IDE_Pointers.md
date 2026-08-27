# Lab 2 — IDE (VS Code / VSCodium) y Pointers
## COEN 2220 — Programming 2

**Duración:** 2 horas — 20 min de repaso de IDE/Git, 100 min de ejercicios de pointers
**Precede a:** N/A — este lab se basa en la lecture de Pointers y Memoria Dinámica (Semanas 1-2)
**Requisitos:** compilador de C++ instalado (deberías tenerlo desde COEN 2210 — ver la Parte 0 si necesitas reinstalarlo), cuenta de GitHub, y haber completado el Lab 1 (Git desde consola).

---

## Objetivos

Al finalizar este laboratorio, el estudiante podrá:
1. Usar el panel de Source Control de VS Code o VSCodium para hacer commit y push a GitHub, incluyendo el manejo de `.gitignore`.
2. Declarar y usar variables tipo pointer, incluyendo `&`, `*`, y `nullptr`.
3. Usar la relación entre arrays y pointers, y aritmética de punteros, correctamente.
4. Escribir una función que reciba pointers como parámetros para modificar datos del llamador.
5. Asignar y liberar memoria dinámica con `new`/`delete`, evitando el error de devolver un pointer a una variable local.
6. Convertir un raw pointer a `unique_ptr` cuando corresponde.

---

## Parte 0 — Instalación (opcional — no cuenta en el tiempo del lab)

> **El compilador ya deberías tenerlo instalado desde COEN 2210.** Si no es así (laptop nueva, reinstalación de sistema), consulta la guía de instalación del Lab 1 de Intro para el compilador (MSYS2/GCC en Windows, Command Line Tools/Clang en Mac) — no se repite aquí. Verifica rápido:

**Windows:**
```powershell
g++ --version
```
**Mac:**
```bash
clang++ --version
```
**Qué deberías ver:** un número de versión. Si sale "comando no reconocido"/"command not found", el compilador no está instalado o no está en el PATH — resuelve eso antes de seguir (guía del Lab 1 de Intro).

### Editor: VS Code o VSCodium

Este curso no exige un editor específico — **VS Code** y **VSCodium** (una versión de VS Code sin telemetría ni marca de Microsoft, basada en el mismo código fuente abierto) funcionan igual de bien para todo lo que vamos a hacer en este lab. Usa el que ya tengas instalado; si no tienes ninguno, instala cualquiera de los dos.

**Instalar VS Code:**

*Windows:*
```powershell
winget install --id Microsoft.VisualStudioCode -e --source winget
```
*Mac:*
```bash
brew install --cask visual-studio-code
```
*(Alternativa: descarga directa desde [code.visualstudio.com](https://code.visualstudio.com/))*

**Instalar VSCodium:**

*Windows:*
```powershell
winget install --id VSCodium.VSCodium -e --source winget
```
*Mac:*
```bash
brew install --cask vscodium
```
*(Alternativa: descarga directa desde [vscodium.com](https://vscodium.com/))*

### ⚠️ Nota importante si usas VSCodium: la extensión oficial de C/C++ no funciona

Desde abril de 2025, Microsoft bloqueó activamente que su extensión oficial **C/C++ (`ms-vscode.cpptools`)** funcione en VSCodium y otros forks no oficiales de VS Code — no aparece en la tienda de extensiones de VSCodium (Open VSX), y si alguien ya la tenía instalada de antes, dejó de funcionar.

**Esto no es un problema para este lab.** Todo lo que vamos a hacer (escribir, compilar, y correr código C++) funciona igual desde la terminal integrada, sin necesidad de esa extensión — la extensión solo agrega autocompletado inteligente (IntelliSense) y una interfaz gráfica para el depurador, ninguno de los dos es requisito aquí.

- **Si usas VS Code:** instala la extensión **"C/C++"** publicada por Microsoft (Extensions, `Ctrl+Shift+X`/`Cmd+Shift+X`) — es opcional, pero da autocompletado.
- **Si usas VSCodium:** puedes buscar alternativas de comunidad en Open VSX (ej. `C++ Extension Pack` de terceros), pero no es necesario para completar este lab. Vamos a compilar y correr todo desde la terminal integrada.

### Verificar que el editor y el compilador funcionan juntos

1. Abre tu editor (VS Code o VSCodium).
2. Abre la terminal integrada: `` Ctrl+` `` (Windows) / `` Cmd+` `` (Mac).
3. Corre el mismo comando de verificación de arriba (`g++ --version` o `clang++ --version`).

**Qué deberías ver:** el mismo número de versión que viste antes de abrir el editor. Si en cambio da error, cierra el editor por completo y vuelve a abrirlo (esto refresca el PATH), y prueba de nuevo.

---

## Parte 1 — VS Code/VSCodium, Git/GitHub desde cero, y `.gitignore` (20 min)

Esta parte asume que Git/GitHub es nuevo para ti — vamos a inicializar un repositorio y hacer el primer push paso a paso, sin dar nada por sabido.

### Tour rápido de la interfaz

| Parte | Dónde está | Atajo |
|---|---|---|
| **Explorer** | ícono de archivos, barra izquierda | — |
| **Editor** | panel central | — |
| **Source Control** | ícono de rama, barra izquierda | `Ctrl+Shift+G` / `Cmd+Shift+G` |
| **Terminal integrada** | `Terminal → New Terminal` | `` Ctrl+` `` / `` Cmd+` `` |

### Abre la carpeta de tu curso

1. Abre tu editor (VS Code o VSCodium).
2. `File → Open Folder...` y selecciona tu carpeta de curso (la misma carpeta padre donde vive `lab1-git/`, del Lab 1) — **no** una carpeta de lab específica todavía, la carpeta que las contiene a todas.

   > **¿Todavía no tienes esa carpeta de curso?** Créala ahora mismo desde el diálogo de "Open Folder" (la mayoría de sistemas permite crear una carpeta nueva ahí mismo, sin salir del editor) o desde tu Explorador de archivos/Finder antes de abrir el diálogo. Ponle un nombre claro (ej. `COEN2220`) y guárdala en un lugar de fácil acceso y que vayas a recordar — por ejemplo, directo en Documentos o en el Escritorio, no enterrada dentro de carpetas de descargas u otras carpetas temporales. Vas a volver a esta misma carpeta durante todo el semestre para cada lab nuevo.

**Qué deberías ver:** el `Explorer` a la izquierda muestra el nombre de tu carpeta de curso como raíz, y adentro, la carpeta `lab1-git/` (o como se llame la que usaste en el Lab 1).

### Crea la carpeta del lab

Desde la terminal integrada (ábrela con `` Ctrl+` `` / `` Cmd+` `` si no está abierta — vas a estar parado dentro de la carpeta de curso que acabas de abrir), crea la carpeta de este lab:

```bash
mkdir lab2-pointers
```

**No entres a esa carpeta desde la terminal todavía** — el siguiente paso es abrirla directamente en el editor, para evitar un error común.

### ⚠️ Abre específicamente la carpeta del lab (no la del curso)

Este paso es importante: si inicializas el repositorio de Git mientras el editor todavía tiene abierta la carpeta **del curso completo** (la carpeta padre, que contiene `lab1-git/`, `lab2-pointers/`, etc.), vas a terminar creando un repositorio que incluye *todo* — todos tus labs mezclados en un solo repo — en vez de un repositorio separado y limpio solo para este lab.

1. `File → Open Folder...` (Windows/Mac).
2. Navega hasta tu carpeta de curso, y **entra dentro de `lab2-pointers`** — selecciona esa carpeta específicamente, no la carpeta padre.
3. Clic en **Open** (Windows) / **Select as Working Directory** (algunas versiones de Mac).

**Qué deberías ver:** el `Explorer` a la izquierda ahora muestra `LAB2-POINTERS` como carpeta raíz (vacía por ahora), no el nombre de la carpeta del curso. Si ves el nombre de la carpeta del curso arriba en vez de `lab2-pointers`, repite este paso — es la señal de que abriste la carpeta equivocada.

### Crea tu primer archivo

Clic derecho sobre `LAB2-POINTERS` en el Explorer → **New File** → nómbralo `direcciones.cpp`, con este contenido mínimo, solo para tener algo que compilar y confirmar que todo funciona:

```cpp
#include <iostream>
using namespace std;

int main() {
    cout << "Lab 2 - Pointers" << endl;
    return 0;
}
```

Compila y corre desde la terminal integrada (ábrela de nuevo si se cerró al cambiar de carpeta: `` Ctrl+` `` / `` Cmd+` ``):

```bash
g++ direcciones.cpp -o direcciones
./direcciones
```
*(En Windows, puede requerir `.\direcciones.exe` en vez de `./direcciones`, dependiendo de tu terminal.)*

**Qué deberías ver:** el mensaje `Lab 2 - Pointers` impreso, y el `Explorer` ahora muestra dos archivos: `direcciones.cpp` y el ejecutable compilado (`direcciones` o `direcciones.exe`).

### Inicializar el repositorio

1. Abre el panel de **Source Control** (ícono de rama en la barra izquierda, o `Ctrl+Shift+G` / `Cmd+Shift+G`).
2. Vas a ver un botón que dice **"Initialize Repository"** — haz clic ahí.

**¿Qué hace esto?** Convierte la carpeta `lab2-pointers` en un repositorio de Git — a partir de este momento, Git empieza a llevar registro de los cambios que hagas ahí (y solo ahí, no en el resto de tu carpeta de curso, gracias a que abriste la carpeta correcta en el paso anterior).

**Qué deberías ver:** ahora aparecen `direcciones.cpp` y el ejecutable (`direcciones`/`direcciones.exe`) listados bajo **"Changes"** — Git ya está "viendo" ambos archivos, aunque todavía no les ha hecho commit a ninguno.

### El problema: Git ve el ejecutable como un archivo más

```
direcciones.cpp
direcciones          (o direcciones.exe en Windows)
```

El ejecutable **no debería subirse a GitHub** — no es código fuente, es un archivo binario generado automáticamente cada vez que compilas, distinto en cada computadora, y puede pesar bastante. Peor aún: cada vez que recompiles, Git lo va a marcar como "modificado" otra vez, ensuciando tu historial con cambios que no dicen nada útil.

### La solución: `.gitignore`

Un archivo `.gitignore` le dice a Git qué archivos o patrones de archivos **ignorar por completo** — nunca los va a mostrar como "Changes", nunca los vas a poder hacer commit por accidente.

Crea un archivo nuevo llamado exactamente `.gitignore` (con el punto al inicio, sin extensión) en la raíz de `lab2-pointers`, con este contenido:

```gitignore
# Ejecutables compilados (Windows)
*.exe

# Ejecutables compilados (Mac/Linux) - sin extensión.
# Lista aquí el nombre de cada ejecutable que generes en este lab:
direcciones
swap_demo
memoria_dinamica

# Archivos objeto intermedios (si tu compilador los genera)
*.o

# Carpeta de configuración del editor (opcional, común excluirla)
.vscode/
```

**¿Por qué no hay un patrón genérico tipo `*` sin extensión que capture *cualquier* ejecutable?** Porque eso también ignoraría carpetas y otros archivos sin extensión que sí quieras trackear — en Mac/Linux, un ejecutable no tiene una extensión que lo distinga de otro archivo cualquiera, así que hay que nombrarlo explícitamente. Es un poco más de mantenimiento (agregar una línea cada vez que crean un ejecutable nuevo con nombre distinto), pero es más seguro que un patrón demasiado amplio.

**Verifica que funcionó:** vuelve al panel de Source Control. `direcciones` (o `direcciones.exe`) ya **no debería aparecer** en la lista de "Changes" — solo `direcciones.cpp` y `.gitignore`.

> ⚠️ **Si ya habías hecho commit del ejecutable antes de crear el `.gitignore`:** agregar el archivo a `.gitignore` no lo elimina de lo que Git ya está trackeando. Tendrías que quitarlo explícitamente (`git rm --cached direcciones` desde la terminal). Para este lab, como estás empezando de cero, no deberías toparte con esto — pero es bueno saberlo para tus propios proyectos futuros.

### Primer commit

Con el `.gitignore` ya en su lugar, solo `direcciones.cpp` y `.gitignore` deberían seguir apareciendo bajo "Changes".

1. Clic en el `+` junto a **"Changes"** (arriba de la lista) — esto hace *stage* de todos los archivos a la vez.

   **¿Qué es "stage"?** Es la lista de cambios que vas a incluir en tu próximo commit. Puedes hacer stage de todos los archivos a la vez (como acabas de hacer) o de uno por uno, con el `+` que aparece junto a cada archivo individual al pasar el mouse por encima — útil cuando quieres separar cambios distintos en commits distintos.

2. Escribe un mensaje descriptivo en el cuadro de texto de arriba, por ejemplo: `"Setup inicial + gitignore"`.
3. Clic en el ✓ **Commit**.

   **¿Qué acaba de pasar?** Ese conjunto de cambios quedó guardado en el historial **local** de tu repositorio — todavía no está en GitHub, solo en tu computadora.

**Qué deberías ver:** la lista de "Changes" queda vacía — los archivos ya están guardados en el historial local.

### Primer push a GitHub

4. Busca el botón **"Publish Branch"** en la parte inferior del panel de Source Control (aparece después de tu primer commit).
5. Haz clic ahí.

   **¿Qué hace esto?** Crea automáticamente un repositorio nuevo en tu cuenta de GitHub (no necesitas ir a github.com a crearlo a mano) y sube tu commit — es el equivalente a los comandos `git remote add`, `git branch -M main`, y `git push -u` combinados en un solo botón.

   **Autenticación (solo la primera vez):** el editor va a abrir tu navegador pidiéndote iniciar sesión en GitHub y autorizar la extensión de Git. Sigue ese flujo normalmente — después de la primera vez en esa computadora, no te lo vuelve a pedir.

6. Cuando te pregunte si el repositorio será público o privado, elige **público** (recomendado para material de clase que el profesor necesita revisar).

**Qué deberías ver:** un mensaje de confirmación en el editor, y si visitas tu perfil de GitHub, un repositorio nuevo llamado `lab2-pointers` con `direcciones.cpp` y `.gitignore` adentro — **sin** el ejecutable.

A partir de la próxima vez que hagas cambios y quieras subirlos, el botón ya no dirá "Publish Branch" sino **"Sync Changes"** (o un ícono de flechas circulares) — ese es el que vas a usar el resto del lab.

Vas a repetir este ciclo (editar → compilar → stage → commit → Sync Changes) varias veces durante el resto del lab — a partir de aquí no se vuelve a explicar paso a paso, solo se recuerda con "💾 **Commit**" al final de cada parte.

---

## Parte 2 — Direcciones y Pointers Básicos (15 min)

En `direcciones.cpp`, reemplaza el contenido con:

```cpp
#include <iostream>
using namespace std;

int main() {
    int num = 25;

    // Dirección de memoria
    cout << "Direccion de num: " << &num << endl;

    // Declarar y asignar un pointer
    int *ptr = nullptr;
    ptr = &num;

    cout << "ptr apunta a: " << ptr << endl;
    cout << "Valor apuntado (*ptr): " << *ptr << endl;

    // Modificar el valor original a traves del pointer
    *ptr = 100;
    cout << "num despues de *ptr = 100: " << num << endl;

    return 0;
}
```

Compila y corre. **Qué deberías ver:** la dirección de `num` (un valor hex tipo `0x...`), el mismo valor como "ptr apunta a", `25` como valor apuntado, y `100` como valor final de `num` — confirmando que modificar `*ptr` modificó `num` directamente.

**Ejercicios (modifica el archivo y prueba cada uno):**

1. Declara `double precio = 19.99;` y un pointer `double *pptr` que apunte a `precio`. Imprime tanto la dirección como el valor apuntado.
2. ¿Qué pasa si intentas `int *malPtr = &precio;` (un pointer de `int` apuntando a un `double`)? Inténtalo y lee el error del compilador con cuidado — ¿qué te está diciendo?

<details>
<summary>Ver respuesta del ejercicio 2</summary>

El compilador da un error de tipos incompatibles (algo como *"cannot convert 'double*' to 'int*'"*). No es un error arbitrario: el compilador necesita saber el tipo exacto para calcular correctamente el tamaño de lo que hay "del otro lado" del pointer — un `int` y un `double` no ocupan lo mismo en memoria.

</details>

3. Declara un pointer `int *sinInicializar;` (sin `nullptr` ni dirección) y trata de hacer `cout << *sinInicializar;`. **No lo dejes en el código al terminar** — solo obsérvalo, comenta la línea después. ¿Qué tan predecible te pareció el resultado?

<details>
<summary>Ver una posible respuesta</summary>

El comportamiento es indefinido — puede imprimir basura, un número sin sentido, o hacer que el programa se caiga (*segmentation fault*). Es exactamente el problema que `nullptr` está diseñado para prevenir: un pointer inicializado con `nullptr` al menos falla de forma predecible y detectable (`if (!ptr)`), en vez de apuntar a memoria aleatoria.

</details>

💾 **Commit:** `"Ejercicios de direcciones y pointers basicos"`

---

## Parte 3 — Array↔Pointer y Aritmética de Punteros (20 min)

Crea un archivo nuevo `array_pointer.cpp`:

```cpp
#include <iostream>
using namespace std;

int main() {
    int vals[] = {4, 7, 11, 18, 25};

    cout << "Nombre del array (direccion): " << vals << endl;
    cout << "Primer elemento (*vals): " << *vals << endl;

    int *valptr = vals;   // valptr apunta al primer elemento

    cout << "\n--- Recorriendo con [] ---" << endl;
    for (int i = 0; i < 5; i++) {
        cout << valptr[i] << " ";
    }
    cout << endl;

    cout << "\n--- Recorriendo con aritmetica de punteros ---" << endl;
    for (int i = 0; i < 5; i++) {
        cout << *(valptr + i) << " ";
    }
    cout << endl;

    cout << "\n--- Usando ++ para avanzar el pointer ---" << endl;
    int *p = vals;
    for (int i = 0; i < 5; i++) {
        cout << *p << " ";
        p++;
    }
    cout << endl;

    return 0;
}
```

Compila y corre. **Qué deberías ver:** las tres formas de recorrer (`[]`, aritmética con `+`, e incremento `++`) imprimen exactamente los mismos 5 números — `4 7 11 18 25` — confirmando que son equivalentes.

**Ejercicios:**

1. Agrega un `cout` que imprima el **último** elemento del array usando aritmética de punteros (no `[]`), sabiendo que el array tiene 5 elementos.

<details>
<summary>Ver respuesta</summary>

```cpp
cout << *(valptr + 4);   // el índice del último elemento es tamaño - 1
```

</details>

2. Usando dos pointers (`int *inicio = vals;` y `int *fin = vals + 4;`), escribe un `cout` que imprima cuántos elementos hay **entre** ellos usando resta de pointers (`fin - inicio`). ¿El resultado que obtienes tiene sentido con la cantidad de elementos del array?

<details>
<summary>Ver una posible respuesta</summary>

`fin - inicio` da **4** — no 5. Tiene sentido: `fin` apunta al último elemento (índice 4), `inicio` al primero (índice 0), y la diferencia de *posiciones* entre ellos es 4, aunque haya 5 elementos en total (la diferencia entre índices, no la cuenta de elementos).

</details>

3. **Error común para observar (no corregir):** cambia el `for` de `i < 5` a `i < 7` en cualquiera de los tres bloques y corre el programa. ¿El compilador te detiene? ¿Qué imprime? Recuerda: ni arrays ni pointers hacen *bounds checking* en C++ — esto es exactamente por qué es peligroso.

💾 **Commit:** `"Ejercicios de array y aritmetica de punteros"`

---

## Parte 4 — Pointers como Parámetros: `swap()` (15 min)

Crea `swap_demo.cpp`:

```cpp
#include <iostream>
using namespace std;

void swap(int *x, int *y) {
    int temp = *x;
    *x = *y;
    *y = temp;
}

int main() {
    int num1 = 2, num2 = -3;

    cout << "Antes: num1=" << num1 << " num2=" << num2 << endl;
    swap(&num1, &num2);
    cout << "Despues: num1=" << num1 << " num2=" << num2 << endl;

    return 0;
}
```

Compila y corre. **Qué deberías ver:** `num1` y `num2` intercambiados después de llamar a `swap`.

**Ejercicios:**

1. Escribe una función `void duplicar(int *valor)` que multiplique por 2 el valor apuntado (sin `return` — modifica directamente a través del pointer). Pruébala con una variable en `main`.

2. Escribe una función `void ordenarPar(int *a, int *b)` que reciba dos pointers y, si `*a > *b`, los intercambie (usando la lógica de `swap` de arriba dentro de la función). Pruébala con al menos dos pares de números, uno ya ordenado y otro no.

<details>
<summary>Ver una posible solución del ejercicio 2</summary>

```cpp
void ordenarPar(int *a, int *b) {
    if (*a > *b) {
        int temp = *a;
        *a = *b;
        *b = temp;
    }
}
```

</details>

💾 **Commit:** `"Ejercicios de pointers como parametros"`

---

## Parte 5 — Memoria Dinámica: `new` y `delete` (20 min)

Crea `memoria_dinamica.cpp`:

```cpp
#include <iostream>
using namespace std;

int main() {
    // Un solo valor dinámico
    int *num = new int;
    *num = 42;
    cout << "Valor dinamico: " << *num << endl;
    delete num;
    num = nullptr;

    // Un array dinámico, con tamaño decidido en tiempo de ejecucion
    int tamano;
    cout << "\nCuantos numeros quieres guardar? ";
    cin >> tamano;

    int *arr = new int[tamano];
    for (int i = 0; i < tamano; i++) {
        arr[i] = (i + 1) * 10;
    }

    cout << "Array dinamico: ";
    for (int i = 0; i < tamano; i++) {
        cout << arr[i] << " ";
    }
    cout << endl;

    delete [] arr;
    arr = nullptr;

    return 0;
}
```

Compila y corre con distintos valores de `tamano` (prueba con `3` y con `7`). **Qué deberías ver:** el array dinámico se ajusta al tamaño que pediste — algo que un array normal (`int arr[tamano];`) no puede hacer, porque su tamaño debe conocerse al compilar, no al ejecutar.

**Ejercicios:**

1. Comenta la línea `delete [] arr;` (déjala como `// delete [] arr;`) y corre el programa varias veces. El programa igual "funciona" — ¿por qué esto sigue siendo un problema real, aunque no lo veas fallar de inmediato? *(Pista: piensa en un programa que corre por horas, no unos segundos.)*

<details>
<summary>Ver respuesta</summary>

Es un **memory leak**: cada vez que el programa pide memoria con `new[]` y nunca la libera, esa memoria queda "reservada" hasta que el programa termina por completo. En un programa que corre brevemente no se nota, pero en un programa de larga duración (un servidor, una app que queda abierta) que repite esto muchas veces, el uso de memoria crece sin parar hasta agotar los recursos del sistema. Vuelve a descomentar la línea antes de continuar.

</details>

2. Modifica el programa para pedir dos arrays dinámicos de `double` en vez de uno de `int` (por ejemplo, notas de dos exámenes distintos), y libera ambos correctamente.

💾 **Commit:** `"Ejercicios de memoria dinamica"`

---

## Parte 6 — 🐛 Encuentra el Bug: Returning Pointers (15 min)

Este ejercicio es directamente sobre el error que se marcó como crítico en la lecture. Crea `bug_returning.cpp` con este código **tal cual está** (no lo corrijas todavía):

```cpp
#include <iostream>
using namespace std;

int* triplicar(int valor) {
    int resultado = valor * 3;
    return &resultado;
}

int main() {
    int *ptr = triplicar(5);
    cout << "Resultado: " << *ptr << endl;
    return 0;
}
```

Compílalo y corre. Dependiendo de tu compilador, puede que:
- Compile con una **advertencia** (warning) que menciona "address of local variable" o similar — léela con atención si aparece.
- Imprima `15` de todas formas (por pura casualidad de cómo el sistema reutiliza esa memoria del stack en ese instante).
- Imprima basura, o falle.

**Ninguno de esos resultados hace que el código esté bien** — es comportamiento indefinido, no importa qué haya salido esta vez en tu máquina.

**Tu tarea:**

1. Identifica exactamente cuál línea viola la regla vista en la lecture ("solo devolver un pointer a datos recibidos como argumento, o a memoria dinámica — nunca a una variable local").
2. Corrígela usando memoria dinámica con `new`, siguiendo el patrón visto en la lecture.
3. Después de corregirlo, agrega en `main` la línea `delete ptr;` — ¿por qué ahora sí hace falta, y en la versión rota no tenía sentido agregarla?

<details>
<summary>Ver la solución completa</summary>

```cpp
#include <iostream>
using namespace std;

int* triplicar(int valor) {
    int *resultado = new int;
    *resultado = valor * 3;
    return resultado;
}

int main() {
    int *ptr = triplicar(5);
    cout << "Resultado: " << *ptr << endl;
    delete ptr;
    ptr = nullptr;
    return 0;
}
```

`delete ptr;` en la versión rota no tenía sentido porque `&resultado` apuntaba a una variable del *stack* — memoria que el sistema maneja automáticamente y que **nunca** se libera con `delete` (de hecho, hacerlo sería otro error grave). En la versión corregida, `resultado` vive en el *heap* (memoria dinámica) porque se creó con `new` — esa memoria sí es responsabilidad de quien la recibe, y por eso ahora `delete` es necesario para evitar un memory leak.

</details>

💾 **Commit:** `"Bug fix: returning pointers"`

---

## Parte 7 — De Raw Pointer a `unique_ptr` (15 min)

Crea `smart_pointer.cpp`. Primero, la versión con raw pointer:

```cpp
#include <iostream>
using namespace std;

int main() {
    int *edad = new int;
    *edad = 21;
    cout << "Edad: " << *edad << endl;
    delete edad;
    return 0;
}
```

Ahora conviértela a `unique_ptr`:

```cpp
#include <iostream>
#include <memory>
using namespace std;

int main() {
    unique_ptr<int> edad(new int);
    *edad = 21;
    cout << "Edad: " << *edad << endl;
    // no hace falta delete - se libera solo al salir de scope
    return 0;
}
```

Compila y corre ambas versiones — deberían imprimir lo mismo.

**Ejercicios:**

1. Convierte tu solución de la Parte 6 (`bug_returning.cpp` ya corregido) para que `triplicar` devuelva un `unique_ptr<int>` en vez de un raw pointer. *(Pista: el tipo de retorno de la función también cambia, no solo la variable en `main`.)*

<details>
<summary>Ver una posible solución</summary>

```cpp
#include <iostream>
#include <memory>
using namespace std;

unique_ptr<int> triplicar(int valor) {
    unique_ptr<int> resultado(new int);
    *resultado = valor * 3;
    return resultado;
}

int main() {
    unique_ptr<int> ptr = triplicar(5);
    cout << "Resultado: " << *ptr << endl;
    // no hace falta delete
    return 0;
}
```

</details>

2. Con base en el criterio de la lecture (raw vs. smart pointer), explica en un comentario dentro del código: ¿por qué el array dinámico de la Parte 5 (`memoria_dinamica.cpp`) también sería un buen candidato para `unique_ptr`? *(No hace falta que lo reescribas, solo justifica en una o dos líneas.)*

💾 **Commit final:** `"Conversion a unique_ptr"`

---

## Apéndice — Solución de Problemas

### "Select a debug configuration" al usar el botón ▷ (Windows)

Si te aparece una pantalla pidiendo elegir entre "gdb launch" y "windows launch", casi siempre falta `gdb` (instalado por separado de `g++` en MSYS2). Desde la terminal **"MSYS2 UCRT64"**:
```bash
pacman -S mingw-w64-ucrt-x86_64-gdb
```
Cierra todas las ventanas de terminal y del editor, vuelve a abrir, y prueba de nuevo.

### La extensión C/C++ no aparece o da error en VSCodium

Esto es esperado desde abril 2025 (ver Parte 0) — no es un problema de tu instalación. Sigue usando la terminal integrada para compilar y correr (`g++`/`clang++` manual); no necesitas la extensión para completar este lab.

### El ejecutable sigue apareciendo en Source Control después de agregarlo a `.gitignore`

Si ya le habías hecho commit antes de crear el `.gitignore`, Git lo sigue trackeando. Desde la terminal integrada:
```bash
git rm --cached nombre_del_ejecutable
```
Luego haz commit de ese cambio — a partir de ahí, `.gitignore` sí lo va a ignorar en el futuro.

---

## Entregable del laboratorio

Envía al profesor el enlace de tu repositorio `lab2-pointers` en GitHub, con:
- Al menos **6 commits** visibles (uno por cada parte con ejercicios: 2 a 7).
- Un `.gitignore` funcionando (ningún archivo ejecutable/binario visible en el repo).
- Los archivos: `direcciones.cpp`, `array_pointer.cpp`, `swap_demo.cpp`, `memoria_dinamica.cpp`, `bug_returning.cpp` (ya corregido), y `smart_pointer.cpp`.

---

## Próxima sesión

**Semana 3 — Object-Oriented Programming (parte 1), lab correspondiente.** Empezamos a combinar datos y comportamiento en clases — los pointers que acaban de practicar van a reaparecer pronto, cuando construyan sus propias estructuras de datos más adelante en el curso.
