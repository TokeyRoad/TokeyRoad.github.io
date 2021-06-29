---
title: Redis设计与实现(列表对象)
date: 2019-07-17 09:19:55
tags:
- Redis
- 列表对象
categories: 
- Redis设计与实现
copyright: true
---

列表对象的编码可以是 `ziplist` 或者 `linkedlist` 。

- `ziplist` 编码的列表对象使用压缩列表作为底层实现，每个压缩列表节点（entry）保存了一个列表元素。
- `linkedlist` 编码的列表对象使用双端链表作为底层实现，每个双端链表节点（node）都保存了一个字符串对象，而每个字符串对象都保存了一个列表元素。

>  注：linkedlist编码的列表对象在底层的双端链表结构中包含了多个字符串对象，这种嵌套字符串对象的行为在后续的哈希对象，集合对象，有序集合对象中都会出现，字符串对象是Redis五种类型的对象中唯一的一个会被嵌套的对象。

示例：

```Redis
redis> RPUSH numbers 1 "three" 5
(integer) 3
```

创建出一个列表对象 作为 `numbers` 键的值。

如果 `numbers`键的值对象使用的是 `ziplist` 编码，这个值对象结构如下：

![ListObject_1](Redis设计与实现-列表对象\ListObject_1.png)

如果 `numbers` 键的值对象使用的不是 `ziplist` 编码，而是 `linkedlist` 编码，值结构如下：

![ListObject_2](Redis设计与实现-列表对象\ListObject_2.png)

下图表示一个包含了字符串值 `“three”` 的字符串对象：

<!--more-->

简化表示：

![ListObject_3](Redis设计与实现-列表对象\ListObject_3.png)

实际结构：

![ListObject_4](Redis设计与实现-列表对象\ListObject_4.png)

#### 编码转换

当列表对象可以同时满足以下两个条件时，列表对象使用 `ziplist` 编码：

1. 列表对象保存的所有字符串元素的长度都小于 `64` 字节；
2. 列表对象保存的元素数量小于 `512` 个；

不能满足这两个条件的列表对象需要使用 `linkedlist` 编码。

> 两个上限值可以在配置文件中修改 `list-max-ziplist-value` 和 `list-max-ziplist-entries`选项。

对于使用 `ziplist` 编码的列表对象，当两个条件任意一个不满足时，对象的编码转换操作就会被执行；原本保存在压缩列表里的所有列表元素都会被转移并保存到双端链表里，对象的编码也会从 `ziplist` 变为 `linkedlist`。

#### 列表命令的实现

| 命令    | `ziplist` 编码的实现方法                                     | `linkedlist` 编码的实现方法                                  |
| ------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| LPUSH   | 调用 `ziplistPush` 函数，将新元素推入到压缩列表的表头。      | 调用 `listAddNodeHead` 函数，将新元素推入到双端链表的表头。  |
| RPUSH   | 调用 `ziplistPush` 函数，将新元素推入到压缩列表的末尾。      | 调用 `listAddNodeTail` 函数，将新元素推入到双端链表的表尾。  |
| LPOP    | 调用 `ziplistIndex` 函数定位压缩列表的表头节点，在向用户返回节点所保存的元素之后，调用 `ziplistDelete` 函数删除表头节点。 | 调用 `listFirst` 函数定位双端链表的表头节点，在向用户返回节点所保存的元素之后，调用 `listDelNode` 函数删除表头节点。 |
| RPOP    | 调用 `ziplistIndex` 函数定位压缩列表的表尾节点，在向用户返回节点所保存的元素之后，调用 `ziplistDelete` 函数删除表尾节点。 | 调用 `listLast` 函数定位双端链表的表尾节点，在向用户返回节点所保存的元素之后，调用 `listDelNode` 函数删除表尾节点。 |
| LINDEX  | 调用 `ziplistIndex` 函数定位压缩列表中的指定节点，然后返回节点所保存的元素。 | 调用 `listIndex` 函数定位双端链表中的指定节点，然后返回节点所保存的元素。 |
| LLEN    | 调用 `ziplistLen` 函数返回压缩列表的长度。                   | 调用 `listLength` 函数返回双端链表的长度。                   |
| LINSERT | 插入新节点到压缩列表的表头或者表尾是，使用 `ziplistPush` 函数；插入新节点到压缩列表其他位置时，使用 `ziplistInsert` 函数。 | 调用 `listInsertNode` 函数，将新节点插入到双端链表的指定位置。 |
| LREM    | 遍历压缩列表节点，并调用 `ziplistDelete` 函数删除包含了指定元素的节点。 | 遍历双端链表节点，并调用 `listDelNode` 函数删除包含了指定元素的节点。 |
| LTRIM   | 调用 `ziplistDeleteRange` 函数，删除压缩列表中所有不在指定索引范围内的节点。 | 遍历双端链表节点，并调用 `listDelNode` 函数删除链表中所有不在指定索引范围内的节点。 |
| LSET    | 调用 `ziplistDelete` 函数，先删除压缩列表指定索引上的现有节点，然后调用 `ziplistInsert` 函数，将一个包含给定元素的新节点插入到相同索引上。 | 调用 `listIndex` 函数，定位到双端链表指定索引上的节点，通过赋值操作更新节点的值。 |

