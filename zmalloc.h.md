### 头文件定义开始，防止重复引用

    #ifndef __ZMALLOC_H

### 定义头文件

    #define __ZMALLOC_H

### 两次宏扩展以便对宏定义值进行字符串化

    /* Double expansion needed for stringification of macro values. */
    #define __xstr(s) __str(s)
    #define __str(s) #s

### 在使用TC_MALLOC或JEMALLOC或APPLE的情况下，使用相应的malloc替换zmalloc_size函数
- 如果使用了TCMALLOC，则ZMALLOC_LIB定义为tcmalloc-a.b，a为TC_VERSION_MAJOR的值，b为TC_VERSION_MINOR的值；引入头文件google/tcmalloc.h；如果tcmalloc版本在1.6或以上则定义HAVE_MALLOC_SIZE为1，且用tc_malloc_size函数替换zmalloc_size函数，否则报错“需要更高版本”；


    #if defined(USE_TCMALLOC)
    #define ZMALLOC_LIB ("tcmalloc-" __xstr(TC_VERSION_MAJOR) "." __xstr(TC_VERSION_MINOR))
    #include <google/tcmalloc.h>
    #if (TC_VERSION_MAJOR == 1 && TC_VERSION_MINOR >= 6) || (TC_VERSION_MAJOR > 1)
    #define HAVE_MALLOC_SIZE 1
    #define zmalloc_size(p) tc_malloc_size(p)
    #else
    #error "Newer version of tcmalloc required"
    #endif

- 如果使用了JEMALLOC，则ZMALLOC_LIB定义为jemalloc-a.b.c，a为JEMALLOC_VERSION_MAJOR的值，b为JEMALLOC_VERSION_MINOR的值，c为JEMALLOC_VERSION_BUGFIX的值；引入头文件jemalloc/jemalloc.h；如果jemalloc版本在2.1或以上，则定义HAVE_MALLOC_SIZE为1，且用je_malloc_usable_size函数替换zmalloc_size函数，否则报错“需要更高版本”


    #elif defined(USE_JEMALLOC)
    #define ZMALLOC_LIB ("jemalloc-" __xstr(JEMALLOC_VERSION_MAJOR) "." __xstr(JEMALLOC_VERSION_MINOR) "." __xstr(JEMALLOC_VERSION_BUGFIX))
    #include <jemalloc/jemalloc.h>
    #if (JEMALLOC_VERSION_MAJOR == 2 && JEMALLOC_VERSION_MINOR >= 1) || (JEMALLOC_VERSION_MAJOR > 2)
    #define HAVE_MALLOC_SIZE 1
    #define zmalloc_size(p) je_malloc_usable_size(p)
    #else
    #error "Newer version of jemalloc required"
    #endif


- 如果使用了APPLE，则引入malloc/malloc.h，则定义HAVE_MALLOC_SIZE为1，且用malloc_size函数替换zmalloc_size函数（注意，未定义ZMALLOC_LIB，也不进行版本判断）


    #elif defined(__APPLE__)
    #include <malloc/malloc.h>
    #define HAVE_MALLOC_SIZE 1
    #define zmalloc_size(p) malloc_size(p)
    #endif


### 如果未定义ZMALLOC_LIB，ZMALLOC_LIB就定义为"libc"

    #ifndef ZMALLOC_LIB
    #define ZMALLOC_LIB "libc"
    #endif

### 函数声明

    void *zmalloc(size_t size);
    void *zcalloc(size_t size);
    void *zrealloc(void *ptr, size_t size);
    void zfree(void *ptr);
    char *zstrdup(const char *s);
    size_t zmalloc_used_memory(void);
    void zmalloc_enable_thread_safeness(void);
    void zmalloc_set_oom_handler(void (*oom_handler)(size_t));
    float zmalloc_get_fragmentation_ratio(void);
    size_t zmalloc_get_rss(void);
    size_t zmalloc_get_private_dirty(void);
    void zlibc_free(void *ptr);

### 如果不是上述三种，声明zmalloc_size函数，实现：如果使用了tcmalloc就用tcmalloc，如果使用了jemalloc，就用jemalloc，如果使用apple，就用malloc

    #ifndef HAVE_MALLOC_SIZE
    size_t zmalloc_size(void *ptr);
    #endif

### 头文件定义结束 

    #endif /* __ZMALLOC_H */
