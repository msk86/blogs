# 如何使用Javascript快速创建0-100的数组

我们经常会遇到这样的需求，需要产生一个序列数组，基于这个数组，进行一些后续处理。例如：

    sequence.map(function(num) { return 5 * num + ' days'; });
    // => ['5 days', '10 days', ...]

那么问题就出现了，在Javascript中，我们都是如何产生这样一个[0, 100)的数字序列数组呢？

## 旧时代

在Javascript开发资源还比较匮乏的年代，一般我们都是这么实现的：

    var sequence = [];
    for(var i = 0; i < 100; i++) {
      sequence.push(i);
    }

但是看看当时一些其他开发语言——Python、Ruby——人家的代码是怎么写的：

    sequence = range(1, 100) // Python

或者

    sequence = (1..100).to_a // Ruby

无论是从代码量还是代码优美程度上来讲，都毫无可比性。
所以我们勤劳的程序员们想了很多办法来解决这个问题：

* 抽象方法，一个`range(from, to)`方法已经能很好的解决问题。又或是将之放在ArrayUtils.js中。
* 引用类库，例如使用`underscore.js`的`_.range(100)`。这在本质上和自己抽象并无太大的区别，只是在项目需要使用`underscore.js`的时候，给我们提供了一些额外的便利。
* CoffeeScript或者TypeScript（特殊时期的产物，虽然还在维护，但是我认为ES6、ES7最终会成为主流），提供了类似Ruby的Range语法：`array = [1..100]`
* 一些快捷的创建方法，例如：

        sequence = Array.apply(0, Array(100))
            .map(function(v, k) {return k;});
        // [0, 1, ..., 99]
        // 因为: 1 in Array(100) === false
        // 即此时数组还没有任何下标，所以这里必须用Array(100)再创建一个数组，才会有可用的数组下标
        
作为一个程序员，我希望能够写出一些非常漂亮的代码，这样写出来的`range`方法非常炫酷，细看的话，每个人也能看明白是什么意思，用它来代替`for loop`我认为是一个还不错的选择。

## 新时代

ES6的到来，能否帮助我们直接解决这个问题？

很遗憾，答案是否定的，依然没有一种语法糖或者内建方法帮助我们简单的创建一个序列数组。所以我们还是需要自己解决问题，而且要漂亮的解决。

用上面的思路，我们先简单地把代码改为：

    sequence = Array(100).fill(0).map((v, k) => k);
    // [0, 1, ..., 99]

只要快速的填充数组之后，我们就能简单的取到数组的下标了。

还是有点长？那就再来看看[Array Comprehension](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Array_comprehensions)，这是ES6的一个实验性功能，这个语法可以让我们快速的给数组赋值。我们可以先试试这个语法。

    sequence = [for(i of Array(100).keys()) i]
    // [0, 1, ..., 99]
    sequence = [for(i of Array(100).keys()) i * 10 + 1]
    // [1, 11, ..., 991]

[Array.prototype.keys](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/keys)方法会返回一个包含有所有数组元素下标的迭代器，用Comprehension的方法可以简单的取到迭代器里的所有值。

看起来好像还不错，可是仍然没有达到Python和Ruby的简洁程度，而且还有一个很重要的问题，这货是实验语法！

也就是说，将来ES6时代来临后，Comprehension还有没有，我们的代码到时候还能不能用，都不确定！

所以还得再改改。

考虑如果不使用Comprehension，那么如何快速的从迭代器转化为数组就成为了我们需要解决的问题了。
迭代器除了`iter.next()`、`for...of`的使用方法之外，还有没有其他的取值方法呢？

ES6引入了另外一个方法：[Array.from](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from)。
这个方法可以快速的把一个类似数组的对象——如iterator、arguments等——转化为一个数组，于是就有了：

    sequence = Array.from(Array(100).keys())
    // [0, 1, ..., 99]
    sequence = Array.from(Array(100).keys(), (n) => n * 10 + 1)
    // [1, 11, ..., 991]

## 我们到底该怎么写

言归正传，虽然我们已经有了这么些看起来很有趣也很快捷的创建序列数组的方法，但是出于代码的可读性、可重用等要求，我还是不推荐直接写这样的奇淫巧计在代码里。

所以**抽象**和**封装**势在必行。

以前我们会封装一个方法来返回结果数组，而在ES6时代，一个`Generator`应该更符合我们的要求。

    function* range(start = 0, end = 0, interval = 1) {
      for(let i = start; i < end; i = i + interval) {
        yield i;
      }
    }

我们可以用多种方法来消费这个`Generator`：

    for(let i of range(0, 100, 3)) {
      doSomething(i);
    }
    // 0
    // 3
    // ...
    // 99

    let iter = range(0, 100);
    iter.next(); // {value: 0, done: false}
    iter.next(); // {value: 1, done: false}
    ...
    iter.next(); // {value: 99, done: false}
    iter.next(); // {value: undefined, done: true}

相比创建数组，创建Generator有什么优势呢？
Generator在运行过程中速度更快、占用内存更少，因为它不会在一开始的时候就生成所有的数据。

当我们真的需要一个数组的时候，`Array.from(range(0, 100))`可以帮助我们。
