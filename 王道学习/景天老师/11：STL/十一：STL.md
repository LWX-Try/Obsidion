## 11.1：STL六大组件
---
1、容器：就是用来存储数据的，也就是数据结构。**（重要）**
- 序列式容器 vector、list、deque等
- 关联式容器 set、map等
- 无序关联式容器 unordered_set、unordered_map等

2、迭代器：就是为了访问容器中的元素，可以将其看成是一种指针，称为泛型指针。
- vector\<int> nums = {1,2,3};
- vector\<int>::iterator it = nums.begin();

3、算法：就是一些普通函数，可以操作容器中的元素。
- std::sort(vec.begin(),vec.end());

4、适配器：起到适配的作用。
- 容器适配器 stack、queue、priority_queue
- 迭代器的适配器
- 函数适配器 bind、bind1st、bind2nd、function等

5、函数对象：做定制化的操作

6、空间配置器：管理内存的

数据结构 + 算法 = 程序
![[STL六个组件之间的关系.png]]
**六者之间的关系**
1、容器作为STL的主体，是许多不同的数据结构
2、分配器为容器的实现分配应有的空间
3、泛型算法用来处理容器中的数据
4、迭代器是泛型算法和容器之间的桥梁
5、仿函数使得算法可以有更加灵活的自定义模式
6、适配器保证了自定义的功能可以和STL中现有的功能相融合

## 11.2：容器
---
### 11.2.1：容器分类
容器分为<font color="#ff0000">序列式容器，关联式容器，无序关联式容器。</font>
1. **序列式容器**: 底层采用线性表
    - array
    - **vector**
    - **deque**
    - forward_list
    - **list**
2. **关联式容器** : 底层采用红黑树
    - set
    - multiset
    - map
    - multimap
3. **无序关联式容器**: 底层采用哈希表
    - unordered_set
    - unordered_map
    - unordered_multiset
    - unordered_multimap
### 11.2.2：序列式容器
序列式容器包括：
<font color="#ff0000">静态数组array、动态数组vector、双端队列deque、单链表forward_list、双链表list。</font>

这五个容器中，我们<font color="#ff0000">重点了解三个vector、deque、list的使用。</font>
包括：初始化、遍历以及一些基本操作。另外，list还有一些特别的操作，也需要进行了解。

#### vector：
![[vector容器逻辑结构.png]]
vector头部是固定的，不能进行插入与删除，只提供了在尾部进行插入与删除的操作，所以如果真的要在头部插入或者删除，那么其他的元素会发生移动，这样操作就比较复杂。
**底层**：动态数组（连续内存）  
**复杂度**：随机访问 O(1)，尾部插入均摊 O(1)，中间插入/删除 O(n)

#### list：
![[list逻辑结构.png]]
list的实现是双向链表。
**底层**：双向链表  
**复杂度**：已知位置插入/删除 O(1)，查找 O(n)，<font color="#ff0000">不支持随机访问。</font>

#### deque：
![[deque的逻辑结构.png]]
从应用层面上，deque所实现的效果是双端队列，自然可以支持头部插入删除与尾部插入删除。
**底层**：缓冲区片段+ 中控器（map） 
**复杂度**：随机访问 O(1)，头尾插入/删除 O(1)，中间插入/删除 O(n)。

#### deque的底层实现（重要）：
![[deque的底层原理.png]]
deque是由多个片段组成的，<font color="#ff0000">片段内部是连续的，但是片段之间不连续的</font>，分散的，<font color="#ff0000">多个片段被一个称为中控器的结构控制</font>（也称为map 指针数组，但跟std::map不是一回事）

<font color="#ff0000">deque对象的底层由两部分组成，中控器，多个缓存区片段。</font>
中控器本质上是一个指针数组，一个deque对象，默认中控器元素个数为8个元素。
缓存区片段，再g++中，默认为512个字节，但是有些编译器使用的存储的对象个数为一个缓存区片段，如MSVC使用16个对象为一个缓存区片段。
如果一个对象的大小大于512，那么对象独占一个缓存区片段。

所以说deque是**在物理上是不连续的，但是逻辑上是连续的**。

通常情况下<font color="#ff0000">，在初始化时第一个元素会被放置在第一个缓冲区中相对靠前的位置（而非第一个缓冲区的首地址，这样方便在头部添加元素）。</font>

通常情况下，<font color="#ff0000">第一个缓冲区片段的首地址被放在中控器map的居中位置。</font>

**如果在尾部添加元素value**
找到最后一个存放着元素的缓冲区片段，在其中最后一个元素的后面添加元素value。
如果此时最后的缓冲区片段是满的，那么就再申请一个新的缓冲区片段，将要添加的元素value存放在新片段的首个位置。同时中控器也加上一个元素（新片段的首地址），让中控器与新片段联系起来。

<font color="#ff0000">如果中控器满了，就会像vector一样，对中控器进行扩容。扩容为原来的2倍，将指针数据复制到新的中控器中，销毁旧的中控器。</font>

**如果是在deque的头部添加元素value**
找到已经存储的第一个元素的位置，在它的前面存放新元素value。
经过若干次头部添加元素之后，如果第一个缓冲区片段满了，还要继续在头部添加元素，则新开辟一个缓冲区片段（作为第一个片段），将value存在这个片段中的最后一个位置。同时在中控器的相应位置存放新的缓冲片段的首地址。

**那么这里会有一个小小的问题：**
如果一直往deque头部添加元素，中控器的前半部分已经满了，后半部分还没满，会怎么样呢？

这种情况会通过一个reallocate_map函数来管理中控器的设置，先做一个判断，我们可以简单理解为如果中控器后半部分的空位较多，那么就将中控器中记录的地址全都后移若干位，这样中控器的前半部分就有空位了。如果中控器后半部分的空位较少，就再开辟新的更大的空间作为中控器，将原本中控器中记录的地址复制到新的中控器中。

同理，如果一直往deque尾部添加元素，中控器的后半部分已经满了，前半部分还没满，也是一样的处理流程。

注意：
<font color="#ff0000">在deque头部添加元素或者尾部添加元素的过程中，原本已经存在deque中的元素始终没有改变位置，这一点与vector形成很大的区别。</font>


如果元素类型大小大于等于 512 字节，函数返回 1。这表示每个缓冲区片段只存放一个元素。 比如自定义类型A的对象作为deque的元素，单个元素的大小超过了 512 字节，那么 __deque_buf_size(sizeof(A)) 的结果就是 1，每个缓冲区片段只会存放一个 A类型的元素。

<font color="#ff0000">deque的迭代器底层和vector迭代器的底层不一样。</font>
deque的迭代器比vector的迭代器复杂得多，`vector<T>`的迭代器实际是对`T*`的包装，但<font color="#ff0000">deque的迭代器实际应该视为对象，数据成员包含四个指针</font>，并且进行了一系列的运算符重载，使得迭代器能够像指针一样被使用。
deque的首迭代器。
![[deque的迭代器.png]]
deque对象的尾迭代器。![[deque的尾迭代器.png]]
#### 11.2.3：容器的基本操作
![[顺序容器.png]]
##### 初始化
```C++
//1、创建空对象(空容器)
vector<int>number1;
//2、count个value值
vector<int>number2(10,1);
//3、迭代器范围
int arr[10]={1,3,5,7,9,8,6,4,2,18};
vector<int>number3(arr,arr+10); //[,),左闭右开
/4、拷贝或者移动
vector<int>number4 { number3 };
//5、初始化列表方式
vector<int>number5={1,3,5,7,9,2,4,6,8,10};
```
<font color="#ff0000">总结：三种序列式容器vector、deque、list都具备五种初始化的方式，包括：无参、count个value、迭代器范围、拷贝或者移动、大括号范围</font>

##### 遍历（三种for循环）
```C++
// 以vector为例
vector<int> box1 {1,2,3,4,5};

// fori 下标循环
for(int i = 0; i < box1.size(); ++i)
{
    cout << box1[i] << " ";
}    
cout << endl;

// 迭代器遍历
for(auto it = box1.begin(); it != box1.end(); ++it)
{
    cout << *it << " ";
}
cout << endl;

// foreach遍历
for(auto & e : box1)
{
    cout << e << " ";
}
cout << endl;
```
总结：
三种序列式容器vector、deque、list都<font color="#ff0000">支持使用迭代器</font>以及<font color="#ff0000">增强for循环方式</font>进行遍历。

但是因为<font color="#ff0000">list不支持下标</font>，所以list不能使用下标进行遍历；<font color="#ff0000">vector与deque支持下标</font>，可以使用下标进行遍历。

##### 尾部插入和删除（push_back（），pop_back（））
```C++
vector<int> box;
// 尾部插入数据
box.push_back(1);
box.push_back(2);
// 尾部删除数据
box.pop();
```
<font color="#ff0000">总结：三种序列式容器vector、deque、list都支持在尾部进行插入与删除</font>

##### 头部插入和删除（push_front（），pop_front（））
```C++
list<int> box;
// 头部插入元素
box.push_front(1);
box.push_front(2);
// 头部删除元素
box.pop_front();
// vector不支持头部操作
```
总结：<font color="#ff0000">deque与list支持在头部进行插入与删除，但是vector不支持。</font>

##### insert（）函数（重要）
之前的push_back和push_front尽管可以插入元素，但是插入的位置都比较固定。

实际上三种<font color="#ff0000">序列式容器都允许在任意位置插入元素</font>，使用insert函数即可，它们都具备以下四种插入的功能。
![[insert定义.png]]
举例如下：
![[插入方法的实例.png]]

<font color="#ff0000">但是注意迭代器失效的问题！！！</font>
list迭代器失效：
![[list迭代器失效.png]]
deque迭代器失效：
![[deque迭代器失效.png]]

<font color="#ff0000">虽然insert函数有多种形式，但它们的返回值都是指向第一个被插入元素的迭代器</font>
要及时的更新迭代器：
![[迭代器更新.png]]

对于vector，存在扩容机制，不更新迭代器，可能会导致更大的隐患。
当vector使用insert进行插入时，可能会发生动态扩容。比如上述的nums是一个“存满”状态的vector，此时size和capacity的结果是一样的。增加新的元素会进行扩容。

扩容的过程中会申请一块新的空间，然后将老的空间上的元素拷贝到新的空间来，然后会清理老的空间、这时如果还继续使用原本的迭代器，对应的也就是老的空间的地址，会出现问题，因为此时迭代器已经失效了。

<font color="#ff0000">解决方案：每次在使用迭代器进行insert操作时，更新迭代器（重新置位）</font>

##### erase（）函数
三种序列式容器都可以进行erase操作，用来删除容器中的单个元素或多个元素
![[erase的定义.png]]
注意：
<font color="#ff0000">erase的返回值是一个迭代器，在使用的时候，也会造成迭代器实现问题。要及时更新。</font>


list的erase操作

由于list的原理，list中的元素无论是插入还是删除都非常的方便。

erase后迭代器it仍然指向原本的地址（**实际已经不属于list元素的空间，也可以归为一种迭代器失效问题**），想要继续正常使用it，需要进行迭代器更新。

<font color="#ff0000">erase函数的返回值为被删除元素后一位元素对应的迭代器。</font>

##### 元素的清空（clear（））
![[clear.png]]
总结：
<font color="#ff0000">三种序列式容器vector、deque、list都有clear清空元素、以及获取元素个数的函数size；</font>
<font color="#ff0000">对于vector与deque，还有回收多余空间的函数shrink_to_fit；</font>
<font color="#ff0000">对于vector还有记录容量大小的函数capacity</font>

##### 其他成员函数
```C++
//三种序列式容器都具备交换的函数
void swap( list& other );//此处的list可以改为vector或者deque
//vector与vector交换，deque与deque交换，list与list交换

//三种序列式容器都具备改变元素个数的函数
void resize( size_type count, T value = T() );
void resize( size_type count );
void resize( size_type count, const value_type& value )
    
//获取容器的第一个元素与最后一个元素(三种都有)
reference front();
const_reference front() const;
    
reference back();
const_reference back() const; 
```

##### emplace_back（）函数
这三种序列式容器还有两个值得关注的函数：emplace和emplace_back

emplace与insert对比，前者是直接在指定位置构造对象，后者是将一个已存在的对象复制并插入到容器中指定位置。

emplace_back与push_back对比， 前者是直接在容器尾部构造对象，后者是将一个已存在的对象复制并插入到容器末尾。
![[emplace定义.png]]
emplace_back举例：
![[emplace_back举例.png]]

#### 11.2.4：list的特殊操作（重要）
<font color="#ff0000">list还具有一些vector与deque没有的函数。</font>
注意，这里说的特殊操作也是属于list的成员函数，在使用时仍然是成员函数的使用方式。
##### sort（）函数
list里面的sort函数有两个版本，无参数版本，有参数版本。
![[list的sort函数的两个版本.png]]
- 无参数版本主要是比较C++的内置数据类型。如果要使用自定义数据类型，需要自己重载小于运算符。operator<，告诉程序比较的规则。可以在类外定义，也可以在类内定义。默认是升序。

- 有参数版本主要是比较自定义的数据类型。需要提供一个  可调用实体  ，可以是类函数，函数指针，函数，只要是可以调用的，都是可调用实体。

第二个版本的sort函数有一个参数，参数为Compare类型的对象。关于Compare，第一次见到应该是在set类模板声明中。
![[set中的compare.png]]

**这里对std::less的使用与之前unique_ptr的默认删除器的使用是很相似的。**
![[compare的比较.png]]

通过cpp参考文档能够了解到std::less进行了函数调用运算符的重载，在operator()函数中接受两个参数，代表两个元素，在函数定义中对两个元素进行了比较。
![[operator（）的重载.png]]
如果想要按照降序排列，对应位置可以换成`std::greater<int>()` 
        ----> <font color="#ff0000">std::greater和std::less一样是C++提供的函数对象。</font>

<font color="#ff0000">当然，针对需要排序的各种元素类型，可能需要更灵活的Compare，那么我们可以像定制化删除器一样定制化Compare。</font>

自定义比较器, 可以针对内置类型或者自定义类型进行比较
```C++
class Student
{
public:
    Student(int id, string name, int age)
    :m_id(id)
    ,m_name(name)
    ,m_age(age)
    {}
    int m_id;
    string m_name;  
    int m_age;
};

// 自定义比较器
struct StudentCompare
{
    // operator()  为了sort()排序需要
    bool operator()(const Student & lhs, const Student & rhs)
    {
        // 制定比较规则
        // 按照年龄升序排序
        return lhs.m_age < rhs.m_age;
    }
};

void test()
{
    Student s1(2,"张三",19);
    Student s2(1,"李四",21);
    Student s3(3,"王五",20);
    list<Student> box
    {
        s1,
        s2,
        s3,
    };
    cout << "-----sort-----" << endl;
    // list中的学生对象按照年龄升序排序
    box.sort(StudentCompare()); // 里面是函数对象

}
```
——<font color="#ff0000"> 那么vector和deque没有这样的成员函数，该怎样对容器内元素排序呢？</font>
使用算法库中的算法 `std::sort(box.begin(),box.end());`

##### reverse（）函数

将list的所有元素顺序反转
![[reverse的定义.png]]
注意和vector的reserve函数区分，拼写上非常相似。
![[reverse举例.png]]

##### unique（）函数

去重list中重复的元素
**需要先排序**（无论是从小到大还是从大到小），才能够实现效果
![[unique去重.png]]

##### merge（）函数
将一个list合并到当前list中
注意：如果合并前的list是无序的，合并后的list整体也是无序的
<font color="#ff0000">想要合并后的list结果是有序的，需要合并前两个list中的元素先排好序。</font>
如果使用如下版本的merge，那么必须先进行升序排列。
![[merge使用.png]]

如果相应更灵活的排序方式，需要使用如下版本的merge。
![[merge的灵活使用.png]]


lst2中的元素已经合并到lst1中了，<font color="#ff0000">lst2的元素被清空了。</font>

##### remove/remove_if（）函数
remove函数能够移除所有等于 value 的元素；

remove_if函数能够移除所有符合条件的元素，需要传入一个一元谓词（返回值为bool的一元函数）—— 如果某元素作为函数参数能够返回true，则判定为应该删除的元素。
![[remove-remove-if.png]]

##### splice（）函数
![[splice.png]]
示例如下：
![[splice示例.png]]



查找的函数。
count/find函数，再std里面。



## 11.3：迭代器
---
迭代器是一个抽象的概念，不是一个具体的东西。
比如，容器里面的迭代器，是自己的实现的。
这里学的迭代器和容器里面的迭代器不一样，实现不一样，但是都叫迭代器。

是一个抽象的概念。
迭代器算是C++一个非常重要的抽象概念，<font color="#ff0000">它的意义在于：</font>
- <font color="#ff0000">为访问容器中的元素提供了统一的方式。</font>比如vector/deque/list三种容器存储元素的方式完全不同，存储元素的类型也可能不同，但是都可以用迭代器来遍历和操作容器中的元素，而不需要关心容器的具体实现细节。
- <font color="#ff0000">使算法能够独立于容器进行设计和实现。</font>例如，std::sort算法可以对任何支持随机访问迭代器的容器进行排序，而不需要为每种容器单独实现排序算法。这样可以提高代码的可复用性和灵活性。

### 11.3.1：迭代器类型
迭代器分类:
1.输入迭代器 input iterator :  读取数据   num =  *it
2.输出迭代器 output iterator: 写入数据   *it =  num
3.前向迭代器 forward iterator : it只能从左往右移动 ++it
4.双向迭代器 bidirectional iterator:  it可以从左往右移动,也可以从右往左移动 ++it , --it
5.随机访问迭代器 random access iterator : ++it,--it, it += n , it -= n

五种迭代器之间的关系（逻辑层面上近似于继承关系）
![[五种迭代器之间的关系.png]]
越向下，迭代器的功能越强大。




各容器迭代器的类型：
![[各容器迭代器类型.png]]
总的来说：
![[容器迭代器类型总结.png]]

总结：迭代器就是访问容器中元素的手段，<font color="#ff0000">大多数时候并不需要研究迭代器的具体实现，能够根据容器的迭代器类型进行正确的使用即可</font>。

### 11.3.2：流迭代器
#### 输入流迭代器
流迭代器是特殊的迭代器，其作用有：
（1）可以将输入/输出流作为**容器**看待(因为输入输出流都有**缓冲区**的概念，也可以视为存一些数据)，流迭代器可以用来访问输入输出流这种“容器”中的数据。
（2）STL 中有许多算法都是基于迭代器设计的，输入流迭代器和输出流迭代器使得这些算法能够直接应用于输入输出流。因此任何接受迭代器参数的算法都可以和流一起工作。

![[输出流迭代器的构造.png]]

尝试使用一下输出流迭代器，如下（使用了algorithm提供的copy函数）

最常用的一种使用方式：使用 `std::copy` 算法通过输入流迭代器从文件中读取数据，然后通过输出流迭代器将数据输出到控制台，就像操作普通容器一样。
![[copy函数.png]]
![[copy函数实现.png]]
![[copy举例.png]]

输出流迭代器的类模板形式：
```C++
template <class _Tp,
          class _CharT = char, class _Traits = char_traits<_CharT> >
class ostream_iterator 
{
    //根据上述例子中的构造方式
    //__s = cout;
    // __c = " ";
    //_M_stream = &cout;
    //_M_string = "  "
    ostream_iterator(ostream_type& __s, const _CharT* __c) 
    : _M_stream(&__s)
    , _M_string(__c)  
    {}
    
    ostream_iterator<_Tp>& operator*() { return *this; }
    ostream_iterator<_Tp>& operator++() { return *this; } 
    ostream_iterator<_Tp>& operator++(int) { return *this; }
    
    ostream_iterator<_Tp>& operator=(const _Tp& __value) 
    { 
        *_M_stream << __value;//cout << 1
        if (_M_string) *_M_stream << _M_string;//cout << "  "
        return *this;
    }
    
private:
  ostream_type* _M_stream;
  const _CharT* _M_string
};
```

copy参考
```C++
//copy(vec.begin(), vec.end(), osi);
//first = vec.begin();
//last = vec.end()
//d_first = osi 
OutputIt copy(InputIt first, InputIt last, OutputIt d_first)
{
    while (first != last) {
        //需要查看ostream_iterator的后置++与解引用（相应的运算符重载函数） 
        *d_first++ = *first++;//++优先级高于解引用
    }
    return d_first;
}

//******分析过程**********
osi = 1 //ostream_iterator调用operator=函数
        last
1 5 9 7
f ------> f
```

使用例子说明：
```C++
void test4()
{
    vector<int> box{1,2,3,4,5};
    // 把vector中元素依次打印出来
    // 创建一个输出流迭代器对象
    ostream_iterator<int> oi{ cout, " " };
    std::copy(box.begin(), box.end(), oi);
    cout << endl;
    cout << "-------" << endl;
    std::copy(box.begin(), box.end(), ostream_iterator<int>{ cout, " " });
    cout << endl;
}
```

#### 输出流迭代器
输入流迭代器构造函数：
![[输入流迭代器构造函数.png]]

正确使用：
![[back_inserter.png]]
如果只去直接在copy中使用迭代器的话，会造成错误，vector不会扩容，可能会空间不足，造成错误。


```C++
template <class _Tp, 
          class _CharT = char, class _Traits = char_traits<_CharT>,
          class _Dist = ptrdiff_t> 
class istream_iterator 
{
    //无参构造
    istream_iterator() 
    : _M_stream(nullptr)
    , _M_ok(false) {}  
    
    //接收一个输入流对象的构造函数
    //__s = cin;
    //_M_stream = &cin;
    istream_iterator(istream_type& __s) 
    : _M_stream(&__s) 
    {
        _M_read(); 
    }
    
    void _M_read() 
    {
        _M_ok = (_M_stream && *_M_stream) ? true : false;
        if (_M_ok) 
        {
            *_M_stream >> _M_value;//cin >> _M_value;
            //输入流的状态为goodbit，则_M_ok = true
            _M_ok = *_M_stream ? true : false;
        }
    }
    
    bool _M_equal(const istream_iterator& __y) const
    { 
        return (_M_ok == __y._M_ok) && (!_M_ok || _M_stream == __y._M_stream); 
    }
    
    istream_iterator operator++(int)  
    {
        istream_iterator __tmp = *this;
        _M_read();
        return __tmp;
    }
    
    reference operator*() const { return _M_value; }
private:
  istream_type* _M_stream;
  _Tp _M_value;
  bool _M_ok;
};


//isi._M_equal(__y);
inline bool operator!=(const istream_iterator& __x, const istream_iterator& __y) 
{
  return !__x._M_equal(__y);
}


//例子中使用copy函数的语句
//std::copy(isi, istream_iterator<int>(), std::back_inserter(vec));


// first = isi;
//last = istream_iterator<int>()
//d_first = std::back_inserter(vec)
OutputIt copy(InputIt first, InputIt last, OutputIt d_first)
{
    while (first != last) 
    { //两个输入流迭代器对象进行!=比较，其必然对!=符号进行了重载
        //*d_first++ = 上一次输入的值
        //d_first是back_inserter函数返回值，跳转查看back_inserter函数
        *d_first++ = *first++;   //first每次进行++操作，会进行一次输入，如果输入过程使得流对象状态进入非goodbit状态，就可以影响到_M_ok
    }
    return d_first;
}


template <class _Container>
inline back_insert_iterator<_Container> back_inserter(_Container& __x) {
  return back_insert_iterator<_Container>(__x);
}
```

### 11.3.3：迭代器适配器
#### 三组需要掌握的适配器
<font color="#ff0000">以下这三组迭代器适配器需要掌握：</font>
![[三组迭代器适配器需要掌握.png]]
- back_inserter是函数，
  该函数的返回类型是back_insert_iterator，它们的底层会调用容器的push_back。

- front_inserter是函数，
  该函数的返回类型是front_insert_iterator，它们的底层会调用容器的push_front。

- inserter是函数，
  该函数的返回类型是insert_iterator，它们的底层会调用容器的insert。

要将list的元素插入到vector尾部，除了以往的常规方式（遍历list的元素，再使用vector的push_back函数添加这些元素），还可以使用上迭代器。
![[使用实例.png]]

再将vector的元素插入到list头部
![[实例.png]]

<font color="#ff0000">注意：使用上述三种插入迭代器的时候，如果容器本身不具备对应插入函数的话，那么就不能乱用。</font>
比如，在vector中不能使用头插，因为vector本身就不能使用front（）函数。因为这里的vector底层是使用的push，没有front_push。所以不能使用。

但是对于vector说，可以使用insert进行任意位置的插入。

#### 反向迭代器适配器
![[反向迭代适配器.png]]
注意遍历时偏移的方式，仍然是自增
![[反向迭代适配器的实例.png]]

## 11.4：算法
---
STL的算法库中的函数，都是属于普通函数（并不是哪一个类的成员函数）。

STL 算法是 C++ 标准库中 `<algorithm>`（以及 `<numeric>`）头文件提供的一大组<font color="#ff0000">通用函数</font>，用来对容器（或任意区间）中的元素进行常见操作：查找、排序、计数、修改、合并、堆操作等。

它们的本质是：**基于迭代器抽象出来的、与容器无关的通用操作模板。**
<font color="#ff0000">本质就是一些函数，要学会使用这些函数。</font>

for_each()，lambda表达式，find（），count（），remove()/remove_if()，
copy（），replace（），sort（），max（），min（），lower_bound（），
upper_bound()，equal_range()。要掌握。


### 11.4.1：for_each函数使用
for_each函数：对迭代器范围内的元素，进行函数 f 操作。
![[for_each的参考文档描述.png]]
unary：一元的意思。

第三个参数：
<font color="#ff0000">需要传一个一元函数（或者仿函数），其函数形参只有一个（不一定要有const或者&）</font>

一元函数：函数的参数只有一个；

二元函数：函数的参数只有两个。

简单应用，遍历打印：
![[使用for_each遍历打印.png]]
想要通过for_each改变vector中元素的内容，就得在func函数上做文章，可以给func函数形参改成引用。

<font color="#ff0000">实际使用时，f函数是否改变容器元素，for_each是没有限制的</font>

for_each的源码：
```C++
template<class InputIt, class UnaryFunction>
constexpr UnaryFunction for_each(InputIt first, InputIt last, UnaryFunction f)
{
    for (; first != last; ++first) 
    {
        f(*first);
    }
    return f; 
}
```

扩展:  
<font color="#ff0000">一元函数UnaryFunction 不一定是个函数</font>， 只要可以像函数一样被调用的结构都可以
可调用实体:
1. 函数
2. 函数指针
3. 函数对象
4. lambda表达式

### 11.4.2：lambda表达式（重要）
#### lambda表达式语法和基本用法

lambda表达式: 
<font color="#ff0000">本质就是一个匿名的函数对象, 可以像函数一样被调用(局部函数)</font>

语法:
```C++
[capture](params) opt -> retureType
{
    body;
}

[capture] : 捕获列表, 用于捕获同一作用域内的局部变量
(params) : 参数列表
opt : 函数选项, 一般不用
-> retureType : 返回值类型
{
    函数体, lambda表达式要做的事情
}
```

lambda表达式举例：
![[lambda表达式举例.png]]

如果没有在`[]`中捕获a、b，那么在匿名函数的函数体中无法访问它们

进一步的，还可以给匿名函数增加函数参数，以及指定函数返回类型
![[lambda表达式的形参.png]]
<font color="#ff0000">注意：捕获列表捕获的是局部变量，全局变量不需要捕获。</font>
![[全局变量不需要捕获.png]]

#### lambda表达式的捕获列表
下面是lambda表达式的一些规则：
![[lambda表达式的捕获列表.png]]
<font color="#ff0000">使用&的是引用外部的变量。单独引用时，要在变量前面加&。可以在内部改变该值。</font>
<font color="#ff0000">使用=是复制外部的变量。单独复制时，不需要在前面加=。不可以在内部改变该值。</font>

<font color="#ff0000">函数选项的位置加上mutable关键字才能对局部变量a/b进行修改操作，但这样做也并不能修改a和b实际的值</font>
![[值捕获.png]]
看到上述的结果，应该能联想到常规函数的值传递场景，想要真正通过形参改变实参，形参需要写成引用形式。

那么对于匿名函数而言，如果想要在匿名函数中修改捕获的变量其本身的值，那么应该采用的是**引用捕获**
![[引用捕获.png]]
<font color="#ff0000">另外，对于全局变量和函数参数，匿名函数中可以直接完成修改。</font>

#### lambda表达式可以被接受

匿名函数也可以被接受，被复用。
![[lambda表达式的复用.png]]

总结：
对于lambda表达式，大家可能会在实际代码中见到非常多种形式，需要读懂它的意思

（1）lambda表达式的参数列表与普通函数的参数列表一样，<font color="#ff0000">如果没有参数，则参数列表可以省略不写；</font>

（2）如果没有需要捕获的内容，需要写上`[]`；

（3）选项如非必需，可以省略
mutable: 可以修改按值传递进来的拷贝（注意是能修改拷贝，而不是值本身）
noexcept: `noexcept` 是一种契约，向编译器和调用者保证函数不会抛出异常。如果违反了这个契约，程序的行为将是未定义的，通常会导致程序终止。

（4）<font color="#ff0000">返回类型可以省略。</font>通过返回值后置语法来定义的。一般情况下，不指定lambda表达式的返回值，编译器会根据return语句自动推导返回值的类型，但需要注意的是labmda表达式不能通过列表初始化自动推导出返回值类型。


#### lambda表达式的类型

<font color="#ff0000">lambda 就是编译器在编译期生成的一个匿名仿函数类，捕获的变量是它的成员，函数体是它的 `operator()`，lambda 表达式本身是这个类的对象。</font>

至此仍然存在一个问题auto推导出来的类型究竟是什么呢？

lambda表达式的类型与匿名函数的returnType并非是同一件事。

<font color="#ff0000">lambda表达式的类型在C++11中会被看做是一个带operator()的类，即仿函数。</font>按照C++标准，<font color="#ff0000">lambda表达式的operator()默认是const的</font>，一个const成员函数是无法修改成员变量值的。<font color="#ff0000">mutable选项的作用就在于取消operator()的const属性。</font>

这里附上一个网站：[https://cppinsights.io/](https://cppinsights.io/)

可以通过它查看编译器是如何处理代码的。

因为lambda表达式在C++中会被看做是一个仿函数，因此可以使用std::function和std::bind来存储和操作lambda表达式（算法适配器的使用）

### 11.4.3：remove_if函数
remove_if()的基本语法：
![[remove_if.png]]
第三个参数p是一个<font color="#ff0000">一元谓词</font>。
其实这里不仅可以填一元谓词，只要是可调用实体，都可以就使用。

<font color="#ff0000">一元函数：函数的参数只有一个。</font>
<font color="#ff0000">二元函数：函数的参数只有两个。</font>

<font color="#ff0000">一元断言/谓词：函数的参数只有一个，并且返回类型是bool。</font>
<font color="#ff0000">二元断言/谓词：函数的参数只有两个，并且返回类型是bool。</font>

注意：remove_if（）函数，不是删除元素的，而是对元素进行重新排列的。符合条件放在前面，不符合条件的放在后面。返回第一个不符合条件的迭代器。

<font color="#ff0000">remove_if函数的返回值就是待删除的元素的首迭代器。</font>
![[remove_if和erase配合.png]]

<font color="#ff0000">remove_if实际上最终的效果是同时做了两件事，更改了容器中特定位置的元素，并且返回了待删除元素的首迭代器。</font>

### 11.4.4：算法适配器
#### bind（）函数（非常重要）
bind（）函数的作用相当于绑定参数到形参列表，从而改变函数形参数量。
<font color="#ff0000">可以改变函数形态，函数的类型， 函数的参数的顺序 为了适配STL中的一些算法。</font>
![[bind函数.png]]
![[bind函数的参数.png]]


基本使用：
![[bind的基本使用.png]]
第一个参数可以是函数对象、函数指针、函数引用(比较少见)、指向成员函数指针或指向数据成员指针。如上将函数名直接作为参数，相当于是传入函数指针。

<font color="#ff0000">注意：成员函数指针稍微复杂。</font>
因为成员函数指针的第一个参数是一个隐藏的this指针。但是这里要加上。
![[band绑定成员函数指针.png]]

#### 引用折叠
看回bind的声明形式，第一个参数看起来是一个右值引用，为什么在真正使用过程中，传入一些左值也可以呢？

这涉及到模板中的引用折叠

如下代码修改成模板后是可以正常运行的：
![[引用折叠.png]]
模板中a为T&&类型，那么会造成多个引用符号写到一起。我们自己在写代码时不能将两个以上的引用符号写到一起，<font color="#ff0000">但是模板推导时会遵循一套引用折叠的机制。</font>
```C++
//T &&有两种情况
& &&---->&
&& &&---->&&

//如果模板的函数参数为T&形式
//也有两种情况
& & ---->&
&& &----->&
```
根据上面的引用折叠规则可知，模板中写成T&&，实参既可以传左值，也可以传右值；如果模板中写成了T&，那么实参就只能传左值。

<font color="#ff0000">引用折叠规则:</font>
1. 只要有左值引用参与, 折叠为左值引用
2. 都是右值引用, 折叠为右值引用

#### 占位符
回到bind的基本使用，似乎每次都需要将被绑定的函数的所有参数都一起传入。
除此之外，还有一种占位符的写法。
![[占位符.png]]

注意辨析：占位符占的哪个位置，就需要输入那个位置的参数。
![[占位符辩解.png]]

但是，如果占位符改一下，就会发生错误。
![[占位符的错误使用.png]]
<font color="#ff0000">也就是说使用了_4这个占位符，那么在传入实参时至少要传入4个参数。</font>（f调用时可以传很多个参数，多余的参数被舍弃）

对于func函数，第一个参数是传入的10，第二个参数是占位符匹配到的20，第三个参数是占位符匹配到的400，第四个和第五个则是传入的number

<font color="#ff0000">可以理解为占位符整体代表的是形参的位置，占位符中的数字代表的是实参的位置。</font>

#### 引用包装器
![[引用包装器.png]]
<font color="#ff0000">说明了bind的默认传递方式是值传递</font>，<font color="#ff0000">如果在bind中想要使用引用传递，需要用到引用包装器。</font>
![[引用包装器举例.png]]
如上在bind传参时使用**引用包装器std::ref** (reference)

如果func函数中原本形参形式为const引用，相应地可以使用**常引用包装器std::cref** (const reference)
![[const引用.png]]
<font color="#ff0000">引用包装器也属于函数适配器。</font>

#### bind函数返回值
![[函数指针.png]]

不过，<font color="#ff0000">如果直接根据bind所绑定的函数的类型定义函数指针，是不能直接接收bind返回值的</font>

`std::bind` 的返回值是一个**未指定类型的可调用对象（callable object）**，通常称为“绑定对象”或“绑定器”。它把原函数和一部分参数绑定在一起，之后调用它时，相当于调用原函数并传入绑定的参数。

#### function
可以使用function接收bind的对象。
<font color="#ff0000">function本质是一个类。</font>
```C++
vector<int> box;  ----> 数据容器 存储数据用的
vector<string> box2;
vector<Student> box2;

function理解为函数容器 ----> 存储函数(可调用实体)用的
function类的对象, 可以像函数一样被调用
function<函数类型> f;
```


对于函数而言，函数的类型包含了两个信息：函数返回类型 + 函数参数列表（包含了函数参数的类型、个数、顺序）
![[错误接收band.png]]
很多时候，逻辑层面上很好理解，但是原生的语法却并不支持这种表达，STL则提供了一种方式可以支持这种表达。
![[function.png]]
使用如下：
![[使用举例.png]]
function实例：
![[function示例.png]]


<font color="#ff0000">绑定成员函数的第一个参数：</font>
回到funcion接收bind返回值的例子，当bind函数绑定类的成员函数时，需要传入对象的地址以对应this指针这个参数，实际上还有一种写法。
![[绑定成员函数的第一个参数.png]]
bind绑定成员函数的时候，有两种传递的方式，
**第一种：直接传递对象的地址，第二种，将对象拷贝进来。**

—— 如果使用的是第一种，那么会传递的是地址值，那么就只会有一个指针大小的开销；但是如果是传递对象，就会拷贝对象，如果对象占据的空间比较大，那么开销也会比较大。

那传递对象的写法有什么意义呢？
—— 如果使用的是第一种，传递的是地址值，如果对象本身已经销毁了，那么有可能会有野指针的情况。
比如对象ex的创建是A线程，bind的绑定是B线程，在bind绑定时传对象的地址值，但A线程已经销毁了，就会出现这样的问题。
但是如果使用第二种形式，传递的是对象，那么就不会出现野指针的问题。

<font color="#ff0000">所以在多线程环境下，更建议传对象，而不是传地址。</font>

<font color="#ff0000">function称为函数包装器，其存储的是函数类型，所以也可以称为函数容器。</font>

#### bind和function的结合（重要）
<font color="#ff0000">bind和function的结合，可以实现函数式多态。</font>达到和继承一样的效果。

我们了解了std::bind和std::function各自的基本使用，再来看看它们的结合使用，这是一种**非常非常非常**强大的用法，因为bind具备改变函数形态的功能，只要函数的返回类型相同，任何参数类型的函数都可以被bind绑定之后，变成函数类型完全一致的函数对象，然后都可以被function进行接收。

来看一个std::function与std::bind结合使用体现出多态性的例子，也就是注册回调函数与执行回调函数。

在学习多态时，我们学过一段计算图形面积的例子
```C++
#include <math.h>
#include <iostream>
#include <string>
using std::cout;
using std::endl;
using std::string;

class Figure{
public:
    virtual string getName() const = 0;
    virtual double getArea() const = 0;
};

void display(Figure & fig)
{
    cout << fig.getName();
    cout << "的面积：";
    cout << fig.getArea() << endl;
}

class Rectangle
: public Figure
{
public:
    Rectangle(double len,double wid)
    : _length(len)
    , _width(wid)
    {}

    string getName() const override
    {
        return "矩形";
    }

    double getArea() const override
    {
        return _length * _width;
    }
private:
    double _length;
    double _width;
};
//...

void test0()
{
    Rectangle rt(2,5);
    display(rt);
	//...
}

int main(void)
{
    test0();
    return 0;
}
```
![[bind和function的结合.png]]
![[bind和function.png]]
面向对象的写法：继承 + 纯虚函数(或者普通的虚函数)，具有一定的局限性：必须建立继承关系，派生类对基类的虚函数进行覆盖时必须要函数名和函数形式完全一致等；

**基于对象**的写法：std::function + std::bind，这一套写法摒弃了上述的局限性，同样体现出多态性且更灵活。

#### mem_fn成员函数适配器
<font color="#ff0000">mem_fn()函数，主要作用于成员函数。将对象作为参数调用。</font>
mem_fn : 对类中成员函数进行包装, 生成一个新的可调用对象, 可以像函数一样被调用

因为成员函数与算法库中的算法不能直接很好的适配，所以二者结合使用的时候，需要使用成员函数适配器进行适配。
![[mem_fn.png]]
![[mem_fn举例1.png]]
![[mem_fn举例2.png]]

for_each的第三个参数要求是一个一元函数，成员函数print看起来无参，实际上有隐含的this指针作为参数，它的确是一个一元函数。
还有其他方法的实现。
![[bind和for_each.png]]


或者我们也可以用function接一下bind绑定成员函数的结果。
![[function和for_each.png]]

## 11.5：适配器
---
### 容器适配器


### 迭代器适配器


### 算法适配器


## 11.6：函数对象
---
**函数对象也称为仿函数**，严格意义上说函数对象仅仅指重载了函数调用运算符的类的对象。
![[可调用实体.png]]

## 11.7：空间配置器
---
<font color="#ff0000">在STL中，所有的容器，都是使用的allocator空间分配器，allocator是一个类。</font>
在C++中所有STL容器的空间分配其实都是使用的std::allocator。
所有容器（`vector`、`list`、`map` 等）在需要动态内存时，都不直接调用 `new`/`delete` 或 `malloc`/`free`，而是通过空间配置器来完成。

std::allocator可以感知类型的空间分配器，将空间的申请与对象的构建、以及空间的回收与对象的销毁严格分离。

以前我们知道使用new创建单个对象的时候，空间的申请与对象的构造实际也是分开的（可以回顾new表达式的工作步骤）。

    那么为什么要将空间的申请与对象的构建分开呢？
    (1) 减少不必要的对象构造和析构：在容器的使用过程中，有时只是需要预留一些空间，而并不需要立即在这些空间上构造对象。例如，std::vector 的 reserve 函数，它的作用是为容器预留足够的内存空间，但不会构造任何对象。如果不将空间申请和对象构建分开，每次预留空间时都会构造对象，之后又可能因为不需要这些对象而进行析构，这会带来不必要的性能开销。 
    (2)方便实现复杂的容器操作：在实现容器的一些操作，如插入、删除元素时，将空间申请和对象构建分开可以更方便地处理各种情况。例如，在插入元素时，如果当前空间不足，容器可以先申请新的空间，然后将原有元素移动到新空间，最后在合适的位置构造新的对象，这样的操作更加灵活和高效。（比如vector的动态扩容机制背后就使用了空间配置器）

![[allocator把内存和对象分开执行.png]]
![[allocator防止频繁申请释放.png]]

### 11.7.1：函数使用
在类allocator中，有几个重要的函数：
![[allocator类中重要的函数.png]]
```C++
//申请空间
T* allocate( std::size_t n );

//释放空间
void deallocate( T* p, std::size_t n );

//构建对象
void construct( pointer p, const_reference val );

//执行析构
void destroy( pointer p );  //p->~T()
```

### 11.7.2：原理
<font color="#ff0000">空间配置器会分为两级：</font>
- 一级空间配置器使用类模板malloc_alloc_template ，其底层使用的是malloc/free进行空间的申请与释放；<font color="#ff0000">当申请的空间的大于128个字节的使用，allocate函数就会使用malloc/free申请，释放内存。</font>

- 二级空间配置器使用类模板default_alloc_template（默认空间配置器），其底层根据申请空间大小又分为两个分支，第一分支是当申请的空间大于128字节的时候，还是走malloc_alloc_template ，当申请的空间小于或等于128字节的时候，使用<font color="#ff0000">内存池 + 自由链表</font>的方法申请空间。

注意：这里提到的自由链表实际是一个容量为16的指针数组，在源码中使用了_S_free_list这个名字，直译为自由链表。

<font color="#ff0000">有一个容量为16的指针数组，每个元素（指针）指向一个链表 ，用于管理不同大小的内存块。</font>这 16 个指针所指向的链表，分别对应 8、16、24、32、40、48、56、64、72、80、88、96、104、112、120、128 字节的内存块。

每当程序申请小于 128 字节的内存时，二级空间配置器会先将申请的内存大小上调为 8 的倍数，再根据上调后的大小（8的整数倍）找到对应的链表。
![[二级空间配置器原理图.png]]




## 杂项
C++ STL 容器对象的大小是**编译期固定的**，与容器里存了多少元素无关。<font color="#ff0000">对象内部只包含一些管理信息（指针、计数器、迭代器/哨兵等），真正的元素数据在堆上动态分配。</font>

### array
- **大小**：`sizeof(array<T, N>) = N * sizeof(T)`，没有额外开销。
- 元素直接存储在对象内部，所以对象大小随 N 增长。

### vector
- **大小**：`sizeof(vector<T>) = 24` 字节。
- **内部成员**：3 个指针
    - `_M_start`：指向第一个元素。
    - `_M_finish`：指向最后一个元素的下一个位置。
    - `_M_end_of_storage`：指向已分配内存的末尾。

### deque
- **大小**：`sizeof(deque<T>) = 80` 字节
- **内部成员**：
    - `_M_map`：指向中控数组的指针（8 字节） 
    - `_M_map_size`：中控数组大小（8 字节）
    - `_M_start`：
          起始迭代器（32 字节，含 4 个指针：`cur`、`first`、`last`、`node`）
    - `_M_finish`：结束迭代器（32 字节）
    - 合计：8 + 8 + 32 + 32 = 80 字节

### list
- **大小**：`sizeof(list<T>) = 24` 字节
- **内部成员**：
    - 一个哨兵节点（`_List_node_base`），包含 `next` 和 `prev` 两个指针（16 字节）
    - 一个 `size_t` 记录元素个数（8 字节）
    - 合计：24 字节

### string
<font color="#ff0000">注意：string不属于STL的内容。</font>
- **大小**：`sizeof(string) = 32` 字节（libstdc++）
- **内部成员**：
    - 指针（8 字节）：指向堆上的字符数据，或指向内部 SSO 缓冲区
    - `size`（8 字节）：当前字符数
    - 联合体（16 字节）：短字符串时存 SSO 缓冲区，长字符串时存 `capacity`