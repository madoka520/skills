# owo-lib UI Development Skill

基于 [owo-lib 官方测试代码](https://github.com/wisp-forest/owo-lib/tree/1.21/src/testmod)提取的正确用法。
写 UI 前必读，避免重复踩坑。

## 核心原则

1. **不要猜 API 行为** — 对照本文档或官方测试代码确认后再写
2. **vanilla Widget ≠ owo Component** — `TextBoxComponent`/`ButtonComponent` 继承 vanilla `EditBox`/`AbstractWidget`，**不会自动实现 owo 的 `Component` 接口**，需要 `(Component) (Object) widget` 强转才能加入 owo 布局
3. **让 owo 算尺寸** — 用 `Sizing.content()`/`Sizing.fill(n)`/`Sizing.fixed(n)`，不要手动算容器宽度
4. **不要改 z-level** — MC 渲染管线的 z 排序很脆弱，z=-200 会导致内容消失在背景后面

## 布局

### 容器创建

```java
// 垂直流（等价 flex-direction: column）
Containers.verticalFlow(Sizing.content(), Sizing.content())

// 水平流（等价 flex-direction: row）
Containers.horizontalFlow(Sizing.content(), Sizing.content())

// 滚动容器
Containers.verticalScroll(Sizing.content(), Sizing.fixed(50), innerFlow)
    .scrollbar(ScrollContainer.Scrollbar.vanilla())
```

### Sizing

```java
Sizing.fixed(100)    // 固定 100px
Sizing.content()     // 由子组件决定
Sizing.content(20)   // 内容 + 20px
Sizing.fill(50)      // 填充父容器的 50%
Sizing.fill()        // 填充全部可用空间
```

### 对齐

```java
container.horizontalAlignment(HorizontalAlignment.CENTER);
container.verticalAlignment(VerticalAlignment.CENTER);
```

### 间距

```java
component.margins(Insets.of(5));        // 外间距
container.padding(Insets.of(10));       // 内间距
container.gap(5);                        // 子组件间距（FlowLayout）
```

### 定位（叠加/浮层）

```java
// 绝对定位（不参与流式布局）
component.positioning(Positioning.absolute(x, y));
// 相对定位（0-100 百分比）
component.positioning(Positioning.relative(50, 0));
```

## 组件

### 按钮

```java
// owo 原生按钮
var btn = Components.button(Text.of("文字"), b -> { /* 点击 */ });
btn.sizing(Sizing.fixed(20));  // 单参数，宽高相同
// 或
btn.horizontalSizing(Sizing.fixed(95));  // 只设宽度
// 加入容器需要强转
searchRow.child((Component) (Object) btn);
// 自定义外观
btn.renderer(ButtonComponent.Renderer.flat(0x77000000, 0x77070707, 0xA0000000));
```

### 文本框

```java
var textBox = Components.textBox(Sizing.fixed(60));  // 只指定宽度
textBox.onChanged().subscribe(text -> { /* 文字变化 */ });
// 加入容器需要强转
searchRow.child((Component) (Object) textBox);
```

### 网格布局（矩形物品阵列）

```java
// GridLayout：子组件严格按行列矩形排布，物品网格必用（手工 horizontalFlow 分行曾出现阶梯错位）
// 参数顺序：(横向Sizing, 纵向Sizing, rows, columns) —— 行在前！
var grid = Containers.grid(Sizing.content(), Sizing.content(), rows, 9);
// child 的参数顺序同样是 (row, column)：
grid.child(component, i / 9, i % 9);
```

## 物品图标

```java
// owo 原生 ItemComponent（自动处理渲染和 tooltip）
var icon = Components.item(new ItemStack(Items.EMERALD, 16));
icon.setTooltipFromStack(true);   // 悬停显示原版 tooltip
icon.showOverlay(true);          // 显示数量角标（原版风格）
icon.stack(stack);               // 更新物品
// 加入容器不需要强转（已是 owo Component）
container.child(icon);
```

**不要在自定义组件里调 `context.renderItem()`（vanilla 路径）**。
ItemComponent 用自己的 BufferSource + `renderStatic` + `endBatch()` + 独立光照渲染，
这是官方验证可行的路径；需要自定义格子时直接 `extends ItemComponent` 再叠加 fill 描边/数量。

### 标签

```java
var label = Components.label(Text.literal("文字"));
label.color(Color.ofRgb(0xE2A3E3));
```

### 盒子（纯色块）

```java
var box = Components.box(Sizing.fixed(18), Sizing.fixed(18));
box.color(Color.ofRgb(0x37373B));
```

## 背景（Surface）

```java
// 深色面板（owo 自带）
container.surface(Surface.DARK_PANEL);

// 纯色
container.surface(Surface.flat(0x77000000));

// 带内嵌的面板
container.surface(Surface.panelWithInset(6));

// 模糊（需要 MUI/ModernUI）
container.surface(Surface.blur(3, 5));

// 无背景
container.surface(Surface.BLANK);
```

## 弹窗 / 模态覆盖层（官方组件）

```java
// owo 官方弹窗容器：全屏遮罩 + 子内容自动居中 + 点击外部关闭 + ESC 关闭
var content = Containers.verticalFlow(Sizing.fixed(220), Sizing.content());
content.surface(Surface.DARK_PANEL);           // 弹窗背景
var overlay = Containers.overlay(content);
overlay.surface(Surface.flat(0x90000000));     // 全屏遮罩（默认是 VANILLA_TRANSLUCENT）
root.child(overlay);                            // 挂到根容器，后挂的在最上层
// 关闭：overlay.remove() 或 root.removeChild(overlay)
```

- 鼠标事件按子组件**逆序**派发（后挂的先命中），overlay 自动拦截下层 UI 的点击
- 绝对定位（`Positioning.absolute`）的子组件不参与流式布局，不影响已排好的面板
- **覆盖层必须显式 `overlay.zIndex(300)` 之类抬高**：GUI 深度测试开启，
  物品图标渲染在 z=100（ItemComponent 硬编码）、自绘文字常在 z=200、
  vanilla tooltip 在 z=400；zIndex=0 的 overlay 遮罩会被底下界面的物品图标穿透。
  owo 的 tooltip 分发会把 hover 组件祖先链的 zIndex 累加，所以 overlay 抬高后
  其内部物品的 tooltip 仍在弹窗之上
- **屏幕级输入处理（override mouseClicked 等）要先判断 overlay 是否打开**：
  屏幕层拦截跑在 owo 分发之前，overlay 盖住的区域点击会被屏幕层吃掉，
  overlay 内的组件收不到事件（`if (!isPopupOpen() && myArea.contains(...)) { ... }`）
- vanilla Button（经 mixin wrapper）做绝对定位小按钮时出现过视觉与命中区错位，
  小热区按钮用「FlowLayout 定宽高 + 居中 Label + mouseDown 订阅」更可靠
- **不要在 super.render() 之后手动画 fill 做弹窗**——那条路径画不出背景；
  弹窗必须做成 owo 组件树（和主面板同一条渲染管线才有背景）

## 事件

### 鼠标点击（owo 组件）

```java
component.mouseDown().subscribe((x, y, button) -> {
    if (button == 0) { /* 左键 */ return true; }
    return false;  // 返回 false 让事件继续传播
});
```

### 鼠标点击（屏幕级别，覆盖 BaseOwoScreen）

```java
@Override
public boolean mouseClicked(double mouseX, double mouseY, int button) {
    // 先处理自定义区域
    if (myCondition) { /* 处理 */ return true; }
    // 不处理则交给 owo
    return super.mouseClicked(mouseX, mouseY, button);
}
```

### 键盘

```java
@Override
public boolean keyPressed(int keyCode, int scanCode, int modifiers) {
    if (keyCode == 69 && !(getFocused() instanceof EditBox)) {  // E 键关闭
        onClose();
        return true;
    }
    return super.keyPressed(keyCode, scanCode, modifiers);
}
```

## 自定义组件（BaseComponent）

```java
public class MyComponent extends BaseComponent {
    @Override
    public void draw(OwoUIDrawContext context, int mouseX, int mouseY, float partialTick, float delta) {
        // this.x, this.y = 组件屏幕坐标
        // this.width, this.height = 组件尺寸
        // 直接用 context.fill() / context.renderItem() / context.drawString()
        // 不需要管 z-level 或 pose push/pop（除非做缩放）
    }
}
```

### 设置尺寸

```java
// 在构造器中
this.sizing(Sizing.fixed(18), Sizing.fixed(18));
```

## 圆角渲染（需要 MUI）

```java
var exGr = new ExtendedGuiGraphics(guiGraphics);
exGr.setColor(fillColor);
exGr.fillRoundRect(x1, y1, x2, y2, radius);
exGr.setColor(strokeColor);
exGr.setStrokeWidth(1.5f);
exGr.strokeRoundRect(x1, y1, x2, y2, radius);
```

## 常见陷阱

| 陷阱 | 正确做法 |
|---|---|
| `button.sizing(w, h)` 编译报错 | ButtonComponent 继承 vanilla，用 `.sizing(Sizing.fixed(20))` 单参数或 `horizontalSizing()` |
| vanilla Widget 加入 owo 容器报错 | `(Component) (Object) widget` 强转（ClickableWidgetMixin 已注入 Component 接口） |
| 改容器宽度后 vanilla Widget 被拉伸 | vanilla EditBox 会填满可用空间，用 `Sizing.fixed()` 锁定宽度 |
| 带边框 EditBox 与其他元素高度不齐 | vanilla EditBox 边框渲染向四周各溢出 1px（visual = 尺寸+2），owo 的内容高 20 也是给它留的；对齐时把部件高度设为目标-2，或包一个目标尺寸的容器（居中 + 裁剪） |
| vanilla Button 尺寸/位置不可控 | 小按钮用「FlowLayout 定宽高 + 居中 Label + mouseDown 订阅」纯 owo 热区替代 |
| 物品网格出现阶梯/错位 | 用 `Containers.grid`；物品渲染走 `ItemComponent`，不要用 vanilla `renderItem` |
| `z=-200` 物品消失 | 不要改 z-level，用其他方式处理遮挡 |
| 组件收起后仍占空间 | 用 `positioning(Positioning.absolute())` 脱离流式布局，或手动在 render() 里画 |
| owo 事件坐标 vs draw 坐标不一致 | owo 的 `draw()` 收到的是全局屏幕坐标，`mouseDown()` 的 x/y 也是全局坐标 |
