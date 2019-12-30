<!-- TOC -->

- [函数式接口（`@FunctionalInterface`）](#函数式接口functionalinterface)
- [`lambda`表达式](#lambda表达式)
- [`Optional`对象](#optional对象)
- [内置的函数式接口](#内置的函数式接口)
- [Lambda表达式一些应用的场景](#lambda表达式一些应用的场景)
  - [循环遍历](#循环遍历)
  - [参数的传递行为并不仅仅是传值](#参数的传递行为并不仅仅是传值)

<!-- /TOC -->

# 函数式接口（`@FunctionalInterface`）
因为函数式编程是大势所趋，所以在Java8中引入了例如lambda等一些新的特性，使得可以将函数作为参数进行传递。而函数式接口是为了更好的使用lambda表达式而设计的一种解决方案。
“函数式接口”是指仅仅只包含一个抽象方法,但是可以有多个非抽象方法的接口。在Java8中接口中可以包含使用`default`或者`static`修饰有方法体的方法。不再局限于之前的，接口中
只能包含`public`、`static`、`final`修饰的变量、`abstract`修饰的方法。  
通常在接口中只包含一个抽象方法，像这样的接口可以被隐式的转换为lambda表达式，但是如果该接口增加了其他的抽象方法，那使用了lambda表达式的就会报错，为了显式的声明该接口就是一个函数式接口所以增加了`@FunctionalInterface`注解。比如java.util.function包下的Function接口就是一个标准的函数式接口。  
```
@FunctionalInterface
public interface Function<T, R> {

    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    static <T> Function<T, T> identity() {
        return t -> t;
    }
}

```

# `lambda`表达式
Lambda表达式的语法由参数列表、箭头符号->和函数体组成。函数体既可以是一个表达式，也可以是一个语句块。  
比如：`(int x, int y) -> x + y;`
Lambda是一个匿名函数，使用Lambda会使得代码更加简洁，使用Lambda编写的代码会被编译成一个函数式接口。

# `Optional`对象
**`Optional`是用来防止`NullPointException`的工具**，将值包装成Optional.
```
public final class Optional<T> {
    private static final Optional<?> EMPTY = new Optional<>();

    private final T value;

    private Optional() {
        this.value = null;
    }

    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }

    private Optional(T value) {
        this.value = Objects.requireNonNull(value);
    }

    public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }

    public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }

    public T get() {
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }

    public boolean isPresent() {
        return value != null;
    }

    public void ifPresent(Consumer<? super T> consumer) {
        if (value != null)
            consumer.accept(value);
    }

    public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
    }

    public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }

    public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Objects.requireNonNull(mapper.apply(value));
        }
    }

    public T orElse(T other) {
        return value != null ? value : other;
    }

    public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }

    public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
    }

}
```

# 内置的函数式接口
1. `Function<T, R>`: 接受一个输入参数，返回一个结果
2. `Consumer<T>`: 代表接收一个输入参数并无返回结果的操作
3. `Predicate<T>`: 接收一个输入参数，返回一个布尔结果
4. `Supplier<T>`: 无参数返回一个结果

**在Java8之前使用匿名内部类实现接口**
```
Function<Integer, String> function = new Function<Integer, String>() {

    @Override
    public String apply(Integer integer) {
        return "hello";
    }
};

System.out.println(function.apply(1));
```
**现在可以使用Lambda表达式实现接口**功能型函数式接口
```
Function<Integer, String> function = integer -> "hello";
System.out.println(function.apply(1));
```

**Consumer**消费型函数式接口
```
Consumer consumer = o -> System.out.println(o.toString());
consumer.accept("hello");
```

**Predicate**断言型函数式接口
```
Predicate predicate = o -> o == null;
predicate.test(null);
```

**Supplier**供给型函数式接口
```
Supplier supplier = () -> "hello";
System.out.println(supplier.get());
```

# Lambda表达式一些应用的场景
## 循环遍历
1. 外部循环：（1）顺序不可变（2）无法充分使用多核CPU
```
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
for (int number : numbers) {  
    System.out.println(number);  
} 
```
2. 内部循环：（1）顺序可以不确定（2）可以并行处理
```
List<Integer> list = Arrays.asList(1, 2, 3, 4);
list.forEach(e -> System.out.println(e));
```

## 参数的传递行为并不仅仅是传值



