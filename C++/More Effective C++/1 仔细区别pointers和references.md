# 条款1 仔细区别pointers和references
## 区别
### 1 没有null reference，即reference 必须总代表一个对象，即必须有初值。
```C++
string &rs; //错误，引用必须被初始化
string s("hello world");
string & rs=s; //正确
```
没有null reference，即使用reference时不用再测试是否有效，即对于pointer必须检查是否为nullptr。
### 2 pointer可以被指向另一个对象，reference不可
```C++
string s1("hello"),s2("world");
string &rs=s1;
string *ps=&s1;
rs=s2 //rs仍然指向s1,但s1值变为world
ps=&s2 //ps指向s2,s1无变化
```

- 如果目的是指向一个对象或不指向任何对象，或可指向不同的对象，则应该用pointer.
- 如果变量总是代表一个对象，即不允许此变量为null，并且一旦代表了此对象就不再改变，则应该用reference
- 此外，[]操作符也将返回reference
```C++
char *pointer=nullptr;
char &reference=*pointer;
```
此代码将产生未定义行为。

