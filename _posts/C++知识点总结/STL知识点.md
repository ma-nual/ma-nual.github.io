### string

string类的底层是一个字符串指针。

```c++
#include <iostream>
#include <string>

using namespace std;

class String{
public:
    String(const char *str = NULL); //普通构造函数
    String(const String &other); //拷贝构造函数
    ~String(); //析构函数
    String &operator=(const String &other); //拷贝赋值函数
    String operator+(const String &other); //字符串连接
    bool operator==(const String &other); //判断相等
    char operator[](int idx); //返回访问字符
    friend ostream &operator<<(ostream& os, const String& str); //输出字符串
    int getLength(); //返回长度
private:
    char *m_data; //私有变量保存字符串
};

String::String(const char *str){
    if(str == NULL){
        m_data = new char[1];
        *m_data = '\0';
    }
    else{
        int length = strlen(str);
        m_data = new char[length + 1];
        strcpy(m_data,str);
    }
}

String::~String(){
    if(m_data){
        delete[] m_data;
        m_data = 0;
    }
}

String::String(const String &other){ //输入参数为const型
    if(!other.m_data){ //对m_data加NULL判断
        m_data = 0;
    }
    m_data = new char[strlen(other.m_data) + 1];
    strcpy(m_data,other.m_data);
}

String &String::operator=(const String &other){ //输入参数为const型
    if(this != &other){ //检查是否自赋值
        delete[] m_data; //释放原有的内存资源
        if(!other.m_data){ //对m_data做NULL判断
            m_data = 0;
        }
        else{
            m_data = new char[strlen(other.m_data) + 1];
            strcpy(m_data,other.m_data);
        }
    }
    return *this; //返回本对象的引用
}

String String::operator+(const String &other){
    String newString;
    if(!other.m_data){
        newString = *this;
    }
    else if(!m_data){
        newString = other;
    }
    else{
        newString.m_data = new char[strlen(m_data) + strlen(other.m_data) + 1];
        strcpy(newString.m_data,m_data);
        strcat(newString.m_data,other.m_data);
    }
    return newString;
}

bool String::operator==(const String &other){
    if(strlen(m_data) != strlen(other.m_data)){
        return false;
    }
    else{
        return strcmp(m_data,other.m_data) ? false:true;
    }
}

char String::operator[](int idx){
    if(idx > 0 && idx < strlen(m_data)){
        return m_data[idx];
    }
}

ostream &operator<<(ostream& os, const String& str){
    os << str.m_data;
    return os;
}

int String::getLength(){
    return strlen(m_data);
}

int main(){
    String str1 = "yes";
    String str2 = "buy";
    str2 = str1;
    cout << str1[2] << endl;
    return 0;
}
```

### vector

vector是线性容器，有一块连续的存储空间，可以自动增长或者缩小存储空间。

**vector的优点**

- 可以使用下标访问个别的元素。
- 迭代器可以按照不同的方式遍历容器。
- 可以在容器的末尾增加或删除元素。

**容器的大小和容器的容量的区别**

大小是指**元素的个数**，容量是**分配的内存大小**，容器的容量一般不小于容器的大小。

- `vector::size()`返回容器的大小。
- `vector::capacity()`返回容量值，容量多于容器大小的部分用于以防容器大小的增加使用。
- `vector::resize(Container::size_type n)`用来强制把容器改为容纳n个元素。调用resize函数之后，size函数将会返回n。如果n小于当前大小，容器尾部的元素会被销毁。如果n大于当前大小，在元素加入之前会进行重新分配，新默认构造的元素会添加到容器尾部。
- 每次重新分配内存都会很影响程序的性能，所以一般分配的容量都大于容器的大小，若要自己指定分配的容量的大小，则可以使用`vector::reserve(Container::size_type n)`强制容器把它的容量改为不小于n，提供的n不小于当前所需的大小。因为容量需要增加，这一般会强迫进行一次重新分配。如果n小于当前容量，vector会忽略它，则这个调用什么都不做。

**vector的内存管理与效率**

- 使用reserve()函数提前设定容量大小。
  - 对于vector容器来说，如果有大量的数据需要进行push_back，应当使用reserve()函数提前设定其容量大小，否则会出现许多次容量扩充操作，导致效率低下。
- 使用“交换技巧”来修整vector过剩空间/内存。
  - `vector<int>(ivec).swap(ivec);`，表达式vector<int>(ivec)表示建立一个临时vector，它是ivec的一份拷贝。但是vector的拷贝构造函数只分配拷贝的元素需要的内存，所以这个临时vector没有多余的容量。然后临时vector和ivec交换数据完成，但ivec只有临时变量的修整过的容量，而这个临时变量则持有了曾经在ivec中的没用到的过剩容量。在这个语句结尾处，临时 vector被销毁，以释放以前ivec使用的内存，收缩到合适的大小。

**vector的扩容机制**

不同的编译器，vector有不同的扩容大小，**在vs下是1.5倍，在gcc下是2倍**，vector容器分配的是一块连续的内存空间，每次容器的增长，并不是在原有连续的内存空间后再进行简单的叠加，而是重新申请一块更大的新内存，并把现有容器中的元素逐个复制过去，同时销毁旧的内存。这时原有指向旧内存空间的迭代器已经失效，所以当操作容器时，迭代器要及时更新。

### 各种容器对比

|      容器      |    底层数据结构     |                         时间复杂度                         | 有无序 | 可不可重复 |                             其他                             |
| :------------: | :-----------------: | :--------------------------------------------------------: | :----: | :--------: | :----------------------------------------------------------: |
|     array      |        数组         |                       随机读改 O(1)                        |  无序  |   可重复   |                       支持快速随机访问                       |
|     vector     |        数组         | 随机读改、尾部插入、尾部删除 O(1)、头部插入、头部删除 O(n) |  无序  |   可重复   |                       支持快速随机访问                       |
|      list      |      双向链表       |               插入、删除 O(1)、随机读改 O(n)               |  无序  |   可重复   |                         支持快速增删                         |
|     deque      |      双端队列       |                  头尾插入、头尾删除 O(1)                   |  无序  |   可重复   | 一个中央控制器 + 多个缓冲区，支持首尾快速增删，支持随机访问  |
|     stack      |    deque / list     |                  顶部插入、顶部删除 O(1)                   |  无序  |   可重复   | deque 或 list 封闭头端开口，不用 vector 的原因应该是容量大小有限制，扩容耗时 |
|     queue      |    deque / list     |                  尾部插入、头部删除 O(1)                   |  无序  |   可重复   | deque 或 list 封闭头端开口，不用 vector 的原因应该是容量大小有限制，扩容耗时 |
| priority_queue | vector + max - heap |                    插入、删除 O(log2n)                     |  有序  |   可重复   |                  vector容器 + heap处理规则                   |
|      set       |       红黑树        |                 插入、删除、查找 O(log2n)                  |  有序  |  不可重复  |                                                              |
|    multiset    |       红黑树        |                 插入、删除、查找 O(log2n)                  |  有序  |   可重复   |                                                              |
|      map       |       红黑树        |                 插入、删除、查找 O(log2n)                  |  有序  |  不可重复  |                                                              |
|    multimap    |       红黑树        |                 插入、删除、查找 O(log2n)                  |  有序  |   可重复   |                                                              |

### map

map内部自建一棵**红黑树**（一种非严格意义上的平衡二叉树），这棵树具有对数据自动排序的功能，所以在map内部所有的数据都是**有序的**。自动建立Key-value的一一对应关系，key和value可以是任意你需要的类型，但是需要注意的是对于key的类型，唯一的约束就是必须支持<操作符（一般使用<实现==的功能，因为==可能没有重载）。对于迭代器来说，**不可以修改键值，只能修改其对应的实值，查找的复杂度基本是Log(N)**。

**map的插入**

map的插入有3种方式：用insert函数插入pair数据，用insert函数插入value_type数据和用数组方式插人数据。用insert函数插入数据，在数据的插入上涉及集合的唯一性这个概念，即**当map中有这个关键字时，insert操作是插入数据不了的**，但是用数组方式就不同了，它可以**覆盖以前该关键字对应的值**。

**map的查找**

- **用count函数来判定关键字是否出现，其缺点是无法定位数据出现位置**，由于map的一对一的映射特性，就决定了count函数的返回值只有两个，要么是0，要么是1，当要判定的关键字出现时返回1。
- **用find函数来定位数据出现位置，它返回的一个迭代器**，当数据出现时，它返回数据所在位置的迭代器;如果map中没有要查找的数据，它返回的迭代器等于end函数返回的迭代器。

**红黑树的原理**

红黑树，一种二叉查找树，但在每个结点上增加一个存储位表示结点的颜色，可以是Red或Black。通过对任何一条从根到叶子的路径上各个结点着色方式的限制，红黑树确保没有一条路径会比其他路径长出两倍，因而是接近平衡的。

二叉查找树，也称有序二叉树（ordered binary tree），或已排序二叉树（sorted binary tree），是指一棵空树或者具有下列性质的二叉树：

- 若任意节点的左子树不空，则左子树上 所有结点的值均小于它的根结点的值;
- 若任意节点的右子树不空，则右子树上所有结点的值均大于它的根结点的值;
- 任意节点的左，右子树也分别为二叉查找树;
- 没有键值相等的节点（no duplicate node）。

因为一棵由n个结点随机构造的二叉查找树的高度为lgn，所以二叉查找树的一般操作的执行时间为O(lgn)，但二叉查找树若退化成了一棵具有n个结点的线性链后，则这些操作最坏情况运行时间为O(n)。

红黑树虽然本质上是一棵二叉查找树，但它在二叉查找树的基础上增加了着色和相关的性质使得红黑树相对平衡，从而保证了红黑树的查找，插入，删除的时间复杂度最坏为O(logn)。

红黑树的性质：

- 每个结点要么是红的要么是黑的;
- 根结点是黑的;
- 每个叶结点（叶结点即指树尾端NIL指针或NULL结点）都是黑的;
- 如果一个结点是红的，那么它的两个儿子都是黑的;
- 对于任意结点而言，其到叶结点树尾端NIL指针的每条路径都包含相同数目的黑结点。

### 函数对象（仿函数）

函数对象是调用操作符的类，其对象常称为函数对象（function object），它们是行为类似函数的对象。表现出一个函数的特征，就是通过“对象名+（参数列表 ）” 的方式使用一个类，其实质是对operator()操作符的重载。

**函数对象示例：**

```c++
template <class T>
struct less:binary_function<T,T,bool>{
  bool operator()(const T& x, const T& y)const{
    return x < y;
  }
};
```

### set

set作为一个容器也是用来存储同一数据类型的数据类型，并且能从一个数据集合中取出数据，**在set中每个元素的值都唯一的**，而且系统能根据元素的值自动进行排序，应该注意的是set中数元素的值不能直接被改变。

**map和set的插入删除效率比序列容器高的原因**

因为对于关联容器来说，不需要做内存拷贝和内存移动，set容器内所有元素都是以节点的方式来存储，其节点结构和链表差不多，指向父节点和子节点。

### STL迭代器失效的情况和原因

迭代器失效分三种情况考虑，也是分三种数据结构考虑，分别为数组型，链表型，树型数据结构。

- **数组型数据结构**
  - 该数据结构的元素是分配在连续的内存中。
  - 相对于vector来说，每一次删除和插入，指针都有可能失效。 因为为了保证内部数据的连续存放，iterator指向的那块内存在删除和插入过程中可能已经被其他内存覆盖或者内存已经被释放了。即使用push_back的时候，容器内部空间可能不够，需要一块新的更大的内存，只有把以前的内存释放，并申请新的更大的内存，并复制已有的数据元素到新的内存，最后把需要插入的元素放到最后来解决，那么以前的内存指针自然就不可用了。
  - insert和erase操作，都会使得删除点和插入点之后的元素挪位置，所以，插入点和删除掉之后的迭代器全部失效，也就是说`insert(*iter)`(或`erase(*iter)`)，然后再`iter++`，是没有意义的。需要写为`mapStudent.erase(iter++);`，而不是`erase(iter)`，然后`iter++`，因为iter指针被erase之后就失效了。
  - 解决方法：`erase(*iter)`的返回值是下一个有效迭代器的值。 `iter = cont.erase(iter);`。

```c++
//不要直接在循环条件中写++iter
for (iter = cont.begin(); iter != cont.end();)
{
   (*it)->doSomething();
   if (shouldDelete(*iter))
      iter = cont.erase(iter);  //erase删除元素，返回下一个迭代器
   else
      ++iter;
}
```

- **链表型数据结构**
  - 对于list型的数据结构，使用了不连续分配的内存。
  - 插入不会使得任何迭代器失效。
  - 删除运算使指向删除位置的迭代器失效，但是不会失效其他迭代器。
  - 解决办法两种，erase(*iter)会返回下一个有效迭代器的值，或者erase(iter++)。
- **树形数据结构**
  - 使用红黑树来存储数据。
  - 插入不会使得任何迭代器失效。
  - 删除运算使指向删除位置的迭代器失效，但是不会失效其他迭代器。
  - **erase迭代器只是被删元素的迭代器失效，但是返回值为void**，所以要采用erase(iter++)的方式删除迭代器。

**注意** ：经过`erase(iter)`之后的迭代器完全失效，该迭代器iter不能参与任何运算，包括`iter++`，`*ite`。

### 哈希表实现原理

STL中的哈希表使用的是拉链法解决哈希表冲突问题。

![STL哈希表](/Users/wushengna/manual/img/img-post/STL哈希表.png)

哈希表中的bucket所维护的list既不是list也不是slist，而是其自己定义的由hashtable_node数据结构组成的linked-list，而bucket聚合体本身使用vector进行存储。哈希表的迭代器只提供前进操作，不提供后退操作。

在哈希表设计bucket的数量上，其内置了28个质数[53，97，19，....，429496729]，在创建哈希表时，会根据存入的元素个数选择大于等于元素个数的质数作为哈希表的容量（vector的长度），其中每个bucket所维护的linked-list长度也等于哈希表的容量。如果插入哈希表的元素个数超过了bucket的容量，就要进行重建table操作，即找出下一个质数（判断n是不是质数的方法：用n除2到sqrt(n)范围内的数），创建新的buckets vector，重新计算元素在新哈希表的位置（注意STL没有直接将数据从旧桶遍历拷贝数据插入到新桶，而是通过指针转换两个桶的地址），通过swap函数将新桶和旧桶交换，销毁新桶。

### STL中的swap函数

- 除了数组，其他容器在交换后本质上是将内存地址进行了交换，而元素本身在内存中的位置是没有变化。
- swap在交换的时候并不是完全将2个容器的元素互换，而是交换了2个容器内的内存地址。

### vector的iterator，const_iterator和const iterator

- 三种的区别
  - iterator，可遍历，可改变所指元素。
  - const_iterator，可遍历，不可改变所指元素。
  - const iterator，不可遍历，可改变所指元素。
- const_iterator转iterator，iterator不能转const_iterator
  - const_iterator 主要是 **在容器被定义成常量、或者非常量容器但不想改变元素值的情况** 下使用的，而且容器被定义成常量之后，它返回的迭代器只能是const_iterator。
  - 有些容器成员函数只接受iterator作为参数，而不是const_iterator。那么，如果你只有一const_iterator，在它所指向的容器位置上插入新元素的方式：
    - const_iterator转iterator。
    - 强制转换的函数会报错，只能通过 `advance(a, distance(a, b));` 其中，distance用于取得两个迭代器之间的距离，advance用于将一个迭代器移动指定的距离。
    - 如果a是iterator，b是const_iterator，distance会报错，可以显式的指明distance调用的模板参数类型，从而避免编译器自己得出它们的类型。

```c++
typedef deque<int> IntDeque;
typedef IntDeque::iterator iter;
typedef IntDeque::const_iterator ConstIter;
IntDeque d;
ConstIter ci;
Iter i(d.begin());
advance(i,distance<ConstIter>(i,ci)); 
```