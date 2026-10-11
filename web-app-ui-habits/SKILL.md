---
name: web-app-ui-habits
description: Web 界面的 app 化习惯(不限框架、不限主题、不限项目)。写任何 web 前端界面、做 UI 迭代、review 界面交互时使用:文字默认不可选(入口全局设定)、不写 cursor:pointer、可交互元素必有过渡反馈、界面文案用中文。
---

# Web 界面的 app 化习惯

界面是 app 不是网页,所有规则由此推导。

## 文字默认不可选

- 在项目入口 `index.html` 全局设 `* { user-select: none }` 并对 `input, textarea` 放开 `user-select: text` —— 比逐组件挂可靠(命名视图/Teleport/弹层全覆盖)
- 只有明确给用户复制的区域(代码块、错误详情、路径)才在组件里局部放开 `user-select: text`
- 组件库组件自带 `user-select: none` 保留(要在任意宿主下自洽);业务组件里不再写这条

## 不写 cursor:pointer

- 交互元素一律保持系统默认光标;可点击性靠 hover/active 的样式变化(边框、光晕、位移)表达

## 必有过渡反馈

- 每个可交互元素都有 transition,**没有过渡的状态突变是反模式**
- hover 有反馈(scale/translateY/发光增强等),active 至少 `opacity: 0.7` 或加深底色
- 具体的时长与曲线参数由项目风格决定(如 madoka-ui-style 的曲线令牌)

## 文案中文

- 界面文案、占位符、提示语全部中文
