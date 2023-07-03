# SVG 实现

| --- | --- |
| 负责人 | @XGHeaven |
| 开始时间 | 2022-9-28 |
| 状态 | 进行中 |

---

随着时代的发展，SVG 的使用会越来越多，也不再仅仅限于一些矢量图的渲染，某些图标也开始尝试使用 SVG 去渲染。

WebF 想要更好的与社区接轨，降低整体的接入成本，SVG 是一个比较关键的实现节点。

WebF 也无需实现完整的 SVG 渲染实现，只需要在保证性能的情况下，实现前端依赖的核心能力即可。如无必要，勿增实体。

## 规范文件

- https://www.w3.org/TR/SVG2/
- https://svgwg.org/svg2-draft/

## Roadmap

由于目前 WebF 对 SVG 来说，能力缺失的东西比较多，按照以下步骤一步步补齐：

- 依赖
    - [ ] element namespace 概念引入
    - [x] 新 bridge https://github.com/openwebf/webf/pull/18
    - [x] CSS 动画能力 https://github.com/openwebf/webf/pull/11
- 基础，满足最常见的 Icon 渲染
    - [ ] SVG 解析
    - [ ] SVG 渲染实现，包括 Element 如何转 RenderObject，样式属性控制等等
    - [ ] 基础 SVG 元素的提供，主要包含图形类和组类
    - [ ] 基础 SVG Interface 提供，支持通过 JS 直接控制
    - [ ] 测试主流前端框架的使用渲染，包括 Vue/React/Svelte/Angular
- 交互、功能
    - [ ] SVG 交互事件实现
    - [ ] SVG 支持通过 CSS 控制，支持 CSS 的选择器
    - [ ] img 元素/CSS Background 支持异步加载网络 SVG
    - [ ] SVG 动画，支持 CSS 动画以及 `<animation>` 元素
- 进阶
    - [ ] 更多更复杂的 SVG 元素支持
    - [ ] 更多的 Attribute 支持
    - [ ] CSS url/filter 函数支持 #id 选择器
- 开发、优化
    - [ ] DevTools 能力优化
    - [ ] 性能评测，这部分可能一次完不成，后续补充
    - [ ] CanIUse 文档补齐

## 设计

### namespcaeURI

在标准中，一个元素的行为其实还受到 namespace 所定义，同一个元素 tagName 相同但 namespace 不同依旧是不同的元素。

所以需要补齐这部分能力：

- 支持 xmlns 属性
- 添加 Element.namespaceURI
- 添加 document.createElementNS 方法
- 添加 Attr.namespaceURI
- 添加 Element.setAttributeNS 方法

实现上需要修改整个 createElement 的逻辑，因为内部现在的元素创建仅仅只有一个 `tagName` 的概念，所以需要再补上 `namespaceURI` 的参数。

整体逻辑不复杂，就是改动点会很多。

### SVG 解析

目前看了下，HTML 现在的 parser 是支持解析 SVG 的，只不过需要修改下对应的命名空间而已。

解析完成之后，利用补齐的 createElementNS 方法创建 SVG 元素即可。

### SVG Interface

按照标准，所有的 SVG 元素都是继承自 SVGElement，那么在 Dart 侧和 JS 侧都需要实现相关的接口。

接口列表：https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model#svg_interfaces

规范文件：https://svgwg.org/svg2-draft/types.html#InterfaceSVGElement

#### 扩展

目前 WebF 的 Dart 的自定义组件应该还无法实现在 JS 侧进行动态的绑定类型，可以在后续中进行优化。降低这部分接口实现的难度和压力，只需要在 Dart 侧实现，就可以自动在 JS 侧创建并绑定对应的类型。

### SVG 渲染

整体渲染逻辑和普通的 Element 类似。

`SVGElement` -> `SVGRenderObject` -> `paint`

- `SVGElement` 就是 DOM 元素
- `SVGRenderObject` 就是与 SVGElement 所对应的 RenderObject，主要用于处理样式、事件等和 Element 相关的东西
- `paint` 将元素全部绘制出来

`SVGRenderObject` 有两种模式：

- Attach 模式，表示附属于某个 Element，能够处理并响应 style、事件等
- Detach 模式，独立运行，不绑定 Element 实例，无法响应 style，只能使用默认的样式，也无法响应事件。主要用于将 SVG 以背景的方式进行渲染。

### SVG 元素

基础图形元素：

- `<circle>`
- `<line>`
- `<path>`
- `<polygon>`
- `<polyline>`
- `<rect>`
- `<ellipse>`
- `<text>`
- `<image>` TBD

动画元素：

- `<animatin>`
- `<animateMotion>`
- `<animateTransform>`
- `<mpath>`
- `<set>`

容器元素：

- `<svg>`
- `<use>`
- `<a>`
- `<defs>`
- `<g>`
- `<marker>`
- `<mask>`
- ~`<missing-glyph>`~
- `<pattern>`
- `<switch>` TBD
- `<symbol>`
- `<clipPath>`
- `<textPath>`

渐变元素：

- `<linearGradient>`
- `<radialGradient>`
- `<stop>`

描述性元素：

> 以空实现为主，如果后续有需要，可以进一步进行实现

- `<desc>`
- `<metadata>`
- `<title>`

字体元素（废弃）：不实现

灯源元素：不实现

过滤器元素：TBD 建议后续看使用场景决定，目前暂时不需要实现

其他元素：

- `<script>` / `<style>` 暂不实现，但是需要能够兼容这种情况，避免 SVG 内部的标签影响到外面的元素


### SVG 事件

理论上来说，SVG 的事件其实和 DOM 的没有太大区别。主要的区别在于触摸类事件中 SVGElement 做 hitTest 的时候的处理。

又因为 SVG 的绘制是严格按照文档顺序的，所以不需要考虑排序的问题，整体上来讲比 DOM 的实现要简单一些。

目前 SVG 主要需要实现如下事件：

- [ ] 触摸类，比如 Touch/Mouse 等
- [ ] 键盘事件
- [ ] 焦点事件

### SVG 与 CSS

根据规范，SVG 的元素能够通过 CSS 进行样式控制，包括但不限于 class/style 等。

规范文件：https://www.w3.org/TR/SVG/styling.html#UsingPresentationAttributes

具体到实现，根据规范可以得知，所有的 `presentation-attributes` 都可以通过同名的样式进行控制，且拥有最低的优先级。也就是说只要有 style/class，`attribute` 就是失效的。

所以在实现中只需要将 `attribute` 作为优先级最低的样式处理即可，`getComputedStyle` 也会自动返回包含 `attribute` 的结果。

### SVG 动画

在 HTML 中的 SVG 动画主要有两个来源，分别是 CSS 的动画与 `<animation>` 标签。
大部分前端开发者主要使用 CSS 的方式，少部分 SVG 会使用动画标签。

- CSS 动画：走现在的渲染流程即可
- Animation 标签：转换成 CSS Animation 之后添加到父元素的 animation 内。

### 纯图片渲染

SVG 除了直接可以在 HTML 中使用，还可以作为 img、background 的值去渲染，此时 SVG 只需要渲染为一张图片即可。

此时这个状态下有挺多问题需要解决的。

#### 解析

由于原先的解析方式是通过 bridge 里面的 HTML Parser 实现的，所以理论上如果要实现直接将 SVG 渲染到图片上，那么需要经过：

`Dart String` -> `C String` -> `parse` -> `C GumboNode` -> `JSX Like Array` -> `Dart Node` -> `SVGRenderObject Detach Mode` -> `paint`

需要将字符串先拷贝到 C 中，然后再将解析后的结构化数据传递回 Dart 内。
而且 C 侧的 GumboNode 很难直接发送到 Dart 侧，所以这里需要将其先转换成 JSX Like Array 的一种数据结构。类似于：

```ts
{
    type: NativeString // 标签名
    props: NativeString[] // 属性的 pair
    childrenCount: number // 有几个子元素，按照 DFS 的顺序遍历
}[]
```

TBD: 不过这种方案，性能很差，还有待讨论。比较好的解决方案就是将 HTML 解析逻辑从 C 中迁移回 Dart 内。

## 开发体验

### DevTools 适配

- 支持 DevTools inspect
- 支持修改属性

### 性能测试

目前性能测试还没有一个很好的基础建设，可以顺便一起建设了。

- HTML 字符串的 SVG 性能，包括解析和运行性能
- JS 创建、删除的 SVG 性能
- Animation 性能
- JS 控制的 Animation 性能
- SVG 作为背景图片使用的时候的性能

### CanIUse

配合之前 andycall 提出的方案，尝试并实现这部分 caniuse 的能力。

> 我这边实现了一个库，可以直接将兼容性的数据渲染为类似于 caniuse 的网站。
> 后续可以尝试直接嫁接到这个库上。
> https://github.com/XGHeaven/canyouuse
