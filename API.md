# OFDElement 组件 API 文档

## 概述

`OFDElement` 是 ofd-lzy 的核心查看器组件，支持在浏览器中解析并渲染 OFD（GB/T 33190）文档。

### 安装注册

```js
// 全局注册（Vue 插件方式）
import OfdLzy from 'ofd-lzy'
Vue.use(OfdLzy)

// 或按需引入
import { OFDElement } from 'ofd-lzy'
export default {
  components: { OFDElement }
}
```

### 基本用法

```html
<OFDElement
  ref="ofd"
  url="/path/to/document.ofd"
  @loaded="onLoaded"
  @page-change="onPageChange"
/>
```

通过 `ref` 调用程序化 API：

```js
this.$refs.ofd.api.nextPage()
this.$refs.ofd.api.setZoom(1.5)
```

---

## Props

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `url` | `string` | `''` | OFD 文档地址（URL 或 base64 data URI）。变更后自动重新加载文档 |
| `freetypeEnabled` | `boolean` | `true` | 是否启用 FreeType WASM 进行内嵌字体字形渲染。关闭后退回浏览器原生字体引擎 |
| `freetypeMaxMemoryMB` | `number` | `30` | FreeType 字体缓存内存上限（MB）。超出后新字体跳过 FreeType，改用系统字体回退 |
| `renderScaleFactor` | `number` | `1.5` | 画布超采样系数（`1` ~ `2`）。低 DPI 屏（devicePixelRatio=1）下超采样可使文字边缘更清晰；高分屏（devicePixelRatio≥2）下不叠加超采样 |

---

## Events

| 事件名 | 参数 | 触发时机 |
|--------|------|----------|
| `loaded` | `{ totalPages: number, docInfo: object }` | 文档解析完成，页数与文档信息就绪 |
| `page-change` | `(currentPage: number, totalPages: number)` | 当前可见页码发生变化（翻页/滚动） |
| `zoom-change` | `(value: number \| 'fit-width' \| 'fit-page', actualScale: number)` | 缩放级别变化 |
| `rotate-change` | `(rotation: number)` | 页面旋转角度变化（`0` / `90` / `180` / `270`） |

### 示例

```html
<OFDElement
  url="/doc.ofd"
  @loaded="({ totalPages, docInfo }) => console.log('总页数:', totalPages)"
  @page-change="(page, total) => console.log(`第 ${page} / ${total} 页`)"
  @zoom-change="(val) => console.log('缩放:', val)"
  @rotate-change="(deg) => console.log('旋转:', deg)"
/>
```

---

## 程序化 API（`ref.api.*`）

通过组件 `ref` 访问 `api` 对象调用各项功能：

```js
const api = this.$refs.ofd.api
```

### 文档加载

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `loadUrl(url)` | `url: string` | `void` | 加载或切换 OFD 文档 |
| `isLoaded()` | — | `boolean` | 文档是否已加载完成 |

### 翻页导航

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `goToPage(page)` | `page: number` | `void` | 跳转到指定页码（从 `1` 开始） |
| `previousPage()` | — | `void` | 上一页 |
| `nextPage()` | — | `void` | 下一页 |
| `firstPage()` | — | `void` | 跳转到第一页 |
| `lastPage()` | — | `void` | 跳转到最后一页 |
| `getCurrentPage()` | — | `number` | 当前页码 |
| `getTotalPages()` | — | `number` | 文档总页数 |

### 缩放

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `zoomIn()` | — | `void` | 放大一档 |
| `zoomOut()` | — | `void` | 缩小一档 |
| `setZoom(value)` | `value: number \| 'fit-width' \| 'fit-page'` | `void` | 设置缩放级别。数值表示比例（如 `1.5` = 150%）；`'fit-width'` 适合宽度；`'fit-page'` 适合整页 |
| `fitWidth()` | — | `void` | 适合容器宽度 |
| `fitPage()` | — | `void` | 适合容器整页 |
| `getZoom()` | — | `number` | 当前实际缩放比例 |

档位列表：`0.25 / 0.5 / 0.75 / 1 / 1.25 / 1.5 / 2 / 3 / 4 / 6`

### 旋转

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `rotateLeft()` | — | `void` | 逆时针旋转 90° |
| `rotateRight()` | — | `void` | 顺时针旋转 90° |
| `getRotation()` | — | `0 \| 90 \| 180 \| 270` | 当前旋转角度 |

### 视图工具

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `setTool(tool)` | `tool: 'select' \| 'hand'` | `void` | 切换工具模式。`'select'`：选择文字/点选印章/绘制批注；`'hand'`：按住拖动平移 |
| `getTool()` | — | `'select' \| 'hand'` | 当前工具模式 |
| `panBy(dx, dy)` | `dx: number, dy: number` | `boolean` | 按像素增量平移文档区域（正值向右/向下）。返回是否实际发生了滚动 |
| `canScrollHorizontally()` | — | `boolean` | 页面宽度是否超出视窗（可横向滚动） |

### 查找

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `openSearch()` | — | `void` | 打开查找对话框 |
| `search(keyword, options?)` | `keyword: string`<br>`options?: { caseSensitive?: boolean; fuzzyMatch?: boolean }` | `Promise<SearchResult[]>` | 执行查找并高亮所有匹配项，返回结果列表 |

`SearchResult` 类型：

```ts
interface SearchResult {
  page: number         // 所在页码（从 1 开始）
  text: string         // 匹配文本片段（含上下文）
  elementIndex: number // 页面内文字对象的下标
}
```

### 面板控制

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `toggleLeftPanel(visible?)` | `visible?: boolean` | `void` | 显示/隐藏左侧面板（缩略图 / 大纲 / 签章 / 批注）。不传参时切换 |
| `toggleRightPanel(visible?)` | `visible?: boolean` | `void` | 显示/隐藏右侧工具面板。不传参时切换 |
| `showProperty()` | — | `void` | 打开文档属性弹窗 |

### 打印与导出

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `print()` | — | `void` | 打印文档（通过隐藏 iframe 渲染后调用浏览器打印） |
| `toPdf()` | — | `Promise<void>` | 导出为 PDF（浏览器打印另存为 PDF 方式） |
| `toImage()` | — | `Promise<void>` | 导出为图片（逐页 PNG / JPEG，自动下载） |
| `toSvg()` | — | `Promise<void>` | 导出当前页为 SVG 文件 |
| `extractText()` | — | `void` | 提取全文文本并导出为 JSON 文件 |
| `exportInvoiceData()` | — | `void` | 提取发票结构化字段（发票代码、号码、日期、金额等）并导出为 JSON |
| `downloadAnnotatedOfd()` | — | `Promise<void>` | 导出含用户批注的 OFD 文件 |

### 签章

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `verifyStamps()` | — | `void` | 校验文档内所有签章的有效性并展示验证报告 |

### 文档信息

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `getDocInfo()` | — | `DocInfo \| null` | 文档基本信息（标题、作者、创建时间、主题等） |
| `getFonts()` | — | `FontResource[]` | 文档字体资源列表 |

`DocInfo` 类型（部分字段）：

```ts
interface DocInfo {
  title?: string
  author?: string
  subject?: string
  keywords?: string
  creationDate?: string
  modificationDate?: string
  creator?: string
  docRoot: string
}
```

`FontResource` 类型：

```ts
interface FontResource {
  id: string
  fontName: string
  familyName: string
  charset: string
  style: string       // 如 'Regular' / 'Bold' / 'Italic'
  embedded: boolean   // 是否内嵌字体文件
}
```

---

## 完整示例

### Vue 2

```html
<template>
  <div>
    <button @click="prev">上一页</button>
    <button @click="next">下一页</button>
    <span>{{ currentPage }} / {{ totalPages }}</span>
    <button @click="zoomIn">放大</button>
    <button @click="zoomOut">缩小</button>

    <OFDElement
      ref="ofd"
      :url="fileUrl"
      :freetype-enabled="true"
      :render-scale-factor="1.5"
      @loaded="onLoaded"
      @page-change="onPageChange"
    />
  </div>
</template>

<script>
export default {
  data() {
    return {
      fileUrl: '/example.ofd',
      currentPage: 1,
      totalPages: 0,
    }
  },
  methods: {
    onLoaded({ totalPages, docInfo }) {
      this.totalPages = totalPages
      console.log('文档标题:', docInfo?.title)
    },
    onPageChange(page, total) {
      this.currentPage = page
      this.totalPages = total
    },
    prev()   { this.$refs.ofd.api.previousPage() },
    next()   { this.$refs.ofd.api.nextPage() },
    zoomIn() { this.$refs.ofd.api.zoomIn() },
    zoomOut(){ this.$refs.ofd.api.zoomOut() },
    async searchDoc() {
      const results = await this.$refs.ofd.api.search('关键词', { caseSensitive: false })
      console.log('查找结果:', results)
    },
  }
}
</script>
```

### Vue 3 / Composition API

```html
<template>
  <OFDElement
    ref="ofdRef"
    :url="fileUrl"
    @loaded="onLoaded"
  />
</template>

<script setup>
import { ref } from 'vue'
const ofdRef = ref(null)
const fileUrl = ref('/example.ofd')

function onLoaded({ totalPages }) {
  console.log('共', totalPages, '页')
}

function nextPage() {
  ofdRef.value.api.nextPage()
}
</script>
```

### 纯 HTML（UMD）

```html
<script src="dist/lib/ofd-lzy.umd.min.js"></script>

<div id="app">
  <ofd-element ref="ofd" url="/example.ofd" @loaded="onLoaded"></ofd-element>
</div>

<script>
Vue.use(OfdLzy.default)
new Vue({
  el: '#app',
  methods: {
    onLoaded({ totalPages }) {
      console.log('总页数:', totalPages)
      this.$refs.ofd.api.fitWidth()
    }
  }
})
</script>
```

---

## 注意事项

- `api` 对象在文档未加载（`isLoaded()` 返回 `false`）时，导航/导出等方法调用无效果，部分方法会显示提示信息。
- `loadUrl()` 会销毁当前渲染上下文并重新初始化，切换文档时调用。
- `toImage()` / `toPdf()` 等导出方法依赖浏览器的 Canvas 和 Blob API，在 SSR 环境中不可用。
- `freetypeEnabled` 和 `renderScaleFactor` 只在组件初始化时生效，运行时更改不会触发重新渲染。
