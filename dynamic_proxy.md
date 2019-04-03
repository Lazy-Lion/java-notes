# 动态代理
## 一、代理模式 (Proxy Pattern)

> 定义(from wikipedia): A proxy is a wrapper or agent object that is being called by the client to access the real serving object behind the scenes.

### 1.1 代理模式的作用
- 控制对象的访问，不允许直接访问目标对象；
- 扩展额外的功能时不需要直接修改目标类，而是通过添加代理的方式，如日志功能等;

### 1.2 代理模式UML类图
![UML](../master/images/proxy_uml_class.png)

- Subject: 代理类和目标类共同实现的接口
- Proxy: 代理类
    - 为了可以替代目标类，需要实现和目标类相同的接口。并且目标类可以使用的地方，代理类也要可以使用；
    - 包含可以访问目标类的引用；
- RealSubject: 需要被代理的目标类

## 二、jdk 动态代理 (java 8)
### 2.1 静态代理与动态代理
使用静态代理时，需要创建与目标类对应的代理类，代码量过大。同时，修改目标类后，代理类可能也要进行相应的修改。而动态代理不需要我们手动创建代理类，由jdk在运行时自动创建。
### 2.2 jdk 动态代理
##### 2.2.1 关键类和接口
**java.lang.reflect.Proxy**
```java
public class Proxy implements java.io.Serializable {
    
    ...

    /**
     * Returns an instance of a proxy class for the specified interfaces
     * that dispatches method invocations to the specified invocation
     * handler.
     */
  @CallerSensitive
  public static Object newProxyInstance(ClassLoader loader,
                                        Class<?>[] interfaces,
                                        InvocationHandler h)
          throws IllegalArgumentException{
            ...
          } 
}
```

- Proxy：代理类，jdk动态代理自动生成的代理类都是Proxy的子类；
- Proxy 提供静态方法newProxyInstance()用于创建代理类实例
    + 参数 loader: 载入代理类的类加载器
    + 参数 interfaces: 代理类需要实现的一组接口
    + 参数 h: InvocationHandler 实例
    + 方法返回类型为Object，实际上是Proxy的子类，并实现了interfaces中的接口

**java.lang.reflect.InvocationHandler**
```java
/**
  * Processes a method invocation on a proxy instance and returns
  * the result.  This method will be invoked on an invocation handler
  * when a method is invoked on a proxy instance that it is
  * associated with.
  */
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
} 
```

- 每个代理实例(proxy)都会关联一个InvocationHandler实例(h)，proxy调用方法被处理成调用p.invoke()
- invoke():
    + 参数 proxy: 代理类实例
    + 参数 method: 目标类指定方法的Method实例
    + 参数 args: 目标类指定方法的一组参数
    + h.invoke()方法返回类型为Object，假设其实际类型为R:
        * 如果被代理方法的返回类型是基本类型(boolean,char,int,...),R为其对应的包装类(不能是null)
        * 如果被代理方法的返回类型是引用类型(T)，R可以是T及其子类(可以是null)

##### 2.2.2 show me code 
```java
public interface Subject {
    int value(int value);  
    void key(Integer i);
    Number refer(int number);
}
```
```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class InvocationHandlerImp implements InvocationHandler {
    Object s;  // 被代理类实例

    public InvocationHandlerImp(Object s){
        this.s = s;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("调用方法前! ");
        // return null, value()报错：java.lang.NullPointerException,
        // key(),refer() 正常执行  

        // 对于refer(), return new Double(2.0)也可以正常执行, 因为Double是Number的子类

        // 对于key()方法, 可以return any Object类型 (or null)
        Object r = method.invoke(s, args);   // note: s 为被代理类实例
        return r;
    }
}
```
```java
public class RealSubject implements Subject{
    @Override
    public int value(int value) {
        return value;
    }

    @Override
    public void key(Integer i) {

    }

    @Override
    public Number refer(int number) {
        return new Integer(number);
    }
} 
```
```java 
import java.lang.reflect.Proxy;

public class Main {
    public static void main(String[] args){
        Subject p = (Subject) Proxy.newProxyInstance(Main.class.getClassLoader()
                , new Class[]{Subject.class}, new InvocationHandlerImp(new RealSubject()));
        System.out.println(p.refer(1));
    }
}
```

## 三、jdk 动态代理内部实现 (java 8)
### 3.1 Proxy类
```java
    // Proxy实现了序列化接口
    private static final long serialVersionUID = -2222568056686623797L; 

    // Proxy类允许使用InvocationHandler实例构造对象
    private static final Class<?>[] constructorParams =
        { InvocationHandler.class };
    
    private Proxy() { }  // Proxy禁用无参构造

    protected Proxy(InvocationHandler h) {  // Proxy只允许其子类构造实例
        Objects.requireNonNull(h);
        this.h = h;
    }

    // 使用缓存避免动态代理类重复生成，新的代理类由ProxyClassFactory工厂产生
    private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
      proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());

    // 与代理类绑定的InvocationHandler实例
    protected InvocationHandler h;
```

Proxy类向外提供了4个方法：
```java
// 返回指定动态代理类的InvocationHandler实例
public static InvocationHandler getInvocationHandler(Object proxy)
        throws IllegalArgumentException; 

// 返回动态代理类的类对象
public static Class<?> getProxyClass(ClassLoader loader,
                                     Class<?>... interfaces)
        throws IllegalArgumentException;

// 判断指定类对象是否是动态代理类
public static boolean isProxyClass(Class<?> cl);

// 创建动态代理类
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h)
        throws IllegalArgumentException;
```

### 3.2 创建动态代理类的类对象：
```java
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h)
    throws IllegalArgumentException
{
    Objects.requireNonNull(h);

    final Class<?>[] intfs = interfaces.clone();
    final SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
    }

     /*
      * Look up or generate the designated proxy class.
      */
    Class<?> cl = getProxyClass0(loader, intfs);

    /*
     * Invoke its constructor with the designated invocation handler.
     */
    try {
        if (sm != null) {
            checkNewProxyPermission(Reflection.getCallerClass(), cl);
        }

        final Constructor<?> cons = cl.getConstructor(constructorParams);
        final InvocationHandler ih = h;
        if (!Modifier.isPublic(cl.getModifiers())) {
            AccessController.doPrivileged(new PrivilegedAction<Void>() {
                public Void run() {
                    cons.setAccessible(true);
                    return null;
                }
            });
        }
        return cons.newInstance(new Object[]{h});  // 通过构造器创建代理对象
    } catch (IllegalAccessException|InstantiationException e) {
        throw new InternalError(e.toString(), e);
    } catch (InvocationTargetException e) {
        Throwable t = e.getCause();
        if (t instanceof RuntimeException) {
            throw (RuntimeException) t;
        } else {
            throw new InternalError(t.toString(), t);
        }
    } catch (NoSuchMethodException e) {
        throw new InternalError(e.toString(), e);
    }
} 
```

newProxyInstance()方法包括一些权限检查、代理类的类对象以及代理对象的生成，代理对象的生成是通过类对象的Constructor完成，所以关键点在 *Class<?> cl = getProxyClass0(loader, intfs);*。

```java
private static Class<?> getProxyClass0(ClassLoader loader,
                                       Class<?>... interfaces) {
    if (interfaces.length > 65535) {
        throw new IllegalArgumentException("interface limit exceeded");
    }

    return proxyClassCache.get(loader, interfaces);
} 
```

代理类的类对象从proxyClassCache中获取，从 *proxyClassCache = new WeakCache(new KeyFactory(), new ProxyClassFactory());* 可以看到，Cache中的值依赖于ProxyClassFactory。
```java 
private static final class ProxyClassFactory
    implements BiFunction<ClassLoader, Class<?>[], Class<?>>
{
    // prefix for all proxy class names
    private static final String proxyClassNamePrefix = "$Proxy";

    // next number to use for generation of unique proxy class names
    private static final AtomicLong nextUniqueNumber = new AtomicLong();

    @Override
    public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {

        Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);  // IdentityHashMap只依据引用判断key是否相等
        for (Class<?> intf : interfaces) {
            /*
             * Verify that the class loader resolves the name of this
             * interface to the same Class object.
             */
            Class<?> interfaceClass = null;
            try {
                interfaceClass = Class.forName(intf.getName(), false, loader);
            } catch (ClassNotFoundException e) {
            }
            if (interfaceClass != intf) {
                throw new IllegalArgumentException(
                    intf + " is not visible from class loader");
            }
            /*
             * Verify that the Class object actually represents an
             * interface.
             */
            if (!interfaceClass.isInterface()) {
                throw new IllegalArgumentException(
                    interfaceClass.getName() + " is not an interface");
            }
            /*
             * Verify that this interface is not a duplicate.
             */
            if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
                // interfaceSet.put()返回旧值，如果没有旧值则返回null
                throw new IllegalArgumentException(
                    "repeated interface: " + interfaceClass.getName());
            }
        }

        String proxyPkg = null;     // package to define proxy class in
        int accessFlags = Modifier.PUBLIC | Modifier.FINAL;

        /*
         * Record the package of a non-public proxy interface so that the
         * proxy class will be defined in the same package.  Verify that
         * all non-public proxy interfaces are in the same package.
         */
        for (Class<?> intf : interfaces) { 
        // 1.java 中接口访问修饰符只要两种:public,default(无修饰符，即包内可见);
        // 2.如果存在non-public接口，proxyPkg使用non-public接口所在包, 且代理类
        // 3.访问修饰符为defalut; 如果存在多个non-public接口且不在同一个包内，
        // 抛出异常
        // 4.代理类是final类
            int flags = intf.getModifiers();
            if (!Modifier.isPublic(flags)) {
                accessFlags = Modifier.FINAL;
                String name = intf.getName();
                int n = name.lastIndexOf('.');
                String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
                if (proxyPkg == null) {
                    proxyPkg = pkg;
                } else if (!pkg.equals(proxyPkg)) {
                    throw new IllegalArgumentException(
                        "non-public interfaces from different packages");
                }
            }
        }

        if (proxyPkg == null) {
            // if no non-public proxy interfaces, use com.sun.proxy package;
            proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
        }

        /*
         * Choose a name for the proxy class to generate.
         */
         long num = nextUniqueNumber.getAndIncrement();
         String proxyName = proxyPkg + proxyClassNamePrefix + num;

        /*
         * Generate the specified proxy class.
         */
        byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
            proxyName, interfaces, accessFlags);
        try {
            return defineClass0(loader, proxyName,
                                proxyClassFile, 0, proxyClassFile.length);
        } catch (ClassFormatError e) {
            /*
             * A ClassFormatError here means that (barring bugs in the
             * proxy class generation code) there was some other
             * invalid aspect of the arguments supplied to the proxy
             * class creation (such as virtual machine limitations
             * exceeded).
             */
            throw new IllegalArgumentException(e.toString());
        }
    }
}
```
代码分析：

- proxyClassNamePrefix和nextUniqueNumber一起定义了生成类对象的名称,如 "$Proxy1"
- 工厂类实现BiFunction<ClassLoader, Class<?>[], Class<?>>接口,cache实际上是通过该接口的apply()方法获取Class<?>
- BiFunction: function interface, BiFunction<T,U,R> 表示参数类型为T,U; 返回值类型为R
- ProxyGenerator.generateProxyClass()方法生成代理类的字节码文件
- defineClass0()是一个native方法，通过字节码文件生成代理类的类对象
- 其他参考代码注释

ProxyGenerator在 *sun.misc* 包下，jdk中没有相关源代码，参照 [OpenJDK](http://hg.openjdk.java.net/jdk8u/jdk8u/jdk/file/acab6dbdd0b5/src/share/classes/sun/misc/ProxyGenerator.java)

ProxyGenerator 代码分析：

1. 可以通过设置 *sun.misc.ProxyGenerator.saveGeneratedFiles* 属性，决定是否将字节码文件输出
2. 动态生成的代理类默认包含java.lang.Object的3个方法 (hashCode(),equlas(),toString())
3. 代理的接口中，当方法签名相同时(方法名，参数列表(类型、个数、顺序)相同)：
    - 如果有多个返回类型，返回类型不允许存在基本类型或void
    - 如果有多个返回类型，且返回类型之间存在继承关系，则指定该方法的返回类型为最底层子类型；不允许存在多个没有继承关系的返回类型
4. 添加构造方法，参数类型为InvocationHandler，通过调用父类(java.lang.reflect.Proxy)构造方法实现( **3.1** 可以看到，Proxy类提供了一个protected构造方法)
5. 添加Object中的3个方法以及各个接口的方法。 并添加对应各个方法的 **private static Method** 域，域名为对应的方法名(方法和域的最大数量限制是65535)
6. 添加静态块，用于初始化之前添加的Method域


设置 *System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles","true");*，将 **2.2.2** 中动态生成的代理类反编译得到如下代码：
```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

final class $Proxy0 extends Proxy implements Subject {
    private static Method m1;
    private static Method m4;
    private static Method m5;
    private static Method m2;
    private static Method m3;
    private static Method m0;

    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final void key(Integer var1) throws  {
        try {
            super.h.invoke(this, m4, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final Number refer(int var1) throws  {
        try {
            return (Number)super.h.invoke(this, m5, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int value(int var1) throws  {
        try {
            return (Integer)super.h.invoke(this, m3, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final int hashCode() throws  {
        try {
            return (Integer)super.h.invoke(this, m0, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m4 = Class.forName("dynamicproxy.Subject").getMethod("key", Class.forName("java.lang.Integer"));
            m5 = Class.forName("dynamicproxy.Subject").getMethod("refer", Integer.TYPE);
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m3 = Class.forName("dynamicproxy.Subject").getMethod("value", Integer.TYPE);
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}
```

## 小结
代理模式可以控制目标类的访问，易于功能的扩展。动态代理在静态代理的基础上，无需手动编写和维护代理类，大大减少了代码量。<br />
jdk动态代理自动生成的代理类都是java.lang.reflect.Proxy的子类，并且实现了被代理的接口。jdk动态代理要求被代理的目标类实现一个或多个接口(ProxyClassFactory获取代理类时会进行判断)，并且只能代理接口中的方法以及Object类中的3个方法。<br />
当需要代理没有实现接口的类时，CGLIB会是一个好的选择。[Create Proxies Dynamically Using CGLIB Library](http://jnb.ociweb.com/jnb/jnbNov2005.html)

        



