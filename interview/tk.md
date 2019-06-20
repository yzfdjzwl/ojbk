## pdd一面

1、事件轮循
    - 两个线程，主线程和 event loop 线程
    - 一种数据结构，用来等待和发送消息
2、浏览器渲染页面的原理
    - dom tree 和 style tree 合成 render tree
    - js style layout paint composite
    - 浏览器所做的工作实际上是：
    - https://segmentfault.com/a/1190000000490328
1. 获取DOM后分割为多个图层
2. 对每个图层的节点计算样式结果（Recalculate style--样式重计算）
3. 为每个节点生成图形和位置（Layout--回流和重布局）
4. 将每个节点绘制填充到图层位图中（Paint Setup和Paint--重绘）
5. 图层作为纹理上传至GPU
6. 符合多个图层到页面上生成最终屏幕图像（Composite Layers--图层重组）

Chrome中满足以下任意情况就会创建图层：
* 3D或透视变换（perspective transform）CSS属性
* 使用加速视频解码的<video>节点
* 拥有3D（WebGL）上下文或加速的2D上下文的<canvas>节点
* 混合插件（如Flash）
* 对自己的opacity做CSS动画或使用一个动画webkit变换的元素
* 拥有加速CSS过滤器的元素
* 元素有一个包含复合层的后代节点（一个元素拥有一个子元素，该子元素在自己的层里）
* 元素有一个z-index较低且包含一个复合层的兄弟元素（换句话说就是该元素在复合层上面渲染）
- 注意事项
- 如果图层中某个元素需要重绘，那么整个图层都需要重绘。比如一个图层包含很多节点，其中有个gif图，gif图的每一帧，都会重回整个图层的其他节点，然后生成最终的图层位图。所以这需要通过特殊的方式来强制gif图属于自己一个图层（translateZ(0)或者translate3d(0,0,0)），CSS3的动画也是一样（好在绝大部分情况浏览器自己会为CSS3动画的节点创建图层）
- 改变的属性仅仅影响图层的组合，变换（transform）和透明度（opacity）就属于这种情况。通过节点的transform可以修改节点的位置、旋转、大小等。我们平常会使用left和top属性来修改节点的位置，但正如上面所述，left和top会触发重布局，修改时的代价相当大。取而代之的更好方法是使用translate，这个不会触发重布局
- opacity
* translate
* rotate
* scale
3、deepClone
    - arrayLike 和 symbol 出问题了
        - getOwnPropertySymbols 用于获取 symbol
        - Reflect.ownKeys 可以遍历到 symbol
    - http://es6.ruanyifeng.com/#docs/symbol
4、大促性能优化
    - 层爆炸是如何引起的？
    - transfrom 和 opacity 为什么能优化动画的性能
7、 重绘和重排 重排是布局重新计算，类似 CSS 引擎，重绘是渲染引擎的工作
- https://www.ruanyifeng.com/blog/2015/09/web-page-performance-in-depth.html
- 样式的写操作之后，如果有下面这些属性的读操作，都会引发浏览器立即重新渲染
- offsetTop/offsetLeft/offsetWidth/offsetHeight
scrollTop/scrollLeft/scrollWidth/scrollHeight
clientTop/clientLeft/clientWidth/clientHeight
getComputedStyle()
- 从性能角度考虑，尽量不要把读操作和写操作，放在一个语句里面。
- 样式表越简单，重排和重绘就越快。
重排和重绘的DOM元素层级越高，成本就越高。
table元素的重排和重绘成本，要高于div元素
- https://www.ruanyifeng.com/blog/2015/09/web-page-performance-in-depth.html
8、 React 优化
    - vdom 的 diff 算法
    - https://www.infoq.cn/article/react-dom-diff
    - 给定任意两棵树，找到最少的转换步骤。但是标准的的 Diff 算法复杂度需要 O(n^3)，这显然无法满足性能要求。
    - O(n)
        - 两个相同组件产生类似的 DOM 结构，不同的组件产生不同的 DOM 结构；
        - 对于同一层次的一组子节点，它们可以通过唯一的 id 进行区分。
    - 不同节点类型的比较
    - （1）节点类型不同 
        - 对树进行逐层比较
        - constructor: 构造函数，组件被创建时执行；
componentDidMount: 当组件添加到 DOM 树之后执行；
componentWillUnmount: 当组件从 DOM 树中移除之后执行，在 React 中可以认为组件被销毁；
componentDidUpdate: 当组件更新时执行。
    - （2）节点类型相同，但是属性不同
        - 对属性进行重设从而实现节点的转换
    - 列表节点的比较(不在同一层的节点的比较，即使它们完全一样，也会销毁并重新创建)
        - 添加、删除和排序
- DocumentFragment，文档片段接口，表示一个没有父级文件的最小文档对象。它被作为一个轻量版的 Document 使用，用于存储已排好版的或尚未打理好格式的XML片段。最大的区别是因为 DocumentFragment 不是真实DOM树的一部分，它的变化不会触发 DOM 树的（重新渲染) ，且不会导致性能等问题。
9、 看源码不
10、node 做了哪些东西  https://juejin.im/post/5aba10fef265da2398674a36
- v8 引擎解析 js 脚本
- 解析后代码执行 node API
- libUV 库负责 node API 的执行，分成不同的线程，行程 event loop ，已异步的方式将结果返给 v8
- v8 引擎将结果返回给用户
11、继承
    - https://www.cnblogs.com/codetker/p/4672135.html
    - 组合模式
12、node 的内存分配  https://juejin.im/post/5a4f1281f265da3e484ba02f
    - https://juejin.im/post/5a4f1281f265da3e484ba02f
    - 所有的JS对象都是通过堆来进行分配的。
    - 使用process.memoryUsage()
    - 内存限制主要原因是v8的垃圾回收制度。1.5GB内存做一次小的回收需要50MS，做一次非增量性回收需要1S以上，并且这会使JS线程暂停。因此限制内存。

￼

￼
当处理一个垃圾回收周期时，暂停所有程序的执行。(stop-the-world 全停顿)  
在大多数垃圾回收周期，每次仅处理部分堆中的对象，使暂停程序所带来的影响降至最低。(增量标记等算法) 
准确知道在内存中所有的对象及指针，避免错误地把对象当成指针所带来的内存泄露。(标记指针法:在每个指针的末位预留一位来标记这个字代表的是指针或数据。)
在V8中，对象堆被分为两个部分：新创建的对象所在的新生代，以及在一个垃圾回收周期后存活的对象被提升到的老生代。
如果一个对象在一个垃圾回收周期中被移动，那么V8将会更新所有指向此对象的指针。
13、React16 的新特性
    - flow？Store？
14、箭头函数的区别
    - https://www.jianshu.com/p/eca50cc933b7
15、node 垃圾回收？gc
16、对于多端统一的建议
    - floy？

## pdd二面

拼多多二面：
- 自我介绍
    - 为什么想跳槽
    - 简历里面看着除了 React 还有 RN 和 Vue，为啥
- React 和 Vue 的区别
    - Vue 数据双向绑定，在一个文件里面
    - React jsx 和 style 拆分得比较好（可能是历史问题），模块化
    - （生命周期不同？）
    - 强调 React 比较多比较熟悉
- react 高阶组件，比如 withRouter（没了解）
- 伪元素
    - before
    - after 的作用
    - 比如说画图，画三角形（before 和 after 是内外？）
        - border-left： 50% 50% transparent
        - border-right:  50% 50% transparent
        - border-top:   100% 100% red
- css 选择器的优先级
    - id 扫描器，最优先，扫描到就不继续往下扫码了
    - class 全扫描，并且是倒着来的
    - a.classNameA 的优先级比 .classNameA 高
    - 其他的暂时没了解
- 常见的垂直和水平居中
    - 文字的
        - text-align
        - height 和 line-hieght
    - 块级元素的
        - display: table-cell
        - vertical-align: middle
    - margin: 0 auto
    - 相对定位
        - postion: relative， left: 50% 相对于父级左移 50%
        - trasform: 相对于自己左移 50%
- 写了一个数据去重的算法
    - 判断条件如果不用 indexOf 用啥
    - filter
- React 的生命周期
    - react 16 的新特性
        - componentDidCatch
        - 还有另一个仅监听子组件错误的
        - 还有两个属性用于支持 fiber 的（可以渲染到一半看看有没有更高优先级的属性）
    - key 有什么用
        - 用于 diff 算法的优化
            - diff 算法的大前提
            - 用于数据的排序优化
- 两个同级的 react 组件如何通信
    - 通过父组件
    - mbox 的 store（父组件 provider + 子组件 inject）
    - React 16 的 context？
    - React 16 的一个新属性
        - 子组件传递给父组件，父组件更新 state 然后修改子组件的 props，会导致子组件二次渲染，用于解决这个问题
- constructor 和 didMount 的区别
- 为什么请求不能放在 constructor 里面
    - state 可能在 constructor 里面或外面（一般习惯性写外面，但这样 state 都没有定义，setState 不方便控制内存）
    - 无 fiber 的情况下组件层层渲染会阻塞，可能请求回来了还没有初次 render 结束
- 多个请求同时发送
    - promise.all
    - 定义一个通用回调函数，外面有个变量来统计回调返回的数量，达到后再统一执行代码

- 主要是 React 技术栈，做 ssr 用了 next.js。我这边主要负责商业化部门的前端需求，包括多多进宝网站、商家后台广告相关部分，维护内部运营平台。

## pdd3面

- http1.1 keep-alive 和 http 2.0
- react 为什么支持写 html
- react 生命周期，ComponentWillReceiveProps 是否触发
    - 不会被触发，我特么自己试过的
- node js 的优点，为什么支持高并发（优势）
- 继承的实现（Class  function?）
- 性能优化
    - dom 的优化，vdom
    - 合成层
        - tansform opacity
    - 打包构建
    - 多个 js combone 成一个的好处？如果少于 4 个的话请求变大
- mobx 如何监控状态管理

- nodeJs 页面直出来优化性能
- 大促、交易（1）、秒杀（1） ect

## pdd4面

浏览器有限列表和无限列表
浏览器上拉和下拉的实现以及具体的事件
loading的时机和优化
dom的回收和释放
双端渲染及注意事项（HTML没有绑定事件，再渲染一遍）
react router，控制反转，context，providerContext，decorator，extend


## tt

1 Css bfc static 和 relative的区别
2 node require 和 import的区别，require看起来像注册在全局
3 写版本号排序的代码（及优化），时间复杂度
4 http 的 cache ，浏览器如何获取设置
5 react 的diff算法，key 的作用
6 是否喜欢看源码，看源码的感觉
7 不知道，没了解⊙▽⊙



垂直居中
arguments是数组吗？如果不是，怎么转换成数组？
event loop
手写节流函数

## oppo

https，http2.0
contentready和onreay和onload
劫持（广告）
Arender和wolf和数据引擎
前端监控
页面性能 performance.timing
接口层面的监控，状态码和时间

他们的业务情况
应聘岗位前一个人离职的原因?
