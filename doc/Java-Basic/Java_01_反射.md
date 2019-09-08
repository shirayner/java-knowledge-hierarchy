[TOC]





# 前言





# 一、概述

## 1.什么是反射

通过反射，我们可以在运行时动态获取类的信息。



> 通过类的信息可以获取如下内容：
>
> 1. Class 对象
> 2. 类名
> 3. 修饰符
> 4. 包信息
> 5. 父类
> 6. 实现的接口
> 7. 构造器
> 8. 方法
> 9. 变量
> 10. 注解



## 2.反射的用途

只要是需要在运行时动态地获取类的信息的时候，就可以使用反射。

常见的反射的使用场景，如下：

（1）JDBC加载驱动

（2）框架的开发。在许多框架中，需要根据配置文件加载相应的对象，如Spring的Bean的配置文件。





## 3.Class对象

Class类代表着运行中的Java应用中的类和接口。



在你想检查一个类的信息之前，你首先需要获取类的 Class 对象，获取一个类的 Class 对象，有如下方式：

（1）通过类的class属性

```java
Class clazz = MyObject.class;
```

每一个类都有一个隐含的静态成员变量class，可以通过类的静态成员变量class获取类的Class对象



（2）通过对象的 getClass() 方法

```java
MyObject myobject = new MyObject();
Class clazz = myobject.getClass();
```

每一个对象都有一个 getClass() 方法



（3）通过 `Class.forName(className)`

```
String className = "com.ray.study.javabasic.reflect.MyObject";
Class clazz = Class.forName(className);
```







# 二、获取相关信息

有了Class对象，我们就可以通过获取类的相关信息了。

> - getDeclared： 获取当前类的所有的...（构造器、变量、方法）
> - get：获取当前类及其父类的所有的public...（构造器、变量、方法）



## 1.构造器

 Constructor 对象代表着类的构造器



（1）获取构造器

```java
Class clazz = ...  //获取Class对象

// 获取当前类所有已声明的构造器
Constructor[]  constructors = clazz.getDeclaredConstructors();
Constructor constructor =clazz.getDeclaredConstructor(new Class[]{String.class}); 

// 获取当前类及其父类所有已声明的public构造器   
Constructor[] constructors = clazz.getConstructors();
Constructor constructor =clazz.getConstructor(new Class[]{String.class}); 
```



（2）获取构造方法参数

```java
Constructor constructor = ... //获取Constructor对象
Class[] parameterTypes = constructor.getParameterTypes();
```



（3）利用 Constructor 对象实例化一个类

```java
Constructor constructor = MyObject.class.getConstructor(String.class);
MyObject myObject = (MyObject)constructor.newInstance("constructor-arg1");
```







## 2.变量

Field 对象代表着类的成员变量



（1）获取成员变量

```java
Class clazz = ...  //获取Class对象
 
// 获取当前类所有已声明的成员变量
Field[] fields = clazz.getDeclaredFields();
Field field = clazz.getDeclaredField("someField"); 

// 获取当前类及其父类所有已声明的public成员变量
Field[] fields = clazz.getFields();
Field field = clazz.getField("someField"); 
```





（2）获取变量的相关信息

```java
Field field = ... //获取Field对象

// 获取变量名称、变量类型
String fieldName = field.getName(); 
Object fieldType = field.getType();

// 获取/设置变量值
MyObject objectInstance = new MyObject();
// field.setAccessible(true); // 访问私有属性前，需设置允许访问
Object value = field.get(objectInstance);  
field.set(objetInstance, value);
```





## 3.方法

Method 对象代表着类的成员方法



（1）获取 Method 对象

```java
Class clazz = ...  //获取Class对象
    
// 获取当前类所有已声明的成员方法
Method[] methods = clazz.getDeclaredMethods();
Method method = clazz.getDeclaredMethod("doSomething", new Class[]{String.class}); 

// 获取当前类及其父类所有已声明的public成员方法
Method[] methods = clazz.getMethods(); 
Method method = clazz.getMethod("doSomething", new Class[]{String.class}); 


```



（2）获取方法参数以及返回类型

```java
Class[] parameterTypes = method.getParameterTypes();
Class returnType = method.getReturnType();
```



（3）通过 Method 对象调用方法

```java
Object returnValue = method.invoke(null, "parameter-value1");  // 静态调用传入对象为null
MyObject objectInstance = new MyObject();
Object returnValue = method.invoke(objectInstance, "parameter-value1");
```



## 4.私有变量和私有方法

（1）私有变量

```java
// 获取当前类所有已声明的成员变量
Field[] fields = clazz.getDeclaredFields();
Field field = clazz.getDeclaredField("someField"); 

// 访问私有属性前，需设置允许访问
field.setAccessible(true); 
Object value = field.get(objectInstance);  
field.set(objetInstance, value);
```



（2）私有方法

```java
// 获取当前类所有已声明的成员方法
Method[] methods = clazz.getDeclaredMethods();
Method method = clazz.getDeclaredMethod("doSomething", new Class[]{String.class}); 

// 访问私有方法前，需设置允许访问
method.setAccessible(true);
Object returnValue = method.invoke(objectInstance, null);
```









## 6.注解

通过反射机制，你可以在运行期访问类，方法或者变量的注解信息。



（1）获取类注解

```java
Class clazz = ...  //获取Class对象
Annotation[] annotations = clazz.getAnnotations();
Annotation annotation = clazz.getAnnotation(MyAnnotation.class);
```



（2）获取方法注解

```java
Class clazz = ...  //获取Class对象
Method method = clazz.getDeclaredMethod("doSomething", new Class[]{String.class}); 

Annotation[] annotations = method.getDeclaredAnnotations();
Annotation annotation = method.getDeclaredAnnotation(MyAnnotation.class);

```



（3）获取参数注解

```
Annotation[][] parameterAnnotations = method.getParameterAnnotations();
Class[] parameterTypes = method.getParameterTypes();
```



（4）获取变量注解

```java
Annotation[] annotations = field.getDeclaredAnnotations();
Annotation annotation = field.getAnnotation(MyAnnotation.class);
```





注解示例：

```java
@MyAnnotation(name="someName",  value = "Hello World")
public class TheClass {
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface MyAnnotation {
  public String name();
  public String value();
}
```





## 7.泛型

下面是两个典型的使用泛型的场景： 1、声明一个需要被参数化（parameterizable）的类/接口。 2、使用一个参数化类。

```java
  public class MyClass {

      protected List<String> stringList = ...;

      public List<String> getStringList(){
        return this.stringList;
      }
      
      public void setStringList(List<String> list){
        this.stringList = list;
      }
  }
```





（1）泛型方法返回类型

```java
Method method = MyClass.class.getMethod("getStringList", null);

Type returnType = method.getGenericReturnType();

if(returnType instanceof ParameterizedType){
    ParameterizedType type = (ParameterizedType) returnType;
    Type[] typeArguments = type.getActualTypeArguments();
    for(Type typeArgument : typeArguments){
        Class typeArgClass = (Class) typeArgument;
        System.out.println("typeArgClass = " + typeArgClass);
    }
}
```



（2）泛型方法参数类型

```java
method = Myclass.class.getMethod("setStringList", List.class);

Type[] genericParameterTypes = method.getGenericParameterTypes();

for(Type genericParameterType : genericParameterTypes){
    if(genericParameterType instanceof ParameterizedType){
        ParameterizedType aType = (ParameterizedType) genericParameterType;
        Type[] parameterArgTypes = aType.getActualTypeArguments();
        for(Type parameterArgType : parameterArgTypes){
            Class parameterArgClass = (Class) parameterArgType;
            System.out.println("parameterArgClass = " + parameterArgClass);
        }
    }
}
```



（3）泛型变量类型

```java
Field field = MyClass.class.getField("stringList");

Type genericFieldType = field.getGenericType();

if(genericFieldType instanceof ParameterizedType){
    ParameterizedType aType = (ParameterizedType) genericFieldType;
    Type[] fieldArgTypes = aType.getActualTypeArguments();
    for(Type fieldArgType : fieldArgTypes){
        Class fieldArgClass = (Class) fieldArgType;
        System.out.println("fieldArgClass = " + fieldArgClass);
    }
}
```





## 8.反射创建对象

（1）通过`clazz.newInstance`

```java
MyObject objectInstance = (MyObject)clazz.newInstance();
```

内部是通过 `constructor.newInstance`实现



（2）通过`constructor.newInstance`

```java
MyObject myObject = (MyObject)constructor.newInstance("constructor-arg1");
```







# 三、示例

## 1.父类 Animal

```java
package com.ray.study.javabasic.reflect;

/**
 * description
 *
 * @author shira 2019/08/02 9:54
 */
public class Animal {

    private String type;

    public Animal(){
    }

    public Animal(String type){
        this.type = type;
    }


    private boolean say(String content) {
        System.out.println(content);
        return true;
    }

    public void say(){
        say("i am a "+type);
    }

    public void run() {

    }


    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }
}

```



## 2.子类 Dog

```
package com.ray.study.javabasic.reflect;

/**
 * description
 *
 * @author shira 2019/08/02 9:54
 */
public class Dog extends Animal {

    private  String name;

    private int age;

    public Dog() {
    }

    private Dog(String type, String name) {
        super(type);
        this.name = name;
    }

    public Dog(String type, String name, int age) {
        super(type);
        this.name = name;
        this.age = age;
    }

    @Override
    public void run() {
        System.out.println("i am a dog,i can run fast!");
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}

```



## 3.测试类  AnimalTest



```java
package com.ray.study.javabasic.reflect;

import org.junit.Test;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

/**
 * description
 *
 * @author shira 2019/08/02 10:19
 */
public class AnimalTest {

    /**
     * 1.构造器
     * @throws NoSuchMethodException
     */
    @Test
    public void constructor() throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        Class clazz = Dog.class;
        Constructor[] constructors = null;

        // ===========================================
        // （1）获取构造器
        // ===========================================

        // 获取当前类所有已声明的构造器
        System.out.println("===========getDeclaredConstructors==============");
        constructors = clazz.getDeclaredConstructors();
        for(Constructor constructor:constructors){
            System.out.println(constructor);
        }

        // 获取当前类及其父类所有已声明的public构造器
        System.out.println("==========getConstructors===============");
        constructors = clazz.getConstructors();
        for(Constructor constructor:constructors){
            System.out.println(constructor);
        }

        //        System.out.println("=========getConstructor================");
        // 只能获取public构造器
        //        Constructor constructor =clazz.getConstructor(new Class[]{String.class, String.class});
        //        System.out.println(constructor);


        System.out.println("=========getDeclaredConstructor================");
        Constructor constructor =clazz.getDeclaredConstructor(new Class[]{String.class, String.class});
        System.out.println(constructor);

        // ===========================================
        // （2）获取构造方法参数
        // ===========================================
        Class[] parameterTypes = constructor.getParameterTypes();


        // ===========================================
        // （3）通过构造器实例化类
        // ===========================================
        //设置是否允许访问，因为该构造器是private的，所以要手动设置允许访问，如果构造器是public的就不需要这行了。
        constructor.setAccessible(true);
        Animal animal = (Animal)constructor.newInstance("dog","huskie");
        System.out.println(animal.getType());
    }


    /**
     * 2.成员变量
     */
    @Test
    public void field() throws NoSuchFieldException, IllegalAccessException {
        Dog dog = new Dog("dog", "huskie", 10);
        Class clazz = dog.getClass();

        // ===========================================
        // （1）获取成员变量
        // ===========================================

        // 获取当前类所有已声明的成员变量
        System.out.println("==========getDeclaredFields===============");
        Field[] fields = clazz.getDeclaredFields();
        for(Field field : fields){
            System.out.println(field);
            //System.out.println(field.getName() +"  "+ field.getType());
        }

        // 获取当前类及其父类所有已声明的public成员变量
        System.out.println("==========getFields===============");
        fields = clazz.getFields();
        for(Field field : fields){
            System.out.println(field);
            //System.out.println(field.getName() +"  "+ field.getType());
        }



        // ===========================================
        // （2）获取/设置变量值
        // ===========================================
        System.out.println("==========get/set field value===============");
        Field field = clazz.getDeclaredField("name");  // 指定变量名来访问
        //设置是否允许访问
        field.setAccessible(true);
        Object value = field.get(dog);  // 获取变量值
        System.out.println(value);
        field.set(dog, "huskie2");   // 设置变量值
        System.out.println(dog.getName());

    }


    /**
     * 3.成员方法
     */
    @Test
    public void method() throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        Dog dog = new Dog("dog", "huskie", 10);
        Class clazz = dog.getClass();

        // ===========================================
        // （1）获取成员方法
        // ===========================================

        // 获取当前类所有已声明的成员方法
        System.out.println("==========getDeclaredMethods===============");
        Method[] methods = clazz.getDeclaredMethods();
        for(Method method : methods){
            System.out.println(method);
        }

        // 获取当前类及其父类所有已声明的public成员方法
        System.out.println("==========getMethods===============");
        methods = clazz.getMethods();
        for(Method method : methods){
            System.out.println(method);
        }

        Method method = clazz.getDeclaredMethod("getName", null);  // 根据参数名称和参数类型，获取指定方法
        System.out.println(method);


        // ===========================================
        // （2）获取方法参数以及返回类型
        // ===========================================
        System.out.println("==========getParameterTypes and returnType===============");
        Class[] parameterTypes = method.getParameterTypes();
        for(Class parameterType : parameterTypes) {
            System.out.println(parameterType);
        }
        Class returnType = method.getReturnType();
        System.out.println(returnType);

        // ===========================================
        // （3）通过 Method 对象调用方法
        // ===========================================
        System.out.println("==========method.invoke===============");
        Object returnValue = method.invoke(dog, null);
        System.out.println(returnValue);

    }




}
```







# 四、反射的使用示例

## 1.JDBC加载驱动

```java
package com.ray.study.javabasic.reflect;

import java.sql.*;

/**
 * description
 *
 * @author shira 2019/08/02 14:57
 */
public class JDBCDemo {

    public static void main(String[] args) throws SQLException, ClassNotFoundException {
        // 加载驱动类
        Class.forName("com.mysql.jdbc.Driver");

        // 通过DriverManager获取数据库连接
        String url = "jdbc:mysql://192.168.1.150/test";
        String user = "teamtalk";
        String password = "123456";
        Connection connection = (Connection) DriverManager.getConnection(
                url, user, password);

        PreparedStatement statement = (PreparedStatement) connection.prepareStatement(
                "insert person (name, age) value (?, ?)");
        statement.setString(1, "hdu");
        statement.setInt(2, 21);
        statement.executeUpdate();

        ResultSet resultSet = statement.executeQuery("select * from person");
        // 操作ResultSet结果集
        while (resultSet.next()) {
            // 第一种获取字段方式
            System.out.println(resultSet.getString(1) + " " +
                    resultSet.getString(2) + " " + resultSet.getString(3));
        }

        // 关闭数据库连接
        resultSet.close();
        statement.close();
        connection.close();

    }
}

```









# 参考资料

1. [java 类](http://wiki.jikexueyuan.com/project/java-reflection/java-classes.html)
2. [Java Reflection Tutorial](http://tutorials.jenkov.com/java-reflection/index.html)
3. [Java 反射由浅入深 | 进阶必备](https://juejin.im/post/598ea9116fb9a03c335a99a4)
4. [Java基础与提高干货系列——Java反射机制](http://tengj.top/2016/04/28/javareflect/)
5. [学习java应该如何理解反射？](https://www.zhihu.com/question/24304289)
6. [深入解析Java反射（1） - 基础](https://www.sczyh30.com/posts/Java/java-reflection-1/)
7. 

