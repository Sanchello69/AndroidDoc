# Generics

> Согласно официальной документации:
> 
> Обобщения — это классы, структуры, интерфейсы и методы, имеющие плейсхолдеры (параметры типов) для одного или более хранимых или применяемых типов. Класс обобщенной коллекции может использовать параметр типа как плейсхолдер для типа хранимых в нем объектов. Параметры типа выступают как типы его полей и типы параметров методов. Обобщенный метод может задействовать свой параметр типа как тип возвращаемого значения или как тип одного из его формальных параметров.

Если коротко, то Generics — это гибкая система, позволяющая работать с любыми типами данных. Она снимает с вас все заботы о типах и поручает их компилятору.

Cвойство функции, позволяющее обрабатывать значения разных типов одним способом (используя один алгоритм), называется параметрическим полиморфизмом. То есть дженерики это реализация параметрического полиморфизма в Kotlin.

```kotlin
interface MyInterface<T> {
    fun getList() : List<T>
}

class MyClass<T>(private val list: List<T>): MyInterface<T>{
    override fun getList(): List<T> = list
}

fun <E> MyClass<E>.getFirstItem() : E = this.getList()[0]
```

## Type Erasure

Так как Kotlin тесно связан с Java, мы сталкиваемся с затиранием типов (Type Erasure).
Компилятор стирает информацию о типе, заменяя все параметры без ограничений (unbounded) типом Object, а параметры с границами (bounded) — на эти границы.
Кроме стирания типов, компилятор может добавлять приведение (cast) к нужному типу и создавать переходные bridge-методы, чтобы сохранить полиморфизм в классах-наследниках.

Вот так будет выглядеть наш верхний пример в байт-коде Java:

```java
public interface MyInterface {
   @NotNull
   List getList();
}

public final class MyClass implements MyInterface {
   private final List list;

   @NotNull
   public List getList() {
      return this.list;
   }

   public MyClass(@NotNull List list) {
      Intrinsics.checkNotNullParameter(list, "list");
      super();
      this.list = list;
   }
}

public static final Object getFirstItem(@NotNull MyClass $this$getFirstItem) {
      Intrinsics.checkNotNullParameter($this$getFirstItem, "$this$getFirstItem");
      return $this$getFirstItem.getList().get(0);
   }
```

Получается в Java и Kotlin по итогу будет один класс (в отличие к примеру от C++, где для каждого типа будет свой класс), где общий тип будет просто заменен на верхнюю границу.

Получается, что из полезного с T можно делать следующее:
- использовать как тип
- использовать в as (получится, потому что вместо t as T подставится t as Any?, а это проверка всегда пройдет)

```kotlin
fun main() {
    println(example<String>(5))
}

fun <E> example(x: Int) {
    val y = x as E
    println(y)
}
```

Нельзя делать с T:
- создавать новые объекты
- проверять на is
- использовать в companion object

Многие недостатки решает модификатор inline, но это в другой раз)).

## Compile-time safety

Система типов проверяет дженерики на этапе компиляции. Если не использовать небезопасные операции, во время выполнения все будет хорошо.

<p align="center">
  <img height="300" src="https://github.com/Sanchello69/AndroidDoc/blob/main/main/topic1/res/kotlin_type_system.png">
</p>

Далее мы сталкиваемся с таким понятием как вариантность. Вариантность описывает, как обобщенные типы, типизированные классами из одной иерархии наследования, соотносятся друг с другом.

```kotlin
val strs = mutableListOf<String>()
val objs: MutableList<Any> = strs
```

Этот код не выполнится, так как дженерики в Kotlin инвариантны по умолчанию.

**Инвариантность — отсутствие наследования между производными типами.**

Инвариантность предполагает, что, если у нас есть классы Base и Derived, где Derived - производный класс от Base, то класс C Base  не является ни базовым классом для С Derived, ни производным.

```kotlin
interface Messenger<T: Message>

open class Message(val text: String)
class EmailMessage(text: String): Message(text)
```

```kotlin
fun changeMessengerToEmail(obj: Messenger<EmailMessage>){
    val messenger: Messenger<Message> = obj   // ! Ошибка
}
fun changeMessengerToDefault(obj: Messenger<Message>){
    val messenger: Messenger<EmailMessage> = obj      // ! Ошибка
}
```
    
```kotlin
fun changeMessengerToDefault(obj: Messenger<Message>){
    val messenger: Messenger<Message> = obj
}
fun changeMessengerToEmail(obj: Messenger<EmailMessage>){
    val messenger: Messenger<EmailMessage> = obj
}
```
    
**Ковариантность — это сохранение иерархии наследования исходных типов в производных типах в том же порядке.**

Ковариантость предполагает, что, если у нас есть классы Base и Derived, где Base - базовый класс для Derived, то класс SomeClass Base является базовым классом для SomeClass Derived
    
```kotlin
interface Messenger<out T: Message>
    
    
open class Message(val text: String)
class EmailMessage(text: String): Message(text)
```
    
```kotlin
fun changeMessengerToEmail(obj: Messenger<EmailMessage>){
    val messenger: Messenger<Message> = obj
}
```
    
Система типов в Kotlin ковариантна, следовательно мы можем, например, выполнить val obj: Any = 5. Аргументы функции также являются ковариантными.
    
Возвращаемое значение функции находится в контравариантной позиции.
    
**Контравариантность — это обращение иерархии исходных типов на противоположную в производных типах.**
    
Контравариантость предполагает в какой-то степени обратную ситуацию. Контравариантость предполагает, что, если у нас есть классы Base и Derived, где Base - базовый класс для Derived, то объекту SomeClass Derived  мы можем присвоить значение SomeClass Base  (при ковариантности, наоборот, - объекту SomeClass Base  можно присвоить значение SomeClass Derived)
    
Для определения обобщенного типа как контравариантного параметр обобщения определяется с ключевым словом in.
    
```kotlin
interface Messenger<in T: Message>
    
open class Message(val text: String)
class EmailMessage(text: String): Message(text)
```

```kotlin
fun changeMessengerToDefault(obj: Messenger<Message>){
    val messenger: Messenger<EmailMessage> = obj
}
```
    
Вместе с тем, инвариантные дженерики мешают выразить довольно много полезных вещей.

Например, мы хотим сделать метод для вставки списка в середину другого списка:

```kotlin
class List<T>{}

infix fun <E> List<E>.inject(other: List<E>): List<E> {
    var res = listOf<E>()
    res += this.slice(0..this.size/2)
    res += other
    res += this.slice(this.size/2+1..this.lastIndex)
    return res
}
```

Следующий код не выполнится, так как функция ожидает абсолютно одинаковый тип, как раз из-за инвариантности по умолчанию в дженериках:

```kotlin
class Item(private val name: String)
class Ad(private val id: Long)

fun main() {
    var elems: List<Any> = listOf(Item("item1"), Item("item2"), Item("item3")) //ковариантность
    elems = elems inject listOf(Ad(1), Ad(2), Ad(3)) //инвариатность (не выполнится)
}
```

Но для неизменяемого списка мы хотим иметь возможность доставать не только конкретный тип, но также и его надтипы. Это как раз и есть ковариантность. Для этого есть модификатор out. 

out T === можно доставать T или любой его супертип

```kotlin
class List<out T>{}
```

out гарантирует, что типовой параметр T только возвращается (производится), и никогда не потребляется. 

Используя out мы можем только получить некий тип T, но не писать в него. По‑другому — не скомпилится: то, что вы делаете внутри класса с таким параметром — Котлину не важно, но он четко следит за тем чтобы он не просочился наружу. Вот так получился класс, который может только отдавать значение, он условно называется «Производитель«.

Например, у нас есть клетки с животными разных типов: Cat, Dog, Bird,… все они наследуются от Animal, и нам не важно кто они, мы просто хотим всех покормить — вызвать некую функцию feedMe() на классе Animal, например.

Для выражения контравариантности есть модификатор in. Такой тип может только потребляться (передаваться на вход в качестве параметра), но не может производиться.

in T === можно передать T или любого его наследника

Таким образом, out в основном расширяется вниз, а in расширяется вверх.
    
Все верхние примеры реализуют вариантность на уровне объявления.
    
#### Итог
    
инвариантность: Type<Base> != Type<Derived>, вместо базового нельзя использовать производный и наоборот

ковариантность: вместо базового типа можно использовать производный, относится к возвращению данных из метода
    
контрвариантность: вместо производного типа можно исользовать базовый, относится к передаче параметров в методов
