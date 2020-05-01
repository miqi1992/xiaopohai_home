## Java内存区域与内存溢出异常

### 2.1 概述

对于从事C、C++程序开发的开发人员来说，在内存管理领域，他们即是拥有最高权力的皇帝又是执行最基础工作的劳动人民--拥有每一个对象的"所有权"，有担负着每一个对象生命开始到终结的维护责任。  

对于Java程序员来说，不需要在为每一个new操作去写配对的delete/free，不容易出现内存卸扣和内存溢出问题，看起来JVM管理内存一切很美好。不过，也正是因为Java程序员把内存控制的权利交给了JVM，一旦出现内存泄露和溢出，如果不了解JVM是如何使用内存的，那排查错误将会是一件非常困难的事。  

### 2.2 运行时数据区域

Java虚拟机在执行Java程序的过程中会把它所管理的内存划分为若干这不同的数据区域。这些区域都有各自的用途，以及创建和销毁的时间，有的区域随着虚拟机进程的启动而存在，有些区域则依赖用户线程的启动和结束而建立和销毁。根据《Java虚拟机规范(Java SE7版)》的规定，Java虚拟机所管理的内存将会是以下几个运行时数据区域。  

![img](https://xiaopohai-1254153894.cos.ap-chengdu.myqcloud.com/xiaopohai-blog/1168971-20190307110035509-308623185.png)

#### 2.2.1 程序计数器

程序计数器(Program Counter Register)是一块较小的内存空间，它可以看做是当前线程所执行的字节码的行号显示器。在虚拟机的概念模型里(仅是概念模型，各种虚拟机可能会通过一些更加高效的方式去实现)，字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

由于Java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器(对于多个处理器来说是一个内核)都只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各条线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。  

如果线程正在执行的是一个Java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是Native方法，这个计数器则为空(Undefined)。此内存区域是唯一一个在Java虚拟机规范中没有规定任何OutOfMemberError情况的区域。  

[Java程序计数器刨根问底](https://developer.51cto.com/art/201908/601044.htm)

2.2.2 Java虚拟机栈

与程序计数器一样，Java虚拟机栈(Java Virtual Machine Stacks)也是线程私有的，它的生命周期与线程相同。虚拟机描述的是Java方法执行的内存模型：每个方法在执行的同时都会创建一个栈帧(Stach France)用于**存储局部变量表、操作数栈、动态链接、方法出口**等信息。每一个方法都从调用直至执行完成的过程，就对应一个栈帧在虚拟机中入栈到出栈的过程。  

经常有人把Java虚拟机分为堆内存(Heap)和栈内存(Stack)，这种方法比较粗糙，Java内存区的划分是上远比这复杂。这种划分方式的流行只能说明大多数程序员最关注的、与对象内存分配关系最密切的内存区域是这两块。其中所指的“堆”笔者在后面会专门讲述，而所指的“栈”就是现在讲的虚拟机栈，或者说是虚拟机栈中局部变量表部分。  

**局部变量表**存放了编译期可知的各种**基本数据类型**(boolean、byte、char、short、int、float、long、double)、**引用对象**(reference类型，它不等同于对象本身，可能是一个指向对象起始地址的引用指正，也可能是指向一个代表对象的句柄或其他对象相关的位置)和returnAddress类型(指向了一条字节码指令的地址)。  

其中64位长度的long和double类型的数据会占用2个局部变量空间(Slot)，其余的数据类型只占用一个。其余的数据类型只占用1个。局部变量表所需的内存空间在编译器间完成分配，当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。  

在Java虚拟机规范中，对这个区域规定了两种异常状况： 如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError异常，如果虚拟机栈可以动态扩展(当前大部分的Java虚拟机都可动态扩展，只不过Java虚拟机规范中也允许固定长度的虚拟机栈)，如果扩展时无法申请到足够的内存，就会抛出OutOfMemberError异常。  

#### 2.2.3 本地方法栈

本地方法栈(Native Method Stachk)与虚拟机所发挥的作用是非常相似的，他们之间的区别不过是虚拟机栈为虚拟机执行Java方法(也就是字节码)服务，而本地方法栈则为虚拟机使用到的Netive方法服务。在虚拟机规范中对本地方法中方法使用的语言、使用方式与数据结构没有强制规定，因此具体的虚拟机可以自由实现它。甚至有的虚拟机(譬如Sun HotSpot虚拟机)直接把本地方法栈和虚拟机栈合二为一。与虚拟机栈一样，本地方法栈区也会抛出StackOverFlowError和OutOfMemberError异常。  

#### 2.2.4 Java堆

对于大多数应用来说，Java堆(Java Heap)是Java虚拟机所管理的内存中最大的一块。Java堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。这一点在Java虚拟机规范中的描述是：所有的对象实例以及数组都要在堆上分配，但是随着JIT编译器的发展与逃逸分析技术逐渐成熟，栈上分配、标量替换技术将会导致一些微妙的变化发生，所有的对象都分配在堆上也渐渐变的不是那么“绝对”了。 

Java堆是垃圾收集器管理的主要区域，因此很多时候也被称作“GC堆”(Garbage Collected Heap，幸好国内没翻译成“垃圾堆”)。从内存回收的角度来看，由于现在收集器基本都采用分代收集算法，所以Java堆中还可以细分为：新生代和老年代；在细致一点的有Eden空间、From Survivor空间等。从内存分配的角度来看，线程共享的Java堆中可能划分出多个线程私有的分配缓冲区(Thread Local Allocation Buffer, TLAB)。不过无论如何划分，都有存放内容无关，无论哪个区域，存储的都仍然是对象实例，进一步划分的目的是为了更好的回收内存，或者更快地分配内存。在本章中，我们仅仅对内存区域的作用进行讨论，Java堆中的上述各个区域的分配、回收等细节将是第3章的主题。  

根据Java虚拟机的规范，Java堆上可以处于物理上不连续的内存空间，只要逻辑上是连续的即可，就想我们的磁盘空间一样。在实现时，即可以实现成固定大小的，也可是可扩展的，不过当前主流的虚拟机都是按照可扩展来实现的(通过-Xmx和-Xms控制)。如果堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出OutOfMemberError异常。

#### 2.2.5 方法区

方法区(Method Area)与Java堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的**类信息、常量、静态变量、方法**、即时编译器编译后的代码等数据。虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做Non-Heap(非堆)，目的应该是与Java堆区分开来。  

对于习惯在HotSpot虚拟机上开发、部署程序的开发者来说，很多人愿意把方法区称为"永久代"(Permanent Generation)，本质上两者并不等价，仅仅是因为HotSpot虚拟机的设计团队选择把GC分代收集扩展至方法区，或者说使用永久代来实现方法区而已，这样HotSpot的垃圾收集器可以像管理Java堆一样管理这部分内存，能够省去专门为方法区编写内存管理代码的工作。对于其他虚拟机(如BEA JRockit、IBM J9等)来说是不存在永久代的概念的。原则上，如何实现方法区属于虚拟机实现细节，不受虚拟机规范约束，但使用永久代来实现方法区，现在看来并不是一个好主意，因为这样更容易遇到内存溢出问题(永久代有-XX：MaxPermSize的上限，J9和JRockit只要没有接触到进程可用内存的上限，例如32位系统的4GB，就不会出现问题)，而且有极少数方法(例如String.intern()会因这个原因导致虚拟机下有不同的标下。 因此，对于HotSpot虚拟机，根据官方发布的路线图信息，现在也有放弃永久代并逐步改为采用Native Memory来实现方法区的规划了，在目前已经发布的JDK1.7中的HotSpot中，已经把原本放在永久代的字符串常量池中移出。  

Java虚拟机规范对方法区的限制非常宽松，除了和Java堆一样不需要连续的内存和可以选择固定大小或者可扩展外，还可以选择不实现垃圾收集。相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入了方法区就如永久代的名字一样“永久”存在了。这区域的内存回收目标主要是针对常量池的回收和对类型的卸载，一般来说，这个区域的回收“成绩”比较难以令人满意，尤其是类型的卸载，条件相当苛刻，但是这部分区域的回收确实是必要的。在Sun公司的BUG列表中，曾出现过的若干个严重的BUG就是由于低版本的HotSpot虚拟机对此区域未完全回收而导致内存泄漏。根据Java虚拟机规范的规定，当方法区无法满足内存分配需求时，将抛出OutOfMemoryError异常。

#### 2.2.6 运行时常量池

运行时常量池（Runtime Constant Pool）是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池（Constant Pool Table），用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。

Java虚拟机对Class文件每一部分（自然也包括常量池）的格式都有严格规定，每一个字节用于存储哪种数据都必须符合规范上的要求才会被虚拟机认可、装载和执行，但对于运行时常量池，Java虚拟机规范没有做任何细节的要求，不同的提供商实现的虚拟机可以按照自己的需要来实现这个内存区域。不过，一般来说，除了保存Class文件中描述的符号引用外，还会把翻译出来的直接引用也存储在运行时常量池中。

运行时常量池相对于Class文件常量池的另外一个重要特征是具备动态性，Java语言并不要求常量一定只有编译期才能产生，也就是并非预置入Class文件中常量池的内容才能进入方法区运行时常量池，运行期间也可能将新的常量放入池中，这种特性被开发人员利用得比较多的便是String类的intern()方法。

既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出OutOfMemoryError异常。

2.2.7 直接内存

直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域。但是这部分内存也被频繁地使用，而且也可能导致OutOfMemoryError异常出现，所以我们放到这里一起讲解。

在JDK 1.4中新加入了NIO（New Input/Output）类，引入了一种基于通道（Channel）与缓冲区（Buffer）的I/O方式，它可以使用Native函数库直接分配堆外内存，然后通过一个存储在Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在Java堆和Native堆中来回复制数据。

显然，本机直接内存的分配不会受到Java堆大小的限制，但是，既然是内存，肯定还是会受到本机总内存（包括RAM以及SWAP区或者分页文件）大小以及处理器寻址空间的限制。服务器管理员在配置虚拟机参数时，会根据实际内存设置-Xmx等参数信息，但经常忽略直接内存，使得各个内存区域总和大于物理内存限制（包括物理的和操作系统级的限制），从而导致动态扩展时出现OutOfMemoryError异常。



### 2.3 HotSpot虚拟机对象探秘

介绍完Java虚拟机的运行时数据区之后，我们大致知道了虚拟机内存的概况，读者了解了内存中放了些什么后，也许就会想更进一步了解这些虚拟机内存中的数据的其他细节，譬如它们是如何创建、如何布局以及如何访问的。对于这样涉及细节的问题，必须把讨论范围限定在具体的虚拟机和集中在某一个内存区域上才有意义。基于实用优先的原则，笔者以常用的虚拟机HotSpot和常用的内存区域Java堆为例，深入探讨HotSpot虚拟机在Java堆中对象分配、布局和访问的全过程。

#### 2.3.1 对象的创建

Java是一门面向对象的编程语言，在Java程序运行过程中无时无刻都有对象被创建出来。在语言层面上，创建对象（例如克隆、反序列化）通常仅仅是一个new关键字而已，而在虚拟机中，对象（文中讨论的对象限于普通Java对象，不包括数组和Class对象等）的创建又是怎样一个过程呢？

虚拟机遇到一条new指令时，首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化过。如果没有，那必须先执行相应的类加载过程，本书第7章将探讨这部分内容的细节。

在类加载检查通过后，接下来**虚拟机将为新对象分配内存**。对象所需内存的大小在类加载完成后便可完全确定(如何确定将在2.3.2节中介绍)，为对象分配空间的任务等同于把一块确定大小的内存从Java堆中划分出来。假设Java堆中内存时绝对规整的，所有用过的内存都放到一遍，空闲的内存放到另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那个指针向空闲空间那边挪动一段与对象大小相等的距离，该中分配方式称为"指针碰撞"(Bump the Pointer)。如果Java堆中的内存并不是完整的，已使用的内存可空闲的内存相互交错，那就没有办法简单地进行指针碰撞了，虚拟机就必须维护一个列表，记录是哪个哪些内存块时可用的，在分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录，这种分配方式称为"空闲列表"(Free List)。选择哪种分配方式由Java堆是否规整决定，而Java堆是否规整又由所采用的垃圾收集器师傅偶带有压缩整理功能决定。因此，在使用Serial、ParNew等带Compact过程的收集器时，系统采用的分配算法时指针碰撞，而使用CMS这种基于Mark-Sweep算法的收集器时，通常采用空闲列表。  

除如何划分可用空间之外，还有另外一个需要考虑的问题是对象创建在虚拟机中是非常频繁的行为，即使是仅仅修改一个指针所指向的位置，在并发情况下也并不是线程安全的，可能出现正在给对象A分配内存，指针还没来得及修改，对象B又同时使用了原来的指针来分配内存的情况。解决这个问题有两种方案，一种是对分配内存空间的动作进行同步处理--实际上虚拟机采用CAS配上失败重试的方式保证更新操作的原子性；另一种是把内存分配的动作按照线程划分在不同的空间之中进行，即每个线程在Java堆中预先分配一小块内存，成为本地线程分配缓冲(Thread Local Allocation Buffer, TLAB)。哪个线程要分配内存，就在哪个线程的TLAB上分配，只有TLAB用完并分给新的TLAB时，才需要同步锁定。虚拟机是否采用TLAB，可以通过-XX: +/-UseTLAB参数来设定。  

内存分配完成后，虚拟机需要将分配到的**内存空间都初始化为零值**(不包括对象头)，如果使用TLAB，这一工作过程也可以提前至TLAB分配时进行。这一步操作保证了对象的实例字段在Java代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。  

接下来，虚拟机要对对象进行必要的设置，**例如 这个对象时哪个类的实例**、**如何才能找到类的元数据信息**、**对象哈希吗**、**对象的GC分代年龄等信息**。**这些信息存放在对象的对象头**(Object Header)之中。根据虚拟机当前的运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。关于对象头的具体内容，稍后再做详细介绍。  

在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了，但从Java程序的视角来看，对象创建才刚刚开始--<init>方法还没有执行，所有的字段都还为零。所以，一般来说(由字节码中是否跟随invokespecial指令所决定)，执行new指令之后会接着执行<init>方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全生产出来。  

下面的代码清单是HotSpot虚拟机bytecodeInterpreter.cpp中的代码片段(这个解释器实现很少有机会实际使用，因为大部分平台上都使用模板解释器；当代码通过JIT编译器执行时差异就更大了。不过，这段代码用于了解HotSpot的运作过程是没有什么问题的)。  

``` cpp
//确保常量池中存放的是已解释的类
if(!constants->tag_at(index).is_unresolved_klass()){
//断言确保是klassOop和instanceKlassOop（这部分下一节介绍）
oop entry=（klassOop）*constants-＞obj_at_addr（index）;
assert（entry-＞is_klass（），"Should be resolved klass"）;
klassOop k_entry=（klassOop）entry;
assert（k_entry-＞klass_part（）-＞oop_is_instance（），"Should be instanceKlass"）;
instanceKlass * ik=（instanceKlass*）k_entry-＞klass_part（）;
//确保对象所属类型已经经过初始化阶段
if(ik->is_initialized()&&ik->can_be_fastpath_allocated())
{
//取对象长度
size_t obj_size=ik->size_helper();
oop result=NULL；
//记录是否需要将对象所有字段置零值
bool need_zero=!ZeroTLAB；
//是否在TLAB中分配对象
if(UseTLAB){
result=（oop）THREAD->tlab().allocate(obj_size);
}
if（result==NULL）{
need_zero=true；
//直接在eden中分配对象
retry：
HeapWord * compare_to=*Universe：heap（）-＞top_addr（）；
HeapWord * new_top=compare_to+obj_size；
/*cmpxchg是x86中的CAS指令，这里是一个C++方法，通过CAS方式分配空间，如果并发失败，
转到retry中重试，直至成功分配为止*/
if（new_top＜=*Universe：heap（）-＞end_addr（））{
if（Atomic：cmpxchg_ptr（new_top,Universe：heap（）-＞top_addr（），compare_to）！=compare_to）{
goto retry；
}
result=（oop）compare_to；
}
}
if（result！=NULL）{
//如果需要，则为对象初始化零值
if（need_zero）{
HeapWord * to_zero=（HeapWord*）result+sizeof（oopDesc）/oopSize；
obj_size-=sizeof（oopDesc）/oopSize；
if（obj_size＞0）{
memset（to_zero，0，obj_size * HeapWordSize）；
}
}
//根据是否启用偏向锁来设置对象头信息
if（UseBiasedLocking）{
result-＞set_mark（ik-＞prototype_header（））；
}else{
result-＞set_mark（markOopDesc：prototype（））；
}r
esult-＞set_klass_gap（0）；
result-＞set_klass（k_entry）；
//将对象引用入栈，继续执行下一条指令
SET_STACK_OBJECT（result，0）；
UPDATE_PC_AND_TOS_AND_CONTINUE（3，1）；
}
}
}
```

#### 2.3.2 对象的内存布局

在HotSpot虚拟机中，对象在内存中存储的布局可以分为3块区域：对象头(Header)、实例数据(Instance Data)和对齐补充(Padding)。

HotSpot虚拟机的对象头包括两部分信息：第一部分用于存储对象自身的运行数据，如哈希码(HashCode)、GC分代年龄、锁状态标示、线程持有的锁、偏向锁ID、偏向时间戳等，这部分数据的长度在32位和64位的虚拟机(未开启压缩指针)中分别为32bit和64it，官方称它为"Mark Word"。对象需要存储的运行时数据很多，其实已经超出了32位、64位Bitmap结构所能记录的限度，但是对象头信息是与对象自身定义的数据无关的额外存储成本，考虑到虚拟机的空间效率，Mark Word被设计成一个非固定的数据结构以便在极小的空间内存储尽量多的信息，它会根据对象的状态复用自己的存储空间。例如，在32位的HotSpot虚拟机中，如果对象处于未被锁定的状态下，那么Mark Word的32bit空间中的25bit用于存储对象哈希码，4bit用于存储对象分代年龄，2bit用于存储锁标志位，1bit固定为0，而在其他状态(轻量级锁定、重量级锁定、GC标记、可偏向)下对象的存储内容见表

![这里写图片描述](https://xiaopohai-1254153894.cos.ap-chengdu.myqcloud.com/xiaopohai-blog/20161209164720457)

对象头的另外一部分是类型指针，即**对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例**。并不是所有的虚拟机实现都必须在对象数据上保留类型指针，换句话说，查找对象的元数据信息并不一定要经过对象本身，这点将在2.3.3节讨论。

另外，如果对象是一个Java数组，那在对象头中还必须有一块用于记录数组长度的数据，因为虚拟机可以通过普通Java对象的元数据信息确定Java对象的大小，但是从数组的元数据中却无法确定数组的大小。 

代码清单为HotSpot虚拟机markOop.cpp中的代码（注释）片段，它描述了32bit下MarkWord的存储状态。

``` xml
//Bit-format of an object header(most significant first,big endian layout below):
//32 bits:
//--------
//hash：25------------>|age：4 biased_lock：1 lock：2（normal object）
//JavaThread*：23 epoch：2 age：4 biased_lock：1 lock：2（biased object）
//size：32------------------------------------------>|（CMS free block）
//PromotedObject*：29---------->|promo_bits：3----->|（CMS promoted object）
```

接下来的实例数据部分是对象真正存储的有效信息，也是在程序代码中所定义的各种类型的字段内容。无论是从父类继承下来的，还是在子类中定义的，都需要记录起来。这部分的存储顺序会受到虚拟机分配策略参数（FieldsAllocationStyle）和字段在Java源码中定义顺序的影响。HotSpot虚拟机默认的分配策略为longs/doubles、ints、shorts/chars、bytes/booleans、oops（Ordinary Object Pointers），从分配策略中可以看出，相同宽度的字段总是被分配到一起。在满足这个前提条件的情况下，在父类中定义的变量会出现在子类之前。如果CompactFields参数值为true（默认为true），那么子类之中较窄的变量也可能会插入到父类变量的空隙之中。

第三部分对齐填充并不是必然存在的，也没有特别的含义，它仅仅起着占位符的作用。由于HotSpot VM的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说，就是对象的大小必须是8字节的整数倍。而对象头部分正好是8字节的倍数（1倍或者2倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。



#### 2.3.3 对象的访问定位

建立对象时为了使用对象，我们的Java程序需要通过栈上的reference数据类操作堆上的具体对象。由于reference类型在Java虚拟机规范中只规定了一个指向对象的引用，并没有定义这个引用应该通很尴尬何种方式去定位、访问堆中的对象的具体位置，所以对象访问方式也是取决于虚拟机实现而定位的。目前主流的访问方式由**使用句柄**和**直接指针**两种。  

如果使用句柄访问的话，那么Java堆中将会划分出一块内存俩作为句柄池，reference中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址信息，如图：  

![这里写图片描述](https://xiaopohai-1254153894.cos.ap-chengdu.myqcloud.com/xiaopohai-blog/20161209170908358)

如果直接使用指针访问，那么Java堆对象的布局中就必须考虑如何放置访问类型数据的相关信息，而erference中存储的直接就是对象地址，如图所示。  

![这里写图片描述](https://xiaopohai-1254153894.cos.ap-chengdu.myqcloud.com/xiaopohai-blog/20161209170949905)

这两种对象访问方式各有优势，使用句柄来访问的最大好处就是reference中存储的是稳定的句柄地址，在对象被移动(垃圾收集时移动对象是非常普遍的行为)时只会改变句柄中的实例数据类型，而reference本身不需要修改。  

使用直接指针访问方式的最大好处就是速度更快，它节省了一次指针定位的时间开销，由于对象的访问在Java中是非常频繁的，因此这类开销积少成多也是一项非常可观的执行成本。就本书讨论的主要虚拟机Sun HotSpot而言，它是使用第二种方式进行对象访问的，但从整个软件开发的范围来看，各种语言和框架使用句柄来访问的情况也十分常见。  

### 2.4  实战： OutOfMemoryError异常

在Java虚拟机规范的描述中，除了程序计数器之外，虚拟机内存的其它几个运行时区域都有发生OutOfMemoryError(下文称OOM)异常的可能。  本节将通过若干实例来验证异常发生的场景（代码清单2-3～代码清单2-9的几段简单代码），并且会初步介绍几个与内存相关的最基本的虚拟机参数。

本节内容的目的有两个：第一，通过代码验证Java虚拟机规范中描述的各个运行时区域存储的内容；第二，希望读者在工作中遇到实际的内存溢出异常时，能根据异常的信息快速判断是哪个区域的内存溢出，知道什么样的代码可能会导致这些区域内存溢出，以及出现这些异常后该如何处理。

下文代码的开头都注释了执行时所需要设置的虚拟机启动参数（注释中“VM Args”后面跟着的参数），这些参数对实验的结果有直接影响，读者调试代码的时候千万不要忽略。如果读者使用控制台命令来执行程序，那直接跟在Java命令之后书写就可以。如果读者使用Eclipse IDE，则可以参考图在Debug/Run页签中的设置。 

![这里写图片描述](https://xiaopohai-1254153894.cos.ap-chengdu.myqcloud.com/xiaopohai-blog/20161213132850439)

下文的代码都是基于Sun公司的HotSpot虚拟机运行的，对于不同公司的不同版本的虚拟机，参数和程序运行的结果可能会有所差别。

#### 2.4.1 Java堆溢出

Java堆用于存储对象实例，只要不断地创建对象，并且保证GC Roots到对象之间有可达路径来避免垃圾回收机制清除这些对象，那么在对象数量到达最大堆的容量限制后就会产生内存溢出异常。

代码清单2-3中代码限制Java堆的大小为20MB，不可扩展（将堆的最小值-Xms参数与最大值-Xmx参数设置为一样即可避免堆自动扩展），通过参数-XX：+HeapDumpOnOutOfMemoryError可以让虚拟机在出现内存溢出异常时Dump出当前的内存堆转储快照以便事后进行分析。

``` java
/**
 * VM Args：-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
 * @author zzm
 */
public class HeapOOM {
    static class OOMObject {
    }
    public static void main(String[] args) {
        List<OOMObject> list = new ArrayList<OOMObject>();
        while (true) {
            list.add(new OOMObject());
        }
    }
}
```

运行结果：  

``` java
java.lang.OutOfMemoryError :Java heap space
Dumping heap to java_pid3404.hprof.
Heap dump file created[22045981 bytes in 0.663 secs]
```

Java堆内存的OOM异常是实际应用中常见的内存溢出异常情况。当出现Java堆内存溢出时，异常堆栈信息“java.lang.OutOfMemoryError”会跟着进一步提示“Java heap space”。

要解决这个区域的异常，一般的手段是先通过内存映像分析工具（如Eclipse Memory Analyzer）对Dump出来的堆转储快照进行分析，重点是确认内存中的对象是否是必要的，也就是要先分清楚到底是出现了内存泄漏（Memory Leak）还是内存溢出（Memory Overflow）。下图显示了使用Eclipse Memory Analyzer打开的堆转储快照文件。 

![这里写图片描述](https://xiaopohai-1254153894.cos.ap-chengdu.myqcloud.com/xiaopohai-blog/20161213133159066)

如果是内存泄露，可进一步通过工具查看泄露对象到GC Roots的引用链。于是就能找到泄露对象是通过怎样的路径与GC Roots相关联并导致垃圾收集器无法自动回收它们的。掌握了泄露对象的类型信息及GC Roots引用链的信息，就可以比较准确地定位出泄露代码的位置。

如果不存在泄露，换句话说，就是内存中的对象确实都还必须存活着，那就应当检查虚拟机的堆参数（-Xmx与-Xms），与机器物理内存对比看是否还可以调大，从代码上检查是否存在某些对象生命周期过长、持有状态时间过长的情况，尝试减少程序运行期的内存消耗。

以上是处理Java堆内存问题的简单思路，处理这些问题所需要的知识、工具与经验是后面3章的主题。



#### 2.4.2 虚拟机栈和本地方法栈溢出

由于在HotSpot虚拟机中并不区分虚拟机栈和本地方法栈，因此，对于HotSpot来说，虽然-Xoss参数(设置本地方法栈大小)存在，但实际上是无效的，栈容量只由-Xss参数设定。  

关于虚拟机栈和本地方法栈，在Java虚拟机规范中描述了两种异常：  

1. 如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出StackOverflowError异常。  
2. 如果虚拟机在扩展栈时无法申请到足够的内存空间，则抛出OutOfMemoryError异常。  

这里把异常分成两种情况，看似更加严谨，但却存在着一些互相重叠的地方：当栈空间无法继续分配时，到底是内存太小，还是已使用的栈空间太大，其本质上只是对同一件事的两种描述而已。  

在笔者的试验中，将试验范围限制于单线程的操作，尝试了下面两种方法均无法让虚拟机产生OutOfMemoryError异常，尝试的结果都是获得StackOverflowError异常，测试代码乳袋清单2-4所示。  

+ 使用-Xss参数减少栈内存空间。结果：将抛出StackOverflowError异常，异常出现时数据的堆栈深度相应缩小。  
+ 定义了大量的本地变量，增大此方法帧中本地变量表的长度。结果：抛出StackOverflowError异常时输出的堆栈深度相应缩小。  

代码清单2-4　虚拟机栈和本地方法栈OOM测试（仅作为第1点测试程序）

``` java
/**
 * VM Args：-Xss128k
 * @author zzm
 */
public class JavaVMStackSOF {
    private int stackLength = 1;
    public void stackLeak() {
        stackLength++;
        stackLeak();
    }
    
    public static void main(String[] args) throws Throwable {
        JavaVMStackSOF oom = new JavaVMStackSOF();
        try {
            oom.stackLeak();
        } catch (Throwable e) {
            System.out.println("stack length:" + oom.stackLength);
            throw e;
        }
    }
}
```

运行结果：  

``` java
stack length :2402
Exception in thread"main"java.lang.StackOverflowError
at org.fenixsoft.oom.VMStackSOF.leak (WIStackSOF.java :20 ) at org.fenixsoft.oom.VMStackSOF.leak (WIStackSOF.java :21 ) at org.fenixsoft.oom.VMStackSOF.leak (WIStackSOF.iava :21 ) 
.....后续异常堆栈信息省略
```

实验结果表明：在单个线程下，无论是由于栈帧太大还是虚拟机栈容量太小，当内存无法分配的时候，虚拟机抛出的都是StackOverflowError异常。

如果测试时不限于单线程，通过不断地建立线程的方式倒是可以产生内存溢出异常，如代码清单2-5所示。但是这样产生的内存溢出异常与栈空间是否足够大并不存在任何联系，或者准确地说，在这种情况下，为每个线程的栈分配的内存越大，反而越容易产生内存溢出异常。

其实原因不难理解，操作系统分配给每个进程的内存是有限制的，譬如32位的Windows限制为2GB。虚拟机提供了参数来控制Java堆和方法区的这两部分内存的最大值。剩余的内存为2GB（操作系统限制）减去Xmx（最大堆容量），再减去MaxPermSize（最大方法区容量），程序计数器消耗内存很小，可以忽略掉。如果虚拟机进程本身耗费的内存不计算在内，剩下的内存就由虚拟机栈和本地方法栈“瓜分”了。每个线程分配到的栈容量越大，可以 建立的线程数量自然就越少，建立线程时就越容易把剩下的内存耗尽。

这一点读者需要在开发多线程的应用时特别注意，出现StackOverflowError异常时有错误堆栈可以阅读，相对来说，比较容易找到问题的所在。而且，如果使用虚拟机默认参数，栈深度在大多数情况下（因为每个方法压入栈的帧大小并不是一样的，所以只能说在大多数情况下）达到1000～2000完全没有问题，对于正常的方法调用（包括递归），这个深度应该完全够用了。但是，如果是建立过多线程导致的内存溢出，在不能减少线程数或者更换64位虚拟机的情况下，就只能通过减少最大堆和减少栈容量来换取更多的线程。如果没有这方面的处理经验，这种通过“减少内存”的手段来解决内存溢出的方式会比较难以想到。 

``` java
/**
 * VM Args：-Xss2M （这时候不妨设大些）
 * @author zzm
 */
public class JavaVMStackOOM {

       private void dontStop() {
              while (true) {
              }
       }

       public void stackLeakByThread() {
              while (true) {
                     Thread thread = new Thread(new Runnable() {
                            @Override
                            public void run() {
                                   dontStop();
                            }
                     });
                     thread.start();
              }
       }

       public static void main(String[] args) throws Throwable {
              JavaVMStackOOM oom = new JavaVMStackOOM();
              oom.stackLeakByThread();
       }
}
```

注意，特别提示一下，如果读者要尝试运行上面这段代码，记得要先保存当前的工作。由于在Windows平台的虚拟机中，Java的线程是映射到操作系统的内核线程上的，因此上述代码执行时有较大的风险，可能会导致操作系统假死。 
运行结果：

``` java
Exception in thread"main"java.lang.OutOfMemoryError :unable to create new native thread
```



#### 2.4.3 方法区和运行时常量池溢出

由于运行时常量池是方法区的一部分，因此这两个区域的溢出测试就放在一起进行。前面提到JDK1.7开始逐步“去永久代”的事情，在此就医测试代码