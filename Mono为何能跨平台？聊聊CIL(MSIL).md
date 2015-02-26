<h2>前言：</h2>
<p>其实小匹夫在U3D的开发中一直对U3D的跨平台能力很好奇。到底是什么原理使得U3D可以跨平台呢？后来发现了Mono的作用，并进一步了解到了CIL的存在。所以，作为一个对Unity3D跨平台能力感兴趣的U3D程序猿，小匹夫如何能不关注CIL这个话题呢？那么下面各位看官就拾起语文老师教导我们的作文口诀（<a href="#why" target="_blank">Why</a>，<a href="#what" target="_blank">What</a>，<a href="#how" target="_blank">How</a>），和小匹夫一起走进CIL的世界吧~</p>
<h2><a name="why"></a>Why?</h2>
<p>回到本文的题目，U3D或者说Mono的跨平台是如何做到的？</p>
<p>如果换做小匹夫或者看官你来做，应该怎么实现一套代码对应多种平台呢？</p>
<p>其实原理想想也简单，生活中也有很多可以参考的例子，比如下图（谁让小匹夫是做移动端开发的呢，只能物尽其用从自己身边找例子了T.T）：</p>
<p><img src="http://images.cnitblog.com/blog/686199/201501/082157220463329.png" alt="" /></p>
<p>像这样一根线，管你是安卓还是ios都能充电。所以从这个意义上，这货也实现了跨平台。那么我们能从它身上学到什么呢？对的，那就是从一样的能源（电）到不同的平台（ios，安卓）之间需要一个中间层过度转换一下。</p>
<p>那么来到U3D为何能跨平台，简而言之，其实现原理在于使用了叫CIL（Common Intermediate Language通用中间语言，也叫做MSIL微软中间语言）的一种代码指令集，CIL可以在任何支持CLI（Common Language Infrastructure，通用语言基础结构）的环境中运行，就像.NET是微软对这一标准的实现，Mono则是对CLI的又一实现。由于CIL能运行在所有支持CLI的环境中，例如刚刚提到的.NET运行时以及Mono运行时，也就是说和具体的平台或者CPU无关。这样就无需根据平台的不同而部署不同的内容了。所以到这里，各位也应该恍然大了。代码的编译只需要分为两部分就好了嘛：</p>
<ol>
<li>从代码本身到CIL的编译（其实之后CIL还会被编译成一种位元码，生成一个CLI assembly）</li>
<li>运行时从CIL（其实是CLI assembly，不过为了直观理解，不必纠结这种细节）到本地指令的即时编译（这就引出了为何U3D官方没有提供热更新的原因：<strong>在iOS平台中Mono无法使用JIT引擎，而是以Full AOT模式运行的，所以此处说的额即时编译不包括IOS</strong>）</li>
</ol>
<h2><a name="what"></a>What？</h2>
<p>上文也说了CIL是指令集，但是不是还是太模糊了呢？所以语文老师教导我们，描述一个东西时肯定要先从外貌写起。遵循老师的教导，我们不妨先通过工具来看看CIL到底长什么样。</p>
<p>工具就是ildasm了。下面小匹夫写一个简单的.cs看看生成的CIL代码长什么样。</p>
<p>C#代码：</p>
<div class="cnblogs_code">
<pre><span style="color: #0000ff;">class</span><span style="color: #000000;"> Class1
{
    </span><span style="color: #0000ff;">public</span> <span style="color: #0000ff;">static</span> <span style="color: #0000ff;">void</span> Main(<span style="color: #0000ff;">string</span><span style="color: #000000;">[] args)
    {
        System.Console.WriteLine(</span><span style="color: #800000;">"</span><span style="color: #800000;">hi</span><span style="color: #800000;">"</span><span style="color: #000000;">);
    }
}</span></pre>
</div>
<p>CIL代码：</p>
<div class="cnblogs_code">
<pre>.<span style="color: #0000ff;">class</span> <span style="color: #0000ff;">private</span><span style="color: #000000;"> auto ansi beforefieldinit Class1
       extends [mscorlib]System.Object
{
  .method </span><span style="color: #0000ff;">public</span> hidebysig <span style="color: #0000ff;">static</span> <span style="color: #0000ff;">void</span>  Main(<span style="color: #0000ff;">string</span><span style="color: #000000;">[] args) cil managed
  {
    .entrypoint
    </span><span style="color: #008000;">//</span><span style="color: #008000;"> 代码大小       13 (0xd)</span>
    .maxstack  <span style="color: #800080;">8</span><span style="color: #000000;">
    IL_0000:  nop
    IL_0001:  ldstr      </span><span style="color: #800000;">"</span><span style="color: #800000;">hi</span><span style="color: #800000;">"</span><span style="color: #000000;">
    IL_0006:  call       </span><span style="color: #0000ff;">void</span> [mscorlib]System.Console::WriteLine(<span style="color: #0000ff;">string</span><span style="color: #000000;">)
    IL_000b:  nop
    IL_000c:  ret
  } </span><span style="color: #008000;">//</span><span style="color: #008000;"> end of method Class1::Main</span>
<span style="color: #000000;">
  .method </span><span style="color: #0000ff;">public</span><span style="color: #000000;"> hidebysig specialname rtspecialname 
          instance </span><span style="color: #0000ff;">void</span><span style="color: #000000;">  .ctor() cil managed
  {
    </span><span style="color: #008000;">//</span><span style="color: #008000;"> 代码大小       7 (0x7)</span>
    .maxstack  <span style="color: #800080;">8</span><span style="color: #000000;">
    IL_0000:  ldarg.</span><span style="color: #800080;">0</span><span style="color: #000000;">
    IL_0001:  call       instance </span><span style="color: #0000ff;">void</span><span style="color: #000000;"> [mscorlib]System.Object::.ctor()
    IL_0006:  ret
  } </span><span style="color: #008000;">//</span><span style="color: #008000;"> end of method Class1::.ctor</span>
<span style="color: #000000;">
} </span><span style="color: #008000;">//</span><span style="color: #008000;"> end of class Class1</span></pre>
</div>
<p>好啦。代码虽然简单，但是也能说明足够多的问题。那么和CIL的第一次亲密接触，能给我们留下什么直观的印象呢？</p>
<ol>
<li>以&ldquo;.&rdquo;一个点号开头的，例如上面这份代码中的：.class、.method 。我们称之为<strong>CIL指令（directive）</strong>，用于描述.NET程序集总体结构的标记。为啥需要它呢？因为你总得告诉编译器你处理的是啥吧。</li>
<li>貌似CIL代码中还看到了private、public这样的身影。姑且称之为<strong>CIL特性（attribute）</strong>。它的作用也很好理解，通过CIL指令并不能完全说明.NET成员和类，针对CIL指令进行补充说明成员或者类的特性的。市面上常见的还有：extends，implements等等。</li>
<li>每一行CIL代码基本都有的，对，那就是<strong>CIL操作码</strong>咯。小匹夫从网上找了一份汉化的操作码表放在<a href="#fulu" target="_blank"><strong>附录</strong></a>部分，当然英文版的你的vs就有。</li>
</ol>
<p>直观的印象有了，但是离我们的短期目标，说清楚（或者说介绍个大概）CIL是What，甚至是终极目标，搞明白Mono为何能跨平台还有2万4千9百里的距离。</p>
<p>好啦，话不多说，继续乱侃。</p>
<p>参照附录中的操作码表，对照可以总结出一份更易读的表格。那就是如下的表啦。</p>
<table style="font-size: 12px; width: 101%;" border="1" cellspacing="0" cellpadding="1">
<tbody>
<tr>
<td colspan="4" align="center" width="25%" height="13">主要操作</td>
<td colspan="3" align="center" width="25%" height="13">操作数范围/条件</td>
<td colspan="3" align="center" width="25%" height="13">操作数类型</td>
<td colspan="3" align="center" width="25%" height="13">操作数</td>
</tr>
<tr>
<td align="center" width="4%" height="13">缩写</td>
<td align="center" width="7%" height="13">全称</td>
<td colspan="2" align="center" width="14%" height="13">含义</td>
<td align="center" width="4%" height="13">缩写</td>
<td align="center" width="7%" height="13">全称</td>
<td align="center" width="14%" height="13">含义</td>
<td align="center" width="4%" height="13">缩写</td>
<td align="center" width="7%" height="13">全称</td>
<td align="center" width="14%" height="13">含义</td>
<td align="center" width="4%" height="13">缩写</td>
<td align="center" width="7%" height="13">全称</td>
<td align="center" width="14%" height="13">含义</td>
</tr>
<tr>
<td rowspan="19" align="center" width="4%" height="13">ld</td>
<td rowspan="19" align="center" width="7%" height="13">load</td>
<td rowspan="19" colspan="2" align="center" width="14%" height="13">将操作数压到堆栈当中，相当于：<br />push ax</td>
<td rowspan="6" align="center" width="4%" height="4">arg</td>
<td rowspan="6" align="center" width="7%" height="4">argument</td>
<td rowspan="6" align="center" width="14%" height="4">参数</td>
<td rowspan="5" align="center" width="4%" height="2">?</td>
<td rowspan="5" align="center" width="7%" height="2">?</td>
<td rowspan="5" align="center" width="14%" height="2">操作数中的数值</td>
<td align="center" width="4%" height="1">.0</td>
<td align="center" width="7%" height="1">?</td>
<td align="center" width="14%" height="1">第零个参数&nbsp;</td>















</tr>
<tr>
<td align="center" width="4%" height="1">.1</td>
<td align="center" width="7%" height="1">?</td>
<td align="center" width="14%" height="1">第一个参数</td>















</tr>
<tr>
<td align="center" width="4%" height="0">.2</td>
<td align="center" width="7%" height="0">?　</td>
<td align="center" width="14%" height="0">第二个参数</td>















</tr>
<tr>
<td align="center" width="4%" height="0">.3</td>
<td align="center" width="7%" height="0">?</td>
<td align="center" width="14%" height="0">第三个参数</td>















</tr>
<tr>
<td align="center" width="4%" height="0">.s xx</td>
<td align="center" width="7%" height="0">(short)</td>
<td align="center" width="14%" height="0">参数xx</td>















</tr>
<tr>
<td align="center" width="4%" height="1">a</td>
<td align="center" width="7%" height="1">address</td>
<td align="center" width="14%" height="1">操作数的地址</td>
<td colspan="3" align="center" width="25%" height="1">只有 .s xx，参见ldarg.s</td>















</tr>
<tr>
<td align="center" width="4%" height="3">loc</td>
<td align="center" width="7%" height="3">local</td>
<td align="center" width="14%" height="3">局部变量</td>
<td colspan="6" align="center" width="50%" height="3">参见ldarg</td>















</tr>
<tr>
<td align="center" width="4%" height="3">fld</td>
<td align="center" width="7%" height="3">field</td>
<td align="center" width="14%" height="3">字段（类的全局变量）</td>
<td colspan="3" align="center" width="25%" height="3">参见ldarg</td>
<td align="center" width="4%" height="3">xx</td>
<td align="center" width="7%" height="3">?</td>
<td align="center" width="14%" height="3">xx字段，eg:<br />ldfld xx</td>















</tr>
<tr>
<td rowspan="10" align="center" width="4%" height="2">c</td>
<td rowspan="10" align="center" width="7%" height="2">const</td>
<td rowspan="10" align="center" width="14%" height="2">常量</td>
<td rowspan="7" align="center" width="4%" height="1">.i4</td>
<td rowspan="7" align="center" width="7%" height="1">int 4 bytes</td>
<td rowspan="7" align="center" width="14%" height="1">C#里面的int，其他的类型例如short需要通过conv转换</td>
<td align="center" width="4%" height="1">.m1</td>
<td align="center" width="7%" height="1">minus 1</td>
<td align="center" width="14%" height="1">-1</td>















</tr>
<tr>
<td align="center" width="4%" height="0">.0</td>
<td align="center" width="7%" height="0">?</td>
<td align="center" width="14%" height="0">0</td>















</tr>
<tr>
<td align="center" width="4%" height="0">.1</td>
<td align="center" width="7%" height="0">?</td>
<td align="center" width="14%" height="0">1</td>















</tr>
<tr>
<td colspan="3" align="center" width="25%" height="0">　&hellip;&hellip;</td>















</tr>
<tr>
<td align="center" width="4%" height="0">.8</td>
<td align="center" width="7%" height="0">　</td>
<td align="center" width="14%" height="0">8</td>















</tr>
<tr>
<td align="center" width="4%" height="0">.s</td>
<td align="center" width="7%" height="0">(short)</td>
<td align="center" width="14%" height="0">后面跟一个字节以内的整型数值（有符号的）</td>















</tr>
<tr>
<td align="center" width="4%" height="0">?</td>
<td align="center" width="7%" height="0">?</td>
<td align="center" width="14%" height="0">后面跟四个字节的整型数值</td>















</tr>
<tr>
<td align="center" width="4%" height="1">.i8</td>
<td align="center" width="7%" height="1">int 8 bytes</td>
<td align="center" width="14%" height="1">C#里面的long</td>
<td align="center" width="4%" height="1">?</td>
<td align="center" width="7%" height="1">?</td>
<td align="center" width="14%" height="1">后面跟八个字节的整型数值</td>















</tr>
<tr>
<td align="center" width="4%" height="0">.r4</td>
<td align="center" width="7%" height="0">real 4 bytes</td>
<td align="center" width="14%" height="0">C#里面的float</td>
<td align="center" width="4%" height="0">?</td>
<td align="center" width="7%" height="0">?</td>
<td align="center" width="14%" height="0">后面跟四个字节的浮点数值</td>















</tr>
<tr>
<td align="center" width="4%" height="0">.r8</td>
<td align="center" width="7%" height="0">real 8 bytes</td>
<td align="center" width="14%" height="0">C#里面的double</td>
<td align="center" width="4%" height="0">?</td>
<td align="center" width="7%" height="0">?</td>
<td align="center" width="14%" height="0">后面跟八个字节的浮点数值</td>















</tr>
<tr>
<td align="center" width="4%" height="2">null</td>
<td align="center" width="7%" height="2">null</td>
<td align="center" width="14%" height="2">空值（也就是0）</td>
<td align="center" width="4%" height="2">?</td>
<td align="center" width="7%" height="2">?</td>
<td align="center" width="14%" height="2">?</td>
<td align="center" width="4%" height="2">?</td>
<td align="center" width="7%" height="2">?</td>
<td align="center" width="14%" height="2">?</td>















</tr>
<tr>
<td align="center" width="4%" height="13">st</td>
<td align="center" width="7%" height="13">store</td>
<td colspan="2" align="center" width="14%" height="13">计算堆栈的顶部弹出当前值，相当于：<br />pop ax</td>
<td colspan="9" align="center" width="75%" height="13">参见ld&nbsp;</td>















</tr>
<tr>
<td rowspan="8" align="center" width="4%" height="13">conv</td>
<td rowspan="8" align="center" width="7%" height="13">convert</td>
<td rowspan="8" colspan="2" align="center" width="14%" height="13">数值类型转换，仅仅用纯粹的数值类型间的转换，例如int/float等</td>
<td rowspan="8" align="center" width="4%" height="13">?</td>
<td rowspan="8" align="center" width="7%" height="13">?</td>
<td rowspan="8" align="center" width="14%" height="13">?</td>
<td align="center" width="4%" height="2">.i1</td>
<td align="center" width="7%" height="2">int 1 bytes</td>
<td align="center" width="14%" height="2">C#里面的sbyte</td>
<td rowspan="8" align="center" width="4%" height="13">?</td>
<td rowspan="8" align="center" width="7%" height="13">?</td>
<td rowspan="8" align="center" width="14%" height="13">?</td>















</tr>
<tr>
<td align="center" width="4%" height="2">.i2</td>
<td align="center" width="7%" height="2">int 2 bytes</td>
<td align="center" width="14%" height="2">C#里面的short</td>















</tr>
<tr>
<td align="center" width="4%" height="2">.i4</td>
<td align="center" width="7%" height="2">int 4 bytes</td>
<td align="center" width="14%" height="2">C#里面的int</td>















</tr>
<tr>
<td align="center" width="4%" height="2">.i8</td>
<td align="center" width="7%" height="2">int 8 bytes</td>
<td align="center" width="14%" height="2">C#里面的long</td>















</tr>
<tr>
<td align="center" width="4%" height="2">.r4</td>
<td align="center" width="7%" height="2">real 4 bytes</td>
<td align="center" width="14%" height="2">C#里面的float</td>















</tr>
<tr>
<td align="center" width="4%" height="1">.r8</td>
<td align="center" width="7%" height="1">real 8 bytes</td>
<td align="center" width="14%" height="1">C#里面的double</td>















</tr>
<tr>
<td align="center" width="4%" height="1">.u4</td>
<td align="center" width="7%" height="1">uint 4 bytes</td>
<td align="center" width="14%" height="1">C#里面的uint</td>















</tr>
<tr>
<td align="center" width="4%" height="1">.u8</td>
<td align="center" width="7%" height="1">uint 8 bytes</td>
<td align="center" width="14%" height="1">C#里面的ulong</td>















</tr>
<tr>
<td rowspan="10" align="center" width="4%" height="13">b/br</td>
<td rowspan="10" align="center" width="7%" height="13">branch</td>
<td rowspan="10" align="center" width="12%" height="13">条件和无条件跳转，相当于：<br />jmp/jxx label_jump</td>
<td rowspan="4" align="center" width="3%" height="6">br</td>
<td rowspan="2" align="center" width="4%" height="2">?</td>
<td rowspan="2" align="center" width="7%" height="2">?</td>
<td rowspan="2" align="center" width="14%" height="2">无条件跳转</td>
<td rowspan="2" align="center" width="4%" height="2">?</td>
<td rowspan="2" align="center" width="7%" height="2">?</td>
<td rowspan="2" align="center" width="14%" height="2">?</td>
<td align="center" width="4%" height="1">?</td>
<td align="center" width="7%" height="1">?</td>
<td align="center" width="14%" height="1">后面跟四个字节的偏移量（有符号）</td>















</tr>
<tr>
<td align="center" width="4%" height="1">.s</td>
<td align="center" width="7%" height="1">(short)</td>
<td align="center" width="14%" height="1">后面跟一个字节的偏移量（有符号）</td>















</tr>
<tr>
<td align="center" width="4%" height="2">false</td>
<td align="center" width="7%" height="2">false</td>
<td align="center" width="14%" height="2">值为零的时候跳转</td>
<td align="center" width="4%" height="2">?</td>
<td align="center" width="7%" height="2">?</td>
<td align="center" width="14%" height="2">?</td>
<td rowspan="8" colspan="3" align="center" width="25%" height="11">参见br</td>















</tr>
<tr>
<td align="center" width="4%" height="2">true</td>
<td align="center" width="7%" height="2">true</td>
<td align="center" width="14%" height="2">值不为零的时候跳转</td>
<td align="center" width="4%" height="2">?</td>
<td align="center" width="7%" height="2">?</td>
<td align="center" width="14%" height="2">?</td>















</tr>
<tr>
<td rowspan="6" align="center" width="3%" height="7">b</td>
<td align="center" width="4%" height="2">eq</td>
<td align="center" width="7%" height="2">equal to</td>
<td align="center" width="14%" height="2">相等</td>
<td align="center" width="4%" height="2">?</td>
<td align="center" width="7%" height="2">?</td>
<td align="center" width="14%" height="2">?</td>















</tr>
<tr>
<td align="center" width="4%" height="1">ne</td>
<td align="center" width="7%" height="1">not equal to</td>
<td align="center" width="14%" height="1">不相等</td>
<td rowspan="5" align="center" width="4%" height="5">un</td>
<td rowspan="5" align="center" width="7%" height="5">unsigned or unordered</td>
<td rowspan="5" align="center" width="14%" height="5">无氟好的（对于整数）或者无序的（对于浮点）</td>















</tr>
<tr>
<td align="center" width="4%" height="1">gt</td>
<td align="center" width="7%" height="1">greater than</td>
<td align="center" width="14%" height="1">大于</td>















</tr>
<tr>
<td align="center" width="4%" height="1">lt</td>
<td align="center" width="7%" height="1">less than</td>
<td align="center" width="14%" height="1">小于</td>















</tr>
<tr>
<td align="center" width="4%" height="1">ge</td>
<td align="center" width="7%" height="1">greater than or equal to</td>
<td align="center" width="14%" height="1">大于等于</td>















</tr>
<tr>
<td align="center" width="4%" height="1">le</td>
<td align="center" width="7%" height="1">less than or equal to</td>
<td align="center" width="14%" height="1">小于等于</td>















</tr>
<tr>
<td rowspan="2" align="center" width="4%" height="13">call</td>
<td rowspan="2" align="center" width="7%" height="13">call</td>
<td rowspan="2" colspan="2" align="center" width="14%" height="13">调用</td>
<td align="center" width="4%" height="7">?</td>
<td align="center" width="7%" height="7">?</td>
<td align="center" width="14%" height="7">?</td>
<td align="center" width="4%" height="7">?</td>
<td align="center" width="7%" height="7">?</td>
<td align="center" width="14%" height="7">（非虚函数）</td>
<td rowspan="2" colspan="3" align="center" width="25%" height="14">?</td>















</tr>
<tr>
<td align="center" width="4%" height="6">?</td>
<td align="center" width="7%" height="6">?</td>
<td align="center" width="14%" height="6">?</td>
<td align="center" width="4%" height="7">virt</td>
<td align="center" width="7%" height="7">virtual</td>
<td align="center" width="14%" height="7">虚函数</td>















</tr>















</tbody>















</table>
<p>在此，小匹夫想请各位认真读表，然后心中默数3个数，最后看看都能发现些什么。</p>
<h4>基于堆栈</h4>
<p>如果是小匹夫的话，第一感觉就是基本每一条描述中都包含一个&rdquo;栈&ldquo;。不错,CIL是<span style="color: #ff0000;">基于堆栈<span style="color: #000000;">的，也就是说CIL的VM（mono运行时）是一个栈式机。这就意味着数据是推入堆栈，通过堆栈来操作的，而非通过CPU的寄存器来操作，这更加验证了其和具体的CPU架构没有关系。为了说明这一点，小匹夫举个例子好啦。</span></span></p>
<p><span style="color: #ff0000;"><span style="color: #000000;">大学时候学单片机（大概是8086，记不清了）的时候记得做加法大概是这样的：</span></span></p>
<div class="cnblogs_code">
<pre>add eax,-<span style="color: #800080;">2</span></pre>
</div>
<p>其中的eax是啥？寄存器。所以如果CIL处理数据要通过cpu的寄存器的话，那也就不可能和cpu的架构无关了。</p>
<p>当然，CIL之所以是基于堆栈而非CPU的另一个原因是相比较于cpu的寄存器，操作堆栈实在太简单了。回到刚才小匹夫说的大学时候曾经学过的单片机那门课程上，当时记得各种寄存器，各种标志位，各种。。。，而堆栈只需要简单的压栈和弹出，因此对于虚拟机的实现来说是再合适不过了。所以想要更具体的了解CIL基于堆栈这一点，各位可以去看一下堆栈方面的内容。这里小匹夫就不拓展了。</p>
<h4>面向对象</h4>
<p>那么第二感觉呢？貌似附录的表中有new对象的语句呀。嗯，的确，CIL同样是<span style="color: #ff0000;">面向对象</span>的。</p>
<p>这意味着什么呢？那就是在CIL中你可以创建对象，调用对象的方法，访问对象的成员。而这里需要注意的就是对方法的调用。</p>
<p>回到上表中的右上角。对，就是对参数的操作部分。静态方法和实例方法是不同的哦~</p>
<ol>
<li>静态方法：ldarg.0么有被占用，所以参数从ldarg.0开始。</li>
<li>实例方法：ldarg.0是被this占用的，也就是说实际上的参数是从ldarg.1开始的。</li>
</ol>
<p>举个例子：假设你有一个类Murong中有一个静态方法Add(int32 a, int32 b)，实现的内容就如同它的名字一样使两个数相加，所以需要2个参数。和一个实例方法TellName(string name)，这个方法会告诉你传入的名字。</p>
<div class="cnblogs_code">
<pre><span style="color: #0000ff;">class</span><span style="color: #000000;">  Murong
{
    </span><span style="color: #0000ff;">public</span> <span style="color: #0000ff;">void</span><span style="color: #000000;"> TellName(<span style="color: #3366ff;">string</span> name)
    {
        System.Console.WriteLine(name</span><span style="color: #000000;">);
    }

    </span><span style="color: #0000ff;">public</span> <span style="color: #0000ff;">static</span> <span style="color: #0000ff;">int</span> Add(<span style="color: #0000ff;">int</span> a, <span style="color: #0000ff;">int</span><span style="color: #000000;"> b)
    {
       </span><span style="color: #0000ff;">return</span> a +<span style="color: #000000;"> b;
    }
}</span></pre>
</div>
<h4>静态方法的处理：</h4>
<p>那么其中的静态方法Add的CIL代码如下：</p>
<div class="cnblogs_code">
<pre><span style="color: #008000;">//</span><span style="color: #008000;">小匹夫注释一下。</span>
.method <span style="color: #0000ff;">public</span> hidebysig <span style="color: #0000ff;">static</span><span style="color: #000000;"> int32  Add(int32 a,
                                           int32 b) cil managed
{
  </span><span style="color: #008000;">//</span><span style="color: #008000;"> 代码大小       9 (0x9)</span>
  .maxstack  <span style="color: #800080;">2</span><span style="color: #000000;">
  .locals init ([</span><span style="color: #800080;">0</span>] int32 CS$<span style="color: #800080;">1</span>$<span style="color: #800080;">0000</span><span style="color: #000000;">)   <span style="color: #339966;">//初始化局部变量列表。因为我们只返回了一个int型。所以这里声明了一个int32类型。索引为0</span>
  IL_0000:  nop
  IL_0001:  ldarg.</span><span style="color: #800080;">0</span>     <span style="color: #008000;">//</span><span style="color: #008000;">将索引为 0 的参数加载到计算堆栈上。</span>
  IL_0002:  ldarg.<span style="color: #800080;">1</span>     <span style="color: #008000;">//</span><span style="color: #008000;">将索引为 1 的参数加载到计算堆栈上。</span>
  IL_0003:  add          <span style="color: #008000;">//</span><span style="color: #008000;">计算</span>
  IL_0004:  stloc.<span style="color: #800080;">0</span>      <span style="color: #008000;">//</span><span style="color: #008000;">从计算堆栈的顶部弹出当前值并将其存储到索引 0 处的局部变量列表中。</span>
<span style="color: #000000;">  IL_0005:  br.s       IL_0007
  IL_0007:  ldloc.</span><span style="color: #800080;">0</span>     <span style="color: #008000;">//</span><span style="color: #008000;">将索引 0 处的局部变量加载到计算堆栈上。</span>
  IL_0008:  ret           <span style="color: #008000;">//</span><span style="color: #008000;">返回该值</span>
} <span style="color: #008000;">//</span><span style="color: #008000;"> end of method Murong::Add</span></pre>
</div>
<p>那么我们调用这个静态函数应该就是这样咯。</p>
<div class="cnblogs_code">
<pre>Murong.Add(<span style="color: #800080;">1</span>, <span style="color: #800080;">2</span>);</pre>
</div>
<p>对应的CIL代码为：</p>
<div class="cnblogs_code">
<pre>  IL_0001:  ldc.i4.<span style="color: #800080;">1 //将整数1压入栈中</span><span style="color: #000000;">
  IL_0002:  ldc.i4.</span><span style="color: #800080;">2 //将整数2压入栈中</span><span style="color: #000000;">
  IL_0003:  call       int32 Murong::Add(int32,
                                         int32)  //调用静态方法</span></pre>
</div>
<p>可见CIL直接call了Murong的Add方法，而不需要一个Murong的实例。</p>
<h4><span style="color: #000000;">实例方法的处理：</span></h4>
<p><span style="color: #000000;">Murong类中的实例方法TellName()的CIL代码如下：</span></p>
<div class="cnblogs_code">
<pre>.method <span style="color: #0000ff;">public</span> hidebysig instance <span style="color: #0000ff;">void</span>  TellName(<span style="color: #0000ff;">string</span><span style="color: #000000;"> name) cil managed
{
  </span><span style="color: #008000;">//</span><span style="color: #008000;"> 代码大小       9 (0x9)</span>
  .maxstack  <span style="color: #800080;">8</span><span style="color: #000000;">
  IL_0000:  nop
  IL_0001:  ldarg.</span><span style="color: #800080;">1</span>     <span style="color: #008000;">//</span><span style="color: #008000;">看到和静态方法的区别了吗？</span>
  IL_0002:  call       <span style="color: #0000ff;">void</span> [mscorlib]System.Console::WriteLine(<span style="color: #0000ff;">string</span><span style="color: #000000;">)
  IL_0007:  nop
  IL_0008:  ret
} </span><span style="color: #008000;">//</span><span style="color: #008000;"> end of method Murong::TellName</span></pre>
</div>
<p>看到和静态方法的区别了吗？对，第一个参数对应的是ldarg.1中的参数1，而不是静态方法中的0。因为此时参数0相当于this，this是不用参与参数传递的。</p>
<p><span style="color: #000000;">那么我们再看看调用实例方法的C#代码和对应的CIL代码是如何的。</span></p>
<div class="cnblogs_code">
<pre>//C#<br />Murong murong = <span style="color: #0000ff;">new</span><span style="color: #000000;"> Murong();
murong.TellName(</span><span style="color: #800000;">"</span><span style="color: #800000;">chenjiadong</span><span style="color: #800000;">"</span>);</pre>
</div>
<p>CIL：</p>
<div class="cnblogs_code">
<pre>.locals init ([<span style="color: #800080;">0</span>] <span style="color: #0000ff;">class</span> Murong murong)   <span style="color: #008000;">//</span><span style="color: #008000;">因为C#代码中定义了一个Murong类型的变量，所以局部变量列表的索引0为该类型的引用。
</span><span style="color: #008000;">//</span><span style="color: #008000;">....</span>
IL_0009:  newobj     instance <span style="color: #0000ff;">void</span> Murong::.ctor() <span style="color: #008000;">//</span><span style="color: #008000;">相比上面的静态方法的调用，此处new一个新对象，出现了instance方法。</span>
IL_000e:  stloc.<span style="color: #800080;">0</span><span style="color: #000000;">
IL_000f:  ldloc.</span><span style="color: #800080;">0</span><span style="color: #000000;">
IL_0010:  ldstr      </span><span style="color: #800000;">"</span><span style="color: #800000;">chenjiadong</span><span style="color: #800000;">"</span> <span style="color: #008000;">//</span><span style="color: #008000;">小匹夫的名字入栈</span>
IL_0015:  callvirt   instance <span style="color: #0000ff;">void</span> Murong::TellName(<span style="color: #0000ff;">string</span>) <span style="color: #008000;">//</span><span style="color: #008000;">实例方法的调用也有instance</span></pre>
</div>
<p>到此，受制于篇幅所限（小匹夫不想写那么多字啊啊啊！）CIL是What的问题大致介绍一下。当然没有再拓展，以后小匹夫可能会再详细写一下这块。</p>
<h2><span style="line-height: 1.5;"><a name="how"></a>How？</span></h2>
<p><span style="line-height: 1.5;">记得语文老师说过，写作文最重要的一点是要首尾呼应。既然咱们开篇就提出了U3D为何能跨平台的问题，那么接近文章的结尾咱们就再来</span></p>
<p><span style="font-size: 16px;"><strong><span style="line-height: 1.5;">提问：</span></strong></span></p>
<p>Q:上面的Why部分，咱们知道了U3D能跨平台是因为存在着一个能通吃的中间语言CIL，这也是所谓跨平台的前提，但是为啥CIL能通吃各大平台呢？当然可以说CIL基于堆栈，跟你CPU怎么架构的没啥关系，但是感觉过于理论化、学术化，那还有没有通俗化、工程化的说法呢？</p>
<p>A:原因就是前面小匹夫提到过的，.Net运行时和Mono运行时。也就是说CIL语言其实是运行在虚拟机中的，具体到咱们的U3D也就是mono的运行时了，换言之mono运行的其实CIL语言，CIL也并非真正的在本地运行，而是在mono运行时中运行的，运行在本地的是被编译后生成的原生代码。当然看官博的文章，他们似乎也在开发自己的&ldquo;mono&rdquo;，也就是被称为脚本的未来的IL2Cpp，这种类似运行时的功能是将IL再编译成c++，再由c++编译成原生代码，据说效率提升很可观，小匹夫也是蛮期待的。</p>
<p>这里为了&ldquo;实现跨平台式的演示&rdquo;，小匹夫用mac给各位做个测试好啦：</p>
<h4>从C#到CIL</h4>
<p>新建一个cs文件，然后使用mono来运行。这个cs文件内容如下：</p>
<p><img src="http://images.cnitblog.com/blog/686199/201501/100034397038251.png" alt="" /></p>
<p>然后咱们直接在命令行中运行这个cs文件试试~</p>
<p><img src="http://images.cnitblog.com/blog/686199/201501/100035575009350.png" alt="" /></p>
<p>说的很清楚，文件没有包含一个CIL映像。可见mono是不能直接运行cs文件的。假如我们把它编译成CIL呢？那么我们用mono带的mcs来编译小匹夫的Test.cs文件。</p>
<div class="cnblogs_code">
<pre>mcs Test.cs</pre>
</div>
<p>生成了什么呢？如图：</p>
<p><img src="http://images.cnitblog.com/blog/686199/201501/100039109378522.png" alt="" /></p>
<p>好像没见有叫.IL的文件生成啊？反而好像多了一个.exe文件？可是没听说Mac能运行exe文件呀？可为啥又生成了.exe呢？各位看官可能要说，小匹夫你是不是拿windows截图P的啊？嘿嘿，小匹夫可不敢。辣么真相其实就是这个exe并不是让Mac来运行的，而是留给mono运行时来运行的，换言之这个文件的可执行代码形式是CIL的位元码形态。到此，我们完成了从C#到CIL的过程。接下来就让我们运行下刚刚的成果好啦。</p>
<div class="cnblogs_Highlighter">
<pre class="brush:csharp;gutter:true;">mono Test.exe
</pre>
</div>
<p>&nbsp;<img src="http://images.cnitblog.com/blog/686199/201501/100045087653170.png" alt="" /></p>
<p>结果是输出了一个大大的&ldquo;Hi&rdquo;。这里，就引出了下一个部分。</p>
<h4>从CIL到Native Code</h4>
<p>这个&ldquo;HI&rdquo;可是在小匹夫的MAC终端上出现的呀，那么就证明这个C#写的代码在MAC上运行的还挺&ldquo;嗨&rdquo;。</p>
<p>为啥呢？为啥C#写的代码能跑在MAC上呢？这就不得不提从CIL如何到本机原生代码的过程了。Mono提供了两种编译方式，就是我们经常能看到的：JIT（Just-in-Time compilation，即时编译）和AOT（Ahead-of-Time，提前编译或静态编译）。这两种方式都是将CIL进一步编译成平台的原生代码。这也是实现跨平台的最后一步。下面就分头介绍一下。</p>
<h4>JIT即时编译：</h4>
<p>从名字就能看的出来，即时编译，或者称之为动态编译，是在程序执行时才编译代码，解释一条语句执行一条语句，即将一条中间的托管的语句翻译成一条机器语句，然后执行这条机器语句。但同时也会将编译过的代码进行缓存，而不是每一次都进行编译。所以可以说它是静态编译和解释器的结合体。不过你想想机器既要处理代码的逻辑，同时还要进行编译的工作，所以其运行时的效率肯定是受到影响的。因此，Mono会有一部分代码通过AOT静态编译，以降低在程序运行时JIT动态编译在效率上的问题。</p>
<p>不过一向严苛的IOS平台是不允许这种动态的编译方式的，这也是U3D官方无法给出热更新方案的一个原因。而Android平台恰恰相反，Dalvik虚拟机使用的就是JIT方案。</p>
<h4>AOT静态编译：</h4>
<p>其实Mono的AOT静态编译和JIT并非对立的。AOT同样使用了JIT来进行编译，只不过是被AOT编译的代码在程序运行之前就已经编译好了。当然还有一部分代码会通过JIT来进行动态编译。下面小匹夫就手动操作一下mono，让它进行一次AOT编译。</p>
<div class="cnblogs_code">
<pre><span style="color: #008000;">//</span><span style="color: #008000;">在命令行输入</span>
mono --aot Test.exe</pre>
</div>
<p>结果：<img src="http://images.cnitblog.com/blog/686199/201501/110157240469330.png" alt="" /></p>
<p>从图中可以看到JIT time： 39 ms,也就是说Mono的AOT模式其实会使用到JIT，同时我们看到了生成了一个适应小匹夫的MAC的动态库Test.exe.dylib，而在Linux生成就是.so（共享库）。</p>
<p>AOT编译出来的库，除了包括我们的代码之外，还有被缓存的元数据信息。所以我们甚至可以只编译元数据信息而不变异代码。例如这样：</p>
<div class="cnblogs_code">
<pre><span style="color: #008000;">//</span><span style="color: #008000;">只包含元数据的信息</span>
mono --aot=metadata-only Test.exe</pre>
</div>
<p><img src="http://images.cnitblog.com/blog/686199/201501/110248517655449.png" alt="" /></p>
<p>可见代码没有被包括进来。</p>
<p>那么简单总结一下AOT的过程：</p>
<ol>
<li>收集要被编译的方法</li>
<li>使用JIT进行编译</li>
<li>发射（Emitting）经JIT编译过的代码和其他信息</li>
<li>直接生成文件或者调用本地汇编器或连接器进行处理之后生成文件。（例如上图中使用了小匹夫本地的gcc)</li>
</ol>
<h4>Full AOT</h4>
<p>当然上文也说了，IOS平台是禁止使用JIT的，可看样子Mono的AOT模式仍然会保留一部分代码会在程序运行时动态编译。所以为了破解这个问题，Mono提供了一个被称为Full AOT的模式。即预先对程序集中的所有CIL代码进行AOT编译生成一个本地代码映像，然后在运行时直接加载这个映像而不再使用JIT引擎。目前由于技术或实现上的原因在使用Full AOT时有一些限制，不过这里不再多说了。以后也还会更细的分析下AOT。</p>
<p>&nbsp;</p>
<h4>总结：</h4>
<p>好啦，写到现在也已经到了凌晨3：04分了。感觉写的内容也差不多了。那么对本文的主题U3D为何能跨平台以及CIL做个最终的总结陈词：</p>
<ol>
<li>CIL是CLI标准定义的一种可读性较低的语言。</li>
<li>以.NET或mono等实现CLI标准的运行环境为目标的语言要先编译成CIL，之后CIL会被编译，并且以位元码的形式存在（源代码---&gt;中间语言的过程）。</li>
<li>这种位元码运行在虚拟机中(.net mono的运行时）。</li>
<li>这种位元码可以被进一步编译成不同平台的原生代码（中间语言---&gt;原生代码的过程）。</li>
<li>面向对象</li>
<li>基于堆栈</li>
</ol>

<h3>装模作样的<span style="color: #ff0000;"><strong>声明</strong>一下：本博文章若非特殊注明皆为<span style="color: #800000;"><span style="color: #3366ff;">原创，<span style="color: #000000;">若需转载请保留<a href="http://www.cnblogs.com/murongxiaopifu/p/4211964.html" target="_blank">原文链接</a>（<a id="Editor_Edit_hlEntryLink" title="view: 匹夫细说Unity3D(二)&mdash;&mdash;为何能跨平台？聊聊CIL(MSIL)" href="http://www.cnblogs.com/murongxiaopifu/p/4211964.html" target="_blank">http://www.cnblogs.com/murongxiaopifu/p/4211964.html</a>）<span style="color: #800080;"><span style="color: #000000;">及作者<span style="color: #ff6600;"><span style="color: #000000;">信息<a href="http://www.cnblogs.com/murongxiaopifu" target="_blank">慕容小匹夫</a></span></span></span></span></span></span></span></span></h3>
<p><span style="font-size: 15px;"><strong><span style="color: #ff0000;"><span style="color: #800000;"><span style="color: #3366ff;"><span style="color: #000000;"><span style="color: #800080;"><span style="color: #000000;"><span style="color: #ff6600;"><span style="color: #000000;">你也可以在游戏蛮牛读到这篇文章：</span></span></span></span></span></span></span></span></strong></span></p>
<p><span style="font-size: 15px;"><strong><span style="color: #ff0000;"><span style="color: #800000;"><span style="color: #3366ff;"><span style="color: #000000;"><span style="color: #800080;"><span style="color: #000000;"><span style="color: #ff6600;"><span style="color: #000000;"><a href="http://www.unitymanual.com/thread-36240-1-1.html" target="_blank">匹夫细说Unity3D(三)&mdash;&mdash;为何能跨平台？聊聊CIL(MSIL)</a><br />(出处: -u3d游戏开发者社区【游戏蛮牛】unity3d官网)</span></span></span></span></span></span></span></span></strong></span></p>
<h2><a name="fulu"></a>附录：</h2>
<div>
<table style="width: 1029px;" border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr style="background-color: #ffff99;">
<td align="center" width="114">名称</td>
<td align="center" width="913">说明</td>


</tr>
<tr>
<td>Add</td>
<td width="913">将两个值相加并将结果推送到计算堆栈上。</td>


</tr>
<tr>
<td>Add.Ovf</td>
<td width="913">将两个整数相加，执行溢出检查，并且将结果推送到计算堆栈上。</td>


</tr>
<tr>
<td>Add.Ovf.Un</td>
<td width="913">将两个无符号整数值相加，执行溢出检查，并且将结果推送到计算堆栈上。</td>


</tr>
<tr>
<td>And</td>
<td width="913">计算两个值的按位&ldquo;与&rdquo;并将结果推送到计算堆栈上。</td>


</tr>
<tr>
<td>Arglist</td>
<td width="913">返回指向当前方法的参数列表的非托管指针。</td>


</tr>
<tr>
<td>Beq</td>
<td width="913">如果两个值相等，则将控制转移到目标指令。</td>


</tr>
<tr>
<td>Beq.S</td>
<td width="913">如果两个值相等，则将控制转移到目标指令（短格式）。</td>


</tr>
<tr>
<td>Bge</td>
<td width="913">如果第一个值大于或等于第二个值，则将控制转移到目标指令。</td>


</tr>
<tr>
<td>Bge.S</td>
<td width="913">如果第一个值大于或等于第二个值，则将控制转移到目标指令（短格式）。</td>


</tr>
<tr>
<td>Bge.Un</td>
<td width="913">当比较无符号整数值或不可排序的浮点型值时，如果第一个值大于第二个值，则将控制转移到目标指令。</td>


</tr>
<tr>
<td>Bge.Un.S</td>
<td width="913">当比较无符号整数值或不可排序的浮点型值时，如果第一个值大于第二个值，则将控制转移到目标指令（短格式）。</td>


</tr>
<tr>
<td>Bgt</td>
<td width="913">如果第一个值大于第二个值，则将控制转移到目标指令。</td>


</tr>
<tr>
<td>Bgt.S</td>
<td width="913">如果第一个值大于第二个值，则将控制转移到目标指令（短格式）。</td>


</tr>
<tr>
<td>Bgt.Un</td>
<td width="913">当比较无符号整数值或不可排序的浮点型值时，如果第一个值大于第二个值，则将控制转移到目标指令。</td>


</tr>
<tr>
<td>Bgt.Un.S</td>
<td width="913">当比较无符号整数值或不可排序的浮点型值时，如果第一个值大于第二个值，则将控制转移到目标指令（短格式）。</td>


</tr>
<tr>
<td>Ble</td>
<td width="913">如果第一个值小于或等于第二个值，则将控制转移到目标指令。</td>


</tr>
<tr>
<td>Ble.S</td>
<td width="913">如果第一个值小于或等于第二个值，则将控制转移到目标指令（短格式）。</td>


</tr>
<tr>
<td>Ble.Un</td>
<td width="913">当比较无符号整数值或不可排序的浮点型值时，如果第一个值小于或等于第二个值，则将控制转移到目标指令。</td>


</tr>
<tr>
<td>Ble.Un.S</td>
<td width="913">当比较无符号整数值或不可排序的浮点值时，如果第一个值小于或等于第二个值，则将控制权转移到目标指令（短格式）。</td>


</tr>
<tr>
<td>Blt</td>
<td width="913">如果第一个值小于第二个值，则将控制转移到目标指令。</td>


</tr>
<tr>
<td>Blt.S</td>
<td width="913">如果第一个值小于第二个值，则将控制转移到目标指令（短格式）。</td>


</tr>
<tr>
<td>Blt.Un</td>
<td width="913">当比较无符号整数值或不可排序的浮点型值时，如果第一个值小于第二个值，则将控制转移到目标指令。</td>


</tr>
<tr>
<td>Blt.Un.S</td>
<td width="913">当比较无符号整数值或不可排序的浮点型值时，如果第一个值小于第二个值，则将控制转移到目标指令（短格式）。</td>


</tr>
<tr>
<td>Bne.Un</td>
<td width="913">当两个无符号整数值或不可排序的浮点型值不相等时，将控制转移到目标指令。</td>


</tr>
<tr>
<td>Bne.Un.S</td>
<td width="913">当两个无符号整数值或不可排序的浮点型值不相等时，将控制转移到目标指令（短格式）。</td>


</tr>
<tr>
<td>Box</td>
<td width="913">将值类转换为对象引用（O 类型）。</td>


</tr>
<tr>
<td>Br</td>
<td width="913">无条件地将控制转移到目标指令。</td>


</tr>
<tr>
<td>Br.S</td>
<td width="913">无条件地将控制转移到目标指令（短格式）。</td>


</tr>
<tr>
<td>Break</td>
<td width="913">向公共语言结构 (CLI) 发出信号以通知调试器已撞上了一个断点。</td>


</tr>
<tr>
<td>Brfalse</td>
<td width="913">如果 value 为 false、空引用（Visual Basic 中的 Nothing）或零，则将控制转移到目标指令。</td>


</tr>
<tr>
<td>Brfalse.S</td>
<td width="913">如果 value 为 false、空引用或零，则将控制转移到目标指令。</td>


</tr>
<tr>
<td>Brtrue</td>
<td width="913">如果 value 为 true、非空或非零，则将控制转移到目标指令。</td>


</tr>
<tr>
<td>Brtrue.S</td>
<td width="913">如果 value 为 true、非空或非零，则将控制转移到目标指令（短格式）。</td>


</tr>
<tr>
<td>Call</td>
<td width="913">调用由传递的方法说明符指示的方法。</td>


</tr>
<tr>
<td>Calli</td>
<td width="913">通过调用约定描述的参数调用在计算堆栈上指示的方法（作为指向入口点的指针）。</td>


</tr>
<tr>
<td>Callvirt</td>
<td width="913">对对象调用后期绑定方法，并且将返回值推送到计算堆栈上。</td>


</tr>
<tr>
<td>Castclass</td>
<td width="913">尝试将引用传递的对象转换为指定的类。</td>


</tr>
<tr>
<td>Ceq</td>
<td width="913">比较两个值。如果这两个值相等，则将整数值 1 (int32) 推送到计算堆栈上；否则，将 0 (int32) 推送到计算堆栈上。</td>


</tr>
<tr>
<td>Cgt</td>
<td width="913">比较两个值。如果第一个值大于第二个值，则将整数值 1 (int32) 推送到计算堆栈上；反之，将 0 (int32) 推送到计算堆栈上。</td>


</tr>
<tr>
<td>Cgt.Un</td>
<td width="913">比较两个无符号的或不可排序的值。如果第一个值大于第二个值，则将整数值 1 (int32) 推送到计算堆栈上；反之，将 0 (int32) 推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ckfinite</td>
<td width="913">如果值不是有限数，则引发 ArithmeticException。</td>


</tr>
<tr>
<td>Clt</td>
<td width="913">比较两个值。如果第一个值小于第二个值，则将整数值 1 (int32) 推送到计算堆栈上；反之，将 0 (int32) 推送到计算堆栈上。</td>


</tr>
<tr>
<td>Clt.Un</td>
<td width="913">比较无符号的或不可排序的值 value1 和 value2。如果 value1 小于 value2，则将整数值 1 (int32 ) 推送到计算堆栈上；反之，将 0 ( int32 ) 推送到计算堆栈上。</td>


</tr>
<tr>
<td>Constrained</td>
<td width="913">约束要对其进行虚方法调用的类型。</td>


</tr>
<tr>
<td>Conv.I</td>
<td width="913">将位于计算堆栈顶部的值转换为 native int。</td>


</tr>
<tr>
<td>Conv.I1</td>
<td width="913">将位于计算堆栈顶部的值转换为 int8，然后将其扩展（填充）为 int32。</td>


</tr>
<tr>
<td>Conv.I2</td>
<td width="913">将位于计算堆栈顶部的值转换为 int16，然后将其扩展（填充）为 int32。</td>


</tr>
<tr>
<td>Conv.I4</td>
<td width="913">将位于计算堆栈顶部的值转换为 int32。</td>


</tr>
<tr>
<td>Conv.I8</td>
<td width="913">将位于计算堆栈顶部的值转换为 int64。</td>


</tr>
<tr>
<td>Conv.Ovf.I</td>
<td width="913">将位于计算堆栈顶部的有符号值转换为有符号 native int，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.Ovf.I.Un</td>
<td width="913">将位于计算堆栈顶部的无符号值转换为有符号 native int，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.Ovf.I1</td>
<td width="913">将位于计算堆栈顶部的有符号值转换为有符号 int8 并将其扩展为 int32，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.Ovf.I1.Un</td>
<td width="913">将位于计算堆栈顶部的无符号值转换为有符号 int8 并将其扩展为 int32，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.Ovf.I2</td>
<td width="913">将位于计算堆栈顶部的有符号值转换为有符号 int16 并将其扩展为 int32，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.Ovf.I2.Un</td>
<td width="913">将位于计算堆栈顶部的无符号值转换为有符号 int16 并将其扩展为 int32，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.Ovf.I4</td>
<td width="913">将位于计算堆栈顶部的有符号值转换为有符号 int32，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.Ovf.I4.Un</td>
<td width="913">将位于计算堆栈顶部的无符号值转换为有符号 int32，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.Ovf.I8</td>
<td width="913">将位于计算堆栈顶部的有符号值转换为有符号 int64，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.Ovf.I8.Un</td>
<td width="913">将位于计算堆栈顶部的无符号值转换为有符号 int64，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.Ovf.U</td>
<td width="913">将位于计算堆栈顶部的有符号值转换为 unsigned native int，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.Ovf.U.Un</td>
<td width="913">将位于计算堆栈顶部的无符号值转换为 unsigned native int，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.Ovf.U1</td>
<td width="913">将位于计算堆栈顶部的有符号值转换为 unsigned int8 并将其扩展为 int32，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.Ovf.U1.Un</td>
<td width="913">将位于计算堆栈顶部的无符号值转换为 unsigned int8 并将其扩展为 int32，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.Ovf.U2</td>
<td width="913">将位于计算堆栈顶部的有符号值转换为 unsigned int16 并将其扩展为 int32，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.Ovf.U2.Un</td>
<td width="913">将位于计算堆栈顶部的无符号值转换为 unsigned int16 并将其扩展为 int32，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.Ovf.U4</td>
<td width="913">将位于计算堆栈顶部的有符号值转换为 unsigned int32，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.Ovf.U4.Un</td>
<td width="913">将位于计算堆栈顶部的无符号值转换为 unsigned int32，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.Ovf.U8</td>
<td width="913">将位于计算堆栈顶部的有符号值转换为 unsigned int64，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.Ovf.U8.Un</td>
<td width="913">将位于计算堆栈顶部的无符号值转换为 unsigned int64，并在溢出时引发 OverflowException。</td>


</tr>
<tr>
<td>Conv.R.Un</td>
<td width="913">将位于计算堆栈顶部的无符号整数值转换为 float32。</td>


</tr>
<tr>
<td>Conv.R4</td>
<td width="913">将位于计算堆栈顶部的值转换为 float32。</td>


</tr>
<tr>
<td>Conv.R8</td>
<td width="913">将位于计算堆栈顶部的值转换为 float64。</td>


</tr>
<tr>
<td>Conv.U</td>
<td width="913">将位于计算堆栈顶部的值转换为 unsigned native int，然后将其扩展为 native int。</td>


</tr>
<tr>
<td>Conv.U1</td>
<td width="913">将位于计算堆栈顶部的值转换为 unsigned int8，然后将其扩展为 int32。</td>


</tr>
<tr>
<td>Conv.U2</td>
<td width="913">将位于计算堆栈顶部的值转换为 unsigned int16，然后将其扩展为 int32。</td>


</tr>
<tr>
<td>Conv.U4</td>
<td width="913">将位于计算堆栈顶部的值转换为 unsigned int32，然后将其扩展为 int32。</td>


</tr>
<tr>
<td>Conv.U8</td>
<td width="913">将位于计算堆栈顶部的值转换为 unsigned int64，然后将其扩展为 int64。</td>


</tr>
<tr>
<td>Cpblk</td>
<td width="913">将指定数目的字节从源地址复制到目标地址。</td>


</tr>
<tr>
<td>Cpobj</td>
<td width="913">将位于对象（&amp;、* 或 native int 类型）地址的值类型复制到目标对象（&amp;、* 或 native int 类型）的地址。</td>


</tr>
<tr>
<td>Div</td>
<td width="913">将两个值相除并将结果作为浮点（F 类型）或商（int32 类型）推送到计算堆栈上。</td>


</tr>
<tr>
<td>Div.Un</td>
<td width="913">两个无符号整数值相除并将结果 ( int32 ) 推送到计算堆栈上。</td>


</tr>
<tr>
<td>Dup</td>
<td width="913">复制计算堆栈上当前最顶端的值，然后将副本推送到计算堆栈上。</td>


</tr>
<tr>
<td>Endfilter</td>
<td width="913">将控制从异常的 filter 子句转移回公共语言结构 (CLI) 异常处理程序。</td>


</tr>
<tr>
<td>Endfinally</td>
<td width="913">将控制从异常块的 fault 或 finally 子句转移回公共语言结构 (CLI) 异常处理程序。</td>


</tr>
<tr>
<td>Initblk</td>
<td width="913">将位于特定地址的内存的指定块初始化为给定大小和初始值。</td>


</tr>
<tr>
<td>Initobj</td>
<td width="913">将位于指定地址的值类型的每个字段初始化为空引用或适当的基元类型的 0。</td>


</tr>
<tr>
<td>Isinst</td>
<td width="913">测试对象引用（O 类型）是否为特定类的实例。</td>


</tr>
<tr>
<td>Jmp</td>
<td width="913">退出当前方法并跳至指定方法。</td>


</tr>
<tr>
<td>Ldarg</td>
<td width="913">将参数（由指定索引值引用）加载到堆栈上。</td>


</tr>
<tr>
<td>Ldarg.0</td>
<td width="913">将索引为 0 的参数加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldarg.1</td>
<td width="913">将索引为 1 的参数加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldarg.2</td>
<td width="913">将索引为 2 的参数加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldarg.3</td>
<td width="913">将索引为 3 的参数加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldarg.S</td>
<td width="913">将参数（由指定的短格式索引引用）加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldarga</td>
<td width="913">将参数地址加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldarga.S</td>
<td width="913">以短格式将参数地址加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldc.I4</td>
<td width="913">将所提供的 int32 类型的值作为 int32 推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldc.I4.0</td>
<td width="913">将整数值 0 作为 int32 推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldc.I4.1</td>
<td width="913">将整数值 1 作为 int32 推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldc.I4.2</td>
<td width="913">将整数值 2 作为 int32 推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldc.I4.3</td>
<td width="913">将整数值 3 作为 int32 推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldc.I4.4</td>
<td width="913">将整数值 4 作为 int32 推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldc.I4.5</td>
<td width="913">将整数值 5 作为 int32 推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldc.I4.6</td>
<td width="913">将整数值 6 作为 int32 推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldc.I4.7</td>
<td width="913">将整数值 7 作为 int32 推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldc.I4.8</td>
<td width="913">将整数值 8 作为 int32 推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldc.I4.M1</td>
<td width="913">将整数值 -1 作为 int32 推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldc.I4.S</td>
<td width="913">将提供的 int8 值作为 int32 推送到计算堆栈上（短格式）。</td>


</tr>
<tr>
<td>Ldc.I8</td>
<td width="913">将所提供的 int64 类型的值作为 int64 推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldc.R4</td>
<td width="913">将所提供的 float32 类型的值作为 F (float) 类型推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldc.R8</td>
<td width="913">将所提供的 float64 类型的值作为 F (float) 类型推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldelem</td>
<td width="913">按照指令中指定的类型，将指定数组索引中的元素加载到计算堆栈的顶部。</td>


</tr>
<tr>
<td>Ldelem.I</td>
<td width="913">将位于指定数组索引处的 native int 类型的元素作为 native int 加载到计算堆栈的顶部。</td>


</tr>
<tr>
<td>Ldelem.I1</td>
<td width="913">将位于指定数组索引处的 int8 类型的元素作为 int32 加载到计算堆栈的顶部。</td>


</tr>
<tr>
<td>Ldelem.I2</td>
<td width="913">将位于指定数组索引处的 int16 类型的元素作为 int32 加载到计算堆栈的顶部。</td>


</tr>
<tr>
<td>Ldelem.I4</td>
<td width="913">将位于指定数组索引处的 int32 类型的元素作为 int32 加载到计算堆栈的顶部。</td>


</tr>
<tr>
<td>Ldelem.I8</td>
<td width="913">将位于指定数组索引处的 int64 类型的元素作为 int64 加载到计算堆栈的顶部。</td>


</tr>
<tr>
<td>Ldelem.R4</td>
<td width="913">将位于指定数组索引处的 float32 类型的元素作为 F 类型（浮点型）加载到计算堆栈的顶部。</td>


</tr>
<tr>
<td>Ldelem.R8</td>
<td width="913">将位于指定数组索引处的 float64 类型的元素作为 F 类型（浮点型）加载到计算堆栈的顶部。</td>


</tr>
<tr>
<td>Ldelem.Ref</td>
<td width="913">将位于指定数组索引处的包含对象引用的元素作为 O 类型（对象引用）加载到计算堆栈的顶部。</td>


</tr>
<tr>
<td>Ldelem.U1</td>
<td width="913">将位于指定数组索引处的 unsigned int8 类型的元素作为 int32 加载到计算堆栈的顶部。</td>


</tr>
<tr>
<td>Ldelem.U2</td>
<td width="913">将位于指定数组索引处的 unsigned int16 类型的元素作为 int32 加载到计算堆栈的顶部。</td>


</tr>
<tr>
<td>Ldelem.U4</td>
<td width="913">将位于指定数组索引处的 unsigned int32 类型的元素作为 int32 加载到计算堆栈的顶部。</td>


</tr>
<tr>
<td>Ldelema</td>
<td width="913">将位于指定数组索引的数组元素的地址作为 &amp; 类型（托管指针）加载到计算堆栈的顶部。</td>


</tr>
<tr>
<td>Ldfld</td>
<td width="913">查找对象中其引用当前位于计算堆栈的字段的值。</td>


</tr>
<tr>
<td>Ldflda</td>
<td width="913">查找对象中其引用当前位于计算堆栈的字段的地址。</td>


</tr>
<tr>
<td>Ldftn</td>
<td width="913">将指向实现特定方法的本机代码的非托管指针（native int 类型）推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldind.I</td>
<td width="913">将 native int 类型的值作为 native int 间接加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldind.I1</td>
<td width="913">将 int8 类型的值作为 int32 间接加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldind.I2</td>
<td width="913">将 int16 类型的值作为 int32 间接加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldind.I4</td>
<td width="913">将 int32 类型的值作为 int32 间接加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldind.I8</td>
<td width="913">将 int64 类型的值作为 int64 间接加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldind.R4</td>
<td width="913">将 float32 类型的值作为 F (float) 类型间接加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldind.R8</td>
<td width="913">将 float64 类型的值作为 F (float) 类型间接加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldind.Ref</td>
<td width="913">将对象引用作为 O（对象引用）类型间接加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldind.U1</td>
<td width="913">将 unsigned int8 类型的值作为 int32 间接加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldind.U2</td>
<td width="913">将 unsigned int16 类型的值作为 int32 间接加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldind.U4</td>
<td width="913">将 unsigned int32 类型的值作为 int32 间接加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldlen</td>
<td width="913">将从零开始的、一维数组的元素的数目推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldloc</td>
<td width="913">将指定索引处的局部变量加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldloc.0</td>
<td width="913">将索引 0 处的局部变量加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldloc.1</td>
<td width="913">将索引 1 处的局部变量加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldloc.2</td>
<td width="913">将索引 2 处的局部变量加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldloc.3</td>
<td width="913">将索引 3 处的局部变量加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldloc.S</td>
<td width="913">将特定索引处的局部变量加载到计算堆栈上（短格式）。</td>


</tr>
<tr>
<td>Ldloca</td>
<td width="913">将位于特定索引处的局部变量的地址加载到计算堆栈上。</td>


</tr>
<tr>
<td>Ldloca.S</td>
<td width="913">将位于特定索引处的局部变量的地址加载到计算堆栈上（短格式）。</td>


</tr>
<tr>
<td>Ldnull</td>
<td width="913">将空引用（O 类型）推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldobj</td>
<td width="913">将地址指向的值类型对象复制到计算堆栈的顶部。</td>


</tr>
<tr>
<td>Ldsfld</td>
<td width="913">将静态字段的值推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldsflda</td>
<td width="913">将静态字段的地址推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldstr</td>
<td width="913">推送对元数据中存储的字符串的新对象引用。</td>


</tr>
<tr>
<td>Ldtoken</td>
<td width="913">将元数据标记转换为其运行时表示形式，并将其推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ldvirtftn</td>
<td width="913">将指向实现与指定对象关联的特定虚方法的本机代码的非托管指针（native int 类型）推送到计算堆栈上。</td>


</tr>
<tr>
<td>Leave</td>
<td width="913">退出受保护的代码区域，无条件将控制转移到特定目标指令。</td>


</tr>
<tr>
<td>Leave.S</td>
<td width="913">退出受保护的代码区域，无条件将控制转移到目标指令（缩写形式）。</td>


</tr>
<tr>
<td>Localloc</td>
<td width="913">从本地动态内存池分配特定数目的字节并将第一个分配的字节的地址（瞬态指针，* 类型）推送到计算堆栈上。</td>


</tr>
<tr>
<td>Mkrefany</td>
<td width="913">将对特定类型实例的类型化引用推送到计算堆栈上。</td>


</tr>
<tr>
<td>Mul</td>
<td width="913">将两个值相乘并将结果推送到计算堆栈上。</td>


</tr>
<tr>
<td>Mul.Ovf</td>
<td width="913">将两个整数值相乘，执行溢出检查，并将结果推送到计算堆栈上。</td>


</tr>
<tr>
<td>Mul.Ovf.Un</td>
<td width="913">将两个无符号整数值相乘，执行溢出检查，并将结果推送到计算堆栈上。</td>


</tr>
<tr>
<td>Neg</td>
<td width="913">对一个值执行求反并将结果推送到计算堆栈上。</td>


</tr>
<tr>
<td>Newarr</td>
<td width="913">将对新的从零开始的一维数组（其元素属于特定类型）的对象引用推送到计算堆栈上。</td>


</tr>
<tr>
<td>Newobj</td>
<td width="913">创建一个值类型的新对象或新实例，并将对象引用（O 类型）推送到计算堆栈上。</td>


</tr>
<tr>
<td>Nop</td>
<td width="913">如果修补操作码，则填充空间。尽管可能消耗处理周期，但未执行任何有意义的操作。</td>


</tr>
<tr>
<td>Not</td>
<td width="913">计算堆栈顶部整数值的按位求补并将结果作为相同的类型推送到计算堆栈上。</td>


</tr>
<tr>
<td>Or</td>
<td width="913">计算位于堆栈顶部的两个整数值的按位求补并将结果推送到计算堆栈上。</td>


</tr>
<tr>
<td>Pop</td>
<td width="913">移除当前位于计算堆栈顶部的值。</td>


</tr>
<tr>
<td>Prefix1</td>
<td width="913">基础结构。此指令为保留指令。</td>


</tr>
<tr>
<td>Prefix2</td>
<td width="913">基础结构。此指令为保留指令。</td>


</tr>
<tr>
<td>Prefix3</td>
<td width="913">基础结构。此指令为保留指令。</td>


</tr>
<tr>
<td>Prefix4</td>
<td width="913">基础结构。此指令为保留指令。</td>


</tr>
<tr>
<td>Prefix5</td>
<td width="913">基础结构。此指令为保留指令。</td>


</tr>
<tr>
<td>Prefix6</td>
<td width="913">基础结构。此指令为保留指令。</td>


</tr>
<tr>
<td>Prefix7</td>
<td width="913">基础结构。此指令为保留指令。</td>


</tr>
<tr>
<td>Prefixref</td>
<td width="913">基础结构。此指令为保留指令。</td>


</tr>
<tr>
<td>Readonly</td>
<td width="913">指定后面的数组地址操作在运行时不执行类型检查，并且返回可变性受限的托管指针。</td>


</tr>
<tr>
<td>Refanytype</td>
<td width="913">检索嵌入在类型化引用内的类型标记。</td>


</tr>
<tr>
<td>Refanyval</td>
<td width="913">检索嵌入在类型化引用内的地址（&amp; 类型）。</td>


</tr>
<tr>
<td>Rem</td>
<td width="913">将两个值相除并将余数推送到计算堆栈上。</td>


</tr>
<tr>
<td>Rem.Un</td>
<td width="913">将两个无符号值相除并将余数推送到计算堆栈上。</td>


</tr>
<tr>
<td>Ret</td>
<td width="913">从当前方法返回，并将返回值（如果存在）从调用方的计算堆栈推送到被调用方的计算堆栈上。</td>


</tr>
<tr>
<td>Rethrow</td>
<td width="913">再次引发当前异常。</td>


</tr>
<tr>
<td>Shl</td>
<td width="913">将整数值左移（用零填充）指定的位数，并将结果推送到计算堆栈上。</td>


</tr>
<tr>
<td>Shr</td>
<td width="913">将整数值右移（保留符号）指定的位数，并将结果推送到计算堆栈上。</td>


</tr>
<tr>
<td>Shr.Un</td>
<td width="913">将无符号整数值右移（用零填充）指定的位数，并将结果推送到计算堆栈上。</td>


</tr>
<tr>
<td>Sizeof</td>
<td width="913">将提供的值类型的大小（以字节为单位）推送到计算堆栈上。</td>


</tr>
<tr>
<td>Starg</td>
<td width="913">将位于计算堆栈顶部的值存储到位于指定索引的参数槽中。</td>


</tr>
<tr>
<td>Starg.S</td>
<td width="913">将位于计算堆栈顶部的值存储在参数槽中的指定索引处（短格式）。</td>


</tr>
<tr>
<td>Stelem</td>
<td width="913">用计算堆栈中的值替换给定索引处的数组元素，其类型在指令中指定。</td>


</tr>
<tr>
<td>Stelem.I</td>
<td width="913">用计算堆栈上的 native int 值替换给定索引处的数组元素。</td>


</tr>
<tr>
<td>Stelem.I1</td>
<td width="913">用计算堆栈上的 int8 值替换给定索引处的数组元素。</td>


</tr>
<tr>
<td>Stelem.I2</td>
<td width="913">用计算堆栈上的 int16 值替换给定索引处的数组元素。</td>


</tr>
<tr>
<td>Stelem.I4</td>
<td width="913">用计算堆栈上的 int32 值替换给定索引处的数组元素。</td>


</tr>
<tr>
<td>Stelem.I8</td>
<td width="913">用计算堆栈上的 int64 值替换给定索引处的数组元素。</td>


</tr>
<tr>
<td>Stelem.R4</td>
<td width="913">用计算堆栈上的 float32 值替换给定索引处的数组元素。</td>


</tr>
<tr>
<td>Stelem.R8</td>
<td width="913">用计算堆栈上的 float64 值替换给定索引处的数组元素。</td>


</tr>
<tr>
<td>Stelem.Ref</td>
<td width="913">用计算堆栈上的对象 ref 值（O 类型）替换给定索引处的数组元素。</td>


</tr>
<tr>
<td>Stfld</td>
<td width="913">用新值替换在对象引用或指针的字段中存储的值。</td>


</tr>
<tr>
<td>Stind.I</td>
<td width="913">在所提供的地址存储 native int 类型的值。</td>


</tr>
<tr>
<td>Stind.I1</td>
<td width="913">在所提供的地址存储 int8 类型的值。</td>


</tr>
<tr>
<td>Stind.I2</td>
<td width="913">在所提供的地址存储 int16 类型的值。</td>


</tr>
<tr>
<td>Stind.I4</td>
<td width="913">在所提供的地址存储 int32 类型的值。</td>


</tr>
<tr>
<td>Stind.I8</td>
<td width="913">在所提供的地址存储 int64 类型的值。</td>


</tr>
<tr>
<td>Stind.R4</td>
<td width="913">在所提供的地址存储 float32 类型的值。</td>


</tr>
<tr>
<td>Stind.R8</td>
<td width="913">在所提供的地址存储 float64 类型的值。</td>


</tr>
<tr>
<td>Stind.Ref</td>
<td width="913">存储所提供地址处的对象引用值。</td>


</tr>
<tr>
<td>Stloc</td>
<td width="913">从计算堆栈的顶部弹出当前值并将其存储到指定索引处的局部变量列表中。</td>


</tr>
<tr>
<td>Stloc.0</td>
<td width="913">从计算堆栈的顶部弹出当前值并将其存储到索引 0 处的局部变量列表中。</td>


</tr>
<tr>
<td>Stloc.1</td>
<td width="913">从计算堆栈的顶部弹出当前值并将其存储到索引 1 处的局部变量列表中。</td>


</tr>
<tr>
<td>Stloc.2</td>
<td width="913">从计算堆栈的顶部弹出当前值并将其存储到索引 2 处的局部变量列表中。</td>


</tr>
<tr>
<td>Stloc.3</td>
<td width="913">从计算堆栈的顶部弹出当前值并将其存储到索引 3 处的局部变量列表中。</td>


</tr>
<tr>
<td>Stloc.S</td>
<td width="913">从计算堆栈的顶部弹出当前值并将其存储在局部变量列表中的 index 处（短格式）。</td>


</tr>
<tr>
<td>Stobj</td>
<td width="913">将指定类型的值从计算堆栈复制到所提供的内存地址中。</td>


</tr>
<tr>
<td>Stsfld</td>
<td width="913">用来自计算堆栈的值替换静态字段的值。</td>


</tr>
<tr>
<td>Sub</td>
<td width="913">从其他值中减去一个值并将结果推送到计算堆栈上。</td>


</tr>
<tr>
<td>Sub.Ovf</td>
<td width="913">从另一值中减去一个整数值，执行溢出检查，并且将结果推送到计算堆栈上。</td>


</tr>
<tr>
<td>Sub.Ovf.Un</td>
<td width="913">从另一值中减去一个无符号整数值，执行溢出检查，并且将结果推送到计算堆栈上。</td>


</tr>
<tr>
<td>Switch</td>
<td width="913">实现跳转表。</td>


</tr>
<tr>
<td>Tailcall</td>
<td width="913">执行后缀的方法调用指令，以便在执行实际调用指令前移除当前方法的堆栈帧。</td>


</tr>
<tr>
<td>Throw</td>
<td width="913">引发当前位于计算堆栈上的异常对象。</td>


</tr>
<tr>
<td>Unaligned</td>
<td width="913">指示当前位于计算堆栈上的地址可能没有与紧接的 ldind、stind、ldfld、stfld、ldobj、stobj、initblk 或 cpblk 指令的自然大小对齐。</td>


</tr>
<tr>
<td>Unbox</td>
<td width="913">将值类型的已装箱的表示形式转换为其未装箱的形式。</td>


</tr>
<tr>
<td>Unbox.Any</td>
<td width="913">将指令中指定类型的已装箱的表示形式转换成未装箱形式。</td>


</tr>
<tr>
<td>Volatile</td>
<td width="913">指定当前位于计算堆栈顶部的地址可以是易失的，并且读取该位置的结果不能被缓存，或者对该地址的多个存储区不能被取消。</td>


</tr>
<tr>
<td>Xor</td>
<td width="913">计算位于计算堆栈顶部的两个值的按位异或，并且将结果推送到计算堆栈上。</td>


</tr>


</tbody>


</table>


</div>