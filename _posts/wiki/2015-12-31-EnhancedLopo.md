---
layout: post
title: 逐渐挖掘Enhanced for Loop
category: wiki
description: 2015-12-31  J2SE 1.5提供了另一种形式的for循环。借助这种形式的for循环，可以用更简单地方式来遍历数组和Collection等类型的对象。
---


## 0.遍历数组和Collection对象循环的一般写法：

### 清单1：遍历数组的传统方式
/* 建立一个数组 */
int[] integers = {1, 2, 3, 4};
/* 开始遍历 */
for (int j = 0; j < integers.length; j++) {
    int i = integers[j];
    System.out.println(i);
}

### 清单2：遍历Collection对象的传统方式
/* 建立一个Collection */
String[] strings = {"A", "B", "C", "D"};
Collection stringList = java.util.Arrays.asList(strings);
/* 开始遍历 */
for (Iterator itr = stringList.iterator(); itr.hasNext();) {
    Object str = itr.next();
    System.out.println(str);
}

##遍历数组和Collection对象循环的for循环写法：
## 1. 第二种for循环
不严格的说，Java的第二种for循环基本是这样的格式：

for (循环变量类型 循环变量名称 : 要被遍历的对象) 循环体
借助这种语法，遍历一个数组的操作就可以采取这样的写法：

### 清单3：遍历数组的简单方式
/* 建立一个数组 */
int[] integers = {1, 2, 3, 4};
/* 开始遍历 */
for (int i : integers) {
    System.out.println(i);/* 依次输出“1”、“2”、“3”、“4” */
}
这里所用的for循环，会在编译期间被看成是这样的形式：

### 清单4：遍历数组的简单方式的等价代码
/* 建立一个数组 */
int[] integers = {1, 2, 3, 4};
/* 开始遍历 */
for (int 变量名甲 = 0; 变量名甲 < integers.length; 变量名甲++) {
    System.out.println(变量名甲);/* 依次输出“1”、“2”、“3”、“4” */
}
这里的“变量名甲”是一个由编译器自动生成的不会造成混乱的名字。

而遍历一个Collection的操作也就可以采用这样的写法：

### 清单5：遍历Collection的简单方式
/* 建立一个Collection */
String[] strings = {"A", "B", "C", "D"};
Collection list = java.util.Arrays.asList(strings);
/* 开始遍历 */
for (Object str : list) {
    System.out.println(str);/* 依次输出“A”、“B”、“C”、“D” */
}
这里所用的for循环，则会在编译期间被看成是这样的形式：

### 清单6：遍历Collection的简单方式的等价代码
/* 建立一个Collection */
String[] strings = {"A", "B", "C", "D"};
Collection stringList = java.util.Arrays.asList(strings);
/* 开始遍历 */
for (Iterator 变量名乙 = list.iterator(); 变量名乙.hasNext();) {
    System.out.println(变量名乙.next());/* 依次输出“A”、“B”、“C”、“D” */
}
这里的“变量名乙”也是一个由编译器自动生成的不会造成混乱的名字。

因为在编译期间，J2SE 1.5的编译器会把这种形式的for循环，看成是对应的传统形式，所以不必担心出现性能方面的问题。

不用“foreach”和“in”的原因
Java采用“for”（而不是意义更明确的“foreach”）来引导这种一般被叫做“for-each循环”的循环，并使用“:”（而不是意义更明确的“in”）来分割循环变量名称和要被遍历的对象。这样作的主要原因，是为了避免因为引入新的关键字，造成兼容性方面的问题——在Java语言中，不允许把关键字当作变量名来使用，虽然使用“foreach”这名字的情况并不是非常多，但是“in”却是一个经常用来表示输入流的名字（例如java.lang.System类里，就有一个名字叫做“in”的static属性，表示“标准输入流”）。

的确可以通过巧妙的设计语法，让关键字只在特定的上下文中有特殊的含义，来允许它们也作为普通的标识符来使用。不过这种会使语法变复杂的策略，并没有得到广泛的采用。

## 2. 防止在循环体里修改循环变量
在默认情况下，编译器是允许在第二种for循环的循环体里，对循环变量重新赋值的。不过，因为这种做法对循环体外面的情况丝毫没有影响，又容易造成理解代码时的困难，所以一般并不推荐使用。

Java提供了一种机制，可以在编译期间就把这样的操作封杀。具体的方法，是在循环变量类型前面加上一个“final”修饰符。这样一来，在循环体里对循环变量进行赋值，就会导致一个编译错误。借助这一机制，就可以有效的杜绝有意或无意的进行“在循环体里修改循环变量”的操作了。

### 清单7：禁止重新赋值
int[] integers = {1, 2, 3, 4};
for (final int i : integers) {
    i = i / 2; /* 编译时出错 */
}
注意，这只是禁止了对循环变量进行重新赋值。给循环变量的属性赋值，或者调用能让循环变量的内容变化的方法，是不被禁止的。

### 清单8：允许修改状态
Random[] randoms = new Random[]{new Random(1), new Random(2), new Random(3)};
for (final Random r : randoms) {
    r.setSeed(4);/* 将所有Random对象设成使用相同的种子 */
    System.out.println(r.nextLong());/* 种子相同，第一个结果也相同 */
}
## 3. 类型相容问题
为了保证循环变量能在每次循环开始的时候，都被安全的赋值，J2SE 1.5对循环变量的类型有一定的限制。这些限制之下，循环变量的类型可以有这样一些选择：

循环变量的类型可以和要被遍历的对象中的元素的类型相同。例如，用int型的循环变量来遍历一个int[]型的数组，用Object型的循环变量来遍历一个Collection等。
清单9：使用和要被遍历的对象中的元素相同类型的循环变量
int[] integers = {1, 2, 3, 4};
for (int i : integers) {
    System.out.println(i);/* 依次输出“1”、“2”、“3”、“4” */
}
循环变量的类型可以是要被遍历的对象中的元素的上级类型。例如，用int型的循环变量来遍历一个byte[]型的数组，用Object型的循环变量来遍历一个Collection<String>（全部元素都是String的Collection）等。
### 清单10：使用要被遍历的对象中的元素的上级类型的循环变量
String[] strings = {"A", "B", "C", "D"};
Collection<String> list = java.util.Arrays.asList(strings);
for (Object str : list) {
    System.out.println(str);/* 依次输出“A”、“B”、“C”、“D” */
}
循环变量的类型可以和要被遍历的对象中的元素的类型之间存在能自动转换的关系。J2SE 1.5中包含了“Autoboxing/Auto-Unboxing”的机制，允许编译器在必要的时候，自动在基本类型和它们的包裹类（Wrapper Classes）之间进行转换。因此，用Integer型的循环变量来遍历一个int[]型的数组，或者用byte型的循环变量来遍历一个Collection<Byte>，也是可行的。
### 清单11：使用能和要被遍历的对象中的元素的类型自动转换的类型的循环变量
int[] integers = {1, 2, 3, 4};
for (Integer i : integers) {
    System.out.println(i);/* 依次输出“1”、“2”、“3”、“4” */
}
注意，这里说的“元素的类型”，是由要被遍历的对象的决定的——如果它是一个Object[]型的数组，那么元素的类型就是Object，即使里面装的都是String对象也是如此。

可以限定元素类型的Collection
截至到J2SE 1.4为止，始终无法在Java程序里限定Collection中所能保存的对象的类型——它们全部被看成是最一般的Object对象。一直到J2SE 1.5中，引入了“泛型（Generics）”机制之后，这个问题才得到了解决。现在可以用Collection<T>来表示全部元素类型都是T的Collection，如Collection<String>、Collection<Integer>等。不过这里的T不能是一个简单类型，象Collection<int>之类的写法是不被认可的。

## 4. 被这样遍历的前提
有两种类型的对象可以通过这种方法来遍历——数组和实现了java.lang.Iterable接口的类的实例。试图将结果是其它类型的表达式放在这个位置上，只会在编译时导致一个提示信息是“foreach not applicable to expression type”的问题。

java.lang.Iterable接口中定义的方法只有一个：

iterator()
返回一个实现了java.util.Iterator接口的对象
而java.util.Iterator接口中，则定义了这样三个方法：

hasNext()
返回是否还有没被访问过的对象
next()
返回下一个没被访问过的对象
remove()
把最近一次由next()返回的对象从被遍历的对象里移除。这是一个可选的操作，如果不打算提供这个功能，在实现的时候抛出一个UnsupportedOperationException即可。因为在整个循环的过程中，这个方法根本没有机会被调用，所以是否提供这个功能，在这里没有影响。
借助这两个接口，就可以自行实现能被这样遍历的类了。

### 清单12：一个能取出10个Object元素的类
import java.util.*;
class TenObjects implements Iterable {
    public Iterator iterator() {
        return new Iterator() {
            private int count = 0;
            public boolean hasNext() {
                return (count < 10);
            }
            public Object next() {
                return new Integer(count++);
            }
            public void remove() {
                throw new UnsupportedOperationException();
            }
        };
    }
    public static void main(String[] args) 
    {
        TenObjects objects = new TenObjects();
        for (Object i : objects)
        {
            System.out.println(i);/* 依次输出从“0"到“9”的十个整数 */
        }
    }
}
Collection的资格问题
在J2SE 1.5的API中，所有能被这样遍历的对象的类型都是java.util.Collection的子类型，看上去很象java.util.Collection获得了编译器的特殊对待。

不过，造成这种现象的实际原因，是在J2SE 1.5中，java.util.Collection被定义成了java.lang.Iterable的子接口。编译器并没有给Collection什么特别的关照。

从理论上说，完全可以制造出一些拒不实现Collection接口的容器类，而且能让它们和Collection一样被用这种方法遍历。不过这样的容器类，可能会因为存在兼容性的问题，而得不到广泛的流传。

若干方法的命名问题
在java.lang.Iterable接口中，使用iterator()，而不是getIterator()；而java.util.Iterator接口中，也使用hasNext()和next()，而不是hasNextElement()和getNextElement()。造成这种现象的原因，是Java Collections Framework的设计者们，认为这些方法往往会被频繁的调用（每每还会挤到一行），所以用短一点的名字更为合适。

## 5. 加入更精确的类型控制
如果在遍历自定义的可遍历对象的时候，想要循环变量能使用比Object更精确的类型，就需要在实现java.lang.Iterable接口和java.util.Iterator接口的时候，借助J2SE 1.5中的泛型机制，来作一些类型指派的工作。

如果想要使循环变量的类型为T，那么指派工作的内容是：

在所有要出现java.lang.Iterable的地方，都写成“Iterable<T>”。
在所有出现java.util.Iterator的地方，都写成“Iterator<T>”。
在实现java.util.Iterator的接口的时候，用T作为next()方法的返回值类型。
注意，这里的T不能是一个基本类型。如果打算用基本类型作为循环变量，那么得用它们的包裹类来代替这里的T，然后借助Auto-Unboxing机制，来近似的达到目的。

### 清单13：用int型的循环变量来遍历一个能取出10个Integer元素的类
import java.util.*;
public class TenIntegers implements Iterable<Integer> {
    public Iterator<Integer> iterator() {
        return new Iterator<Integer>() {
                private int count = 0;
                public boolean hasNext() {
                    return (count < 10);
                }
                public Integer next() {
                    return Integer.valueOf(count++);
                }
                public void remove() {
                    throw new UnsupportedOperationException();
                }
            };
    }
    public static void main(String[] args) 
    {
        TenIntegers integers = new TenIntegers();
        for (int i : integers)
        {
            System.out.println(i);/* 依次输出从“0"到“9”的十个整数 */
        }
    }
}
另外，一个类只能实现一次java.lang.Iterable接口，即使在后面的尖括号里使用不同的类型。类似“class A implements Iterable<String>, Iterable<Integer>”的写法，是不能通过编译的。所以，没有办法让一个可遍历对象能在这样遍历时，既可以使用Integer，又可以使用String来作为循环变量的类型（当然，把它们换成另外两种没有继承和自动转化关系的类也一样行不通）。

## 6. 归纳总结
借助J2SE 1.5中引入的第二种for循环，可以用一种更简单地方式来完成遍历。能用这种方法遍历的对象的类型，可以是数组、Collection或者任何其它实现了java.lang.Iterable接口的类。通过跟同样是在J2SE 1.5中引入的泛型机制配合使用，可以精确的控制能采用的循环变量的类型。而且，因为这么编写的代码，会在编译期间被自动当成是和传统写法相同的形式，所以不必担心要额外付出性能方面的代价。

参考资源
可以通过Sun的Java Technology页面找到下载J2SE 1.5的SDK及其文档的链接，目前最新的版本是J2SDK 1.5 Beta 2。注意在使用这一版本的javac的时候，要加上“-source 1.5”作为参数，才能编译使用了J2SE 1.5中新增语言特性的源代码。
John Zukowski在《驯服 Tiger：Tiger 预览版现已推出》一文中，介绍了如何开始使用J2SDK 1.5的基础知识。不过因为这篇文章是依照J2SDK 1.5 Alpha版的状况所写，所以里面提到的一些细节（如下载地址和默认安装路径）已经发生了变化。
《JSR 201: Extending the Java Programming Language with Enumerations, Autoboxing, Enhanced for loops and Static Import》定义了很多J2SE 1.5中的新语言特性，包括了因为拥有了第二种形式而“增强了的for循环（Enhanced for Loop）”。
《Java Collections API Design FAQ》解释了Java Collections Framework为什么被设计成了现在这个样子，其中谈到了为什么java.util.Iterator接口中的方法要那样命名。
《JSR 14: Adding Generics to the Java Programming Language》定义了J2SE 1.5中的泛型机制。
Gilad Bracha在《Generics in the Java Programming Language》一文中，细致的介绍了J2SE 1.5中的泛型机制的使用方法和各种限制。
Calvin Austin在《J2SE 1.5 in a Nutshell》一文中，对J2SE 1.5中的各种新特性，进行了全面而概括的介绍。

 <div id="disqus_thread"></div>
<script type="text/javascript">
    /* * * CONFIGURATION VARIABLES * * */
    var disqus_shortname = 'x-flowing';
    
    /* * * DON'T EDIT BELOW THIS LINE * * */
    (function() {
        var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
        dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
        (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
    })();
    
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>

