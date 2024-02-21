> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/wpc2018/article/details/115252873)

#### 文章目录

*   [Java 进阶——注解和反射](#Java_8)
*   *   [1、注解](#1_10)
    *   *   [1.1、什么是注解 (Annotation)](#11Annotation_12)
        *   [1.2、内置注解](#12_30)
        *   [1.3、自定义注解和元注解 (meta-annotation)](#13metaannotation_94)
    *   [2、反射](#2_218)
    *   *   [1、反射和反射机制](#1_220)
        *   [2、CLass 类](#2CLass_235)
        *   [3、哪些类型可以有 Class 对象](#3Class_285)
        *   [4、类加载理解](#4_337)
        *   [5、类加载器](#5_362)
        *   [6、通过反射动态的创建对象](#6_416)
        *   [7、反射操作注解](#7_439)

Java 进阶——注解和反射
--------------

### 1、注解

#### 1.1、什么是注解 (Annotation)

从 JDK5 开始, Java 增加对元数据的支持，也就是注解，注解与注释是有一定区别的，可以把注解理解为代码里的特殊标记，

注解的作用:

*   不是程序本身，可以对程序作出解释
    
*   可以在程序编译，类加载，运行时被读取，并执行相应的处理。
    

注解的格式：

*   注解是以 "@注释名" 在代码中存在的，还可以添加一些参数值，例如：@SuppersWarnings(valus=“unchecked”)

注解在哪里使用：

*   可以附加在 package,class,method,filed 等上面，相当于给他们添加了额外的辅助信息，我们可以通过反射机制编程实现对这些元数据的访问。

#### 1.2、内置注解

*   **1. 限定父类重写方法:@Override**
    
    当子[类重写](https://so.csdn.net/so/search?q=%E7%B1%BB%E9%87%8D%E5%86%99&spm=1001.2101.3001.7020)父类方法时，子类可以加上这个注解，那这有什么什么用？这可以确保子类确实重写了父类的方法，避免出现低级错误
    
    ```
    @Override //该注解标识的类，代表的覆盖的是其父类的方法
    public boolean equals(Object obj) {
    	return super.equals(obj);
    ```
    
*   **2. 标示已过时:@Deprecated**
    
    这个注解用于表示某个程序元素类，方法等已过时，当其他程序使用已过时的类，方法时[编译器](https://so.csdn.net/so/search?q=%E7%BC%96%E8%AF%91%E5%99%A8&spm=1001.2101.3001.7020)会给出警告（删除线，这个见了不少了吧）。
    
    ```
    @Deprecated //该注解标识的类，代表该类已经过时
    public static void sayHello() {
    	System.out.println("hello world");
    }
    ```
    
*   **3. 抑制编译器警告:@SuppressWarnings**
    
    被该注解修饰的元素以及该元素的所有子元素取消显示编译器警告，例如修饰一个类，那他的字段，方法都是显示警告
    
    ```
    @SuppressWarnings("deprecation")
    public static void main(String[] args) {
    	//这个类是过时的，使用@SuppressWarnings来压制警告
    	System.runFinalizersOnExit(true);
    }
    ```
    
*   **4.“堆污染” 警告与 @SafeVarargs**
    
    想理解这个就要明白什么是堆污染，堆污染是什么？
    
    其实很好理解，就是把不带泛型的对象赋给一个带泛型的对象，为什么不行？很简单，因为不带泛型的话，默认会给泛型设定为 object，意思就是什么类型都可以往里面塞，那你一个不带泛型的怎么可能给一个带泛型塞呢。
    
    例如运行如下代码：
    
    List list = new ArrayList(); list.add(20); List ls = list; System.out.println(ls.get(0)); 则会抛出堆污染异常 Exception in thread “main” java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String at Test.Test1.main(Test1.java:29)
    
    注意：可变参数更容易引发堆污染异常，因为 java 不允许创建泛型数组，可变参数恰恰是数组。  
    　　抑制这个警告的方法有三个:
    
    1.@SafeVarargs 修饰引发该警告的方法或构造器
    
    2. 使用 @suppressWarnings(“unchecked”)
    
    3. 编译时使用 - Xlint:varargs
    
*   **5. 函数式接口与 @Functionallnterface**
    
    什么是函数式？如果接口中只有一个抽象方法（可以包含多个默认方法或多个 static 方法）
    
    接口体内只能声明常量字段和抽象方法，并且被隐式声明为 public，static，final。
    
    接口里面不能有私有的方法或变量。
    
    这个注解有什么用？这个注解保证这个接口只有一个抽象方法，注意这个只能修饰接口
    

#### 1.3、自定义注解和元注解 (meta-annotation)

**定义注解**：

*   第一步，用`@interface`定义注解：
    
    ```
    public @interface Report {
    }
    ```
    
*   第二步，添加参数、默认值, 如果没有默认值，就必须给参数赋值：
    
    ```
    public @interface Report {
            //参数类型 +参数名()
        int type() default 0;
        String level() default "info";
        String value() default "万里";
    }
    ```
    

​ 把最常用的参数定义为`value()`，推荐所有参数都尽量设置默认值。

*   第三步，用元注解配置注解：
    
    ```
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface Report {
        int type() default 0;
        String level() default "info";
        String value() default "";
    }
    ```
    

​ 其中，必须设置`@Target`和`@Retention`，`@Retention`一般设置为`RUNTIME`，因为我

​ 们自定义的注解通常要求在运行期读取。一般情况下，不必写`@Inherited`和`@Repeatable`。

在使用的时候，如果一个类用到了我们定义的注解，则它的子类也定义该注解

```
@Report(type=1)
public class Person {
}
```

```
public class Student extends Person {
}
```

**元注解**：

*   元注解的作用就是负责注解其他注解，Java 定义了 4 个标准的元注解类型，他们被用来提供对其他注解类型作说明
    
*   这些类型和他们所支持的类在 java.lang.annotation 包中可以找到
    
    1、@Target：用于描述注解的使用范围
    
    *   类或接口：`ElementType.TYPE`；
    *   字段：`ElementType.FIELD`；
    *   方法：`ElementType.METHOD`；
    *   构造方法：`ElementType.CONSTRUCTOR`；
    *   方法参数：`ElementType.PARAMETER`。
    
    ```
    public class Test{
        @MyAnnotation
        public void test(){       
        }
    }
    //定义一个注解MyAnnotation,再给注解加上一个元注解
    @Target(value = ElementType.METHOD)//表示该注解只能用在方法上
    //可以设置多个范围：@Target(value ={ElementType.METHOD,ElementType.type})
    @interface MyAnnotation{
    }
    ```
    
    2、@Retention：表示需要在什么级别保存该注释信息，用于描述注解的生命周期
    
    *   仅编译期：`RetentionPolicy.SOURCE`；
        
    *   仅 class 文件：`RetentionPolicy.CLASS`；
        
    *   运行期：`RetentionPolicy.RUNTIME`。
        
        范围：RUNTIME>CLASS>SOURCE
        
    
    如果 @Retention 不存在，则该 Annotation 默认为 CLASS。因为通常我们自定义的 Annotation 都是 RUNTIME，所以，务必要加上`@Retention(RetentionPolicy.RUNTIME)`这个元注解：
    
    ```
    @Retention(RetentionPolicy.RUNTIME)
    @interface MyAnnotation{
    }
    ```
    
    3、@Document：说明该注将被包含在 javadoc 中
    
    4、@Iherited：使用 @Inherited 定义子类是否可继承父类定义 Annotation。
    
    ​ @Inherited 仅针对 `@Target(ElementType.TYPE)`类型的 annotation 有效，并且仅针对
    
    ​ class 的继承，对 interface 的继承无效：
    
    ```
    @Inherited
    @Target(ElementType.TYPE)
    public @interface Report {
        int type() default 0;
        String level() default "info";
        String value() default "";
    }
    ```
    

### 2、反射

#### 1、反射和反射机制

**反射 (Reflection)：**

Java 的反射是指程序在运行期可以拿到一个对象的所有信息。

**反射的优点和缺点：**

*   优点：可以实现动态创建对象和编译，灵活性大
*   缺点：对性能有影响，反射操作总是慢于直接执行相同操作

**反射机制：**

Java 的反射机制是指在程序的运行状态中，** 可以构造任意一个类的对象，可以了解任意一个对象所属的类，可以了解任意一个类的成员变量和方法，可以调用，操作任意一个对象的属性和方法。** 这种动态获取程序信息以及动态调用对象的功能称为 Java 语言的反射机制。反射被视为动态语言（在程序运行的时候可以改变其结构）的关键。

#### 2、CLass 类

**java.lang.reflect.Class 类，实现反射的核心类**

*   Class 类只能由系统建立对象
*   一个加载的类在内存 (JVM) 中只有一个 Class 对象
*   一个类被加载后，类的整个结构都会被封装在 Class 对象中
*   每个类的实例都会记得自己是由哪个 Class 实例所生成
*   Class 类是反射的根源，针对任何你想动态加载，运行的类，唯有获得相应的 Class 对象

> 其它 API

java.lang.reflect.Method：代表类的方法  
java.lang.reflect.Field：代表类的属性  
java.lang.reflect.Constructor：代表类的构造器

**获得 Class 类实例的五种方式：**

```
//方式一：调用Class类的静态方法 forName(String className)
Class c1 = Class.forName("com.cheng.reflection.User");

//方式二 已知某个类的实例，调用该实例的getClass()方法，getClass是Object类中的方法、因为所有类都继承Object类。
Class c2 = user.getClass();

//方式三 已知具体类，通过类的class属性获取，该方法最安全可靠，程序性能最高
Class c3 = User.class;

//方式四：通过基本内置类型的包装类的TYPE属性获得CLass实例
//以int的包装类Intege类为例   源码：public static final Class<Integer>  TYPE = (Class<Integer>) Class.getPrimitiveClass("int");
Class<Integer> c4 = Integer.TYPE;

//方式五：通过当前子类的Class对象获得父类的Class对象
Class c5 = c1.getSuperclass();//c1为子类的CLass对象
```

**Class 类的常用方法：**

<table><thead><tr><th>方法名</th><th>功能说明</th></tr></thead><tbody><tr><td>static ClssforName(String name)</td><td>返回指定类名 namedeClass 对象</td></tr><tr><td>Obiect newInstance()</td><td>调用无参构造函数，返回 Class 对象的一个实例</td></tr><tr><td>getName()</td><td>返回此 Class 对象所表示的实体（类，接口，数组类或 void）的名称</td></tr><tr><td>Class getSuperclass()</td><td>返回当前 Class 对象的父类的 Class 对象</td></tr><tr><td>Class[] getinterfaces()</td><td>返回当前 Class 对象的接口</td></tr><tr><td>ClassLoader getClassLoader()</td><td>返回该类的类加载器</td></tr><tr><td>Method getMethod(name,String .class)</td><td>返回对象 Method 一个数组，此对象的形参类型为 paramType</td></tr><tr><td>Field getDeclaredFields()</td><td>返回对象的 Field(属性) 一个数组</td></tr><tr><td>Constructor[] getConstructors()</td><td>返回一个包含某些 Constructor（构造器）对象的数组</td></tr></tbody></table>

#### 3、哪些类型可以有 Class 对象

*   class：外部类，成员（成员内部类，静态内部类），局部内部类，匿名内部类。
    
    ```
    Class c1 = Object.class;
    ```
    
*   interface：接口
    
    ```
    Class c2 = Comparable.class;
    ```
    
*   []：数组
    
    ```
    Class c3 = String[].class;
    Class c4 = int[][].class;
    ```
    
*   enum：枚举
    
    ```
    Class c5 = ElementType.class;
    ```
    
*   annotation：注解 @interface
    
    ```
    Class c6 = Override.class;
    ```
    
*   primitive type：基本数据类型
    
    ```
    Class c7 = Interger.class;
    ```
    
*   void
    
    ```
    Class c8 = void.class;
    Class c9 = Class.class;
    ```
    

#### 4、类加载理解

*   加载：将 class 字节码文件内容加载到内存中，并将这些数据装换成方法区的运行时数据结构，然后生成一个代表这个类的 java.lang.Class 对象
*   链接：将 Java 类的二进制代码合并到 JVM 的运行状态之中的过程
    *   验证：确保加载的类信息符合 JVM 规范，没有安全方面的问题
    *   准备：正式为类变量（static）分配内存并设置类变量默认初始值的阶段，这些内存都将在方法去中进行分配
    *   解析：虚拟机常量池内的符号引用（常量名）替换为直接引用（地址）的过程
*   初始化：
    *   执行类构造器`<clinit>()`方法的过程。类构造器`<clinit>()`方法是由编译期自动收集类中所有类变量的赋值动作和静态代码块中的语句合并产生的。（类构造器是构造类信息的，不是构造该类对象的构造器）。
    *   当初始化一个类的时候，如果发现其父类还没有进行初始化，则需要先触发其父类的初始化。
    *   虚拟机会保证一个类的在多线程环境中被正确加锁和同步。

什么时候会发生类初始化：

*   类的主动引用（一定会发生类初始化）
    *   当虚拟机启动，先初始化 mian 方法所在的类
    *   new 一个类的对象
    *   调用类的静态成员（处理 final 常量）和静态方法
    *   使用 java.lang.reflect 包的方法对类进行反射调用
    *   当初始化一个类的时候，如果发现其父类还没有进行初始化，则先初始化其父类
*   类的被动引用（不会发生类初始化）
    *   当访问一个静态域时，只有正真声明这个域的类才会被初始化。如：当通过子类引用父类的静态变量，不会导致子类初始化
    *   通过数组定义类引用，不会发生类初始化
    *   引用常量不会触发此类的初始化（常量在链接阶段就存入调用类的常量池中了）

#### 5、[类加载器](https://so.csdn.net/so/search?q=%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8&spm=1001.2101.3001.7020)

**类加载器**：完成类的加载

**类加载器的作用**：将 class 字节码文件内容加载到内存中，并将这些数据装换成方法区的运行时 数据结构，然后生 成一个代表这个类的 java.lang.Class 对象，作为方法区中类数据的访问入口

**JVM 三种预定义类型类加载器**，当 JVM 启动的时候，Java 开始使用如下三种类型的类加载器：

*   `根/启动（Bootstrap）类加载器`：根类加载器是用本地代码实现的类加载器，它负责将 JAVA_HOME/lib 下面的核心类库或 - Xbootclasspath 选项指定的 jar 包等虚拟机识别的类库加载到内存中。由于根类加载器涉及到虚拟机本地实现细节，开发者无法直接获取到根类加载器的引用。
    
*   `扩展（Extension）类加载器`：扩展类加载器是由 Sun 的 ExtClassLoader（sun.misc.Launcher$ExtClassLoader）实现的，它负责将 JAVA_HOME /lib/ext 或者由系统变量 - Djava.ext.dir 指定位置中的类库加载到内存中。开发者可以直接使用标准扩展类加载器。
    
*   `系统（System）类加载器`：系统类加载器是由 Sun 的 AppClassLoader（sun.misc.Launcher$AppClassLoader）实现的，它负责将用户类路径 (java -classpath 或 - Djava.class.path 变量所指的目录，即当前类所在路径及其引用的第三方类库的路径，如第四节中的问题 6 所述) 下的类库加载到内存中。开发者可以直接使用系统类加载器。
    

**类加载三种机制：**

*   全盘负责机制：就是当一个类加载器负责加载某个 Class 时，该 Class 所依赖和引用其他 Class 也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入。
*   双亲委派机制：所谓的双亲委派，则是先让父类加载器试图加载该 Class，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类。**通俗的讲，就是某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父加载器，依次递归，如果父加载器可以完成类加载任务，就成功返回；只有父加载器无法完成此加载任务时，才自己去加载。**
*   缓存机制机制：缓存机制将会保证所有加载过的 Class 都会被缓存，当程序中需要使用某个 Class 时，类加载器先从缓存区中搜寻该 Class，只有当缓存区中不存在该 Class 对象时，系统才会读取该类对应的二进制数据，并将其转换成 Class 对象，存入缓冲区中。这就是为很么修改了 Class 后，必须重新启动 JVM，程序所做的修改才会生效的原因。

**获取三种预定义类型类加载器:**

```
//获取系统类加载器
       ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
       System.out.println(systemClassLoader);//sun.misc.Launcher$AppClassLoader@18b4aac2

       //获取系统类加载器的父类加载器-->扩展类加载器
       ClassLoader parent = systemClassLoader.getParent();
       System.out.println(parent);//sun.misc.Launcher$AppClassLoader@18b4aac2

       //获取扩展类加载器的父类加载器-->根类加载器
       ClassLoader parent1 = parent.getParent();
       System.out.println(parent1);//null
```

**类加载的简单使用**

```
//测试当前类是由哪个类加载器加载的
        ClassLoader classLoader = Class.forName("com.cheng.annotation.Test01").getClassLoader();                     System.out.println(classLoader);
        //sun.misc.Launcher$AppClassLoader@18b4aac2 统类加载器

        //测试JDK内置类是由哪个类加载器加载的
        ClassLoader classLoader1 = Class.forName("java.lang.Object").getClassLoader();
        System.out.println(classLoader1);//null 根类加载器

        //获得系统类加载器可以加载的路径
        String property = System.getProperty("java.class.path");
        System.out.println(property);
```

#### 6、通过反射动态的创建对象

**通过构造器创建对象**

```
//通过构造器创建对象
Class c1 = class.forName("com.cheng.User");
Constructor constructor = c1.getDeclaredConstructor(String.class,int.class,int.class);
User user = (User)constructor.newInstance("万里",1,18);
System.out.println(user);//USer{name='万里',id=1,age=18}
```

**调用指定方法**

```
Class c1 = class.forName("com.cheng.User");
User user = (USer)c1.newInstance();
Method setName = c1.getDeclaredMethod("setName",String.class);
//invoke激活
setName.invoke(user,"万里");
System.out.println(user，getName());//万里
```

#### 7、反射操作注解

```
package com.cheng.reflection;
import java.lang.annotation.*;
import java.lang.reflect.Field;

//练习反射操作注解
public class Test {
    
    public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException {
        Class c1 = Class.forName("com.cheng.reflection.student2");

        //1.通过反射获得注解
        Annotation[] annotations = c1.getAnnotations();
        for (Annotation annotation : annotations) {
            System.out.println(annotation);
        }

        //2.获得注解的value值
        Cheng cheng = (cheng) c1.getAnnotation(cheng.class);
        System.out.println(cheng.value());

        //3.获得类指定字段的注解
        Field f = c1.getDeclaredField("id");
        Fieldhaha annotation = f.getAnnotation(Fieldcheng.class);
        System.out.println(annotation.columnName());
        System.out.println(annotation.length());
        System.out.println(annotation.type());
    }
}

@Cheng("db_student")
class student2{
    @Fieldcheng(columnName = "db_id",type = "int",length = 10)
    private int id;
    @Fieldcheng(columnName = "db_age",type = "int",length = 10)
    private int age;
    @Fieldcheng(columnName = "db_name",type = "varchar",length = 3)
    private String name;
  
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface cheng{
    String value();
}

//定义注解
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@interface Fieldcheng{
    String columnName();
    String type();
    int length();
}   
        
    public student2() {
    }
    public student2(int id, int age, String name) {
        this.id = id;
        this.age = age;
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "student2{" +
                "id=" + id +
                ", age=" + age +
                ",  + name + '\'' +
                '}';
    }
}
```