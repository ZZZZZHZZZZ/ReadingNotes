# 条款2 尽量使用const, enum, inline 替换#define(以编译器替代预处理器)
```C++
#define ASPECT_RATIO 1.653
```
define不被视为语言的一部分，可能导致ASPECT_RATIO从未被编译器看到（在编译器处理源码之前就被预处理器移走），导致此常量没有进入符号表，从而若使用此常量时出现错误，编译错误信息将无法提示到ASPECT_RATIO。

使用常量替换宏定义
```C++
const double AspectRatio=1.653
```
优势在于
- 常量肯定会被编译器看到并添加进记号表中
- 对浮点型常量，使用常量可能获得比define更小量的数据代码（因为编译器可能直接使用1.653替换代码中ASPECT_RATIO的部分，导致生成的目标代码中出现了多份1.653，而使用常量则不会出现此问题）。

#### 常量指针
```C++
const char* temp1; //指针所指向的数据是一个常量不可改变，但指针本身可以改变
char* const temp2; //指针是一个常量，不能更改其指向的地址，但指针指向地址内保存的内容可以改变。
const char* const temp3="hello  world";
```
#### 类常量成员
```C++
class GamePlayer{
    private:
        static const int NumTurns=5;
        int scores[NUMTurns];
}
```
此为NumTurns的声明式而非定义式，但对于整数型（int,  char, bool）的类static常量，如果不取用他们的地址，则可以只声明而不用定义。

若需提供定义式
```C++
const int GamePlayer::NumTurns //无赋值操作
```
将此语句放入实现文件而非头文件，因为class常量已在声明时获得初值，因此定义时不可再设初值。

- define不可用于定义class专属常量。
- 取一个enum与define的地址是非法的，但取一个const的地址是合法的
  - 如果不想让别人获得一个pointer或reference指向某个整数常量，则可用enum实现此约束。

#### 宏函数
将所有的宏函数替换为template inline函数
```C++
#define CALL_WITH_MAX(a,b) f((a) > (b) ? (a):(b));
template<typename T>
inline void callWithMax(const T& a, const T& b){
    f(a > b ? a : b);
}
```
#### 总结
- 对于单纯常量，最好用const对象或enums替换#define.
- 对于形似函数的宏，最好用inline函数替换#define.
