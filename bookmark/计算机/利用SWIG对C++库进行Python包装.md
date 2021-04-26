

# 利用 SWIG 对 C++ 库进行 Python 包装

> via: https://segmentfault.com/a/1190000013219667

如果你也像我们一样，同时使用Python和C++，以获得两种语言的优势，一定也会希望寻找一种好的方式集成这两种语言，相比而言，让Python能够方便使用C++的库更加重要，我们选择SWIG来实现这一需求，原因请见”途径”一节对几种实现途径的比较。

这篇博文介绍使用SWIG将C++库包装成Python接口，建议将”常用功能说明”之后的内容当做参考使用，因为那些内容牵涉到C++语言的各个特性，但不影响对SWIG整体使用的理解，可以在需要时参考。

另外，这篇博文中有很多例示代码，解释不多。是因为我觉得例示代码本身是很好的解释，清楚、准确、简练。如有问题，欢迎留言交流。

# 转载者的话 by [amc](https://segmentfault.com/u/amc)

本文章除了本小节是[博主 amc](https://segmentfault.com/u/amc) 写的之外，其他部分均为转载，不同的是博主尽可能还原了原文的格式。

本文转载自这里：“[利用SWIG对C++库进行Python包装](http://blog.csdn.net/xxxl/article/details/8288387)”，然而这篇文章也是转载的，但是却不注明出处！！而原文到底是哪一篇、在哪里，也找不到了……

此外，那篇文章转载了之后，但是只是普通的复制粘贴，格式什么的根本就不管，导致文章看起来实在是太难受了，所以我干脆自己转载一份好了。如果本文章的原作者看到了，**请务必联系我，我会将最原创的链接完整附上**。

最后，感谢原创者的贡献，同时谴责一下转载不注明出处的博主们。

# 途径

为C++库提供Python接口有以下几种常见途径:

### Python C API

Python解释器提供的一组C API，利用这组API，可以用C/C++实现Python module，也可以将Python解释器做为一个脚本引擎嵌入到C/C++程序中，为C/C++程序提供运行Python脚本的能力。Python C API是其他途径的基础，其他途径最终都以某种方式以Python C API实现。然而，直接使用Python C API相当繁琐，容易出错，因此很少直接使用。

### ctypes

ctypes是Python标准库提供的调用动态链接库的模块，使用这个模块可以直接在Python里加载动态链接库，调用其中的函数。使用ctypes 的优势是门槛低，不用编写或修改C/C++代码。然而我只简单地使用过这种方式，没有深入研究，不了解它对C/C++的支持是否完整。

### Boost.Python

Boost.Python是Boost提供的一个C++的模板库，用以支持Python和C++的无缝互操作。相对SWIG来说，这个库的优势是功能通过C++ API完成，不用学习写新的接口文件。对C++的支持更自然、完整。这个库的问题是：1)有外部依赖；2)文档不好，我看到有人说他看到三个不同的Boost.Python的tutorial，而这三个tutorial却完全不一样。我花了两个小时尝试Boost.Python，连tutorial的例子都没跑通，就放弃了。

### SWIG

SWIG是本文描述的重点，也是我们采用的途径。SWIG完整支持ANSI C，支持除嵌套类外的所有C++特性。SWIG是一个接口编译器，旨在为C/C++方便地提供脚本语言接口。SWIG不仅可以为C/C++程序生成 Python接口，目前可以生成CLISP,Java,Lua,PHP,Ruby,Tcl等19种语言的接口。SWIG被Subversion, wxPython, Xapian等项目使用。值得一提的是，Google也使用SWIG。

# SWIG的工作方式

SWIG本质上是个代码生成器，为C/C++程序生成到其他语言的包装代码(wrapper code)，这些包装代码里会利用各语言提供的C API，将C/C++程序中的内容暴露给相应语言。为了生成这些包装代码，SWIG需要一个接口描述文件，描述将什么样的接口暴露给其他语言。

SWIG的 接口描述文件可以包含以下内容：

1. ANSI C函数原型声明
2. ANSI C变量声明
3. SWIG指示器(directive)相关内容

SWIG可以直接接受 `.h` 头文件做为接口描述文件。在有了接口描述文件后，就可以利用 swig 命令生成包装代码了，然后将包装代码编译链接成可被其他语言调用的库。

# SWIG对Python支持到何种程度？

利用SWIG，可以现实以下功能：

- 用Python调用C/C++库
- 用Python继承C++类，并在Python中使用该继承类
- C++使用Python扩展（通过文档描述应该可以支持，未验证）

# 版本说明

SWIG的最新版本为2.0.1。因为我们现在使用的SWIG版本为1.3.40，本篇博客里的说明仅针对1.3.40版

# SWIG文档说明

SWIG的文档非常详尽，我甚至觉得太过详尽，不可能全看。我刚开始因为对SWIG文档组织不熟悉，看完一部分SWIG Basices就开始尝试，一路摸索到可以使用，后来才发现SWIG还有针对Python的专门文档。相比之下我之前摸索到的方案相当丑陋。

SWIG文档大体分两部分：

1. 一部分为SWIG本身：SWIG基本使用，对C及C++的支持，SWIG库及扩展等
2. 另一部分为SWIG对每一个目标语言的文档，如SWIG和Python的文档。

我建议只看和具体语言相关的文档，遇到问题时再去看SWIG本身的相关部分。

这篇博文应该会描述到用SWIG对C++进行Python包装的各个方面，不过喜欢原汁原味且有充足时间又comfortable with English的同学可直接看SWIG的文档。

# SWIG包含的内容

SWIG包含以下几部分内容：

- 一个代码生成器(swig)：代码生成器根据接口说明文件，生成相应的包装代码。
- 一个库：SWIG将常用的内容放到SWIG库里了，比如对数组、指针的支持，字符串支持，STL支持等。可以在接口文件中直接引用库里的内容，大大方便接口文件的编写。
- 一个简单示例

本节给出一个简单示例，提供对SWIG的直观认识，文章末尾处给出了一个更完整的例子。

**example.h**

```
#include <iostream>
using namespace std;
class Example{
    public:
    void say_hello();
};
```

**example.cpp**

```
#include "example.h"

void Example::say_hello(){
    cout<<"hello"<<endl;
}
```

**example.i**

```
%module example
%{
#include "example.h"
%}
%include "example.h"
setup.py
#!/usr/bin/env python

"""
setup.py file for SWIG C\+\+/Python example
"""
from distutils.core import setup, Extension

example_module = Extension('_example',
    sources=['example.cpp', 'example_wrap.cxx',],
)
setup (
    name = 'example',
    version = '0.1',
    author = "www.99fang.com",
    description = """Simple swig C\+\+/Python example""",
    ext_modules = [example_module],
    py_modules = ["example"],
)
```

运行以下命令:

```
swig -c\+\+ -python example.i
python setup.py build_ext --inplace
```

如果编译无误的话，就可以测试啦：

```
>>> import example
>>> example.Example().say_hello()
hello
```

以上我用distutils构建了example module，也可以通过编译器直接构建, 比如：

```
gcc -fPIC -I/usr/include/python2.5/ -lstdc\+\+ -shared -o _example.so example_wrap.cxx example.cpp
```

注意，-fPIC和-lstdc++都是必要的。_example.so前的’_'也是必要的。

# SWIG生成代码说明

`swig -c++ -python example.i` 命令生成了两个文件：`example_wrap.cxx`, `example.py`。`example_wrap.cxx`里会对`Example`类提供类使以下的扁平接口:

```
Example* new_Example();
void say_hello(Example* example);
viod delete_Example(Example *example);
```

这个接口被编译到`_example.so`里，_example可以做为一个 Python module 直接加载到 Python 解释器中。 `example.py` 利用 _example 里提供的接口，将扁平的接口还原为 Python 的 Example 类，这个类做为 C++ Example 类的代理类型，这样使用方式就更加自然了。

# SWIG接口文件的结构

SWIG 接口文件指导 SWIG 生成包装代码，其中包含 `%module` 声明，接口声明 `(%include “example.h”)`，以及 `%{ … %}` 中的内容。`%{ … %}` 中的内容会原封不动地拷贝到生成的包装代码中，上节例子中的 `#include “example.h”` 是必要的，因为接口声明中仅是声明接口中要暴露哪些内容(Example类)，但如果没有 `#include “example.h”` 的话，生成的包装代码是无法通过编译的。

# 常用功能说明

### 处理输入输出参数

C++包装的一个常见问题是有的C++函数以指针做为函数参数, 如：

```
void add(int x, int y, int *result) {
    *result = x + y;
}
```

或

```
int sub(int *x, int *y) {
    return *x-*y;
}
```

处理这种情况的最方便方式是使用SWIG库里的typemaps.i (关于SWIG库和Typemap见之后内容):

```
%module example
%include "typemaps.i"

void add(int, int, int *OUTPUT);
int sub(int *INPUT, int *INPUT);

>>> a = add(3,4)
>>> print a
7
>>> b = sub(7,4)
>>> print b
3
```

另一种写法：

```
%module example
%include "typemaps.i"

%apply int *OUTPUT { int *result };
%apply int *INPUT { int *x, int *y};

void add(int x, int y, int *result);
int sub(int *x, int *y);
```

对于既是输入又是输出参数的处理：

```
void negate(int *x) {
*x = -(*x);
}
-----------------------------
%include "typemaps.i"
...
void negate(int *INOUT);

-----------------------------
>>> a = negate(3)
>>> print a
-3
```

对于多个返回参数的处理：

```
/* send message, return number of bytes sent, along with success code */
int send_message(char *text, int len, int *success);
-----------------------------

%module example
%include "typemaps.i"
%apply int *OUTPUT { int *success };
...
int send_message(char *text, int *success);
-----------------------------

bytes, success = send_message("Hello World")
if not success:
print "Whoa!"
else:
print "Sent", bytes
```

当输出都通过参数给出情况的处理：

```
void get_dimensions(Matrix *m, int *rows, int *columns);

%module example
%include "typemaps.i"
%apply int *OUTPUT { int *rows, int *columns };
...
void get_dimensions(Matrix *m, int *rows, *columns);

>>> r,c = get_dimensions(m)
```

注意，typemaps.i只支持了基本数据类型，所以不能写void foo(Bar *OUTPUT);，因为typemaps.i里没有对Bar定义OUTPUT规则。

### C数组实现

有的C函数要求传入一个数组作为参数，调用这种函数时不能直接传入一个Python list或tuple, 有三种方式能解决这个问题：

使用类型映射(Typemap), 将数组代码生成为Python list或tuple相应代码使用辅助函数，用辅助函数生成和操作数组对象，再结合在接口文件中插入一些Python代码，也可使Python直接传入list或tuple。这种方式在之后说明。使用SWIG库里的carrays.i
这里先介绍carrays.i方式：

```
int sumitems(int *first, int nitems) {
    int i, sum = 0;
    for (i = 0; i < nitems; i\+\+) {
        sum += first[i];
    }
    return sum;
}
```

------

```
%include "carrays.i"
%array_class(int, intArray);
```

------

```
>>> a = intArray(10000000) # Array of 10-million integers
>>> for i in xrange(10000): # Set some values
... a[i] = i
>>> sumitems(a,10000)
49995000
```

通过 `%array_class` 创建出来的数组是C数组的直接代理，非常底层和高效，但是，它也和C数组一样不安全，一样没有边界检查。

### C/C++辅助函数

可以通过辅助函数来完一些SWIG本身不支持的功能。事实上，辅助函数可谓SWIG包装的瑞士军刀，一旦了解它使用，你可以使SWIG支持几乎所有你需要的功能，不过提醒一下，有很多C++特性是SWIG本身支持或者通过库支持的，不需要通过辅助函数实现。

同样的，直接上例示代码：

```
void set_transform(Image *im, double m[4][4]);

>>> a = [
... [1,0,0,0],
... [0,1,0,0],
... [0,0,1,0],
... [0,0,0,1]]
>>> set_transform(im,a)
Traceback (most recent call last):
File "<stdin>", line 1, in ?
TypeError: Type error. Expected _p_a_4__double
```

可以看到，set_transform是不能接受Python二维List的，可以用辅助函数帮助实现：

```
%inline %{
    /* Note: double[4][4] is equivalent to a pointer to an array double (*)[4] */
    double (*new_mat44())[4] {
    return (double (*)[4]) malloc(16*sizeof(double));
}
void free_mat44(double (*x)[4]) {
    free(x);
}
void mat44_set(double x[4][4], int i, int j, double v) {
    x[i][j] = v;
}
double mat44_get(double x[4][4], int i, int j) {
    return x[i][j];
}
%}

>>> a = new_mat44()
>>> mat44_set(a,0,0,1.0)
>>> mat44_set(a,1,1,1.0)
>>> mat44_set(a,2,2,1.0)
...
>>> set_transform(im,a)
>>>
```

当然，这样使用起来还不够优雅，但可以工作了，接下来介绍通过插入额外的Python代码来让使用优雅起来。

### 插入额外的Python代码

为了让set_transform函数接受Python二维list或tuple，我们可以对它的Python代码稍加改造：

```
void set_transform(Image *im, double x[4][4]);

...
/* Rewrite the high level interface to set_transform */
%pythoncode %{
    def set_transform(im,x):
    a = new_mat44()
    for i in range(4):
    for j in range(4):
    mat44_set(a,i,j,x[i][j])
    _example.set_transform(im,a)
    free_mat44(a)
%}

>>> a = [
... [1,0,0,0],
... [0,1,0,0],
... [0,0,1,0],
... [0,0,0,1]]
>>> set_transform(im,a)
```

SWIG还提供了%feature(“shadow”), %feature(“pythonprepend”), %feature(“pythonappend”)来支持重写某函数的代理函数，或在某函数前后插入额外代码，在%feature(“shadow”)中 可用$action来指代对C++相应函数的调用：

```
%module example

// Rewrite bar() python code

%feature("shadow") Foo::bar(int) %{
def bar(*args):
#do something before
$action
#do something after
%}

class Foo {
    public:
       int bar(int x);
}
```

或者：

```
%module example

// Add python code to bar() 

%feature("pythonprepend") Foo::bar(int) %{
#do something before C\+\+ call
%}

%feature("pythonappend") Foo::bar(int) %{
#do something after C\+\+ call
%}

class Foo {
public:
int bar(int x);
}
```

### 用%extend指示器扩展C++类

你可以通过%extend指示器扩展C++类，甚至可用通过这种方式重载Python运算符：

```
%module example
%{
#include "someheader.h"
%}

struct Vector {
   double x,y,z;
};

%extend Vector {
char *__str__() {
    static char tmp[1024];
    sprintf(tmp,"Vector(%g,%g,%g)", $self->x,$self->y,$self->z);
    return tmp;
}
Vector(double x, double y, double z) {
    Vector *v = (Vector *) malloc(sizeof(Vector));
    v->x = x;
    v->y = y;
    v->z = z;
    return v;
}

Vector __add__(Vector *other) {
    Vector v;
    v.x = $self->x + other->x;
    v.y = $self->y + other->y;
    v.z = $self->z + other->z;
    return v;
}

};

>>> v = example.Vector(2,3,4)
>>> print v
Vector(2,3,4)
>>> v = example.Vector(2,3,4)
>>> w = example.Vector(10,11,12)
>>> print v+w
Vector(12,14,16)
```

注意，在%extend里this用$self代替。

### 字符串处理

SWIG将char *映射为Python的字符串，但是Python字符串是不可修改的（immutable），如果某函数有修改char* ，很可能导致Python解释器崩溃。对由于这种情况，可以使用SWIG库里的cstring.i。

# 模块

SWIG通过%module指示器指定Python模块的名字

# 函数及回调函数

全局函数被包装为%module指示模块下的函数，如：

```
%module example
int add(int a, int b);

>>>import example
>>>print example.add(3, 4)
7
```

# 全局变量

SWIG创建一个特殊的变量’cvar’来存取全局变量，如：

```
%module example
%inline %{
double density = 2.5;
%}

>>>import example
>>>print example.cvar.density
2.5
```

inline是另一个常见的SWIG指示器，用来在接口文件中插入C/C++代码，并将代码中声明的内容输出到接口中。

# 常量和枚举变量

用#define, enum或者%constant指定常量：

```
#define PI 3.14159
#define VERSION "1.0"

enum Beverage { ALE, LAGER, STOUT, PILSNER };

%constant int FOO = 42;
%constant const char *path = "/usr/local";
```

# 指针，引用，值和数组

SWIG完整地支持指针：

%module example

```
FILE *fopen(const char *filename, const char *mode);
int fputs(const char *, FILE *);
int fclose(FILE *);

>>> import example
>>> f = example.fopen("junk","w")
>>> example.fputs("Hello World\n", f)
>>> example.fclose(f)
>>> print f
<Swig Object at _08a71808_p_FILE>
>>> print str(f)
_c0671108_p_FILE
```

指针的裸值可以通过将指针对象转换成int获得，不过，无法通过一个int值构造出一个指针对象。

```
>>> print int(f)
135833352
```

’0′或NULL被表示为None.

对指针的类型转换或运算必须通过辅助函数完成，特殊要注意的是，对C++指针的类型转换，应该用C++方式的转换，而不是用C方式的转换，因为在转换无法完成是，C++方式的转换会返回NULL，而C方式的转换会返回一个无效的指针：

```
%inline %{
/* C-style cast */
Bar *FooToBar(Foo *f) {
    return (Bar *) f;
}

/* C\+\+-style cast */
Foo *BarToFoo(Bar *b) {
   return dynamic_cast<Foo*>(b);
}

Foo *IncrFoo(Foo *f, int i) {
   return f+i;
}
```

在C++中，函数参数可能是指针，引用，常量引用，值，数据等，SWIG将这些类型统一为指针类型处理（通过相应的包装代码）:

```
void spam1(Foo *x); // Pass by pointer
void spam2(Foo &x); // Pass by reference
void spam3(const Foo &x);// Pass by const reference
void spam4(Foo x); // Pass by value
void spam5(Foo x[]); // Array of objects

>>> f = Foo() # Create a Foo
>>> spam1(f) # Ok. Pointer
>>> spam2(f) # Ok. Reference
>>> spam3(f) # Ok. Const reference
>>> spam4(f) # Ok. Value.
>>> spam5(f) # Ok. Array (1 element)
```

返回值是也同样的：

```
Foo *spam6();
Foo &spam7();
Foo spam8();
const Foo &spam9();
```

这些函数都会统一为返回一个Foo指针。

结构和类，以及继承
结构和类是以Python类来包装的：

```
struct Vector {
   double x,y,z;
};

>>> v = example.Vector()
>>> v.x = 3.5
>>> v.y = 7.2
>>> print v.x, v.y, v.z
7.8 -4.5 0.0
>>>
```

如果类或结构中包含数组，该数组是通过指针来操纵的：

```
struct Bar {
int x[16];
};

>>> b = example.Bar()
>>> print b.x
_801861a4_p_int
>>>
```

对于数组赋值，SWIG会做数据的值拷贝：

```
>>> c = example.Bar()
>>> c.x = b.x # Copy contents of b.x to c.x
```

但是，如果一个类或结构中包含另一个类或结构成员，赋值操作完全和指针操作相同。
对于静态类成员函数，在Python中有三种访问方式:

```
class Spam {
    public:
        static int bar;
        static void foo();
};

>>> example.Spam_foo() # Spam::foo()
>>> s = example.Spam()
>>> s.foo() # Spam::foo() via an instance
>>> example.Spam.foo() # Spam::foo(). Python-2.2 only
```

其中第三种方式Python2.2及以上版本才支持，因为之前版本的Python不支持静态类成员函数。

静态类成员变量以全局变量方式获取：

```
>>> print example.cvar.Spam_bar
```

SWIG支持C++继承，可以用Python工具函数验证这一点：

```
class Foo {
    ...
};

class Bar : public Foo {
    ...
};

>>> b = Bar()
>>> instance(b,Foo)
1
>>> issubclass(Bar,Foo)
1
>>> issubclass(Foo,Bar)
0
```

同时，如果有形如void spam(Foo *f);的函数，可以传b = Bar()进去。

SWIG支持多继承。

# 重载

SWIG支持C++重载：

```
void foo(int);
void foo(char *c);

>>> foo(3) # foo(int)
>>> foo("Hello") # foo(char *c)
但是，SWIG不能支持所有形式的C++重载，如：

void spam(int);
void spam(short);
```

或

```
void foo(Bar *b);
void foo(Bar &b);
```

这种形式的声明会让SWIG产生警告，可以通过重名命或忽略其中一个来避免这个警告：

```
%rename(spam_short) spam(short);
```

或

```
%ignore spam(short);
```

# 运算符重载

SWIG能够自动处理运算符重载：

```
class Complex {
    private:
        double rpart, ipart;
    public:
        Complex(double r = 0, double i = 0) : rpart(r), ipart(i) { }
        Complex(const Complex &c) : rpart(c.rpart), ipart(c.ipart) { }
        Complex &operator=(const Complex &c);
        
        Complex operator+=(const Complex &c) const;
        Complex operator+(const Complex &c) const;
        Complex operator-(const Complex &c) const;
        Complex operator*(const Complex &c) const;
        Complex operator-() const;
        
        double re() const { return rpart; }
        double im() const { return ipart; }
};

>>> c = Complex(3,4)
>>> d = Complex(7,8)
>>> e = c + d
>>> e.re()
10.0
>>> e.im()
12.0
>>> c += d
>>> c.re()
10.0
>>> c.im()
12.0
```

如果重载的运算符不是类的一部分，SWIG无法直接支持，如：

```
class Complex {
    ...
    friend Complex operator+(double, const Complex &c);
    ...
};
```

这种情况下SWIG是报一个警告，不过还是可以通过一个特殊的函数，来包装这个运算符：

```
%rename(Complex_add_dc) operator+(double, const Complex &);
```

不过，有的运算符无法清晰地映射到Python表示，如赋值运算符，像这样的重载会被忽略。

# 名字空间

名字空间不会映射成Python的模块名，如果不同名字空间有同名实体要暴露到接口中，可以通过重命名指示器解决：

```
%rename(Bar_spam) Bar::spam;

namespace Foo {
    int spam();
}

namespace Bar {
    int spam();
}
```

# 模板

SWIG对C/C++的包装是二进制级别的，但C++模板根本不是二进制级别的概念，所以对模板的包装需要将模板实例化，SWIG提供%template指示器支持这项功能：

```
%module example
%{
#include "pair.h"
%}

template<class T1, class T2>
struct pair {
    typedef T1 first_type;
    typedef T2 second_type;
    T1 first;
    T2 second;
    pair();
    pair(const T1&, const T2&);
    ~pair();
};

%template(pairii) pair<int,int>;

>>> import example
>>> p = example.pairii(3,4)
>>> p.first
3
>>> p.second
4
```

如果你要同时映射一个模板，以及以这个模板为参数的另一个模板，还要做一点特殊的工作, 比如，同时映射pair< string, string >和 vector< pair <string, string> >，需要像下面这样做：

```
%module testpair
%include "std_string.i"
%include "std_vector.i"
%include "std_pair.i"
%{
#include <string>
#include <utility>
#include <vector>
using namespace std;
%}

%template(StringPair) std::pair<std::string ,std::string>;
SWIG_STD_VECTOR_SPECIALIZE_MINIMUM(StringPair, std::pair< std::string, std::string >);
%template(StringPairVector) std::vector< std::pair<std::string, std::string> >;
```

遗憾的是，我并没有在文档中发现对这种做法的说明，以上做法是在swig用户组中问到的。

# 智能指针

有的函数的返回值是智能指针，为了调用这样的函数，只需要对智能指针类型做相应声明：

```
%module example
...
%template(SmartPtrFoo) SmartPtr<Foo>;
...

>>> p = example.CreateFoo() # CreatFool()返回一个SmartPtr<Foo>
>>> p.x = 3 # Foo::x
>>> p.bar() # Foo::bar
```

可以通过p.__deref__()得到相应的Foo*

# 引用记数对象支持

对于使用引用记数惯例的C++对象，SWIG提供了%ref和%unref指示器支持，使用Python里使用时不用手工调用ref和unref函数。因为我们目前没有使用引用记数技术，具体细节这里不详述了。

# 内存管理

SWIG是通过在Python里创建C++相应类型的代理类型来包装C++的，每个Python代理对象里有一个.thisown的标志，这个标志 决定此代理对象是否负责相应C++对象的生命周期：如果.thisown这个标志为1，Python解释器在回收Python代理对象时也会销毁相应的 C++对象，如果没有这个标志或这个标志的值是0，则Python代理对象回收时不影响相应的C++对象。

当创建对象，或通过值返回方式获得对象时，代理对象自动获得.thisown标志。当通过指针方式获得对象时，代理对象.thisown的值为0：

```
class Foo {
    public:
        Foo();
        Foo bar();
        Foo *spam();
};

>>> f = Foo()
>>> f.thisown
1
>>> g = f.bar()
>>> g.thisown
1
>>> f = Foo()
>>> s = f.spam()
>>> print s.thisown
0
```

当这种行为不是期望的行为的时候，可以人工设置这个标志的值：

```
>>> v.thisown = 0
```

# 跨语言多态

当你希望用Python扩展（继承）C++的类型的时候，你就需要跨语言多态支持了。SWIG提供了一个调度者(director)特性支持此功能，但此特性默认是关闭的，通过以下方式打开此特性：

首先，在module指示器里打开

```
%module(directors="1") modulename
```

其次，通过%feature指示器告诉SWIG哪些类和函数需要跨语言多态支持：

```
// generate directors for all classes that have virtual methods
%feature("director"); 

// generate directors for all virtual methods in class Foo
%feature("director") Foo; 

// generate a director for just Foo::bar()
%feature("director") Foo::bar;
```

可以使用%feature(“nodirector”)指示器关闭某个类型或函数的的跨语言多态支持：

```
%feature("director") Foo;
%feature("nodirector") Foo::bar;
```

# 类型映射(Typemaps)

类型映射是SWIG最核心的一部分，类型映射就是告诉SWIG对某个C类型，生成什么样的代码。不过，SWIG的文档里说类型映射是SWIG的高级自定义部分，不是使用SWIG需要理解的，除非你要提升自己的NB等级

以下的类型映射可用于将整数从Python转换为C:

```
%module example

%typemap(in) int {
    $1 = (int) PyLong_AsLong($input);
    printf("Received an integer : %d\n",$1);
}
%inline %{
    int add(int a, int b){
    return a+b;
}
%}

>>> import example
>>> example.add(3,4)
Received an integer : 3
Received an integer : 4
7
```

# SWIG库

SWIG提供了一组库文件，用以支持常用的包装，如数组，标准库等。可以在接口文件中引入这些库文件。比如，在%include “std_string.i”后，就可以直接给需要string参数数的函数传Python字符串了。对”std_vector.i”举例如下：

```
%module example
%include "std_vector.i"

namespace std {
%template(vectori) vector<int>;
};

>>> from example import *
>>> v = vectori()
>>> v.push_back(1)
>>> print v.size()
1
```

# 参考资料

- SWIG和Python: http://www.swig.org/Doc1.3/SWIGDocumentation.html#Python
- SWIG基础： http://www.swig.org/Doc1.3/SWIGDocumentation.html#SWIG
- SWIG和C++: http://www.swig.org/Doc1.3/SWIGDocumentation.html#SWIGPlus