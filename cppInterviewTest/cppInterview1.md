## 参考fei guo面试题，自己做了知识点的深挖，总结
#### 结构体和共同体的区别 struct 和 union
在回答这个问题之前，先来看看这两段代码
```cpp
struct Node{
    int i;
    Node *next;
    char ctr;
    const char* name;
};

union Type{
    int i;
    char c;
    double d;
};
```
容我捋一捋。首先， **union是一种特殊的struct,struct又是一种特殊的class。** 在上面这个代码中，其实最主要的区别就是在内存的存储方式不同，以及因为这个存储方式的不同，所产生的对应的限制也不一样。
* 先来看看struct，这个Node在内存的空间中的存储，是通过对齐的方式存储的，也就是说，next指针占最大的空间存储，一般在64位系统占的是8位，int占4位，而在32位系、统占的是2位，指针占4位。因此，采用对齐来计算Node占的字节就是32字节，在64位系统。那么struct里面的声明是怎样的呢？里面的声明也是按声明的顺序以此在内存排布存放的。那么对于以上的声明当然是通过对齐的方式依照声明的顺序来排布，那么以下这个情况呢？
```cpp
struct Linka{
    int *next;
    char a;
    char b;
};
```
那么以上的字节是多少呢？是16，为什么呢？因为内存采用对齐的方式存放，首先的8位存放了指针，然后采用对齐的方式存放char类型的字符，char类型占的是1个字节，两个char就是两个字节，那么采用对齐的方式，应该占用的是3*8才对啊？可是不然，这里占用的是16字节，因为在对象对齐，很多机器上，会造成一定程度上的空洞，为了弥补这个空洞，机器会将char持续放在char的后面，从而优化了程序的性能。（这里可以说是比较好的设计方式，通过排列方式减少内存的占用）。
* 那么对于声明来说，struct有什么特别之处呢？<br>
在struct中，声明了基本上就可以直接使用了，来看看下面这个代码
```cpp
struct getRecur{
    getRecur member; // 递归定义
};
```
这个在使用的过程中，会出现一个很严重的问题，递归定义，这个在编译器中会告知你这是一个未完成的声明，因此不能使用。而且，通常来说，在前面定义了这个类型，而且不是指针的情况下，就是要实例这个对象，而这个对象都尚未确定，何来实例呢？所以一般指针，是可以的。那为什么指针可以呢？指针又是一个什么样的类型呢？在这个🌰 `getRecur *ptr`就是表示ptr是一个指向getRecur的类型的指针，所以ptr并不是一个实例化对象，它仅作为一个声明。<br>
* 对于一个struct的结构，与class有什么区别呢？<br>
struct是一种class，他可以包含，默认的cpp生成的四种函数，还有也包含成员函数，但是他的成员默认是public的<br>
我们来看一个🌰:<br>
```cpp
struct Points{
    Points(int _n):n(_n){}
    private:
        int n;
}
```
这样也是可以的，它里面也能有对应的虚函数，并且也能对类内的成员进行初始化。那么在别样的构造也是可以的嘛？据我所知的初始化方式还有一种就是直接给他赋值，来看一下下面这个例子吧。
```cpp
struct Points{
     int num;
     char c;
     char pctr[2];
};
Points la = {1,'p',{'k','o'}};
```
这样的初始化也是可以的，那么为什么也是可以的呢？这得益于struct的有序声明，可以以此将声明初始化存放在依次的有序。<br>
说了那么多的struct我们来探讨一下union。<br>
* union取其英文字就是“联合”的意思。<br>
联合，就是联合在一起，在内存也是一样，内存地址联合在一起。一起使用一串内存。union中，它给所有成员都分配在同一个地址空间中。因此在使用一个union，它实际占用的空间大小与其最大的成员是一样的。(这里区分了struct的对齐方式)。但是我也在上面说过，union是一种特殊的struct，所以，struct有的东西，它没有，但是它有的东西，struct就有，就像那个弥补空洞的地方。我们来看一段代码
```cpp
union Linkb{
    char *s;
    int i;
}
```
这段代码的占用空间是多少呢？是8字节，没错，很对，这里其实也看出来联合的意思了。那么使用的时候呢？是不是两个值都能用？是的，两个值都可以使用，但是不能同时用，也就是说，只能一次用一个。
```cpp
    Linkb lb;
    lb.s = &i;
    cout << *(lb.s) << endl;
```
上述代码就会打印出一个字符,那么，当你使用lb.i的时候，编译器就会告诉你，i并不是这个linkb的成员。所以，当且仅当，我们有些代码需要动态变化，仅使用一个的时候，才会去使用union。
* 由于union是一个特殊的struct，所以有些东西，因为联合的原因，它是不能有的，就像虚函数，引用类型成员，基类，析构函数，构造函数.union不能用作其他类的基类。这些在编译期间都会出错，而且因为这样的出错，避免了很多错误的发生。<br>
* 那么union通常用来做什么？<br>
优化代码的性能。很简单。<br>
* 但是要复杂呢？
那么你可以定一个包含union的类，在里面定义构造函数，析构函数等，用来操作union。
所以总的来说，union的值不能同时存在，但是要是对一个值赋值，又对另外一个赋值，肯定会改写整个地址空间的值
```cpp
    int i = 42;
    Linkb lb;
    lb.next = 42;
    lb.a = 'p';
    cout << lb.a << endl;
```
这段代码lb.next输出是112，lb.a输出是p，而p的ascii码是112，所很明显是改了的。也充分说明了，这个union里面的成员，同时占用同一个内存地址空间。
#### 未完待续...