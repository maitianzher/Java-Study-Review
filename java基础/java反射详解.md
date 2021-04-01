[TOC]

# java反射详解

## 一、反射的定义（什么是反射）

**定义及其实现:** Java反射机制是指在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。用一句话总结就是反射可以实现在运行时可以知道任意一个类的属性和方法。程序中一般的对象的类型都是在编译期就确定下来的，而 Java 反射机制可以动态地创建对象并调用其属性，这样的对象的类型在编译期是未知的。所以我们可以通过反射机制直接创建对象，即使这个对象的类型在编译期是未知的。反射的核心是 JVM 在运行时才动态加载类或调用方法/访问属性，它不需要事先（写代码的时候或编译期）知道运行对象是谁。

**java反射提供主要功能：**

1. **在运行时**判断任意一个对象所属的类
2. **在运行时**构造任意一个类的对象
3. **在运行时**判断任意一个成员变脸和方法（private方法也可以）
4. **在运行时**调用任意一个对象方法

## 二、反射的主要途径 ##

**反射最重要的用途用于开发各种通用框架**。很多框架（例如Spring）都是通过配置化（XML配置文件到Bean），为了保障框架的通用性，他们可以根据配置文件加载不同的对象和类，调用不同的方法，这时需要用到反射，运行时动态加载需要的对象。对与框架开发人员来说，反射虽小但作用非常大，它是各种容器实现的核心。而对于一般的开发者来说，不深入框架开发则用反射用的就会少一点，不过了解一下框架的底层机制有助于丰富自己的编程思想，也是很有益的。

## 三、反射的基本运用

上面我们提到了反射可以用于判断任意对象所属的类，获得 Class 对象，构造任意一个对象以及调用一个对象。这里我们介绍一下基本反射功能的使用和实现(反射相关的类一般都在 java.lang.relfect 包里)。

**1.获得Class对象**

三种方法获得：

1. 使用Class类的forName静态方法（实用类的名字获取）：

   ```java
   public static Class<?> forName(String className){}
   
   例如JDBC中加载数据库驱动
   Class.forname(driver);
   ```

2. 直接获取一个对象的class

    ```java
   Class<?> classes=int.class;
   Class<?> classInt=Integer.TYPE
   ```

3. 调用对象的`gelClass()`方法

    ```java
   StringBuilder str = new StringBuilder("123");
   Class<?> klass = str.getClass();
   ```

**2.判断是否为某个类的实例**

一般地，我们用 `instanceof` 关键字来判断是否为某个类的实例。同时我们也可以借助反射中 Class 对象的 `isInstance()` 方法来判断是否为某个类的实例，它是一个 native 方法：

```java
public native boolean isInstance(Object obj){}
```

**3.创建实例**

通过反射创建实例，通常有两种方法

> (1)使用Class对象的newInstance()方法创建Class对象的对应类的实例

```java
Class<?> c=String.getClass();
Object obj=c.newInstance();
```

> (2) 通过Class对象获取指定的Constructor对象，在调用Constructor对象的newInstance()方法来创建实例。该方法可以用指定的构造器构造类实例。

```java
//获取String对应的Class对象
Class c=String.class;
//获取String类带参数的构造器（参数是对象字节码）
Constructor constructor=c.getConstructor(String.class);
//根据构造器创建实例
Object obj=constructor.newInstance("2333");
sout(obj);
```

**4.获取方法**

获取某个Class对象的方法集合，主要有以下几个方法：

- `getDeclaredMethods` 方法返回类或接口声明的所有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法。

  ```java
  public Method[] getDeclaredMethods() throws SecurityException
  ```

- `getMethods` 方法返回某个类的所有公用（public）方法，包括其继承类的公用方法。

  ```java
  public Method[] getMethods() throws SecurityException
  ```

- `getMethod` 方法返回一个特定的方法，其中第一个参数为方法名称，后面的参数为方法的参数对应Class的对象。

  ```java
  public Method getMethod() throws SecurityExecption
  ```

  例子来理解这三个方法：

  ```java
  package org.ScZyhSoft.common;
  import java.lang.reflect.InvocationTargetException;
  import java.lang.reflect.Method;
  public class test1 {
  	public static void test() throws IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
  	        Class<?> c = methodClass.class;
  	        Object object = c.newInstance();
  	        Method[] methods = c.getMethods();
  	        Method[] declaredMethods = c.getDeclaredMethods();
  	        //获取methodClass类的add方法
  	        Method method = c.getMethod("add", int.class, int.class);
  	        //getMethods()方法获取的所有方法
  	        System.out.println("getMethods获取的方法：");
  	        for(Method m:methods)
  	            System.out.println(m);
  	        //getDeclaredMethods()方法获取的所有方法
  	        System.out.println("getDeclaredMethods获取的方法：");
  	        for(Method m:declaredMethods)
  	            System.out.println(m);
  	    }
      }
  class methodClass {
      public final int fuck = 3;
      public int add(int a,int b) {
          return a+b;
      }
      public int sub(int a,int b) {
          return a+b;
      }
  }
  ```

  输出结果：

  ```java
  getMethods获取的方法：
  public int org.ScZyhSoft.common.methodClass.add(int,int)
  public int org.ScZyhSoft.common.methodClass.sub(int,int)
  public final void java.lang.Object.wait() throws java.lang.InterruptedException
  public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
  public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
  public boolean java.lang.Object.equals(java.lang.Object)
  public java.lang.String java.lang.Object.toString()
  public native int java.lang.Object.hashCode()
  public final native java.lang.Class java.lang.Object.getClass()
  public final native void java.lang.Object.notify()
  public final native void java.lang.Object.notifyAll()
  getDeclaredMethods获取的方法：
  public int org.ScZyhSoft.common.methodClass.add(int,int)
  ```

  可以看到，通过 `getMethods()` 获取的方法可以获取到父类的方法,比如 java.lang.Object 下定义的各个方法

**5.获取构造器**

获取类构造器的用法与上述获取方法的用法类似。主要是通过Class类的getConstructor方法得到Constructor类的一个实例，而Constructor类有一个newInstance方法可以创建一个对象实例。此方法可以根据传入的参数来调用对应的Constructor创建对象实例。

```java
public T newInstance(Object ... initargs)
```

**6.获取类的成员变量信息**

主要是这几个方法，在此不再赘述：

- `getFiled`：访问公有的成员变量
- `getDeclaredField`：所有已声明的成员变量，但不能得到其父类的成员变量

`getFileds` 和 `getDeclaredFields` 方法用法同上（参照 Method）。

**7.调用方法**

如4中所说的，获得方法之后，可以用`invloke()`方法来调用这个方法，方法原型：

```java
public Object invoke(Object obj, Object... args)
        throws IllegalAccessException, IllegalArgumentException,
           InvocationTargetException
```

```java
//例子
public class test1 {
    public static void main(String[] args) throws IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
        Class<?> klass = methodClass.class;
        //创建methodClass的实例
        Object obj = klass.newInstance();
        //获取methodClass类的add方法
        Method method =klass.getMethod("add",int.class,int.class);
        //调用method对应的方法 => add(1,4)
        Object result = method.invoke(obj,1,4);
        System.out.println(result);
    }
}

class methodClass {
    public final int fuck = 3;
    public int add(int a,int b) {
        return a+b;
    }
    public int sub(int a,int b) {
        return a+b;
    }
}

```

**8.利用反射创建数据**

数组在Java里是比较特殊的一种类型，它可以赋值给一个Object Reference。下面我们看一看利用反射创建数组的例子：

```java
public static void testArray() throws ClassNotFoundException {
        Class<?> cls = Class.forName("java.lang.String");
        Object array = Array.newInstance(cls,25);
        //往数组里添加内容
        Array.set(array,0,"hello");
        Array.set(array,1,"Java");
        Array.set(array,2,"fuck");
        Array.set(array,3,"Scala");
        Array.set(array,4,"Clojure");
        //获取某一项的内容
        System.out.println(Array.get(array,3));
    }
```

其中的Array类为java.lang.reflect.Array类。我们通过Array.newInstance()创建数组对象，它的原型是:

```java
public static Object newInstance(Class<?> componentType, int length)
        throws NegativeArraySizeException {
        return newArray(componentType, length);
    }
```

而 `newArray` 方法是一个 native 方法，它在 HotSpot JVM 里的具体实现我们后边再研究，这里先把源码贴出来：

```java
private static native Object newArray(Class<?> componentType, int length)
        throws NegativeArraySizeException;
```

源码目录：`openjdk\hotspot\src\share\vm\runtime\reflection.cpp`

```c++
arrayOop Reflection::reflect_new_array(oop element_mirror, jint length, TRAPS) {
  if (element_mirror == NULL) {
    THROW_0(vmSymbols::java_lang_NullPointerException());
  }
  if (length < 0) {
    THROW_0(vmSymbols::java_lang_NegativeArraySizeException());
  }
  if (java_lang_Class::is_primitive(element_mirror)) {
    Klass* tak = basic_type_mirror_to_arrayklass(element_mirror, CHECK_NULL);
    return TypeArrayKlass::cast(tak)->allocate(length, THREAD);
  } else {
    Klass* k = java_lang_Class::as_Klass(element_mirror);
    if (k->oop_is_array() && ArrayKlass::cast(k)->dimension() >= MAX_DIM) {
      THROW_0(vmSymbols::java_lang_IllegalArgumentException());
    }
    return oopFactory::new_objArray(k, length, THREAD);
  }
}
```

另外，Array 类的 `set` 和 `get` 方法都为 native 方法，在 HotSpot JVM 里分别对应 `Reflection::array_set` 和 `Reflection::array_get` 方法，这里就不详细解析了。

## 四、反射的注意事项（优缺点）

**优点：**

1. 增加程序的灵活性，避免将代码写死
2. 代码简洁，提高代码复用率，外部调用方便
3. 对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法     

**缺点**：

1. 性能问题。使用反射基本上是一种解释操作，用于字段和方法接入时要远慢于直接代码。因此Java反射机制主要应用在对灵活性和扩展性要求很高的系统框架上,普通程序不建议使用。
2. 反射包括了一些动态类型，所以JVM无法对这些代码进行优化。因此，反射操作的效率要比那些非反射操作低得多。我们应该避免在经常被 执行的代码或对性能要求很高的程序中使用反射。
3. 使用反射会模糊程序内部逻辑
4. 安全限制，内部暴漏。反射调用方法时可以忽略权限检查，因此可能会破坏封装性而导致安全问题



blog：<https://blog.csdn.net/javazejian/article/details/70768369>

​			<https://blog.csdn.net/sinat_38259539/article/details/71799078>

​			<https://blog.csdn.net/javazejian/article/details/70768369>

​	

​	



























































