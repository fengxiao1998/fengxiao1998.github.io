---
title: 关于chrome拓展开发的实践与思考(五)
date: 2022-05-12 09:40:18
tags: [chrome插件,chrome拓展]
---

## 前言：

拓展开发所需要的主要功能大致已经叙述完毕，在这章想进行细节补充以及个人思考。

## permissions权限：

这是放置在manifest文件中的配置项，我们的拓展可以使用chrome提供的api来实现部分功能，但使用的api需要在manifest中进行报备，这就是permissions的主要功能。

**tips:**不同文件可以使用的权限不一，background可以使用绝大多数权限，而content-script只能使用较少的权限，比较content-script是注入页面的脚本，还是要从安全性进行考虑。如果你在使用权限时发现未生效，不妨去看看你是否注册了该权限或者该权限是否支持在你的文件中运行。

## 个人思考

至此笔者想说的知识点都已结束，不妨说说自己的想法，笔者在开发中使用的一直是原生的html,js,css，如果你的拓展是轻量级的，倒也还好。如果是中小型的拓展，不妨试试引入jquery文件，chrome是支持的。如果是比较庞大复杂的拓展，原生开发就显得有些力不从心，不妨可以尝试vue+scss开发。开发永远是要基于自身考量，一昧的模仿可不太行，另外，在2023年manifest_version: 2将被废弃，建议读者多看看版本3的文档，不断学习。

版本3官方文档：https://developer.chrome.com/docs/extensions/mv3/manifest/