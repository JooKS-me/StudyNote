## Stream的工作流程

- 创建一个流
- 指定将流转换为其他流的中间操作（惰性操作，当开始计算的时候，转换才会产生）
- 应用终止操作，产生结果。此后，这个流就不能再用了。

## 流的创建

1. Collection接口的stream方法

   ```java
   Stream<String> as = new ArrayList<String>().stream();
   Stream<String> hs = new HashSet<String>().stream();
   ```

2. Arrays.stream

   ```java
   Stream<String> b1 = Arrays.stream("a,b,c,d,e".split(","),3,5);
   ```

3. 利用Stream类进行转化

   - of方法

     ```java
     Stream<Integer> c1 = Stream.of(new Integer[5]);
     Stream<String> c2 = Stream.of("a,b,c".split(","));
     Stream<String> c3 = Stream.of("a", "b", "c");
     ```

   - empty方法，产生空流

     ```java
     Stream<String> d1 = Stream.empty();
     ```

   - generate方法，接收一个Lambda表达式（实现Supplier接口），有无数个元素

     ```java
     Stream<String> e1 = Stream.generate(()->"hello");
     Stream<Double> e2 = Stream.generate(Math::random);
     ```

   - iterate方法，接收一个种子，和一个Lambda表达式

     ```java
     Stream<BigInteger> e3 = Stream.iterate(BigInteger.zero, n->n.add(BigInteger.ONE));
     ```

4. 基本类型流（只有三种）

   - IntStream, LongStream, DoubleStream

   ```java
   IntStream s1 = IntStream.of(1,2,3,4,5);
   s1 = Arrays.stream(new int[] {1,2,3});
   s1 = IntStream.generate(()->(int)(Math.random*100));
   s1 = IntStream.range(1,5); //1,2,3,4
   s1 = IntStream.rangeClosed(1,5); //1,2,3,4,5
   
   // 基本类型流转对象流
   Stream<Integer> s2 = s1.boxed();
   
   // 对象流转基本类型
   IntStream s3 = s2.mapToInt(Integer::intValue);
   ```

5. 并行流

   使得所有中间的转换操作都将被并行化

   - Collections.parallelStream()将任何集合转为并行流
   - Stream.parallel()方法，产生一个并行流

   - **需要保证传给并行流的操作不存在竞争！**

6. 其他

   - Files.lines方法（NIO包）

     ```java
     Stream<String> contents = Files.lines(Path.get("abc.txt"));
     ```

   - Pattern的splitAsStream方法

     ```java
     Stream<String> words = Pattern.compile(",").splitAsStream("a,b,c");
     ```

## 流的转换

1. 过滤，去重

   - filter(predicate<? super T> predicate)

     接收一Lambda表达式，对每个元素进行判定，符合条件留下

   - distinct()

     对流的元素进行过滤，去除重复，只留下不重复的元素

     先调用hashCode方法，再调用equals方法，与hashSet相同。

2. 排序

   - sorted()

   - 可提供Comparator，对流元素进行排序

     ```java
     s3.sorted(Comparator.comparing(String::length));
     ```

   - 对于自定义对象，可以实现Comparator接口，重写compareTo方法

3. 转化

   - map(Function<? super T, ? extends R> mapper)

     **会保持括号中原来的结构**

     - 可利用方法引用对流的每个元素进行函数计算

     ```java
     s1.map(Math::abs);
     ```

     - 也可利用Lambda表达式对流的每个元素进行函数计算

   - flatMap()

     **会把所有计算结果合并成一个流**

4. 抽取/跳过/连接

   - 抽取limit()
   - 跳过skip()
   - 连接concat()，连接两个流

5. 其他

   - 额外调试peek

     保持原来的流不变，但是能额外执行括号内的函数

## Optional类型

- Optional\<T>创建
  -  of方法
  - empty方法
  - ofNullable方法，对于对象可能为null的情况下，安全创建

- Optional\<T>使用
  - get，获取值，不安全
  - orElse，获取值，若为null，采用替代物的值
  - orElseGet，获取值，若为null，采用Lambda表达式返回
  - orElseThrow，获取值，若为null，抛出异常
  - ifPresent，判断是否为空，不为空，返回true
  - isPresent(Consumer)，判断是否为空，不为空则进行Consumer操作，为空则不做任何处理
  - map(Function)，将值传递给Function函数进行计算。如果为空，则不计算。

## 流的计算

1. 简单约简（聚合函数）
   - count，计数
   - max(Comparator)，最大值，需要比较器
   - min(Comparator)，最小值，需要比较器
   - findFirst()，找到第一个元素
   - findAny()，找到任意一个元素
   - anyMatch(Predicate)，如果有任意一个元素满足Predicate，返回true
   - allMatch(Predicate)，如果所有元素满足Predicate，返回true
   - noneMatch(Predicate)，如果没有任何元素满足Predicate，返回true
2. 自定义约简
   - Reduce，传递一个二元函数 BinaryOperator，对流元素进行计算
3. 查看/遍历元素
   - iterator()，遍历元素
   - forEach(Consumer)，应用一个函数到每个元素上
4. 存放到数据结构中
   - toArray()，将结果转为数组
   - collect(Collectors.toList())，将结果转为List
   - collect(Collecttors.toSet())
   - collect(Collecttors.toMap())
   - collect(Collecttors.joining())，将结果连接起来

## 注意事项

- 一个流，一次只能一个用途，不能多个用途，用了不能再用

- 避免创建无限流

  ```java
  IntStream.iterate(0, i->(i+1)%2)
    .distinct()    // 这个地方distinct读取无限流，会一直读，一直等待结束
    .limit(10)
    .forEach(System.out::println);
  ```

- 注意操作的顺序

- 谨慎使用并行流

  - 底层是Fork-Join Pool，处理计算密集型任务，会占用所有CPU资源
  - 数据量过小就不别用
  - 数据结构不容易分解的时候不用，如LinkedList等
  - 数据频繁拆箱装箱不用
  - 涉及findFirst或者limit的时候不用

- Stream VS Collection

  - Stream和Collection两者可以相互转化
  - 如果数据可能无限，用Stream
  - 如果数据很大很大，用Stream
  - 如果调用者将使用查找/过滤/聚合等操作，用Stream
  - 当调用者使用过程中，发生数据改变，而调用者需要对数据一致性有较高要求，用Collection
  - https://stackoverflow.com/questions/24676877/should-i-return-a-collection-or-a-stream/24679745%2324679745

