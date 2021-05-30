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

### vector（2）

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
- resize()函数只改变容器的元素数目，不改变容器大小。
  - 用reserve(size_type)只是扩大capacity值，这些内存空间可能还是“野”的，如果此时使用下标访问，则可能会越界，而resize(size_type new_size)会真正使容器具有new_size个对象。
- 使用“交换技巧”来修整vector过剩空间/内存。
  - vector的内存占用空间只增不减，使用erase是清除元素，而不是容量，所有内存空间是在vector析构时候才能被系统回收。
  - `vector<int>(ivec).swap(ivec);`，表达式vector< int >(ivec)表示建立一个临时vector，它是ivec的一份拷贝。但是**vector的拷贝构造函数只分配拷贝的元素需要的内存**，所以这个临时vector没有多余的容量。然后临时vector和ivec交换数据完成，但ivec只有临时变量的修整过的容量，而这个临时变量则持有了曾经在ivec中的没用到的过剩容量。在这个语句结尾处，临时 vector被销毁，以释放以前ivec使用的内存，收缩到合适的大小。

**vector的扩容机制**

不同的编译器，vector有不同的扩容大小，**在vs下是1.5倍，在gcc下是2倍**，vector容器分配的是一块连续的内存空间，每次容器的增长，并不是在原有连续的内存空间后再进行简单的叠加，而是重新申请一块更大的新内存，并把现有容器中的元素逐个复制过去，同时销毁旧的内存。这时原有指向旧内存空间的迭代器已经失效，所以当操作容器时，迭代器要及时更新。

### vector，array和数组的区别（1）

**相同点：**

- 三者均可以使用下标运算符对元素进行操作，即vector和array都针对下标运算符[]进行了重载。

- 三者在内存的方面都使用连续内存，即在vector和array的底层存储结构均使用数组。

**不同点：**

- vector属于变长容器，即可以根据数据的插入删除重新构建容器容量（1.5倍或者2倍）；但array和数组属于定长容量。

- vector和array提供了更好的数据访问机制，即可以使用front和back以及at访问方式，使得访问更加安全。而数组只能通过下标访问，在程序的设计过程中，更容易引发访问错误。

- vector和array提供了更好的遍历机制，即有正向迭代器和反向迭代器两种。

- vector和array提供了size和判空的获取机制，而数组只能通过遍历或者通过额外的变量记录数组的size。

- vector和array提供了两个容器对象的内容交换，即swap的机制，而数组对于交换只能通过遍历的方式，逐个元素交换的方式使用。

- array提供了初始化所有成员的方法fill。

- vector提供了可以动态插入和删除元素的机制，而array和数组则则需要自己实现完成。

- 由于vector的动态内存变化的机制，在插入和删除时，需要考虑迭代器的是否失效的问题。

### vector和list的区别（1）

- vector和**数组**类似，**拥有一段连续的内存空间**，并且**起始地址不变**，因此能**高效的进行随机存取**，时间复杂度为**O(1)**;但因为内存空间是连续的，所以在进行**插入和删除操作**时，会造成内存块的拷贝，时间复杂度为**O(n)**。另外当数组中内存空间不够时，会重新申请一块内存空间并进行内存拷贝。连续存储结构：vector是可以实现动态增长的对象数组，支持对数组高效率的访问和在数组尾端的删除和插入操作，在中间和头部删除和插入相对不易，需要挪动大量的数据。它与数组最大的区别就是vector不需程序员自己去考虑容量问题，库里面本身已经实现了容量的动态增长，而数组需要程序员手动写入扩容函数进形扩容。

- list是由**双向链表**实现的，还是一个环状双向链表，因此**内存空间是不连续的**，只能通过指针访问数据，实现**随机存取**，时间复杂度为**O(n)**;但由于链表的特点，能**高效地进行插入和删除**，时间复杂度为**O(1)**。非连续存储结构：list是一个双链表结构，支持对链表的双向遍历。每个节点包括三个信息：元素本身，指向前一个元素的节点(prev)和指向下一个元素的节点(next)，因此list可以高效率的对数据元素任意位置进行访问和插入删除等操作。由于涉及对额外指针的维护，所以**开销比较大**。list不提供随机访问，所以不能用下标直接访问到某个位置的元素，要访问list里的元素只能遍历，不过要是只需要访问list的最后N个元素的话，可以用反向迭代器来遍历。

**区别：**

- vector拥有连续的内存空间，并且起始地址不变，vector的随机访问效率高，但在插入和删除时（不包括尾部）需要挪动数据，不易操作。list拥有不连续的内存空间，list的访问要遍历整个链表，它的随机访问效率低，但对数据的插入和删除操作等都比较方便，改变指针的指向即可。
- vector的维护开销较小，list需要维护指针开销较大。
- vector和list对于迭代器的支持不同。
  - 相同点是vector< int >::iterator和list< int >::iterator都重载了“++”操作。
  - 不同点是在vector中，iterator支持 ”+“、”+=“，”<"等操作，而list中则不支持。
- vector中的迭代器在使用后就失效了，而list的迭代器在使用之后还可以继续使用。

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

### map（1）

map内部自建一棵**红黑树**（一种非严格意义上的平衡二叉树），这棵树具有对数据自动排序的功能，所以在map内部所有的数据都是**有序的**。自动建立Key-value的一一对应关系，key和value可以是任意你需要的类型，但是需要注意的是对于key的类型，唯一的约束就是必须支持<操作符（一般使用<实现==的功能，因为==可能没有重载）。对于迭代器来说，**不可以修改键值，只能修改其对应的实值，查找的复杂度基本是Log(N)**。

**map的插入**

map的插入有3种方式：用insert函数插入pair数据，用insert函数插入value_type数据和用数组方式插人数据。用insert函数插入数据，在数据的插入上涉及集合的唯一性这个概念，即**当map中有这个关键字时，insert操作是插入数据不了的**，但是用数组方式就不同了，它可以**覆盖以前该关键字对应的值**。

**map的查找**

- **用count函数来判定关键字是否出现，其缺点是无法定位数据出现位置**，由于map的一对一的映射特性，就决定了count函数的返回值只有两个，要么是0，要么是1，当要判定的关键字出现时返回1。
- **用find函数来定位数据出现位置，它返回的一个迭代器**，当数据出现时，它返回数据所在位置的迭代器；如果map中没有要查找的数据，它返回的迭代器等于end函数返回的迭代器。

**map使用下标访问**

- 将key作为下标去执行查找，并返回相应的值；如果不存在这个key，就将一个具有该key和value的某人值插入这个map。

**红黑树的原理**

红黑树，一种二叉查找树，但在每个结点上增加一个存储位表示结点的颜色，可以是Red或Black。通过对任何一条从根到叶子的路径上各个结点着色方式的限制，红黑树确保没有一条路径会比其他路径长出两倍，因而是接近平衡的。

二叉查找树，也称有序二叉树（ordered binary tree），或已排序二叉树（sorted binary tree），是指一棵空树或者具有下列性质的二叉树：

- 若任意节点的左子树不空，则左子树上所有结点的值均小于它的根结点的值;
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

### STL迭代器失效的情况和原因（2）

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

**注意**：经过`erase(iter)`之后的迭代器完全失效，该迭代器iter不能参与任何运算，包括`iter++`，`*ite`。

### 哈希表实现原理（2）

STL中的哈希表使用的是拉链法解决哈希表冲突问题。

![STL哈希表](/Users/wushengna/manual/img/img-post/STL哈希表.png)

哈希表中的bucket所维护的list既不是list也不是slist，而是其自己定义的由hashtable_node数据结构组成的linked-list，而bucket聚合体本身使用vector进行存储。哈希表的迭代器只提供前进操作，不提供后退操作。

在哈希表设计bucket的数量上，其内置了28个质数[53，97，19，....，429496729]，在创建哈希表时，会根据存入的元素个数选择大于等于元素个数的质数作为哈希表的容量（vector的长度），其中每个bucket所维护的linked-list长度也等于哈希表的容量，当linked-list长度超过8时，就自动转为红黑树进行组织。如果插入哈希表的元素个数超过了bucket的容量，就要进行重建table操作，即找出下一个质数（判断n是不是质数的方法：用n除2到sqrt(n)范围内的数），创建新的buckets vector，重新计算元素在新哈希表的位置（注意STL没有直接将数据从旧桶遍历拷贝数据插入到新桶，而是通过指针转换两个桶的地址），通过swap函数将新桶和旧桶交换，销毁新桶。

### STL中unordered_map和map的区别

* unordered_map是使用哈希表实现的，占用内存比较多，查询速度比较快，是常数时间复杂度，它内部是无序的，需要实现==操作符。
* map底层是采用红黑树实现的，插入删除查询时间复杂度都是O(logn)，它的内部是有序的，因此需要实现比较操作符(<)。

### STL中的swap函数

- 除了数组，其他容器在交换后本质上是将内存地址进行了交换，而元素本身在内存中的位置是没有变化。
- swap在交换的时候并不是完全将2个容器的元素互换，而是交换了2个容器内的内存地址。

### vector的iterator，const_iterator和const iterator

- 三种的区别
  - iterator，可遍历，可改变所指元素。
  - const_iterator，可遍历，不可改变所指元素。
  - const iterator，不可遍历，可改变所指元素。
- const_iterator转iterator，iterator不能转const_iterator
  - const_iterator 主要是**在容器被定义成常量、或者非常量容器但不想改变元素值的情况**下使用的，而且容器被定义成常量之后，它返回的迭代器只能是const_iterator。
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

### 什么是trivial destructor

“trivial destructor”是指用户没有自定义析构函数，而由系统生成的，反之用户自定义了析构函数，则称之为“non-trivial destructor”，这种析构函数**如果申请了新的空间一定要显式的释放，否则会造成内存泄露**。

对于trivial destructor，如果每次都进行调用，对效率是一种伤害，STL可以进行判断是否使用，首先利用value_type()获取所指对象的型别，再利用`__type_traits`判断该型别的析构函数是否trivial，若是(`__true_type`)，则什么也不做，若为(`__false_type`)，则去调用destory()函数，也就是说，在实际的应用当中，STL库提供了相关的判断方法`__type_traits`。除了trivial destructor，还有trivial construct、trivial copy construct等，如果能够对是否trivial进行区分，可以采用内存处理函数memcpy()、malloc()等更加高效的完成相关操作，提升效率。

### STL中的traits技法

traits技法利用“内嵌型别“的编程技巧与**编译器的template参数推导功能**，常用的有iterator_traits和type_traits。

**iterator_traits**

被称为**特性萃取机**，能够方面的让外界获取以下5中型别：

- value_type：迭代器所指对象的型别。
- difference_type：两个迭代器之间的距离。
- pointer type：迭代器所指向的型别。
- reference type：迭代器所引用的型别。
- iterator_category：
  - Input Iterator：这种迭代器所指对象只读。
  - Output Iterator：只写。
  - Forward Iterator：在这种迭代器形成的区间上进行读写操作。
  - Bidirectional Iterator：可以双向移动。
  - Random Access Iterator：可以随机移动。
  - Iterator_traits< I >::iterator_category()产生临时对象，根据这个型别，编译器决定调用哪个__advance()重载函数。

**type_traits**

关注的是型别的**特性**，例如这个型别是否具备non-trivial defalt ctor（默认构造函数），non-trivial copy ctor（拷贝构造函数），non-trivial assignment operator（赋值运算符） 和non-trivial dtor（析构函数），如果答案是否定的，可以采取直接操作内存的方式提高效率，一般来说，type_traits支持以下5中类型的判断：

```cpp
__type_traits<T>::has_trivial_default_constructor
__type_traits<T>::has_trivial_copy_constructor
__type_traits<T>::has_trivial_assignment_operator
__type_traits<T>::has_trivial_destructor
__type_traits<T>::is_POD_type
```

由于编译器只针对class object形式的参数进行参数推导，因此上式的返回结果不应该是个bool值，实际上使用的是一种空的结构体：

```cpp
struct __true_type{};
struct __false_type{};
```

这两个结构体没有任何成员，不会带来其他的负担，又能满足需求，如果我们自行定义了一个Shape类型，也可以针对这个Shape设计type_traits的特化版本。

```cpp
template<> struct __type_traits<Shape>{
    typedef __true_type has_trivial_default_constructor;
    typedef __false_type has_trivial_copy_constructor;
    typedef __false_type has_trivial_assignment_operator;
    typedef __false_type has_trivial_destructor;
    typedef __false_type is_POD_type;
};
```

### STL的两级空间配置器

**为什么需要二级空间配置器**

动态开辟内存时，要在堆上申请，但是如果我们频繁的在堆开辟释放内存，则就会**在堆上造成很多外部碎片**，浪费了内存空间；每次都要进行调用**malloc、free**函数等操作，使空间就会增加一些附加信息，降低了空间利用率；随着外部碎片增多，内存分配器在找不到合适内存情况下需要合并空闲块，浪费了时间，大大降低了效率。于是就设置了二级空间配置器，**当开辟内存<=128bytes时，即视为开辟小块内存，则调用二级空间配置器**，一般默认选择的为二级空间配置器，**如果大于128字节再转去一级配置器**。

**一级配置器**

**一级空间配置器**中重要的函数就是allocate、deallocate、reallocate ，一级空间配置器是以malloc()，free()，realloc()等C函数执行实际的内存配置：

- 直接allocate分配内存，其实就是malloc来分配内存，成功则直接返回，失败就调用处理函数。
- 如果用户自定义了内存分配失败的处理函数就调用，没有的话就返回异常。
- 如果自定义了处理函数就进行处理，完成再继续分配试试。

**二级配置器**

![空间配置器](/Users/wushengna/manual/img/img-post/空间配置器.png)

- 维护16条链表，分别是0-15号链表，最小8字节，以8字节逐渐递增，最大128字节，传入一个字节参数，表示需要多大的内存，会自动校对到第几号链表（如需要13bytes空间，我们会给它分配16bytes大小），在找到第n个链表后查看链表是否为空，如果不为空直接从对应的free_list中拔出，将已经拨出的指针向后移动一位。
- 对应的free_list为空，先看其内存池是不是空时，如果内存池不为空：
  - 先检验它剩余空间是否够20个节点大小（即所需内存大小（提升后） * 20），若足够则直接从内存池中拿出20个节点大小空间，将其中一个分配给用户使用，另外19个当作自由链表中的区块挂在相应的free_list下，这样下次再有相同大小的内存需求时，可直接拨出。
  - 如果不够20个节点大小，则看它是否能满足1个节点大小，如果够的话则直接拿出一个分配给用户，然后从剩余的空间中分配尽可能多的节点挂在相应的free_list中。
  - 如果连一个节点内存都不能满足的话，则将内存池中剩余的空间挂在相应的free_list中（找到相应的free_list），然后再给内存池申请内存。
- 内存池为空，申请内存，此时二级空间配置器会使用malloc()从heap上申请内存，（一次所申请的内存大小为2 * 所需节点内存大小（提升后）* 20 + 一段额外空间），申请40块，一半拿来用，一半放内存池中。
- malloc没有成功，如果malloc()失败了，说明heap上没有足够空间分配给我们了，这时，二级空间配置器会从比所需节点空间大的free_list中一一搜索，从比它所需节点空间大的free_list中拔出一个节点来使用，如果这也没找到，说明比其大的free_list中都没有自由区块了，那就要调用一级适配器了。

释放时调用deallocate()函数，若释放的n>128，则调用一级空间配置器，否则就直接将内存块挂上自由链表的合适位置。

**STL二级空间配置器的缺点**

- 因为自由链表的管理问题，它会把我们需求的内存块自动提升为8的倍数，这时若你需要1个字节，它会给你8个字节，即浪费了7个字节，所以它又引入了**内部碎片**的问题，若相似情况出现很多次，就会造成很多内部碎片；
- 二级空间配置器是在堆上申请大块的狭义内存池，然后用自由链表管理，供现在使用，在程序执行过程中，它将申请的内存一块一块都挂在自由链表上，即不会还给操作系统，并且它的实现中所有成员全是静态的，所以它申请的所有内存只有在进程结束才会释放内存，还给操作系统，由此带来的问题有：
  - 我不断的开辟小块内存，最后整个堆上的空间都被挂在自由链表上，若我想开辟大块内存就会失败；
  - 若自由链表上挂很多内存块没有被使用，当前进程又占着内存不释放，这时别的进程在堆上申请不到空间，也不可以使用当前进程的空闲内存，由此就会引发多种问题。

### STL迭代器如何实现

- 迭代器是一种抽象的设计理念，通过迭代器可以在不了解容器内部原理的情况下遍历容器，除此之外，STL中迭代器一个最重要的作用就是作为容器与STL算法的粘合剂。
- 迭代器的作用就是**提供一个遍历容器内部所有元素的接口**，因此迭代器内部必须保存一个与容器相关联的指针，然后重载各种运算操作来遍历，其中最重要的是*运算符与->运算符，以及++、--等可能需要重载的运算符重载，这和智能指针很相似，智能指针也是将一个指针封装，然后通过引用计数或是其他方法完成自动释放内存的功能。
- 最常用的迭代器的相应型别有五种：value type，difference type，pointer type，reference type，iterator catagoly。

### deque

vector是单向开口（尾部）的连续线性空间，deque则是一种双向开口的连续线性空间，虽然vector也可以在头尾进行元素操作，但是其头部操作的效率十分低下（主要是涉及到整体的移动）。deque和vector的最大差异一个是**deque运行在常数时间内对头端进行元素操作**，二是**deque没有容量的概念，它是动态地以分段连续空间组合而成，可以随时增加一段新的空间并链接起来**。deque虽然也提供随机访问的迭代器，但是其迭代器并不是普通的指针，其复杂程度比vector高很多，因此除非必要，否则一般使用vector而非deque。如果需要对deque排序，可以先将deque中的元素复制到vector中，利用sort对vector排序，再将结果复制回deque。

deque由一段一段的定量连续空间组成，一旦需要增加新的空间，只要配置一段定量连续空间拼接在头部或尾部即可，因此deque的最大任务是如何维护这个整体的连续性。deque内部有一个指针指向map，map是一小块连续空间，其中的每个元素称为一个节点，node，每个node都是一个指针，指向另一段较大的连续空间，称为缓冲区，这里就是deque中实际存放数据的区域，默认大小512bytes。

<img src="/Users/wushengna/manual/img/img-post/deque.png" alt="deque" style="zoom:80%;" />

deque迭代器的“++”、“--”操作是远比vector迭代器繁琐，其主要工作在于缓冲区边界，如何从当前缓冲区跳到另一个缓冲区，当然deque内部在插入元素时，如果map中node数量全部使用完，且node指向的缓冲区也没有多余的空间，这时会配置新的map（2倍于当前+2的数量）来容纳更多的node，也就是可以指向更多的缓冲区，在deque删除元素时，也提供了元素的析构和空闲缓冲区空间的释放等机制。

### stack

stack（栈）是一种先进后出（First In Last Out）的数据结构，只有一个入口和出口，那就是栈顶，除了获取栈顶元素外，没有其他方法可以获取到内部的其他元素，stack这种单向开口的数据结构很容易由**双向开口的deque和list**形成，只需要根据stack的性质对应移除某些接口即可实现，由于stack只能操作顶端的元素，因此其内部元素无法被访问，也不提供迭代器。

stack这种“修改某种接口，形成另一种风貌”的行为，成为adapter（配接器），常将其归类为container adapter而非容器。

### queue

queue（队列）是一种先进先出（First In First Out）的数据结构，只有一个入口和一个出口，分别位于最底端和最顶端，出口元素外，没有其他方法可以获取到内部的其他元素，queue这种“先进先出”的数据结构很容易由双向开口的deque和list形成，只需要根据queue的性质对应移除某些接口即可实现，queue也是一类container adapter，也可以使用list作为底层容器，不具有遍历功能，没有迭代器。

### priority_queue

priority_queue，优先队列，是一个拥有权值观念的queue，它跟queue一样是顶部入口，底部出口，在插入元素时，元素并非按照插入次序排列，它会自动根据权值（通常是元素的实值）排列，权值最高，排在最前面，priority_queue使用一个max-heap完成，底层容器使用的是一般为vector为底层容器，堆heap为处理规则来管理底层容器实现，priority_queue不是容器，而是一种容器配接器，priority_queue的所有元素，进出都有一定的规则，只有queue顶端的元素（权值最高者）才有机会被外界取用，它没有遍历功能，也不提供迭代器。

