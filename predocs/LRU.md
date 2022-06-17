## leetcode-146：LRU 缓存机制

```bash
Question：运用你所掌握的数据结构，设计和实现一个  LRU (最近最少使用) 缓存机制 （先点进来回顾一下这是个啥~）
```

首先接收一个`capacity`参数作为缓存的最大容量
**get(key) API**：如果关键字 ( key ) 存在于缓存中，则获取关键字的值（总是正数），否则返回 -1 。
**put(key, value) API**：如果关键字已经存在，则变更其数据值；如果关键字不存在，则写入该数据。当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据，从而为新数据留出空间。
在 O(1)时间复杂度内完成上面的两个操作

示例：

```bash
输入
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]
输出
[null, null, null, 1, null, -1, null, -1, 3, 4]
```

先说一下理解：

设置一个缓存容量 2
写入数据（key, value）时：需要写入的数据的 key 已经存在，则更新它的 value，同时更新数据的位置；不存在，如果缓存容量还有，就直接写入，容量不足就把最久没有使用的数据删除，将新数据写入该空间。
读取数据（key）时：只要数据被读取成功，代表这个数据最近使用了，我们将它移动到头部。
题目要求上面的写入和读取操作的时间复杂度都为 O(1)，注意这里其实要求了查找、插入、删除、排序四个操作。
下面举例做个解释：

```js
/*设置缓存容量为 2*/
LRUCache cache = new LRUCache(2);
// 将 cache 看成成一个队列，左头右尾
// 这里圆括号表示键值对 (key, val)

cache.put(1, 1);  // cache = [(1, 1)]
cache.put(2, 2);  // cache = [(2, 2), (1, 1)]
cache.get(1);       // 返回键 1 对应的值 1
// 因为最近访问了键 1，所以提前至队头（最近使用） 所以此时 cache = [(1, 1), (2, 2)]
cache.put(3, 3);  // cache = [(3, 3), (1, 1)]
// 上一步缓存容量就已经满了，需要删除最久没有使用的内容空出位置存新数据（队尾的数据）
// 然后把新的数据插入队头
cache.get(2);       // cache 中不存在键为 2 的数据，返回 -1  此时不变 cache = [(3, 3), (1, 1)]
cache.put(1, 4);  // cache = [(1, 4), (3, 3)]
// 键 1 已存在，只需要把原始值 1 覆盖为 4。同时将键值对提前到队头（最近使用）
```

**note**：利用对象可以获得全部键值对，但新加入的（key，value）不是直接放到最后，而是按码值排序，因此会一直按如 ‘0’ '1' '2' 形式排序。而哈希 map 的键值对是有序的。

### Method 1：哈希表 + 双向链表（原生 JS 实现）

那么，如何设计满足要求的数据结构呢？（盯住上面分析出来的四个操作）

时间复杂度为 O(1)，且能够快速查找到 => 哈希表
但是哈希表是无序的，我们需要知道缓存里数据的顺序（队头为最近使用，队尾为最久使用）=> 链表，有顺序，而且插入删除快，但是查找慢
所以，我们使用 哈希表 + 链表 。那具体应该用什么链表呢？

- 单向链表：删除节点需要访问前驱节点，只能从前遍历查找，时间复杂度为 O(n)，不满足要求。 => 我们需要能够快速访问到前驱节点
- 双向链表：节点有前驱指针，删除和移动节点都是指针的变动，时间复杂度都为 O(1)，满足要求。

最终选择：哈希表 + 双向链表

- 双向链表：存数据的键值对（key，value）
- 哈希表：快速访问到双向链表中的数据

开始代码实现吧~~~~

```js
// 创建节点类
class ListNode {
  constructor(key, value) {
    this.key = key;
    this.value = value;
    this.prev = null;
    this.next = null;
  }
}
// 定义 LRUCache
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity; // 缓存的容量
    this.hash = {}; // 哈希表
    this.count = 0; // 缓存数目
    this.dummyHead = new ListNode(); // 虚拟头结点
    this.dummyTail = new ListNode(); // 虚拟尾节点
    // 初始化虚拟头尾节点的联系
    this.dummyHead.next = this.dummyTail;
    this.dummyTail.prev = this.dummyHead;
  }
  // get 方法
  get(key) {
    let node = this.hash[key]; // 从哈希表中获取对应的节点
    if (node === null) return -1; // 如果哈希表中没有该节点，返回-1
    // 将该节点移动到头部（标志着最近使用）
    this.removeFromList(node); // 先从链表中删除节点
    this.addToHead(node); // 再添加到链表的头部
    return node.value; // 返回该节点的值 value
  }
  // put 方法
  put(key, value) {
    let node = this.hash[key]; // 从哈希表中获取对应的节点
    if (node === null) {
      // 如果哈希表中没有该节点（的 key）
      if (this.count === this.capacity) {
        // 且缓存容量已满
        this.removeOldNode(); // 将最久未使用的节点删除
      }
      let newNode = new ListNode(key, value); // 缓存容量没满，创建新节点
      this.hash[key] = newNode; // 将新节点存入哈希表
      this.addToHead(newNode); // 并将新节点添加到双链表头部（更新的节点 即为 最近使用的节点）
      this.count++; // 注意缓存的数目要加一
    } else {
      // 哈希表中有该节点（的 key）
      node.value = value; // 将节点的值替换
      // 同时将该节点移动到头部（标志着最近使用）
      this.removeFromList(node); // 先从链表中删除节点
      this.addToHead(node); // 再添加到链表的头部
    }
  }
  // 删除某个节点
  removeFromList(node) {
    let temp1 = node.prev; // 暂存它的后继节点
    let temp2 = node.next; // 暂存它的前驱节点
    temp1.next = temp2; // 前驱节点的next指向后继节点
    temp2.prev = temp1; // 后继节点的prev指向前驱节点
  }
  // 移动节点到头部
  addToHead(node) {
    // 插入到虚拟头结点和真实头结点之间
    node.prev = this.dummyHead; // node的prev指针，指向虚拟头结点
    node.next = this.dummyHead.next; // node的next指针，指向原来的真实头结点
    this.dummyHead.next.prev = node; // 原来的真实头结点的prev，指向node
    this.dummyHead.next = node; // 虚拟头结点的next，指向node
  }
  // 删除最近未使用的节点
  removeOldNode() {
    let tail = this.popTail(); // 将它从链表尾部删除
    delete this.hash[tail.key]; // 哈希表中也将它删除
    this.count--; // 注意缓存的数目要减一
  }
  popTail() {
    // 删除链表尾节点
    let tail = this.dummyTail.prev; // 通过虚拟尾节点找到它
    this.removeFromList(tail); // 删除该真实尾节点
    return tail; // 返回被删除的节点
  }
}
```

时间复杂度 O(1)，空间复杂度 O(n)

### Method 2：ES6 Map

经过上面的沉淀，我们可以很好地理解下面的代码，这里不再重复了，直接上代码

```js
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.map = new Map();
  }
  get(key) {
    let value = this.map.get(key);
    if (value === undefined) return -1;
    this.map.delete(key); // 获取 = （最近）被使用，先删除
    this.map.set(key, value); // 再重新插入，（最后的）表示最近被使用
    return value;
  }
  put(key, value) {
    if (this.map.has(key)) {
      this.map.delete(key);
    }
    this.map.set(key, value);
    if (this.map.size > this.capacity) {
      // this.map.entries() 返回一个迭代器，这里只调用了一次next()
      // 则 next().value 返回迭代器的第一个键值对(最久未使用),如[0, 'a'],所以我们取key值 value[0]
      this.map.delete(this.map.entries().next().value[0]);
      // 当然这里也可以用 keys()直接拿到键值
      // this.map.delete(this.map.keys.next().value);
    }
  }
}
```

**note：**

**entries()** ：返回一个新的包含 [key, value] 对的 Iterator 对象，返回的迭代器的迭代顺序与 Map 对象的插入顺序相同。

## 腾讯：数组扁平化、去重、排序

```bash
已知如下数组：var arr = [ [1, 2, 2], [3, 4, 5, 5], [6, 7, 8, 9, [11, 12, [12, 13, [14] ] ] ], 10];
```

编写一个程序将数组扁平化去并除其中重复部分数据，最终得到一个升序且不重复的数组

思路：

1. 数组扁平化
2. 数组去重
3. 数组排序（升序）

先来看一个步骤清晰的分解式答案叭！

```js
// 扁平化
const flat = (array) => array.flat(Infinity);

// 去重
const unique = (array) => Array.from(new Set(array));

// 排序
const sort = (array) => array.sort((a, b) => a - b);

// 函数组合（函数从右开始执行到最左）
const compose =
  (...fns) =>
  (initValue) =>
    fns.reduceRight((y, fn) => fn(y), initValue);

// 组合后函数
const myFun = compose(sort, unique, flat);

// test
const arr = [
  [1, 2, 2],
  [3, 4, 5, 5],
  [6, 7, 8, 9, [11, 12, [12, 13, [14]]]],
  10,
];
console.log(myFun(arr));
// [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14]
```

**note：**偷一下懒哈哈哈哈，避免写的文章有太多重复啦！

关于数组扁平化的一些方法，可以参考这篇文章

关于数组去重的一些方法（总结了 13 种哦！），可以参考这篇文章

关于函数组合，可以参考这篇文章

综合一下，来个更 cool 的解法 ==》

```js
const getArray = (arr, dep) => Array.from(new Set(arr.flat(dep))).sort((a,b) => a-b);
// else 先把 arr 所有元素转化成字符串
const getArray = (arr, dep) => Array.from(new Set(arr.toString().split('')).sort((a,b) => a-b);
```

这里注意`sort()`，要求是升序哦！所以别忘了改写成`sort((a,b) => a-b)`

## 阿里：编写一个函数计算多个数组的交集

要求：输出结果中的每个元素唯一

思路：

检查数组个数情况
`reduce`+`filter`（内置函数法）

### Set 去重

```js
const intersection = function (...args) {
  if (args.length === 0) {
    return [];
  }
  if (args.length === 1) {
    return args[0];
  }
  return [
    ...new Set(
      args.reduce((acc, cur) => {
        return acc.filter((item) => cur.includes(item));
      })
    ),
  ];
};
```
