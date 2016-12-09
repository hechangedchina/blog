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