
## 二次宏扩展以便对宏定义值进行字符传化
### 来源

zmalloc.h

### 代码

```c
#define _xstr(s) str(s)
#define str(s) #s
```

### 说明
#### 宏定义
宏定义有四种形式(C99 6.10)：
> \# define identifier replacement-list new-line  
> \# define identifier lparen identifier-listopt ) replacement-list new-line  
> \# define identifier lparen ... ) replacement-list new-line  
> \# define identifier lparen identifier-list, ...) replacement-list new-line  

lparen指一个其后不是一个white-space的左括号，

实际上是一种值替换和三种函数替换。

#### 参数替换
1. 如果替换序列中的参数没有前导#或前导##或后续##，则参数会先进行宏展开，然后整个函数宏会被扩展
2. 函数宏的替换列表中的#应该在后面跟着一个宏参数，作为替换列表中的下一个预处理节点。参数和#会一起被一个字符串字面量预处理节点替换。这个节点包含了对应参数的拼写值。

 
### 参考阅读

ISO+IEC+9899-1999 6.10.3.2  
http://gcc.gnu.org/onlinedocs/cpp/Stringification.html  
http://gcc.gnu.org/onlinedocs/cpp/Argument-Prescan.html  
https://developer.apple.com/legacy/library/documentation/DeveloperTools/gcc-4.0.1/cpp/Stringification.html#Stringification  
http://my.oschina.net/moooofly/blog/126187  
http://www.2cto.com/kf/201303/195299.html  


## do{}while(0)在宏定义中的使用
### 来源

zmalloc.c

### 代码

```c
#define update_zmalloc_stat_alloc(__n) do { \
    size_t _n = (__n); \
    if (_n&(sizeof(long)-1)) _n += sizeof(long)-(_n&(sizeof(long)-1)); \
    if (zmalloc_thread_safe) { \
        update_zmalloc_stat_add(_n); \
    } else { \
        used_memory += _n; \
    } \
} while(0)
```
### 说明
为什么会在宏定义里使用do{...}while(0)这样的写法呢？和直接写...有什么区别呢？
考虑代码

```c
#define mymacro() do {\
stmt1;\
stmt2;\
} while(0)

if(flag)
    mymacro();
```
#### 方案1：如果在宏定义里直接使用：
```c
#define mymacro() stmt1; stmt2
```
>注意，我们通常不在宏定义中加最后的“;”，而是在使用时加一个分号，这样更加符合编码习惯。

则
```c
if(flag)
    mymacro();
```
展开之后变为
```c
if(flag)
    stmt1;
stmt2;
```
看出来问题了吗？这样的话，语句逻辑会产生问题。除非stmt1，stmt2用大括号括起来。

#### 方案2：宏定义不使用外圈...，在使用时将mymacro()函数括起来。
这样是可以解决问题的，但是，不是一个好的解决方案。一个优秀的编码人员在编写基本工具时，应该遵守这样的原则：__不要对使用者的行为做任何假设__。但是事实上，作为一个优秀的编码人员，也应该在if语句中使用{}，无论其中有几条语句。也就是__严于律己，宽以待人__吧。

#### 方案3：在宏定义时使用{...}替换do{...}while(0)
我们上述代码展开后变成
```c
if(flag)
    {stmt1;stmt2;};
```
乍看，这样好像没什么问题，虽然最后有个多余的";"，也就是一个多余的空语句。但是如果我们是这样使用的呢：
```c
if(flag)
    mymacro();
else
    ...
```
此时宏展开后变成
```c
if(flag)
    {stmt1;stmt;}
;
else ...
```    
这样就无法通过编译。所以这种方案也不好。

#### do{...}while(0)用于避免写goto。
有的时候我们在走入某一分支后，需要跳过后面一大段代码，使用一个套住的循环，则可以利用break来跳过。

### 参考阅读
<a href = "http://gcc.gnu.org/onlinedocs/gcc-4.1.1/gcc/Statement-Exprs.html#Statement-Exprs">Statement-Exprs</a>
