- **C++输入输出流机制**
C++ 的 I/O 发生在流中，流是字节序列。如果字节流是从设备（如键盘、磁盘驱动器、网络连接等）流向内存，这叫做输入操作。如果字节流是从内存流向设备（如显示屏、打印机、磁盘驱动器、网络连接等），这叫做输出操作。

- **C++常用流类型**
C++ 的输入与输出包括以下3方面的内容:
1. 对系统指定的标准设备的输入和输出。即从键盘输入数据，输出到显示器屏幕。这种输入输出称为标准的输入输出，简称**标准** I/O 。
2. 以外存磁盘文件为对象进行输入和输出，即从磁盘文件输入数据，数据输出到磁盘文件。以外存文件为对象的输入输出称为文件的输入输出，简称**文件** I/O 。
3. 对内存中指定的空间进行输入和输出。通常指定一个字符数组作为存储空间（实际上可以利用该空间存储任何信息）。这种输入和输出称为字符串输入输出，简称**串** I/O 。
![[流的种类.png]]
![[流.png]]
# 1：流的四种状态（重点）
流的状态有四种：
- **badbit** 表示发生**系统级的错误**，如不可恢复的读写错误。通常情况下一旦 badbit 被置位，流就无法再使用了。
- **failbit** 表示发生**可恢复的错误**，如期望读取一个int数值，却读出一个字符串等错误。这种问题通常是可以修改的，流还可以继续使用。
- **eofbit**表示**到达流结尾位置**， 流在正常输入输出的情况下结束，会被置为eofbit状态。
- **goodbit** 表示流处于**有效状态**。流在有效状态下，才能正常使用。如果 badbit 、 failbit 和 eofbit 任何一个被置位，则流无法正常使用。

这四种状态都定义在类<font color="#ff0000"> ios_base </font>中，作为其数据成员存在。
![[ios_base.png]]

通过以下四个函数，可以得到流的状态：
```C++
bool good() const      //流是goodbit状态，返回true，否则返回false
bool bad() const       //流是badbit状态，返回true，否则返回false
bool fail() const      //流是failbit状态，返回true，否则返回false
bool eof() const       //流是eofbit状态，返回true，否则返回false
```
cin是一个输入流对象，以cin为例：
```C++
// 以cin为例 cin是istream 这个类中的全局对象
// istream & is = cin
void printState(istream & is)
{
    cout << "good:" << is.good() << endl;
    cout << "fail:" << is.fail() << endl;
    cout << "bad:" << is.bad() << endl;
    cout << "eof:" << is.eof() << endl;
}
```
# 2：标准输入输出流
对系统指定的标准设备的输入和输出。即从键盘输入数据，输出到显示器屏幕。这种输入输出称为标准输入输出，简称**标准** I/O。
C++标准库定义了三个预定义的标准输入输出流对象:
- **标准输入流 (`std::cin`)**：从键盘（或其他输入设备）读取数据。
- **标准输出流 (`std::cout`)**：向标准输出设备(屏幕)（或其他输出设备）输出数据。
- **标准错误流 (`std::cerr`)**：用于向屏幕输出错误信息，不缓冲。
标准输入、输出的内容包含在头文件iostream中。
## 2.1：标准输入流
istream 类定义了一个全局输入流对象，即 cin , 代表的是**标准输入**，它从标准输入设备(键盘)获取数据，程序中的变量通过流提取符 “>>”（输入流符号） 从流中提取数据。

流提取符 “>>” 从流中提取数据时通常跳过输入流中的空格、 tab 键、换行符等空白字符, 会把这些**空白字符作为分隔符**, 只有在输入完数据再按回车键后，该行数据才被送入键盘**缓冲区**，形成输入流，提取运算符 “>>” 才能从中提取数据。需要注意保证从流中读取数据能正常进行。（流的缓冲机制在下一节中学习）
```C++
void printStreamStatus(std::istream & is){ 
    cout << "is's goodbit:" << is.good() << endl;
    cout << "is's badbit:" << is.bad() << endl;
    cout << "is's failbit:" << is.fail() << endl;
    cout << "is's eofbit:" << is.eof() << endl;
}

void test0(){
    printStreamStatus(cin);  //goodbit状态
    int num = 0;    
    cin >> num;   
    cout << "num:" << num << endl;
    printStreamStatus(cin);  //进行一次输入后再检查cin的状态
}
```
如果没有进行正确的输出，输入流会进入failbit状态，无法正常工作，需要恢复流的状态。
可以利用<font color="#ff0000">clean（）和ignore（）</font>函数配合使用。
clean（）：恢复流的状态。
ignore（）：清空缓冲区。
```C++
//忽略缓冲区的所有剩余字符 
cin.ignore(std::numeric_limits<std::streamsize>::max(),'\n');
```
举例如下：
```C++
void test()
{
    int num = 10;
    cout << "执行输入操作前流的状态:" << endl;
    printStreamStatus(cin);
    
    cin >> num;
    cout << "执行输入操作后流的状态:" << endl;
    printStreamStatus(cin);
    
    if(!cin.good())
    {
        // 恢复流的状态
        cin.clear();
        // 清空缓冲区 后才能使用
        // 忽略当前输入流中的所有字符，直到遇到换行符为止. 
        //这种方法适用于清空缓冲区，因为它可以跳过所有字符。
        cin.ignore(std::numberic_limits<std::streamsize>::max(), '\n');
        printStreamStatus(cin);    
    }
    
    // 没有正常输入num被设置为0。
    cout << "num=" << num << endl;
    string line;
    cin >> line;
    cout << "line:" << line << endl;
}
```
补充：注意cin表达式的返回值:
- cin对象作为条件时,隐式转换为布尔类型
- cin对象完成一次输入后,返回值为自身对象,可以进行连续链式的输入
```C++
void test1()
{
    int num;
    cout << "input a num: " << endl;
    cin >> num;
    /* if(cin.good()){ */
    // cin的状态是goodbit状态, 在作为判断条件时隐式转换为布尔类型 这里为true
    if(cin)
    {
        cout << "num = " << num;
        cout << endl;
    }
}

void test2()
{
    int num;
    cout << "input a num: " << endl;
    // cout << &cin << endl;
    cin >> num;
    // cout << &cin << endl;
    // 输入前后cin地址相同 是同一对象
    
    // 连续输入
    int num1, num2;
    cin >> num1 >> num2;
    cout << num1 << ", " << num2 << endl;
 }
```
通用输入输出流：istream，ostream
cin：标准输入流，默认键盘输入数据。
    本质：istream的一个全局对象。
cout：标准输入流，
    本质：ostream的一个全局对象。
## 2.2：缓冲机制
<font color="#ff0000">cout带有缓冲区：1024</font>
GCC中标准输出流的默认缓冲区大小就是1024个。

行缓冲：换行刷新。如果行满了，也会输出。
全缓冲：满了刷新。
非缓冲：直接输出，没有缓冲区。

- **缓冲机制**
    缓冲机制分为三种类型：**全缓冲、行缓冲和不带缓冲**。
    全缓冲：在这种情况下，当填满缓冲区后才进行实际 I/O 操作。全缓冲的典型代表是对磁盘文件的读写。
    
    行缓冲：在这种情况下，当在输入和输出中遇到换行符时，执行真正的 I/O 操作。这时，我们输入的字符先存放在缓冲区，等按下回车键换行时才进行实际的 I/O 操作。典型代表是cin。
    
    不带缓冲：也就是不进行缓冲，有多少数据就刷新多少。标准错误输出 cerr是典型代表，这使得出错信息可以直接尽快地显示出来。
## 2.3：标准输出流
ostream类定义了全局输出流对象cout，在缓冲区刷新时将数据输出到终端。
以下几种情况会导致输出缓冲区内容被刷新：
1. 程序正常结束。
2. 缓冲区满了。
3. 使用了操作符显示刷新输出缓冲区，如endl。
   endl是定义在ostream中的，他的作用是<font color="#ff0000">刷新缓冲区+换行</font>。

关于操作符：
endl：用来完成换行，并且刷线缓冲区。
flush：用来直接刷新缓冲区。           cout.flush();

## 2.4：标准错误
ostream类中还定义了全局输出流对象cerr，标准错误流（没有缓冲区）。
```C++
#include <unistd.h>
void test1(){
	cerr << 1;                //这里会立刻输出。
	cout << 3;
	sleep(2);                                                           
}
```

# 3：文件输入输出流（重点）
C++ 对文件进行操作的流类型有三个:
- ifstream（文件输入流）
- ofstream（文件输出流）
- fstream （文件输入输出流）

**文件模式**
根据不同的情况，对文件的读写操作，可以采用不同的文件打开模式。文件模式在 GNU GCC7.4 源码实现中，是用一个叫做 openmode 的枚举类型定义的，它位于 ios_base 类中。文件模式一共有六种，它们分别是:
    **in** : 以读取模式打开文件（默认用于 `ifstream`）；如果文件不存在，打开失败
    **out** : 以写入模式打开文件（默认用于 `ofstream`），如果文件存在覆盖文件内容；如果文件不存在，则直接创建一个
    **app** : 追加，写入将始终发生在文件的末尾
    **ate** : 打开文件后，将文件指针定位到文件末尾。
    trunc : 截断，如果打开的文件存在，其内容将被丢弃，其大小被截断为零
    binary : 二进制，读取或写入文件的数据为二进制形式
![[文件模式.png]]
## 3.1：文件输入流
ifstream：文件输入流，从文件中读取数据。

读取数据步骤：
1. 创建ifstream对象并打开文件。
2. 检查文件时候成功打开。
3. 读取文件。
4. 关闭文件。

文件输入流的信息传递方式：
文件--------》文件输入流对象的缓冲区（在内存上）----------》程序中的数据结构

创建文件对象的方式后很多（有很多的构造函数），这里说三种：
```C++
void test(void)
{
//1：先创建后绑定
ifstream ifs;
ifs.open("test.txt");         //将流对象和文件绑定，默认为读模式

//2：创建时绑定，C风格。
ifstream ifs2("test.txt");

//3：创建是绑定，C++字符串风格。 
string filename="test.txt";
ifstream ifs3(filename);
}
```
### 读取数据
1：逐词读取
使用>>运算符，默认以换行，空格作为间隔符。
```C++
// 默认以换行符,空格作为间隔符
// 一次读取一个字符串
string word;
// 只要ifs是goodbit状态就会一直读取
while(ifs >> word)
{
    cout << word << endl;
}
// 规范操作, 使用完之后关闭流
ifs.close();
```
<font color="#ff0000">注意：>>运算符的返回值是文件输入流的对象。</font>
流对象可以作为循环的条件，会隐式类型转换为bool类型。
流状态为good时，转化为true。否则，转换为false。

2：逐行读取
方法一：使用ifstream类中的成员函数getline，这种方式兼容C写法。
```C++
#include <string.h>
ifstream ifs("test.cc");
//方法一，兼容C的写法，使用较少
char buff[100] = {0};
while(ifs.getline(buff,sizeof(buff)))
{
    cout << buff << endl;
    // 清空缓冲区
    memset(buff,0,sizeof(buff));
}
```
准备好一片空间存放一行的内容，但是有一个弊端就是我们并不知道一行的内容会有多少个字符，如果超过了设置的字符串长度将无法完成该行的读取，也将跳出循环。

方法二：使用string提供的getline方法，<font color="#ff0000">工作中常用</font>。
（默认换行符分割，可以自己指定）
这个getline是string的非成员函数，只是定义在了string中。
```C++
string line;
//getline()返回的是，流对象。
while(getline(ifs,line))//从ifs绑定的文件中取出一行，放到line中。
{
    cout<<line<<endl;
}
```
3：读取指定字节的内容
read函数+seekg函数+tellg函数：
read：从当前文件指针处读取指定字节数到缓冲区，读完后指针后移。istream的成员。
tellg：告诉你当前文件指针在文件中的什么位置。（从文件开头算起）
seekg：移动文件指针到指定位置。有两种方式：
        1：绝对位置。
        2：相对位置。需要传入偏移量和基准点。![[seekg方式二.png]]
```C++
void test0()
{
    string filename = "test.cc";
    ifstream ifs(filename); 

    if(!ifs)
    {
        cerr << "ifs open file fail!";
        return;
    }
    
    //读取一个文件的所有内容先要获取文件的大小
    //将游标放到了文件的最后（尾后）
    fs.seekg(0,std::ios::end);
    long length = ifs.tellg();//获取尾后下标，实际就是总的字符数
    cout << length << endl;

    char * pdata = new char[length + 1]();
    //需要将游标再放置到文件开头
    ifs.seekg(0,std::ios::beg);
    ifs.read(pdata,length);

    // 还可以使用string构造函数 根据内容创建string对象
    //content包含了文件的所有内容，包括空格、换行
    string content(pdata);
    cout << "content:" << content << endl;
    /* cout << pdata << endl; */
    
    // delete
    delete[] pdata;
    pdata = nullptr;
    // close
    ifs.close();
}
```
如果文件读到最后了，这时候ifs的eof和fail标志位都会值1。此时需要使用ifs.seekg(0)和ifs.clean()，才能正常使用。

## 3.2：文件输出流
**基本步骤**：
1. 创建 `ofstream` 对象并打开文件。
2. 检查文件是否成功打开。
3. 写入数据。
4. 关闭文件。

数据传输的过程：
string对象-----------------》ofs对象-------------》ofs对象绑定的文件

另外，跟istream里面tellg（），seekg（）相对应的是tellp（），seekp（），存在于ostream。
对于 `fstream`（可读可写），这两个指针是**独立存在**的：读的位置和写的位置互不影响。

tellg（）和seekg（）操作的是读指针。
tellp（）和seekp（）操作的是写指针。

名字里的 `p` 和 `g`
- **`g`** = **get** pointer（读指针），用于输入流，对应 `tellg` / `seekg`。
- **`p`** = **put** pointer（写指针），用于输出流，对应 `tellp` / `seekp`。

### 写入数据
1：通过输入流运算符写内容
```C++
string filename = "test3.cc";
ofstream ofs3(filename);

string line("hello,world!\n");
ofs << line; 

ofs.close();
```
但是我们会发现进行多次写入，并没有保留下多次的内容，因为这种创建方式会使打开模式默认为`std::ios::out`，**每次都会清空文件的内容**。
如果目标文件不存在，则自动创建这个文件。

<font color="#ff0000">为了实现在文件流结尾追加写入内容的效果，可以在创建流对象时指定打开模式为std::ios::app（追加模式）</font>
```C++
string filename = "test3.cc";
ofstream ofs3(filename，std::ios::app);
```

2：通过write函数写内容
```C++
sting msg{"hello"};
ofs.writr(msg.c_type,msg.size());

char buff[100] = "hello,world!";
ofs.write(buff,strlen(buff));
```
使用write（）函数，ofs的模式使用默认的就可以，因为会自动追加。
**动态查看指令**
为了更方便地查看多次写入的效果（动态查看文件的内容）可以使用指令
```C++
tail 文件名 -F   //动态查看文件内容  
ctrl + c        //退出查看
```
# 4：字符串输入输出流
## 4.1：字符串输入流





## 4.2：字符串输出流


















\>>和<<是左结合，=是右结合。
**`cin >> num1 >> num2` 先执行左边，是因为 `>>` 是左结合，分组为 `(cin >> num1) >> num2`；`a = b = c` 先执行右边，是因为 `=` 是右结合，分组为 `a = (b = c)`。** 这是语言规定的运算符结合性。

关键字：explicit
这个关键字用来修饰构造函数或者转运函数。<font color="#ff0000">作用是禁止编译器进行隐式类型转换。</font>
一般用在：
1. 单参数构造函数。
2. 多参数构造函数。
3. 转换运算符。
`explicit` 是“禁止隐式转换”的关键字。它让构造函数或转换运算符只能被显式调用，不能悄悄地把其他类型自动转成你的类。
看一个不加explicit的例子：
```C++
class MyString 
{
public:
    MyString(int size) // 构造函数，接受一个 int
    {   
        // 分配 size 个字符的空间
    }
};

void print(MyString s) { }

int main() 
{
    MyString s1 = 10;      // ✅ 隐式转换：int → MyString
    print(10);             // ✅ 隐式转换：int → MyString
    
    //但是，如果定义MyString s2 = 'a'。
    //'a'会隐式转换为97，然后分配了97个字符空间。
    //这显然是错误的。
}
```