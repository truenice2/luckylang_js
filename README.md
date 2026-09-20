# luckylang_js

幸福语言 **LuckyLang** 的 wasm（静态模式）构建产物，由 Emscripten 编译，经 **jsDelivr** 分发。
网页通过 `<script>` 引入加载器，加载器按需拉取同目录的 `luckylang.wasm`，浏览器内直接跑幸福语言。

构建来源：`D:\luckylang\build\wasm\`（`luckylang.js` + `luckylang.wasm`）

## jsDelivr 链接

| 文件 | 链接 |
| --- | --- |
| 加载器 | `https://cdn.jsdelivr.net/gh/truenice2/luckylang_js@main/build/luckylang.js` |
| wasm | `https://cdn.jsdelivr.net/gh/truenice2/luckylang_js@main/build/luckylang.wasm` |
| 目录列表 | `https://cdn.jsdelivr.net/gh/truenice2/luckylang_js@main/build/` |

- 固定版本（推荐，避免缓存刷新延迟）：把 `@main` 换成 tag，如 `@v2.2.0`。
- jsDelivr 对 gh 仓库有缓存，新推送后若还是旧文件，访问
  `https://purge.jsdelivr.net/gh/truenice2/luckylang_js@main/build/luckylang.wasm` 清缓存。

### 镜像 / 代理（国内加速）

需求里列的 gh 代理同样可以套在前面：

```
https://ghfast.top/https://raw.githubusercontent.com/truenice2/luckylang_js/main/build/luckylang.js
https://v6.gh-proxy.org/https://cdn.jsdelivr.net/gh/truenice2/luckylang_js@main/build/luckylang.wasm
https://hk.gh-proxy.org/https://cdn.jsdelivr.net/gh/truenice2/luckylang_js@main/build/luckylang.wasm
https://cdn.gh-proxy.org/https://cdn.jsdelivr.net/gh/truenice2/luckylang_js@main/build/luckylang.wasm
https://edgeone.gh-proxy.org/https://cdn.jsdelivr.net/gh/truenice2/luckylang_js@main/build/luckylang.wasm
```

## wasm 导出接口

| 导出 | 说明 |
| --- | --- |
| `lk_wasm_版本()` | 版本串，如 `2.2.0-wasm` |
| `lk_wasm_平台名()` | `"wasm"` |
| `lk_wasm_运行(源码)` | 跑一段幸福语言源码，返回 0=成功 |
| `lk_wasm_错误()` | 最近一次错误说明 |
| `lk_wasm_渲染福页(lp源码)` | `.lp` → HTML |
| `lk_wasm_幸福块数(lp源码)` | 页面里 `幸福(){}` / `脚本(){}` 块数 |
| `lk_wasm_提取代码(lp源码, 种类, 序号)` | 取单个代码块原文 |
| `lk_wasm_运行页面脚本(lp源码)` | 渲染并执行页内 `幸福(){}` 块 |
| `lk_wasm_释放(p)` | 释放上面返回的 malloc 串 |

`说` / `输出` 走 emscripten stdout，交给 `Module.print` 回调。

## 用法

### Node.js

```js
import { createRequire } from 'node:module';
const require = createRequire(import.meta.url);

const CDN = 'https://cdn.jsdelivr.net/gh/truenice2/luckylang_js@main/build';

// Emscripten 的 locateFile 指向 CDN，wasm 就不用下到本地
const { default: LuckyLang } = require(`${CDN}/luckylang.js`);

const Module = await LuckyLang({
  locateFile: (p) => `${CDN}/${p}`,
  print: (s) => console.log(s),
});

// 需要手动 fetch 一份 wasm 给 Node（Node 里没有浏览器的自动 fetch）
const wasmBinary = await (await fetch(`${CDN}/luckylang.wasm`)).arrayBuffer();

Module.ccall('lk_wasm_运行', 'number', ['string'], ['说"你好，幸福语言"。']);
```

完整可跑的测试见 `test/node测试.mjs`。

### 浏览器

```html
<script src="https://cdn.jsdelivr.net/gh/truenice2/luckylang_js@main/build/luckylang.js"></script>
<script>
  const Module = await LuckyLang({ print: (s) => console.log(s) });
  Module.ccall('lk_wasm_运行', 'number', ['string'], ['说"你好，幸福语言"。']);
</script>
```

静态 `.htm` 页面里（需求里 `<lkp>头(){样()} 体(){块(){}}</lkp>` 那种写法）：

```html
<lkp>
头(){
    脚本(源="https://cdn.jsdelivr.net/gh/truenice2/luckylang_js@main/build/luckylang.js")
    幸福(){
        说"你好，幸福语言"。
    }
}
体(){ 块(){ 标题1(){ 幸福语言 wasm } } }
</lkp>
```
