C++源文件后缀名.cc/.cpp，头文件后缀名.hh/.hpp。
安装g++命令：**sudo apt install g++**。
**编译命令 g++ 文件名.cc/.cpp [-o name]**

pro git


vscode插件
c/c++
traecode
remote - ssh

C++中的头文件都是没有.h的。

using 声明机制
std：标准库的命名空间。
cout 标准输出流，默认输出设备，终端。
endl 换行
cin 标准输入流，默认输入设备，键盘。
<<是输出运算符。
\>>是输入运算符。

cout << "hello world" << endl;
int num;
cin >>num;

argc：命令行参数个数。
argv：具体的命令行参数。

设置模板：


vim多窗口操作技巧
```C++
:e 文件名（带路径） 若文件存在就打开，不存在就创建
:bp 打开上一个窗口
:bn 打开下一个窗口
:bd 关闭当前窗口
// 如果觉得每次切换窗口麻烦 可以使用Tab键进行替换 需要在.vimrc中进行设置
// 将如下内容添加到.vimrc中
nmap <Tab> :bn<cr>
```



# 1：命名空间
---
目的：为了解决命名冲突，引入命名空间，所谓命名空间就是一个可以由用户自己定
的作用域，在不同的作用域中可以定义相同名字的变量，互不干扰，系统能够区分它们。
## 1.1：命名空间的使用方式
定义命名空间的基本格式：
```C++
//以下定义一个名为liweixing的命名空间
namespace liweixing
{
    int value=10;
    char val2;
}
```
### 1.1.1：使用作用域限定符
作用域限定符就是::，每次要使用某个命名空间的实体时，都要加上作用域限定符::。
```C++
namespace liweixing
{
    int num = 10;
    void display()
    {
        //cout，endl都是std空间中的实体，所以前面要加上std::。
        std::cout << "这里是文本" << endl;   
    }
}

void test1()
{
    std::cout << "命名空间liweixing里面的num值为：" << liweixing::num;
    liweixing::display();
}
```
优点：准确。
缺点：繁琐。
### 1.1.2：使用using声明机制
using声明机制就是在文件开头加上using namespace 空间名。
```C++
#include <iostream>

using namespace std;    //这里使用std命名空间。在后面就不用加std::。

int main(int argc, char *argv[])
{
    cout << "hello world!" <<endl;    //这里cout，endl前都没有加std::。
    return 0;
}
```
### 1.1.3：使用using编译指令
using编译指令：需要哪个空间里面的变量就声明哪个。
```C++
#include <iostream>

//写在开头，就表示作用全局。如果想作用在局部，写在局部的{}里面。
using std::cout;
using std::endl;

int main()
{
    cout << "hello world!" << endl;
    
    return 0;
}
```
using编译指令尽量写在局部作用域。
## 1.2：命名空间的嵌套使用
特殊的命名空间：
1：嵌套命名空间。
2：匿名命名空间。
### 1.2.1：嵌套命名空间
嵌套命名空间就是命名空间里面定义命名空间。
使用的时候，需要先进入外层命名空间，在进入内层命名空间。
```C++
//外部命名空间
namespace outer
{
    int num1=1;
    void func1()
    {
        cout<< "outer::func1()"<< endl ;
    }

    //内部命名空间。
    namespace inner
    {
        int num2=2;
        void func2()
        {
            cout<<"inner::func2()"<<endl;
        }
    }
}

int main(int argc,char *argv[])
{
    using std::cout;
    using std::endl;
    
    cout << "inner num2:" << outer::inner::num2 <<endl;
    
    return 0;
}
```
### 1.2.2：匿名空间（了解）
匿名的命名空间：没有名字的命名空间
```C++
//注意：namespace后面应该有空间名字的，但是匿名空间没有。
namespace
{
    int num_1=9;
    void func_1()
    {
        std::cout << "匿名空间" << endl;
    }
}

//使用匿名命名空间实体。
void test()
{
    //可以直接使用。
    cout << num_1 << endl;
    func_1();
    
    //或者::num_1使用。
    std::cout << ::num_1 <<endl;
    ::func_1();
}
```
- 匿名空间中不要定义和全局变量同名的实体。
- 匿名空间不能跨模块使用。
## 1.3：跨模块调用问题
- 一个.c/.cc/\*.cpp的文件称为一个模块。
  跨模块就是在a文件使用b文件的变量。
这些结构可以跨模块：
    1：有名字的命名空间。
    2：全局变量和函数可以跨模块调用。
使用extern进行跨模块调用。
编译的时候联合编译。
```C++
//1：全局变量和函数。
在A文件有：
int num=100;
void test_1()
{
    cout << "hello" << endl;
}
//在B文件调用：
extern int num;    //外部引用声明。
extern void test_1();
//下面就可以正常使用。
//但是编译的时候要联合编译：
//g++ A.cpp B.cpp -o A.cpp

------------------------------------------------------------------

//2：又名空间中的实体可以跨模块调用。
在A文件中有：
namespace liweixing
{
    int num=10;
    void display()
    {
        cout << "hhhhh" << endl;
    }
}
在B文件中调用：
namespace liweixing
{
    //进行联合编译时，这两次定义被认为是同一个命名空间。
    extern int num;
    extern void display();
}
//然后可以正常使用。
//g++ A.cpp B.cpp -o A.cpp
```
- 静态变量和函数只能在本模块内部使用
- 匿名空间的实体只能在本模块内部使用。

头文件规范：
1. 先放自己定义的头文件。
2. 再放C的。
3. 再放C++的。
4. 最后放第三方的头文件。
# 2：const关键字
---
## 2.1：修饰内置类型
被const修饰的变量称为常量。被const修饰的变量定义是必须初始化。
<font color="#ff0000">但是，虽然是叫常量，但是还是可以通过一些方式修改它的值（数组越界，scanf（）输入）。</font>
## 2.2：修饰指针类型
分为两大类：
1：指向常量的指针。pointer to constant
2：常量指针。          constant pointer
## 2.3：区分
1. 注意区分：
   const int \*p = &a;
   int const \*p = &a;
   int * const p =&a;
   const int * const p = &a;
2. 注意区分：
   int \*p[10];
   int (\*p)[10];
3. 注意区分：
   int * test(int x, int y);
   int (\*test)(int x, int y);

## 2.4：const


# 3：new/delete运算符
---
## 3.1：new/delete运算符
new相当与C语言中的malloc，用于在堆上申请一块内存。
delete相当于C语言中的free，用于释放堆上的内存。
对于指针的置空，使用nullptr，不再使用NULL。
new语句可以不加参数，初始化为各类型默认值；也可以加参数，参数代表要初始化的值。
```C++
void test1()
{
    int *p1 = new int();    //默认为0；
    cout << *p1 << endl;
    
    //值初始化
    int *p2 = new int(10);    //初始化为10；
    int *p3 = new int{};    //可以使用{}，也可以使用().
    
    //释放内存
    delete p1;
    delete p2;
    delete p3;
    
    //数组数据
    int *arr1 = new int[3]();
    cout << arr[0] << endl;
    cout << arr[1] << endl;
    cout << arr[2] << endl;
    
    //数组一般使用{}
    int * arr2 = new int[3]{1,2,3};
    cout << arr[0] << endl;
    cout << arr[1] << endl;
    cout << arr[2] << endl;
    
    //释放连续内存空间。
    delete[] arr1;
    delete[] arr2;
}
```
## 3.2：常考面试题：
malloc/free和new/delete的区别。
    1：malloc/free是库函数，new/delete是运算符，后者使用时不是函数的写法。
    2：new类型返回的是相应类型的指针，malloc返回void * ；
    3：malloc申请之后不会初始化，new得到的是初始化后的数据。
    4：malloc的参数是字节，new不需要，会根据相应类型自动获取。
## 3.3：valgrind工具集
valgring工具集中的memcheck，可以检查内存泄露。
安装：**sudo apt install valgrind**

使用以下命令可以查看泄露信息。
命令：valgrind --tool=memcheck --leak-check=full ./a.out
命令长，不方便，可以设置：
1. 在home目录下编辑.bashrc文件，改别名：
   alias memcheck='valgrind --tool=memcheck --leak-check=full --show-reachable=yes'
2. 重新加载 source .bashrc
## 3.4：回收空间
```C++
//三组申请空间，回收空间的匹配组合
malloc            free
new               delete
new int[5]()      delete[]
```
另外，内存回收后，指向内存的指针要置空：
p1 = nullptr;
```C++
    int * p1 = new int();//初始化为该类型的默认值
    cout << *p1 << endl;
    delete p1;
    p1 = nullptr;//安全回收
```
# 4：引用（最重要）
---
## 4.1：引用介绍
什么是引用？
    就是别名，给变量起另外一个变量。通过别名间接影响本体。

语法：
数据类型 & 引用名 = 已经存在的变量或对象。
引用本质：特殊的指针，常量指针。引用一旦绑定，就不能修改。

使用场景：
    1：作为函数的参数。直接改变参数本体（相当于指针，但是比指针简便）。
    2：引用作为函数返回值。因为正常return时会进行复制操作，返回的是num的副本。
```C++
void test1()
{
    int num = 1;
    //定义一个引用绑定到这个变量上。
    int & refNum = num;
    
    //现在操纵refNum就是对num的操作。完全是一样的。
    cout << num << endl;
}
```
## 4.2：面试题：引用和指针的区别
1. 联系
   1：引用和指针都有地址的概念，都是用来间接访问变量。
   2：引用的底层还是指针来实现的，可以把引用视为一个受限的指针。
2. 区别
   1：引用必须初始化，指针可以不初始化。
   2：引用不能修改绑定，但是指针可以修改指向。
   3：在代码底层对引用本身取地址取到变量本体的地址，
     但是对指针取地址得到指针变量地址。
## 4.3：引用作为函数参数
```C++
//在实参传给swap时，
//其实就是发生了初始化int & x = a;
//int & y = b;
void swap(int & x, int & y){//引用传递，不复制
    int temp = x;
    x = y;
    y = temp;
}
```
参数传递的方式包括`值传递`、`指针传递`和`引用传递`。
### **常引用：使用const修饰的引用**
不能通过引用修改变量，通常用做函数参数。
    1：不会修改值。
    2：不会复制。
```C++
void test()
{
    int num = 10;
    int & inf_1 = num;                  //正常引用
    const int & inf_2 = num;            //常引用
    //既不能修改引用，也不能通过引用修改变量的值。
}

void test_1(const int & a, const int & b)
{
    //函数内部不能修改a, b的值。
}
```
## 4.4：引用作为函数的返回值
一定要保证：<font color="#ff0000">返回的变量其生命周期一定是大于函数的生命周期。</font>
目的：避免复制，节省开销。（<font color="#ff0000">正常返回要经历一次复制</font>）
```C++
int a = 1;
// 返回值为int类型
int  func(){
    //...
    return a;   //在函数内部，当执行return语句时，会发生复制 
}   

// 返回值为int类型的引用
int & func2(){
     //...
    return a;   //在函数内部，当执行return语句时，不会发生复制
}
//func2本质上是对全局变量a的修改。
```
**引用是否占用内存取决于使用场景：在简单局部使用时编译器可能完全不分配内存；但作为函数参数或类成员时，底层通常用指针实现，占 8 字节。**
# 5：强制转换
---
C语言中的强制转化在C++中，仍然可以使用。
```C++
int a = 10;
long b = (int)a;
```
但是，这种转化仍有缺点，缺乏安全检查，还会有其他问题。
```C++
void func()
{
    double a = 3.14;
    int *p = (int *)&a;    //编译虽然可以通过，但是会导致未定义行为。
    cout << *p << endl;    //输出结果无法预测。
}
```

C++引进了4个新的类型转化符：
1. static_cast
2. const_cast(了解)
3. dynamic_cast（继承部分学习）
4. reinterpret_cast(了解)
## 5.1：static_cast
这是用的最多的类型转化。格式如下：
`目标类型 转换后的变量 = static_cast<目标类型>(要转换的变量)`
1：支持基本数据类型之间的转换。
2：支持void * 类型转化为其他类型的转换。
3：<font color="#ff0000">不支持两个指针类型间的转换。（C++有检查，会报错。C语言没有检查，不会报错）</font>
```C++
//1：支持基本类型之间的转化。
int iNumber = 100;
float fNumber = 0;
fNumber = static_cast<float>(iNumber);

//2：把void指针转化成目标类型的指针，但是不安全。
void * pVoid = malloc(4);
int * pInt = static_cast<int *>(pVoid);

//注意: 不能完成任意两个指针类型间的转换
int iNumber = 1;
int * pInt = &iNumber;
float * pFloat = static_cast<float *>(pInt);    //错误写法。
```
## 5.2：const_cast(了解)
去除const属性。
该运算符用来修改指针/引用的const属性，基本不用，**仅做了解**。
# 6：函数重载
---
## 6.1：什么是函数重载
- C++允许多个函数拥有相同的名字，只要函数的入参不同就行。- 
- 函数的返回值类型，这里不关注。
- **在同一作用域内，可以有一组具有相同函数名，不同参数列表的函数，这组函数被称为重载函数。**
## 6.2：实现函数重载的条件
函数名相同，参数列表不同，可以构成重载：
1. 函数参数的数量。
2. 数量相同，类型不同。
3. 数量，类型都相同，参数顺序不同。
<font color="#ff0000">注意: 只有返回类型不同，参数完全相同，是不能构成重载的。</font>
```C++
// 函数名相同 参数列表不同
int add(int x, int y) {
    return x + y;
}

int add(int x, int y, int z) {
    return x + y + z;
}

int add(float x, int y) {
    return x + y;
}

int add(int x, float y) {
    return x + y;
}
```
## 6.3：函数重载的实现原理
实现原理： 名字改编(name mangling)机制——当函数名称相同时 ，会根据参数的类型、顺序、个数进行改编。
代码在编译的时候，会把重构的函数进行改名字，从而识别函数重载。
![[函数重载的原理.png]]

C语言没有名字改编机制，所以C语言没有函数重载。
缺点是，C++在编译的时候，比C语言多了一个步骤，效率有所降低。
## 6.4：extern “C”
如果想在C语言中，对部分内容按照C的方式进行编译，可以使用extern “C”
```C++
extern "C" void func() //用 extern"C"修饰单个函数
{

}

//如果是多个函数都希望用C的方式编译
//或是需要使用C语言的库文件
//都可以放到如下{}中
extern "C"
{
//……
}
```
![[extern_c.png]]
# 7：函数默认参数
---
## 7.1：什么是函数默认参数
函数默认参数，就是在定义函数的时候：
<font color="#ff0000">为一个或者多个参数提供默认值，当调用函数的时候，如果没有为这些参数传递具体的值，则使用默认值。</font>

函数默认参数的目的：
1. 函数调用的时候，进行缺省调用。
2. 减少函数重载。
## 7.2：默认参数声明
一般默认参数在，函数声明中提供。
```C++
//在定义的时候设置默认值。
void func(int x = 0, int y = 0){
    cout << "x = " << x << endl;
	cout << "y = " << y << endl;
}

void test0(){
    // 如果传参,实际参数就是传递的参数, 如果不传,那么参数就是默认参数
    func(24,30);
    func(100);
    func();
}
```
<font color="#ff0000">注意：设置默认值时，必须从右往左进行调用。</font>

如果函数重载，和默认值同时出现，在调用的时候可能会有二义性。
```C++
void func4(int x);
void func4(int x, int y = 0);

func4(1);    //error,编译器无法确定调用的是哪种形式的func4 
```
所以，重载的时候，要谨慎使用默认参数。
# 8：bool类型
---
bool类型是C++中的基本数据类型，只有false（0）和true（非0）。
bool类型强转为int类型，false为0，true为1。
bool类型占用一个字节。

<font color="#ff0000">注意：ture是非0，即使是负数也表示true。</font>
# 9：inline函数（了解）
---
## 9.1：什么是内联函数
内联函数：用inline修饰的函数。
用函数体来替换函数调用。
当内联函数接收到编译器的指示时，即可发生内联：编译器将使用函数的定义体来替换函数的调用语句，<font color="#ff0000">这种替换行为发生在编译阶段，而非程序运行阶段。</font>

内联的场景：
1：如果函数比较简短，调用比较频繁，适合内联。
2：如果函数臃肿，不适合使用。
```C++
// 内联函数
inline int max(int x, int y)
{
	return x > y ? x : y;
}

void test() {
    cout << max(1,2) << endl;
    // 在编译阶段 编译器采用了内联建议 内联展开:使用函数定义替代函数调用  cout << 1 > 2 ? 1:2 << endl
    // 使用内联函数 不会有函数调用的开销
}
```
## 9.2：宏 VS 内联函数

| 特性        | 内联函数 (`inline`)    | 宏 (`#define`)     |
| --------- | ------------------ | ----------------- |
| 类型安全      | 提供类型安全，编译器进行类型检查   | 没有类型检查，可能产生不匹配的错误 |
| 编译期替换     | 编译器决定是否内联（有优化机制）   | 预处理器简单文本替换        |
| 代码可读性和调试性 | 支持断点调试，可读性和普通函数相似  | 调试困难，无法跟踪宏的展开过程   |
| 副作用       | 参数只求值一次，不会有多次求值副作用 | 参数会多次求值，可能导致副作用   |
| 代码膨胀      | 函数被多次内联可能导致代码膨胀    | 频繁替换也会导致代码膨胀      |
| 灵活性       | 适用于明确类型的函数         | 可以处理不同类型的参数       |
| 性能        | 小型函数可以避免函数调用开销     | 无函数调用开销           |

## 9.3：内联函数注意事项
1. inline是一个建议，并不是强制性的，可能会失效。
2. inline的建议如果有效，就会在**编译时**展开，可以理解为是一种更高级的代码替换机制（类似于宏——预处理）
3. 函数体内容如果太长或者有循环之类的结构，不建议inline，以免造成代码膨胀；
   **比较短小并且比较常用的代码适合用inline。**
4. **如果要把inline函数声明在头文件中，则必须把函数定义也写在头文件中。**
# 10：异常处理（了解）
---
异常处理并非代码编辑错误，或者语法错误，而是逻辑错误，比如分母为0。
C++中，异常处理涉及到的三个关键字：try，catch，throw。
```C++
double division(double x,double y){
    if(y == 0)//如果y为0，那么就是会有异常，返回throw后的东西。
    {
        throw "Deivision by zero";
    }
    return x/y;
}

void test0(){
    double x = 100, y = 0;
    
    try    //try里面的代码是可能有异常的代码。
    {
        cout << "before" << endl;
        cout << division(x,y) << endl;//异常行后的代码不会执行。
        //如果division函数执行有异常，那么就返回throw后面的东西。
        //throw后，可以是字符串，可以是整形，等等。
        cout << "after" << endl;
    }
    catch(const char * msg)//catch的小括号里是类型，如果throw返回字符串就捕捉。
    { 
    	cout << "hello" << endl;
        cout << "hello," << msg << endl;
    }
    catch(double x)//如果throw返回double类型，就捕捉。
    {
        cout << "double" << endl;
    }
    catch(int x)//如果throw返回int类型，就捕捉。
    {
        cout << "int" << endl;
    }
    
    cout << "end test" << endl;
}
```
异常处理可以优雅的处理错误，并且可以继续执行C++程序，不用终止程序。
## 10.1：try-catch语句块
try-catch语句块中的catch可以有多个。
1. 如果没有发生异常，就不会执行catch，继续往后执行代码。
2. <font color="#ff0000">注意：catch是throw返回的类型，不是具体信息。</font>
## 10.2：throw语句
如果检测到异常，那么就抛出异常。
# 11：内存布局（重要）
---
![[内存布局.png]]
从高地址到低地址可以分为五个区域：
- 栈区：操作系统控制，由高地址向低地址生长，编译器做了优化，显示地址时栈区和其他区域保持一致的方向。
- 堆区：程序员分配，由低地址向高地址生长，堆区与栈区没有明确的界限。
- 全局/静态区：读写段（数据段），存放全局变量、静态变量。
- 文字常量区：只读段，存放程序中直接使用的常量，如const char * p = "hello"; hello这个内容就存在文字常量区。
- 程序代码区：只读段，存放函数体的二进制代码。
# 12：C风格字符串
## 12.1：字符数组形式
```C++
void test1(){
    // 字符数组形式
    char str1[] = "hello";
    // 等价于下面 最后一位'\0'
    char str2[] = {'h','e','l','l','o','\0'};
    
    // cout输出流运算符默认重载,连接char类型的数组名/指针名时
    // 输出的是内容,而不是地址
    cout << "str1 = " << str1 << endl; // hello
    cout << "str2 = " << str2 << endl; // hello
    str1[0] = 'H';
    cout << "str1 = " << str1 << endl; // Hello
}
```
字符数组最后一位是‘\0’：
sizeof（str1），结果为6。
strlen（str1），结果为5。
## 12.2：字符指针形式
```C++
// 字符指针形式
//c++标准 要使用const char *
// 即指向常量的指针,不能通过指针修改
const char *str1 = "hello";
const char *str2 = "world";

cout << "str1 = " << str1 << endl;
cout << "str2 = " << str2 << endl;

/* str1[0] = "H"; // error read only*/
str1 = "wd";
cout << "str1 = " << str1 << endl;
```