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
现在类型推断可以通过目标类型推断出泛型方法调用的类型参数. Java7可以对赋值语句使用类型推断, 而在Java8中会对更多的上下文中的目标类型进行类型推断.

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

## Java5
### [*泛型](http://docs.oracle.com/javase/8/docs/technotes/guides/language/generics.html)
这种期待已久的对类型系统的增强允许类型或方法对各种类型的对象进行操作，同时提供编译时类型的安全性。它将编译器类型安全添加到集合框架中, 消除了类型转换的苦差事.  
Java中的泛型和C++的模板看上去很相似, 但这相似性是肤浅的: Java的泛型的实现机制是类型擦除, 并不支持泛型对象实例化以及模板元编程.

Java5之前的集合操作写法
```java
// Removes 4-letter words from c. Elements must be strings
static void expurgate(Collection c) {
    for (Iterator i = c.iterator(); i.hasNext(); )
      if (((String) i.next()).length() == 4)
        i.remove();
}
```
使用泛型的集合操作
```java
// Removes the 4-letter words from c
static void expurgate(Collection<String> c) {
    for (Iterator<String> i = c.iterator(); i.hasNext(); )
      if (i.next().length() == 4)
        i.remove();
}
```
更多的泛型方面资料需参考官方文档和书籍学习.
### [*For-each](http://docs.oracle.com/javase/8/docs/technotes/guides/language/foreach.html)
这个循环的新特性能让你更方便的迭代遍历数组和集合
Java5之前的写法
```java
void cancelAll(Collection<TimerTask> c) {
    for (Iterator<TimerTask> i = c.iterator(); i.hasNext(); )
        i.next().cancel();
}
for (Iterator i = suits.iterator(); i.hasNext(); ) {
    Suit suit = (Suit) i.next();
    for (Iterator j = ranks.iterator(); j.hasNext(); )
        sortedDeck.add(new Card(suit, j.next()));
}
```
ForEach写法
```java
void cancelAll(Collection<TimerTask> c) {
    for (TimerTask t : c)
        t.cancel();
}
for (Suit suit : suits)
    for (Rank rank : ranks)
        sortedDeck.add(new Card(suit, rank));
```
数组的ForEach操作
```java
int sum(int[] a) {
    int result = 0;
    for (int i : a)
        result += i;
    return result;
}
```
### [*自动装箱/拆箱](https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html)
Java5会在需要的时候自动在基本类型和相应的包装类型之间进行转换

```java
List<Integer> li = new ArrayList<>();
for (int i = 1; i < 50; i += 2)
    li.add(i);

public static int sumEven(List<Integer> li) {
    int sum = 0;
    for (Integer i: li)
        if (i % 2 == 0)
            sum += i;
        return sum;
}
```
其实是
```java
List<Integer> li = new ArrayList<>();
for (int i = 1; i < 50; i += 2)
    li.add(Integer.valueOf(i));

public static int sumEven(List<Integer> li) {
    int sum = 0;
    for (Integer i : li)
        if (i.intValue() % 2 == 0)
            sum += i.intValue();
        return sum;
}
```
这个转换过程被编译器自动完成了

### [*类型安全枚举](http://docs.oracle.com/javase/8/docs/technotes/guides/language/enums.html)
在java的以往版本中实现枚举模式通常使用整形常数, 这会带来很多问题:
* 类型不安全: 它可以当做数字使用
* 无名字空间: 必须在名字上添加前缀以同其他枚举区分
* 较脆弱: 如果改变了枚举的顺序或者增加新的枚举, 就需要重新编译所有代码以保证不会出现未定义行为
* 不直观: 输出枚举值的Log打印出来的仅仅只是int, 不能直观的表达任何信息

新增的关键字enum可以定义一个无法被实例化和继承(final)的类, 在这个类里可以定义一系列作为枚举值的常量供用户使用. 类型安全枚举的好处:
* 枚举常量是枚举类的实例, 他是类型安全且不会和其他枚举类型发生冲突的. 
* 增加了新的枚举常量也不会影响到原有代码的正常工作
* 枚举常量使用toString()可以输出直接常量字面值, 更加直观.
* 用户可以在枚举类中定义一些枚举变量可以使用的方法, 让程序逻辑更加清晰.

```java
public enum Operation {
  PLUS   { double eval(double x, double y) { return x + y; } },
  MINUS  { double eval(double x, double y) { return x - y; } },
  TIMES  { double eval(double x, double y) { return x * y; } },
  DIVIDE { double eval(double x, double y) { return x / y; } };

  // Do arithmetic op represented by this constant
  abstract double eval(double x, double y);
}
public static void main(String args[]) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    for (Operation op : Operation.values())
        System.out.printf("%f %s %f = %f%n", x, op, y, op.eval(x, y));
}
```
```bash

$ java Operation 4 2
4.000000 PLUS 2.000000 = 6.000000
4.000000 MINUS 2.000000 = 2.000000
4.000000 TIMES 2.000000 = 8.000000
4.000000 DIVIDE 2.000000 = 2.000000
```
在Effective Java的第21条提到用类代替enum结构, 其内容讲的就是实现一种"类型安全枚举"的模型, 用这种更安全的枚举模型来替代原有的类C语言的枚举.而Java5直接在语言层面实现了这种类型安全的枚举模型.在没有极其苛刻的性能要求的情况下(如果你的JVM不是运行在蜂窝电话或者烤面包机上),类型安全枚举带来的额外开销(装载枚举类以及构造常量对象)不会引起用户注意.

### [*变长参数](http://docs.oracle.com/javase/8/docs/technotes/guides/language/varargs.html)
方法调用的最后一个参数允许使用变长参数, 消除了用户手动创建数组打包参数传入方法的痛苦.

Java5以前想要格式化字符串只能这样写
```java
Object[] arguments = {
    new Integer(7),
    new Date(),
    "a disturbance in the Force"
};

String result = MessageFormat.format(
    "At {1,time} on {1,date}, there was {2} on planet "
     + "{0,number,integer}.", arguments);
```
而现在
```java
String result = MessageFormat.format(
    "At {1,time} on {1,date}, there was {2} on planet "
    + "{0,number,integer}.",
    7, new Date(), "a disturbance in the Force");
```
在Java5里String类基于变长参数的特性提供了新的api, format(). 如果阅读一下源码会发现它直接调用MessageFormat.format()做格式串生成.
```java
public static String format(String pattern,
                                Object... arguments);
```

### [*静态导入](http://docs.oracle.com/javase/8/docs/technotes/guides/language/static-import.html)
导入类内所有静态成员到当前名字空间, 不需要添加类前缀即可引用被导入的静态成员. 官方建议谨慎和适当的使用静态导入以免污染名字空间以及破坏程序的可读性和可维护性.

```java
double r = Math.cos(Math.PI * theta);

import static java.lang.Math.*;
double r = cos(PI * theta);
```

### [*注解](http://docs.oracle.com/javase/8/docs/technotes/guides/language/annotations.html)
这个新特性可以让程序员在代码中使用声明式编程: 在代码中添加注解, 用工具从注解中生成代码. 这种特性可以极大的减少程序员的工作量. 参考库ButterKnife. 此外程序本身也可以通过反射获取属性和方法的注解信息.
定义注解以及使用: 
```java
public @interface RequestForEnhancement {
    int    id();
    String synopsis();
    String engineer() default "[unassigned]"; 
    String date()    default "[unimplemented]"; 
}

@RequestForEnhancement(
    id       = 2868724,
    synopsis = "Enable time-travel",
    engineer = "Mr. Peabody",
    date     = "4/1/3007"
)
public static void travelThroughTime(Date destination) { ... }
```
更多的注解方面资料需参考官方文档和书籍学习.


-------

感想: Java5之前的Java程序员真辛苦