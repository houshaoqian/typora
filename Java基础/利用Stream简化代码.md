当前项目的JDK版本升级为1.8已有一段时间。作为“程序猿”，应当追逐技术潮流，充分利用技术升级带来的便利性。项目中往往充斥着各种各样的集合遍历操作，“看起来难受，写起来手酸”。利用JDK1.8新增的Stream抽象，完美解决了以上问题，极大程度上提升了开发效率。对Stream的操作可以分为以下三个阶段：
## 一、创建Stream
此处的Stream区别于IO中的流。Stream只是模拟IO流的操作方式，它没有输入流和输出流的区分。只能通过容器类型的数据结构去创建。Collection类提供了两个获取流的方法stream()和parallelStream()。两者的区别在于前者是串行流后者是并行流。在日常开发中，对Stream的操作往往开始于.stream()。
```java
// Collection集合类创建Stream
List<Integer> list = Arrays.asList(1, 2, 3);
list.stream();

// Map类型需要将 键/值 转化为集合类才能创建Steam
// 将值转化为Stream
mp.values().stream();
// 将键转化为Stream
mp.keySet().stream();
```
## 二、操作Stream
操作Stream大致分为以下3类，筛选、映射和排序。

1.筛选<br />筛选包括去重和条件筛选。可以直接使用stream.distinct()进行去重，去重的依据是equals()和hasCode()。在实际场景中，条件筛选是最常见的操作。例如，在多规格商品中，去除未参加活动的规格。
```java
// 去重 打印结果为 1, 2, 3, 4
Arrays.asList(1, 2, 3, 4, 4).stream().distinct().forEach(System.out::println);

// 根据条件过滤, filter的参数为符合Predicate类型函数式接口即可
Arrays.asList(1, 2, 3, 4, 4).stream().filter(e->e>2).forEach(System.out::println);
```
2.映射<br />映射可以将一种数据类型转化成其他数据类型，常用的业务场景有ID和名称的相互转换，一对一数据关系的转换等。
```java
// 将ID转化为对应的名称 ,getNameById为普通的转化方法，有返回值，参数为集合中的泛型类即可
Arrays.asList(1, 2, 3, 4, 4).stream().map(e->getNameById(e)).forEach(System.out::println);
```
3.排序<br />按照一定规则将数据进行排序。通过stream.sorted()方法进行排序，参数为符合Comparator的函数式接口
```java
List<Item> items = new ArrayList<>();
items.add(new Item(11, "毛笔", true));
items.add(new Item(12, "手机", false));
items.add(new Item(13, "电脑", true));
// Item是实现了Comparable接口
items.stream().sorted((e1, e2)->(e2.compareTo(e1))).forEach(System.out::println);
```

## 三、收集Stream
经过以上步骤后，生成的Stream是我们最终需要的数据，但是Stream不能被我们直接被使用，需要转换成相应的数据类型。收集Stream也可分为以下几种类型：<br />1．统计类<br />统计Stream中数据的个数，stream.count()，取Stream数据中的最大值stream.max()或最小值stream.min()。例如：取出数据中Id最大的那一条数据
```java
final ArrayList<People> list = new ArrayList();
list.add(new People("alan", 18));
list.add(new People("smith", 20));
list.add(new People("tom", 21));
// 统计Stream中元素的个数
long count = list.stream().count();
System.out.println("count=" + count);

// 计算Stream中元素的最大值 比较逻辑通过函数式接口Comparator实现
Optional<People> max = list.stream().max(Comparator.comparingInt(People::getAge));
System.out.println(max);
```
2．规约类<br />将Stream数据中的所有项的某个字段合并到一起，例如，计算订单中所有商品的总价格。
```java
final ArrayList<Order> list2 = new ArrayList();
list2.add(new Order(12.0, "茶杯"));
list2.add(new Order(25.0, "奶茶"));
list2.add(new Order(3.0, "棒冰"));
Optional<Order> reduce = list2.stream().reduce((e1, e2) -> new Order(e1.getPrice() + e2.getPrice(), e1.getItemid() + "-" + e2.getItemid()));
// 输出结果为：{"price":40.0,"itemid":"茶杯-奶茶-棒冰"}
System.out.println(reduce.get());
```

3．汇总<br />汇总是最常用的操作，是将Stream转化成我们需要的数据格式的过程。<br />3.1转Map类型，用对象中的某个属性为key值，value为对象本身，转化为Map。
```java
// 以itemid属性为key，order本身为值，转化为Map
Map<String, Order> map = list2.stream().collect(Collectors.toMap(e -> e.getItemid(), e -> e));
System.out.println(map);
```
3.2转集合类型。
```java
// 转化为集合类-List
List<Order> orders = list2.stream().collect(Collectors.toList());
// 转化为集合类-Set
Set<Order> sets = list2.stream().collect(Collectors.toSet());
```

原本需要十几行甚至几十行的代码，通过使用Stream对数据进行处理，现在仅需要一行就能完成，难道“它”不香吗！
