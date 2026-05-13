# Memory module

##  文件

memtypes.c	memtypes.h	memory.h	memory.c

## 内存管理

结构体：

```c
static struct 
{
  char *name;
  long alloc;				// 记录各类内存区域创建的数量
} mstat [MTYPE_MAX];
```

自定义内存类型：

```c
enum
{
  MTYPE_TMP = 1,
  MTYPE_STRVEC,
  MTYPE_VECTOR,
  MTYPE_VECTOR_INDEX,
  MTYPE_LINK_LIST,
  MTYPE_LINK_NODE,
  ......
  ......
  MTYPE_MAX,
};
```

关键函数原型:

```c
void *zmalloc(int type, size_t size);					// 使用malloc

void *zzcalloc(int type, size_t size);					// 使用calloc

void *zrealloc(int type, void *ptr, size_t size);		// 使用realloc重新分配ptr执行的内存区域

void zfree(int type, void *ptr);						// 使用fres

char *zstrdup(int type, const char *str);				// 使用strdup
    
static void alloc_inc(int type);						// mstat[type]的数量 + 1

static void alloc_dec(int type);						// mstat[type]的数量 - 1
```

关键宏定义：

```c
#define XMALLOC(mtype, size)       zmalloc ((mtype), (size))
#define XCALLOC(mtype, size)       zzcalloc ((mtype), (size))
#define XREALLOC(mtype, ptr, size) zrealloc ((mtype), (ptr), (size))
#define XFREE(mtype, ptr)          do { \
                                     zfree ((mtype), (ptr)); \
                                     (ptr) = NULL; } \
                                   while (0)
#define XSTRDUP(mtype, str)        zstrdup ((mtype), (str))
```



> /* malloc	calloc	realloc */
>
> - malloc
>
>   malloc函数用于申请一块指定大小的内存区域。
>
>   函数原型：void *malloc(size_t size);	其中，size是要分配的内存大小，单位是字节
>
>   malloc分配的内存区域中的初始值是不确定的，可能包含随机数据。
>
> - calloc
>
>   calloc函数也用于申请内存，但它会初始化所分配的内存区域。
>
>   函数原型：void *calloc(size_t count, size_t size);	其中，count是要分配的元素数量，size是每个元素的大小。
>
>   calloc分配的内存区域会初始化为0。
>
> - realloc
>
>   realloc用于重新分配内存空间。
>
>   函数原型：void *realloc(void *ptr, size_t size);	其中，ptr指向之前所分配的内存块，若是空指针，则会分配一个新的内存块；size是内存块新的大小，以字节为单位。若ptr指向一个已存在的内存块，size为0，则会释放ptr所指向的内存，并返回空指针。
>
>   

## 内存信息打印



# Vector module

## 文件

vector.h	vector.c

## 结构体

```c
struct _vector 
{
  unsigned int active;		/* number of active slots */
  unsigned int alloced;		/* number of allocated slot */
  void **index;			    /* index to data */
};
typedef struct _vector *vector;
```

## 函数原型

```c
/* XCALLOC分配一个_vector大小的内存空间，并XCALLOC分配一个size * sizeof(void *)大小的内存，由index管理 */
vector vector_init(unsigned int size);

/* 释放内存，仅仅释放分配的struct _vector *类型的内存空间 */
void vector_only_wrapper_free(vector v);

/* 释放内存，仅仅释放index管理的内存 */
void vector_only_index_free(void *index);

/* 同时进行wrapper_free和index_free */
void vector_free (vector v);

/* 拷贝一份新的vector */
vector vector_copy(vector v);

/* 确保v中index分配的大小大于num,否则两倍扩容 */
void vector_ensure (vector v, unsigned int num);

/* 查找v中index里索引最小的一个空slot */
int vector_empty_slot (vector v);

/* 设置索引最小空slot的值 */
int vector_set(vector v, void *val);

/* 设置特定索引slot中的值 */
int vector_set_index (vector v, unsigned int i, void *val);

/* 查找索引为i的slot中的值 */
void *vector_lookup(vector v, unsigned int i);

/* 查找索引为i的slot中的值，并确保v的alloced大于i */
void *vector_lookup_ensure(vector v, unsigned int i);

/* 复原指定索引的slot */
void vector_unset(vector v, unsigned int i);

/* 计算非空slot的数量 */
unsigned int vector_count(vector v);
```

宏定义：

````c
#define vector_slot(V,I)  	((V)->index[(I)])
#define vector_active(V) 	((V)->active)
````

# Stream module

## 文件



# Workqueue module

# Buffer module

# Commond Module
## 文件
command.h和command.c
## 结构体
### struct cmd_node
struct cmd_node 是 Quagga VTY 命令系统中的命令节点结构体，用于表示一个独立的配置模式或命令上下文。
字段详解
| 字段 | 类型 | 说明 |
| :------: | :-------: | :------: |
| node | enum node_type | 节点索引，标识节点类型，如（CONFIG_NODE、OSPF_NODE、BGP_NODE等） |
| prompt | const char * | VTY界面的提示符字符串，如“quagga(config-router)#" |
| vtysh | int | 是否将配置写入vtysh（统一shell），1标识写入，0表示不写入 |
| func | int (*)(struct vty *) | 函数指针，用于将节点配置写入配置文件 |
| cmd_vector | vector | 该节点的所有命令列表（向量结构） |
| cmd_hash | struct hash * | 命令的哈希索引，用于去重和提高查找效率 |

使用示例
```c
// 在 ospfd/ospf_vty.c 中
struct cmd_node ospf_node =
{
  OSPF_NODE,                // node: 节点类型
  "%s(config-router)# ",    // prompt: 提示符
  1,                        // vtysh: 写入配置
  ospf_config_write,        // func: 配置写入函数
  NULL,                     // cmd_vector: 命令向量（初始化时为空）
  NULL,                     // cmd_hash: 哈希表（初始化时为空）
};
```
节点层次关系

VIEW_NODE (用户视图)
    ↓
ENABLE_NODE (特权模式)
    ↓
CONFIG_NODE (全局配置模式)
    ├── OSPF_NODE (OSPF配置)
    ├── BGP_NODE (BGP配置)
    ├── INTERFACE_NODE (接口配置)
    ├── RIP_NODE (RIP配置)
    └── ...

每个 cmd_node 代表一个独立的配置上下文，用户通过 router ospf 等命令在不同节点间切换：
- prompt 字段决定了切换后的提示符
- func 负责将该节点的配置写入文件
- cmd_vector 存储该节点下所有可用的命令

### struct cmd_element结构体详解
struct cmd_element 是 Quagga VTY 命令系统中的命令元素结构体，用于描述一个具体的 CLI 命令。

#### 字段详解

| 字段 | 类型 | 说明 |
| :------: | :-------: | :------: |
| string | const char * | 命令规范字符串，定义命令语法(如"router ospf") |
| func | int (*)(...) | 命令处理函数指针，执行命令时的回调函数 |
| doc | const char * | 命令的帮助文档字符串 |
| daemon | int | 命令所属的守护进程标识（哪个协议使用） |
| tokens | 	vector | 命令的词法标记向量，用于解析命令 |
| attr | u_char | 命令属性（隐藏、弃用等）|

#### 使用示例
以 router ospf 命令为例：
```c
// 通过 DEFUN 宏展开后生成
struct cmd_element router_ospf_cmd =
{
  .string = "router ospf",                    // 命令字符串
  .func = router_ospf,                        // 处理函数
  .doc = "Enable a routing process\n"         // 帮助文档
       "Start OSPF configuration\n",
  .attr = 0,                                  // 无特殊属性
  .daemon = 0,                                // OSPF 守护进程
};
```
#### 与DEFUN宏的关系

DEFUN 宏内部使用 DEFUN_CMD_ELEMENT 来创建 cmd_element：
```c
#define DEFUN_CMD_ELEMENT(funcname, cmdname, cmdstr, helpstr, attrs, dnum) \
  struct cmd_element cmdname = \
  { \
    .string = cmdstr, \
    .func = funcname, \
    .doc = helpstr, \
    .attr = attrs, \
    .daemon = dnum, \
  };
```
#### 命令注册
定义后通过 install_element() 安装到指定节点：
```c
install_element (CONFIG_NODE, &router_ospf_cmd);
```
这将命令注册到 CONFIG_NODE（全局配置模式），用户在该模式下可以执行此命令。

#### 命令属性
通过 attr 字段设置：
```c
enum
{
  CMD_ATTR_DEPRECATED = 1,  // 弃用
  CMD_ATTR_HIDDEN,          // 隐藏
};
```
总结：cmd_element 是 VTY 命令系统的核心数据结构，每个命令对应一个 cmd_element 实例，包含命令的语法定义、处理函数、帮助文档和所属守护进程等信息。

### struct cmd_token结构体详解
struct cmd_token 是 Quagga VTY 命令系统中的命令标记结构体，用于表示命令字符串解析后的单个词法单元（token）。

#### 字段详解
| 字段 | 类型 | 说明 |
| :------: | :-------: | :------: |
| type | enum cmd_token_type | 标记类型（TERMINAL、MULTIPLE、KEYWORD） |
| terminal | enum cmd_terminal_type | 终端类型（字面量、可选、变量、范围、IP等） |
| multiple | vector | MULTIPLE 类型时使用，存储多个候选标记 |
| keyword | vector | KEYWORD 类型时使用，存储关键字选项 |
| cmd | char * | 命令字符串（TERMINAL 类型使用）|
| desc | char * | 命令描述/帮助文本）|

#### 标记类型枚举
cmd_token_type（标记类型）
```c
enum cmd_token_type
{
  TOKEN_TERMINAL = 0,   // 基础终端标记
  TOKEN_MULTIPLE,       // 多选一标记 (reject|blackhole)
  TOKEN_KEYWORD,        // 关键字标记 {always|metric|route-map}
};
```
cmd_terminal_type（终端类型）
```c
enum cmd_terminal_type
{
  _TERMINAL_BUG = 0,
  TERMINAL_LITERAL,      // 字面量（固定字符串）
  TERMINAL_OPTION,       // 可选参数 [IFNAME]
  TERMINAL_VARIABLE,     // 变量 <0-16777214>
  TERMINAL_VARARG,       // 可变参数 .LINE
  TERMINAL_RANGE,        // 范围 <1-65535>
  TERMINAL_IPV4,         // IPv4 地址 A.B.C.D
  TERMINAL_IPV4_PREFIX,  // IPv4 前缀 A.B.C.D/M
  TERMINAL_IPV6,         // IPv6 地址 X:X::X:X
  TERMINAL_IPV6_PREFIX,  // IPv6 前缀 X:X::X:X/M
};
```





