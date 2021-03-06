## const 及作用

关键字 const 可以对变量类型加以限定。const 对象一旦创建后，其值就不能发生改变，所以 const 对象必须初始化。初始值可以是任意表达式。

默认状态下，const 对象仅在文件内有效。（见《C++ Primer》P54），比如有这么一个表达式：

```cpp
const int buffer = 512;
```

编译器在编译的时候，会把用到该变量的地方都替换成对应的值。为了执行上述操作，编译器必须知道变量的初始值。如果程序用了多个文件，则每一个用到 const 对象的文件都必须得能访问到它的初始值才行。要做到这一点，就必须在每一个用到变量的文件中都对它有定义。为了支持这一做法，默认情况下，const 对象被设定为仅在文件内有效。

某些时候有这样一种 const 变量，它的初始值不是一个常量表达式，但又确实有必要在文件间共享。这种情况下，我们不希望编译器为每个文件分别生成独立的变量。相反，我们想让这类 const 对象像其他对象一样工作，也就是说，只在一个文件中定义 const ，而在其他多个文件中声明并使用它。

解决的办法就是，对于 const 变量不管事声明还是定义都添加 extern 关键字，这样只需定义一次就可以了。

```cpp
//file1.cc 定义并初始化了一个常量，该常量能被其他文件访问
extern const int bufsize = fcn();
//file1.h 头文件
extern const int bufsize;//与 file1.cc 中定义的一个 bufsize 是同一个
```

如上述程序所示，`file1.cc` 定义并初始化了 bufsize .因为这条语句中包含了初始值，所以它是一次定义。然而，bufsize 是一个常量，必须用 extern 加以限定使其被其他文件使用。

### const 的引用

可以把引用绑定到 const 对象上，就像绑定到其他的对象上一样。称之为**对常量的引用**。对常量的引用不能被用作修改它绑定的对象。

```cpp
const int ci = 1024;
const int &r1 = ci;//正确，引用及其对应的对象都是常量

r1 = 42;//错误，r1 是对常量的引用
int &r2 = ci;//错误，试图让一个非常量引用指向一个常量对象
```

术语：常量引用是对 const 的引用。

### 初始化和对 const 的引用

引用的类型必须与其引用对象的类型一致，但有两个例外，第一个是

1. 在初始化常量引用时允许用任意表达式作为初始值，只要该表达式的结果能转换成引用的类型即可

```cpp
int i = 42;
const int &r1 = i;
const int &r2 = 42;//绑定一个临时量常量
const int &r3 = r1 * 2;//绑定临时量
```

### 对 const 的引用可能引用一个并非 const 对象

常量引用仅对引用可参与的操作做出了限定，对于引用的对象本身是不是一个常量没有限定。因为对象也可能是一个非常量。所以允许通过其他途径改变他的值。

```cpp
int i = 42;
int &r1 = i;       //绑定对象 i
const int &r2 = i; //r2 也绑定对象 i，但是不允许修改 r2 来修改 i 的值
r1 = 0;            //可以，r1 不是常量
r2 = 0;            //错误，r2 是常量引用
```

### 指针和 const

与引用一样，也可以令指针指向常量或者非常量。类似于常量引用，**指向常量的指针**不能用于改变其所指对象的值。

```cpp
const double pi = 3.14;//定义 pi 不能改变
double *ptr = &pi; //错误，ptr 是一个普通指针
const double *cptr = &pi;//正确
*cptr = 42 ;//错误。不可改变

double dval = 3.14;
cptr = &dval;//正确，但是不能通过 cptr 修改 dval 的值
```

和常量引用一样，指向常量的指针也没有规定其所指对象必须是一个常量。所谓指向常量的指针，仅仅要求不能通过该指针改变对象的值。

> 指向常量的指针和引用，「自以为是」的以为自己指向了一个常量，所以不能改变值

### const 指针

指针是对象而引用不是。则允许把指针本身定义为常量。**常量指针**必须初始化，而且一旦初始化完成，则它的值（他也就是存放在指针中的那个地址）就不可变了。把 * 放在 const 关键字之前用来说明指针是一个常量，这样书写表明，不变的是指针本身的值而非指向的那个指。

```cpp
int e = 0;
int *const cur = &e;//cur将一直指向 e
const double pi = 3.14159;
const double *const pip = &pi;//pip 是一个指向常量对象的常量指针
```

> 指针常量：可以改变指向，不可以改变值
>
> 常量指针（指向常量的指针）：不能改变指向，但是可以改变值
>
> 常量指针常量：都不可以改变

### 顶层 const

因为指针是一个对象，所以指针是不是一个常量和指针所指的对象是不是一个常量是两个相互独立的问题。

用名词 **顶层 const** 表示指针本身是一个常量，用名词 **底层 const** 表示指针指向的对象是一个常量

```cpp
int i=0;
int *const p1 = &i;//不能改变 p1 的值，这是一个顶层 const
const int ci = 42;//不能改变 ci 的值，这是一个顶层 const
const int *p2 = &ci;//允许改变 p2 的值，这是一个底层 const
const int *const p3 = p2;//靠右的 const 是顶层 const ,靠左的 const 是底层 const
const int &r = ci;//用于声明引用的 const 都是底层 const
```

