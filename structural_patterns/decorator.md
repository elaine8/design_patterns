

 装饰模式
====================

模式动机
--------------------
一般有两种方式可以实现给一个类或对象增加行为：

- 继承机制，使用继承机制是给现有类添加功能的一种有效途径，通过继承一个现有类可以使得子类在拥有自身方法的同时还拥有父类的方法。但是这种方法是静态的，用户不能控制增加行为的方式和时机。
- 关联机制，即将一个类的对象嵌入另一个对象中，由另一个对象来决定是否调用嵌入对象的行为以便扩展自己的行为，我们称这个嵌入的对象为装饰器(Decorator)

装饰模式以对客户透明的方式动态地给一个对象附加上更多的责任，换言之，客户端并不会觉得对象在装饰前和装饰后有什么不同。装饰模式可以在不需要创造更多子类的情况下，将对象的功能加以扩展。这就是装饰模式的模式动机。


模式定义
--------------------
装饰模式(Decorator Pattern) ：动态地给一个对象增加一些额外的职责(Responsibility)，就增加对象功能来说，装饰模式比生成子类实现更为灵活。其别名也可以称为包装器(Wrapper)，与适配器模式的别名相同，但它们适用于不同的场合。根据翻译的不同，装饰模式也有人称之为“油漆工模式”，它是一种对象结构型模式。


模式结构
--------------------
装饰模式包含如下角色：

- Component: 抽象构件
- ConcreteComponent: 具体构件
- Decorator: 抽象装饰类
- ConcreteDecorator: 具体装饰类


![UML 模型图](/_static/Decorator.jpg)


时序图
--------------------
![时序图](/_static/seq_Decorator.jpg)

模式分析
--------------------
- 与继承关系相比，关联关系的主要优势在于不会破坏类的封装性，而且继承是一种耦合度较大的静态关系，无法在程序运行时动态扩展。在软件开发阶段，关联关系虽然不会比继承关系减少编码量，但是到了软件维护阶段，由于关联关系使系统具有较好的松耦合性，因此使得系统更加容易维护。当然，关联关系的缺点是比继承关系要创建更多的对象。
- 使用装饰模式来实现扩展比继承更加灵活，它以对客户透明的方式动态地给一个对象附加更多的责任。装饰模式可以在不需要创造更多子类的情况下，将对象的功能加以扩展。


典型案例
--------------------
# 实例：JAVA IO库

Java IO库的设计就是Decorator模式的典范。在IO处理中，Java将数据抽象为流（Stream）。在IO库中，最基本的是InputStream和OutputStream两个分别处理输出和输入的对象（为了叙述简便起见，这儿只涉及字节流，字符流和其完全相似），但是在InputStream和OutputStream中只提供了最简单的流处理方法，只能读入/写出字符，没有缓冲处理，无法处理文件，等等。它们只是提供了最纯粹的抽象，最简单的功能。

![案例图](/_static/Decorator_eg.jpg)

<p>首先来看一段用来创建IO流的代码，以下是代码片段：<p>
<pre><code>try {
    DataOutputStream out = new DataOutputStream(new FileOutputStream("test.txt"));
} catch (FileNotFoundException e) {
    e.printStackTrace();
}
</code></pre>

这段代码对于使用过JAVA输入输出流的人来说再熟悉不过了，我们使用DataOutputStream封装了一个FileOutputStream。这是一个典型的Decorator模式的使用，FileOutputStream相当于Component，DataOutputStream就是一个Decorator。由于FileOutputStream和DataOutputStream有公共的父类OutputStream，因此对对象的装饰对于用户来说几乎是透明的。
　　下面就来看看OutputStream及其子类是如何构成Decorator模式的。
　　Component角色：OutputStream
　　ConcreteComponent角色：ByteArrayOutputStream, FileOutputStream, PipedOutputStream, ObjectOutputStream
　　Decorator角色：FilterOutputStream
　　ConcreteDecorator角色：BufferedOutputStream, DataOutputStream, PrintStream
　　顶层的OutputStream是一个抽象类，它是所有输出流的公共父类，其源代码片段如下：
public abstract class OutputStream implements Closeable, Flushable {

    public abstract void write(int b) throws IOException;
    //...
}
　　它定义了write(int b)的抽象方法。这相当于Decorator模式中的Component类。ByteArrayOutputStream，FileOutputStream ，PipedOutputStream 和ObjectOutputStream类都直接从OutputStream继承，以FileOutputStream为例，以下是代码片段：
public class FilterOutputStream extends OutputStream {
    protected OutputStream out;

    public FilterOutputStream(OutputStream out) {
        this.out = out;
    }

    @Override
    public void write(int b) throws IOException {
        out.write(b);
    }

    //...
}
　它实现了OutputStream中的write(int b)方法，因此我们可以用来创建输出流的对象，并完成特定格式的输出。它相当于Decorator模式中的ConcreteComponent类。
接着来看一下FilterOutputStream，代码片段如下
public class FilterOutputStream extends OutputStream {
    protected OutputStream out;

    public FilterOutputStream(OutputStream out) {
        this.out = out;
    }

    @Override
    public void write(int b) throws IOException {
        out.write(b);
    }

    //...
}
　同样，它也是从OutputStream继承。但是，它的构造函数很特别，需要传递一个OutputStream的引用给它，并且它将保存对此对象的引用。而如果没有具体的OutputStream对象存在，我们将无法创建FilterOutputStream。由于out既可以是指向FilterOutputStream类型的引用，也可以是指向ByteArrayOutputStream等具体输出流类的引用，因此使用多层嵌套的方式，我们可以为ByteArrayOutputStream添加多种装饰。这个FilterOutputStream类相当于Decorator模式中的Decorator类，它的write(int b)方法只是将调用转发给了传入的流的write(int b)方法，而没有做更多的处理，因此它本质上没有对流进行装饰，所以继承它的子类必须覆盖此方法，以达到装饰的目的。
　　BufferedOutputStream 和 DataOutputStream是FilterOutputStream的两个子类，它们相当于Decorator模式中的ConcreteDecorator，并对传入的输出流做了不同的装饰。以BufferedOutputStream类为例，以下是代码片段：
public class BufferedOutputStream extends FilterOutputStream {
    //...

    private void flushBuffer() throws IOException {
        if (count > 0) {
            out.write(buf, 0, count);
            count = 0;
        }
    }

    @Override
    public synchronized void write(int b) throws IOException {

        if (count >= buf.length) {
            flushBuffer();
        }
        buf[count++] = (byte) b;
    }

    //...
}
这个类提供了一个缓存机制，等到缓存的容量达到一定的字节数时才写入输出流。首先它继承了FilterOutputStream，并且覆盖了父类的write(int b)方法，在调用输出流写出数据前都会检查缓存是否已满，如果未满，则不写。这样就实现了对输出流对象动态的添加新功能的目的。了解了OutputStream及其子类的结构原理后，我们可以利用Decorator模式为IO写一个新的输出流，来添加新的功能。这里给出一个新的输出流的例子，它将过滤待输出语句中的空格符号。比如需要输出"java io OutputStream"，则过滤后的输出为"javaioOutputStream"。以下为SkipSpaceOutputStream类的代码：

import java.io.InputStream;
import java.io.OutputStream;
import java.io.FilterOutputStream;
import java.io.BufferedInputStream;
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;


class SkipSpaceOutputStream extends FilterOutputStream {

    public SkipSpaceOutputStream(OutputStream out) {
        super(out);
    }

    //重写父类的方法，添加新的功能
    @Override
    public void write(int b) throws IOException {
        if (b != ' ') {
            super.write(b);
        }
    }
}

/**
 * Test the SkipSpaceOutputStream.
 *
 * @author
 */
public class SkipSpaceTest {

    public static void main(String[] args) {
        byte[] buffer = new byte[1024];
        InputStream in = new BufferedInputStream(new DataInputStream(System.in));
        OutputStream out = new SkipSpaceOutputStream(new DataOutputStream(System.out));
        try {
            System.out.print("Please input your words: ");
            int n = in.read(buffer, 0, buffer.length);
            for (int i = 0; i < n;  i++){
                out.write(buffer[i]);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
　它从FilterOutputStream继承，并且重写了它的write(int b)方法。在write(int b)方法中首先对输入字符进行了检查，如果不是空格，则输出。输出结果：
Please input your words: Jack Zhou
JackZhou

优点
--------------------
装饰模式的优点:

- 装饰模式与继承关系的目的都是要扩展对象的功能，但是装饰模式可以提供比继承更多的灵活性。
- 可以通过一种动态的方式来扩展一个对象的功能，通过配置文件可以在运行时选择不同的装饰器，从而实现不同的行为。
- 通过使用不同的具体装饰类以及这些装饰类的排列组合，可以创造出很多不同行为的组合。可以使用多个具体装饰类来装饰同一对象，得到功能更为强大的对象。
- 具体构件类与具体装饰类可以独立变化，用户可以根据需要增加新的具体构件类和具体装饰类，在使用时再对其进行组合，原有代码无须改变，符合“开闭原则”


缺点
--------------------
装饰模式的缺点:

- 使用装饰模式进行系统设计时将产生很多小对象，这些对象的区别在于它们之间相互连接的方式有所不同，而不是它们的类或者属性值有所不同，同时还将产生很多具体装饰类。这些装饰类和小对象的产生将增加系统的复杂度，加大学习与理解的难度。
- 这种比继承更加灵活机动的特性，也同时意味着装饰模式比继承更加易于出错，排错也很困难，对于多次装饰的对象，调试时寻找错误可能需要逐级排查，较为烦琐。


适用环境
--------------------
在以下情况下可以使用装饰模式：

- 在不影响其他对象的情况下，以动态、透明的方式给单个对象添加职责。
- 需要动态地给一个对象增加功能，这些功能也可以动态地被撤销。
- 当不能采用继承的方式对系统进行扩充或者采用继承不利于系统扩展和维护时。不能采用继承的情况主要有两类：第一类是系统中存在大量独立的扩展，为支持每一种组合将产生大量的子类，使得子类数目呈爆炸性增长；第二类是因为类定义不能继承（如final类）.


模式应用
--------------------

模式扩展
--------------------
装饰模式的简化-需要注意的问题:

- 一个装饰类的接口必须与被装饰类的接口保持相同，对于客户端来说无论是装饰之前的对象还是装饰之后的对象都可以一致对待。
- 尽量保持具体构件类Component作为一个“轻”类，也就是说不要把太多的逻辑和状态放在具体构件类中，可以通过装饰类
对其进行扩展。
- 如果只有一个具体构件类而没有抽象构件类，那么抽象装饰类可以作为具体构件类的直接子类。 

总结
--------------------
- 装饰模式用于动态地给一个对象增加一些额外的职责，就增加对象功 能来说，装饰模式比生成子类实现更为灵活。它是一种对象结构型模 式。
- 装饰模式包含四个角色：抽象构件定义了对象的接口，可以给这些对 象动态增加职责（方法）；具体构件定义了具体的构件对象，实现了 在抽象构件中声明的方法，装饰器可以给它增加额外的职责（方法）； 抽象装饰类是抽象构件类的子类，用于给具体构件增加职责，但是具 体职责在其子类中实现；具体装饰类是抽象装饰类的子类，负责向构 件添加新的职责。
- 使用装饰模式来实现扩展比继承更加灵活，它以对客户透明的方式动 态地给一个对象附加更多的责任。装饰模式可以在不需要创造更多子 类的情况下，将对象的功能加以扩展。
- 装饰模式的主要优点在于可以提供比继承更多的灵活性，可以通过一种动态的 方式来扩展一个对象的功能，并通过使用不同的具体装饰类以及这些装饰类的 排列组合，可以创造出很多不同行为的组合，而且具体构件类与具体装饰类可 以独立变化，用户可以根据需要增加新的具体构件类和具体装饰类；其主要缺 点在于使用装饰模式进行系统设计时将产生很多小对象，而且装饰模式比继承 更加易于出错，排错也很困难，对于多次装饰的对象，调试时寻找错误可能需 要逐级排查，较为烦琐。
- 装饰模式适用情况包括：在不影响其他对象的情况下，以动态、透明的方式给 单个对象添加职责；需要动态地给一个对象增加功能，这些功能也可以动态地 被撤销；当不能采用继承的方式对系统进行扩充或者采用继承不利于系统扩展 和维护时。
- 装饰模式可分为透明装饰模式和半透明装饰模式：在透明装饰模式中，要求客 户端完全针对抽象编程，装饰模式的透明性要求客户端程序不应该声明具体构 件类型和具体装饰类型，而应该全部声明为抽象构件类型；半透明装饰模式允 许用户在客户端声明具体装饰者类型的对象，调用在具体装饰者中新增的方法。


其他链接
--------------------

- http://blog.csdn.net/zhoudaxia/article/details/17189757
