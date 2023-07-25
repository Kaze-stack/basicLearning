# iOS相关

目录
[Objective-C](#objective-c)

+ [1. 语言特点](#1-语言特点)

+ [2. 面向对象](#2-面向对象)

+ [3. 引用计数](#3-引用计数)

+ [4. Block](#4-block)

+ [5. 多线程](#5-多线程)

[Swift](#swift)
[iOS开发](#ios开发)

## Objective-C

### 1. 语言特点

#### 1.1 简介

Objective-C(简称oc、objc)是基于C语言、高级、面向对象的编程语言，通过引入SmallTalk式消息传递机制，实现了面向对象的机制，因此OC是C的超集。

#### 1.2 特性

##### 1.2.1 弱类型

在编写程序时，若出现静态类型不匹配，编译器会发出警告，但不会阻止编译。

```objective-c
// 将NSString赋给NSUInteger
// warning: Incompatible pointer to integer conversion
// initializing 'NSUInteger' with an expression of type 'NSString *'
NSUInteger number = @"abcd";
```

##### 1.2.2 动态类型

OC中有id类型，类似于C中的 `void *` ，可以指向任意 **对象**，其类型的确定在运行时中完成。

  ```objective-c
  // 指向一个Person对象实例
  id ptr = [[Person alloc] init];
  // 结果为Person
  NSLog(@"%@", [ptr class]);

  // 指向一个Car对象实例
  ptr = [[Car alloc] init];
  // 结果为Car
  NSLog(@"%@", [ptr class]);
  ```

##### 1.2.3 动态绑定

OC调用对象方法的方式是消息传递机制 `objc_msgSend` ，消息的接收者如何响应消息需要在运行时确定，与C++中调用虚函数的过程类似。
objc_msgSend通过receiver自身类型的 `isa指针` 在类中进行方法查找，先在 `方法缓存(cache)` 中查找，若未命中则进入类的 `方法列表(method_list)` 中查找；在本类中未得到结果，再向父类发起查找，直至找到或进入根类型。

  ```objective-c
  // 可能在receiver自身中找到响应方法，有可能在父类中找到，也有可能无法响应
  [receiver send:msg];
  // 转写为objc_msgSend(receiver, @selector(send:), msg);
  ```

##### 1.2.4 动态加载

得益于运行时系统，OC可以在运行时期间创建新的 `类(Class)` ，对 `属性(property)` 和 `方法(method)` 进行查询、添加、修改。
>运行时内容详见[运行时Runtime](#7-运行时runtime)

### 2. 面向对象

#### 2.1 与C++的异同

+ 同

  + OC和C++都是基于C的、面向对象的高级语言
  + OC和C++都允许子类重写父类方法，然后实现动态绑定
  + ... ...

+ 异
  
  + C++允许多重继承

    ```cpp
    class A : public B, public C, public D {
        // 定义
    };
    ```

    OC不允许直接继承多个类，但可以继承多个协议

    ```objective-c
    @interface A : NSObject<B, C, D>
    // 定义
    @end
    ```

  + OC的对象只能动态分配在堆上

    ```objective-c
    // 动态分配在堆上
    Person *p = [[Person alloc] init];
    ```

    C++的对象可以动态分配在堆上，也可以静态分配在栈上，还可以动态分配在栈上。

    ```cpp
    // 动态分配在堆上
    Person *p1 = new Person;
    // 静态分配在栈上
    Person p2;
    // 动态分配在栈上
    char bytes[128];
    Person *p3 = new(bytes) Person;
    ```

  + OC调用方法是消息传递机制，因此向空指针(nil)发送消息不会报错，而在C++中会崩溃
  + OC有一套运行时系统，可以在运行期间获取类的相关信息(类名等)，C++不行
  + C++允许重载方法，OC不行
  + ARC(自动引用计数)是OC内存管理的 **特性** ，但在C++中只是众多 **工具** 中的一套
  + C++传递对象时可以值传递、引用传递、指针传递、所有权传递，OC只能指针传递
  + ... ...

#### 2.2 定义和实现

##### 2.2.1 **@interface**

类的定义，也被称为 **接口** 。所有的类都继承自NSObject。
在 **.h文件** 中编写时，可以定义 **公有** 的属性(property)和方法(method)，也可以使用限定符 `@public` 、 `@private` 、 `@protected` 和 `@package` 手动指定成员变量(ivar)的访问控制。
<br>

##### 2.2.2 **@implementation**

对定义的实现，除了已有定义的方法，还可以实现未在接口中定义的 **“私有”** 方法。
><font color=PaleVioletRed>注意</font>
>OC并没有真正意义上的 **私有** 属性和方法，所谓的 **公有** 属性和方法是通过 **.h文件** 暴露出去的，因此在 **.m文件** 中，不管使用什么限定符来修饰新定义的成员变量，它都是 **私有** 的。

#### 2.3 属性(@property)

##### 2.3.1 属性修饰符

| 属性名 | 属性类型 | 用途 |
| :------: | :------: |:-------|
| atomic    | 线程安全 | 默认，对ivar的访问为原子操作，线程安全但耗费系统资源，不建议使用 |
| nonatomic | 线程安全 | 对ivar的访问为非原子操作，线程不安全但访问效率高 |
| readwrite | 读写 | 默认，生成getter/setter方法 |
| readonly | 读写 | 只读，仅生成getter方法，但可以通过KVC修改 |
| assign | 内存管理 | MRC环境默认 <br> 1. 对于基本数据类型，就是简单赋值 <br> 2. 对于对象类型，约等于unsafe_retain，使用时会给出警告 |
| retain | 内存管理 | 持有对象，计数+1，仅在MRC环境下使用，ARC环境下会报错 |
| copy | 内存管理 | 复制对象，生成一个新的、计数为1的对象，需要遵守NSCopying协议 |
| strong | 内存管理 | ARC环境默认 <br> 强引用，仅在ARC环境下使用，计数+1 |
| weak | 内存管理 | 弱引用，仅在ARC环境下使用，计数不会+1，对象销毁后会置nil |
| unsafe_retain | 内存管理 | 不持有，仅在ARC环境下使用，计数不会+1，对象销毁后不会置nil |

##### 2.3.2 @synthesize

为属性指定一个成员变量名，默认成员变量名为在属性名前加一个下划线(_)

  ```objective-c
  // 默认
  @synthesize pName = _pName;

  // 手动指定 xName 为 pName 的成员变量名
  @synthesize pName = xName;
  ```

##### 2.3.3 @dynamic

告诉编译器，属性的setter与getter方法由用户自己定义并实现，不自动生成。

#### 2.4 类目(Category)

在不修改类原有内容的基础上，为类添加方法，可以为系统类添加方法。
><font color=PaleVioletRed>注意</font>
>在程序运行中，类目所添加的方法将通过运行时系统，加入到原有类的方法列表中。由于对象的内存分配在运行时已经确定，因此添加的属性只有getter/setter方法，没有实际的成员变量。如果需要添加属性，需要通过运行时系统的关联对象实现。

一般将类目文件命名为 **类名+添加方法名.h/m**

```objective-c
// NSString+Reserve.h
@interface NSString (Reserve)

- (void)Reserve;

@end

// NSString+Reserve.m
@implementation NSString (Reserve)

- (void)Reserve {
    // do something
}

@end
```

#### 2.5 扩展(Extension)

可以看作是一种无名的类目，只能存放在 **.m文件** 中，因此，扩展实际上是对类添加了 **私有** 属性和方法。不能对系统类进行扩展。

><font color=PaleVioletRed>注意</font>
>扩展的内容在编译期就已经与原有类合并，因此添加的方法一定要实现，在运行时处理的类目就没有这个限制。

```objective-c
// Person.m
@interface Person ()
// 为Person类型添加address属性 
@property (nonatomic, strong) NSString *address;

@end
```

#### 2.6 协议(@protocol)

协议是方法声明的集合，分为 **必须实现(@required)** 和 **可选实现(@optional)** 。
OC不允许多继承，但可以继承多个协议。
在需要时，可以使用协议对属性和参数进行约束，例如常见的 `id<NSCopying>`。

```objective-c
// 定义协议, 继承自NSObject协议
@protocol proT<NSObject>
// 必须实现
@required
- (void)method1;

// 可选实现
@optional
- (void)method2;

@end

// 继承proT, NSCopying协议
@interface classT : NSObject<proT, NSCopying>

- (void)method1;

@end 
```

#### 2.7 KVC/KVO

##### 2.7.1 KVC

KVC是Key-Value-Coding的缩写，即键值编码，KVC提供了一种间接访问属性和成员变量的机制，可以通过字符串来访存对应的属性和成员变量，不受限定符(例如 `@private`、`readonly` )的影响。

###### KVC写方法流程

![KVC写方法流程](/专业方向知识/img/KVC_write.jpg)

###### KVC读方法流程

![KVC读方法流程](/专业方向知识/img/KVC_read.jpg)

##### 2.7.2 KVO

KVO是Key-Value-Observing的缩写，即键值监听，KVO提供了一套基于观察者模式的事件通知机制。KVO和NSNotificationCenter都是观察者模式的一种实现，区别在于，NSNotificationCenter可以是一对多的关系，而KVO是一对一的。

+ KVO使用
  + 注册监听 `addObserver:forKeyPath:options:context:`
  + 监听回调 `observeValueForKeyPath:ofObject:change:context:`
  + 移除监听 `removeObserver:forKeyPath:`
<br>

+ KVO本质
  + 注册了对象A的观察后，利用运行时系统动态生成一个子类 `NSKVONotifying_A` ，并且让对象A的 `isa` 指向这个全新的子类。
  + 重写子类的 `setter` 方法。
  + 修改对象A的属性时，先调用子类的 `setter` 方法。
    + 调用 `willChangeValueForKey:` 方法
    + 调用 `父类` 即对象A的 `setter` 方法
    + 调用 `didChangeValueForKey` 方法，并触发监听回调
    ![KVO的实现](img/KVO_impelment.webp)

### 3 引用计数

> 可以对比阅读Swift的 引用计数 部分

#### 3.1 概念

引用计数是一种内存管理机制，一般用于管理对象类型的生命周期，在OC中用于管理OC对象。
一个对象的引用计数大于0时，表示这个对象被持有，不能被释放；当引用计数为0时表示这个对象已经不被持有，需要释放并回收内存。

##### 3.1.1 原理

相较于C++中shared_ptr的非侵入式实现，OC中的引用计数是混合式的，存储在对象内的 `isa指针` 和外部的 `SideTable` 中。

```c
union isa_t 
{
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }
 
    Class cls;
    uintptr_t bits;
    # if __arm64__ // arm64架构
#   define ISA_MASK        0x0000000ffffffff8ULL
#   define ISA_MAGIC_MASK  0x000003f000000001ULL
#   define ISA_MAGIC_VALUE 0x000001a000000001ULL
    struct {
        uintptr_t nonpointer        : 1;
        uintptr_t has_assoc         : 1;
        uintptr_t has_cxx_dtor      : 1;
        uintptr_t shiftcls          : 33;
        uintptr_t magic             : 6;
        uintptr_t weakly_referenced : 1;
        uintptr_t deallocating      : 1;
        uintptr_t has_sidetable_rc  : 1;
        uintptr_t extra_rc          : 19;

#       define RC_ONE   (1ULL<<45)
#       define RC_HALF  (1ULL<<18)
    };
}
```

当 `nonpointer` 为 `0`时，表明是未经过优化的，对象的引用计数直接存放在 `SideTable` 中。

当 `nonpointer` 为 `1` 时，表明是经过优化的，对象的引用计数优先存储在 `extra_rc` 中，最多可以存储 $2^{19}=524288$ 次计数，当 `extra_rc` 存放不下时，将 `has_sidetable_rc` 置1，使用 `SideTable` 存放更多的计数。

```c
struct SideTable {
    spinlock_t slock;
    RefcountMap refcnts;
    weak_table_t weak_table;
    void lock() { slock.lock(); }
    void unlock() { slock.unlock(); }
    void forceReset() { slock.forceReset(); }
    // Address-ordered lock discipline for a pair of side tables.
    static void lockTwo(SideTable *lock1, SideTable *lock2);
    static void unlockTwo(SideTable *lock1, SideTable *lock2);
};
```

通过 `Sidetable` 中的 `RefCountMap` ，从hash表来查找某个对象的引用计数，进而对引用计数进行操作。

#### 3.2 手动引用计数(MRC)

##### 3.2.1 对象计数操作

| 对象操作 | OC方法 | 计数变化 |
| :------: | :------- | :------- |
| 生成并持有 | alloc/new/copy | +1 |
| 持有 | retain | +1 |
| 释放 | release | -1 |
| 自动释放 | autorelease | 释放池销毁时，-1 |
| 获取计数 | retainCount | / |

##### 3.2.2 自动释放池

首先， `autorelease` 与 `ARC` 没有任何联系，是两种不同的东西， `autorelease` 并没有影响对象的引用计数，只是提供了延迟释放的机制。

自动释放池 `autoreleasepool` 就相当于C语言中的作用域，当域中的局部变量超出作用域时，这些变量就会被销毁，不过与C语言不同的是，自动释放池的作用域边界需要手动设定。

```objective-c
int main(int argc, const char * argv[]) {
    // 新建自动释放池
    NSAutoreleasePool *pool = [[NSAutoreleasePool alloc]init];
    id obj1 = [[NSObject alloc] init];
    // obj1 加入自动释放池
    [obj1 autorelease];

    id obj2 = [[NSObject alloc] init];
    // obj2 加入自动释放池
    [obj2 autorelease];

    // 销毁自动释放池
    // obj2 和 obj1 被 release
    [pool drain];
    return 0;
}
```

等价于

```objective-c
int main(int argc, const char * argv[]) {
    // 新建自动释放池
    @autoreleasepool {
        id obj1 = [[NSObject alloc] init];
        // obj1 加入自动释放池
        [obj1 autorelease];

        id obj2 = [[NSObject alloc] init];
        // obj2 加入自动释放池
        [obj2 autorelease];
    }
    // 销毁自动释放池
    // obj2 和 obj1 被 release
    return 0;
}
```

#### 3.3 自动引用计数(ARC)

`ARC` 只是比 `MRC` 多了一步，在编译期编译器自动添加 `retain` 、 `release` 和 `autorelease` 的调用，底层的内存管理机制还是和 `MRC` 一样。
因此，在ARC环境下进行编写时，以上三种方法是不能调用的。

#### 3.4 循环引用

### 4 Block

### 5 多线程

### 6 Runloop

### 7 运行时Runtime

---

## Swift

---

## iOS开发
