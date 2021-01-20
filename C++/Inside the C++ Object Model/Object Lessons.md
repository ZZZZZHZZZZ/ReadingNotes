# Object Lessons 第一章 关于对象
C语言将“数据”与“处理数据的函数”分开声明，语言并没有支持“数据和函数”之间的关联性，这种程序方法称为程序性的；C++中则采用独立的抽象数据类型来实现。

### Class成本
将一个C语言Struct 与全局函数转换为C++的ADT Class并没有增加布局成本。

data member像存在于struct对象一样存在于每一个class object之中。

member function 虽然在class进行了声明，但是并不出现在实例化的object之中。每一个非内联 member function只会诞生一个函数实例；而每一个内联member function则会在每一个使用者身上产生一个实例。

C++在布局和存取时间上的主要额外负担是由virtual引起的，包括：

- virtual function 用以支持一个有效率的执行器绑定。
- virtual base class 用以实现“多次出现在继承体系中的base class，有一个单一而被共享的实例。
- 多重继承下的额外负担，发生在一个派生类和其第二或后继的基类的转换之间。

## 正文
### 1.1 C++对象模式
C++中有两种class data member ：static 和 nonstatic; 三种class member function :static, nonstatic, virtual。

#### 例子 class Point
 ``` C++
class Point{
    public:
        Point (float xval);
        virtual ~Point();
        float x()const;
        static int PointCount();
    protected:
        virtual ostream & print(ostream & os)const;
        float _x;
        static int _point_count;
}
 ``` 
#### 简单对象模型
简单对象模型使用空间和执行期的效率获得了最简单的C++编译器设计复杂度。

此模型中一个object被设计为一系列的slot,object中的每一个成员（数据成员和函数成员）按照其声明顺序，均被一个slot所指向。

在此模型中只有指向object member的指针存放在object中，可以避免因member类型不同而导致需求的存储空间不同的问题（ps.可以重复使用同一块地址空间指向的成员函数以减小内存占用）。
#### 表格驱动对象模型
每一个object内含两个指针
- 一个指针指向member data table,内含所有的成员变量。
- 一个指针指向function member table,内含所有的成员函数（ps.更好的空间复用）。
#### C++对象模型
最初的C++对象模型有简单对象模型继承而来，在此模型中：
- nonstatic data member被配置于每一个class object之内。
- static data member则被存放在个别（？）的class object之外。
- static function于non static function 也被放在个别（？）的class object之外
- vritual function 以两个步骤配置：

    - 每一个class产生对每个virtual function产生一个指针，并将指针存放在virtual function table之中。
    - 每一个class object均包含一个指向此virtual function table的指针VPTR，此指针的设定和重置均由类自动完成，每一个类关联的type_info_object（用以支持runtime type identification) 也由virtual table指出放于表格的第一个slot。

缺点在于如果应用程序代码并未改变，但所用到的class object的nonstatic object（存放于object之中）有所修改，则代码需要重新编译。

#### 虚继承
C++支持单一继承与多重继承，并支持虚继承
```C++
class istream:virtual public ios{};
class ostream:virtual public ios{};
class iostream:public istream,pubilc ostream{};
```
虚继承意为共享，在虚拟继承情况下，不论base class在派生串链中被派生多少次，永远只会存在一个实例，例如iostream之中就只有virtual ios base class的一个实例。

ps
- 虚继承的存在为了是为了解决菱形继承中出现的命名冲突与数据冗余问题
- 例如若不是虚继承，则iostrem将会有两份继承而来的ios成员，而虚继承则保证在派生类中只保存一份来自间接基类的成员

派生类如何产生基类的部分？
-  在简单对象模型中可在派生类中产生一个指向基类部分的指针，以间接性的读取导致的时间与空间的负担换取低耦合性（派生类的大小不会因基类的改变而受到影响，ps. 类似于PIMPL，point to implementation, 指向实现的指针）。

- 类似于表格驱动对象模型，对派生类所指向的每一个基类均产生一个指向此基类的指针，并将其保存在一个 base class table之中，派生类object产生一个base table point 指向此base class table。
  - 相比简单对象模型不但使用间接性的读取导致时间与空间的负担来换取低耦合。
  - 并且在每一个class object之中都有着相同的继承表现方式：
    - 类中存放的base table point 大小与所继承 base class的大小与个数无关。
    - 无需改变class object就可以更改base class table。

C++ 2.0大致采用表格驱动对象模型存储 virtual base class。

### 1.2 关键词所带来的差异
当语言无法区分是一个声明还是一个表达式，则将会判断为声明
```C++
int (*pf) (1024) //函数调用
int (*pq) ()     //函数声明
```
C++中对处于同一access section中的数据，必定保证其以声明顺序出现在内存布局中，但对于放置在多个access section中的数据不能保证相对顺序。

**能用class就别用struct**

C struct在C++中的唯一合理用途，只在于在组合的情况下，将数据封装为一个struct传入一个C的函数中，并可保证拥有与C兼容的空间布局。

如果在继承关系下，编译器会决定是否应该有额外的data member被安置在base struct subobject中

### 1.3 对象的差异
在C++中，多态只存在于一个个public class体系中，要求此object必须可以经由一个pointer或reference来存取，才支持OO的多态性质
```C++
class Book:public Library_materials{};
Library_materials thing1;
Book book;
thing1=book;//book被裁切为thing1
thing1.check_in();//调用的是Library_materials.check_in();
Library_materials& thing2=book;
thing2.check_in();//调用的是Book.check_in();
```
C++以下列方法支持多态
- 经由一组隐式的转换操作，例如将一个derived class指针转换为一个指向其public base class的指针
```C++
shape* ps=new circle();
```
- 经由virtual function机制

经由虚函数的多态可以避免由于“借助某一特定library的materials
”而导致变动无常，使得
    - 当类型有所增加，修改或删除时，我们的程序代码无需改变
    - 使一个新的Library_materials subtype的供应者不许重新写出对继承体系中所有类型都共通的行为和操作。
- 经由dynamic_cast和typeid运算符
```C++
if(circle *pc=dynamic_cast<circle* >(ps)){}
```













