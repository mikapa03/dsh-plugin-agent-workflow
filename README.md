# dsh-plugin-agent-workflow（预构建版，适配 DeepSeek Harness 0.2.1-rc.2）

在 DeepSeek Harness 的会话页「对话」「轨迹」标签页之外，增加一个「工作流」标签页，按用户轮次
可视化模型请求、模型响应与工具调用。

本仓库是上游 [xuanyuanzhifeng/dsh-plugin-agent-workflow](https://github.com/xuanyuanzhifeng/dsh-plugin-agent-workflow)
（v0.3.0，MIT）面向 **DeepSeek Harness 0.2.1-rc.2** 的预构建兼容分发版：编译产物 `lib/` 已提交，
安装时**不运行任何构建脚本**，因此可以像普通 npm/GitHub 插件一样被 DSH 插件管理器安装。

## 为什么需要预构建

上游 0.3.0 只发布 TypeScript 源码，依赖 `prepare` 脚本在安装时现场编译。DSH 桌面版的依赖安装
带供应链策略（`allowBuilds` 白名单），git 依赖的构建脚本会被拒绝，安装直接失败：

```
[ERR_PNPM_GIT_DEP_PREPARE_NOT_ALLOWED] The git-hosted package "..." needs to execute
build scripts but is not in the "allowBuilds" allowlist.
```

本仓库预先把 `lib/index.js`（宿主入口）与 `lib/client.js`（浏览器模块）构建好并提交，
`package.json` 里没有 `prepare` / `prepack` 等安装期脚本，也没有安装期依赖，所以
**从 GitHub 或 npm 安装都不会触发 build-script 审批**。

## 安装

三种方式都只装这份预构建产物，装完在 DSH 侧栏「插件」页的**已安装插件**里即可管理
（启用/停用、单独停用 `ui-workflow` 行、卸载）。

1. **GitHub（推荐，本仓库）** —— 插件管理器 → 添加插件 → 包名或地址，填：

   ```
   https://github.com/mikapa03/dsh-plugin-agent-workflow
   ```

   需要固定版本时用 `#v0.3.1` 之类的 tag/commit，例如
   `https://github.com/mikapa03/dsh-plugin-agent-workflow#v0.3.1`。

2. **npm**（发布到 npm 后）：在添加插件里填 `包名` 或 `包名@版本`。

3. **本地**：添加插件 → 本地目录（绝对路径），或把打包好的 `.tgz` 用「压缩包」方式安装。

安装后刷新 Web UI，会话页签里出现「工作流」。

## 插件管理器里能做什么

- **组合包开关**：启用/停用整个插件（改 `dsh.profile.bundles`，依赖保留）。
- **行开关**：进入插件详情，可单独停用/启用 `ui-workflow` 行（写入 profile 的 `cordis.patch.yml`）。
- **来源信息**：显示安装来源 spec 与当前版本。
- **卸载**：移除依赖与 bundle，插件功能消失。
- **没有「更新」按钮**：DSH 只给“按需安装的官方插件”提供更新；第三方插件的升级方式是
  **先卸载再安装新版**（本仓库发新 tag 后重新安装即可）。

## 兼容性

| 本构建 | dsh |
| --- | --- |
| 0.3.1 | >= 0.2.1-rc.2（在 DeepSeek Harness Desktop 0.2.1-rc.2 上验证） |

编译期使用 dsh 0.2.1-rc.2 的 `@deepseek-ai/*` 类型定义：`tsc -p tsconfig.build.json` 无错误，
上游测试（`vitest`，7 个文件 / 20 个用例）全部通过。相对上游只做了两处非功能改动：
依赖版本对齐 0.2.1-rc.2、移除 `prepare` 脚本（改为预构建提交 `lib/`）。

功能列表、数据来源与只读性说明见上游 README：插件只读取 Session 已记录事件与内置
Trajectory 投影，不会向模型请求里添加任何提示词、消息或工具。

## 从源码重新构建

```bash
pnpm install
pnpm run build      # 产物写入 lib/，需要随源码一起提交
pnpm run typecheck
pnpm test
```

注意：在 macOS 上用 DSH 自带的 node 构建时，`tsdown` 会因该 node 二进制的
hardened runtime 签名无法加载 rolldown 原生模块（`ERR_DLOPEN_FAILED ... different Team IDs`）。
处理办法是把 node 复制一份并重新签名后再用于构建：

```bash
NODE="/Applications/DeepSeek Harness.app/Contents/Resources/runtime/primary-runtime/dependencies/node/bin/node"
mkdir -p /tmp/dsh-toolchain/bin && cp "$NODE" /tmp/dsh-toolchain/bin/node
codesign --force --sign - /tmp/dsh-toolchain/bin/node
```

改完源码后务必提交 `lib/`（`.gitignore` 故意没有忽略它），否则安装到的是旧产物。

## 许可

MIT，继承上游版权与许可证，见 `LICENSE`。
