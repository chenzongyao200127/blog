---
layout: '../../layouts/MarkdownPost.astro'
title: 'Rust Functions In LeetCode'
pubDate: 2222-02-10
description: '收集了俺刷题过程中遇到的不熟悉却好用的Rust内置functions与traits'
author: 'Solar1s'
cover:
    url: 'https://pic.lookcos.cn/i/usr/uploads/2022/04/2067928922.png'
    square: 'https://pic.lookcos.cn/i/usr/uploads/2022/04/2067928922.png'
    alt: 'cover'
tags: ["rust","leetcode"]
theme: 'dark'
featured: false
---

> 2/10/2022
>
> 目的：收集刷题过程中遇到的不熟悉的 Rust 内置function与trait

![img](.\Rust Function In LeetCode.assets\bg.jpeg)

### `drain()` 

~~~rust
// 150. 逆波兰表达式求值
// https://leetcode.cn/problems/evaluate-reverse-polish-notation/
// 为什么大家不用last_mut()非要来回压栈呢？
impl Solution {
    pub fn eval_rpn(mut tokens: Vec<String>) -> i32 {
        let mut v = Vec::with_capacity(10);
        tokens.drain(..).for_each(|x|{
            match x.as_str(){
                "+"=>{let c=v.pop().unwrap();*v.last_mut().unwrap()+=c},
                "-"=>{let c=v.pop().unwrap();*v.last_mut().unwrap()-=c},
                "*"=>{let c=v.pop().unwrap();*v.last_mut().unwrap()*=c},
                "/"=>{let c=v.pop().unwrap();*v.last_mut().unwrap()/=c},
                x=>{v.push(x.parse().unwrap())}
            }
        });
        v.last().copied().unwrap()
    }
}
~~~

~~~rust
pub fn drain<R>(&mut self, range: R) -> Drain<'_, T, A>
where
    R: RangeBounds<usize>, 
~~~

创建一个 draining 迭代器，该迭代器删除 vector 中的指定范围并产生删除的项。

当迭代器被丢弃时，该范围内的所有元素都将从 vector 中删除，即使迭代器未被完全消耗。 如果迭代器没有被丢弃 (例如，使用 [`mem::forget`](https://rustwiki.org/zh-CN/std/mem/fn.forget.html))，则不确定删除了多少个元素。

**Panics**

如果起点大于终点或终点大于 vector 的长度，就会出现 panics。

**Examples**

~~~rust
let mut v = vec![1, 2, 3];
let u: Vec<_> = v.drain(1..).collect();
assert_eq!(v, &[1]);
assert_eq!(u, &[2, 3]);

// 全范围清除 vector
v.drain(..);
assert_eq!(v, &[]);
~~~

### `last_mut()`

~~~rust
pub fn last_mut(&mut self) -> Option<&mut T>
~~~

返回指向切片中最后一个项的可变指针。

~~~rust
let x = &mut [0, 1, 2];

if let Some(last) = x.last_mut() {
    *last = 10;
}
assert_eq!(x, &[0, 1, 10]);
~~~



### `fold()`

~~~rust
fn fold<B, F>(self, init: B, f: F) -> B
where
    F: FnMut(B, Self::Item) -> B, 
~~~

通过应用操作将每个元素 `fold` 到一个累加器中，返回最终结果。

`fold()` 有两个参数：一个初始值，一个闭包，有两个参数：一个 ‘accumulator’ 和一个元素。 闭包返回累加器在下一次迭代中应具有的值。

初始值是累加器在第一次调用时将具有的值。

在将此闭包应用于迭代器的每个元素之后，`fold()` 返回累加器。

该操作有时称为 ‘reduce’ 或 ‘inject’。

当您拥有某个集合，并且希望从中产生单个值时，`fold` 非常有用。

Note: `fold()` 和遍历整个迭代器的类似方法可能不会因无限迭代器而终止，即使在 traits 上，其结果在有限时间内是可确定的。

Note: 如果累加器类型和项类型相同，则可以使用 [`reduce()`](https://rustwiki.org/zh-CN/std/iter/trait.Iterator.html#method.reduce) 将第一个元素用作初始值。

Note: `fold()` 以**左关联**方式组合元素。 对于像 `+` 这样的关联性，元素组合的顺序并不重要，但对于像 `-` 这样的非关联性，顺序会影响最终结果。 

对于 `fold()` 的**右关联**版本，请参见 [`DoubleEndedIterator::rfold()`](https://rustwiki.org/zh-CN/std/iter/trait.DoubleEndedIterator.html#method.rfold)。

<img src="F:\Blog\Programming Languages\Rust\Rust Function In LeetCode\Rust Function In LeetCode.assets\image-20230220101101166.png" alt="image-20230220101101166" style="zoom:80%;" />

**[实现者注意](https://rustwiki.org/zh-CN/std/iter/trait.Iterator.html#实现者注意-1)**

就此而言，其他几种 (forward) 方法都具有默认实现，因此，如果它可以做得比默认 `for` 循环实现更好，请尝试显式实现此方法。

特别是，请尝试将此 `fold()` 放在组成此迭代器的内部部件上。

**Example**

基本用法：

```rust
let a = [1, 2, 3];

// 数组所有元素的总和
let sum = a.iter().fold(0, |acc, x| acc + x);

assert_eq!(sum, 6);
```

让我们在这里遍历迭代的每个步骤：

<img src="F:\Blog\Programming Languages\Rust\Rust Function In LeetCode\Rust Function In LeetCode.assets\image-20230220101322320.png" alt="image-20230220101322320" style="zoom:67%;" />

所以，我们的最终结果，`6`。

这个例子演示了 `fold()` 的左关联特性： 它构建一个字符串，从一个初始值开始，从前面到后面的每个元素继续：

~~~rust
let numbers = [1, 2, 3, 4, 5];

let zero = "0".to_string();

let result = numbers.iter().fold(zero, |acc, &x| {
    format!("({} + {})", acc, x)
});

assert_eq!(result, "(((((0 + 1) + 2) + 3) + 4) + 5)");
~~~

对于那些不经常使用迭代器的人，通常会使用 `for` 循环并附带一系列要建立结果的列表。那些可以变成 `fold () `s：

~~~rust
let numbers = [1, 2, 3, 4, 5];

let mut result = 0;

// for 循环：
for i in &numbers {
    result = result + i;
}

// fold:
let result2 = numbers.iter().fold(0, |acc, &x| acc + x);

// 他们是一样的
assert_eq!(result, result2);
~~~



### `reduce()`

~~~rust
fn reduce<F>(self, f: F) -> Option<Self::Item>
where
    F: FnMut(Self::Item, Self::Item) -> Self::Item, 
~~~

通过重复应用缩减操作，将元素缩减为一个。

如果迭代器为空，则返回 [`None`](https://rustwiki.org/zh-CN/std/option/enum.Option.html#variant.None); 否则，返回 [`None`](https://rustwiki.org/zh-CN/std/option/enum.Option.html#variant.None)。否则，返回减少的结果。

Reduce 函数是一个闭包，有两个参数：一个 ‘accumulator’ 和一个元素。 对于具有至少一个元素的迭代器，这与 [`fold()`](https://rustwiki.org/zh-CN/std/iter/trait.Iterator.html#method.fold) 相同，将迭代器的第一个元素作为初始累加器值，将每个后续元素 fold 到其中。

**example**

~~~rust
fn find_max<I>(iter: I) -> Option<I::Item>
    where I: Iterator,
          I::Item: Ord,
{
    iter.reduce(|accum, item| {
        if accum >= item { accum } else { item }
    })
}
let a = [10, 20, 5, -23, 0];
let b: [u32; 0] = [];

assert_eq!(find_max(a.iter()), Some(&20));
assert_eq!(find_max(b.iter()), None);
~~~



### `std::collections::VecQueue`

~~~rust
pub struct VecDeque<T, A = Global> 
where
    A: Allocator, 
 { /* fields omitted */ }
~~~

使用可增长的环形缓冲区实现的双端队列。

“default” 作为队列的这种用法是使用 [`push_back`](https://rustwiki.org/zh-CN/std/collections/struct.VecDeque.html#method.push_back) 添加到队列，使用 [`pop_front`](https://rustwiki.org/zh-CN/std/collections/struct.VecDeque.html#method.pop_front) 从队列中删除。

[`extend`](https://rustwiki.org/zh-CN/std/collections/struct.VecDeque.html#method.extend) 和 [`append`](https://rustwiki.org/zh-CN/std/collections/struct.VecDeque.html#method.append) 以这种方式推到后面，并从前到后迭代 `VecDeque`。



可以从数组初始化具有已知项列表的 `VecDeque`：

~~~rust
use std::collections::VecDeque;

let deq = VecDeque::from([-1, 0, 1]);
~~~

由于 `VecDeque` 是环形缓冲区，因此它的元素在内存中不一定是连续的。 如果要以单个切片的形式访问元素 (例如为了进行有效的排序)，则可以使用 [`make_contiguous`](https://rustwiki.org/zh-CN/std/collections/struct.VecDeque.html#method.make_contiguous)。 它旋转 `VecDeque`，以使其元素不环绕，并向当前连续的元素序列返回可变切片。

#### `pop_back()`

~~~rust
pub fn pop_back(&mut self) -> Option<T>
~~~

从 `VecDeque` 中删除最后一个元素并返回它; 如果它为空，则返回 `None`。

~~~rust
use std::collections::VecDeque;

let mut buf = VecDeque::new();
assert_eq!(buf.pop_back(), None);
buf.push_back(1);
buf.push_back(3);
assert_eq!(buf.pop_back(), Some(3));
~~~

#### `pop_front()`

~~~rust
pub fn pop_front(&mut self) -> Option<T>
~~~

删除第一个元素并返回它，如果 `VecDeque` 为空，则返回 `None`。

~~~rust
use std::collections::VecDeque;

let mut d = VecDeque::new();
d.push_back(1);
d.push_back(2);

assert_eq!(d.pop_front(), Some(1));
assert_eq!(d.pop_front(), Some(2));
assert_eq!(d.pop_front(), None);
~~~

#### `push_front()`

将元素添加到 `VecDeque`。

~~~rust
use std::collections::VecDeque;

let mut d = VecDeque::new();
d.push_front(1);
d.push_front(2);
assert_eq!(d.front(), Some(&2));
~~~

#### `push_back()`

将一个元素追加到 `VecDeque` 的后面。

~~~rust
use std::collections::VecDeque;

let mut buf = VecDeque::new();
buf.push_back(1);
buf.push_back(3);
assert_eq!(3, *buf.back().unwrap());
~~~

> Example:
>
> #### [649. Dota2 参议院](https://leetcode.cn/problems/dota2-senate/)
>
> 难度中等268收藏分享切换为英文接收动态反馈
>
> Dota2 的世界里有两个阵营：`Radiant`（天辉）和 `Dire`（夜魇）
>
> Dota2 参议院由来自两派的参议员组成。现在参议院希望对一个 Dota2 游戏里的改变作出决定。他们以一个基于轮为过程的投票进行。在每一轮中，每一位参议员都可以行使两项权利中的 **一** 项：
>
> - **禁止一名参议员的权利**：参议员可以让另一位参议员在这一轮和随后的几轮中丧失 **所有的权利** 。
> - **宣布胜利**：如果参议员发现有权利投票的参议员都是 **同一个阵营的** ，他可以宣布胜利并决定在游戏中的有关变化。
>
> 给你一个字符串 `senate` 代表每个参议员的阵营。字母 `'R'` 和 `'D'`分别代表了 `Radiant`（天辉）和 `Dire`（夜魇）。然后，如果有 `n` 个参议员，给定字符串的大小将是 `n`。
>
> 以轮为基础的过程从给定顺序的第一个参议员开始到最后一个参议员结束。这一过程将持续到投票结束。所有失去权利的参议员将在过程中被跳过。
>
> 假设每一位参议员都足够聪明，会为自己的政党做出最好的策略，你需要预测哪一方最终会宣布胜利并在 Dota2 游戏中决定改变。输出应该是 `"Radiant"` 或 `"Dire"` 。
>
> ~~~
> 输入：senate = "RD"
> 输出："Radiant"
> 解释：
> 第 1 轮时，第一个参议员来自 Radiant 阵营，他可以使用第一项权利让第二个参议员失去所有权利。
> 这一轮中，第二个参议员将会被跳过，因为他的权利被禁止了。
> 第 2 轮时，第一个参议员可以宣布胜利，因为他是唯一一个有投票权的人。
> ~~~
>
> ~~~rust
> use std::collections::VecDeque;
> 
> impl Solution {
>     pub fn predict_party_victory(senate: String) -> String {
>         let mut radiant = VecDeque::new();
>         let mut dire = VecDeque::new();
>         senate.chars().enumerate().for_each(|(i, c)| {
>             if c == 'R' {
>                 radiant.push_back(i);
>             } else {
>                 dire.push_back(i)
>             }
>         });
>         while !radiant.is_empty() && !dire.is_empty() {
>             let (r, d) = (radiant.pop_front().unwrap(), dire.pop_front().unwrap());
>             if r < d {
>                 radiant.push_back(r + senate.len());
>             } else {
>                 dire.push_back(d + senate.len());
>             }
>         }
> 
>         if radiant.is_empty() {
>             "Dire".into()
>         } else {
>             "Radiant".into()
>         }
>     }
> }
> ~~~
>
> 
