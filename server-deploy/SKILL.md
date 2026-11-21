---
name: server-deploy
description: 往自己的服务器(尤其 madoka VPS 120.26.117.119)部署/发布/上传文件、scp、center 发版、改服务器上的服务代码时使用。核心铁律:动手前必须四步验明正主(域名→nginx→端口→进程→目录),绝不凭目录名和域名长得像就猜。11-22 实锤事故:误传 madokami 目录致其无谓重启两次。
---

# 服务器部署铁律

## 核心铁律:验明正身,才准动手

一台服务器上常跑多个同名系服务,**目录名和域名长得像 ≠ 它就是这个域名的服务**。上传/改文件前必须走完四步验证链:

```bash
# ① 域名解析出 IP,确认是哪台机器
nslookup <域名>
# ② 机器上查 nginx 站点配置,拿到反代端口(nginx -T 或 sites-enabled)
ssh <user>@<ip> "grep -rn 'proxy_pass\|server_name' /etc/nginx/ 2>/dev/null"
# ③ 端口对到进程,拿到工作目录
ssh <user>@<ip> "ss -tlnp | grep ':<端口> '"
# ④ 进程 cmd 里的路径就是正主目录,向用户展示后再动手
```

四步没走完,一个字节都不许传。拿不准就停下来问用户。

## 已知服务器清单(随发现随补)

### 120.26.117.119(madoka-kami,Debian)

- 登录:`ssh madoka@120.26.117.119`(本机密钥直连;root/ubuntu/admin 都不行)
- nginx 反代域名:`api.nas-center.madokami.cn` → **madoka-nas-center** 服务

| 目录 | 角色 | 纪律 |
|------|------|------|
| `/home/madoka/data/server/madoka-nas-center/` | **center 正主**(桌面 app 的授权面) | 唯一可传 center 代码的地方 |
| `/home/madoka/data/server/madokami/` | 另一个服务(:3000,**域名同名陷阱**) | 严禁碰 |
| `/home/madoka/data/server/wuziqi/` | 五子棋服务 | 严禁碰 |

- 无 git,全是手动 scp 传文件;跑的是 `tsx watch`——**src 下文件一变即自动重启**,传错文件会立即污染运行中的服务
- 服务器上另有 mysql/frp/openvpn 等,与部署无关不要动

## 操作规程(顺序不可乱)

1. **验明正主**(上面的四步)
2. **动手前备份**:`cp <目标文件> <目标文件>.bak-$(date +%Y%m%d)`——出事能原样还原的唯一保险
3. **上传**:`scp <本地文件> <user>@<ip>:<验过的正主目录>`
4. **验证要带鉴权**:fastify 的 onRequest 鉴权钩子在路由匹配**之前**拦截,无鉴权 curl 得到 403 会误判「路由已存在」(旧代码其实 404)。必须带真实凭证(如 x-client-id)请求:
   - 新路由 + 空参数 → **400** 才算部署成功
   - 404 = 没部署上;403 = 请求根本没到路由层,验证无效
5. **误传了怎么办**:立即用第 2 步的备份 `cp` 还原 → `diff` 逐字节确认 → 向用户坦白服务被重启过的次数和不可撤销的影响(内存态丢失),不许隐瞒

## 收尾清理:杀进程必须定点

本地起过的冒烟服务,清理时**只准按 PID 树杀**(`kill <pid>` 或 taskkill /PID),**严禁 `taskkill //IM node.exe` 这类按镜像名一刀切**——机器上跑着用户的 vite/wails dev,同镜像进程全家会被误杀(11-22 二次事故:清本地 center 顺手杀掉了用户 9245 的 vite)。

## 事故存档(为什么有这份 skill)

2026-11-22:给 center 发版,进程列表里 `madokami/` 与域名 `madokami.cn` 长得像,没查 nginx/端口归属直接 scp 了 user.ts 过去——madokami 被 tsx watch 重启两次(内存态丢失,无法撤销)。靠动手前备份才把文件原样还原。根因:用「名字合理」替代了「归属验证」。此 skill 即改正措施。
