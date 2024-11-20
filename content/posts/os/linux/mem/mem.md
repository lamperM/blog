在`free_initmem()` 之后，`totalram_pages()`就固定在一个值。这也是 Linux 可以支配的内存，这之外的内存称之为内存黑洞。

分析从 RAM 启动，到 free_initmem()，中间是怎么对内存进行处理的。

memblock 内存管理机制用于在 Linux 启动后管理内存，一直到 free_initmem()为止。之后 totalram_pages 就稳定在一个数值。

其中对不同类型 memblock 的分配释放主要有如下：

1. 其中 memblock_add()和 memblock_remove()是针对可用 memlbock 操作，即 `memblock.memory`
2. memblock_reserve()和 memblock_free()是针对 reserved 类型 memblock 操作，即 `memblock.reserved`

```c
int __init_memblock memblock_add_node(phys_addr_t base, phys_addr_t size,
				       int nid)
{
	return memblock_add_range(&memblock.memory, base, size, nid, 0);
}

int __init_memblock memblock_add(phys_addr_t base, phys_addr_t size)
{
	return memblock_add_range(&memblock.memory, base, size, MAX_NUMNODES, 0);
}

int __init_memblock memblock_free(phys_addr_t base, phys_addr_t size)
{
	kmemleak_free_part_phys(base, size);
	return memblock_remove_range(&memblock.reserved, base, size);
}

int __init_memblock memblock_reserve(phys_addr_t base, phys_addr_t size)
{
	return memblock_add_range(&memblock.reserved, base, size, MAX_NUMNODES, 0);
}
```

可用的 memblock 有些什么东西，reserved 类型的 memblock 又有些什么东西

TODO：有环境的情况下，用打开调试选项试一下比看代码直观一些，另外还有包括`free -h`的含义这些。

```c
/* lowest address */
phys_addr_t __init_memblock memblock_start_of_DRAM(void)
{
	return memblock.memory.regions[0].base;
}

phys_addr_t __init_memblock memblock_end_of_DRAM(void)
{
	int idx = memblock.memory.cnt - 1;

	return (memblock.memory.regions[idx].base + memblock.memory.regions[idx].size);
}
```

## 参考

[Linux 内存都去哪了：(1)分析 memblock 在启动过程中对内存的影响 - ArnoldLu - 博客园](https://www.cnblogs.com/arnoldlu/p/10526814.html)
