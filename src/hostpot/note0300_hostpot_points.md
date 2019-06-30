


链接：https://www.zhihu.com/question/55141871/answer/143053269


HotSpot VM的解释器是在VM启动的时候生成出来的。它的实现方式是，先用C++为每个要支持的平台写一个Assembler类，于是就可以通过调用这个类上的函数来在内存里生成机器码；然后使用这个类（以及它的包装类MacroAssembler、进一步包装类InterpreterMacroAssembler等）来“手写汇编”实现一些底层功能。具体代码会长这个样子：http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/567e410935e5/src/cpu/x86/vm/templateTable_x86_64.cpp#l360void TemplateTable::sipush() {
  transition(vtos, itos);
  __ load_unsigned_short(rax, at_bcp(1));
  __ bswapl(rax);
  __ sarl(rax, 16);
}
这是纯C++代码。是不是看起来就像“x86-64汇编”一样？另外一个精巧的点是，HotSpot VM使用OS线程来实现Java线程，并且一个Java线程上运行的所有native函数和Java方法都共用一个调用栈。所以HotSpot VM也把这种做法叫做“混合模式栈”（mixed-mode stack或者简称mixed stack）。解释器可以直接使用CPU的栈指针寄存器来表示自己的栈顶指针。然后还有很多…很多有趣的点都发过论文，找来读读比一开始就读源码要有用得多嗯。


# 说起JVM它可以是以下三种：
一个正在运行的Java实例
Java虚拟机规范
一种JVM虚拟机实现