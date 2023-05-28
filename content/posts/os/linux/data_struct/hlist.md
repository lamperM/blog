---
title: "Linux 内核数据结构 hlist"
tags: ["c", "linux"]
date: 2023-05-11T20:51:49+08:00
---

linux 内核为创建【用单链表解决冲突的哈希表】设计了专门的数据结构 hlist。

hlist 整体来说是带头结点的双向链表，头结点的类型为`hlist_head`, 普通节点
的类型为`hlist_node`. **为什么要区别两种类型？节约空间**， 因为哈希表的
表项类型可以是`hlist_head`, 它其实不需要`prev`指针, 比起一般的结点，一个
哈希表能节约一半的空间。

所以一个哈希表和头结点的结构可表示为:

```c
struct hlist_head {
    struct hlist_node *first;
};
struct hlist_head table[TALBE_SZ];
```

## 二象性

任何事物都具有二象性，区分两种类型节约空间的空间，也带了一个问题：
**首个`hlist_node`结点的`prev`指向哪呢？**

正常情况下肯定毫不犹豫的指向头结点，即`hlist_head`，但注意此时类型是
不同的，`prev`不能同时是`struct hlist_head*`和`struct hlist_node *`。

解决方案有两个，首先可以使首个结点的`prev=NULL`, 这样虽然避免了类型引发的
问题，也能保证功能正确，但是却破坏了一致性，使得操作的复杂度上升，增加了许多
判断分支。

```c
// delelt a node
void del_node(struct hlist_head *head, struct hlist_node *node)
{
    // 这个if 本来是不需要的，甚至参数的head 也不需要传，
    // 更好的处理方式见解决方案2
    if (node == head->first) {
        head->first = node->next;
    }
    else {
        node->prev->next = node->next;
    }

    if (node->next) {
        node->next->prev = node->prev;
    }
}
// insert a node
void add_node_before(struct hlist_head *head, struct hlist_node *new
                        struct hlist_node *next)
{
    // 这个if 本来是不需要的，参数head也是不需要传递的
    if (next == head->first) {
        new->prev = NULL;
        head->first = new;
    }
    else {
        new->prev = next->prev;
        new->prev->next = new;
    }
    new->next = next;
    next->prev = new;
```

## 更好的解决方案: `**prev`

改变`struct hlist_node`的构成，使用二级指针:

```c
struct hlist_node {
    struct hlist_node *next;
    struct hlist_node **pprev;
};
```

使得每个结点的`pprev = &(prev_node->next)`, 首先类型是统一的，其次删除和添加
都无需额外的分支了。

```c
void del_node(struct hlist_node *node)
{
    *(node->pprev) = node->next;
    if (node->next)
        node->next->pprev = node->pprev;
}
void add_node(struct hlist_node *new, struct hlist_node *next)
{
    new->pprev = next->pprev;
    *(new->pprev) = new;
    new->next = next;
    next->pprev = &(new->next);
}
```

Ref:

- [`**prev` 可以提高删除的效率](https://zhuanlan.zhihu.com/p/360217911)
- stackoverflow
