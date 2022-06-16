## [leetcode-349：两个数组的交集](https://leetcode.cn/problems/intersection-of-two-arrays/)

**Question：给定两个数组，编写一个函数来计算它们的交集。**

示例：

    输入：nums1 = [1,2,2,1]，nums2 = [2,2]

    输出：[2]

输出结果中的每个元素一定是唯一的，顺序不做要求。

### Method 1：暴力解

上来我就搞了个暴力解先压压惊

```js
const nums1 = [4, 9, 5];
const nums2 = [9, 4, 9, 8, 4];
const intersection = function (nums1, nums2) {
  let arr = [];
  for (let i = 0; i < nums1.length; i++) {
    for (let j = 0; j < nums2.length; j++) {
      if ((nums1[i] = nums2[j])) {
        arr.push(nums1[i]);
      }
    }
  }
  arr = [...new Set(arr)]; // 去重并将Set对象转为真正的数组
  return arr;
};
console.log(intersection(nums1, nums2)); // [9,4,8]
```

What ？结果打印了个啥，不应该是 [4,9] 嘛？

搞了半天，我还以为我哪里思路出问题了，害特意拿出草稿纸比划了一下每一步。But，好像没问题啊？？？再打个断点看看

啥情况，我的 nums1 怎么还能变？再好好瞧瞧，好像是 nums2 的值被赋给了 nums1。OK，值被改了，那在哪里被赋值了呢？

万万没想到，犯了这个低级错误，emmmm，判断语句里的比较运算符 === 写成了赋值运算符 =。大意了大意了！！！（问题虽小而又低级，但是该是要保持警惕，避免给自己添加多余的麻烦哈！）

不过复杂度好像不太行哦！这里用了双层 for 循环进行比较，时间复杂度为 O(mn)。任重而道远......

**note:**

数组的 push ：这里要注意 push() 方法将一个或多个元素添加到数组的末尾，返回的是该数组的新长度，而不是新数组哦！

如果你还想跟我回顾一下 Set 数据结构，不妨点进去看看。

### Method 2：哈希表 + Set

**思路：**

先用哈希集合存储两个数组元素，再遍历较小的集合，判断每个元素是否在另一个集合中，如果在就将该元素添加到返回值中。

```js
const getRes = function (set1, set2) {
  if (set1.size > set2.size) return getRes(set2, set1);
  const res = new Set();
  for (const num of set1) {
    if (set2.has(num)) {
      res.add(num);
    }
  }
  return [...res];
};
const intersection = function (nums1, nums2) {
  const set1 = new Set(nums1);
  const set2 = new Set(nums2);
  return getRes(set1, set2);
};
```

时空复杂度都是 O(m+n)

### Method 3：哈希表

用空间换时间，能不能不要对 nums1 的每一个数都遍历检查 nums2 中是否有相同的数呢？

**思路：**

先遍历 nums1 ，将其中的数存入哈希表，记录为 true。再遍历 nums2，如果当前的数在哈希表中存在，说明这是个交集，推入结果数组。同时把这个在哈希表中的数标记为 false（即哈希表中去除原有的这个 nums1 中出现的数），避免重复推入结果数组。

```js
const intersection = (nums1, nums2) => {
  const map = {};
  const res = []; // 结果数组

  for (const num1 of nums1) {
    map[num1] = true; // 记录nums1出现过的数
  }
  for (const num2 of nums2) {
    if (map[num2]) {
      // 如果nums2的这个数在nums1出现过
      res.push(num2); // 交集数推入res
      map[num2] = false; // 修改哈希表：这个数的记录改为false，避免重复推入res
    }
  }
  return res;
};
```

时空复杂度都为 O(n)

### Method 4：排序 + 双指针

**思路：**

将两个数组升序，方便使用双指针法
初始双指针分别指向数组头部，并添加辅助变量 pre 表示上一次加入答案数组的元素（保证数组中的值唯一，且这里答案数组元素值是递增的）
每次比较指针指向的数值。若两个数值不相等，右移较小数值指针；若相等，且该值不等于当前 pre，将该值加入答案数组，并更新 pre 为该值，同时将两个指针都右移一位。
直到其中一个数组元素遍历完毕，结束遍历。

```js
const intersection = function (nums1, nums2) {
  // 升序
  nums1.sort((a, b) => a - b);
  nums2.sort((a, b) => a - b);
  const len1 = nums1.length;
  const len2 = nums2.length;
  let index1 = 0,
    index2 = 0; // 双指针分别指向数组头部
  const res = [];
  while (index1 < len1 && index2 < len2) {
    const data1 = nums1[index1];
    const data2 = nums2[index2];
    if (data1 === data2) {
      // 保证答案数组元素唯一性(注意考虑res为空的情况，直接插入)
      if (!res.length || data1 != res[res.length - 1]) {
        res.push(data1);
      }
      index1++;
      index2++;
    } else if (data1 < data2) {
      index1++;
    } else {
      index2++;
    }
  }
  return res;
};
```

时间复杂度：O(mlogm+nlogn)

空间复杂度：O(logm+logn)

下面我们使用内置函数法来解决

### Method 5：Set 去重 + 过滤器（filter）

**思考：**

拿到 Set 集合的元素，把其中在另一个 Set 集合中出现的元素过滤出来不就可以了嘛？而且数组过滤器返回的就是一个新数组鸭！(当然，你也可以用 foreach 来替代 filter，办法多多。。。)

话不多说，上代码，一行搞定，我直接 666

```js
const intersection = function (nums1, nums2) {
  return (result = [...new Set(nums1)].filter((v) => new Set(nums2).has(v)));
};
// 或者这样
const intersection = function (nums1, nums2) {
  return (result = [...new Set(nums1.filter((v) => nums2.includes(v)))]);
};
// 还可以换一下转数组的方法
const intersection = function (nums1, nums2) {
  return Array.from(new Set(nums1.filter((v) => nums2.includes(v))));
};
```

方法虽好，看起来一行代码解决几十行代码的问题，但是性能不太行啊~~~~~

那再换一下方法优化一下。。。

### Method 6：filter + indexOf + lastIndexOf

**思路：**

indexOf（也可以用 includes）判断是否存在，lastIndexOf 判断是否重复

```js
const intersection = function (nums1, nums2) {
  return nums1.filter(
    (v, i) => nums2.indexOf(v) > -1 && nums1.lastIndexOf(v) === i
  );
};
```

note ：

indexOf()方法返回在数组中可以找到一个给定元素的第一个索引，如果不存在，则返回-1

lastIndexOf() 方法返回指定元素在数组中的最后一个的索引，如果不存在则返回 -1。从数组的后面向前查找，从 fromIndex 处开始.

哎呦，不错哦~~~~

## [leetcode-350：两个数组的交集 2](https://leetcode.cn/problems/intersection-of-two-arrays-ii/)

Question：给定两个数组，编写一个函数来计算它们的交集。

示例：

    输入：nums1 = [1,2,2,1]，nums2 = [2,2]

    输出：[2，2]

输出结果中每个元素出现的次数，应与元素在两个数组中出现次数的最小值一致。不考虑顺序。

题目是对上一个题目的修改，这里我们需要考虑的是元素出现次数。通过上面的阅读我们能够很快想到，可以使用哈希表做记录的同时，记录相同元素出现的次数。

### Method 1：哈希表

**思路：**

在 hashMap 记录一个数组中存在的数和对应出现的次数
遍历第二个数组，检查元素在 hashMap 中是否存在：存在，记为 true，并将该元素添加到结果中，同时减少 hashMap 中的计数
小优化：这里我们可以先检查数组大小，对较小的数组进行哈希映射，可以减少内存的使用

```js
const intersect = function (nums1, nums2) {
  const map = {};
  const res = []; // 结果数组
  // 检查数组大小做交换
  if (nums1.length > nums2.length) {
    let temp = nums2;
    nums2 = nums1;
    nums1 = temp;
  }
  for (const num1 of nums1) {
    if (map[num1]) {
      // 如果元素已经存在，则增加对应的计数
      map[num1]++;
    } else {
      // // 元素不存在，计数为1
      map[num1] = 1;
    }
  }
  for (const num2 of nums2) {
    if (map[num2] > 0) {
      // 如果nums2的这个数在nums1出现过
      res.push(num2); // 把这个数推入res
      map[num2]--; // 同时减少 map 中对应元素的计数
    }
  }
  return res;
};
```

### Method 2：排序 + 双指针（推荐数据有序时使用）

**思路：**

将两个数组升序
两个指针初始设为 0，指向数组头部，比较两个指针的元素是否相等
如果相等，元素 push 到结果数组，两个指针同时向后移动一位
如果不相等，元素小的指针向后移动一位

```js
const intersect = function (nums1, nums2) {
  let index1 = 0,
    index2 = 0;
  let res = [];
  nums1 = nums1.sort((a, b) => a - b);
  nums2 = nums2.sort((a, b) => a - b);
  while (index1 < nums1.length && index2 < nums2.length) {
    if (nums1[index1] === nums2[index2]) {
      res.push(nums1[index1]);
      index1++;
      index2++;
    } else if (nums1[index1] < nums2[index2]) {
      index1++;
    } else {
      index2++;
    }
  }
  return res;
};
```
