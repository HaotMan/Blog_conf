---
title: java反射机制
date: 2019/7/13 15:46:25
categories: java
tags: java
permalink: Java_reflection

---

#### 一、 什么是反射(Reflection)

反射是java的一种特性，运行中的程序可以用这个特性获取自身的一些信息，也可以用它来操作类或者对象的属性

官方文档中对反射的解释是：

<!--more--> 

> Reflection enables Java code to discover information about the fields, methods and constructors of loaded classes, and to use reflected fields, methods, and constructors to operate on their underlying counterparts, within security restrictions.
> The API accommodates applications that need access to either the public members of a target object (based on its runtime class) or the members declared by a given class. It also allows programs to suppress default reflective access control.

简单来说就是我们可以在运行时获得程序或程序集中每一个类型的成员和成员的信息。程序中一般的对象的类型都是在编译期就确定下来的，而 Java 反射机制可以动态地创建对象并调用其属性，这样的对象的类型在编译期是未知的。

当我们使用java IDE（例如eclipse、idea等）时，在类名后面输入“.”时，IDE会自动列出该类的所有属性用于参考，这就是运用了反射机制，IDE类属性提示并没有运行我们的程序，而是用反射将被提示的类动态的加载进了IDE本身的运行环境中去并获取它的属性。

通常我们在代码中使用new objA()创建类objA的实例后，在编译时就已经将该对象加载进JVM中，以供后面的程序调用，但是如果程序运行过程中用户突然指定需要调用某个类，而这个类之前却没有被加载到JVM中，这时候不可能让程序停下来 让程序员中途写段代码new一下，这时候就是反射机制发挥作用的时候，可以在程序运行过程中动态地把类加载进JVM中。
例如项目底层有时是用mysql，有时用oracle，需要动态地根据实际情况加载驱动类，这时com.java.dbtest.myqlConnection，com.java.dbtest.oracleConnection这两个类我们都要使用，但是我们又不想提前将这两个类都加载进JVM中去，这时候就可以用反射机制动态的加载它们：

```java
Class tc = Class.forName("com.java.dbtest.TestConnection");
//驱动的类名可以由用户给出，也可以从配置文件（如xml文件等）中读取
```

另一个反射的例子就是web项目中的xml配置文件

```xml
<servlet>
      <servlet-name>webSpring</servlet-name>
      <servlet-class>
         org.springframework.web.servlet.DispatcherServlet
      </servlet-class>
      <load-on-startup>1</load-on-startup>
   </servlet>

   <servlet-mapping>
      <servlet-name>webSpring</servlet-name>
      <url-pattern>/</url-pattern>
   </servlet-mapping>
```
其中\<servlet>标签规定了一个叫webSpring的sevlet，其请求处理类为org.springframework.web.servlet.DispatcherServlet，而\<servlet-mapping>标签告诉程序匹配到“/”的url交给名叫webSpring这个servlet来处理。
当web核心程序运行时，根据收到的请求，就会去找xml文件配置中的servlet来处理该请求，而此时就需要动态地将xml文件中规定的servlet请求处理类加载进JVM中去

#### 二、反射的基本运用
##### 1.获取Class对象
- 使用 Class 类的 **forName** 静态方法:

  ```java
  public static Class<?> forName(String className){...}
  
  // 在数据库中加载驱动类
  Class db_class = Class.forName(driver);
  ```
- 直接获取一个对象的class

  ```java
  Class<?> klass = int.class;
  Class<?> classInt = Integer.TYPE;
  ```
- 调用对象的**getClass**方法
  ```java
  Class<?> obj_class = obj.getClass();
  ```
##### 2.判断是否是某个类的实例
一般地，我们用 instanceof 关键字来判断是否为某个类的实例。同时我们也可以借助反射中 Class 对象的 isInstance() 方法来判断是否为某个类的实例，它是一个 native 方法：

```java
public native boolean isInstance(Object obj);
```

- 示例
  ```java
  Person pobj = new Person()
  Class<?> pclass = pobj.getClass();
  pobj.instanceof(Person)
  pclass.isInstance(pobj)
  ```

##### 3.创建实例
- 使用Class对象的newInstance()方法来创建对应类的实例

  ```java
  Class<?> s_class = String.class;
  Object str = s_class.newInstance();
  ```

- 先通过Class对象获取指定的Constructor对象，再调用Constructor对象的newInstance()方法来创建实例。这种方法可以用指定的构造器构造类的实例。

  ```java
  //获取String所对应的Class对象
  Class<?> s_class = String.class;
  //获取String类带一个String参数的构造器
  Constructor constructor = s_class.getConstructor(String.class);
  //根据构造器创建实例
  Object obj = constructor.newInstance("23333");
  System.out.println(obj);
  ```

##### 4.获取方法
- getDeclaredMethods 方法返回类或接口声明的所有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法
  ```java
  public Method[] getDeclaredMethods() throws SecurityException
  ```
- getMethods 方法返回某个类的所有公用（public）方法，包括其继承类的公用方法。

  ```java
  public Method[] getMethods() throws SecurityException
  ```

- getMethod 方法返回一个特定的方法，其中第一个参数为方法名称，后面的参数为方法的参数对应Class的对象。

  ```java
  public Method getMethod(String name, Class<?>... parameterTypes)
  ```

- 示例
  ```java
  public class test1 {
  	public static void test() {
          Class<?> p = Person.class;
          
          //获取Person类中的sleep方法
          Method mtd = p.getMethod("sleep", Time.class, Bed.class, String.class);
      }
  	}
  class Person{
  	...
  	public void sleep(Time tim, Bed bd, String context){...}
  }
  
  
  ```

##### 5.获取类的成员变量信息
- getFiled：访问公有的成员变量
- getDeclaredField：所有已声明的成员变量，但不能得到其父类的成员变量

    getFileds 和 getDeclaredFields 用法同Method。

##### 6.调用方法
使用反射机制从类中获取了一个方法后，我们就可以用 invoke() 方法来调用这个方法。invoke 方法的原型为:

```java
public Object invoke(Object obj, Object... args)
        throws IllegalAccessException, IllegalArgumentException,
           InvocationTargetException
```

示例

```java
public class test1 {
		public static void test() {
	        Class<?> p = Person.class;
	        Object pObj = p.newInstance();
	        
	        //获取Person类中的sleep方法
	        Method mtd = p.getMethod("sleep", Time.class, Bed.class, String.class);
			
			//执行获取到的方法
	        mtd.invoke(pObj, time_1, bed_1, "good luck");
	        
	    }
  	}
  class Person{
		...
		public void sleep(Time tim, Bed bd, String context){...}
	}
	
```

#### 三、 使用反射的注意事项

> - 反射会额外消耗一定的系统资源，如果不需要动态地创建一个对象，那么就不需要用反射。
> - 反射调用方法时可以忽略权限检查，可以利用这个特性来获取到一些类的私有属性和方法，但也可能破坏封装性而导致安全问题。