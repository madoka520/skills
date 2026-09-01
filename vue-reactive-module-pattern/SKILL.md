---
name: vue-reactive-module-pattern
description: 'Generate or refactor Vue 3 script setup reactive code into strict module blocks using reactive + IIFE pattern. Use when writing page state, table/query/dialog control modules, splitting large Vue files, standardizing reactive code structure, reducing nesting, or following the team rule that each reactive module is defined as const Xxx = (() => { const f = {...}; const s = reactive({ ...state, ...f }); return s; })(). 支持 Vue 响应式模块、页面状态分块、列表查询模块、弹窗模块、表单控制模块重构。'
argument-hint: '模块名称、职责、状态字段、方法、是否需要初始化 / onMounted'
user-invocable: true
---
# Vue 响应式模块规范

## 包管理器约束

- 涉及依赖安装、移除、更新、脚本执行和项目初始化时，必须使用 `pnpm`
- 使用 `pnpm install`、`pnpm add`、`pnpm remove`、`pnpm update` 和 `pnpm run <script>`
- 禁止生成或执行 `npm`、`npx`、`yarn` 命令
- 禁止新增或更新 `package-lock.json`、`yarn.lock`；统一维护 `pnpm-lock.yaml`
- 需要临时执行包命令时使用 `pnpm exec` 或 `pnpm dlx`

## 编程道德规范

### 知之为知之，不知为不知

- 不确定时必须明确说明
- 禁止伪造 API
- 禁止猜测未确认需求

### 学而不思则罔，思而不学则殆

- 不要机械套用设计模式
- 优先解决当前问题
- 禁止过度设计

### 巧言令色，鲜矣仁

- 不要用复杂术语包装简单问题
- 不要为了显得高级而增加复杂度
- 不要过度自信

## 固定骨架

默认严格使用以下骨架，除非用户明确要求调整：

```ts
const ExampleModule = (() => {
  // （可选）模块私有非响应式变量，放在 f 之前

  const f = {
    // ===== 同步方法 =====
    fn1: () => {
      console.log(1)
    },
    // ===== 异步方法 =====
    fn2: async () => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      console.log(2)
    },
  }

  const s = reactive({
    // 状态类字段平铺
    state1: '1',
    list: [] as string[],
    // 表单类状态聚合为对象
    form: { keyword: '', dateRange: [] as string[] },
    // 方法用 ...f 挂载，不逐个列举
    ...f,
  })

  // （可选）同步初始化：直接内联语句，不包 init 函数
  s.state1 = 'ready'

  // （可选）挂载后逻辑：直接写 onMounted，不定义 initOnMounted
  onMounted(() => {
    s.fn2()
  })

  return s
})()
```

骨架要点：

1. 方法统一收进 `const f = {...}`，内部顺序固定：同步方法在前、异步方法在后
2. 模块内方法互调统一走 `s.xxx()`（如 `s.clearInputs()`），不直接引用裸函数名
3. `const s = reactive({ ...状态, ...f })`：状态在前，方法用展开挂载
4. **初始化与挂载逻辑直接内联，不定义 `init` / `initOnMounted` 具名函数**；没有对应逻辑就不写，不保留空壳
5. 表单类状态（输入框、选项组等成组字段）聚合为一个对象（如 `form`、`changeForm`、`options`）；`visible`、`list`、`currentRow` 等单一状态字段平铺
6. 模块最终 `return s`

## 函数定义约束

本技能生成或重构代码时，所有函数（含模块内部私有函数、事件处理函数、异步请求函数）都必须使用箭头函数：

- 允许：`const fn = () => {}`、`const fn = async () => {}`
- 禁止：`function fn() {}`、`async function fn() {}`
- 禁止：对象方法简写 `handleSearch() {}`、`async queryTable() {}`

## 命名规则

### 模块命名

模块名使用大驼峰，并体现职责：

- `Control`
- `Table`
- `Detail`
- `Add`
- `Payment`
- `Print`
- `FormModule`
- `QueryModule`

### 方法命名

优先使用表达业务语义的方法名：

- `handleSearch`
- `handleRefresh`
- `queryTable`
- `open` / `close`
- `detail`
- `approve` / `payment`
- `handleExport`

### 状态命名

状态字段保留业务含义，避免无意义缩写：

- 状态类：`visible`、`list`、`total`、`pageSize`、`currPage`、`currentRow`、`info`、`operateType`
- 表单类（聚合为对象）：`form`、`queryForm`、`changeForm`、`options`

## 操作步骤

### 1. 先识别模块边界

将页面逻辑按职责拆分，而不是按语法类型拆分。优先按以下维度分块：

- 查询区控制
- 表格数据与行操作
- 弹窗开关与上下文
- 明细/审批/打印等独立子流程
- 页面初始化依赖

如果一个对象同时负责“查询参数 + 表格请求 + 弹窗控制”，优先将其拆开。

### 2. 按固定骨架创建模块

模块内部顺序固定为：

1. （可选）模块私有非响应式变量
2. `const f = {...}`（同步方法 → 异步方法）
3. `const s = reactive({ ...状态, ...f })`
4. （可选）内联同步初始化语句
5. （可选）`onMounted(() => {...})`
6. `return s`

不要先写 `reactive` 再在下面补函数，避免阅读时来回跳转。

### 3. 控制嵌套层级

必须主动减少包裹层级：

- 避免再套一层 `methods`、`actions`、`state`
- 避免在模块里再创建无必要的对象中转
- 避免多层工具函数嵌套导致逻辑下沉
- 能平铺定义的，就不要额外包一层

## 分支决策

### 如果是新增模块

直接按固定骨架生成，并补齐：模块名称、状态字段、业务方法。

### 如果是重构现有代码

按以下顺序调整：

1. 找出同一职责的状态和方法
2. 合并到一个 IIFE 模块中
3. 将对外方法挂到 `s`
4. 保持原有调用方式尽量不变
5. 检查模板绑定是否仍指向正确模块
6. 禁止删除原本的代码，如要修改逻辑，则应该把原先的逻辑注释，然后添加新的逻辑，并用注释标出

### 如果模块很小

即使只有 1 到 2 个方法，也优先保持同一骨架，除非用户明确要求更轻量。

### 如果模块之间有依赖

- 允许模块之间直接调用，如 `Table.queryTable()`
- 避免循环依赖
- 公共纯函数优先提取到模块外部
- 计算逻辑、转换逻辑可定义在模块外部，业务状态留在模块内

### 如果存在纯工具函数

以下内容通常放在模块外部：

- `computeXxx`
- `formatXxx`
- `mapXxx`
- 与组件生命周期无关的纯计算

## 页面拆分与组件约定

页面文件过长时，把纯展示部分拆分为独立组件，业务逻辑全部留在页面的响应式模块中：

1. 组件放在页面目录的 `_components/` 下，由用户后续替换，不改组件库源码
2. 组件是纯展示：动作通过 `emits` 上抛给页面绑定到模块方法；组件内不写业务逻辑，也不接收整个响应式模块对象作 props
3. 数据传递方式：只读数据与表单对象走 props（表单对象为引用共享，嵌套字段在组件内直接 `v-model` 绑定）；弹窗显隐用 `defineModel<boolean>`。**表单对象不要用 `defineModel`**——defineModel 的语义是子组件可整体替换该值，而表单只是嵌套改字段，属于 props 引用共享。例如：

```ts
// 弹窗开关（v-model，与父级 Form.groupVisible 双向绑定）
const visible = defineModel<boolean>({ default: false })

// 表单数据（props 引用共享）
withDefaults(
  defineProps<{
    form: { id: number; name: string; url: string; color: string }
  }>(),
  {},
)

defineEmits<{ save: [] }>()
```

4. `defineProps` 统一用 `withDefaults` 包裹（与组件库风格一致）：可选 prop 给默认值，全必填时用 `{}` 占位
5. 样式随组件走（scoped 自包含）；拆分组件时把对应样式一并迁走，页面里无引用的样式删除

## 质量检查

生成或重构完成后，逐项检查：

- 是否使用 `const Xxx = (() => { ... })();`
- 是否所有函数都使用箭头函数定义，没有 `function`、`async function`、对象方法简写
- 响应体变量名是否固定为 `s`，方法是否通过 `...f` 挂载
- 是否先定义 `f` 与 `s`，再写初始化 / `onMounted`
- 是否**没有**保留空的 `init` / `onMounted` 壳（无逻辑就不写）
- 表单类状态是否聚合为对象，状态类字段是否平铺
- 是否在 `s` 中暴露了需要给模板/外部使用的方法
- 是否减少了多余缩进和对象包裹
- 是否保持原有业务语义与调用链不变
- 是否避免引入新的循环依赖
- 是否没有把纯工具函数错误塞进响应式对象
- 是否符合当前文件已有命名习惯
- 拆分出的组件是否为纯展示（emits 上抛动作，逻辑留在模块）
- 是否没有把整个响应式模块对象作为 props 传入组件
- 弹窗组件显隐是否用 `defineModel<boolean>`；表单对象是否走 props（未误用 defineModel）
- `defineProps` 是否统一用了 `withDefaults` 包裹

## 输出要求

在执行此技能时，默认按以下方式产出：

1. 先说明将如何分模块、每个模块的职责
2. 然后生成或修改代码
3. 最后说明哪些逻辑放在模块外、哪些留在模块内，并补充简短验收说明

## 针对当前项目的推荐模块划分

对于列表页、审批页、弹窗页，优先考虑以下结构：

- `Control`：查询条件、筛选、导出、初始化查询条件
- `Table`：表格数据、分页、行操作、当前行
- `Detail`：明细/审批弹窗上下文
- `Payment`：收款/退款上下文
- `Add`：新增弹窗控制
- `Print`：打印上下文

## 不要这样做

避免以下反模式：

- `const s = reactive({ handleSearch() {} })`、`const s = reactive({ async queryTable() {} })`（对象方法简写）
- `const state = reactive({ actions: { ... } })`、`const s = reactive({ state: {...}, methods: {...} })`（多余包裹层）
- 在 `reactive` 之后零散定义大量函数
- 所有页面只保留一个巨型模块
- 为了“封装”而增加不必要的层级
- 定义空的或仅转发的 `init` / `initOnMounted` 具名函数（初始化与挂载逻辑直接内联）
- 表单字段散落平铺在响应体里（应聚合为对象）
- 组件里写业务逻辑（应上抛 emits 由模块处理）
- 把整个响应式模块对象传给组件作 props（应拆成只读 props + 表单对象 props + emits）
- 弹窗开关通过 props 传 visible 字段（应用 defineModel + v-model）
- 表单对象用 `defineModel` 传递（对象 model 语义是整体可替换，表单只是嵌套改字段，应用 props 引用共享）
- 裸用 `defineProps`（应统一 `withDefaults(defineProps<...>(), {...})`）

## 示例提示词

- `/vue-reactive-module-pattern 把这个 Vue 列表页拆成 Control、Table、Detail 三个响应式模块`
- `/vue-reactive-module-pattern 按固定模板为这个弹窗页新增一个 FormModule`
- `/vue-reactive-module-pattern 重构当前 script setup，要求严格使用 s 作为响应体名称`
- `/vue-reactive-module-pattern 检查当前页面的 reactive 写法是否符合模块化规范并直接修正`
