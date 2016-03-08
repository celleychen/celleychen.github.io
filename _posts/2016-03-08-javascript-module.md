---
layout: post
title: 模块化JavaScript(转)
category: 技术
comments: true
---

模块化JavaScript之风早已席卷而来， CommonJS 、 AMD 、 NodeJS 、 RequireJS 、 SeaJS 、 curljs  等模块化的JavaScript概念及库扑面而来，不得不承认，对于前端JavaScript代码的组织编写是一次伟大的变革。本文主要参考 snandy 的有关 modular js 系列文章，对SeaJS和RequireJS做一个系统的深入分析及对比。

#### 一、我们为什么要用模块化的JavaScript

相信大家也都经历了“过程式的JavaScript”、“面向对象的JavaScript”，再到现在的“面向模块的JavaScript”。这个“面向模块的JavaScript”到底能走多久，我们拭目以待。
但是我们需要知道，我们为什么要拥抱模块化的JavaScript，它到底有什么奇妙之处，读一下这一篇文章吧—— 前端模块化开发的价值 ，OK，明白了，它解决了两大问题——
其一、恼人的命名冲突：
这个确实够烦人，为了解决之，我们动用闭包作用域、我们动用命名空间，再或者...
其二、烦琐的文件依赖：
这个问题更不必说，我们做前端支持的，常常要替后端同学们解决这类问题，我们自己也常常忽略这类问题。平常的开发中，我们要对我们的JS文件的加载顺序要小心加小心。一个字——确实烦。
好，现在，我们有SeaJS了，这两大问题变得很轻松就能搞定了，开心哪。
针对第一个问题：通过 exports 暴露接口 。这意味着不需要命名空间了，更不需要全局变量。这是一种彻底的命名冲突解决方案。
针对第二个问题：通过 require 引入依赖 。这可以让依赖内置，开发者只需关心当前模块的依赖，其他事情 SeaJS 都会自动处理好。对模块开发者来说，这是一种很好的 关注度分离 ，能让程序员更多地享受编码的乐趣。

#### 二、 浏览器中的模块规范AMD是怎么诞生的？

SeaJS我们领教过了，但是对于它的模块化写法，到底是咋回事儿？我们先看看module写法的演变。直接去看 JavaScript中模块“写法” ，使用 统一的风格来写模块，非nodeJS打头阵不可， 以下是node中是写模块的一个示例。

1> math.js

```javascript
exports.add = function() {
    var sum = 0, i = 0, args = arguments, l = args.length;
    while (i < l) {
        sum += args[i++];
    }
    return sum;
};
```

2> increment.js

```javascript
var add = require('math').add;
exports.increment = function(val) {
    return add(val, 1);
};
```

3> main.js，该文件为入口文件

```javascript
var inc = require('increment').increment;
var a = 1;
inc(a); // 2
```

从以上代码示例可以看到——

1> node要求一个js文件对应一个模块。可以把该文件中的代码想象成是包在一个匿名函数中，所有代码都在匿名函数中，其它模块不可访问除exports外的私有变量
2> 使用exports导出API
3> 使用require加载其它模块

CommonJS module基本要求 如下——

1> 标示符require，为一个函数，它仅有一个参数为字符串，该字符串须遵守Module Identifiers的6点规定
2> require方法返回指定的模块API
3> 如果存在依赖的其它模块，那么依次加载
4> require不能返回，则抛异常
5> 仅能使用标示符exports导出API

打头阵的NodeJS真是享誉甚高，前端的我们是不是也可以使用同样的模块写法写模块呢？ 当然nodeJS是最好的效仿对象。因为前后端有一个统一的方式写JS模块岂不乐哉！可是，事与愿违，读这个文章吧—— Node.js模块风格在浏览器中的尝试 ，深入理解之，便能明白，为什么NodeJS模块写法为什么不适合浏览器端。为了解决之，便出现了 Modules/Wrappings  ，顾名思义包裹的模块。该规范约定如下——

1> 定义模块用module变量，它有一个方法declare
2> declare接受一个函数类型的参数，如称为factory
3> factory有三个参数分别为require、exports、module
4> factory使用返回值和exports导出API
5> factory如果是对象类型，则将该对象作为模块输出

Modules/Wrappings 的出现使得浏览器中实现它变得可能，包裹的函数作为回调。即使用script tag作为模块加载器，script完全下载后去回调，回调中进行模块定义。
接着，也就产生了AMD( 全称为异步模块定义 )规范了（读 AMD：浏览器中的模块规范 ）。 从名称上看便知它是适合script tag的。也可以说AMD是专门为浏览器中JavaScript环境设计的规范。它吸取了CommonJS的一些优点，但又不照搬它的格式。开始AMD作为CommonJS的 transport format   存在，因无法与CommonJS开发者达成一致而独立出来。
AMD开创性的提出了自己的模块风格。但后来又做了妥协，兼容了 CommonJS  Modules/Wrappings。

```javascript
define(function(require, exports, module) {
    var base = require('base');
    exports.show = function() {
        // todo with module base
    } 
});
```

不考虑多了一层函数外，格式和Node.js是一样的。使用require获取依赖模块，使用exports导出API。
除了define外，AMD还保留一个关键字require。 require   作为规范保留的全局标识符，可以实现为 module loader。
目前，实现AMD的库有 RequireJS  、 curl  、 Dojo  、 bdLoad 、 JSLocalnet  、 Nodules  等。

#### 三、SeaJS与RequireJS的区别
AMD设计出一个简洁的写模块API： define(id?, dependencies?, factory);
我们再看一下SeaJS的模块写法——
SeaJS默认使用全局的define函数写模块（可把define当成语法关键字），define定义了三个形参id, deps, factory。
define(id?, deps?, factory);

搞不懂SeaJS和  RequireJS  define的区别。 它们都有个全局的define，形参都是三个，且对应的形参名也一样，会误认为SeaJS也是AMD的实现。 事实上SeaJS和RequireJS的define前两个参数的确一样。 id都为字符串，都遵循  Module Identifiers 。deps都是指依赖模块，类型都为数组。区别仅在于第三个参数factory，虽然类型也都是函数，但factory的参数意义却不同。
RequireJS中factory的参数有两种情况。

a、和deps（数组）元素一一对应。即deps有几个，factory的实参就有几个。(官方推荐的模块定义方法)

```javascript
define(['a', 'b'], function(a, b){
    // todo
});
```

b、固定为require,exports, module（modules/wrappings格式）。

```javascript
define(function(require, exports, module){
    // todo
});
```

这种方式是RequireJS后期向  Modules/Wrappings  的妥协，即兼容了它。而SeaJS的define仅支持RequireJS的第二种写法：Modules/Wrappings。

注意 ：SeaJS遵循的是  Modules/Wrappings  和  Modules/1.1.1 。这两个规范中都没有提到define关键字，Modules/Wrapping中要求定义模块使用module.declare而非define。而恰恰只有AMD规范中有define的定义。即虽然SeaJS不是AMD的实现，但它却采用了让人极容易误解的标识符define。

对于SeaJS和RequireJS的其他差异，可以读 SeaJS与RequireJS的异同 。

两者相同之处——

RequireJS 和 SeaJS 都是模块加载器，倡导模块化开发理念，核心价值是让 JavaScript 的模块化开发变得简单自然。

两者不同之处——

1> 定位有差异。RequireJS 想成为浏览器端的模块加载器，同时也想成为 Rhino / Node 等环境的模块加载器。SeaJS 则专注于 Web 浏览器端，同时通过 Node 扩展的方式可以很方便跑在 Node 环境中。

2> 遵循的规范不同。RequireJS 遵循 AMD（异步模块定义）规范，SeaJS 遵循 CMD （通用模块定义）规范。规范的不同，导致了两者 API 不同。SeaJS 更贴近 CommonJS Modules/1.1 和 Node Modules 规范。

3> 推广理念有差异。RequireJS 在尝试让第三方类库修改自身来支持 RequireJS，目前只有少数社区采纳。SeaJS 不强推，采用自主封装的方式来“海纳百川”，目前已有较成熟的封装策略。

4> 对开发调试的支持有差异。SeaJS 非常关注代码的开发调试，有 nocache、debug 等用于调试的插件。RequireJS 无这方面的明显支持。

5> 插件机制不同。RequireJS 采取的是在源码中预留接口的形式，插件类型比较单一。SeaJS 采取的是通用事件机制，插件类型更丰富。

#### 四、AMD和CMD的区别

接着引深来看，大家都知道SeaJS遵循CMD（通用模块定义），RequireJS遵循AMD，那么CMD和AMD到底有什么区别？
上面描述过 AMD模块定义规范 ，接着我们看看 CMD模块定义规范 ，可大致了解一下CMD到底是怎样的规范。对于两者的差异，读读玉泊对它的回答吧—— AMD和CMD区别有哪些？

AMD 是 RequireJS 在推广过程中对模块定义的规范化产出。
CMD 是 SeaJS 在推广过程中对模块定义的规范化产出。
类似的还有 CommonJS Modules/2.0 规范，是 BravoJS 在推广过程中对模块定义的规范化产出。
还有不少⋯⋯

这些规范的目的都是为了 JavaScript 的模块化开发，特别是在浏览器端的。
目前这些规范的实现都能达成浏览器端模块化开发的目的。
区别：

1. 对于依赖的模块，AMD 是提前执行，CMD 是延迟执行。不过 RequireJS 从 2.0 开始，也改成可以延迟执行（根据写法不同，处理方式不同）。CMD 推崇 as lazy as possible.
2. CMD 推崇依赖就近，AMD 推崇依赖前置。看代码：

```javascript
// CMD
define(function(require, exports, module) {
　　var a = require('./a')
　　a.doSomething()
　　// 此处略去 100 行
　　var b = require('./b') // 依赖可以就近书写
　　b.doSomething()
　　// ... 
})

// AMD 默认推荐的是
define(['./a', './b'], function(a, b) { // 依赖必须一开始就写好
　　a.doSomething()
　　// 此处略去 100 行
　　b.doSomething()
　　...
}) 
```

虽然 AMD 也支持 CMD 的写法，同时还支持将 require 作为依赖项传递，但 RequireJS 的作者默认是最喜欢上面的写法，也是官方文档里默认的模块定义写法。
3. AMD 的 API 默认是一个当多个用，CMD 的 API 严格区分，推崇职责单一。比如 AMD 里，require 分全局 require 和局部 require，都叫 require。CMD 里，没有全局 require，而是根据模块系统的完备性，提供 seajs.use 来实现模块系统的加载启动。CMD 里，每个 API 都简单纯粹。
4. 还有一些细节差异，具体看这个规范的定义就好，就不多说了。

#### 五、总结
写了挺多，当然我承认都是摘录的，不过把模块化来源的思路搞定，收获也就有了。我再理顺一下——
1> 我们为什么要用模块化的JavaScript
最主要的是解决命名冲突、解决文件依赖这两大问题。
2> 浏览器中的模块规范AMD是怎么诞生的？
使用script tag作为模块加载器，script完全下载后去回调，回调中进行模块定义。
3> SeaJS与RequireJS的区别
RequireJS 遵循的是 AMD（异步模块定义）规范，SeaJS 遵循的是 CMD （通用模块定义）规范。

4> AMD和CMD的区别
     （1） 对于依赖的模块，AMD 是 提前执行 ，CMD 是 延迟执行 。不过 RequireJS 从 2.0 开始，也改成可以延迟执行（根据写法不同，处理方式不同）。CMD 推崇 as lazy as possible.
     （2） CMD 推崇 依赖就近 ，AMD 推崇 依赖前置 。（这一点是非常重要的区别）
     （3） AMD 的 API 默认是 一个当多个用 ，CMD 的 API 严格区分，推崇 职责单一 。
    需要记住—— AMD 是 RequireJS 在推广过程中对模块定义的规范化产出。 CMD 是 SeaJS 在推广过程中对模块定义的规范化产出。

#### 六、引深
    本文仅仅讲述了前端Javascript的模块化开发思想，但是对于其他的HTML和CSS，我们是否也可以采用模块化话的开发思想来引导呢？答案是肯定的。尤其在前端多人合作开发的情况下，这种模块化的开发模式会让我们爱不释手。这里我只细读过这样一篇文章， 前端开发：模块化 — 高效重构 。该论题还在研究中，敬请期待... 