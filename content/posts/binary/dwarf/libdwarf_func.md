---
title: "Libdwarf 函数介绍及其实现方法"
tags: ["Dwarf"]
categories: ["Dwarf"]
date: 2023-05-07T14:51:49+08:00
---
## Function

### dwarf_alloc.c

#### _dwarf_get_alloc()
```c
char *
_dwarf_get_alloc(Dwarf_Debug dbg,
    Dwarf_Small alloc_type, Dwarf_Unsigned count)
```
函数功能: 根据类型申请一块空间

注意, 申请时大小会多`DW_RESERVE`, 此函数返回的地址是 `mem_alloc()`返回的地址+`DW_RESERVE`
#### _dwarf_error()
```c
void
_dwarf_error(Dwarf_Debug dbg, Dwarf_Error * error, Dwarf_Sword errval)
```

#### 函数功能
错误处理的函数

#### 函数流程
1. 判断传入参数`error`是否为空, 如果为空则跳转到 step 4.
2. 分配一个新的`Dwarf_Error`空间
3. 将传入的错误信息存入`Dwarf_Error`, 并将其作为指针返回
4. `error`为空时, 一般要在出错时执行一些方法,即 `dbg->de_errhand()`
5. 所以, 此时`dbg` 和 `dbg->de_errhand` 必须非空.



### dwarf_elf_access.c
#### dwarf_elf_object_access_init()
```c
int         
dwarf_elf_object_access_init(u64 elf_base_addr,
    int libdwarf_owns_elf,                     
    Dwarf_Obj_Access_Interface** ret_obj,      
    int *err)                      
```
#### 函数功能
初始化 `Dwarf_Obj_Access_Interface` 结构

#### 函数流程
1. 初始化其成员`object`, 使用的是`dwarf_elf_object_access_internals_init`方法
2. 申请空间
3. 为成员赋值
4. 为全局变量`_dwarf_get_elf_flags_func_ptr`赋值, 为获取elf section flags的方法

#### _dwarf_get_elf_flags_func()
```c
static int                        
_dwarf_get_elf_flags_func(        
    void* obj_in,                 
    Dwarf_Half section_index,     
    Dwarf_Unsigned *flags_out,    
    Dwarf_Unsigned *addralign_out,
    int *error)                   
```

#### 函数功能
获取elf指定section的flags



### dwarf_original_elf_init.c
#### dwarf_elf_object_access_init()
```c
static int      
dwarf_elf_init_file_ownership(u64 elf_base_addr, 
    int libdwarf_owns_elf,          
    Dwarf_Unsigned access,          
    Dwarf_Handler errhand,          
    Dwarf_Ptr errarg,               
    Dwarf_Debug * ret_dbg,          
    Dwarf_Error * error)               
```


### dwarf_init_finish.c
#### this_section_dwarf_relevant()
```c
static int
this_section_dwarf_relevant(const char *scn_name,int type)
```
函数功能: 通过section name判断该section是否与dwarf相关

#### get_basic_section_data()
```c
static int
get_basic_section_data(Dwarf_Debug dbg,
    struct Dwarf_Section_s *secdata,
    struct Dwarf_Obj_Access_Section_s *doas,
    Dwarf_Half section_index,
    Dwarf_Error* error,
    int duperr, int emptyerr )
```
函数功能: 填充 `Dwarf_Section_s` 中一些基本信息, 不包含`dss_data`, 即section load不在这个函数完成


#### _dwarf_load_section()
```c
int
_dwarf_load_section(Dwarf_Debug dbg,
    struct Dwarf_Section_s *section,
    Dwarf_Error * error)
```
函数功能: 加载一个ELF section, 填充对应的`struct Dwarf_Section_s`. 其caller一般为`_dwarf_load_debug_xxx()`, 例如`_dwarf_load_debug_info()`.

函数流程:
1. 检查这个段是否已经被加载了
2. 调用`methods->load_section()` 加载section的dss_data
3. 只需要赋值`dss_data`就行, 因为其他的成员在`get_basic_section_data()`时候已经完成了

#### add_debug_section_info()
```c
static int
add_debug_section_info(Dwarf_Debug dbg, 
    /* Name as seen in object file. */
    const char *name,
    unsigned obj_sec_num,
    struct Dwarf_Section_s *secdata,
    /*  The have_dwarf flag is a somewhat imprecise
        way to determine if there is at least one 'meaningful'
        DWARF information section present in the object file.
        If not set on some section we claim (later) that there
        is no DWARF info present. see 'foundDwarf' in this file */
    int duperr,int emptyerr,int have_dwarf,
    int havezdebug,
    int *err)
```
函数功能: 已经确定好某个section是debug相关section的前提下, 填充`dbg->de_debug_sections[]`


#### ★_dwarf_setup() 
```c
static int                                        
_dwarf_setup(Dwarf_Debug dbg, Dwarf_Error * error)
```
函数功能: 填充dbg中与debug section相关的所有信息

函数流程:
1. 填充dbg中一些字段, 无关紧要
2. 申请所有section的 `struct Dwarf_Section_s`
3. 循环elf中所有的section, 每次index+1
4. 调用`get_section_info()`, 获取该段的信息, 存入`struct Dwarf_Obj_Access_Section_s`
5. 通过名称对比此段是否与dwarf相关, 若非dwarf相关段, 则返回step3 继续解析下一个段头
6. 若属于dwarf相关段, 则将其相关信息添加到`dbg->debug_sections[]`数组中
7. 返回step3进行下一段的初始化



### dwarf_die_deliv.c
#### dwarf_next_cu_header_d()
```c
int
dwarf_next_cu_header_d(Dwarf_Debug dbg,
    Dwarf_Bool is_info,
    Dwarf_Unsigned * cu_header_length,   // Returns the length of the just-read CU header.
    Dwarf_Half * version_stamp, // Returns the version number (2 to 5) of the CU header
    Dwarf_Unsigned * abbrev_offset, // Returns the .debug_abbrev offset from the the CU header
    Dwarf_Half * address_size, // Returns the address size specified for this CU, usually either 4 or 8.
    Dwarf_Half * offset_size, // Returns the offset size (the length of the size field from the header) specified for this CU
    Dwarf_Half * extension_size,  // If the section is standard 64bit DWARF then this value is 4. Else the value is zero.
    Dwarf_Sig8 * signature,
    Dwarf_Unsigned * typeoffset,
    Dwarf_Unsigned * next_cu_offset,
    Dwarf_Half * header_cu_type, // Returns DW_UT_compile, or other DW_UT value
    Dwarf_Error * error);
```
函数功能: return information on next CU header

函数流程:
1. 计算下一个CU在.debug_info section的offset
2. 检查下一个CU是否之前被解析过
3. 如果是第一次解析这个cu, 构造一个`Dwarf_CU_Context`, 链接到`Dwarf_Debug_InfoTypes`中
4. 填充返回参数


### _dwarf_next_die_info_ptr()
```c
static int
_dwarf_next_die_info_ptr(Dwarf_Byte_Ptr die_info_ptr, // start of the DIE
    Dwarf_CU_Context cu_context,
    Dwarf_Byte_Ptr die_info_end,   // end of CU the die belong to
    Dwarf_Byte_Ptr cu_info_start,  // start of CU 
    Dwarf_Bool want_AT_sibling,
    Dwarf_Bool * has_die_child,
    Dwarf_Byte_Ptr *next_die_ptr_out,
    Dwarf_Error *error)
```
函数功能: 这个函数功能分为两部分, 取决于传入参数`want_AT_sibling`
- if true, 且DIE有`AT_sibling`属性, 则其返回指向sibling DIE开始位置的指针. 如果没有属性, 则表现的行为像`false`
- if false, 返回指向下一个紧邻的DIE的指针?

true时的函数流程:
1. 解析出传入DIE的attr和attrform, while解析到`attr=AT_sibling`停止
2. 再根据attrform解析出attr的value, 即 sibling 的 offset
3. cu_info_start + offset 即为 sibling 的开始



### dwarf_siblingof_b()
```c
int
dwarf_siblingof_b(Dwarf_Debug dbg,
    Dwarf_Die die,
    Dwarf_Bool is_info,
    Dwarf_Die * caller_ret_die, Dwarf_Error * error)
{
```
函数功能: 
- 首先这个函数的功能分为两部分, 如果传入参数`die=NULL`, 那么函数的功能是返回当前读取cu的DIE_cu; 
- 如果`die!=NULL`, 那么函数的功能是返回指定die的sibling DIE


在`die!=NULL`时, 函数流程为:
1. 调用 `_dwarf_next_die_info_ptr` 得到 sibling的 global offset
2. 申请一个新的 `Dwarf_Die`, 填充相应字段



### dwarf_offdie()
```c
int
dwarf_offdie(Dwarf_Debug dbg,
    Dwarf_Off offset, Dwarf_Die * new_die, Dwarf_Error * error)
```
函数功能: 给定一个global offset(不是相对于cu的), 返回其对应的`Dwarf_Die`描述体.由caller 负责调用`dwarf_dealloc()`释放空间










## Structure

### Dwarf_Error

```c
typedef struct Dwarf_Error_s*      Dwarf_Error;
```

```c
struct Dwarf_Error_s {
    /* 等效于error */
    Dwarf_Sword er_errval;

    /*  If non-zero the Dwarf_Error_s struct is not malloc'd.
        To aid when malloc returns NULL.
        If zero a normal dwarf_dealloc will work.
        er_static_alloc only accessed by dwarf_alloc.c.

        If er_static_alloc is 1 in a Dwarf_Error_s
        struct (set by libdwarf) and client code accidentally
        turns that 0 to zero through a wild
        pointer reference (the field is hidden
        from clients...) then chaos will
        eventually follow.
    */
    int er_static_alloc;
};
```


### struct Dwarf_Obj_Access_Section_s
Used in the get_section interface function

```c
struct Dwarf_Obj_Access_Section_s {                                        
    /*  addr is the virtual address of the first byte of                   
        the section data.  Usually zero when the address                   
        makes no sense for a given section. */                             
    Dwarf_Addr     addr;                                                   
                                                                           
    /* Section type. */                                                    
    Dwarf_Unsigned type;                                                   
                                                                           
    /* Size in bytes of the section. */                                    
    Dwarf_Unsigned size;                                                   
                                                                           
    /*  Having an accurate section name makes debugging of libdwarf easier.
        and is essential to find the .debug_ sections.  */                 
    const char*    name;                                                   
    /*  Set link to zero if it is meaningless.  If non-zero                
        it should be a link to a rela section or from symtab               
        to strtab.  In Elf it is sh_link. */                               
    Dwarf_Unsigned link;                                                   
                                                                           
    /*  The section header index of the section to which the               
        relocation applies. In Elf it is sh_info. */                       
    Dwarf_Unsigned info;                                                   
                                                                           
    /*  Elf sections that are tables have a non-zero entrysize so          
        the count of entries can be calculated even without                
        the right structure definition. If your object format              
        does not have this data leave this zero. */                        
    Dwarf_Unsigned entrysize;                                              
};                                
```

### struct Dwarf_Section_s
将很多对section相关的描述信息结合到一起

- `dss_data` 指向这个section的起始地址

```c
struct Dwarf_Section_s {
    Dwarf_Small *  dss_data;
    Dwarf_Unsigned dss_size;
  
    /*  dss_index is the section index as things are numbered in
        an object file being read.   An Elf section number. */
    Dwarf_Word     dss_index;
    /*  dss_addr is the 'section address' which is only
        non-zero for a GNU eh section.
        Purpose: to handle DW_EH_PE_pcrel encoding. Leaving
        it zero is fine for non-elf.  */
    Dwarf_Addr     dss_addr;
    Dwarf_Small    dss_data_was_malloc;
    /*  is_in_use set during initial object reading to
        detect duplicates. Ignored after setup done. */
    Dwarf_Small    dss_is_in_use;

    /*  If this is zdebug, to start  data/size are the
        raw section bytes.
        Initially for all sections dss_data_was_malloc set FALSE
            and dss_requires_decompress set FALSE.
        For zdebug dss_requires_decompress then set TRUE

        On translation (ie zlib use and malloc)
        Set dss_data dss_size to point to malloc space and
            malloc size.
        Set dss_requires_decompress FALSE
        Set dss_was_malloc  TRUE */
    Dwarf_Small    dss_requires_decompress;

    /*  For non-elf, leaving the following fields zero
        will mean they are ignored. */
    /*  dss_link should be zero unless a section has a link
        to another (sh_link).  Used to access relocation data for
        a section (and for symtab section, access its strtab). */
    Dwarf_Word     dss_link;
    /*  The following is used when reading .rela sections
        (such sections appear in some .o files). */
    Dwarf_Half     dss_reloc_index; /* Zero means ignore the reloc fields. */
    Dwarf_Small *  dss_reloc_data;
    Dwarf_Unsigned dss_reloc_size;
    Dwarf_Unsigned dss_reloc_entrysize;
    Dwarf_Addr     dss_reloc_addr;
    /*  dss_reloc_symtab is the sh_link of a .rela to its .symtab, leave
        it 0 if non-meaningful. */
    Dwarf_Addr     dss_reloc_symtab;
    /*  dss_reloc_link should be zero unless a reloc section has a link
        to another (sh_link).  Used to access the symtab for relocations
        a section. */
    Dwarf_Word     dss_reloc_link;
    /*  Pointer to the elf symtab, used for elf .rela. Leave it 0
        if not relevant. */
    struct Dwarf_Section_s *dss_symtab;
    /*  dss_name must never be freed, it is a quoted string
        in libdwarf. */
    const char * dss_name;

    /* Object section number in object file. */
    unsigned dss_number;
    
    /*  These are elf flags and non-elf object should
        just leave these fields zero. Which is essentially
        automatic as they are not in
        Dwarf_Obj_Access_Section_s.  */
    Dwarf_Word  dss_flags;
    Dwarf_Word  dss_addralign;
};  
```

### struct Dwarf_dbg_sect_s
描述一个debug相关的section

```c
struct Dwarf_dbg_sect_s {
    /*  Debug section name must not be freed, is quoted string.
        This is the name from the object file itself. */
    const char *ds_name;
    /* The section number in object section numbering. */
    unsigned ds_number;
    /*   Debug section information, points to de_debug_*member
        (or the like) of the dbg struct.  */
    struct Dwarf_Section_s *ds_secdata;

    int ds_duperr;                     /* Error code for duplicated section */
    int ds_emptyerr;                   /* Error code for empty section */
    int ds_have_dwarf;                 /* Section contains DWARF */
    int ds_have_zdebug;                /* Section compressed. */
};
```


### Dwarf_CU_Context

```c
typedef struct Dwarf_CU_Context_s *Dwarf_CU_Context;
```

描述一个CU的内容.

包含描述 CU header的成员:
- cc_length
- cc_version_stamp
- cc_abbrev_offset
- cc_address_size

`cc_debug_offset` 是该CU header在section中的offset

`cc_dbg` 是该CU所属的dbg

### Dwarf_Debug_InfoTypes

```c
typedef struct Dwarf_Debug_InfoTypes_s *Dwarf_Debug_InfoTypes;
```

是一个将.debug_info中所有CU组织起来的数据结构

- `de_cu_context` 指向刚刚读取到的CU
- `de_cu_context_list` .debug_info的所有CU链表
- `de_cu_context_list_end` CU链表的结尾
- ...


### Dwarf_Die

```c
typedef struct Dwarf_Die_s*        Dwarf_Die;
```

描述一个DIE
- `di_debug_ptr` DIE在section中的offset
- `di_cu_context` DIE所在的cu


### Dwarf_Abbrev_List

```c
typedef struct Dwarf_Abbrev_List_s *Dwarf_Abbrev_List;
```

描述每个缩写
- `abl_code`  abbreviation code, unsigned LEB128编码
- `abl_tag` unsigned LEB128编码, 对应DIE的tag
- `abl_has_child` 指代此DIE是否有子项
- `abl_abbrev_ptr` Points to start of attribute and form pairs in the .debug_abbrev




## 其他联动方式
### struct Dwarf_dbg_sect_s 和 struct Dwarf_Section_s 之间的关系
`Dwarf_dbg_sect_s` 是外层, 其成员 `ds_secdata`指向一个`Dwarf_Section_s`.

除此之外, 它还包含, 描述section number, 是否包含dwarf信息等成员.


### Dwarf_Debug 和 其拥有的 struct Dwarf_dbg_sect_s之间的关系
Dwarf_Debug 中的成员`de_debug_sections`记录了加载的所有debug相关section, 成员`de_debug_sections_total_entries` 代表加载的section数量.

同时, Dwarf_Debug 的成员 `de_debug_xxx` 是 `struct Dwarf_Section_s` 类型的, 按照名称记录各个section, 方便直接访问, 不需要遍历 `de_debug_sections` 来查询

### 动态申请的空间何时释放?
我们经常会用到例如 `dwarf_offdie()`, `dwarf_attr()` 等接口来返回一个特定的dwarf相关结构. 在其中都调用了 `_dwarf_get_alloc()` 使用操作系统提供的动态内存分配算法申请一块空间, 那么是否需要我们来控制释放这些空间呢?

某些API的描述中说, 只要是使用`_dwarf_get_alloc`使用的空间, 必须由用户显式调用`dwarf_dealloc()` 释放. 在这个版本的libdwarf中, 这个过程其实是不必要的.

因为在`_dwarf_get_alloc()`中其实会将申请的空间记录在 Dwarf_Debug的成员de_alloc_tree中. 所以在`dwarf_finish()`中会遍历树, 将这些空间依次释放, 所以其实不显式的调用`dwarf_dealloc()`也不会造成内存泄漏.