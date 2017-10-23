#在HTML中使用javascript
###script标签的defer和async
1. 没有 defer 或 async，浏览器会立即加载并执行指定的脚本，“立即”指的是在渲染该 script 标签之下的文档元素之前，也就是说不等待后续载入的文档元素，读到就加载并执行。
2. async,加载和渲染后续文档元素的过程将和 script.js 的加载与执行并行进行（异步）。
3. defer,加载后续文档元素的过程将和 script.js 的加载并行进行（异步），但是 script.js 的执行要在所有元素解析完成之后，DOMContentLoaded 事件触发之前完成。
![async,defer](http://segmentfault.com/img/bVcQV0)
*蓝色线代表网络读取，红色线代表执行时间，这俩都是针对脚本的；绿色线代表 HTML 解析* 
 
###比较:

- efer 和 async 在网络读取（下载）这块儿是一样的，都是异步的
- 它俩的差别在于脚本下载完之后何时执行，显然 defer 是最接近我们对于应用脚本加载和执行的要求的
- 关于 defer，它是按照加载顺序执行脚本的,async则是乱序执行,不管声明的顺序如何，只要加载完了就会立刻执行

书签:p38
