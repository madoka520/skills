---
name: madoka-ui-style
description: 我的 CSS UI 编程风格（提炼自 madohomu-sanctuary 项目）：粉色治愈系 + 毛玻璃 + 微动效的 Vue SFC 样式写法。在为新组件/页面写 style、做 UI 迭代、写任何 Vue 组件样式、用户提到「用我的风格」「按我的 UI 风格写」「写一个 xx 组件」或要求粉色/毛玻璃/梦幻风格界面时使用 —— 即使没明说风格也要用。
---

# madoka-ui-style —— 我的 CSS UI 风格

一套从 madohomu-sanctuary（魔法少女小圆&晓美焰圣堂）项目中蒸馏出的 UI 编程风格。
核心气质：**粉色治愈系 × 毛玻璃 × 到处都有细腻微动效 × 注释带二次元魂**。
写出来的界面应该像「魔法少女发动魔法」：安静时柔和半透明，交互时绽放光晕。

## 默认形态（2026-09 定稿，优先级最高）

- **新组件默认直角**：`border-radius: 0`，靠细边框成形；圆角只在用户明确要求时使用
- **卡片默认不加背景填充**：透明底 + 直角细边框（如 `border: 1.5px solid rgba(255, 183, 197, 0.45)`），交互态只变化边框颜色 + 光晕投影
- 禁用 cursor:pointer、文字默认不可选、必有过渡反馈这三条 app 化习惯**不限本风格，已独立成 `web-app-ui-habits` skill**（所有 web 项目适用）；本 skill 只管风格令牌与形态
- 毛玻璃/大圆角属于旧版气质：仅存量组件保留，迭代时不主动扩散到新组件

## 设计令牌（Design Tokens）

### 配色 —— 粉是主色，紫是副色，一切阴影都带粉

```less
// 粉色系（小圆）——每个组件在 style 块顶部定义自己的变量
@pink-primary: #ffb7c5;   // 主粉（浅，用于边框、背景块）
@pink-dark: #ff85a2;      // 深粉（用于文字、强调）
@pink-hot: #ff69b4;       // 热粉（用于 focus 发光、渐变中段）
@pink-cyber: #ff79c6;     // 赛博粉（用于科技感卡片、扫描线）
@pink-text: #ff8fab;      // 标题文字粉

// 紫色系（焰）——副色，与粉搭配出现
@purple: #a77bc7;
@purple-deep: #b388eb;

// 半透明背景（毛玻璃体质）
@glass-pink: rgba(255, 240, 245, 0.7);        // 亮色弹窗底
@glass-card: rgba(255, 183, 206, 0.06);       // 暗色卡片底（悬浮 0.12 / 激活 0.15）
@glass-dark: rgba(0, 0, 0, 0.3);              // 暗色条带（悬浮 0.5）

// 文字
//  暗底上用 white / rgba(255,255,255,0.85)，次要信息 opacity: 0.6
//  输入框未激活 #9e9e9e / #bdbdbd，激活变粉
```

规则：
- 阴影永远是**粉色调**的，不用纯黑：`box-shadow: 0 8px 32px rgba(255, 183, 197, 0.3)`
- 渐变用粉色系两端 + 热粉中段：`linear-gradient(90deg, pink 0%, #ff69b4 50%, pink 100%)`
- 品牌文字用「双色渐变裁切」：粉紫各管一个词

### 发光阴影 —— 多层叠加，透明度递减

```less
// 悬浮发光：3~4 层，由近到远
box-shadow:
  0 0 4px #ff7ec9bb,
  0 0 8px #ff7ec999,
  0 0 12px #ff7ec999;

// 常规浮起：柔和粉阴影
box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);

// 文字发光（白芯 + 粉晕）
text-shadow: 0 0 2px white, 0 1px 2px rgba(255, 120, 180, 0.5);
```

可以抽成 mixin 复用：

```less
.box-shadow(@color: rgba(255, 255, 255, 0.2); @s1: 30px; @s2: 60px; @s3: 110px) {
  box-shadow: 0 0 @s1 @color, 0 0 @s2 @color, 0 0 @s3 @color;
}
```

## SFC 结构与样式组织

固定顺序：`<template>` → `<script setup lang="ts">` → `<style scoped lang="less">`。

- **永远 scoped**，需要穿透时用 `:deep(selector)`
- **LESS 嵌套优先**：状态用 `&:hover`、`&.is-checked` 嵌在宿主选择器里，不铺平
- 样式块顶部先定义本组件的 less 变量（`@pink-primary` 等），不引全局主题文件
- JS 可变的值用 CSS 变量 + `v-bind()` 下发：`--primary-color: v-bind(color)`
- 跨层级控制（如歌词换色）通过 CSS 变量注入：`--lyric-base-color: #ff6b8a`
- 布局用全局工具类（项目 `layout.less` 已有）：`.flex` `.flex-center` `.flex-col` `.w-full` `.h-full`，模板里直接挂 class，不在 style 里重复写
- 图标统一 MDI：`<i class="mdi mdi-heart" />`
- 响应式尺寸用 `clamp()`：`width: clamp(260px, 22vw, 520px)`、`font-size: clamp(18px, 2.5vw, 1.8rem)`

### 命名 —— BEM-lite

- 组件前缀 + `__` 元素：`.dialog__overlay`、`.card__inner`、`.madoka__input`
- `-` 连字符普通类：`.theme-option`、`.peek-panel`
- `--` 修饰变体：`.card-overlay--error`
- 状态类用 `is-*` / 语义词：`.is-checked` `.is-active` `.is-loading` `.selected` `.liked`，由 JS 切换 class 驱动样式
- 变体系统：`.variant-text` / `.variant-outlined` / `.variant-plain`

## 动效 —— 灵魂所在

### 曲线与时长（背下来）

```less
// 标准：日常 hover、状态切换
transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);

// 弹跳：勾选、弹出、回弹（ overshoot > 1）
cubic-bezier(0.175, 0.885, 0.32, 1.275)
cubic-bezier(0.34, 1.56, 0.64, 1)

// 急停顺滑（expo-out）：大面板、弹窗进出
cubic-bezier(0.1, 1, 0.4, 1)
cubic-bezier(0.16, 1, 0.3, 1)
```

时长梯度：微交互 0.15–0.25s ／ 常规 hover 0.3s ／ 面板抽屉 0.4–0.55s ／ 氛围循环动画 8s infinite。

### 必做的微交互

（「每个可交互元素都有 transition」「hover 反馈」两条通用规则在 `web-app-ui-habits`,此处只列风格特有的微交互形态）

1. **伪元素做装饰线**：下划线用 `::after { transform: scaleX(0); transform-origin: center }`，hover/focus 时 `scaleX(1)` 或 `scaleX(0.6)`；链接下划线用 `width: 0 → 100%`
2. **focus-within 容器级聚焦**：输入框聚焦时整个容器变色 + 图标 `scale(1.1) rotate(5deg)`
3. **pop-in 弹入**：勾选图标 `scale(0)→1` + 弹跳曲线，0.2s
4. **氛围动画**：`float` / `float-slow` / `twinkle` / `pulse` 等无限循环，缓慢位移或呼吸透明度，给页面「活着」的感觉

### JS + CSS 混合动画（进出场）

不用 Vue 内建 Transition 的场景，自己切 class + rAF：

```ts
// 进场：先摆好初始位置，下一帧加 enter-active 触发过渡
requestAnimationFrame(() => el.classList.add('enter-active'))
// 离场：加 leave-active，监听 transitionend 后清理 class、还原内联样式、把元素挪回屏幕外
el.addEventListener('transitionend', onEnd)
// 隐藏元素不是 display:none，是停在 left: -99999px
```

弹窗进场从鼠标位置缩小态 `scale(0.1)` 过渡到居中 `scale(1)`。
点击涟漪（ripple）用 JS 动态创建 span，`transition: transform 0.6s ease, opacity 0.6s ease`，`transitionend` 后自我移除。

## 毛玻璃三板斧

```less
.glass {
  background: rgba(255, 240, 245, 0.7);          // 半透明底
  backdrop-filter: blur(15px);                    // 毛玻璃（记得 -webkit- 前缀）
  -webkit-backdrop-filter: blur(15px);
  border: 1px solid rgba(255, 255, 255, 0.4);     // 微光描边
  box-shadow: 0 8px 32px rgba(255, 183, 197, 0.3); // 粉色投影
}
```

背景图卡片要**把背景塞进 `::before` 并 `z-index: -1`**，配合宿主 `position: relative; isolation: isolate`——这样 blur 滤镜不影响文字，hover 时单独给伪元素加 `filter: blur(8px) brightness(0.8)`。

## 注释风格 —— 中文，解释为什么，带点魂

- 全部中文注释，写「为什么」不写「是什么」：`// 必须是 relative 才能让伪元素定位`
- 关键技巧加 emoji 前缀：`// ✨ 核心增强：只添加毛玻璃和微光边框，不改动布局逻辑`、`// 💫 下面是淡入淡出动画`、`// 👉 内部还能滚，就放行`
- 字体/圆角出处玩梗标注：`border-radius: 29px; // LR_小さな狐の妖精`
- 踩坑警告直接写结论：`//这个await去掉会导致弹窗关闭的时候突然跑到左下角`

## 反模式（不要做）

- ❌ 纯黑/纯灰阴影 —— 阴影必须带色调（粉或场景色）
- ❌ 新组件默认使用圆角/背景填充 —— 默认形态是直角 + 透明底 + 细边框（见「默认形态」节），圆角与毛玻璃仅在明确要求时使用
- ❌ 铺平的 CSS（不用嵌套）、全局裸选择器
- ❌ 干巴巴的英文注释

## 完整组件配方

写新组件前，先读同类的参考实现（含完整 template/script/style）：

- 弹窗/遮罩/抽屉 → 读 [references/components.md](references/components.md) 的「Dialog」「Mask」节
- 卡片（留言卡/玻璃卡/赛博卡）→ 「Card」节
- 按钮（变体系统/渐变描边/下划线）→ 「Button」节
- 输入框/勾选框 → 「Form」节
- 时间轴/悬浮卡片定位 → 「Overlay 定位」节

参考实现都是真实项目组件的节选，照着结构写，样式细节按上面的 token 自行组合。
