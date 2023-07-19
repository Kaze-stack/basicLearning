# iOS相关

>(预留列表)

## Objective-C

### 1. 语言特点

#### 1.1 简介

Objective-C(简称oc、objc)是基于C语言、高级、面向对象的编程语言，通过引入SmallTalk式消息传递机制，实现了面向对象的机制，因此OC是C的超集。

#### 1.2 特性

+ 弱类型
在编写程序时，若出现静态类型不匹配，编译器会发出警告，但不会阻止编译。

```objective-c
// 将NSString赋给NSUInteger
// warning: Incompatible pointer to integer conversion
// initializing 'NSUInteger' with an expression of type 'NSString *'
NSUInteger number = @"abcd";
```

+ 动态类型
OC中有id类型，类似于C中的(void *)，可以指向任意 **对象**，其类型的确定在运行时中完成。

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

+ 动态绑定
OC调用对象方法的方式是消息传递机制(objc_msgSend)，消息的接收者如何响应消息需要在运行时确定，与C++中调用虚函数的过程类似。

```objective-c
// 可能在receiver自身中找到响应方法，有可能在父类中找到，也有可能无法响应
[receiver send:msg];
// 转写为objc_msgSend(receiver, @selector(send:), msg);
```

objc_msgSend通过receiver自身类型的 **isa指针** 在类中进行方法查找，先在 **方法缓存(cache)** 中查找，若未命中则进入类的 **方法列表(method_list)** 中查找；在本类中未得到结果，再向父类发起查找，直至找到或进入根类型。

+ 动态加载
得益于运行时系统，OC可以在运行时期间创建新的 **类(Class)** ，对 **属性(ivar)** 和 **方法(method)** 进行查询、添加、修改。
    <!--\>运行时内容详见[运行时系统](#运行时)-->

### 2. 面向对象

#### 2.1 与C++的异同

+ 同

  + OC和C++都是基于C的、面向对象的高级语言
  + OC和C++都允许子类重写父类方法，然后实现动态绑定
  + ... ...

+ 异
  
  + C++允许多重继承，

    ```cpp
    class A : public B, public C, public D {
        // 
    };
    ```

    OC不允许直接继承多个类，但可以继承多个协议

    ```objective-c
    @interface A : NSObject<B, C, D>
    // 
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

+ **@interface**
类的定义，也被称为 **接口** 。所有的类都继承自NSObject。
在 **.h文件** 中编写时，可以定义 **公有** 的属性(property)和方法(method)，也可以使用限定符 *@public* 、 *@private* 、 *@protected* 和 *@package* 手动指定成员变量(ivar)的访问控制。
<br>

+ **@implementation**
对定义的实现，除了已有定义的方法，还可以实现未在接口中定义的 **“私有”** 方法。
<br>

    ><font color=PaleVioletRed>注意</font>
    >OC并没有真正意义上的 **私有** 属性和方法，所谓的 **公有** 属性和方法是通过 **.h文件** 暴露出去的，因此在 **.m文件** 中，不管使用什么限定符来修饰新定义的成员变量，它都是 **私有** 的。

---

## Swift

---

## iOS开发
