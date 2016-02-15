title: 《Java编程思想》笔记（第5章 初始化与清理）
date: 2016-02-14 17:22:25
categories: Java
tags: Think in Java
comments: true
---

第五章 初始化与清理
======

## 清理：终结处理与垃圾回收

> 程序员都了解初始化的重要性，但常常会忘记同样也重要的清理工作。毕竟，谁需要清理一个int呢？但在使用程序库时，把一个对象用完后就“弃之不顾”的做法并非总是安全的。当然Java有垃圾回收器负责回收无用对象占据的内存资源。但也有特殊情况：假定你的对象（并非使用new）获得了一块“特殊”的内存区域，由于垃圾回收器只知道释放那些经由new分配的内存，所以它不知道该如何释放该对象这块“特殊”内存。为了应对这样的情况，Java允许在类中定义一个名为finalize()的方法。它的工作原理“假定”是这样的：一旦垃圾回收准备好释放对象占用的存储空间，将首先调用其finalize()方法，并且在下一次垃圾回收动作发生时，才会真正回收对象占用的内存。所以要是你打算用finalize()，就能在垃圾回收时刻做一些重要的清理工作。

> 这里有一个潜在的编程陷阱，因为有些程序员（特别是C++程序员）刚开始可能会误把finalize()当作C++的析构函数（C++中销毁对象必须用到这个函数）。所以有必要明确区分一下：在C++中，对象一定会被销毁（如果程序中没有缺陷的话）；而Java里面的对象却并非总是被垃圾回收。或者换句话说：

> 1. 对象可能不被垃圾回收
> 2. 垃圾回收并不等于“析构”

> 牢记这些，就能远离困扰。这意味着在你不需要某个对象之前，如果必须执行某些动作，那么你得自己去做。Java并未提供“析构函数”或相似的概念，要做类似的清理工作，必须自己动手创建一个执行清理工作的普通方法。例如，假设某个对象在创建过程中将自己绘制到屏幕上，如果不是明确地从屏蔽上将其擦除，它可能永远得不到清理。如果在finalize()里加加入某种擦除功能，当“垃圾回收”发生时（不能保证一定会发生），finalize()得到了调用，图片就会被擦除。要是“垃圾回收”没有发生，图像就会一直保留一下来。

> 也许你会发现，只要程序没有濒临存储空间用完的那一刻，对象占用的空间就总也得不到释放。如果程序执行结束，并且垃圾回收器一直都没有释放你创建的任何对象的存在空间，则随着程序的退出，那些资源也会全部交还给操作系统。这个策略是恰当的，因为垃圾回收本身也有开销，要是不使用它，那就不用支付部分开销了。

### finalize()的用途何在
> 此时，读者已经明白了不该将finalize()作为通用的清理方法。那么，finalize()的真正用途是什么呢？

> 这里引出要记住的第三点：

> 3.垃圾回收只与内存有关。

> 也就是说，使用垃圾回收器的唯一原因是为了回收程序不再使用的内存。所以对于与垃圾回收有关的任何行为来说（尤其是finalize()方法），它们必须同内存及其回收有关。

> 但这是否意味着要是对象中含有其它对象，finalize()就应该明确释放那些对象呢？不，无论对象是如何创建的，垃圾回收器都会负责释放对象占据的所有内存。这就将对finalize()的需求限制到一种特殊的情况，即通过某种创建对象方式以外的方式为对象分配了存储空间。不过，读者也看到了，Java中一切皆为对象，那这种特殊情况是怎么回事呢？

> 看来之所以要有finalize()，是由于在分配内存时可能彩了类似C语言中的做法，而非Java中的通常做法。这种情况主要发生在使用“本地方法”的情况下，本地方法是一种在Java中调用非Java代码的方式。本地方法目前只支持C和C++，但它们可以调用其它语言写的代码，所以实际上可以调用任何代码。在非Java代码中，也许会调用C的malloc()函数系列来分配存储空间，而且除非调用free()函数，否则存储空间将得不到释放，从而千万内存泄露。当然free()是C和C++中的函数，所以需要在finalize()中用本地方法调用它。

<!-- more -->

### 垃圾回收器如何工作

> 在以前所用过的程序语言中，在堆上分配对象的代价十分高昂，因此读者自然会觉得Java中所有对象（基本类型除外）都在堆上分配的方式也非常高昂。然而，垃圾回收器对于提高对象的创建速度，却具有明显的效果。听起来很奇怪——存储空间的释放竟然会影响存储空间的分配，但这确实是某些Java虚拟机的工作方式。这也意味着，Java从堆分配空间的咁速度，可以和其它语言从堆栈上分配空间的速度相媲美。

<br />
> 要想更好地理解Java中的垃圾回收，先了解其它系统中的垃圾回收机制将会很有帮助。`引用记数`是一种简单但速度很慢的垃圾回收技术。每个对象都含有一个引用记数器当有引用 连接至对象时，引用计数加1。当引用离开作用域或被置为null时，引用记数减1。虽然管理引用记数的开销不大，但这项开销在整个程序生命周期中将持续发生。垃圾回收器会在含有全部对象的列表上遍历，当发现某个对象的引用计数为0时，就释放其占用的空间（但是，引用记数模式经常会在记数值变为0时立即释放对象）。这种方法有个缺陷，如果对象之间存在循环引用，可交互自引用的对象组所需要的工作量极大。引用记数常用来说明垃圾回收的工作方式，但似乎从未被应用于任何一种Java虚拟机实现。

>在一些更快的模式中，垃圾回收器并非基于引用记数技术。它们依据的思想是：对任何“活“的对象，一定能最终追溯到其存在堆栈或静态存储区之中的引用。这个引用链条可能会穿过数个对象层次。由此，如果从堆栈和静态存储区开始，遍历所有的引用，就能找到所有“活”的对象。对发现的每个引用，必须追踪它所引用的对象，然后是此对象包含的所有引用，如此反复进行，直到“根源于堆栈和静态存储区的引用”所形成的网络全部被访问为止。你所访问过的对象必须都是“活”的。注意，这就解决了“交互自引用的对象组”的问题——这种现象根本不会被发现，因此也就被自动回收了。

> 在这种方式下，Java虚拟机将可采用一种自适应的垃圾回收技术。至于如何处理找到的存活对象，取决于不同的Java虚拟机实现。有一种做法名为`停止—复制`。显然这意味着，先暂停程序的运行（所以它不属于后台回收模式），然后将所有存活的对象从当前堆复制到另一个堆，没有被复制的全部都是垃圾。当对象被复制到新堆时，它们是一个挨着一个的，所以新堆保持紧凑排列，然后就可以按前述方法简单、直接地分配新空间了。

> 当把对象从一处搬到另一处时，所有指向它的那些引用都必须修正。位于堆或静态存储区的引用可以直接被修正，但可能还有其它其它指向这些对象的引用，它们在遍历的过程中才能被找到（可以想像成有个表格，将旧地址映射到新地址）。

> 对于这种所谓的*复制式回收器*而言，效率会降低，这有两个原因，首先，得有两个堆，然后得在这两个分离的堆之间来回捣腾，从而得维护比实际需要多一倍的空间。某些Java虚拟机对此问题的处理方式是，按需从堆中分配几块较大的内存，复制动作发生在这些大块内存之间。

> 第二个问题在于复制。程序进入稳定状态之后，可能只会产生觉得是垃圾。甚至没有垃圾。尽管如此，复制式回收器仍然会将所有存储自一处复制到另一处，这很浪费。为了避免这种买情形，一些Java虚拟机会进行检查：要是没有新垃圾产生，就会转换到另一种工作模式（即*自适应*）。这种模式称为`标记-清扫`，Sun公司早期版本的Java虚拟机使用了这种技术。对一般用途而言，`标记-清扫`方式速度相当慢，但是当你知道只会产生少量垃圾甚至不会产生垃圾时，它的速度就很快了。

> `标记-清扫`所依据的思路同样是从堆栈和静态存储区出发，遍历所有的引用，进而找出所有存活的的对象。每当它找到一个存活的对象，就会给对象设一个标记，这个过程中不会回收任何对象。只有全部标记工作完成的时候，清理动作才会开始。在清理过程中，没有标记的对象将被释放，不会发生任何复制动作。所以剩下的堆空间是不连续的，垃圾回器要是希望得到连续空间的话，就得重新整理剩下的对象。

> `停止-复制`的意思是这种垃圾回收动作不是在后台进行的；圾回收动作发生的同时，程序将会被暂停。在Sun公司的文档中会发现，许多参考文献将垃圾回收视为低优先级的后台进程，但事实上垃圾回收器在Sun公司早期版本的Java虚拟机器并非以这种方式实现的。当可用内存数量较低时，Sun版本的垃圾回器会暂停运行程序，同样，`标记-清扫`工作也必须在程序暂停的情况下才能进行。

> 如前文所述，在这里所讨论的Java虚拟机中，内存分配以较大的“块”为单位。如果对象较大，它会占用单独的块。严格来说，`停止-复制`要求在释放旧有对象之前，必须先把所有存活对象从旧堆里复制到新堆，这将导致大量内存复制行为。有了块之后，垃圾回收器在回收的时候就可以往废弃的块里拷贝对象了。每个块都有相应的`代数`来记录它是否还在存活。通常，如果块在某处被引用，其代数会增加；垃圾回收器将对上次回收动作之后新分配的块进行整理。这对处理大量短命的临时对象很有帮助。增加回收器会定期进行完整的清理动作——大型对象仍然不会被复制（只是其代数会增加），内含小型对象的那些块则被复制并整理。Java虚拟机会进行监视，如果所有对象都很稳定，垃圾回收器的效率降低的话，就切换到`标记-清扫`方式；同样，Java虚拟会跟踪“标记——清扫”的效果，要是堆空间出现很多碎片，就会切换回“停止——复制 ”方式。这就是“自适应”技术，你可以给它个罗嗦的称呼：“自适应的、分代的、停止——复制，标记——清扫”式垃圾回收器。

> Java虚拟机中有许多附加技术用以提升速度。尤其是与加载器操作有关的，被称为“即时”(Just-In-Time，JIT)编译器的技术。这种技术可以把程序全部或部分翻译成本地机器码（这本来是Java虚拟机的工作），程序运行速度因此得以提升，当需要装载某个类（通常是在为该类创建第一个对象）时，编译器会先找到其.class文件，然后将该类的字节码装入内存，此时，有两种方案可供选择。一种是就让即时编译器编译所有代码。但这种做法有两个缺陷：这种加载动作散落在整个程序生命周期内，累加起来要花更多时间；并且会增加可执行代码的长度（字节码比即时编译器展开后的本地机器码小很多），这将导致页面调度，从而降低程序速度。另一种做法称为`惰性评估`，意思是即时编译器只在必要的时候才编译代码。这样，从不会被执行的代码也许就压根不会被JIT所编译。新版JDK中的Java HotSpot技术就采用了类似的方式，代码每次被执行的时候都会做一些优化，所以执行的次数越多，它的速度就越快。

### 静态数据的初始化
```java
public class StaticInitialization {
	public static void main(String[] args) {
		System.out.println("Creating new Cupboard() in main");
		new Cupboard();
		System.out.println("Creating new Cupboard() in main");
		new Cupboard();
		table.f2(1);
		cupboard.f3(1);
	}

	static Table table = new Table();
	static Cupboard cupboard = new Cupboard();
}

class Bowl {
	Bowl(int marker) {
		System.out.println("Bowl(" + marker + ")");
	}

	void f1(int marker) {
		System.out.println("f1(" + marker + ")");
	}
}

class Table {
	static Bowl bowl1 = new Bowl(1);

	Table() {
		System.out.println("Table()");
	}

	void f2(int marker) {
		System.out.println("f2(" + marker + ")");
	}

	static Bowl bowl2 = new Bowl(2);
}

class Cupboard {
	Bowl bowl3 = new Bowl(3);

	static Bowl bowl4 = new Bowl(4);

	Cupboard() {
		System.out.println("Cupboard()");
		bowl4.f1(2);
	}

	void f3(int marker) {
		System.out.println("f3(" + marker + ")");
	}

	static Bowl bowl5 = new Bowl(5);
}
```

输出：
```
Bowl(1)
Bowl(2)
Table()
Bowl(4)
Bowl(5)
Bowl(3)
Cupboard()
f1(2)
Creating new Cupboard() in main
Bowl(3)
Cupboard()
f1(2)
Creating new Cupboard() in main
Bowl(3)
Cupboard()
f1(2)
f2(1)
f3(1)
```

> 初始化的顺序是先静态对象（如果它们尚未因前面的对象创建过程而被初始化），而后是“非静态”对象。从输出结果中可以观察到这一点。要执行main()（非静态方法），必须加载StaticInitialization类，然后其静态域table和cupboard被初始化，这将导致它们对应类也被加载，并且由于它们也都包含静态Bowl对象，因此Bowl随后也被加载了。实际情况通常并非如此，因为在典型的程序中，不会像在本例中所做的那样，将所有的事物都通过static联系起来。

> 总结一下对象的创建过程，假设有个名为Dog的类：

> 1. 即使没有显式地使用static关键字，构造器实际上也是静态方法。因此，当首次创建类型为Dog的对象时（构造器可以看成静态方法），或者**Dog类的静态方法/静态域首次被访问时**，Java解释器必须查找类路径，以定义Dog.class文件。
> 2. 然后载入Dog.class（这将创建一个Class对象），有关静态初始化的所有动作都会执行。因此，静态初始化只在Class对象首次加载的时候进行一次。
> 3. 当用new Dlog()创建对象的时候，首先将在堆上为Dog对象分配足够的存储空间。
> 4. 这块存储空间会被清零，这就自动地将Dog对象中的所有基本类型数据设置成了默认值（对数字来说是0，对布尔类和字符型也相同），而引用则被设置成了null。
> 5. 执行所有出现于字段定义处的初始化动作。
> 6. 执行构造器。这可能会牵涉到很多动作，尤其是涉及到继承的时候。

### 显式的静态初始化

> Java允许将多个静态初始化动作组织成一个特殊的“静态子句”（有时也叫做“静态块”）。
> 尽管静态块看起来像个方法，但它实际只是一段跟在static关键字后面的代码。与其它静态初始化动作一样，这段代码仅执行一次：当首次生成这个类的一个对象时，或者**首次访问属于那个类的静态数据成员时**（即便从未生成过那个类的对象）。

```java
public class ExplicitStatic {
	public static void main(String[] args) {
		System.out.println("Inside main()");
		Cups.cup1.f(99);
	}

}

class Cup {
	Cup(int marker) {
		System.out.println("Cup(" + marker + ")");
	}

	void f(int marker) {
		System.out.println("f(" + marker + ")");
	}
}

class Cups {
	static Cup cup1;
	static Cup cup2;

	static {
		cup1 = new Cup(1);
		cup2 = new Cup(2);
	}

	Cups() {
		System.out.println("Cups()");
	}
}

```
输出：
```
Inside main()
Cup(1)
Cup(2)
f(99)
```

### 非静态实例初始化

```java
public class Mugs {
	Mug mug1;
	Mug mug2;

	{
		mug1 = new Mug(1);
		mug2 = new Mug(2);
		System.out.println("mug1 & mug2 initialized");
	}

	public Mugs() {
		System.out.println("Mugs()");
	}

	public Mugs(int i) {
		System.out.println("Mugs(int)");
	}

	public static void main(String[] args) {
		System.out.println("Inside main()");
		new Mugs();
		System.out.println("new Mugs() completed");
		new Mugs(1);
		System.out.println("new Mugs(1) completed");
	}

}

class Mug {
	Mug(int marker) {
		System.out.println("Mug(" + marker + ")");
	}

	void f(int marker) {
		System.out.println("f(" + marker + ")");
	}
}
```
输出：
```
Inside main()
Mug(1)
Mug(2)
mug1 & mug2 initialized
Mugs()
new Mugs() completed
Mug(1)
Mug(2)
mug1 & mug2 initialized
Mugs(int)
new Mugs(1) completed
```
这种语法对于支持“匿名内部类”的初始化是必须的，但是它也使得你可以保证无论调用了哪个显式构造器，某些操作都会发生。从输出中可以看到实例初始化子句是在两个构造器之前执行的。