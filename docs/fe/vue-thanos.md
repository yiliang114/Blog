### 写在前面

灭霸打响指的消失效果。效果来源于 Google 搜索“灭霸” 或者 “thanos”。算是蹭热度的一个 `Feature`, 我通过 `F12` 试图去查看是如何实现的，也抠了一些音频、图片资源下来。后来在 github 上找到了一个现有的项目 [Thanos_Dust](https://github.com/lichking24/Thanos_Dust), 所以参考了部分它的代码。 其实它的代码已经算比较完善了，在它的基础上，我用 vue 来写了一下，加了一些英雄，修复了一些 bug ，加了一些效果之类的。

![](https://user-gold-cdn.xitu.io/2019/5/4/16a8266169f61670?w=2808&h=1410&f=png&s=1070831)

![](https://user-gold-cdn.xitu.io/2019/5/4/16a8266527ad7ad6?w=2744&h=1400&f=png&s=658305)

## demo

- 点击一下手套，伴随音效和响指的动画，会有一半的英雄消失。
- 消失之后，再点一下，消失的英雄又会回来。

可以点击下面的链接体验一下

[demos](https://yiliang114.github.io/vue-thanos-snap/index.html)

![](https://user-gold-cdn.xitu.io/2019/5/4/16a8275b08db2b4d?w=420&h=267&f=gif&s=1552754)

## 细节

- 随机选取一半的英雄，是通过下面的算法进行选取的：
  ```
  arr.sort(function() {
    return 0.5 - Math.random();
  });
  ```
- 被选中的英雄灰飞烟灭的效果解释：

  1. 使用 [html2canvas](http://html2canvas.hertzen.com/) 库将每一个英雄所在的 `dom` 节点渲染为一个 `canvas` 节点
  2. 通过 [generateFrames](https://github.com/yiliang114/vue-thanos-snap/blob/master/src/components/Main.vue/#L117) 方法，将整块的 `canvas` 画布图像按像素分割成许多块
  3. 创建一个跟选中的英雄所在的 `dom` 节点同一个位置、同样的大小的容器覆盖原 `dom` 节点
  4. 把第二步创建的块绘制到新的画布上，并都通过 `appendChild` 方法添加到第三步创建的父容器中
  5. 随机设置每一块的 `rotate` 角度和 `translate` 像素，就能完成灰飞烟灭的效果
  6. 将被覆盖的英雄的 `dom` 节点设置为不可见的，就完成了响指操作。

- 翻转时间，英雄又回来的效果是将原来的 `dom` 节点设置为可见的，并加了回复动画。（ `google` 的原版恢复动画是将 `color` 设置为 `green` ，因为这里没什么文字效果并不明显，就设置成了 `background-color` ）

## 最后

整个过程其实跟 vue 没什么关系，无论用什么前端技术栈都可以。以前也没有接触过 canvas ，似乎觉得还有点意思， 后面可能慢慢还会做一些改动，会继续学习 canvas 。最后附上[ github 地址](https://github.com/yiliang114/vue-thanos-snap).
