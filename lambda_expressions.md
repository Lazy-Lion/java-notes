# Lambda Expression in java 8
## 一、嵌套类(Nested Class)
### 1.1 概述
java允许在一个类(外部类,enclosing class)里定义类(嵌套类,nested class)。
```java
class OuterClass{
    ...
    class InnerClass{  // 非静态嵌套类(内部类)
        ...
    }

    static class StaticNestedClass{  // 静态嵌套类
        
    }
} 
```

嵌套类是外部类的成员。嵌套类可以使用public,protected,private,package private(default)修饰可见性，外部类只能使用public,package private(default)。

### 1.2 静态嵌套类(staic nested class)和内部类(inner class)
##### 1.2.1 静态嵌套类
```java
/**
 * 创建静态嵌套类对象。
 * 静态嵌套类不能直接访问外部类的非静态域和方法，其与外部类实例之间不存在直接联系，非
 * 静态成员的访问和非嵌套类之间的访问一致，需要通过外部类的实例化对象
 * 访问(可以访问private成员)。
 */
OuterClass.StaticInnerClass staticClassObject = new OuterClass.StaticInnerClass(); 
```

##### 1.2.2 内部类
```java 
/**
 * 创建内部类对象
 * 内部类的对象实例依存于外部类的实例，所以实例化对象之前必须先实例化外部类对象。
 * 由于内部类(Inner Class)依存于外部类实例，所以不能定义静态方法和变量，但是可以直接
 * 访问外部类的任意成员。(例外，内部类可以有静态的常量变量(constant variables)。)
 */
OuterClass outerClass = new OuterClass();
OuterClass.InnerClass innerClass = outerClass.new InnerClass();
```

> Note: **A constant variable** is a variable of primitive type or type String that is declared final and initialized with a compile-time constant expression. A compile-time constant expression is typically a string or an arithmetic expression that can be evaluated at compile time. 

##### 1.2.3 隐藏(Shadowing)
如果某个特定范围和外部范围具有相同名称的类型声明，则外部范围的声明会被隐藏，仅通过名称无法直接访问被隐藏的声明。
```java
class OuterClass{
    private int a = 1;

    class InnerClass{
        int a = 2;
        public void get(int a){
            System.out.println(a);
            System.out.println(this.a);
            System.out.println(OuterClass.this.a);
        }
    }
} 
```
调用get(3),输出结果分别为 3,2,1

## 二、局部类(Local Class)
局部类是一种特殊的内部类。在语句块(如方法体,for循环体，if子句等)中定义的类称为局部类。
```java
public class Main2 {
    static String str = "Outer class";

    public static void main(String[] args){
        get("a");
    }

    public static void get(String param){

        String local = "local variable";

        /**
         * java 8 局部类可以访问外部语句块的局部变量和参数，但是需要局部变量
         * 和参数是final或 effectively final。(Effective final 指的是变量或
         * 参数初始化之后不会改变，即拥有final的含义但没有使用final修饰。实际
         * 编译之后依然是final, 只是一个语法糖。)
         * 
         * 如下两行如果取消注释，则Local Class中get()方法访问local,param时编译
         * 错误"local variables is accessed from within inner class, need to 
         * be final or effectively final"
         */
//      local = "set local";   
//      param = "set a";
        class LocalClass{

            public String get(String str){
                System.out.println(param);
                System.out.println(local);
                System.out.println(Main2.str);
                System.out.println(str);
                return str;
            }
        }

        LocalClass l = new LocalClass();
        l.get("local class");
    }
} 
```

局部类和前文介绍的内部类具有相似的特性：不能定义静态成员，但是可以有静态的常量变量(constant variables)。<br />
定义在不同方法中的局部类对于外部类成员的可访问性不同
- 定义在static方法中: 无法访问外部类的非静态成员
- 定义在实例方法中： 可以访问外部类的任意成员

## 三、匿名类(Anonymous Class)
匿名类是一种特殊的内部类。和局部类的区别在于匿名类没有名字，如果局部类只需要使用一次，可以使用匿名类替代。
```java
public class AnonymousClass {
    public void func(){
        final int b = 3;

        Runnable r = new Runnable() {
            int a = 2;

            @Override
            public void run() {
                System.out.println(a + b);
            }
        };
    }
} 
```
上述代码展示了匿名类的创建，匿名类是一个表达式，该表达式包括如下内容：
- new 操作符
- 实现接口或继承类的名字
- 构造器参数：由于接口没有构造器，所以如果是实现接口，只能使用无参构造；如果是继承类，可以选择构造函数
- 类声明体：由于没有名字，所以不能自定义构造方法

匿名类和局部类具有相似的特性。

## 四、函数式接口(Functional Interface)
### 4.1 概述
函数式接口，又称SAM接口(Single Abstract Method interface),指的是只有一个抽象方法的接口(可以有多个default或static方法)。<br />
可以使用@FunctionalInterface注解进行编译时检查，判断是否满足函数式接口的定义。
```java
@FunctionalInterface
public interface ITest {
    int get();

    @Override
    String toString();

    @Override
    boolean equals(Object obj);
    
    @Override
    int hashCode();
} 
```
上述代码不会报错，java.lang.Object中的toString(),equals(),hashcode()三个方法可以定义在函数式接口中，而不影响其他抽象方法。java类默认继承java.lang.Object,这3个方法必然会被实现。上述代码如果删除get(),而只保留其他三个方法或其中一个，则编译错误"No target method found"。

### 4.2 标准函数式接口
java.util.function包下定义了常用的函数式接口。如Comsumer,Function,Predicate,Supplier,UnaryOperator等。
```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}

@FunctionalInterface
public interface Function<T, R> { 
    R apply(T t);
}

@FunctionalInterface
public interface Predicate<T> { 
    boolean test(T t);
}


@FunctionalInterface
public interface Supplier<T> {
    T get();
}

/**
 *  一元操作，返回值类型和参数类型相同
 */
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {

}
```

### 4.3 其他函数式接口
如 
- java.lang.Runnable
- java.util.Comparator

## 五、java 8 Lambda 表达式
### 5.1 Lambda 表达式
> 定义(wikipedia):Lambda expression in computer programming, also called **anonymous function** , a function (or a subroutine) defined, and possibly called, without being bound to an identifier. Anonymous functions are often arguments being passed to higher-order functions, or used for constructing the result of a higher-order function that needs to return a function.

### 5.2 java 8 lambda expression
##### 5.2.1 java lambda 表达式组成
```java
(Integer p1,Integer p2) -> {
            return (double)(p1 + p2);
        } 
```

```java 
(p1,p2) -> (double) (p1 + p2)
```

1. 参数列表: 可以省略参数类型；如果只有一个参数，可以省略括号
2. 箭头：->
3. 一个表达式或者语句块：
    - 如果只使用一个单独的表达式，返回值取决于表达式的值(void或其他类型的值)
    - 如果是一个语句块，则需要手写return语句返回值



## 、方法引用(Method Reference)

## 四、 闭包(Closure)
### 4.1 概念
> 定义(wikipedia)：In programming languages, a closure, also lexical closure or function closure, is a technique for implementing lexically scoped name binding in a language with first-class functions. Operationally, a closure is a record storing a function together with an environment. The environment is a mapping associating each free variable of the function (variables that are used locally, but defined in an enclosing scope) with the value or reference to which the name was bound when the closure was created.

闭包是由函数和其相关的引用环境组成的实体(环境是指关联函数的自由变量的映射)。<br />

### 4.2 java局部类、匿名类和闭包
java 中的局部类和匿名类具有闭包的语义，但不是真正的闭包。<br />

关于内部类，Core Java中有如下描述：
>Inner classes are translated into regular class files with $ (dollar signs) delimiting outer and inner class names and the virtual machine does not have any special knowledge about them.

三、匿名类(#三、匿名类(Anonymous Class))中的代码编译之后会生成两个class文件：
- AnonymousClass$1
- AnonymousClass
由于run()方法需要引用外部的局部变量b,所以依据闭包的定义，需要将变量b存储在一个可以让上述两个类同时访问的地方(如方法区)；但实际上局部变量存储在栈中，方法调用时压栈，方法返回时出栈。Java通过复制一个局部变量副本将变量b的值传递给内部类，为了保证一致性，要求内部类访问的外部局部变量必须是final。

