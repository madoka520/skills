---
name: vue-reactive-module-pattern
description: 'Generate or refactor Vue 3 script setup reactive code into strict module blocks using reactive + IIFE pattern. Use when writing page state, table/query/dialog control modules, splitting large Vue files, standardizing reactive code structure, reducing nesting, or following the team rule that each reactive module is defined as const Xxx = (() => { ... const s = reactive({...}); init(); onMounted(initOnMounted); return s; })(). 支持 Vue 响应式模块、页面状态分块、列表查询模块、弹窗模块、表单控制模块重构。'
argument-hint: '模块名称、职责、状态字段、方法、是否需要 init / onMounted'
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

## 适用场景

当用户提出以下需求时使用本技能：

- 在 `Vue 3` + `script setup` 中新增响应式业务模块
- 将现有大段逻辑拆分为 `Control`、`Table`、`Detail`、`Dialog`、`Print` 等模块
- 统一团队中的响应式代码风格
- 将普通 `reactive({...})`、零散函数、杂糅逻辑重构为固定模块格式
- 明确要求“严格仿照示例格式”“减少代码包裹层级”“模块化定义响应式代码”

常见触发词：

- Vue 响应式模块
- `reactive` 模块化
- `script setup` 结构整理
- 页面状态分块
- 表格模块 / 查询模块 / 弹窗模块
- 按固定模板生成 Vue 代码

## 目标产物

产出符合以下特征的代码：

1. 每个业务块都定义为一个独立常量模块：`const Xxx = (() => { ... })();`
2. 模块内部尽量减少额外缩进和无意义包裹
3. 模块内部统一包含：同步方法、异步方法、初始化方法、挂载后方法、响应体 `s`
4. 响应体变量名必须固定为 `s`
5. 模块最终统一 `return s`
6. 若存在初始化逻辑，则在模块内直接调用 `init()`
7. 若存在挂载后逻辑，则使用 `onMounted(initOnMounted)`
8. 状态与方法统一挂到 `s` 上，便于模板与业务调用
9. 函数定义必须统一使用箭头函数，禁止使用 `function` 声明、`async function`、对象方法简写等写法
## 固定骨架

默认严格使用以下骨架，除非用户明确要求调整：

```ts
const ExampleModule = (() => {


  const f = {
    fn1: () => {
    console.log(1);
    },
    fn2: async () => {
        await new Promise((resolve) => setTimeout(resolve, 1000));
        console.log(2);
    }
  
  }

  const init = () => {
    console.log("模块初始化示例");
  };

  const initOnMounted = () => {
    console.log("组件挂载后执行示例");
  };

  const s = reactive({
    state1: "1",
    state2: "2",
    fn1,
    fn2,
    ...f
  });

  init();
  onMounted(initOnMounted);

  return s;
})();
```

## 函数定义约束

本技能生成或重构代码时，所有函数都必须使用箭头函数：

- 允许：`const fn = () => {}`
- 允许：`const fn = async () => {}`
- 禁止：`function fn() {}`
- 禁止：`async function fn() {}`
- 禁止：`handleSearch() {}`
- 禁止：`async handleSearch() {}`

即使函数是模块内部私有函数、初始化函数、事件处理函数、异步请求函数，也必须保持箭头函数写法。

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
- `open`
- `close`
- `detail`
- `approve`
- `payment`
- `handleExport`
- `init`
- `initOnMounted`

上述方法名仅表示命名语义，不表示可以使用 `function` 关键字；实际定义时必须写成：

```ts
const handleSearch = () => {};
const queryTable = async () => {};
const init = () => {};
const initOnMounted = () => {};
```

### 状态命名

状态字段保留业务含义，避免无意义缩写：

- `visible`
- `form`
- `list`
- `total`
- `pageSize`
- `currPage`
- `doOpenFlag`
- `currentRow`
- `info`
- `operateType`

## 操作步骤

### 1. 先识别模块边界

将页面逻辑按职责拆分，而不是按语法类型拆分。

优先按以下维度分块：

- 查询区控制
- 表格数据与行操作
- 弹窗开关与上下文
- 明细/审批/打印等独立子流程
- 页面初始化依赖

如果一个对象同时负责“查询参数 + 表格请求 + 弹窗控制”，优先将其拆开。

### 2. 为每个职责创建独立模块

每个模块都使用：

```ts
const Xxx = (() => {
  // 方法定义
  // 初始化定义
  const s = reactive({
    // 状态
    // 方法引用
  });
  init();
  onMounted(initOnMounted);
  return s;
})();
```

### 3. 先定义方法，再声明响应体

模块内部顺序固定为：

1. 同步方法
2. 异步方法
3. `init`
4. `initOnMounted`
5. `const s = reactive({...})`
6. `init()`
7. `onMounted(initOnMounted)`
8. `return s`

不要先写 `reactive` 再在下面补函数，避免阅读时来回跳转。
所有这些方法都必须用箭头函数定义，不允许混入 `function` 风格。

### 4. 控制嵌套层级

必须主动减少包裹层级：

- 避免再套一层 `methods`、`actions`、`state`
- 避免在模块里再创建无必要的对象中转
- 避免多层工具函数嵌套导致逻辑下沉
- 能平铺定义的，就不要额外包一层

## 分支决策

### 如果是新增模块

直接按固定骨架生成，并补齐：

- 模块名称
- 状态字段
- 业务方法
- 初始化逻辑

### 如果是重构现有代码

按以下顺序调整：

1. 找出同一职责的状态和方法
2. 合并到一个 IIFE 模块中
3. 将对外方法挂到 `s`
4. 保持原有调用方式尽量不变
5. 检查模板绑定是否仍指向正确模块
6. 禁止删除原本的代码,如要修改逻辑 则应该把原先的逻辑注释,然后添加新的逻辑,并且用注释标出
### 如果模块很小

即使只有 1 到 2 个方法，也优先保持同一骨架，除非用户明确要求更轻量。

### 如果模块之间有依赖

遵循以下原则：

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

## 质量检查

生成或重构完成后，逐项检查：

- 是否使用 `const Xxx = (() => { ... })();`
- 是否所有函数都使用箭头函数定义
- 是否没有出现 `function`、`async function`、对象方法简写等写法
- 响应体变量名是否固定为 `s`
- 是否先定义方法，再定义 `s`
- 是否在 `s` 中暴露了需要给模板/外部使用的方法
- 是否保留了 `init` 与 `initOnMounted` 入口
- 是否减少了多余缩进和对象包裹
- 是否保持原有业务语义与调用链不变
- 是否避免引入新的循环依赖
- 是否没有把纯工具函数错误塞进响应式对象
- 是否符合当前文件已有命名习惯

## 输出要求

在执行此技能时，默认按以下方式产出：

1. 先说明将如何分模块
2. 再给出每个模块的职责
3. 然后生成或修改代码
4. 最后说明哪些逻辑放在模块外、哪些逻辑留在模块内
5. 补充简短验收说明

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

- `function handleSearch() {}`
- `async function queryTable() {}`
- `const s = reactive({ handleSearch() {} })`
- `const s = reactive({ async queryTable() {} })`
- `const state = reactive({ actions: { ... } })`
- `const s = reactive({ state: {...}, methods: {...} })`
- 在 `reactive` 之后零散定义大量函数
- 所有页面只保留一个巨型模块
- 为了“封装”而增加不必要的层级

## 示例提示词

- `/vue-reactive-module-pattern 把这个 Vue 列表页拆成 Control、Table、Detail 三个响应式模块`
- `/vue-reactive-module-pattern 按固定模板为这个弹窗页新增一个 FormModule`
- `/vue-reactive-module-pattern 重构当前 script setup，要求严格使用 s 作为响应体名称`
- `/vue-reactive-module-pattern 检查当前页面的 reactive 写法是否符合模块化规范并直接修正`
