# WebF API Avaiable Plugin

## Abstract

WebF support a subset of W3C standard features, including Web API and CSS. But our users didn't know the exactly properties when implemented or not supported. 

To help developers to use webf more easily, we needs to develop a code editor plugin to identifying the unsupported W3C features in front-end developer's project.

## Plan

To make it works, we can use ESlint and CSSLint to static analyze user's code when editing, and find these properties which are not supported by WebF, then we can collect these info and send to vscode editor to notify our users.

![img](../images/code_helper.png)

