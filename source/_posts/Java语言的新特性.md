---
title: Java语言的新特性
date: 2016-12-09 10:30:34
tags: Java
---
从[官方文档](https://docs.oracle.com/javase/8/docs/technotes/guides/language/enhancements.html)上学习了一下从Java5到Java8各个版本的语言新特性, 在此总结.

## Java8
### [*lambda表达式](http://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
可以把一个[函数式接口](https://docs.oracle.com/javase/8/docs/api/java/lang/FunctionalInterface.html)写作lambda表达式
```java
//常规写法
new Thread(new Runnable() {
  @Override
  public void run() {
    //do sth
  }
}).start();

//lambda表达式写法
new Thread(()->{/*do sth*/}).start();
```
lambda表达式也支持以下特性
* [方法引用](http://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html): 已有名称的方法可以用紧凑的,容易阅读的lambda表达式来表现  
```java

public class Person {
	......
    public static int compareByAge(Person a, Person b) {
        return a.birthday.compareTo(b.birthday);
    }}
}

Person[] rosterAsArray = roster.toArray(new Person[roster.size()]);

//常规写法        
class PersonAgeComparator implements Comparator<Person> {
    public int compare(Person a, Person b) {
        return a.getBirthday().compareTo(b.getBirthday());
    }
}
Arrays.sort(rosterAsArray, new PersonAgeComparator());

//lambda表达式写法
Arrays.sort(rosterAsArray,
    (Person a, Person b) -> {
        return a.getBirthday().compareTo(b.getBirthday());
    }
);

//lambda表达式写法
Arrays.sort(rosterAsArray,
    (a, b) -> Person.compareByAge(a, b)
);

//方法引用写法
Arrays.sort(rosterAsArray, Person::compareByAge);
```

* [默认方法](http://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html): 可以在interface内添加使用default关键字修饰的具有实现的方法, 它和旧版本的interface是兼容的;  
并且可以在interface中定义静态(static)方法
```java
	interface I {
	  void a();
	  //静态方法
	  static void b() {
	    //do sth
	  }
	  //默认方法
	  default void c() {
	    //do sth
	  }
	}
```

* [新api的增强](http://docs.oracle.com/javase/8/docs/technotes/guides/language/lambda_api_jdk8.html): 加入一些支持lambda表达式的包如[java.util.function](http://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html), [java.util.stream](http://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html).
另外在[java.util.Collection](http://docs.oracle.com/javase/8/docs/api/java/util/Collection.html)接口中增加了一个default方法 
```java
default Stream<E> stream();
```
为集合提供流操作

### *改进的[类型推断](http://docs.oracle.com/javase/tutorial/java/generics/genTypeInference.html)
现在类型推断可以通过目标类型推断出泛型方法调用的类型参数. Java7可以对赋值语句使用类型推断, 而在Java8中会对更多的上下文对目标类型进行类型推断.

```java
List<String> stringList = new ArrayList<>(); //Java7特性: 对泛型实例创建进行类型推断
stringList.add("A");
stringList.addAll(Arrays.asList());  //Java8 特性: 可以从方法参数的类型参数来进行类型推断
```

### [*类型注解](http://docs.oracle.com/javase/tutorial/java/annotations/basics.html)
现在可以在任何使用类型的地方应用注解
```java
	@NonNull String str;
	//实例创建
	new @Interned MyObject();
	//类型转换
	myString = (@NonNull String) str;
	//接口实现
	class UnmodifiableList<T> implements
        @Readonly List<@Readonly T> { ... }
	//异常声明
	void monitorTemperature() throws
        @Critical TemperatureException { ... }
```
### [*重复注解](http://docs.oracle.com/javase/tutorial/java/annotations/repeating.html)
可以在一个类型定义上重复的使用多次注解(加入了@Repeatable注解类型)
```java
public @interface Schedules {
    Schedule[] value();
}
....
@Repeatable(Schedules.class)
public @interface Schedule {
  String dayOfMonth() default "first";
  String dayOfWeek() default "Mon";
  int hour() default 12;
}
...
@Schedule(dayOfMonth="last")
@Schedule(dayOfWeek="Fri", hour="23")
public void doPeriodicCleanup() { ... }
```

### [*方法参数反射](http://docs.oracle.com/javase/tutorial/reflect/member/methodparameterreflection.html)
现在可以使用[java.lang.reflect.Executable.getParameters](http://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Executable.html#getParameters)得到[构造器](http://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Constructor.html),和[方法](http://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Method.html)的参数表
(Executable是构造器和方法的父类)

```java
Method method = ...;
method.getParameters();

Constructor constructor = ...;
constructor.getParameters();
```
## Java7
### [*二进制字面常量](http://docs.oracle.com/javase/8/docs/technotes/guides/language/binary-literals.html)
可以使用二进制字面常量来表达整形类型(byte, short, int, long), 以0b(0B)作为前缀
```java
System.out.println(0b11); // 3
System.out.println(0B111); // 7
```
### [*字面常量数字的下划线](http://docs.oracle.com/javase/8/docs/technotes/guides/language/underscores-literals.html)
在数字常量中可以使用下划线来提高数字的可读性, 不能将下划线添加在数字头尾部
```java
System.out.println(11_1); // 111
System.out.println(011_1); // 73
System.out.println(0x11_1); // 273
System.out.println(0B11_1); // 7
```

### [*String类可以用于switch语句](http://docs.oracle.com/javase/8/docs/technotes/guides/language/strings-switch.html)
这里case语句中的statement只能使用常量字符串
```java
String str = new String("aaa");
switch (str) {
    case "aaa":
        System.out.println(true);
        break;
    case "bbb":
        System.out.println(false);
        break;
    default:
        break;
}
```

### [*泛型实例创建的类型推断](http://docs.oracle.com/javase/8/docs/technotes/guides/language/type-inference-generic-instance-creation.html)
只要编译器可以根据上下文推断出泛型实例的类型参数, 在调用泛型类型的构造器时可以用<>(学名叫做菱形"diamond")来替代指定类型参数.
```java
List<Integer> integerList = new ArrayList<>();
```
### [*优化变长参数的方法调用](http://docs.oracle.com/javase/8/docs/technotes/guides/language/non-reifiable-varargs.html)
在变长参数中使用不可具体化的类型(例如List<String>)时, 编译器会报出警告, 该警告可以被
``` java
@SafeVarargs
```
和
``` java
@SuppressWarnings({"unchecked", "varargs"})
```
抑制.
```java

//Main.java
...
    public static void main(String[] args) {
        List<String> a = new ArrayList<String>();
        fun(a, a);
    }

    public static <T> T fun(T... args) {
        for (T t : args) {
            System.out.println(t);
        }
        return args[0];
    }
...
```
使用java6 编译Main.java 加上 -Xlint:unchecked参数
``` bash
jdk1.6.0_45\bin\javac.exe -Xlint:unchecked Main.java
Main.java:38: 警告：[unchecked] 对于 varargs 参数，类型 java.util.List<java.lang.String>[] 的泛型数组创建未经检查
        fun(a, a);
           ^
1 警告
```
使用java7 编译Main.java 加上 -Xlint:unchecked参数
``` bash
jdk1.7.0_80\bin\javac.exe -Xlint:unchecked Main.java
Main.java:32: 警告: [unchecked] 对于类型为List<String>[]的 varargs 参数, 泛型数组创建未经过检查
        fun(a, a);
           ^
Main.java:35: 警告: [unchecked] 参数化 vararg 类型T的堆可能已受污染
    public static <T> T fun(T... args) {
                                 ^
  其中, T是类型变量:
    T扩展已在方法 <T>fun(T...)中声明的Object
2 个警告
```
添加 @SafeVarargs
``` java
@SafeVarargs
public static <T> T fun(T... args) {
    for (T t : args) {
        System.out.println(t);
    }
    return args[0];
}
```
编译器警告消失

添加 @SuppressWarnings({"unchecked", "varargs"}), 
``` bash
jdk1.7.0_80\bin\javac.exe -Xlint:unchecked Main.java
Main.java:32: 警告: [unchecked] 对于类型为List<String>[]的 varargs 参数, 泛型数组创建未经过检查
        fun(a, a);
           ^
1 个警告
```
### [*try_with_resource](http://docs.oracle.com/javase/8/docs/technotes/guides/language/try-with-resources.html)
使用try_with_resource语句, 可以对实现了AutoCloseable(java7 加入的接口, 原有的Closeable成为它的子类) 的资源进行自动关闭, 而不用手动在finally语句块中关闭.

```java
//常规写法
static String readFirstLineFromFileWithFinallyBlock(String path) throws IOException {
  BufferedReader br = new BufferedReader(new FileReader(path));
  try {
    return br.readLine();
  } finally {
    if (br != null) br.close();
  }
}

//try_with_resource
static String readFirstLineFromFile(String path) throws IOException {
  try (BufferedReader br = new BufferedReader(new FileReader(path))) {
    return br.readLine();
  }
}

//try_with_resource
public static void writeToFileZipFileContents(String zipFileName, String outputFileName)
        throws java.io.IOException {

    java.nio.charset.Charset charset = java.nio.charset.StandardCharsets.US_ASCII;
    java.nio.file.Path outputFilePath = java.nio.file.Paths.get(outputFileName);

    // Open zip file and create output file with try-with-resources statement

    try (
            java.util.zip.ZipFile zf = new java.util.zip.ZipFile(zipFileName);
            java.io.BufferedWriter writer = java.nio.file.Files.newBufferedWriter(outputFilePath, charset)
    ) {

        // Enumerate each entry

        for (java.util.Enumeration entries = zf.entries(); entries.hasMoreElements(); ) {

            // Get the entry name and write it to the output file

            String newLine = System.getProperty("line.separator");
            String zipEntryName = ((java.util.zip.ZipEntry) entries.nextElement()).getName() + newLine;
            writer.write(zipEntryName, 0, zipEntryName.length());
        }
    }
}
```
这里需要注意的是: 
* 若try语句块抛出异常, 则在try_with_resource关闭资源调用AutoCloseable.close()时抛出的IOException会被抑制, catch到的异常将是try语句块抛出的异常. 这里可以调用Throwable.getSuppressed()获取被抑制的异常. 
* 在使用finally语句块中关闭资源时, 若try语句块抛出了异常, 则在finally语句块中抛出的IOException异常会覆盖掉try语句块中抛出的异常.
* Closeable 是java5加入的关闭资源的方法, 多次调用Closeable.close()关闭资源不会引发错误
* AutoCloseable 是java7加入的关闭资源的方法, 仅允许调用一次AutoCloseable.close();

### [* 多重异常捕获&优化对重抛出异常的类型检查](http://docs.oracle.com/javase/8/docs/technotes/guides/language/catch-multiple.html)
多重异常捕获
```java
//之前的写法
catch (IOException ex) {
     logger.log(ex);
     throw ex;
} catch (SQLException ex) {
     logger.log(ex);
     throw ex;
}

//现在可以这样写
catch (IOException|SQLException ex) {
    logger.log(ex);
    throw ex;
}
```
重抛出更精确描述的异常, 在java7之前的版本不允许这样写
```java
public void rethrowException(String exceptionName)
        throws IOException, NullPointerException {
    try {
    }
    catch (Exception e) {
        //在这里重抛出类型只能为IOException, NullPointerException, 
        //若有其他类型, 必须在throw子句中声明
        throw e;
    }
}
```

## Java6
java6 在语言方面没有改动, 主要改动在新增了一些包之类的.