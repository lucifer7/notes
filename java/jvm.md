变量槽
操作数栈


基于栈的指令集
区别于Android，基于寄存器

[Reference url](http://wiki.jikexueyuan.com/project/java-vm/storage.html)



#### 2.2 活跃分析 Liveness analysis 
考虑变量的生存期
- any-path backward data-flow analysis

变量只有在被使用的地方才是“活跃”的
- a variable is alive only in used

内联 Inline in C/ final in JAVA(not for sure) 
函数调用被直接替换为语句  ？

安全点 safepoint
- JIT编译后的代码可以进入GC的特定位置
- 解释执行的可以在任意位置进入   ？

JAVA的new Object()在字节码中是两条指令
- new 分配空间和默认初始化
- invokespecial 会调用构造器

什么栈上创建啊，逃逸分析啊，标量替换啊.....