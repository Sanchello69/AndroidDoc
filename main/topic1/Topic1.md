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

