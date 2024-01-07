---
title: "Spel表达式学习"
date: 2024-01-07T21:45:35+08:00
draft: true
lastmod: 2024-01-07
tags: [
    "Spring"
]
categories: [
    "Development"
]
---


## 简介

Spring Expression Language (SpEL) 是强大的表达式语言，支持查询、操作运行时对象图，以及解析逻辑、算术表达式。SpEL可以独立使用，无论是否使用Spring框架。还有一些其他的表达式语言，比如`OGNL`，可用增强`Arthas`问题排查。

## 示例

### 解析计算

```java
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'"); 
String message = (String) exp.getValue();

// Hello World
```

基本所有可能用到的SpEL类和接口都在`org.springframework.expression`和其子包下面，比如`spel.support`。<br />`ExpressionParser`接口用以解析表达式字符串，在上述例子中，表达式串是由单引号`'`括起来的一个字符串。`Expression`接口用来计算先前已解析的表达式字符串。在此过程可能抛两种异常，`ParseException`和<br />`EvaluationException`，分别在`parser.parseExpression`的时候和`exp.getValue()`的时候。

SpEL还可支持调用方法，访问属性，调用构造器等。

```java
// Hello World!
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'.concat('!')"); (1)
String message = (String) exp.getValue();

// invokes 'getBytes().length'
Expression exp = parser.parseExpression("'Hello World'.bytes.length"); (1)
int length = (Integer) exp.getValue();

Expression exp = parser.parseExpression("new String('hello world').toUpperCase()"); 
String message = exp.getValue(String.class);

```

SpEL 更常见的用法是提供针对特定对象实例（称为根对象）进行j表达式字符串。下面是一个调用构造方法的例子：

```java
// Create and set a calendar
GregorianCalendar c = new GregorianCalendar();
c.set(1856, 7, 9);

// The constructor arguments are name, birthday, and nationality.
Inventor tesla = new Inventor("Nikola Tesla", c.getTime(), "Serbian");

ExpressionParser parser = new SpelExpressionParser();

Expression exp = parser.parseExpression("name"); // Parse name as an expression
String name = (String) exp.getValue(tesla);
// name == "Nikola Tesla"

exp = parser.parseExpression("name == 'Nikola Tesla'");
boolean result = exp.getValue(tesla, Boolean.class);
```

## 应用

### Bean Definitions

可以通过XML和注解两种方式来进行使用，语法都是`#{ <expression string> }`

#### XML

```xml
<bean id="numberGuess" class="org.spring.samples.NumberGuess">
  <property name="randomNumber" value="#{ T(java.lang.Math).random() * 100.0 }"/>
</bean>
```

应用程序上下文中的所有Bean都可以作为预定义的变量，以其共同的Bean名称来使用。这包括标准的上下文Bean，如`environment`（类型为`org.springframework.core.env.Environment`），以及用于访问运行时环境的`systemProperties`和`systemEnvironment`（类型为`Map<String, Object>`）。注意：不必在这里用`#`符号给预定义的变量加前缀。

```xml
<bean id="taxCalculator" class="org.spring.samples.TaxCalculator">
    <property name="defaultLocale" value="#{ systemProperties['user.region'] }"/>
</bean>

// numberGuess
<bean id="numberGuess" class="org.spring.samples.NumberGuess">
    <property name="randomNumber" value="#{ T(java.lang.Math).random() * 100.0 }"/>
</bean>

// 引用其他bean
<bean id="shapeGuess" class="org.spring.samples.ShapeGuess">
    <property name="initialShapeSeed" value="#{ numberGuess.randomNumber }"/>
</bean>
```

#### Annotation

常用的`@Value`注解实可使用SpEL表达式：

```java
@Value("#{ systemProperties['user.region'] }")
private String defaultLocale;
```

## 参考语法

### Literal Expressions

支持下列几种：

- strings
- numeric values: integer (`int`or `long`), hexadecimal (`int`or `long`), real (`float`or `double`)
- boolean values: `true`or `false`
- null

```java
// evaluates to "Tony's Pizza"
String pizzaParlor = (String) parser.parseExpression("'Tony''s Pizza'").getValue();

double avogadrosNumber = (Double) parser.parseExpression("6.0221415E+23").getValue();
```

### Properties, Arrays, Lists, Maps, and Indexers

属性首字母不区分大小写，下例也可表示为`Birthdate.Year + 1900`

```java
// evaluates to 1856
int year = (Integer) parser.parseExpression("birthdate.year + 1900").getValue(context);
```

数组和列表的内容是通过使用`[]`获得的，如下例所示：

```java
// Officer's Dictionary
Inventor pupin = parser.parseExpression("officers['president']").getValue(
        societyContext, Inventor.class);
// evaluates to "Idvor"
String city = parser.parseExpression("officers['president'].placeOfBirth.city").getValue(
        societyContext, String.class);
// setting values
parser.parseExpression("officers['advisors'][0].placeOfBirth.country").setValue(
        societyContext, "Croatia");
```

### Inline Lists

可以直接通过`{}`来表示lists

```java
// evaluates to a Java list containing the four numbers
List numbers = (List) parser.parseExpression("{1,2,3,4}").getValue(context);

List listOfLists = (List) parser.parseExpression("{{'a','b'},{'x','y'}}").getValue(context);
```

### Inline Maps

可以直接通过`{key:value}`来表示maps，`{:}`表示一个空的map。

```java
Map inventorInfo = (Map) parser.parseExpression("{name:'Nikola',dob:'10-July-1856'}").getValue(context);
```

### Array Construction（数组构造）

```java
int[] numbers1 = (int[]) parser.parseExpression("new int[4]").getValue(context);

// Array with initializer
int[] numbers2 = (int[]) parser.parseExpression("new int[]{1,2,3}").getValue(context);

// Multi dimensional array
int[][] numbers3 = (int[][]) parser.parseExpression("new int[4][5]").getValue(context);
```

### Methods（方法调用）

可以通过常规的Java语法来进行方法调用：

```java
// string literal, evaluates to "bc"
String bc = parser.parseExpression("'abc'.substring(1, 3)").getValue(String.class);

// evaluates to true
boolean isMember = parser.parseExpression("isMember('Mihajlo Pupin')").getValue(
        societyContext, Boolean.class);
```

### Operators（运算符）

| **类型** | **操作符** |
| --- | --- |
| 算术运算 | +, -, *, /, %, ^, div, mod |
| 关系运算 | <, >, ==, !=, <=, >=, lt, gt, eq, ne, le, ge |
| 逻辑运算 | and, or, not, &&, &#124;&#124;, ! |
| 条件运算 | ?: |
| 正则 | matches |

```java
// evaluates to true
boolean trueValue = parser.parseExpression("2 == 2").getValue(Boolean.class);

// evaluates to false
boolean falseValue = parser.parseExpression("2 < -5.0").getValue(Boolean.class);

// evaluates to true
boolean trueValue = parser.parseExpression("'black' < 'block'").getValue(Boolean.class);

// uses CustomValue:::compareTo
boolean trueValue = parser.parseExpression("new CustomValue(1) < new CustomValue(2)").getValue(Boolean.class);
```

> 注意：
> 当大于小于与`null`进行比较的时候，遵循这么一个规则：null相当于什么都没有，所以所有值都大于`null`,任意值都不小于null，即`x > null`永远为`true`，`x < null`永远为`false`.

```java
// evaluates to false
boolean falseValue = parser.parseExpression(
        "'xyz' instanceof T(Integer)").getValue(Boolean.class);

// evaluates to true
boolean trueValue = parser.parseExpression(
        "'5.00' matches '^-?\\d+(\\.\\d{2})?$'").getValue(Boolean.class);

// evaluates to false
boolean falseValue = parser.parseExpression(
        "'5.0067' matches '^-?\\d+(\\.\\d{2})?$'").getValue(Boolean.class);
```

> 注意：
> 原始类型要小心，因为它们会立即被装箱到它们的包装类型上。`1 instanceof T(int)`等于`false`，`1 instanceof T(Integer)`等于`true`

```java
// evaluates to false
boolean falseValue = parser.parseExpression("true and false").getValue(Boolean.class);
// evaluates to true
boolean trueValue = parser.parseExpression("true or false").getValue(Boolean.class);
// -- AND and NOT --
String expression = "isMember('Nikola Tesla') and !isMember('Mihajlo Pupin')";
boolean falseValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);
```

#### 数学运算符

`+`号可用于数字和字符，`-``*``/``%``^`仅能用于数字，同时保持正常的运算优先级

```java
// Addition
int two = parser.parseExpression("1 + 1").getValue(Integer.class);  // 2

String testString = parser.parseExpression(
        "'test' + ' ' + 'string'").getValue(String.class);  // 'test string'

// Subtraction
int four = parser.parseExpression("1 - -3").getValue(Integer.class);  // 4

double d = parser.parseExpression("1000.00 - 1e4").getValue(Double.class);  // -9000

// Multiplication
int six = parser.parseExpression("-2 * -3").getValue(Integer.class);  // 6
```

#### 赋值运算符

`=`与`setValue`等价

```java
Inventor inventor = new Inventor();
EvaluationContext context = SimpleEvaluationContext.forReadWriteDataBinding().build();

parser.parseExpression("name").setValue(context, inventor, "Aleksandar Seovic");

// alternatively
String aleks = parser.parseExpression(
        "name = 'Aleksandar Seovic'").getValue(context, inventor, String.class);
```

### Types

你可以使用特殊的T操作符来指定一个`java.lang.Class`（类型）的实例。静态方法也是通过使用这个操作符来调用的。`StandardEvaluationContext`使用`TypeLocator`来寻找类型，而`StandardTypeLocator`（可以被替换）是在了解`java.lang`包的情况下建立的。这意味着对`java.lang`包内类型的T()引用不需要完全限定，但所有其他类型的引用必须限定。

```java
Class dateClass = parser.parseExpression("T(java.util.Date)").getValue(Class.class);

Class stringClass = parser.parseExpression("T(String)").getValue(Class.class);

boolean trueValue = parser.parseExpression(
        "T(java.math.RoundingMode).CEILING < T(java.math.RoundingMode).FLOOR")
        .getValue(Boolean.class);
```

### Constructors

可以通过`new`调用构造器方法，如果不是`java.lang`包中的类，需要使用完整的类名

```java
// create new Inventor instance within the add() method of List
p.parseExpression(
        "Members.add(new org.spring.samples.spel.inventor.Inventor(
            'Albert Einstein', 'German'))").getValue(societyContext);
```

### Variables（变量）

 通过`#variables`的形式来引用变量，通过在`EvaluationContext`上`setVariable`可以设置变量

```java
Inventor tesla = new Inventor("Nikola Tesla", "Serbian");

EvaluationContext context = SimpleEvaluationContext.forReadWriteDataBinding().build();
context.setVariable("newName", "Mike Tesla");

parser.parseExpression("name = #newName").getValue(context, tesla);
System.out.println(tesla.getName())  // "Mike Tesla"
```

通过`#this`引用当前对象

```java
// create an array of integers
List<Integer> primes = new ArrayList<Integer>();
primes.addAll(Arrays.asList(2,3,5,7,11,13,17));

// create parser and set variable 'primes' as the array of integers
ExpressionParser parser = new SpelExpressionParser();
EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataAccess();
context.setVariable("primes", primes);

// all prime numbers > 10 from the list (using selection ?{...})
// evaluates to [11, 13, 17]
List<Integer> primesGreaterThanTen = (List<Integer>) parser.parseExpression(
        "#primes.?[#this>10]").getValue(context);
```

### Functions（自定义函数）

通过`EvaluationContext`可以注册自定义的方法

```java
Method method = ...;

EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();
context.setVariable("myFunction", method);
```

```java
public abstract class StringUtils {

    public static String reverseString(String input) {
        StringBuilder backwards = new StringBuilder(input.length());
        for (int i = 0; i < input.length(); i++) {
            backwards.append(input.charAt(input.length() - 1 - i));
        }
        return backwards.toString();
    }
}

ExpressionParser parser = new SpelExpressionParser();

EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();
context.setVariable("reverseString",
        StringUtils.class.getDeclaredMethod("reverseString", String.class));

String helloWorldReversed = parser.parseExpression(
        "#reverseString('hello')").getValue(context, String.class);
```

### Bean References（Bean引用）

如果evaluation context中已经配置过bean解析器，可以使用`@`来查找beans

```java
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();
context.setBeanResolver(new MyBeanResolver());

// This will end up calling resolve(context,"something") on MyBeanResolver during evaluation
Object bean = parser.parseExpression("@something").getValue(context);
```

如果需要factory bean，需要用`&`

```java
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();
context.setBeanResolver(new MyBeanResolver());

// This will end up calling resolve(context,"&foo") on MyBeanResolver during evaluation
Object bean = parser.parseExpression("&foo").getValue(context);
```

### Ternary Operator (If-Then-Else)（三目运算）

```java
parser.parseExpression("name").setValue(societyContext, "IEEE");
societyContext.setVariable("queryName", "Nikola Tesla");

expression = "isMember(#queryName)? #queryName + ' is a member of the ' " +
        "+ Name + ' Society' : #queryName + ' is not a member of the ' + Name + ' Society'";

String queryResultString = parser.parseExpression(expression)
        .getValue(societyContext, String.class);
// queryResultString = "Nikola Tesla is a member of the IEEE Society"
```

### The Elvis Operator

简化版的三目运算符

```java
ExpressionParser parser = new SpelExpressionParser();
EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();

Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
String name = parser.parseExpression("name?:'Elvis Presley'").getValue(context, tesla, String.class);
System.out.println(name);  // Nikola Tesla

tesla.setName(null);
name = parser.parseExpression("name?:'Elvis Presley'").getValue(context, tesla, String.class);
System.out.println(name);  // Elvis Presley

@Value("#{systemProperties['pop3.port'] ?: 25}")
// 如果定义了pops.port则取其值，否则取25
```

### Safe Navigation Operator

从`Groovy`中吸取过来为了避免出现空指针的情况，js中也存在类似语法

```java
ExpressionParser parser = new SpelExpressionParser();
EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();

Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
tesla.setPlaceOfBirth(new PlaceOfBirth("Smiljan"));

String city = parser.parseExpression("placeOfBirth?.city").getValue(context, tesla, String.class);
System.out.println(city);  // Smiljan

tesla.setPlaceOfBirth(null);
city = parser.parseExpression("placeOfBirth?.city").getValue(context, tesla, String.class);
System.out.println(city);  // null - does not throw NullPointerException!!!
```

### Collection Selection（集合选择器）

Selection是一种强大的表达式功能，可让您通过从其条目中进行选择来将源集合转换为另一个集合。其语法为`.?[selectionExpression]`，支持只要实现了`java.lang.Iterable`或者`java.util.Map`的类。

```java
List<Inventor> list = (List<Inventor>) parser.parseExpression(
        "members.?[nationality == 'Serbian']").getValue(societyContext);

Map newMap = parser.parseExpression("map.?[value<27]").getValue();


```

如果获取首个匹配的元素语法为`.^[selectionExpression]`，获取最后一个语法为`.$[selectionExpression]`

### Collection Projection

集合映射，相当于从一个集合中通过子表达式获得一个新的集合。语法为`.![projectionExpression]`：

```java
// returns ['Smiljan', 'Idvor' ]
List placesOfBirth = (List)parser.parseExpression("members.![placeOfBirth.city]");
```

### Expression templating（表达式模板）

表达式模板可以混合普通字符串和其他解析表达式模块，每一个块都用`#{}`来进行区分：

```java
String randomPhrase = parser.parseExpression(
        "random number is #{T(java.lang.Math).random()}",
        new TemplateParserContext()).getValue(String.class);

// evaluates to "random number is 0.7038186818312008"
```

`parseExpression()`方法的第二个参数是 `ParserContext` 类型。`ParserContext`接口被用来影响表达式的解析方式，以支持表达式模板化功能。`TemplateParserContext`是这么定义的：

```java
public class TemplateParserContext implements ParserContext {

    public String getExpressionPrefix() {
        return "#{";
    }

    public String getExpressionSuffix() {
        return "}";
    }

    public boolean isTemplate() {
        return true;
    }
}
```

## 源码分析

## 总结

SpEL是一种功能强大、支持良好的表达式语言，可以在Spring组合的所有产品中使用。我们可以用它来配置Spring应用程序，也可以用它来编写解析器，以便在任何应用程序中执行更普遍的任务。

## 参考文档

1. SpEL官方文档地址：[Spring Expression Language](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#expressions)
2. [Spring Expression Language Guide](https://www.baeldung.com/spring-expression-language)
