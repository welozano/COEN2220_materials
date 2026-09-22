# Semana 6 — Templates, Class Templates y Abstract Data Types
## COEN 2220 — Programming 2

**Instructor:** Wilson Lozano

**Duración:** 170 min (sesión de 3 horas crédito nominales)  
**Precede a:** Lab 6 — Templates  
**Lectura complementaria:** Gaddis, *Starting Out with C++*, 8va ed., Cap. 16 — secciones 16.2–16.4 (Function Templates, Where to Start When Defining Templates, Class Templates). Las secciones sobre exceptions y STL no forman parte de este curso.

## Objetivos

Al finalizar la sesión, el estudiante podrá:

1. Distinguir entre una function template y una class template.
2. Explicar qué problema de duplicación resuelve una class template.
3. Declarar, instanciar y utilizar una class template simple con distintos tipos de datos.
4. Interpretar la sintaxis ClassName<T> dentro y fuera de una class template.
5. Identificar las operaciones que un tipo debe soportar para usarse con una template.
6. Definir un Abstract Data Type (ADT) y distinguir su interfaz de su implementación.

## Parte 0 — Puente: de una operación genérica a una colección genérica (10 min)

En la Semana 5 construyeron **function templates**. Una función como largerValue permitía escribir una regla una vez y usarla con valores de distintos tipos, siempre que esos tipos soportaran la comparación necesaria.

Una **class template** aplica la misma idea a una clase completa. En lugar de repetir una función, evita repetir data members y member functions cuando la estructura y las operaciones son iguales, pero el tipo almacenado cambia.

La pregunta de hoy es:

> Si una clase debe guardar valores y ofrecer las mismas operaciones para int y double, ¿debemos escribir una clase distinta para cada tipo?

La respuesta no siempre es “usar una template”. Primero hay que verificar que las clases realmente tienen la misma responsabilidad y que solo cambia el tipo de dato. Si sus reglas de negocio cambian, probablemente son clases distintas, no versiones de una misma template.

| Concepto | Qué se generaliza |
| --- | --- |
| Function template | Una operación, como comparar o intercambiar dos valores. |
| Class template | Una clase completa: sus datos y operaciones. |

Al final de la sesión usarán esta idea para describir una colección reutilizable. Esa colección será también el puente hacia los **Abstract Data Types (ADT)** y, más adelante, las linked lists.

## Parte 1 — El problema: dos clases casi idénticas (20 min)

Supongan que una oficina universitaria quiere guardar cantidades de asistentes a eventos y, por separado, *ratings* de esos eventos. Las cantidades son int; los *ratings* son double. Para ambos casos se desea:

- agregar un valor;
- saber cuántos valores hay;
- consultar un valor por su posición;
- saber si la colección está vacía.

Una primera solución podría crear una clase IntCollection y otra DoubleCollection. El problema es que sus responsabilidades son idénticas. Solo cambia el tipo de los valores almacenados.

~~~cpp
class IntCollection
{
private:
    int values[5];
    int count;

public:
    bool add(int value);
    int get(int index) const;
    int size() const;
    bool isEmpty() const;
};

class DoubleCollection
{
private:
    double values[5];
    int count;

public:
    bool add(double value);
    double get(int index) const;
    int size() const;
    bool isEmpty() const;
};
~~~

El código anterior no está “mal”: cada clase expresa una colección válida. Pero mantener ambas sería repetitivo. Si se descubre un error en add, habría que corregirlo dos veces. Si luego se necesita guardar char u objetos propios, aparecería una tercera clase casi igual.

Una class template permite escribir una sola clase y dejar que el compilador cree una versión apropiada cuando el programa la necesite.

> **Contexto del ejemplo:** por ahora la colección usa un arreglo con capacidad fija. No es una linked list y todavía no intenta crecer dinámicamente. Esto mantiene el foco en templates. En una semana posterior podrán cambiar la representación interna sin cambiar necesariamente las operaciones que la colección ofrece.

## Parte 2 — Declarar una class template (25 min)

Una class template comienza con un **template prefix**:

~~~cpp
template <typename T>
~~~

T es un **type parameter**: un nombre que representa el tipo que se elegirá después. El libro usa con frecuencia template <class T>; para este propósito, class y typename significan lo mismo. En esta guía usaremos typename porque comunica con claridad que T representa un tipo.

El siguiente programa define una colección genérica de capacidad fija. Puede almacenar valores int o double sin duplicar la clase.

~~~cpp
#include <iostream>
using namespace std;

template <typename T>
class FixedCollection
{
private:
    static const int CAPACITY = 5;
    T values[CAPACITY];       // Store values of the selected type.
    int count;                // Track how many positions are in use.

public:
    FixedCollection()
    {
        count = 0;            // The collection starts with no stored values.
    }

    bool add(const T& value)
    {
        if (count == CAPACITY)
        {
            return false;     // Reject the value because no space remains.
        }

        values[count] = value;
        count++;
        return true;
    }

    T get(int index) const
    {
        // Precondition: index is between 0 and count - 1.
        return values[index];
    }

    int size() const
    {
        return count;
    }

    bool isEmpty() const
    {
        return count == 0;
    }
};

int main()
{
    FixedCollection<int> attendanceCounts;
    FixedCollection<double> eventRatings;

    attendanceCounts.add(28);
    attendanceCounts.add(41);

    eventRatings.add(4.25);
    eventRatings.add(4.75);

    cout << "First attendance count: "
         << attendanceCounts.get(0) << endl;
    cout << "Second event rating: "
         << eventRatings.get(1) << endl;

    return 0;
}
~~~

**Qué deberías ver:**

~~~text
First attendance count: 28
Second event rating: 4.75
~~~

La declaración `FixedCollection<int> attendanceCounts;` crea un objeto cuya colección almacena `int`. Para ese objeto, cada aparición de `T` dentro de la clase representa `int`.

La declaración `FixedCollection<double> eventRatings;` crea otra versión de la misma idea, esta vez con `double`. No son dos objetos de exactamente el mismo tipo: uno es `FixedCollection<int>` y el otro es `FixedCollection<double>`. Ambos provienen del mismo molde, pero el compilador genera las versiones necesarias según el tipo indicado entre `< >`.

> **Importante:** FixedCollection no es un objeto completo hasta que se indica un tipo. Deben escribir `FixedCollection<int>`, `FixedCollection<double>` u otro tipo válido.

<details>
<summary>Pregunta de verificación: ¿por qué <code>count</code> sigue siendo <code>int</code> y no <code>T</code>?</summary>

count representa cuántos elementos hay en la colección. Esa cantidad siempre es un número entero, aunque los valores guardados sean double, char u objetos de una clase propia. Solo los valores almacenados necesitan el tipo genérico T.
</details>

## Parte 3 — Definiciones fuera de la clase y la sintaxis ClassName<T>:: (20 min)

En OOP ya han visto que una member function puede declararse dentro de una clase y definirse después. Con una class template, la definición externa debe repetir el template prefix y especificar que la función pertenece a la versión genérica de la clase.

Por ejemplo:

~~~cpp
template <typename T>
bool FixedCollection<T>::add(const T& value)
~~~

- template <typename T> declara nuevamente el type parameter.
- FixedCollection<T>:: indica que add pertenece a la class template.
- const T& value recibe un valor del tipo elegido sin copiarlo innecesariamente.

El siguiente programa es equivalente al de la parte anterior, pero organiza las definiciones fuera de la clase.

~~~cpp
#include <iostream>
using namespace std;

template <typename T>
class FixedCollection
{
private:
    static const int CAPACITY = 5;
    T values[CAPACITY];       // Store values of the selected type.
    int count;                // Track how many positions are in use.

public:
    FixedCollection();
    bool add(const T& value);
    T get(int index) const;
    int size() const;
    bool isEmpty() const;
};

template <typename T>
FixedCollection<T>::FixedCollection()
{
    count = 0;                // A new collection has no stored values.
}

template <typename T>
bool FixedCollection<T>::add(const T& value)
{
    if (count == CAPACITY)
    {
        return false;         // Do not write past the fixed array capacity.
    }

    values[count] = value;
    count++;
    return true;
}

template <typename T>
T FixedCollection<T>::get(int index) const
{
    // Precondition: index is between 0 and count - 1.
    return values[index];
}

template <typename T>
int FixedCollection<T>::size() const
{
    return count;
}

template <typename T>
bool FixedCollection<T>::isEmpty() const
{
    return count == 0;
}

int main()
{
    FixedCollection<int> roomNumbers;

    roomNumbers.add(101);
    roomNumbers.add(204);

    cout << "Stored rooms: " << roomNumbers.size() << endl;
    cout << "First room: " << roomNumbers.get(0) << endl;

    return 0;
}
~~~

**Qué deberías ver:**

~~~text
Stored rooms: 2
First room: 101
~~~

Las definiciones de templates deben estar disponibles cuando el compilador encuentra un uso de la template. Por esa razón, en este curso mantendrán la class template y sus definiciones en el mismo archivo .cpp o en el mismo header. Separar una template en un .h y un .cpp como una clase ordinaria puede causar errores de compilación si el compilador no ve la definición al necesitar generar una versión para un tipo.

<details>
<summary>Pregunta de verificación: ¿por qué <code>FixedCollection&lt;T&gt;::add</code> incluye <code>&lt;T&gt;</code>?</summary>

Porque FixedCollection por sí sola no identifica una versión completa de la clase. FixedCollection<T> indica que la función pertenece a la versión genérica que usa el type parameter T.
</details>

## Parte 4 — Una template tiene requisitos sobre el tipo T (15 min)

Decir que una template funciona con “cualquier tipo” es una simplificación. Una template funciona con los tipos que soportan las operaciones que la template intenta realizar.

Hasta ahora, `FixedCollection<T>` puede almacenar muchos tipos porque `add` solamente asigna el valor recibido a una posición del arreglo. Pero supongan que la oficina quiere preguntar si ya registró cierto *rating* antes de agregarlo otra vez. Esa necesidad requiere añadir una operación `contains` a la misma clase `FixedCollection<T>` usada en las partes anteriores.

El siguiente método se agrega en la sección `public` de esa clase. Puede usar `values` y `count` porque es una member function de `FixedCollection<T>`:

~~~cpp
bool contains(const T& target) const
{
    for (int index = 0; index < count; index++)
    {
        if (values[index] == target)
        {
            return true;     // The target value was found.
        }
    }

    return false;            // No stored value matched the target.
}
~~~

Con una colección de valores numéricos, la llamada es natural porque C++ ya sabe comparar `int` y `double` con `==`:

~~~cpp
if (eventRatings.contains(4.75))
{
    cout << "Rating already stored." << endl;
}
~~~

Ahora conectemos el mismo método con una clase propia. Si en el futuro se crea una `FixedCollection<Book>`, la operación `contains` intenta evaluar `storedBook == targetBook`. C++ no puede decidir por sí solo cuándo dos libros deben considerarse iguales. La clase `Book` debe definir esa regla, por ejemplo comparando su ISBN:

~~~cpp
#include <string>

class Book
{
private:
    std::string isbn;

public:
    Book(const std::string& bookIsbn)
    {
        isbn = bookIsbn;
    }

    bool operator==(const Book& other) const
    {
        return isbn == other.isbn;  // Books match when their ISBN values match.
    }
};
~~~

Esta operator overload retoma la Semana 4. Una vez que `Book` define `operator==`, la misma función `contains` puede comparar objetos `Book`. Sin esa operación, el compilador indicaría que no existe una comparación válida entre dos objetos `Book`.

| Operación dentro de una template | Requisito para T |
| --- | --- |
| first + second | Debe soportar +. |
| first < second | Debe soportar <. |
| stored == target | Debe soportar ==. |
| cout << stored | Debe poder enviarse a <<. |

Esto conecta con la Semana 4. Si un objeto de una clase propia necesita una operación que C++ no conoce automáticamente, el programador puede definir una operator overload apropiada. La template no “adivina” cómo comparar o imprimir un objeto.

Por ejemplo, si más adelante una FixedCollection<Book> necesitara contains, la clase Book tendría que ofrecer una comparación válida con ==. No es suficiente que ambas variables sean objetos de tipo Book.

> **Límite de esta sesión:** no trabajarán con template specialization, templates con herencia ni templates avanzadas. El objetivo es comprender una class template simple y reutilizable.

## Parte 5 — Abstract Data Types: el contrato antes de la estructura (25 min)

Antes de definir un ADT, aclaremos algo importante: un ADT no es una cuarta clase de C++ con una palabra reservada propia. Por eso no verán una línea como `adt Collection` ni un símbolo comparable a `template <typename T>` o `= 0`.

| Idea | Cómo se reconoce en código C++ | Qué representa |
| --- | --- | --- |
| Clase | `class Book { ... };` | Un tipo con data members y member functions. |
| Abstract class | Al menos una pure virtual function, como `virtual void print() const = 0;` | Una clase que no puede instanciarse directamente. |
| Class template | Un template prefix, como `template <typename T>` | Un molde para generar clases con distintos tipos. |
| ADT | No tiene sintaxis propia. Se expresa con una especificación y una interfaz pública. | Un contrato de datos y operaciones. |

Un **Abstract Data Type (ADT)** es una idea de diseño que se define antes de escribir la implementación. Especifica qué conjunto de datos se manejará y qué operaciones puede solicitar el resto del programa. No decide todavía si esos datos vivirán en un arreglo, una linked list, una queue o un tree.

En este contexto, un **contrato** es una descripción clara de lo que cada operación promete hacer y de las condiciones para usarla correctamente. El contrato incluye el nombre de la operación, sus parámetros, su resultado y sus reglas. Por ejemplo, el contrato de `get(index)` puede decir: “devuelve el elemento en la posición indicada, siempre que `index` sea una posición válida”.

El contrato establece responsabilidades para ambos lados:

- Quien usa la clase debe llamar `get` con un índice válido.
- Quien implementa la clase debe devolver el elemento correcto cuando el índice es válido.

El contrato no revela si el elemento se busca en un arreglo o en una linked list. Describe el comportamiento visible, no los detalles internos.

El flujo de diseño es el siguiente:

~~~text
1. Identify the program need
       ↓
2. Define the ADT contract
   What data is managed? What operations are allowed?
       ↓
3. Choose a concrete implementation
   Array, linked list, queue, tree, and so on
       ↓
4. Write client code
   Use only the public operations of the implementation
~~~

Por ejemplo, para guardar *ratings* de eventos, primero se identifica la necesidad: agregar un *rating*, consultar uno por posición, saber cuántos existen y verificar si todavía no hay ninguno. Ese conjunto de datos y operaciones puede llamarse conceptualmente el **Collection ADT**.

La clase `FixedCollection<T>` que construyeron en las partes anteriores **no declara el ADT**; es una class template concreta que implementa una primera versión del Collection ADT. Sus data members privados `values` y `count` muestran cómo esta versión guarda los datos: con un arreglo fijo y un contador. Sus member functions públicas expresan el contrato que el resto del programa puede usar.

Por tanto, ADT y class template responden preguntas diferentes:

| Concepto | Pregunta que responde |
| --- | --- |
| ADT | ¿Qué datos y operaciones necesita ofrecer esta estructura? |
| Class template | ¿Cómo puedo reutilizar una misma implementación con distintos tipos de datos? |

Un ADT puede implementarse con una clase ordinaria. Por ejemplo, una clase `TemperatureLog` que solo guarda valores `double` puede implementar un Collection ADT aunque no use una template. Por otro lado, una class template no se convierte automáticamente en un ADT solo por tener `T`; una template es una herramienta del lenguaje para generalizar código.

Se complementan cuando usamos una class template, como `FixedCollection<T>`, para implementar el mismo Collection ADT con distintos tipos de elementos. Su interfaz pública propone este contrato:

| Operación | Propósito | Condición importante |
| --- | --- | --- |
| add(value) | Agrega un valor a la colección. | Debe haber espacio disponible. |
| get(index) | Devuelve el valor en una posición. | index debe ser válido. |
| size() | Devuelve cuántos valores se almacenan. | No cambia la colección. |
| isEmpty() | Indica si no hay valores. | No cambia la colección. |

El **programador que escribe código cliente** necesita saber cómo llamar esas funciones y qué puede esperar de ellas. No necesita conocer todos los detalles internos. No estamos hablando aquí del usuario final de la cafetería; hablamos del programador que escribe otra parte del mismo programa. Por ejemplo, ese código cliente puede llamar:

~~~cpp
eventRatings.add(4.75);
~~~

sin necesitar saber si internamente se usa un arreglo, una linked list u otra estructura.

En el ejemplo actual, la implementación usa:

~~~text
FixedCollection<T>
- fixed-size array
- count
~~~

Eso es una decisión de implementación, no la definición completa de la idea de “colección”.

> **No confundan ADT con abstract class.** Una abstract class es una característica concreta de C++ que estudiaron en la Semana 5. Un ADT es un contrato conceptual de operaciones. A veces una abstract class puede ayudar a expresar una interfaz, pero un ADT no exige usar inheritance ni pure virtual functions.

También deben distinguir estas dos preguntas:

| Pregunta de diseño | Ejemplo |
| --- | --- |
| ¿Qué operaciones necesito? | Agregar, consultar, saber tamaño. |
| ¿Cómo las implementaré? | Con arreglo fijo, arreglo dinámico o linked list. |

Primero se decide la responsabilidad y las operaciones. Después se escoge una estructura apropiada para implementarlas.

<details>
<summary>Pregunta de verificación: ¿por qué <code>add</code>, <code>get</code> y <code>size</code> pueden mantenerse aunque se cambie un arreglo por una linked list?</summary>

Porque describen lo que la colección ofrece a quien la utiliza. La forma interna de guardar los datos puede cambiar, siempre que la nueva implementación respete el mismo contrato de operaciones.
</details>

### Ejemplo completo: definir, implementar y usar un ADT

Veamos exactamente dónde aparece cada idea en el ejemplo de *ratings*.

Como un ADT es conceptual, se documenta mediante una **especificación de ADT**, no mediante una palabra reservada de C++. Para este curso, una especificación debe indicar como mínimo:

1. El nombre del ADT.
2. Los datos que el ADT representa o administra.
3. Las operaciones disponibles.
4. Para cada operación, qué recibe, qué devuelve o cambia, y sus reglas importantes, como precondiciones.

El siguiente bloque es, por tanto, el **formato de especificación** del Collection ADT. Todavía no es código C++.

**Paso 1 — Aquí se define el ADT.** Antes de escribir C++, el diseñador escribe la especificación del Collection ADT. Esta especificación puede aparecer en el diseño del programa, en la documentación o en los comentarios de un header; no es una instrucción que el compilador ejecute.

~~~text
Collection ADT

Data: an ordered group of values

Operations:
- add(value): store one value if space is available
- get(index): return the value at a valid position
- size(): return the number of stored values
- isEmpty(): report whether no values are stored
~~~

**Paso 2 — Aquí se implementa el ADT en C++.** La class template `FixedCollection<T>` convierte ese contrato en código. Sus member functions públicas implementan las operaciones del ADT; sus data members privados representan una decisión de implementación: usar un arreglo fijo y un contador.

~~~cpp
template <typename T>
class FixedCollection
{
private:
    static const int CAPACITY = 5;
    T values[CAPACITY];       // Store the collection values in an array.
    int count;                // Track how many values are stored.

public:
    bool add(const T& value);
    T get(int index) const;
    int size() const;
    bool isEmpty() const;
};
~~~

**Paso 3 — Aquí se usa el ADT.** El programador que escribe `main` no accede a `values` ni cambia `count`. Crea una implementación concreta y usa únicamente las operaciones públicas prometidas por el contrato.

~~~cpp
FixedCollection<double> eventRatings;

eventRatings.add(4.75);
eventRatings.add(4.25);

cout << eventRatings.get(0) << endl;
cout << eventRatings.size() << endl;
~~~

En este ejemplo, el diseñador definió el Collection ADT en el Paso 1; `FixedCollection<double>` es la implementación concreta elegida en el Paso 2; y el código en `main` es el cliente que usa el ADT en el Paso 3. Si más adelante se crea una `LinkedList<double>` con las mismas operaciones públicas, el programador podrá reconocer el mismo contrato aunque cambie la forma interna de guardar los *ratings*.

#### El mismo razonamiento con una clase normal

Una template no es obligatoria para aplicar un ADT. Imaginen un programa pequeño y separado, usado por el personal de Campus Facilities para registrar la temperatura de un salón cada mañana durante una semana. El programa solo necesita almacenar hasta siete temperaturas, y cada lectura siempre es un valor `double`.

Para ese escenario, el diseñador puede definir un TemperatureLog ADT con operaciones como agregar una temperatura, consultar cuántas se han registrado y verificar si el registro está vacío.

~~~text
TemperatureLog ADT

Data: up to seven temperature readings for one classroom during one week

Operations:
- add(temperature): store one reading if space is available
- size(): return the number of stored readings
- isEmpty(): report whether no readings are stored
~~~

La siguiente clase normal implementa ese ADT. No tiene `template <typename T>` porque su propósito es guardar solamente temperaturas:

~~~cpp
class TemperatureLog
{
private:
    static const int CAPACITY = 7;
    double temperatures[CAPACITY];  // Store only temperature values.
    int count;

public:
    TemperatureLog()
    {
        count = 0;
    }

    bool add(double temperature)
    {
        if (count == CAPACITY)
        {
            return false;           // Reject a value when the log is full.
        }

        temperatures[count] = temperature;
        count++;
        return true;
    }

    int size() const
    {
        return count;
    }

    bool isEmpty() const
    {
        return count == 0;
    }
};
~~~

El código cliente usa el ADT mediante sus operaciones públicas:

~~~cpp
TemperatureLog morningTemperatures;

morningTemperatures.add(72.5);
cout << morningTemperatures.size() << endl;
~~~

Aquí `TemperatureLog` es una clase ordinaria que implementa un ADT específico para `double`. `FixedCollection<T>` implementa una idea parecida, pero usa una class template para reutilizarla con varios tipos. El ADT sigue siendo el contrato conceptual en ambos casos.

## Parte 6 — Puente al proyecto y a las linked lists (20 min)

El proyecto de cafetería requiere administrar colecciones de órdenes e ítems. Por ejemplo, una Order tiene varios OrderItem, y el sistema necesita mantener órdenes activas.

Hoy no implementarán una linked list. En cambio, usarán el razonamiento que precede a su implementación:

~~~text
What does the collection need to do?

- Add an item
- Remove an item
- Find an item
- Report its size
- Check whether it is empty
~~~

Una colección de capacidad fija, como FixedCollection<T>, puede servir para practicar la interfaz y la sintaxis de templates. Sin embargo, tiene una limitación clara: no puede crecer más allá de su capacidad.

Más adelante, una LinkedList<T> podrá ofrecer operaciones similares con una representación diferente:

| Colección | Representación | Limitación o ventaja principal |
| --- | --- | --- |
| FixedCollection<T> | Arreglo de capacidad fija | Simple, pero limitada por CAPACITY. |
| LinkedList<T> | Nodos conectados mediante pointers | Puede crecer dinámicamente. |

En el proyecto, una futura LinkedList<OrderItem> o LinkedList<Order*> debe justificarse por una necesidad del sistema, no por el simple hecho de que existe una template. La discusión de ownership y lifetime de los pointers se retomará cuando trabajen con las estructuras dinámicas.

El diseño correcto empieza con el contrato del ADT. Después se decide si un arreglo fijo, una lista enlazada, una queue, una stack o un tree satisface mejor ese contrato.

## Parte 7 — Diseño guiado y resumen (35 min)

Trabajen en parejas durante unos minutos. Sin escribir todavía una implementación completa, diseñen la interfaz para una colección de ítems de una orden de cafetería.

La clase podría llamarse OrderItemCollection. Respondan:

1. ¿Qué datos deben ser private?
2. ¿Qué operaciones necesita el resto del programa?
3. ¿Cuáles operaciones cambian la colección y cuáles deben ser const?
4. ¿Qué precondición tendría una operación que obtiene un ítem por posición?
5. Si en el futuro se reemplaza un arreglo por una linked list, ¿qué operaciones públicas deberían conservarse?

<details>
<summary>Ver una posible interfaz</summary>

~~~cpp
class OrderItemCollection
{
public:
    bool add(const OrderItem& item);
    bool remove(int index);
    OrderItem get(int index) const;
    int size() const;
    bool isEmpty() const;
};
~~~

La respuesta no es la única interfaz válida. Lo importante es que las operaciones expresen una responsabilidad clara y que la representación interna permanezca privada.
</details>

En la discusión final, relacionen las ideas principales:

- Una function template reutiliza una operación; una class template reutiliza una clase.
- T representa un tipo que se elegirá al crear un objeto de la template.
- Una template puede requerir que el tipo soporte ciertas operaciones.
- Un ADT define qué operaciones ofrece una estructura antes de decidir cómo se guardarán los datos.
- En el proyecto, y más adelante con linked lists, deberán justificar tanto la interfaz como la implementación.

Los últimos 15 minutos de esta parte se reservan para preguntas, recompilar ejemplos y corregir confusiones antes del Lab 6.

## Resumen de la sesión

- Una class template evita duplicar una clase cuando su estructura y operaciones son iguales, pero el tipo almacenado cambia.
- `FixedCollection<int>` y `FixedCollection<double>` son versiones distintas generadas desde la misma class template.
- Las definiciones externas de una template requieren tanto el template prefix como ClassName<T>::.
- Una template solo funciona con tipos que soportan las operaciones que la template utiliza.
- Un ADT describe un contrato de operaciones; no depende de una representación interna específica.
- Las linked lists de semanas posteriores permitirán implementar colecciones dinámicas con ideas de interfaz similares.

## Próxima sesión

En el Lab 6 construirán y probarán una class template pequeña. Después profundizarán en ADT y prepararán el vocabulario de diseño necesario para las linked lists, stacks y queues.
