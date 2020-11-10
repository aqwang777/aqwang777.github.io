---
layout: post
title:  "java8的lambda表达式所有用法合集"
categories: java8
tags:  java8
author: aqwang
---

* content
{:toc}


### .map()

**用于映射每个元素到对应的结果。关于.map()用法的三个demo：**

```java
		List<Student> str=new ArrayList<>();       
		List<Integer> collect = str.stream().map(Student::getNum).collect(Collectors.toList());
```

```java
        List<String> collect1 = str.stream().map(a -> a.getSex()).collect(Collectors.toList());
```

```java
        List<String> collect2 = str.stream().map(a -> {
            a.getName();
            a.getNum();
            return a.getSex();
        }).collect(Collectors.toList());
```

- 如上图所示，第一个demo利用.map()和::关键字，直接获取到了每一个元素的Num属性，最后再从流转化为集合，得到想要的num的集合。
- 第二个demo利用.map()和->关键字，获取到了sex的集合。
- 第三个demo相对于第二个demo多了大括号{}。加入大括号就必须要进行return，加了大括号之后，可以在大括号里面做一些其他的操作，比如赋值给值，最后return自己想要的东西，然后转换为集合。




### .foreach()

**迭代流中的每个数据。**

```
str.stream().forEach(a->{a.getNum();});
```

用foreach()可以在里面做很多操作，同.map()的区别是foreach()没有返回。这个方法我的理解是，就是用来处理一些集合的业务逻辑的，然后也不能进行返回。



### .filter()

**filter 方法用于通过设置的条件过滤出元素。**

```java
 List<Student> collect = str.stream().filter(a -> StringUtils.equals(a.getName(),a.getSex())).collect(Collectors.toList());
```

用.filter()方法找出符合条件的元素。



### .limit()/.skip()

**limit 返回 Stream 的前面 n 个元素；skip 则是扔掉前 n 个元素.**

```
List<String> forEachLists = new ArrayList<>();
forEachLists.add("a");
forEachLists.add("b");
forEachLists.add("c");
forEachLists.add("d");
forEachLists.add("e");
forEachLists.add("f");
List<String> limitLists = forEachLists.stream().skip(2).limit(3).collect(Collectors.toList());
```

返回的是c,d,e



### .sorted()

**以自然序排序**

```
 List<Integer> sortLists = new ArrayList<>();
        sortLists.add(1);
        sortLists.add(4);
        sortLists.add(6);
        sortLists.add(3);
        sortLists.add(2);
 List<Integer> afterSortLists = sortLists.stream().sorted().collect(Collectors.toList());
```

排序过后的集合是1，2，3，4，6



### .distinct()

**对一个集合进行查重**

```
List<String> distinctList = new ArrayList<>();
distinctList.add("a");
distinctList.add("a");
distinctList.add("c");
distinctList.add("d");
List<String> afterDistinctList = distinctList.stream().distinct().collect(Collectors.toList());
```

返回为a,c,d



### Match方法

有的时候，我们只需要判断集合中是否全部满足条件，或者判断集合中是否有满足条件的元素，这时候就可以使用match方法：
 allMatch：Stream 中全部元素符合传入的 predicate，返回 true
 anyMatch：Stream 中只要有一个元素符合传入的 predicate，返回 true

noneMatch：Stream 中没有一个元素符合传入的 predicate，返回 true

**allMatch的demo**

```
List<String> matchList = new ArrayList<>();
matchList.add("a");
matchList.add("a");
matchList.add("c");
matchList.add("d"); 

boolean isExits = matchList.stream().anyMatch(s -> s.equals("c"));
```

**noneMatch的demo**

```
List<String> matchList = new ArrayList<>();
matchList.add("a");
matchList.add("");
matchList.add("a");
matchList.add("c");
matchList.add("d");
boolean isNotEmpty = matchList.stream().noneMatch(s -> s.isEmpty());
```



### .findAny()

**该方法可以获取Stream中的任意一个元素**

```
       List<String> maxLists = new ArrayList<>();
        maxLists.add("a");
        maxLists.add("b");
        maxLists.add("c");
        maxLists.add("d");
        maxLists.stream().filter(a->!a.equals("b")).findAny().orElse(null);
```



### .flatMap()

**两个集合合为一个**

```

    List<AClass> unionResult = Stream.of(aClassList1, aClassList2).flatMap(Collection::stream).distinct().collect(Collectors.toList());

```



### ：：（双冒号）

方法引用的格式为`<ClassName | instance>::<MethodName>`。也就是被引用的方法所属的类名和方法名用双冒号`::`隔开，构造器方法是个例外，引用会用到`new`关键字，总结了一下：

| 引用方式           | 说明                                               |
| ------------------ | -------------------------------------------------- |
| 静态方法说明       | ClassName：：staticMethodName 例如Math：：abs      |
| 构造器引用         | ClassName：：new 例如通过Supplier<T>返回新实例     |
| 类任意实例方法引用 | ClassName：：instanceMethodName 例如String::concat |
| 类特定实例方法引用 | instance：：instanceMethodName  例如 this::equals  |



### for循环中的break和continue关键字

continue关键字可以用return来暂时替代，break的话就要根据情况来看，可以考虑使用anymatch,filter等api.