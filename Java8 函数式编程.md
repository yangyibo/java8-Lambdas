# java8 函数式编程

[toc]

### 第一章  简介



------



### 第二章 Lambda表达式

1. ##### 第一个Lambda表达式

* 匿名内部类表示方式：

```java
button.addActionListener(new ActionListener() {
	public void actionPerformed(ActionEvent event) {
		System.out.println("button clicked");
	}
});
```

* Lambda表示方式：

```java
button.addActionListener(event -> System.out.println("button clicked"));
```



2. ##### 如何辨别Lambda表达式

* 匿名内部类表示方式：

```java
Runnable noArguments = () -> System.out.println("Hello World");

ActionListener oneArgument = event -> System.out.println("button clicked");

Runnable multiStatement = () -> {
    System.out.print("Hello");
    System.out.println(" World");
};

BinaryOperator<Long> add = (x, y) -> x + y;

BinaryOperator<Long> addExplicit = (Long x, Long y) -> x + y;
```



- 目标类型是指Lambda表达式所在上下文环境的类型。



3. ##### 引用值，而不是变量

* Lambda表达式中引用既成事实上的final变量，如下代码是正确的

  ```java
  String name = getUserName();
  button.addActionListener(event -> System.out.println("hi " + name));
  ```

* 如下代码会编译不通过，因为name不是**“既成事实上的final变量”**

  ```java
  String name = getUserName();
  name = formatUserName(name);
  button.addActionListener(event -> System.out.println("hi " + name));
  ```



4. ##### 函数接口

* 函数接口是只有一个抽象方法的接口，用作Lambda表达式的类型。

* ActionListener 接口： 接受 ActionEvent 类型的参数， 返回空：

  ```java
  public interface ActionListener extends EventListener {
      public void actionPerformed(ActionEvent event);
  }
  ```

  ![image-20200731090722075](C:\Users\y00318985\AppData\Roaming\Typora\typora-user-images\image-20200731090722075.png)

* Java中重要的函数接口

  ![image-20200731090902580](C:\Users\y00318985\AppData\Roaming\Typora\typora-user-images\image-20200731090902580.png)



5. ##### 类型推断

* Predicate 接口的源码， 接受一个对象， 返回一个布尔值  

  ```java
  public interface Predicate<T> {
      boolean test(T t);
  }
  ```

  ![image-20200731114136461](C:\Users\y00318985\AppData\Roaming\Typora\typora-user-images\image-20200731114136461.png)

* 复杂的类型推断，依赖泛型参数，例如下面的`Long`，如果没有这个参数，编译器会报错。

  ```java
  BinaryOperator<Long> addLongs = (x, y) -> x + y;
  ```

  

6. ##### 要点回顾

* Lambda表达式是一个匿名方法，将行为向数据一样进行传递。
* Lambda表达式的常见结构：`BinaryOperator<Integer> add = (x, y) → x + y  `。
* 函数接口指仅具有单个抽象方法的接口，用来表示Lambda表达式的类型。



7. ##### 练习



------



### 第三章 流

1. ##### 从外部迭代到内部迭代

* 外部迭代

```java
int count = 0;
Iterator<Artist> iterator = allArtists.iterator();
while(iterator.hasNext()) {
    Artist artist = iterator.next();
    if (artist.isFrom("London")) {
        count++;
    }
}
```

![image-20200731120037098](C:\Users\y00318985\AppData\Roaming\Typora\typora-user-images\image-20200731120037098.png)



* 内部迭代

  分解为两步：

  1. 找出所有来自伦敦的艺术家；
  2. 计算他们的人数。

```java
long count = allArtists.stream()
.filter(artist -> artist.isFrom("London"))
.count();
```

![image-20200731120659781](C:\Users\y00318985\AppData\Roaming\Typora\typora-user-images\image-20200731120659781.png)



* Stream是用函数式编程方式在集合类上进行复杂操作的工具。



2. ##### 实现机制

* 像filter这样只描述Stream，最终不产生新集合的方法叫**惰性求值方法**。

* 判断一个操作是惰性求值还是及早求值很简单：只需看它的返回值。如果返回值是Stream，那么是惰性求值；如果反回值是另一个值或为空，那么就是及早求值。

* 整个过程和建造者模式有共通之处。建造者模式使用一系列操作设置属性和配置，最后调用一个build方法，这时，对象才被真正创建。



3. ##### 常用的流操作

   1. collect(toList())

      collect(toList()) 方法由 Stream 里的值生成一个列表， 是一个及早求值操作。  

      ```java
      List<String> collected = Stream.of("a", "b", "c").collect(Collectors.toList());
      assertEquals(Arrays.asList("a", "b", "c"), collected); 
      ```

   2. map

      如果有一个函数可以将一种类型的值转换成另外一种类型， map 操作就可以使用该函数， 将一个流中的值转换成一个新的流。  

      传统写法：

      ```java
      List<String> collected = new ArrayList<>();
      for (String string : asList("a", "b", "hello")) {
          String uppercaseString = string.toUpperCase();
          collected.add(uppercaseString);
      }
      assertEquals(asList("A", "B", "HELLO"), collected);
      ```

      ![image-20200731144046777](C:\Users\y00318985\AppData\Roaming\Typora\typora-user-images\image-20200731144046777.png)

      Stream写法：

      ```java
      List<String> collected = Stream.of("a", "b", "hello")
      .map(string -> string.toUpperCase())
      .collect(toList());
      assertEquals(asList("A", "B", "HELLO"), collected);
      ```

      传给 map 的 Lambda 表达式只接受一个 String 类型的参数， 返回一个新的 String。 参数
      和返回值不必属于同一种类型， 但是 Lambda 表达式必须是 Function 接口的一个实例， Function 接口是只包含一个参数的普通函数接口。

      ![image-20200731145252981](C:\Users\y00318985\AppData\Roaming\Typora\typora-user-images\image-20200731145252981.png)

   3. filter

      ![image-20200731145446269](C:\Users\y00318985\AppData\Roaming\Typora\typora-user-images\image-20200731145446269.png)

      传统写法：

      ```java
      List<String> beginningWithNumbers = new ArrayList<>();
      for(String value : asList("a", "1abc", "abc1")) {
          if (isDigit(value.charAt(0))) {
              beginningWithNumbers.add(value);
          }
      }
      assertEquals(asList("1abc"), beginningWithNumbers);
      ```

      Stream写法：

      ```java
      
      ```

      

