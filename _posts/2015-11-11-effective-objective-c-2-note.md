---
layout: post
title:  "《Effective Objective-C 2.0》读书笔记"
date:  2015-11-11
categories: reading-notes
featured_image: /images/notes_cover.jpg
---

###《Effective Objective-C 2.0》读书笔记


####笔记

>1.了解Objective-C语言的起源

Objective-C由`Smalltalk`演化而来。Objectiv-C运行时所应执行的代码由`运行环境`来决定，而非编译器。

>2.在类的头文件中尽量少引入其他头文件

将引入头文件的时机尽量延后，只在确有需要时才引入。`避免循环引用`。

>3.多用字面量语法，少用与之等价的方法

	NSNumber *someNumber = @1;
	NSArray *someArray = @[@"12", @"23", @"34"];
	NSDictionary *someDic = @{@"key" : @"value"};

需要注意的是字面量语法有个小小的限制，除了字符串以外，所创建出来的对象必须属于Foundation框架。创建出来的字符串、数组、字典都是`不可变`的。
	
	NSMutableArray *mutable = [@[@"123", @"234"] mutableCopy];
	
用字面量语法创建数组或字典时，若值中有nil，则会抛出异常。

>4.多用类型常量，少用#define预处理指令

	static const NSTimeInterval kAnimationDuration = 0.3f;
	static NSString *const kTestString = @"test";
	
static修饰符意味着该变量仅在定义此变量的编译单元中可见。

	// In the header file
	extern NSString *const LStringConstant;
	
	// In the implementation file
	NSString *const LStringConstant = @"test";
	
>5.用枚举表示状态、选项、状态码

凡是需要以`按位或`操作来组合的枚举都应该使用NS_OPTIONS定义。若是枚举不需要互相组合，则应使用NS_ENUM来定义。

若是使用枚举来定义状态机，则最好不要有default分支，这样做的好处是，如果稍后添加一种状态，那么编译器就会发出警告信息，提示新加入的状态并未在switch分支中处理。

>6.理解“属性”这一概念

使用`@dynamic`关键字来阻止编译器自动合成存取方法。

assign/strong/weak/unsafe_unretained/copy

`unsafe_unretained`类似于assign，但是它适用于`object type`，而assign只针对scalar type。

如果属性类型为NSString时，经常使用copy来保护其`封装性`。

>7.在对象内部尽量直接访问实例变量

直接访问实例变量的速度比通过属性访问快；不会调用其“设置方法”，绕过了为相关属性所定义的“内存管理语意”；不会触发KVO通知。

合理的折中方法是，在写入实例变量时，通过其“设置方法”来做，读取实例变量时，则直接访问。

在lazy initialization这种情况下，必须通过“获取方法”来访问属性。

	- (SomeClass*)someClass {
		if (!_someClass) {
			_someClass = [[SomeClass alloc] init];
		}
		return _someClass;
	}

>8.理解“对象等同性”这一概念

NSObject协议中有两个用于判断等同性的关键方法：

	- (BOOL)isEqual:(id)object;
	- (NSUInteger)hash;
	
相同的对象必须具有相同的哈希码，但是两个哈希码相同的对象未必相同。

编写hash方法时，应该使用计算速度快而且哈希码碰撞几率低的算法。

	- (NSUInteger) hash {
		NSUInteger firstNameHash = [_firstName hash];
		NSUInteger lastNameHash = [_lastName hash];
		NSUInteger ageHash = _age;
		return firstNameHash ^ lastNameHash ^ ageHash;
	}
	
 如果把某个对象放入set之后又修改其内容，那么后面的行为将很难预料。

>9.以“类族模式”隐藏实现细节

class cluster就是一种`抽象工厂模式`，例如

	+ (UIButton*)buttonWithType:(UIButtonType)type


>10.在既有类中使用关联对象存放自定义数据

可以通过“关联对象”机制来把两个对象连起来。

定义关联对象时可以指定内存管理语意，用以模仿定义属性时所采用的“拥有关系”与“非拥有关系”。

>11.理解objc_msgSend的作用

objc_msgSend会将匹配结果缓存在fast map里面，每个类中都有一块cache。

	objc_msgSend_stret返回结构体
	objc_msgSend_fpret返回浮点数
	objc_msgSendSuper给超类发消息

>12.理解消息转发机制

消息转发分为两个大阶段：第一阶段为征询接收者（dynamic method resolution），第二阶段为完整的消息转发机制（full forwarding mechanism）

	+ (BOOL)resolveInstanceMethod:(SEL)selector
	+ (BOOL)resolveClassMethod:(SEL)selector
	- (id)forwardingTargetForSelector:(SEL)selector
	
	- (void)forwardInvocation:(NSInvocation*)invocation

一个失败的消息转发流程如下所示：
	
	resolveInstanceMethod->forwardingTargetForSelector->forwardInvocation->doesNotRecognizeSelector

若对象无法响应某个SEL，则进入消息转发流程。对象可以把其无法解读的某些SEL转交给其他对象来处理。经过上述两步之后，还是没有办法处理SEL，那就启动完整的消息转发机制。

>13.用“方法调配技术”调试“黑盒方法”

method swizzling技术，AOP

	Method class_getInstanceMethod(Class aClass, SEL aSelector)
	void method_exchangeImplementation(Method m1, Method m2)

可以为那些完全不知道其具体实现的黑盒方法增加`日志记录功能`，若是滥用会令代码变得不易读懂而且难于维护。

>14.理解“类对象”的用意

	typedef struct objec_object {
		Class isa;
	} *id;

	typedef struct objc_class *Class 
	struct objc_class {
		Class isa;
		Class super_class;
		const char *name;
		long version;
		long info;
		long instance_size;
		struct objc_ivar_list *ivars;
		struct objc_method_list **methodLists;
		struct objc_cache *cache;
		struct objc_protocol_list *protocols;
	}
	
更详细的信息可以查看`objc/runtime.h`中的相关定义。

`元类(metaclass)`用来描述类对象本身所具备的元数据。
	
>15.用前缀避免命名空间冲突

Apple宣城其保留使用所有`“两个字母前缀（two-letter prefix）”`的权利，我们应该使用`至少三个字母`的前缀。

>16.提供“指定初始化器”

如果子类的`designated initializer`与超类方法的名称不同，那么总应覆写超类的对应方法。

>17.实现description方法

在自定义类的description方法中，可以将`待打印的信息放到dictionary中`，简化实现。

	- (NSString*)description {
		return [NSString stringWithFormat:@"<%@: %p, %@>",
				[self class], self,
				@{@"title":_title,
				  @"latitude":@(_latitude),
				  @"longitude":@(_longitude),}
			    ];
	}
	
还有一个debugDescription，此方法是在开发者在调试器中以控制台命令打印对象时才调用。

>18.尽量使用不可变对象

尽量把对外公布的属性设为`只读`，而且`只有在确有必要时`才将属性对外公布。

>19.实用清晰而协调的命名方式

清晰的方法名从左到右读起来想一段文章。

>20.为私有方法名加前缀

单用一个下划线作为私有方法的前缀是预留给苹果公司的，因此开发者不应该单用一个下划线作为私有方法的前缀。

可以使用`p_method`

>21.理解Objective-C错误模型

只在极其罕见的情况下抛出异常，`异常抛出后，无需考虑恢复问题，应用程序此时也应该退出。`

使用`NSError`

>22.理解NSCopying协议

	- (id)copyWithZone:(NSZone*)zone
	- (id)mutableCopyWithZone:(NSZone*)zone;

>23.通过委托与数据源协议进行对象间通信

delegate & data source

>24.将类的实现代码分散到便于管理的数个分类之中

可以使用`category`将类的实现代码分成易于管理的小块，将视为私有的方法归入名为Private的分类中，以隐藏实现细节。

>25.总是为第三方类的分类名称加前缀

加上前缀`避免命名冲突`。

>26.勿在分类中声明属性

属性只是定义实例变量及相关存取方法所用的语法糖，所以必须把属性都定义在主接口中，但是`只读属性（readonly）`还是可以在分类中使用的。

属性索要表达的意思是：类中有数据在支持着它，属性是用来`封装数据`的。因此不要在分类中声明属性。

>27.使用“class-continuation分类”隐藏实现细节

	@interface-extension - Objctice Class Extension

>28.通过协议提供匿名对象

	@property (nonatomic, weak) id<SomeDelegate> delegate;

>29.理解引用计数

retainCount

>30.以ARC简化引用计数

ARC通过命名约定将内存管理规则标准化。

ARC只负责管理Objective-C对象的内存，CoreFoundation对象不归ARC管理，开发者必须适时调用CFRetain/CFRelease。

>31.在dealloc方法中只释放引用并解除监听

	- (void)dealloc {
		[[NSNotificationCenter defaultCenter] removeObserver:self];
		[super dealloc];
	}

>32.编写“异常安全代码”时留意内存管理问题

捕获异常时，一定要注意将try中创建的对象清理干净。在默认情况下，ARC不声称安全处理异常所需要的清理代码，开启编译器标志（-fobjc-arc-exceptions）后，可生成这种代码，不过会导致应用程序变大，而且会降低运行效率。

>33.以弱引用避免保留环

避免retain cycle最佳方式是用`弱引用`。

>34.以“自动释放池块”降低内存峰值

将自动释放池嵌套，可以借此控制应用程序的内存峰值，使其不致过高。

>35.用“僵尸对象”调试内存管理问题

僵尸对象（Zombie Object）是调试内存管理问题的最佳方式。通过环境变量`NSZombieEnabled`开启此功能。

僵尸类（zombie class）是从名为`_NSZombie_的模板类`里复制出来的，这些僵尸类没有多少事可做，只是`充当一个标记`。

_NSZombie_类（以及所有从该类拷贝出来的类）并未实现任何方法，此类没有超类，和NSObject一样，也是一个根类，只有一个实例变量，isa。

核心是___forwarding___，调用僵尸类指向的方法，只是log输出相关信息。

>36.不要使用retainCount

`单例对象（singleton object）`的保留计数器很大，例如2^63 - 1，一次NSString打印出来的retainCount通常是一个很大的值。

苹果在引用ARC后就将retainCount正式废弃了，因此尽量不要使用retainCount。

>37.理解“块”这一概念

	return_type (^block_name)(parameters)
	
C语言层面的特性，函数指针。

block会把它所捕获的所有变量都拷贝一份，但是拷贝的不是对象本身，而是指向这些对象的指针变量。

分为全局块、栈块和堆块。

给block对象发送copy消息后，可以将此块从栈复制到堆上。

>38.为常用的块类型创建typedef

	typedef int(^SomeBlock)(BOOL flag, int value);

>39.用handler块降低代码分散程度

在delegate中实现block，将不同逻辑集中处理。

>40.用块引用其所属对象时不要出现保留环

如果block所捕获的对象间接或直接地保留了块本身，那么就得小心retain cycle，一定要找个适当的时机解除保留环，而不能把责任推给API的调用者。

>41.用派发队列，少用同步锁

	@synchronized(self)

	NSLock、NSRecursiveLock

滥用@synchronized(self)会降低代码效率，共用一个锁的那些同步块，都必须按顺序执行。

serial synchronization queue/concurrent queue

	dispatch_create_queue
	dispatch_get_global_queue
	dispatch_sync
	dispatch_async
	dispatch_barrier_async
	dispatch_barrier_sync
	
>42.多用GCD，少用performSelector系列方法

performSelector系列方法在内存管理方面容易疏失，它无法确定将要执行的SEL具体是什么，因而ARC编译器也就无法插入适当的内存管理方法。

performSelector系列方法所能处理的SEL太过局限了，SEL的返回值类型及发送给方法的参数个数都受到限制。

可以使用dispatch_after、dispatch_async等方法来替代，可以更加灵活操作。

>43.掌握GCD及操作队列的使用时机

GCD是纯C的API，而NSOperationQueue则是Objective-C的对象。

NSOperation好处如下：

	取消某个操作
	指定操作间的依赖关系
	通过键值观测机制监控NSOperation对象的属性
	指定操作的优先级
	重用NSOperation对象
	
应该尽可能选用高层API，只在`确有必要`时才求助于底层。

>44.通过Dispatch Group机制，根据系统资源状况来执行任务

一系列任务可归入一个dispatch_group中，开发者可以在这组任务执行完毕时获得通知。

通过dispatch_group执行多项任务时，GCD会根据系统资源状况来调度这些并罚执行的任务。

>45.使用dispatch_once来执行只需运行一次的线程安全代码

使用dispatch_once可以简化代码并且彻底保证线程安全。

	+ (id)sharedInstance {
		static SomeClass *sharedInstance = nil;
		static dispatch_once_t onceToken;
		dispatch_once(&onceToken, ^{
			sharedInstance = [[self alloc] init];
		});
		return sharedInstance;
	}

>46.不要使用dispatch_get_current_queue

dispatch_get_current_queue函数的行为常常与开发者所预期的不同，由于dispatch queue是按层级来组织的，所以无法单用某个队列对象来描述“当前队列”这一概念。而且此函数已经废弃。

>47.熟悉系统框架

toll-free bridging可以把CoreFoundation中的C语言数据结构平滑转换为Foundation中的Objective-C对象。

CoreAnimation本身不是框架，是QuartzCore框架的一部分。

>48.多用块枚举，少用for循环

Objective-C 2.0引入了快速遍历功能，只需要遵从NSFastEnumeration协议，即可采用此语法来迭代该对象。

	- (NSUInteger)countByEnumeratingWithState:(NSFastEnumerationState*)state
									   objects:(id*)stackbuffer
									     count:(NSUInteger)length


	NSArray *arr = @[@"123", @"234"];
	for (NSString* string in arr) {
		NSLog("%@", string);
	}
	
基于块的遍历方式通过GCD来兵法执行遍历操作

	- (void)enumerateObjectsUsingBlock:
			(void(^)(id object, NSUInteger idx, BOOL *stop))block
	

>49.对自定义其内存管理语意的collection使用无缝桥接

`__bridge`是ARC仍然具备这个Objective-C对象的所有权，`__bridge_retained`则与之相反，意味着ARC将交出对象的所有权。反向转换可以通过`__bridge_transfer`来实现。这三种转换方式成为bridged cast。

	NSArray *arr = @[@1, @2, @3];
	CFArrayRef aCFArray = (__bridge CFArrayRef)arr;

>50.构建缓存时选用NSCache而非NSDictionary

NSCache可以自动删除缓存，采用的策略是`最久未使用`的删减掉。NSCache是`线程安全`的。

>51.精简initialize与load的实现代码

对于加入运行期系统中的每个类及分类，必定会调用load，而且`仅调用一次`。load方法问题在于，执行该方法时，运行期系统处于`脆弱状态（fragile state）`，根据某个给定的程序库，无法判断出其各类的`载入顺序`。因此，在load方法中使用其他类是`不安全`的。由于整个应用程序在执行load方法时都会`阻塞`，因此load方法务必实现的精简，尽量减少其所执行的操作。`不要使用它。`

initialize是`惰性调用`的，只有当程序用到了相关的类时，才会调用。在运行期系统执行该方法时，是处于`正常状态`，因此，从运行期系统完整度上来说，此时可以`安全使用并调用任意类中的任意方法`。而且运行期系统也能确保initialize一定会在`线程安全`的环境中执行。

若某个全局状态无法在编译器初始化，可以放在initialize里来做。

	static NSMutableArray *kSomeObjects;
	
	@implementation SomeClass
	
	+ (void)initialize {
		if (self == [SomeClass class]) {
			kSomeObjects = [NSMutableArray array];
		}
	}

>52.别忘了NSTimer会保留其目标对象

NSTimer对象会保留其目标，直到计时器本身失败为止。

####后感

这本书看起来挺慢的，倒不是读书速度下降了，有两个原因吧，第一个是翻译的有点烂，这个会影响看书速度；第二个是怕遗漏了些啥。这篇读书笔记写了好几天了，写读书笔记的时候，算是把书中标记的重点又温习了一遍，因此我并不觉得是在浪费时间哈。

容我吐槽一会儿吧，好多IT书籍的翻译者们，你们不懂的名词缩写，千万别对着单词就翻译出来了，把GCD翻译成大中枢派发，我勒个去，还有其他的，我就不举例了哈。其实以前读书的时候，看一些教材，通常看的稀里糊涂的，有一次，听一个老教授讲课，他对着书中一段话念了好几遍，然后来了一句，唉，这些译者，翻译的教材自己估计就读不懂，找一帮研究生每人翻译几部分，自己整理下就出书了。

现在对于专业相关的书籍，如果是学校出版社的，我会特别谨慎，因为有相当部分翻译的太烂了。

这本书还是挺推荐iOS开发者阅读的，如果有时间和精力的话，还是推荐直接看原版吧，这本书算是一个桥梁吧，我觉得，如果看的都是XX开发教程的话，很难对oc有所更深入的了解，毕竟太多的iOS开发者仅仅是API调用者哈。哈哈～
