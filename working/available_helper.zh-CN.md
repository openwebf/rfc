# WebF API 可用插件

## 前言

WebF 支持 W3C 标准功能的子集，包括 Web API 和 CSS。 但是我们的用户在实施或不支持时并不知道确切的属性。

为了帮助开发者更方便地使用 webf，我们需要开发一个代码编辑器插件来识别前端开发者项目中不支持的 W3C 特性。

## 计划

为了让它工作，我们可以在编辑时使用 ESlint 和 CSLint 静态分析用户的代码，然后找到那些不支持的属性，再通过 VSCode 插件 API 反馈给用户。

![img](../images/code_helper.png)

实现整个工作的核心是一个 CLI 插件，它内置了 ESlint 以及 StyleLint.

使用 ESLint 来完成对 JavaScript/TypeScript 的验证，以及使用 StyleLint 来完成对 CSS/SCSS/Less 的验证。

## 输入数据结构设计

插件通过输入这个 JSON，来获取检查必要的规则内容。

> 第一期先手动维护一下，二期做自动生成

```json
{
    "JavaScript": [
        {
            "property": ""
        }
    ],
    "CSS": [
        
    ]
}
```

## 如何和 Vue 的配合

Vue 使用 .vue 文件进行开发，一个 .vue 文件中同时包含了 HTML/CSS/JavaScript 的代码，直接基于 .vue 文件进行分析并不现实。

不过好在 Vue 的构建工具 Vue-cli/Vite 都提供了 eslint/stylelint 的支持，只需要安装对应的插件，然后通过 eslint/stylelint 来自定义我们想要的规则。

1. https://vue-loader.vuejs.org/guide/linting.html#eslint
2. https://www.npmjs.com/package/vite-plugin-stylelint
3. https://www.npmjs.com/package/vite-plugin-eslint

## 如何自定义 ESLint 规则




## 相关资料

1. https://github.com/stylelint/awesome-stylelint
