# Rachel Ji 个人网站 — 设计逻辑 & 协作文档

> 写给未来接手这个网站的协作者。  
> 这份文档描述的不是"代码在哪里"，而是"为什么这样设计"。读完这份文档，再看代码，你就能快速上手。

---

## 一、网站的核心思路

这是一个人的网站，不是简历，不是作品集——是**一个有品味的人的空间**。

RJ 有两个身份：
- **The Creator**（创作者）——纪录片、摄影、小说、音乐、绘画
- **The Builder**（实干者）——语言教学、ScriptBridge 项目、媒体从业经历

这两个身份的受众不同，所以网站的第一个问题是：「你想怎样认识我？」  
访客选择自己感兴趣的一面进入，而不是被迫看所有东西。

---

## 二、技术架构

**单文件静态网站。** 所有 HTML、CSS、JavaScript 都在 `index.html` 一个文件里，部署在 GitHub Pages。

不使用框架，不使用构建工具，不引入 npm。原则：**能看懂就能改**。

### 2.1 主题切换（深色 / 浅色）

用 `data-theme` 属性控制：
```html
<html data-theme="light">  <!-- 浅色 -->
<html>                      <!-- 默认深色（无属性）-->
```

所有颜色都是 CSS 自定义属性（变量），定义在 `:root`（深色默认）和 `[data-theme="light"]`（浅色覆盖）里：
```css
:root { --bg: #080812; --violet: #9D7FE8; --amber: #E8B87A; ... }
[data-theme="light"] { --bg: #F7F3EE; --violet: #6840C0; ... }
```

要改颜色：**只改变量，不改具体元素**。

### 2.2 语言切换（中文 / 英文）

用 `data-lang` 属性控制：
```html
<html data-lang="zh">  <!-- 中文 -->
<html data-lang="en">  <!-- 英文 -->
```

双语内容的写法：同一段话写两遍，分别加 `.zh` / `.en` class：
```html
<div class="work-body en">English text here.</div>
<div class="work-body zh">中文内容在这里。</div>
```

CSS 规则让对应语言的内容自动隐藏：
```css
[data-lang="en"] .zh { display: none; }
[data-lang="zh"] .en { display: none; }
```

### 2.3 视图切换（创作者 / 实干者 / 全部）

用 `data-view` 属性控制：
```html
<html data-view="creative">  <!-- 只看创作 -->
<html data-view="work">      <!-- 只看工作 -->
<html data-view="all">       <!-- 看全部 -->
```

对应的 CSS 规则：
```css
html[data-view="creative"] #work,
html[data-view="creative"] #work-nav,
html[data-view="creative"] #hr-work { display: none; }

html[data-view="work"] #creative,
html[data-view="work"] #creative-nav { display: none; }
```

JavaScript 切换函数：
```js
function setView(v) { document.documentElement.setAttribute('data-view', v); }
```

---

## 三、页面结构

```
Gateway（入口选择屏）
  ↓ 选择
Nav（固定顶部导航）
  ↓
#creative（创作板块）
  ↓
#creative-nav（板块尾部导航 → 引导去看工作板块）
  ↓
<hr id="hr-work">（分隔线）
  ↓
#work（工作板块）
  ↓
#work-nav（板块尾部导航 → 引导回看创作板块）
  ↓
#contact（联系方式）
  ↓
Footer
```

### 3.1 Gateway（入口选择屏）

第一次访问时全屏显示。提问：「你想怎样认识我？」  
两个选项：The Creator / The Builder，点击后进入对应视图并平滑滚动到板块。  
还有「or explore everything ↓」跳过选择，看全部内容。

技术细节：
- 用 `sessionStorage` 记录"本次会话已访问"，刷新后不再重复出现
- 点 Logo 可以重新触发 Gateway（`showGateway()` 函数）
- Gateway 消失动画：先加 `.leaving` class（淡出 + 上移），750ms 后加 `.gone` class（`display: none`）

### 3.2 板块尾部导航（Section Nav）

每个板块结束时有一个导航区，**只有一个方向**：
- 创作板块尾部 → 「See Work Experience →」（指向工作板块）
- 工作板块尾部 → 「See Creative Work →」（指向创作板块）

**规则：两个方向分开，不能同一个里面两个方向都有。**  
这是为了保持引导的清晰性——每个出口只做一件事。

---

## 四、组件规范

### 4.1 手风琴工作卡片（Work Accordion）

工作经历用可展开卡片呈现。默认收起，点击展开。

HTML 结构：
```html
<div class="work-card" onclick="toggleWork(this)">
  <div class="work-card-header">
    <div class="work-card-left">
      <div class="work-card-role en">Role Title</div>
      <div class="work-card-role zh">职位名称</div>
      <div class="work-card-name en">Organization Name</div>
      <div class="work-card-name zh">机构名称</div>
      <div class="work-card-period en">Year</div>
      <div class="work-card-period zh">年份</div>
    </div>
    <span class="work-card-hint en">See more →</span>
    <span class="work-card-hint zh">点击查看 →</span>
    <span class="work-card-arrow">→</span>
  </div>
  <div class="work-card-body">
    <div class="work-card-body-inner">
      <!-- 详细内容 -->
    </div>
  </div>
</div>
```

展开机制：
- 收起时：`.work-card-body` 的 `max-height: 0`
- 展开时：`.work-card.open .work-card-body` 的 `max-height: 1200px`（用 CSS transition 做动画）
- 每次只能开一张（accordion 行为）：`toggleWork()` 会先关闭所有已开的卡片

`work-card-hint`（「点击查看 →」提示）在卡片打开时透明度降为 0，关闭时恢复显示。

**特殊卡片：`featured` class**  
ScriptBridge 项目卡片用 `class="work-card featured"`，有额外的视觉高亮（紫色边框），内部有统计数字区和功能标签区。

### 4.2 模态框（Modal）

摄影、写作等内容用模态框展示。

打开/关闭：
```js
openModal('modalPhoto');   // 打开
closeModal('modalPhoto');  // 关闭
closeModalOutside(event);  // 点击遮罩关闭
```

CSS：`.modal-overlay` 默认隐藏，加 `.open` class 后显示。

图片展示原则：**不裁剪**。用 `object-fit: contain` + `height: auto`，让图片以原始比例完整显示。

### 4.3 Logo

SVG 内联，Playfair Display 斜体，R 在上（苍绿色）、J 在下（珊瑚红），垂直叠放。

关键参数：
- `viewBox="0 0 64 128"`（高度要够容纳 J 的尾部，否则会被截断）
- R 颜色：`var(--logo-r)`（深色 `#5DAA88`，浅色 `#2A7850`）
- J 颜色：`var(--logo-j)`（深色 `#D46058`，浅色 `#A83830`）

点击 Logo → 触发 `showGateway()`，返回入口选择屏。

---

## 五、内容分区规则

**创作板块**（`#creative`）= 艺术性、个人性质的内容：
- 纪录片 / 短片
- 摄影作品
- 小说 & 写作
- 音乐 & 唱歌
- 绘画

**工作板块**（`#work`）= 有经济价值、专业履历类的内容：
- 教学（语言老师）
- 创业项目（ScriptBridge）
- 媒体从业经历（昆明电视台、云南网、苍山博物馆 等）
- 技术零售（Apple 授权店）

**如果一条经历有歧义**：先确认归属再写代码，不要擅自决定。

---

## 六、素材文件存放规则

**所有图片必须在 `01 开发/` 目录内**，与 `index.html` 并排。  
原因：GitHub Pages 部署的是这个文件夹，上一级的 `02 素材/` 目录**不在 git repo 里**，线上访问不到。

引用方式：`src="photo.jpg"`（相对路径，不加前缀）。

当前已有图片：
```
01 开发/
├── photo.jpg               ← 个人主照片
├── still_003.jpg           ← 《不能说的秘密》封面帧
├── still_007.jpg           ← 《他们也是她们》封面帧
├── photo_zhan_1.jpeg       ← 摄影：展览系列
├── photo_zhan_2.jpeg
├── photo_danche_1.jpeg     ← 摄影：单车系列
├── photo_danche_2.jpeg
├── photo_ziyou.jpeg        ← 摄影：自由
├── photo_meng_1.jpeg       ← 摄影：梦系列
├── photo_meng_2.jpeg
├── photo_shengming_1.jpeg  ← 摄影：生命系列
└── photo_shengming_2.jpeg
```

---

## 七、修改流程（每次迭代前必须走这个流程）

### 第一步：采访（**不得跳过直接写代码**）

在动手之前，必须先问清楚：
1. 这次要改/加什么？（内容、视觉、交互？）
2. 涉及哪个板块？（创作 / 工作 / 全局？）
3. 有没有新素材？（图片要提前放好）
4. 访客如何操作？（点击展开？弹窗？跳转？）
5. 文字由谁来写？（RJ 提供还是 Claude 起稿后确认？）

### 第二步：列清单确认

采访完，写出「这次要改的内容清单」，让 RJ 确认后再写代码。

### 第三步：写代码

按确认后的清单执行，不擅自加料。

### 第四步：推送

```bash
cd "03 项目/个人网站/01 开发"
git add .
git commit -m "简述本次改动"
git push
```

---

## 八、版本历史摘要

| 日期 | 主要改动 |
|------|----------|
| 2026-05 初 | 建站：基础结构、配色、双语切换 |
| 2026-05 中 | 加入 Gateway 入口选择屏 |
| 2026-05 中 | 摄影作品从 PDF 提取并接入，改为模态框展示 |
| 2026-05 中 | 工作板块重构为手风琴卡片 |
| 2026-05 中 | 语言老师条目合并为「Language Teacher」一张卡片 |
| 2026-05 中 | 修正：英文老师 = 独立平台，中文老师 = Preply |
| 2026-05 中 | 板块尾部导航改为单向（各自只指向对方） |
| 2026-05 中 | 点 Logo 返回 Gateway |
| 2026-05-08 | 删除大姚县融媒体中心条目（无薪经历，不对外展示） |

---

## 九、快速参考：常见修改

| 要做什么 | 怎么做 |
|----------|--------|
| 改文字 | 找对应的 `.en` / `.zh` 元素，直接改文本内容 |
| 加一条工作经历 | 复制一个 `work-card` 块，改内容，放在合适位置 |
| 加一张创作卡片 | 复制现有创作卡片结构，改内容和图片 |
| 加图片 | 把图片文件放到 `01 开发/` 目录，用文件名引用 |
| 改颜色 | 只改 `:root` 或 `[data-theme="light"]` 里的变量 |
| 改 Logo 颜色 | 改 `--logo-r` 和 `--logo-j` 变量 |
| 加语言 | 加 `data-lang="xx"` CSS 规则 + 对应 class 的内容 |

---

*文档由 Claude (Claudian) 生成，最后更新：2026-05-08*
