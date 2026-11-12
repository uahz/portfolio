# 个人作品集 · 单文件版

面向面试官与潜在客户的单页作品集。**单个完整 HTML 文件**，内嵌全部 CSS 与 JS，
无外部依赖、无构建步骤，双击即可运行。

- 线上地址：<https://uahz.github.io/portfolio/>
- 技术栈：原生 HTML + CSS + JS，无框架、无 CDN、无字体文件外链
- 封面图：内联 SVG data URI，同样零外部依赖

## 设计约束

| 项 | 规范 |
|---|---|
| 配色 | 主色 `#165DFF`、背景 `#FAFAFA`、正文 `#1D1D1F`、次要文字 `#86868B`，全部收敛到 `:root` 变量 |
| 字体 | 标题 `Sora`（几何无衬线，有辨识度且非 AI 高频模板字体），正文 `Noto Sans SC`；均为系统 fallback，不加载字体文件 |
| 布局 | 12 列栅格；Hero 7/5 分栏，主按钮落在右下角；画廊为 Bento 错落网格（跨度 7/5/8/4 交替 + 垂直偏移） |
| 圆角 | 统一 12px（`--r`），大容器 20px |
| 阴影 | 低透明度柔和阴影 `--sh-1/2/3`，无厚重黑影 |
| 断点 | 375 / 768 / 1200 三档；768 为平板档（两列画廊 + 桌面菜单），375 为移动档（单列 + 汉堡菜单） |
| 背景 | 极淡径向渐变三层，营造空间深度，无光斑 / 粒子 / 网格纹理 |
| 动效 | 全站仅一套入场动效（`.reveal` + `animation-delay` 错峰）；hover 仅卡片上浮 4px + 封面 1.06 倍慢速缩放；全部纯 CSS |

## 目录

```
index.html            ← 交付物：单文件成品，直接双击打开
template.html         ← 源模板（含 __COVER_xx__ 占位符）
gen-covers.py         ← 封面 SVG 生成器（几何构成 + 主色点缀）
build.py              ← 注入封面 data URI → index.html
check.cjs             ← 浏览器实测（CDP 驱动 Edge，38 项断言）
shoot.cjs             ← 截图脚本
deploy-single.cjs     ← GitHub Pages 部署（Git Data API）
```

## 本地预览

```bash
python -m http.server 8920
# 打开 http://127.0.0.1:8920/index.html
```

也可以直接双击 `index.html` —— 没有任何外部依赖，file:// 下也能完整运行。

## 重新构建

改了 `template.html` 或 `gen-covers.py` 之后：

```bash
python build.py
```

`build.py` 会把 `gen-covers.py` 生成的几何封面转成 data URI，注入模板里的
`__COVER_S1__` … `__COVER_S6__` 占位符，产出最终 `index.html`。

## 替换成自己的内容

| 要改什么 | 位置 |
|---|---|
| 姓名 / 标题 / 简介 | `template.html` 的 `.hero__title`、`.hero__desc` |
| 导航项 | `.nav__menu` |
| 作品数据 | 页面底部 `<script>` 里的 `WORKS` 数组 |
| 作品封面 | `WORKS` 里的 `img` 字段换成你的图片 URL；或改 `gen-covers.py` 的 `SPECS` 重新生成几何占位图 |
| 关于我 / 联系方式 | `#about` 与 `#contact` 区块 |
| 配色 | `:root` 里的 `--c-*` 变量，改一处全站生效 |

`WORKS` 数组每项的字段：`id` / `title` / `cat` / `year` / `img` / `desc` / `tools[]` / `links[]`。

## 验收

```bash
node check.cjs http://127.0.0.1:8920
```

38 项断言，覆盖：渲染完整性、无外部依赖、封面 4:3 比例、Bento 跨度非等宽、
色值全部走变量、入场动效与错峰 delay、导航滚动态（毛玻璃 + 底线）、
弹窗三种关闭方式（关闭按钮 / 蒙层 / ESC）、body 滚动锁定与恢复、
375 / 768 / 1200 三档响应式、**控制台零报错**。

## 部署

```bash
node deploy-single.cjs <TOKEN> portfolio main
```

用 Git Data API（`git/trees` + `git/commits` + `git/refs`）一次提交整棵树，
绕开 git over HTTPS 的代理阻断。脚本会自动开启 Pages、等待构建并做线上验证。

> **编码约定（踩过坑）**：
> - `git/trees` 的 `content` 字段要**纯文本** → `fs.readFileSync(file, "utf8")`
> - Contents API 的 `content` 字段要 **base64** → `Buffer.from(text, "utf8").toString("base64")`
> - 在字符串上写 `.toString("base64")` 是 **no-op**，看着像编码其实没编码，别这么写
>
> 若给 `git/trees` 传了 base64，GitHub 会把那串字符当文件内容原样存下：
> API 返回 200、Pages 显示 built、curl 状态码 200 全部正常，但浏览器打开是一整段
> base64 乱码。部署后务必用 `check-not-base64.cjs` 或 `curl ... | head -c 20` 验内容。
