# 控制台外观皮肤（Console look）

一个**纯视觉层**补丁：把 Carbon 的玻璃拟态/靛蓝配色重绘为「浅灰底 + 白卡 + 细描边 + 玫红强调 + 高信息密度」的控制台风格，并提供一组可复用的展示组件。

## 不做的事

- 不复制任何业务页面文案，不实现任何业务功能、接口调用或状态逻辑；
- 不修改 `header.ut` / `footer.ut` / `carbon.js` / `menu-carbon.js`；
- 不改动 `--cb-*` 之外的第三方类，不覆盖 `#topmenu`、`#indicators`、`#maincontent`、`#tabmenu` 等 LuCI DOM 契约。

开关、徽章、端口图标均为纯 CSS 状态（checkbox / 类名 / mask），页面无需新增脚本。

## 改动文件

| 文件 | 作用 |
| --- | --- |
| `styles/20-console-look.css` | 皮肤主体：令牌重绘 + 壳层覆盖 + `cl-*` 组件 |
| `htdocs/luci-static/carbon/carbon.css` | 由 `build-css.py` 重新拼接的生成产物 |
| `preview/console-look.html` | 无脚本静态预览页（占位文案） |
| `docs/console-look.md` | 本文档 |

`build-css.py` 按文件名排序拼接 `styles/[0-9][0-9]-*.css`，因此 `20-` 会排在所有上游分节之后，无需改构建脚本即可覆盖。删除该文件并重新构建即可完整回退。

## 组件一览（前缀 `cl-`，全部可选）

| 类别 | 类名 |
| --- | --- |
| 区块标题 | `.cl-section` `.cl-section__title` `.cl-section__note` |
| 状态条 | `.cl-meta` `.cl-tag` |
| 开关卡片 | `.cl-grid` `.cl-tile` `.cl-tile__head` `.cl-tile__title` `.cl-tile__desc` `.cl-tile__meta` |
| 开关 | `.cl-switch` > `input` + `.cl-switch__track` |
| 徽章 | `.cl-badge` + `--ok` / `--warn` / `--info` / `--busy` / `--idle` |
| 按钮 | `.cl-actions` `.cl-btn` `.cl-btn--muted` |
| 键值条 | `.cl-kv`（4 列网格，每两个单元格一组） |
| 数据表 | `.cl-table` `.cl-cell-key` `.cl-table__actions` |
| 端口矩阵 | `.cl-ports` `.cl-port` `.cl-port--up` `.cl-port__name` `.cl-port__card` `.cl-port__glyph` `.cl-port__state` `.cl-port__bar` `.cl-port__flow` |
| 工具条装饰 | `.cl-search`（仅外观） |

状态色通过 `--cl-*` 变量定义，`:root[data-darkmode="true"]` 下自动切换到深色版本；`prefers-reduced-motion` 下关闭忙碌徽章的动效。

## 应用与验证

```sh
# 在仓库根目录（源码已含本补丁时无需再 apply）
python3 -I scripts/build-css.py           # 重新生成 carbon.css
python3 -I scripts/build-css.py --check   # CI：产物与 styles/ 一致则通过
git apply console-look.patch              # 在干净的 upstream 副本上应用
```

浏览器打开 `preview/console-look.html` 可查看静态效果（不依赖 LuCI 运行时）。
若只想替换路由器上的样式表而不重新打包，把 `carbon.css` 放到 `/www/luci-static/carbon/` 并强制刷新即可；`header.ut` 只引用这一个 CSS 文件。

## 注意

- 全局覆盖集中在 `20-console-look.css`，会影响所有 LuCI 页面；只想给单个页面用时，可把 `:root` 块改为 `body[data-page^="..."]` 作用域。
- 端口图标使用 CSS `mask` + 内联 SVG data URI，无外部图片请求。
- 打印样式在文件末尾做了保护，避免壳层白卡影响打印输出。
