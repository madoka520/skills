# 组件配方（真实项目节选）

以下均节选/整理自 madohomu-sanctuary，保留原始写法。写同类组件时照此结构。

## Dialog（弹窗：遮罩 + 鼠标位置缩放进场）

结构：`MadokaMask`（teleport 到 body + 淡入淡出遮罩）包住 overlay。

```vue
<style scoped>
.mask {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 999999;
}
/* 💫 下面是淡入淡出动画 */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
.content {
  z-index: 1000000;
  position: fixed;
}
</style>
```

overlay 本体（`lang="less" scoped`）：默认停在屏幕外 `left/top: -99999px`，`transform: scale(0.1); opacity: 0`；JS 在 `requestAnimationFrame` 后加 `.enter-active`：

```less
.dialog__overlay {
  display: flex;
  flex-direction: column;
  position: fixed;
  transform: scale(0.1);
  border-radius: 29px; // LR_小さな狐の妖精
  corner-shape: superellipse(1.333);
  left: -99999px;
  top: -99999px;
  // ✨ 核心增强：只添加毛玻璃和微光边框，不改动布局逻辑
  backdrop-filter: blur(15px);
  -webkit-backdrop-filter: blur(15px);
  border: 1px solid rgba(255, 255, 255, 0.4);
  box-shadow: 0 8px 32px rgba(255, 183, 197, 0.3);
  background: white;
  overflow: hidden;
  opacity: 0;

  header {
    padding: 20px;
    font-weight: bold;
    color: #ff8fab;
  }
  main {
    padding: 0 20px;
    flex: 1;
    overflow: auto;
    display: flex;
    align-items: center;
    width: 100%;
    min-height: 0; // flex 子项允许收缩，滚动才能生效
  }
  footer {
    display: flex;
    justify-content: end;
    padding: 20px;
    gap: 10px;
  }
}

.dialog__overlay.enter-active {
  left: 50% !important;
  top: 50% !important;
  transform: translate(-50%, -50%) scale(1) !important;
  opacity: 1 !important;
  transition: all 0.3s cubic-bezier(0.1, 1, 0.4, 1);
}

.dialog__overlay.leave-active {
  opacity: 0 !important;
  transition: all 0.3s cubic-bezier(0.1, 1, 0.4, 1);
  pointer-events: none;
}
```

进场初始位置取鼠标坐标（`useMouse()`），`left = mouse.x - w * 5` 制造「从鼠标处绽放」的观感；离场监听 `transitionend` 清理 class 并还原内联样式。

## Card（玻璃留言卡：伪元素背景层 + isolation）

```vue
<template>
  <div class="card__wrapper">
    <div class="card" :style="{ '--bg-image': `url(${bgUrl})` }">
      <div class="card__inner">…内容…</div>
    </div>
  </div>
</template>
```

```less
.card__wrapper {
  padding: 20px 5px 5px;
  opacity: 0.7; // 静止时半透明，悬浮点亮

  &:hover {
    opacity: 1;
    .card::before {
      // 悬浮时触发毛玻璃效果
      filter: blur(8px) brightness(0.8);
    }
  }

  .card {
    width: clamp(260px, 22vw, 520px);
    height: clamp(140px, 55vh, 700px);
    overflow: hidden;
    position: relative; // 必须是 relative 才能让伪元素定位
    isolation: isolate; // 确保内容层级在伪元素之上
    background: rgba(255, 255, 255, 0.35);
    backdrop-filter: blur(8px);
    border-radius: 18px;
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
    color: white;

    // 使用伪元素处理背景，这样 blur 滤镜才不会影响到文字
    &::before {
      content: '';
      position: absolute;
      inset: 0;
      z-index: -1;
      background-image:
        linear-gradient(rgba(0, 0, 0, 0.35), rgba(0, 0, 0, 0.35)),
        var(--bg-image);
      background-size: cover;
      background-position: center;
      transition: filter 0.3s ease, transform 0.3s ease;
    }

    .card__inner {
      width: 100%;
      height: 100%;
      overflow-y: auto;
      padding: 14px;
      display: flex;
      flex-direction: column;
      gap: 12px;
    }
  }

  // 回复列表的层次感：左侧装饰线 + 缩小一级
  .replies-container {
    margin-top: 16px;
    padding-left: 12px;
    border-left: 2px solid rgba(255, 255, 255, 0.2); // 左侧装饰线
    display: flex;
    flex-direction: column;
    gap: 16px;

    :deep(header) {
      margin-top: 8px;
      transform: scale(0.9); // 让回复者的头像稍小一点
      transform-origin: left center;
      opacity: 0.9;
    }
  }
}
```

## Card（赛博风玻璃卡：clip-path 切角 + 扫描线）

```less
@cyber-pink: #ff79c6;
@glass-bg: rgba(255, 255, 255, 0.05);

.link-card {
  position: relative;
  padding: 30px;
  transition: all 0.4s cubic-bezier(0.16, 1, 0.3, 1);
  overflow: hidden;

  &:hover {
    transform: scale(1.02) skewX(-1deg);
    .card-glass {
      border-color: @cyber-pink;
      box-shadow: 0 0 30px rgba(255, 121, 198, 0.2);
    }
    .scan-line {
      top: 100%;
      opacity: 1;
      transition: top 0.8s ease-out, opacity 0.2s;
    }
  }

  .card-glass {
    position: absolute;
    inset: 0;
    background: @glass-bg;
    backdrop-filter: blur(20px);
    border: 1px solid rgba(255, 255, 255, 0.1);
    z-index: -1;
    clip-path: polygon(0 0, 95% 0, 100% 15%, 100% 100%, 5% 100%, 0 85%);
  }

  .scan-line {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 3px;
    /* 强化亮度，加入白色核心 */
    background: linear-gradient(90deg,
      transparent 0%, @cyber-pink 20%, #ffffff 50%, @cyber-pink 80%, transparent 100%);
    opacity: 0;
    box-shadow: 0 0 15px @cyber-pink, 0 0 5px #ffffff;
    pointer-events: none;
  }
}
```

## Button（变体系统：CSS 变量 + v-bind + YIQ 自动对比色）

```vue
<template>
  <madoka-ripple :color>
    <div class="madoka-btn flex-center" :class="{ [`variant-${variant}`]: true }" tabindex="0">
      <span><slot>{{ text }}</slot></span>
    </div>
  </madoka-ripple>
</template>

<script setup lang="ts">
const props = withDefaults(defineProps<{ color?: string; variant?: Variant; radius?: number }>(), {
  color: 'black',
  radius: 4,
})
function isLightColor(colorStr: string): boolean {
  const [r, g, b] = ColorUtils.getRGB(colorStr) ?? [255, 255, 255]
  return (r * 299 + g * 587 + b * 114) / 1000 >= 128
}
const textColor = computed(() => (isLightColor(props.color) ? 'black' : 'white'))
</script>

<style scoped lang="less">
.madoka-btn {
  --primary-color: v-bind(color);
  --text-color: v-bind(textColor);
  padding: 5px 16px;
  border-radius: v-bind("radius + 'px'");
  gap: 6px;
  user-select: none;
}

.variant-text {
  background: transparent;
  border: none;
  color: var(--primary-color);
  &:hover { background: rgba(0, 0, 0, 0.04); }
}
.variant-outlined {
  background: transparent;
  border: 1px solid var(--primary-color);
  color: var(--primary-color);
  &:hover { opacity: 0.9; }
}
.variant-plain {
  background: var(--primary-color);
  color: var(--text-color);
  border: none;
  &:hover { opacity: 0.9; }
}
</style>
```

主行动按钮（如登录）的粉色版本：hover 弹跳 + 三层发光，active 加深：

```less
.login {
  width: 252px;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  user-select: none;
  background: #ff93d2;
  color: #fff;
  border-radius: 12px;
  transition: all 0.3s;

  &:hover {
    background: #ff7ec9;
    animation: fr_btn_hover 0.4s;
    box-shadow: 0 0 4px #ff7ec9bb, 0 0 8px #ff7ec999, 0 0 12px #ff7ec999;
  }
  &:active { background: #e45dab; }
}

@keyframes fr_btn_hover {
  0% { transform: scale(1); }
  20% { transform: scale(1.04); }
  100% { transform: scale(1); }
}
```

## Form（下划线输入框：::before 基线 + ::after 粉色绽放）

```less
.madoka__input {
  position: relative;
  display: flex;
  align-items: center;
  width: 100%;
  border-radius: 18px;
  transition: all 0.4s cubic-bezier(0.25, 0.46, 0.45, 0.94);
}

/* 下划线：基础灰 */
.madoka__input::before {
  content: '';
  position: absolute;
  bottom: 0;
  left: 10px;
  right: 10px;
  height: 1px;
  background-color: rgba(189, 189, 189, 0.7);
  z-index: 1;
}

/* 粉色魔法动画下划线 */
.madoka__input::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 10px;
  right: 10px;
  height: 2px;
  background: linear-gradient(90deg, pink 0%, #ff69b4 50%, pink 100%);
  box-shadow: 0 0 6px rgba(255, 105, 180, 0.5);
  transform: scaleX(0);
  transform-origin: center;
  transition: transform 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  z-index: 2;
}

/* Focus：粉色绽放 */
.madoka__input:focus-within {
  background: rgba(255, 240, 245, 0.5);
  box-shadow:
    0 4px 20px rgba(255, 105, 180, 0.3),
    0 0 0 1px rgba(255, 105, 180, 0.2);
}
.madoka__input:focus-within::after { transform: scaleX(1); }
.madoka__input:focus-within .icon .mdi {
  color: #ff69b4;
  transform: scale(1.1) rotate(5deg);
}

/* Hover：下划线预览 60% */
.madoka__input:hover:not(:focus-within)::after { transform: scaleX(0.6); }
```

Checkbox：原生 input 铺满透明接收点击（`opacity: 0; z-index: 2`），视觉件单独画，勾选图标 pop-in 弹跳：

```less
@pink-primary: #ffb7c5;
@pink-dark: #ff85a2;

.madoka-checkbox {
  .checkbox-inner {
    width: 20px;
    height: 20px;
    border: 2px solid @pink-primary;
    border-radius: 6px;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.3s;
    .mdi {
      color: white;
      font-size: 14px;
      animation: pop-in 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275);
    }
  }
  &.is-checked .checkbox-inner { background-color: @pink-primary; }
}

@keyframes pop-in {
  0% { transform: scale(0); }
  100% { transform: scale(1); }
}
```

## Overlay 定位（悬浮卡片：getBoundingClientRect + 边界钳制）

时间轴月份悬浮出日历卡：卡片默认停 `-10000px`，显示时 `nextTick` 后测量并定位，超界强制修正：

```ts
let targetLeft = monthRect.left - wrapperRect.left + monthRect.width / 2
const cardHalfWidth = cardRect.width / 2
// 如果太靠左或太靠右，强制修正
const minLeft = cardHalfWidth + 10
const maxLeft = wrapperRect.width - cardHalfWidth - 10
if (targetLeft < minLeft) targetLeft = minLeft
else if (targetLeft > maxLeft) targetLeft = maxLeft
```

卡片本体是暗色毛玻璃 + 白色双层辉光 + 纯 CSS 小箭头：

```less
.date-card {
  position: absolute;
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 4px;
  padding: 8px 10px;
  border-radius: 12px;
  z-index: 100000;
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(14px);
  -webkit-backdrop-filter: blur(14px);
  box-shadow:
    0 0 20px rgba(255, 255, 255, 0.25),
    0 0 40px rgba(255, 255, 255, 0.15);
  color: white;
  transition: opacity 0.2s ease, transform 0.2s ease;

  &::after {
    content: '';
    position: absolute;
    left: 50%;
    top: 100%;
    transform: translateX(-50%);
    border-left: 8px solid transparent;
    border-right: 8px solid transparent;
    border-top: 8px solid rgba(0, 0, 0, 0.7);
  }
  .day:hover { background: rgba(255, 187, 221, 0.35); }
}
```

## 导航（hover 唤醒：show-with-hover + CSS 变量换肤）

```less
.show-with-hover {
  opacity: 0;
  transition: opacity 0.2s;
}
.nav {
  position: fixed;
  transition: background 0.3s ease, box-shadow 0.3s ease;
  &:hover,
  &:focus-within {
    background: rgba(255, 200, 220, 0.8);
    box-shadow: 0 4px 12px rgba(255, 128, 180, 0.2);
    backdrop-filter: blur(6px);
    .show-with-hover { opacity: 1; }
    /* LR_小さな狐の妖精 导航栏悬浮时歌词加深 */
    .desk-lrc {
      --lyric-base-color: #ff6b8a;
      --lyric-glow-color: rgba(255, 107, 138, 0.8);
      --lyric-dim-color: rgba(200, 100, 120, 0.3);
    }
  }
}

// 品牌字：粉/紫双色渐变裁切
.mado {
  background: linear-gradient(135deg, #ff85a2, #ffb6c1);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow: 0 0 2px white, 0 1px 2px rgba(255, 120, 180, 0.5);
}
.homu {
  background: linear-gradient(135deg, #b388eb, #a470d1);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}
```

## 氛围动画（页面永远在轻轻动）

```less
@keyframes float {
  from { transform: translateY(0); }
  to { transform: translateY(-20px); }
}
@keyframes float-slow {
  from { transform: translate(0, 0); }
  to { transform: translate(30px, 40px); }
}
@keyframes stars-twinkle {
  0%, 100% { opacity: 0.3; }
  50% { opacity: 1; }
}
@keyframes pulse {
  0%, 100% { transform: scale(1); opacity: 1; }
  50% { transform: scale(1.05); opacity: 0.8; }
}

// 用法：animation: float 6s ease-in-out infinite alternate;
```

更强交互感时上 Canvas 粒子（星星/爱心贴图），点击按钮或鼠标划过时 `generateRact(image, x, y, w, h, 0.8, 20)` 喷发。
