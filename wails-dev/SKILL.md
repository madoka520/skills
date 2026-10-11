---
name: wails-dev
description: Wails 3 项目开发规范(以 madoka-nas 为准)。在 wails 项目里新增/修改后端服务、前端要调用后端、提到 bindings、跑 wails3 命令、前端连不上 9245、go build 报 runtime/cgo exit status 2 时使用。核心铁律:bindings 是自动生成物,不要管它。
---

# Wails 开发规范

## Bindings:自动生成物,不要管

- `frontend/bindings/` 是 `wails3 generate bindings` 的生成物,已在 .gitignore,等于构建缓存
- **不要手动创建、修改、注册 bindings 文件;也不要花时间逐个读生成物去核对类型** —— 前端直接 `import` 对应模块,类型不对说明没重新生成,而不是去改生成物
- 新增或改动后端服务后,重新生成一次即可:

```bash
cd <wails主项目根目录>
"$(cygpath -u "$USERPROFILE")/go/bin/wails3.exe" generate bindings -ts
```

- **必须带 `-ts`**:不带会生成 JS 版(models.js / index.js),而项目前端全是 TS 导入(`from '@wails/changeme/backend/model'`),会瞬间满屏 Cannot find module
- wails3 可执行文件在 `%USERPROFILE%\go\bin\wails3.exe`

## 新增一个后端服务的固定流程

1. `backend/service/xxx_service.go`:`type XxxService struct{}`,方法首字母大写导出
2. `main.go` 的 `Services` 数组里加 `application.NewService(&service.XxxService{})`
3. `wails3 generate bindings -ts` 重新生成
4. 前端按现有模式封装 `frontend/src/api/xxxApi.ts`:

```ts
import * as xxxApi from "@wails/changeme/backend/service/xxxservice"
export default xxxApi
```

5. 服务方法签名建议「单返回值 + error」(如 `(*XxxResult, error)`):前端拿到的就是一个属性名清晰的对象;多返回值的绑定行为不直观,别用

## 本机命令环境(Git Bash)

- 终端只用 Git Bash,不用 PowerShell;包管理只用 pnpm
- go build 前置 mingw64,否则 MSYS 的 as/ld 让 gcc 静默失败(`runtime/cgo exit status 2`):

```bash
PATH="/c/msys64/mingw64/bin:$PATH" go build . ./backend/...
```

- 验证前端不需要单独 `pnpm dev`:根目录 `dev.cmd` 起 `wails3 dev` 会自动带起前端,端口 9245
- Taskfile 里给 pnpm 传参直接跟在后面,**不要写 `--` 分隔**(会原样传给 vite,导致端口参数失效跑回 5173)
