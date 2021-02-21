函数式接口

- 只带有一个未实现的方法

系统自带的四个重要的函数式接口（位于java.util.function包中）

![image-20210217220207263](https://image-1301164990.cos.ap-shanghai.myqcloud.com/img/20210217220207.png)

方法引用（可以用于替换Lamda表达式）

- 静态方法Class::staticMethod，如Math::abs

- 实例方法Class::instanceMethod，如String::compareToIgnoreCase方法

  也可以object::instanceMethod，System.out::println方法

  - 支持this::instanceMethod调用
  - 支持super::instanceMethod调用

- Class::new，调用某类的构造函数，支持单个对象构建

- Class[]::new，调用某类的构造函数，支持数组对象的构建

特殊情况

![image-20210219202128093](https://image-1301164990.cos.ap-shanghai.myqcloud.com/img/20210219202128.png)

![image-20210219202644115](https://image-1301164990.cos.ap-shanghai.myqcloud.com/img/20210219202644.png)

 