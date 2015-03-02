## 前言：

其实小匹夫在U3D的开发中一直对U3D的跨平台能力很好奇。到底是什么原理使得U3D可以跨平台呢？后来发现了[Mono](http://www.mono-project.com/)的作用，并进一步了解到了CIL的存在。所以，作为一个对Unity3D跨平台能力感兴趣的U3D程序猿，小匹夫如何能不关注CIL这个话题呢？那么下面各位看官就拾起语文老师教导我们的作文口诀（Why，What，How），和小匹夫一起走进CIL的世界吧~

## Why?

回到本文的题目，U3D或者说Mono的跨平台是如何做到的？

如果换做小匹夫或者看官你来做，应该怎么实现一套代码对应多种平台呢？

其实原理想想也简单，生活中也有很多可以参考的例子，比如下图（谁让小匹夫是做移动端开发的呢，只能物尽其用从自己身边找例子了T.T）：

![](http://images.cnitblog.com/blog/686199/201501/082157220463329.png)

像这样一根线，管你是安卓还是ios都能充电。所以从这个意义上，这货也实现了跨平台。那么我们能从它身上学到什么呢？对的，那就是从一样的能源（电）到不同的平台（ios，安卓）之间需要一个中间层过度转换一下。

那么来到U3D为何能跨平台，简而言之，其实现原理在于使用了叫CIL（Common Intermediate Language通用中间语言，也叫做MSIL微软中间语言）的一种代码指令集，CIL可以在任何支持CLI（Common Language Infrastructure，通用语言基础结构）的环境中运行，就像.NET是微软对这一标准的实现，Mono则是对CLI的又一实现。由于CIL能运行在所有支持CLI的环境中，例如刚刚提到的.NET运行时以及Mono运行时，也就是说和具体的平台或者CPU无关。这样就无需根据平台的不同而部署不同的内容了。所以到这里，各位也应该恍然大了。代码的编译只需要分为两部分就好了嘛：

1.  从代码本身到CIL的编译（其实之后CIL还会被编译成一种位元码，生成一个CLI assembly）
2.  运行时从CIL（其实是CLI assembly，不过为了直观理解，不必纠结这种细节）到本地指令的即时编译（这就引出了为何U3D官方没有提供热更新的原因：**在iOS平台中Mono无法使用JIT引擎，而是以Full AOT模式运行的，所以此处说的即时编译不包括IOS**）

## What？

上文也说了CIL是指令集，但是不是还是太模糊了呢？所以语文老师教导我们，描述一个东西时肯定要先从外貌写起。遵循老师的教导，我们不妨先通过工具来看看CIL到底长什么样。

工具就是ildasm了。下面小匹夫写一个简单的.cs看看生成的CIL代码长什么样。

C#代码：

	class Class1
	{
	    public static void Main(string[] args)
	    {
	        	System.Console.WriteLine("hi");
    	}
	}

CIL代码：

	.class private auto ansi beforefieldinit Class1
       extends [mscorlib]System.Object
	{
	  .method public hidebysig static void  Main(string[] args) cil managed
	  {
    	.entrypoint
    	// 代码大小       13 (0xd)
    	.maxstack  8
    	IL_0000:  nop
    	IL_0001:  ldstr      "hi"
    	IL_0006:  call       void 	[mscorlib]System.Console::WriteLine(string)
    	IL_000b:  nop
    	IL_000c:  ret
	  } // end of method Class1::Main

	  .method public hidebysig specialname rtspecialname 
          instance void  .ctor() cil managed
	  {
	    // 代码大小       7 (0x7)
	    .maxstack  8
	    IL_0000:  ldarg.0
	    IL_0001:  call       instance void [mscorlib]System.Object::.ctor()
	    IL_0006:  ret
	  } // end of method Class1::.ctor

	} // end of class Class1

好啦。代码虽然简单，但是也能说明足够多的问题。那么和CIL的第一次亲密接触，能给我们留下什么直观的印象呢？

1.  以“.”一个点号开头的，例如上面这份代码中的：.class、.method 。我们称之为**CIL指令（directive）**，用于描述.NET程序集总体结构的标记。为啥需要它呢？因为你总得告诉编译器你处理的是啥吧。
2.  貌似CIL代码中还看到了private、public这样的身影。姑且称之为**CIL特性（attribute）**。它的作用也很好理解，通过CIL指令并不能完全说明.NET成员和类，针对CIL指令进行补充说明成员或者类的特性的。市面上常见的还有：extends，implements等等。
3.  每一行CIL代码基本都有的，对，那就是**CIL操作码**咯。小匹夫从网上找了一份汉化的操作码表放在**附录**部分，当然英文版的你的vs就有。

直观的印象有了，但是离我们的短期目标，说清楚（或者说介绍个大概）CIL是What，甚至是终极目标，搞明白Mono为何能跨平台还有2万4千9百里的距离。

好啦，话不多说，继续乱侃。

参照附录中的操作码表，对照可以总结出一份更易读的表格。那就是如下的表啦。

![image](http://images.cnblogs.com/cnblogs_com/murongxiaopifu/662093/o_testd.png)

在此，小匹夫想请各位认真读表，然后心中默数3个数，最后看看都能发现些什么。

#### 基于堆栈

如果是小匹夫的话，第一感觉就是基本每一条描述中都包含一个“栈”。不错，CIL是**基于堆栈**的，也就是说CIL的VM（mono运行时）是一个栈式机。这就意味着数据是推入堆栈，通过堆栈来操作的，而非通过CPU的寄存器来操作，这更加验证了其和具体的CPU架构没有关系。为了说明这一点，小匹夫举个例子好啦。

大学时候学单片机的时候记得做加法大概是这样的：

	add eax,-2
	
其中的eax是啥？寄存器。所以如果CIL处理数据要通过cpu的寄存器的话，那也就不可能和cpu的架构无关了。

当然，CIL之所以是基于堆栈而非CPU的另一个原因是相比较于cpu的寄存器，操作堆栈实在太简单了。回到刚才小匹夫说的大学时候曾经学过的单片机那门课程上，当时记得各种寄存器，各种标志位，各种。。。，而堆栈只需要简单的压栈和弹出，因此对于虚拟机的实现来说是再合适不过了。所以想要更具体的了解CIL基于堆栈这一点，各位可以去看一下堆栈方面的内容。这里小匹夫就不拓展了。

#### 面向对象

那么第二感觉呢？貌似附录的表中有new对象的语句呀。嗯，的确，CIL同样是**面向对象**的。

这意味着什么呢？那就是在CIL中你可以创建对象，调用对象的方法，访问对象的成员。而这里需要注意的就是对方法的调用。

回到上表中的右上角。对，就是对参数的操作部分。静态方法和实例方法是不同的哦~

1.  静态方法：ldarg.0没有被占用，所以参数从ldarg.0开始。
2.  实例方法：ldarg.0是被this占用的，也就是说实际上的参数是从ldarg.1开始的。

举个例子：假设你有一个类Murong中有一个静态方法Add(int32 a, int32 b)，实现的内容就如同它的名字一样使两个数相加，所以需要2个参数。和一个实例方法TellName(string name)，这个方法会告诉你传入的名字。

	class  Murong
	{
	    public void TellName(string name)
	    {
	        System.Console.WriteLine(name);
	    }
	
	    public static int Add(int a, int b)
	    {
       		return a + b;
	    }
	}

#### 静态方法的处理：

那么其中的静态方法Add的CIL代码如下：

	//小匹夫注释一下。
	.method public hidebysig static int32  	Add(int32 a,
                                           	int32 b) cil managed
	{
	  // 代码大小       9 (0x9)
	  .maxstack  2
	  .locals init ([0] int32 CS$1$0000)   //初始化局部变量列表。因为我们只返回了一个int型。所以这里声明了一个int32类型。索引为0
	  IL_0000:  nop
	  IL_0001:  ldarg.0     //将索引为 0 的参数加载到计算堆栈上。
	  IL_0002:  ldarg.1     //将索引为 1 的参数加载到计算堆栈上。
	  IL_0003:  add          //计算
	  IL_0004:  stloc.0      //从计算堆栈的顶部弹出当前值并将其存储到索引 0 处的局部变量列表中。
	  IL_0005:  br.s       IL_0007
	  IL_0007:  ldloc.0     //将索引 0 处的局部变量加载到计算堆栈上。
	  IL_0008:  ret           //返回该值
	} // end of method Murong::Add

那么我们调用这个静态函数应该就是这样咯。

	Murong.Add(1, 2);

对应的CIL代码为：

	IL_0001:  ldc.i4.1 //将整数1压入栈中
  	IL_0002:  ldc.i4.2 //将整数2压入栈中
  	IL_0003:  call       int32 Murong::Add(int32,
                                         int32)  //调用静态方法

可见CIL直接call了Murong的Add方法，而不需要一个Murong的实例。

#### 实例方法的处理：
Murong类中的实例方法TellName()的CIL代码如下：

	.method public hidebysig instance void  TellName(string name) cil managed
	{
	  // 代码大小       9 (0x9)
	  .maxstack  8
	  IL_0000:  nop
	  IL_0001:  ldarg.1     //看到和静态方法的区别了吗？
	  IL_0002:  call       void [mscorlib]System.Console::WriteLine(string)
	  IL_0007:  nop
	  IL_0008:  ret
	} // end of method Murong::TellName

看到和静态方法的区别了吗？对，第一个参数对应的是ldarg.1中的参数1，而不是静态方法中的0。因为此时参数0相当于this，this是不用参与参数传递的。

那么我们再看看调用实例方法的C#代码和对应的CIL代码是如何的。

C#：

	//C#
	Murong murong = new Murong();
	murong.TellName("chenjiadong");


CIL：

	.locals init ([0] class Murong murong)   //因为C#代码中定义了一个Murong类型的变量，所以局部变量列表的索引0为该类型的引用。
	//....
	IL_0009:  newobj     instance void 	Murong::.ctor() //相比上面的静态方法的调用，此处new一个新对象，出现了instance方法。
	IL_000e:  stloc.0
	IL_000f:  ldloc.0
	IL_0010:  ldstr      "chenjiadong" //小匹夫的名字入栈
	IL_0015:  callvirt   instance void 	Murong::TellName(string) //实例方法的调用也有instance

到此，受制于篇幅所限（小匹夫不想写那么多字啊啊啊！）CIL是What的问题大致介绍一下。当然没有再拓展，以后小匹夫可能会再详细写一下这块。

## How？

记得语文老师说过，写作文最重要的一点是要首尾呼应。既然咱们开篇就提出了U3D为何能跨平台的问题，那么接近文章的结尾咱们就再来

#### 提问：

Q：上面的Why部分，咱们知道了U3D能跨平台是因为存在着一个能通吃的中间语言CIL，这也是所谓跨平台的前提，但是为啥CIL能通吃各大平台呢？当然可以说CIL基于堆栈，跟你CPU怎么架构的没啥关系，但是感觉过于理论化、学术化，那还有没有通俗化、工程化的说法呢？

A：原因就是前面小匹夫提到过的，.Net运行时和Mono运行时。也就是说CIL语言其实是运行在虚拟机中的，具体到咱们的U3D也就是mono的运行时了，换言之mono运行的其实CIL语言，CIL也并非真正的在本地运行，而是在mono运行时中运行的，运行在本地的是被编译后生成的原生代码。当然看官博的文章，他们似乎也在开发自己的“mono”，也就是被称为脚本的未来的IL2Cpp，这种类似运行时的功能是将IL再编译成c++，再由c++编译成原生代码，据说效率提升很可观，小匹夫也是蛮期待的。

这里为了“实现跨平台式的演示”，小匹夫用mac给各位做个测试好啦：

#### 从C#到CIL

新建一个cs文件，然后使用mono来运行。这个cs文件内容如下：

![](http://images.cnitblog.com/blog/686199/201501/100034397038251.png)

然后咱们直接在命令行中运行这个cs文件试试~

![](http://images.cnitblog.com/blog/686199/201501/100035575009350.png)

说的很清楚，文件没有包含一个CIL映像。可见mono是不能直接运行cs文件的。假如我们把它编译成CIL呢？那么我们用mono带的mcs来编译小匹夫的Test.cs文件。


	mcs Test.cs
生成了什么呢？如图：

![](http://images.cnitblog.com/blog/686199/201501/100039109378522.png)

好像没见有叫.IL的文件生成啊？反而好像多了一个.exe文件？可是没听说Mac能运行exe文件呀？可为啥又生成了.exe呢？各位看官可能要说，小匹夫你是不是拿windows截图P的啊？嘿嘿，小匹夫可不敢。辣么真相其实就是这个exe并不是让Mac来运行的，而是留给mono运行时来运行的，换言之这个文件的可执行代码形式是CIL的位元码形态。到此，我们完成了从C#到CIL的过程。接下来就让我们运行下刚刚的成果好啦。

	mono Test.exe

&nbsp;![](http://images.cnitblog.com/blog/686199/201501/100045087653170.png)

结果是输出了一个大大的“Hi”。这里，就引出了下一个部分。

#### 从CIL到Native Code

这个“HI”可是在小匹夫的MAC终端上出现的呀，那么就证明这个C#写的代码在MAC上运行的还挺“嗨”。

为啥呢？为啥C#写的代码能跑在MAC上呢？这就不得不提从CIL如何到本机原生代码的过程了。Mono提供了两种编译方式，就是我们经常能看到的：JIT（Just-in-Time compilation，即时编译）和AOT（Ahead-of-Time，提前编译或静态编译）。这两种方式都是将CIL进一步编译成平台的原生代码。这也是实现跨平台的最后一步。下面就分头介绍一下。

#### JIT即时编译：

从名字就能看的出来，即时编译，或者称之为动态编译，是在程序执行时才编译代码，解释一条语句执行一条语句，即将一条中间的托管的语句翻译成一条机器语句，然后执行这条机器语句。但同时也会将编译过的代码进行缓存，而不是每一次都进行编译。所以可以说它是静态编译和解释器的结合体。不过你想想机器既要处理代码的逻辑，同时还要进行编译的工作，所以其运行时的效率肯定是受到影响的。因此，Mono会有一部分代码通过AOT静态编译，以降低在程序运行时JIT动态编译在效率上的问题。

不过一向严苛的IOS平台是不允许这种动态的编译方式的，这也是U3D官方无法给出热更新方案的一个原因。而Android平台恰恰相反，Dalvik虚拟机使用的就是JIT方案。

#### AOT静态编译：

其实Mono的AOT静态编译和JIT并非对立的。AOT同样使用了JIT来进行编译，只不过是被AOT编译的代码在程序运行之前就已经编译好了。当然还有一部分代码会通过JIT来进行动态编译。下面小匹夫就手动操作一下mono，让它进行一次AOT编译。

	//在命令行输入
	mono --aot Test.exe

结果：![](http://images.cnitblog.com/blog/686199/201501/110157240469330.png)

从图中可以看到JIT time： 39 ms，也就是说Mono的AOT模式其实会使用到JIT，同时我们看到了生成了一个适应小匹夫的MAC的动态库Test.exe.dylib，而在Linux生成就是.so（共享库）。

AOT编译出来的库，除了包括我们的代码之外，还有被缓存的元数据信息。所以我们甚至可以只编译元数据信息而不编译代码。例如这样：

	//只包含元数据的信息
	mono --aot=metadata-only Test.exe

![](http://images.cnitblog.com/blog/686199/201501/110248517655449.png)

可见代码没有被包括进来。

那么简单总结一下AOT的过程：

1.  收集要被编译的方法
2.  使用JIT进行编译
3.  发射（Emitting）经JIT编译过的代码和其他信息
4.  直接生成文件或者调用本地汇编器或连接器进行处理之后生成文件。（例如上图中使用了小匹夫本地的gcc）

#### Full AOT

当然上文也说了，IOS平台是禁止使用JIT的，可看样子Mono的AOT模式仍然会保留一部分代码会在程序运行时动态编译。所以为了破解这个问题，Mono提供了一个被称为Full AOT的模式。即预先对程序集中的所有CIL代码进行AOT编译生成一个本地代码映像，然后在运行时直接加载这个映像而不再使用JIT引擎。目前由于技术或实现上的原因在使用Full AOT时有一些限制，不过这里不再多说了。以后也还会更细的分析下AOT。


## 总结

好啦，写到现在也已经到了凌晨3：04分了。感觉写的内容也差不多了。那么对本文的主题U3D为何能跨平台以及CIL做个最终的总结陈词：

1.  CIL是CLI标准定义的一种可读性较低的语言。
2.  以.NET或mono等实现CLI标准的运行环境为目标的语言要先编译成CIL，之后CIL会被编译，并且以位元码的形式存在（源代码--->中间语言的过程）。
3.  这种位元码运行在虚拟机中(.net mono的运行时）。
4.  这种位元码可以被进一步编译成不同平台的原生代码（中间语言--->原生代码的过程）。
5.  面向对象
6.  基于堆栈

## [附录](http://images.cnblogs.com/cnblogs_com/murongxiaopifu/662093/o_test2.png)
