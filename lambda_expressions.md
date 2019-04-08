# Lambda Expression in java 8
## 一、Lambda表达式
> 定义(wikipedia):Lambda expression in computer programming, also called **anonymous function** , a function (or a subroutine) defined, and possibly called, without being bound to an identifier. Anonymous functions are often arguments being passed to higher-order functions, or used for constructing the result of a higher-order function that needs to return a function.

## 二、嵌套类(Nested Class)
### 2.1 定义
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

### 2.2 静态嵌套类(staic nested class)和内部类(inner class)
##### 2.2.1 静态嵌套类
```java
/**
 * 创建静态嵌套类对象。
 * 静态嵌套类不能直接访问外部类的非静态域和方法，其与外部类实例之间不存在直接联系，非
 * 静态成员的访问和非嵌套类之间的访问一致，需要通过外部类的实例化对象
 * 访问(可以访问private成员)。
 */
OuterClass.StaticInnerClass staticClassObject = new OuterClass.StaticInnerClass(); 
```

##### 2.2.2 内部类
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

##### 2.2.3 隐藏(Shadowing)
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

## 三、局部类(Local Class)
局部类是一种特殊的内部类。在语句块(如方法体,for循环体，if子句等)中定义的类称为局部类。



## 四、匿名类(Anonymous Class)
匿名类是一种特殊的内部类。


## 、接口

## 、java 8 Lambda 表达式

## 、方法引用(Method Reference)



