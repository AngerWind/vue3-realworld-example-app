# Vue3 学习问答记录

> 学习 vue3-realworld-example-app 过程中的问答整理（问：我的提问 / 答：讲解），按主题分类，方便回看。

## 目录

- [一、项目启动](#一项目启动)
- [二、包管理器 pnpm](#二包管理器-pnpm)
- [三、ESLint](#三eslint)
- [四、VS Code 使用技巧](#四vs-code-使用技巧)
- [五、HTML 语义化标签](#五html-语义化标签)
- [六、项目目录结构](#六项目目录结构)
- [七、import 机制](#七import-机制)
- [八、组件注册与跳转](#八组件注册与跳转)
- [九、CSS 来源](#九css-来源)
- [十、unplugin-vue-components 完整案例](#十unplugin-vue-components-完整案例)

---

## 一、项目启动

**问：我要怎么启动项目？**

答：

```bash
cd /d/vue3-learning/vue3-realworld-example-app
pnpm install   # 首次或依赖变化后执行
pnpm dev       # 启动开发服务器, 浏览器访问 http://localhost:4173
```

- 成功标志：终端出现 `Local: http://localhost:4173/`
- 停止：终端里 `Ctrl + C`；关闭终端窗口也会停
- 改代码不用重启：Vite 热更新，保存后页面自动刷新
- 端口被占用：旧实例没关干净，`Ctrl+C` 掉旧的，或 `pnpm dev --port 4174` 换口
- 推荐在 VS Code 内置终端操作（`` Ctrl + ` `` 呼出）

**问：pnpm dev 的作用是什么？**

答：`pnpm` 是包管理器（同 npm，更快更省磁盘）；`dev` 是 package.json 里定义的脚本名，`pnpm dev` 实际执行 `"dev": "vite --port 4173"`，即**启动 Vite 开发服务器**。

开发服务器做的事：

1. 在 4173 端口起本地网站
2. 按需编译：浏览器请求哪个文件才编译哪个，启动飞快
3. 热更新（HMR）：改代码保存后页面局部刷新，页面状态不丢
4. 报错同时显示在终端和浏览器页面

package.json 常用脚本：

| 命令 | 作用 | 什么时候用 |
|---|---|---|
| `pnpm dev` | 开发服务器（热更新） | 日常学习、写代码 |
| `pnpm build` | 打包静态文件到 dist/ | 部署上线 |
| `pnpm lint` | ESLint 全量检查并自动修复 | 提交前 |
| `pnpm type-check` | vue-tsc 类型检查 | 提交前 |
| `pnpm test:unit` | 跑单元测试 | 验证代码 |

## 二、包管理器 pnpm

**问：（pnpm 的命令）对应到 npm 中是什么命令？**

答：

| 用途 | pnpm | npm |
|---|---|---|
| 安装依赖 | `pnpm install` | `npm install` |
| 启动脚本 | `pnpm dev` | `npm run dev` |
| 打包 | `pnpm build` | `npm run build` |
| 临时执行命令 | `pnpm dlx xxx` | `npx xxx` |
| 全局安装 | `pnpm add -g xxx` | `npm install -g xxx` |

小区别：`npm run xxx` 的 `run` 一般不能省（仅 start/test 等少数内置命令可省，如 `npm test` 可以、`npm dev` 不行）；`pnpm` 可直接省 `run`。

**本项目坚持用 pnpm**：仓库锁定了 `packageManager: pnpm@10.33.0` 且有 `pnpm-lock.yaml`，混用 npm 会生成 package-lock.json，两份锁文件混乱，且 pre-push 钩子写死了 pnpm。

**问：（PowerShell 报错）pnpm：无法加载文件 pnpm.ps1，因为在此系统上禁止运行脚本？**

答：Windows PowerShell 的**执行策略**默认禁止运行 .ps1 脚本，pnpm 在 PowerShell 里正是 .ps1 文件。两种解法：

```powershell
# 方法一(推荐, 一次解决): 允许本地脚本+已签名远程脚本, 仅影响当前用户
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# 方法二(绕过): 走 CMD 通道不受策略限制
pnpm.cmd dev
```

## 三、ESLint

**问：ESLint 是什么，怎么用，这个项目中有哪些是关于 ESLint 的？**

答：ESLint 是**代码检查工具**：按规则扫描代码，抓 bug 隐患（未定义变量、v-for 缺 key、Promise 未处理异常）+ 统一风格（缩进、分号、import 顺序）。类似 Word 的拼写检查。

三个使用层面：

1. **编辑器实时看**：VS Code 的 `dbaeumer.vscode-eslint` 插件自动读项目配置，红/黄波浪线提示，鼠标悬停看原因
2. **命令行**：`pnpm lint`（全量检查并修复）、`npx eslint src/router.ts`（单文件，不加 --fix 只报告）
3. **Git 钩子自动跑**：本项目已配置 commit 时对暂存文件跑 `eslint --fix`（lint-staged + simple-git-hooks），push 前跑类型检查和构建，检查不过提交不了

本项目中 ESLint 相关的文件：

| 位置 | 作用 |
|---|---|
| `eslint.config.js` | 规则配置：基于 `@mutoe/eslint-config`（Anthony Fu 规则集）；强制自定义事件 kebab-case；忽略自动生成的 `src/services/api.ts` 和所有 md |
| `package.json` 的 `lint` 脚本 | `"lint": "eslint --fix ."` 手动全量检查 |
| `package.json` 的 `lint-staged` + `simple-git-hooks` | git 钩子，只检查暂存文件，速度快 |
| devDependencies | `eslint`、`@mutoe/eslint-config`、`eslint-plugin-vuejs-accessibility`（无障碍规则） |

## 四、VS Code 使用技巧

**问：VS Code 设置里开启保存时自动修复 ESLint 要怎么开？**

答：settings.json（`Ctrl+Shift+P` → `Preferences: Open User Settings (JSON)`）加：

```json
"editor.codeActionsOnSave": {
  "source.fixAll.eslint": "explicit"
}
```

验证：在 .vue 文件里故意删掉一个分号再保存，会自动补回来。

**问：终端被我 X 掉了，要从哪里打开？**

答：三种方式：

1. 快捷键 `` Ctrl + ` ``（反引号，Tab 上面、数字 1 左边），再按一次收起
2. 菜单栏：Terminal → New Terminal
3. `Ctrl+Shift+P` → `Terminal: Create New Terminal`

**问：我要怎么设置终端打开来是 Git Bash？**

答：终端面板右上角下拉箭头 `∨` → **Select Default Profile** → 选 **Git Bash**。或 settings.json 加：

```json
"terminal.integrated.defaultProfile.windows": "Git Bash"
```

**问：在 VS Code 中要怎么实现和 IDEA 一样的 Ctrl+Alt+方向键跳转？**

答：VS Code 对应功能是 **Go Back / Go Forward**（默认 `Alt+←`/`Alt+→`）。改成 IDEA 习惯：`Ctrl+Shift+P` → `Preferences: Open Keyboard Shortcuts (JSON)`，加：

```json
[
  { "key": "ctrl+alt+left",  "command": "workbench.action.navigateBack" },
  { "key": "ctrl+alt+right", "command": "workbench.action.navigateForward" }
]
```

IDEA 用户常用快捷键对照：

| IDEA | VS Code | 功能 |
|---|---|---|
| Ctrl+B | `F12` 或 Ctrl+点击 | 跳转定义 |
| Ctrl+Alt+←/→ | Alt+←/→（默认，已改为上面配置） | 前进/后退 |
| 双击 Shift | `Ctrl+P` | 全局搜文件 |
| Ctrl+Shift+F | `Ctrl+Shift+F` | 全局搜内容 |
| Ctrl+E | `Ctrl+P` 直接回车 | 最近文件 |
| Shift+F6 | `F2` | 重命名符号 |

**问：VSC 中我要和 IDEA 一样 Copy Reference 要怎么办？**

答：VS Code 没有完全等同的内置功能（IDEA 复制的是 Java 全限定名，前端靠 import 路径定位，需求不同）。接近的内置功能：

| 功能 | 快捷键 | 结果示例 |
|---|---|---|
| 复制相对路径 | `Ctrl+K Ctrl+Shift+C` | `src/components/AppLink.vue` |
| 复制绝对路径 | `Shift+Alt+C` | `D:\...\AppLink.vue` |
| 复制文件名 | 右键文件 → Copy File Name | `AppLink.vue` |

日常分享代码位置的做法：相对路径 + 状态栏行号，拼成 `src/router.ts:42`。

**问：我要问你具体某行代码的作用，要怎么给你？**

答：直接给"文件路径 + 行号"即可（如 `src/router.ts 第 42 行` 或 `src/router.ts:42`），助手可直接读本机文件并自己翻上下文，不用复制粘贴代码。配合技巧：`Ctrl+K Ctrl+Shift+C` 复制相对路径，状态栏看行号（`Ln 42, Col 10`）。项目外的代码片段则直接贴出来问。

## 五、HTML 语义化标签

**问：nav 是什么标签？**

答：`<nav>` 是 HTML5 **语义化标签**，专用于包裹导航链接。功能上和 `<div>` 完全一样（无默认样式的容器），区别在于标签名说明了内容含义，方便搜索引擎和屏幕阅读器（读屏软件会提示"这是导航区"）。原则：**能用语义标签就别用 div**。

| 标签 | 含义 | 典型页面结构 |
|---|---|---|
| `<header>` | 页头 | header |
| `<nav>` | 导航 | nav |
| `<main>` | 主体内容（一页一个） | main → article / aside |
| `<article>` | 独立完整内容（文章/帖子） | |
| `<section>` | 内容章节分块 | |
| `<aside>` | 侧边栏/附属信息 | |
| `<footer>` | 页脚 | footer |

项目范例：`AppNavigation.vue` 用 `<nav>`，`AppFooter.vue` 用 `<footer>`。

配套练习文件：`D:\code\example4all\frontend\html-css\html相关标签说明\34_语义化标签.html`（带背景色演示，双击浏览器打开可看效果）

## 六、项目目录结构

**问：src 中的各个文件夹主要存放什么含义，为什么有 pages，又有 views，又有 components？**

答：**pages 和 views 是同一个东西**——页面级组件（对应一个 URL 的大组件），只是不同项目的命名习惯（Nuxt/Next 生态叫 pages，国内 admin 模板常叫 views）。本项目只有 pages 没有 views；views 是在 vue-pure-admin 里看到的。

**pages 与 components 的区别**（核心划分）：

| | pages/（页面组件） | components/（普通组件） |
|---|---|---|
| 角色 | 完整页面，直接挂路由 | 页面零件，被页面或其他组件使用 |
| 对应 URL | 是（如 /login → Login.vue） | 否 |
| 类比 | 房间 | 家具 |

例：`pages/Home.vue` 是房间，内部由 `ArticlesList`、`PopularTags` 等家具拼成；`AppPagination` 这种零件被多个房间复用。

本项目 src 完整结构：

```
src/
├── main.ts        # 入口: 创建应用、装配插件, App 从这里启动
├── App.vue          # 根组件: 所有页面的最外层壳
├── router.ts        # 路由表: URL → 页面的映射
├── config.ts        # 读取环境变量(如 API 地址)
├── pages/           # 页面级组件(一个文件 = 一个网址)
├── components/      # 可复用零件组件
├── composable/      # 组合式函数: 可复用的"带响应式的逻辑"(Vue3 精髓)
├── store/           # Pinia 状态管理: 跨组件共享的数据(如登录用户)
├── services/        # 与后端 API 打交道的代码
├── plugins/         # 应用启动时的初始化(全局组件、token 恢复等)
├── utils/           # 纯工具函数(日期格式化、存取 localStorage)
└── types/           # TypeScript 类型声明
```

一句话记忆：pages 是一整页的样子；components 是零件；composable 是可复用逻辑；store 是共享数据；services 和后端说话；utils 是纯函数；plugins 是开机自检。

## 七、import 机制

**问：为什么 import ts 文件不需要文件后缀，import vue 文件就需要？**

答：核心原因——**工具链有一张"自动补全后缀"的白名单，`.ts` 在名单里，`.vue` 不在**。

写 `import router from './router'` 时，TS 和 Vite 按内置扩展名列表依次尝试 `./router.ts → .tsx → .d.ts → .js ...`，找到第一个存在的就用。Vite 默认列表：

```
['.mjs', '.js', '.mts', '.ts', '.jsx', '.tsx', '.json']   ← 注意没有 .vue
```

`.vue` 不进名单的原因：它不是 JS/TS 原生模块格式，需 `@vitejs/plugin-vue` 编译；写全 `.vue` 也是社区约定，让读者一眼认出这是组件。同理 `.css`、`.png` 等也必须写全。

**问：所以只有原生可以 import 的文件不需要后缀，png、jpg、vue 这些都需要加后缀？**

答：方向对，但判断标准要修正——**不是看"原不原生"，而是看"在不在解析器的后缀清单里"**。反例：`.ts` 对浏览器并不原生（浏览器只认识 JS），但它在清单里所以能省。

准确记忆——Vite 默认清单：`.mjs .js .mts .ts .jsx .tsx .json`

- 可省略：`.js .ts .jsx .tsx .mjs .mts .json`
- 必须写全：`.vue .css .scss .png .jpg .svg` 等其他一切

补充：

1. 清单可配置（Vite `resolve.extensions`），但没人把 `.vue` 加进去——显式写是强约定
2. 歧义坑：同目录有 `utils.ts` 和 `utils.json` 时 `import './utils'` 按清单顺序命中。实践习惯：只省 `.ts/.js`，`.json` 也写全

**问：import 一个 js 文件，会导致这个文件被执行是吗？**

答：对。**import 一个模块 = 该文件顶层代码执行一次**（模块有缓存，10 个文件 import 同一模块也只执行第一次）。执行时机是 import 语句被执行的那一刻，应用启动时按依赖链依次执行。

项目实例——副作用导入（side-effect import）：

```ts
// main.ts
import 'src/plugins/set-authorization-token'  // 不接任何导出, 只为让它执行(恢复 token)
```

常用于：注册全局东西（插件、全局组件、全局样式）、初始化逻辑。

经典坑：模块顶层代码在 import 时立刻执行，若依赖的东西还没准备好会出诡异 bug：

```ts
// 危险: import 这个文件时就执行了
export const token = localStorage.getItem('token')
```

经验法则：**模块顶层尽量只放定义，执行逻辑包在函数里按需调用**。

验证实验：新建 `src/test-log.ts` 写 `console.log('我被执行了!')`，在 main.ts import 两次，Console 只打印一次——同时验证"会执行"和"只执行一次"。

## 八、组件注册与跳转

**问：为什么在 App.vue 中 Ctrl+点击 AppNavigation 标签可以跳转，但在 AppNavigation 中点击 AppLink 标签跳转不过去？**

答：涉及 **Vue 组件的两种注册方式**：

- `AppNavigation` 是**局部注册**——App.vue 里有明确 import 语句，Volar 顺着 import 就能跳转
- `AppLink` 是**全局注册**——`plugins/global-components.ts` 里 `app.component('AppLink', AppLink)`，使用处没有 import，Volar 找不到静态引用关系

类型安全靠 `types/global-component.d.ts` 补丁（只给类型，不给跳转）：

```ts
declare module '@vue/runtime-core' {
  export interface GlobalComponents {
    AppLink: typeof AppLink
  }
}
```

**问：全局注册的组件不能点击跳转，只能手动跳转吗，也太不方便了吧？**

答：缓解办法：

1. **两跳法**：Ctrl+点击标签 → 跳到 `global-component.d.ts` 声明行 → 再点该行的 import 路径 → 到达组件文件
2. **`Ctrl+P` 快速开文件**：输入组件名回车（最快，对应 IDEA 双击 Shift）
3. **Vue devtools 反查**：F12 → Vue 面板 → 组件树选中组件 → Open in editor

业界的更好解法：现代项目放弃手动全局注册，改用 `unplugin-vue-components` 插件——**不用写 import，编译时自动补上**，省事且保留跳转（详见第十节）。vue-pure-admin 就是这种方式。

设计取舍观察：本项目只全局注册了 AppLink 一个组件（到处都要用的最基础组件），全局注册前三思——便利性 vs 可追踪性。

## 九、CSS 来源

**问：AppNavigation.vue 中使用的 CSS 是哪里的，怎么没看到定义？**

答：本项目所有样式在 **`public/main.css`**（RealWorld 官方现成模板），组件里完全没有 `<style>` 块。

加载链路：

```
index.html → <link rel="stylesheet" href="/main.css"> → public/main.css
```

`public/` 目录的文件会被 Vite 原样部署到网站根目录，所以 `/main.css` 就对应 `public/main.css`。

Vue 组件样式两种方式：

- **全局 CSS**（本项目）：class 名直接用（如 `class="navbar"`），样式集中在全局文件
- **组件内样式**（多数现代项目）：`<style scoped>` 写在组件里，只对当前组件生效

**反查技巧（调样式基本功）**：F12 → Elements 选中元素 → 右侧 Styles 面板，每条规则右上角标注来源 `main.css:行号`，点击直达定义；勾选/取消样式可实时看效果。

## 十、unplugin-vue-components 完整案例

**问：演示 unplugin-vue-components 的配置方法（含 Element Plus 案例）？**

答：该插件解决"免 import 但跳不动"的问题：**编译时自动补 import**——免手写 + 可跳转 + 类型完整 + 按需打包。

### 10.1 基础案例（本项目 src/components）

**1. 安装**

```bash
pnpm add -D unplugin-vue-components
```

**2. vite.config.ts 配置**（注意：必须放在 `vue()` 之前）

```ts
import Components from 'unplugin-vue-components/vite'

export default defineConfig({
  plugins: [
    Components({
      dirs: ['src/components'],              // 自动扫描的组件目录
      dts: 'src/types/components.d.ts',      // 自动生成的类型声明文件
    }),
    vue(),
  ],
})
```

**3. 使用——模板里直接写，什么都不用 import**

```vue
<template>
  <AppPagination />  <!-- 编译时自动 import src/components/AppPagination.vue -->
</template>
```

获得的能力：

- Ctrl+点击标签**直接跳转**到组件文件
- props 类型提示与校验
- 未使用的组件不进打包产物（按需）
- 首次运行自动生成 `src/types/components.d.ts`（勿手改），形如：

```ts
export {}
declare module 'vue' {
  export interface GlobalComponents {
    AppPagination: typeof import('../components/AppPagination.vue')['default']
    PopularTags: typeof import('../components/PopularTags.vue')['default']
  }
}
```

**4. 应用到本项目的收尾**：删除手动全局注册 `plugins/global-components.ts` 及 `main.ts` 中的调用——自动引入已覆盖 AppLink。

### 10.2 Element Plus 案例（admin 项目典型形态）

通过 **resolver（解析器）** 支持 UI 库按需自动引入：`<el-button>` 直接用，组件和样式都自动按需加载，无需 `app.use(ElementPlus)` 全量注册。

**1. 安装**

```bash
pnpm add element-plus
pnpm add -D unplugin-vue-components unplugin-auto-import
```

`unplugin-auto-import` 是配套插件：自动 import API 函数（`ref`、`computed`、`ElMessage` 等）。

**2. vite.config.ts 完整配置**

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

export default defineConfig({
  plugins: [
    // 自动 import API: ref/computed/ElMessage 等
    AutoImport({
      imports: ['vue', 'vue-router', 'pinia'],
      resolvers: [ElementPlusResolver()],
      dts: 'src/types/auto-imports.d.ts',
    }),
    // 自动 import 组件: 本地组件 + Element Plus 组件
    Components({
      dirs: ['src/components'],
      resolvers: [ElementPlusResolver()],
      dts: 'src/types/components.d.ts',
    }),
    vue(),
  ],
})
```

**3. 使用**

```vue
<template>
  <el-button type="primary" @click="handleClick">提交</el-button>
  <MyLocalComp />
</template>

<script setup lang="ts">
// 不用写 import { ref } from 'vue'
// 不用写 import { ElMessage } from 'element-plus'
const count = ref(0)

function handleClick() {
  count.value++
  ElMessage.success(`点了 ${count.value} 次`)
}
</script>
```

**4. 注意**：main.ts 里删掉 `app.use(ElementPlus)` 全量注册，否则失去按需打包的体积优势。

**5. 效果**：vue-pure-admin / vue3-element-admin 源码里"组件没 import 却能用、还能跳转"，就是这套机制。

### 10.3 常见配置项速查

| 配置项 | 作用 |
|---|---|
| `dirs` | 扫描哪些目录的本地组件，默认 `['src/components']` |
| `dts` | 类型声明文件生成位置，配了才有类型提示 |
| `resolvers` | UI 库解析器数组（ElementPlusResolver、NaiveUiResolver 等） |
| `extensions` | 识别的文件后缀，默认 `['vue']` |
| `deep` | 是否扫描子目录，默认 true |
| `directoryAsNamespace` | 目录名作命名空间，如 `<FooBar />` → `foo/Bar.vue` |
