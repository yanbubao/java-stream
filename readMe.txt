Java8新特性笔记

高阶函数：如果一个函数接受一个函数作为参数，或者将一个函数作为返回值，那么该函数就叫做高阶函数。

一、lambda:用于表示匿名函数或闭包的一种运算符。
*** 传递行为，而不仅仅是值！

方法引用：实际上是lambda表达式的一种语法糖！是lambda的一种具象模式！

可以将方法引用看成一个函数指针，方法引用共分为四类：
01.类::静态方法
02.对象引用::实例方法
02.类::实例方法
03.类::构造方法

使用方法引用这块语法糖的场景：
你的lambda中只有一行，且这一行代码恰好存在已有的代替者，那么就可以嚼这块糖啦！

注意：
01.lambda表达式对局部变量的限制：
lambda没有对实例变量和静态变量的获取进行限制！
但是对于局部变量必须显式的声明为final或事实上是final！（事实上final意思就是赋值之后没改过哈）
eg.
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
portNumber = 31337;  //错误：Lambda表达式引用的局部变量必须是最终的（final）或事实上最终的
原因：
a.实例变量和局部变量背后实现有一个关键不的不同！那就是实例变量是存储在堆中，而局部变量是存储在栈中！
如果lambda直接访问局部变量，而且lambda是在一个线程A中使用的，则该使用lambda的A线程可能在分配该局部变量的B线程将局部变量回收后在触发
访问局部变量。基于这个原因，Java在使用局部变量时实际上是在访问他们的副本，而不是访问的原始变量！但如果局部变量仅仅赋值过一次就没啥区别了。
故此有了这个限制！
b.这一限制不鼓励你使用改变外部变量的典型命令式编程模式（我这种模式会阻碍很容易做到的并行处理）!
**** 闭包 ****
你可能已经听说过闭包（closure，不要和Clojure编程语言混淆）这个词，你可能会想Lambda是否满足闭包的定义。用科学的说法来说，
闭包就是一个函数的实例，且它可以无限制地访问那个函数的非本地变量。例如，闭包可以作为参数传递给另一个函数。它也可以访问和修改其作用域之外的变量。
现在，Java 8的Lambda和匿名类可以做类似于闭包的事情：它们可以作为参数传递给方法，并且可以访问其作用域之外的变量。但有一个限制：
它们不能修改定义Lambda的方法的局部变量的内容。这些变量必须是隐式最终的。可以认为Lambda是对值封闭，而不是对变量封闭。
如前所述，这种限制存在的原因在于局部变量保存在栈上，并且隐式表示它们仅限于其所在线程。如果允许捕获可改变的局部变量，就会引发造成线程
不安全的新的可能性，而这是我们不想看到的（实例变量可以，因为它们保存在堆中，而堆是在线程之间共享的）。


二、函数式接口：java.lang.FunctionalInterface
条件：
01.有且只有一个抽象方法的接口
02.有@FunctionInterface注解
03.没有@FunctionInterface注解，但有且只有一个抽象方法的接口

04.如果接口中含有覆写了java.lang.Object中的方法，那么编译器不会认为此方法为函数式接口中的一个方法，因为所有实现该函数式接口的类都继承自Object类。

实现：可以使用lambda表达式、方法引用、构造方法引用来创造函数式接口的实例。

注意：在将函数作为一等公民的编程语言中，lambda表达式是类型是函数，如python等语言。
但是！在Java中lambda的类型的对象！必须依附于一类特别的对象类型 —— 函数式接口。

java.util.function.* 下为Java8提供的所有函数式接口，BiFunction: Bidirectional（双向的)等等！


三、java.util.Optional<T>

Optional是一个容器对象，可能包含null！

推荐使用Optional.ifPresent进行函数式编写程序，避免编写 if(null == xx) else .. 避免NPE的优雅写法...

注意：Optional没有实现序列化接口！因此不要用Optional最为成员变量或函数参数，一般只作为函数返回值使用！


四、stream
01.基本概念：
两组纬度，串行流和并行流、中间流和节点流。
流本质时函数式的，对流的操作会产生一个新的结果，并不会改变修改底层的数据源！

02.流由三部分构成：源、0个或多个中间操作、终止操作！

03.流操作分为：
惰性求值lazy：只在终止操作时流被初始化！
及早求值：stream.xx.yy.count()


04.分析内部迭代和外部迭代

a.stream和迭代器不同的是，stream可以并行化操作，而迭代器只能命令式的串行化操作！
b.当使用串行化操作时，每个item被读完后再读下一个item！
c.使用并行化操作时，数据会被分为多个数据段，其中每一个段都在不同的线程中处理，最后将结果一起输出！
d.stream的并行化操作依赖与JDK7中的Fork/Join框架！

对于外部迭代呢就是传统的for、for增强foreach，需要我们自己书写内部的处理逻辑，并顺序执行这些逻辑！

内部迭代是将源数据转换为流（不严谨的化可以说把源数据丢在流中），流会现将我们编写的处理逻辑（也就是中间操作们）先存储起来，等出发最终操作时
一并执行中间操作！而且最终操作是流内部也会对中间操作进行优化，如可以并行化处理！

从另一个角度形象的比喻一哈，使用流处理迭代就像流内部已经把处理逻辑的架子搭好了，使用者按需填空就完了。而传统的外部迭代就像写作为，
使用者需编写全部内容！

流和集合的本质差别：集合关注的是数据和数据存储本身！而流关注的是对数据的计算！
流和迭代气iterate相似的一点是都无法重复使用或消费！

举个栗子🌰：
list.stream().filter(a).map.(b).filter(c).foreach(sout);
上述伪代码中不是循环执行a，再操作b，再循环执行c！而是最终出发foreach最终操作时只循环一次并执行所有中间操作！


五、Collector

01.Collector接口作为stream.collect(Collector)方法的参数

JavaDoc中描述到Collector：
是一个可变的汇聚操作，它会将输入元素放置到一个可变的结果容器中！
当所有的输入元素都处理完毕之后，将所有操作累积之后的结果转换到一个最终的表示！（这是个可选操作！）
并且支持串行和并行两种执行方式！

并行不一定比串行效率高哦，例如4个线程在双核CPU上运行，会耗费线程切换的时间！

Collector接口中包含了四个函数：

Supplier Collector#supplier     供应者：创建并返回一个新的结果容器！

BiConsumer Collector#accumulator  累加器：将新的数据元素（流中元素）合并到结果容器！supplier所创建的结果容器！

BinaryOperator Collector#combiner     组合器：将两个结果容器合并为一个！ 与并发相关！

Function Collector#finisher     将中间累积类型转换为最终结果类型！


02.Collectors类提供了关于Collector的常见汇聚实现！Collectors本身是一个工厂！






