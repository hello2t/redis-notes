# 二次宏扩展以便对宏定义值进行字符传化
### 来源

zmalloc.h

### 代码

```c
#define _xstr(s) str(s)
#define str(s) #s
```

### 说明
