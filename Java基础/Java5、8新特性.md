# Java新特性总结

## 一、Java5新特性

### 1.1 枚举



### 1.2 泛型

### 1.3 注解

### 1.4 反射

## 二、Java8新特性

### 2.1 lambda表达式

> 什么是lambda表达式？

lambda表达式是一种用来简化函数式接口的写法。

> 使用lambda表达式前后对比

```java
//用于简化匿名类而诞生，本质还是new了一类，只是写法上趋于简洁。但被代替的内容必须是一个函数式接口
Thread thread4 = new Thread(()->{
    for (int i = 0; i < 10; i++) {
        System.out.println(Thread.currentThread().getName()+" "+i);
    }
});

Thread thread5 = new Thread(new Runnable() {
    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
    }
});
```

> lambda表达式语法

```java
()->{}
//（）中为方法所需参数，{}为方法体
//（）中只有一个参数时，（）可省略
//{}中只有一句代码时，{}可省略
```



### 2.2 函数式接口

> 函数式接口

**函数式接口**：只有一个抽象方法的接口

```java
@FunctionalInterface//这个注解是RunTime级别，如果结果中包含多个抽象方法，会给出提示
public interface Runnable {
    public abstract void run();
}
```

### 2.3 四大函数型接口

位于jdk中的java.util.function包下

> Function 函数型接口

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);//传入参数T，返回结果R
}

public static void main(String[] args) {
    /*Function function = new Function<String,String>() {
            @Override
            public String apply(String str) {
                return str;
            }
        };*/
    Function<String,String> function = (str)->{return str;};//函数式接口可以用lambda表达式简化
    System.out.println(function.apply("123"));
}
```

> Predicate 判定型接口

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);//传入参数T，返回结果boolean
}

public static void main(String[] args) {
    /*Predicate<String> predicate = new Predicate<String>() {
            @Override
            public boolean test(String s) {
                return s.isEmpty();
            }
        };*/
    Predicate<String> predicate = (str)->{return str.isEmpty();};
    System.out.println(predicate.test(""));
}
```

>  Consumer 消费型接口

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);//接受参数T，没有返回值
}

public static void main(String[] args) {
    /*Consumer<String> consumer = new Consumer<String>() {
            @Override
            public void accept(String o) {
                System.out.println(o);
            }
        };*/
    Consumer<String> consumer = o -> System.out.println(o);
    consumer.accept("String");
}
```

> Supplier 供给型接口

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();//没有接受参数，返回T
}

public static void main(String[] args) {
    /*Supplier<Integer> supplier = new Supplier<Integer>() {
            @Override
            public Integer get() {
                return 1024;
            }
        };*/
    Supplier<Integer> supplier = () -> 1024;
    System.out.println(supplier.get());
}
```



### 2.4 Stream流式计算

问题：

```java
public class Stream {
    /*
    * 存储交给集合
    * 计算使用流
    * 1. 筛选出序号为偶数的
    * 2. 筛选出年龄大于23的
    * 3. 把姓名转为大写
    * 4. 按照姓名倒序输出
    * 5. 只输出第一个
    */
    public static void main(String[] args) {
        User u1 = new User(1, "a", 21);
        User u2 = new User(2, "b", 22);
        User u3 = new User(3, "c", 23);
        User u4 = new User(4, "d", 24);
        User u5 = new User(6, "e", 25);
        List<User> users = Arrays.asList(u1, u2, u3, u4, u5);

        //链式编程、Stream流式计算
        users.stream()
                .filter(user-> user.getId()%2 == 0)
                .filter(user-> user.getAge()>23)
                .map(user-> user.getName().toUpperCase())
                .sorted(Comparator.reverseOrder())
                .limit(1)
                .forEach(System.out::println);
    }
}
```

分析：

```java
//Stream接口中的filter函数声明，参数为判定型接口，并返回一个流
Stream<T> filter(Predicate<? super T> predicate);

//Stream接口中的filter函数声明，参数为函数型接口，并返回一个流
<R> Stream<R> map(Function<? super T, ? extends R> mapper);

//Stream接口中的sorted函数声明
Stream<T> sorted(Comparator<? super T> comparator);

//foreach声明
void forEach(Consumer<? super T> action);

//可以看到以上方法的参数都是一个函数式接口
//这些方法呢，其都有一些相关的逻辑实现，但是例如filter，过滤的条件则在判定型接口中由编写者给出
//这样方式和并发中的Thread类，Runnable接口相似。Thread类中包含了很多线程的操作，但是这个线程具体做什么，由编写者在Runnable接口中的run方法中给出。
```



### 2.5 个人理解和反思

- 可以看到在JDK8后很多的方法其参数都是一个函数式接口，对于这些方法，调用时最原始的方式，给他传一个对应接口的实现类。
- 在JDK8后，可以使用lambda表达式来代替函数式接口的实现类（其实际还是new了这个实现类）。
- 所谓链式编程，就是一句代码内包含多个方法调用，后者方法操作前者方法的运行结果。
- Stream流计算，是对于数据的操作。集合就用来做存储、而计算交给流来完成。
- 有什么好处？不知道（还没实际运用过）。