---
name: gwyxingce
description: "公务员考试行政职业能力测验（行测）题目分析器。当用户发送行测题目（通常是拍照图片）时使用此skill。为每道题生成包含7个分析维度的白色打印友好HTML文件（解题思路、解题过程、知识点拆解、思维方式、易错点、变式题、难度定位），同时维护一个汇总索引页面用于备考诊断。触发词：'做这道题'、'分析这道题'、'这题怎么做'、发送行测题目图片。"
---

# 公务员行测题目分析器

为公务员考试行政职业能力测验（行测）题目生成详细的7维分析HTML文件，并维护备考诊断索引。不区分国考/省考/事业单位——按通用行测知识体系归类。

## 工作流程

### Step 0: 学科判断（前置检查）

在做任何事之前，先判断输入是否为行测题目。

**判断标准：** 题目属于行测五大模块之一（言语理解、数量关系、判断推理、资料分析、常识判断）。

**不是行测题的典型情况：**
- 申论题目（主观题，需要写作）
- 高中/初中学科题目
- 其他非行测内容

**若判断为非行测题：** 输出以下提示后**立即终止**，不执行后续任何步骤，不生成任何文件：

> 这道题不属于行测范畴，/gwyxingce 只处理行测题目（言语理解、数量关系、判断推理、资料分析、常识判断）。

**若判断为行测题：** 继续执行后续步骤。

---

### Step 1: 初始化

工作目录：`~/Desktop/jiaxiai/xingce/`

1. 检查 `images/` 子目录是否存在，不存在则 `mkdir -p ~/Desktop/jiaxiai/xingce/images`
2. 检查 `problem.css` 是否存在，不存在则使用本文件中的 **CSS_TEMPLATE** 生成（此文件只需生成一次，所有题目共用）
3. 检查 `index.html` 是否存在，不存在则使用本文件末尾的 **INDEX_TEMPLATE** 生成
4. 统计已有 `problem_*.html` 文件数量，下一个编号 = 已有数量 + 1（三位数补零：001、002...）

### Step 2: 读取题目

**图片输入（常见）：**
1. 将用户提供的图片复制到 `~/Desktop/jiaxiai/xingce/images/problem_NNN.jpg`
2. 如果原图 >2MB，使用 `sips --resampleWidth 1200 <src> --out <dst>` 压缩
3. 从图片中仔细读取题目内容，完整理解题干和ABCD选项

**文字输入：**
- 跳过图片处理，HTML中省略图片区域

### Step 3: 分析题目（7个维度）

#### 维度一：解题思路
- 放在题目图片之后、正式解法之前
- 展示做题前的思考过程：不是标准答案，而是"怎么想到的"
- 关键要素：
  - 审题：识别题型，提取关键条件，第一反应用什么方法
  - 方向：排除法/代入法/逻辑推演/速算，选择哪条路
  - 突破：找到关键突破口的思维节点
  - 陷阱：命题人设了什么干扰项，怎么识别
- 控制篇幅：3-4个思考节点，每个2-3句话
- 语气：像一个刷题老手说"我是这么想的"，不是教科书式叙述
- HTML使用时间轴样式（`.thought-flow` + `.thought-step`），关键洞察节点加 `.ts-insight` 类

#### 维度二：解题过程
- 核心是**选项分析**：逐个说明每个选项为什么正确或错误
- 至少提供2种解法（如有）：通法 + 秒杀技巧
- 选项用 `.option-analysis` + `.option-item.correct/wrong` 样式展示
- 标注解法特点：通法/排除法/代入法/速算法/秒杀
- 常识判断题：展示正确答案的知识点依据，其他选项说明排除理由
- 数量关系/资料分析：公式用 LaTeX（`$...$` 行内，`$$...$$` 独立行）

#### 维度三：知识点拆解
- 格式：`大模块 > 中模块 > 具体题型/知识点`
- 列出所有涉及知识点，包括隐含的
- 不区分国考/省考版本，按通用行测知识体系分类

**主模块分类（用于索引统计）：**
言语理解、数量关系、判断推理、资料分析、常识判断

**各模块常见二级分类：**
- 言语理解：逻辑填空、片段阅读、语句排序、句子填空
- 数量关系：行程问题、工程问题、排列组合、概率、数字规律、统计
- 判断推理：图形推理、类比推理、定义判断、逻辑判断（演绎/削弱加强/归纳）
- 资料分析：增长率、增长量、比重、倍数、平均数、综合计算
- 常识判断：政治法律、经济、历史文化、地理科技

#### 维度四：思维方式
- 逐步标注每个关键节点所用思维
- 格式：`【步骤描述】→ 思维名称：为什么在这用`
- 行测常见：排除法、代入法、特殊值法、比例思维、逆向思维、逻辑推演、关键词定位、语境分析、速算估算、图形规律、类比推理、反驳法

#### 维度五：易错点与陷阱
- 2-4个常见错误
- 每个：❌ 错误做法 → ✅ 正确做法 → 原因分析
- 重点标注命题人的干扰项设计逻辑

#### 维度六：变式题方向
- 2-3个变式
- 每个：改什么条件/题型 → 变成什么题 → 核心区别

#### 维度七：难度定位
- 等级：容易(1) / 中等(2) / 较难(3) / 高难(4)
- 预计正确率（全体考生水平）
- 建议用时（行测单题参考时间）
- 考试策略：容易题秒过；中等题正常作答；较难题超时立刻猜跳；高难题直接跳过不死磕

### Step 4: 生成题目HTML文件

参照 **PROBLEM_TEMPLATE** 的结构生成 `problem_NNN.html`，保存到 `~/Desktop/jiaxiai/xingce/`。

**要求：**
- 数学公式用 LaTeX，KaTeX 自动渲染（数量关系、资料分析用得上）
- 七个section用HTML结构化，不堆纯文本
- 每个section的div加 `id` 属性用于锚点跳转（思路/解法/知识点/思维/易错/变式/难度）
- 样式由外部 `problem.css` 提供，HTML 中只需 `<link rel="stylesheet" href="problem.css">`，**不内联任何 CSS**
- 如有图片，用 `<img src="images/problem_NNN.jpg">`

### Step 5: 更新索引

读取 `~/Desktop/jiaxiai/xingce/index.html`，在 `var problems = [...]` 数组末尾、`];` 之前追加：

```javascript
{
  id: NNN,
  date: 'YYYY-MM-DD',
  file: 'problem_NNN.html',
  title: '题目简短描述',
  difficulty: N,
  diffLabel: '中等',
  modules: ['判断推理'],
  moduleDetails: ['判断推理 > 逻辑判断 > 削弱论证'],
  thinking: ['排除法', '逻辑推演'],
  source: '2023国考真题'
}
```

### Step 6: 输出确认

简要告知：编号、文件路径、主要知识点、难度。

---

## PROBLEM_TEMPLATE

每道题目的HTML页面。白色主题、简洁、适合打印。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>题目 #NNN — 简短描述</title>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.css">
<link rel="stylesheet" href="problem.css">
</head>
<body>
<button class="print-btn no-print" onclick="window.print()" title="打印">&#128424;</button>
<div class="nav"><a href="index.html">&larr; 返回索引</a></div>

<div class="header">
  <span class="problem-id"># NNN</span>
  <div class="header-right">
    <span>YYYY-MM-DD</span>
    <span>来源：XXX</span>
    <span class="difficulty-badge diff-N">难度标签</span>
  </div>
</div>

<div class="problem-image-section">
  <img src="images/problem_NNN.jpg" alt="题目" class="problem-image">
</div>

<div class="section" id="思路">
  <h2 class="section-title st-0">一、解题思路</h2>
  <div class="thought-flow">
    <div class="thought-step">
      <div class="thought-label" style="color:#2563eb">审题 — 识别题型</div>
      <div class="thought-text">判断属于哪个模块哪种题型，提取关键条件</div>
    </div>
    <div class="thought-step">
      <div class="thought-label" style="color:#f59e0b">方向 — 选择方法</div>
      <div class="thought-text">第一反应用什么方法，为什么选这条路</div>
    </div>
    <div class="thought-step ts-insight">
      <div class="thought-label" style="color:#10b981">突破 — 关键洞察</div>
      <div class="thought-text">找到突破口的核心思维，命题人设了什么陷阱</div>
    </div>
  </div>
</div>

<div class="section" id="解法">
  <h2 class="section-title st-1">二、解题过程</h2>
  <div class="method">
    <div class="method-title">解法一 <span class="method-tag">通法</span></div>
    <div class="step">推理过程说明（数量关系/资料分析可用 $...$ 公式）</div>
    <div class="option-analysis">
      <div class="option-item correct">
        <span class="option-label">A</span>
        <div class="option-content">
          <div class="option-text">选项内容</div>
          <div class="option-reason">正确原因分析</div>
        </div>
      </div>
      <div class="option-item wrong">
        <span class="option-label">B</span>
        <div class="option-content">
          <div class="option-text">选项内容</div>
          <div class="option-reason">排除原因分析</div>
        </div>
      </div>
      <div class="option-item wrong">
        <span class="option-label">C</span>
        <div class="option-content">
          <div class="option-text">选项内容</div>
          <div class="option-reason">排除原因分析</div>
        </div>
      </div>
      <div class="option-item wrong">
        <span class="option-label">D</span>
        <div class="option-content">
          <div class="option-text">选项内容</div>
          <div class="option-reason">排除原因分析</div>
        </div>
      </div>
    </div>
  </div>
</div>

<div class="section" id="知识点">
  <h2 class="section-title st-2">三、知识点拆解</h2>
  <div class="knowledge-item">
    <span class="module-tag">判断推理</span>
    判断推理 <span class="arrow">&gt;</span> 逻辑判断 <span class="arrow">&gt;</span> 削弱论证
  </div>
</div>

<div class="section" id="思维">
  <h2 class="section-title st-3">四、思维方式</h2>
  <div class="thinking-item">
    <div>
      <div class="thinking-step">【识别论证结构】</div>
      <div class="thinking-why">明确前提和结论，才能找到削弱点</div>
    </div>
    <span class="thinking-method">逻辑推演</span>
  </div>
</div>

<div class="section" id="易错">
  <h2 class="section-title st-4">五、易错点与陷阱</h2>
  <div class="pitfall">
    <div class="pitfall-wrong">错误做法描述</div>
    <div class="pitfall-right">正确做法描述</div>
    <div class="pitfall-why">原因分析（含命题人干扰项设计逻辑）</div>
  </div>
</div>

<div class="section" id="变式">
  <h2 class="section-title st-5">六、变式题方向</h2>
  <div class="variation">
    <div class="variation-title">变式一：改变条件描述</div>
    <div class="variation-desc">变成什么新题，核心区别是什么</div>
  </div>
</div>

<div class="section" id="难度">
  <h2 class="section-title st-6">七、难度定位</h2>
  <div class="difficulty-box">
    <div class="diff-row"><span class="diff-label">难度等级</span><span class="diff-value">中等</span></div>
    <div class="diff-row"><span class="diff-label">预计正确率</span><span class="diff-value">约55%</span></div>
    <div class="diff-row"><span class="diff-label">建议用时</span><span class="diff-value">60-90秒</span></div>
    <div class="diff-row"><span class="diff-label">考试策略</span><span class="diff-value">正常作答，超时立刻猜答案跳过</span></div>
  </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/contrib/auto-render.min.js"></script>
<script>
document.addEventListener("DOMContentLoaded",function(){
  renderMathInElement(document.body,{
    delimiters:[
      {left:'$$',right:'$$',display:true},
      {left:'$',right:'$',display:false},
      {left:'\\(',right:'\\)',display:false},
      {left:'\\[',right:'\\]',display:true}
    ],
    throwOnError:false
  });
});
</script>
<div style="text-align:center;padding:16px;color:#9ca3af;font-size:12px;border-top:1px solid #e5e7eb;margin-top:24px"><a href="http://www.he321.com/xingce" style="color:inherit;text-decoration:none">嘉析AI</a>官网：<a href="http://www.he321.com/xingce" style="color:inherit;text-decoration:underline">www.he321.com/xingce</a></div>
</body>
</html>
```

---

## CSS_TEMPLATE

所有题目 HTML 共用的样式表。Step 1 检查 `~/Desktop/jiaxiai/xingce/problem.css` 不存在时写入，**只写一次**。

```css
*{margin:0;padding:0;box-sizing:border-box}
body{font-family:-apple-system,'PingFang SC','Microsoft YaHei','Noto Sans SC',sans-serif;
  max-width:800px;margin:0 auto;padding:32px 24px;color:#1a1a1a;background:#fff;line-height:1.8;font-size:15px}

.nav{font-size:12px;color:#9ca3af;margin-bottom:20px}
.nav a{color:#2563eb;text-decoration:none}
.nav a:hover{text-decoration:underline}

.header{display:flex;align-items:center;justify-content:space-between;
  padding-bottom:14px;border-bottom:2px solid #1a1a1a;margin-bottom:24px}
.problem-id{font-size:24px;font-weight:800;color:#2563eb}
.header-right{display:flex;gap:12px;align-items:center;font-size:13px;color:#6b7280}
.difficulty-badge{padding:2px 12px;border-radius:12px;font-weight:600;font-size:12px}
.diff-1{background:#dcfce7;color:#166534}
.diff-2{background:#fef3c7;color:#92400e}
.diff-3{background:#fed7aa;color:#9a3412}
.diff-4{background:#fecaca;color:#991b1b}

.problem-image-section{margin-bottom:28px;text-align:center}
.problem-image{max-width:100%;max-height:500px;border:1px solid #e5e7eb;border-radius:4px}

.section{margin-bottom:36px}
.section-title{font-size:17px;font-weight:700;padding:8px 0 8px 14px;margin-bottom:16px;background:#f9fafb}
.st-1{border-left:4px solid #2563eb}
.st-2{border-left:4px solid #7c3aed}
.st-3{border-left:4px solid #0891b2}
.st-4{border-left:4px solid #dc2626}
.st-5{border-left:4px solid #ea580c}
.st-6{border-left:4px solid #16a34a}
.st-0{border-left:4px solid #d97706}

.thought-flow{position:relative}
.thought-step{position:relative;padding:10px 0 10px 28px;margin-bottom:4px;
  border-left:2px solid #e5e7eb;margin-left:6px}
.thought-step:last-child{border-left-color:transparent}
.thought-step::before{content:'';position:absolute;left:-7px;top:15px;
  width:12px;height:12px;border-radius:50%;border:2px solid #d97706;background:#fff}
.thought-step.ts-insight::before{background:#d97706}
.thought-label{font-size:13px;font-weight:700;margin-bottom:4px}
.thought-text{font-size:14px;color:#374151;line-height:1.8}

.method{margin-bottom:20px;padding:14px 16px;border:1px solid #e5e7eb;border-radius:6px}
.method-title{font-size:15px;font-weight:700;color:#2563eb;margin-bottom:10px}
.method-tag{display:inline-block;font-size:11px;padding:1px 8px;border-radius:4px;
  background:#eff6ff;color:#2563eb;margin-left:8px;font-weight:400}
.step{margin-bottom:10px;padding-left:4px;font-size:14px;color:#374151}

.option-analysis{margin:8px 0 0}
.option-item{display:flex;gap:10px;align-items:flex-start;padding:10px 14px;
  margin-bottom:7px;border-radius:6px;font-size:14px;line-height:1.7}
.option-item.correct{background:#f0fdf4;border:1px solid #bbf7d0}
.option-item.wrong{background:#fef2f2;border:1px solid #fee2e2}
.option-label{font-weight:800;min-width:22px;font-size:15px;padding-top:1px}
.option-item.correct .option-label{color:#16a34a}
.option-item.wrong .option-label{color:#dc2626}
.option-content{flex:1}
.option-text{margin-bottom:3px;font-weight:500}
.option-reason{font-size:12px;color:#6b7280;line-height:1.6}

.knowledge-item{padding:8px 12px;margin-bottom:6px;border-left:3px solid #e5e7eb;
  margin-left:8px;font-size:14px;background:#fafafa;border-radius:0 4px 4px 0}
.knowledge-item .module-tag{display:inline-block;font-size:11px;padding:1px 8px;
  border-radius:10px;background:#f3f4f6;color:#6b7280;margin-right:6px}
.knowledge-item .arrow{color:#9ca3af;margin:0 4px}

.thinking-item{display:flex;gap:12px;padding:10px 0;border-bottom:1px solid #f3f4f6;font-size:14px}
.thinking-step{flex:1;color:#374151}
.thinking-method{flex-shrink:0;padding:2px 10px;border-radius:10px;font-size:12px;
  font-weight:600;background:#f0f4ff;color:#2563eb;white-space:nowrap;align-self:flex-start}
.thinking-why{font-size:12px;color:#6b7280;margin-top:2px}

.pitfall{margin-bottom:14px;padding:12px 16px;border-radius:6px;border:1px solid #fecaca;background:#fef2f2}
.pitfall-wrong{color:#dc2626;font-size:14px;margin-bottom:4px}
.pitfall-wrong::before{content:'❌ '}
.pitfall-right{color:#16a34a;font-size:14px;margin-bottom:4px}
.pitfall-right::before{content:'✅ '}
.pitfall-why{font-size:13px;color:#6b7280;padding-left:20px}

.variation{margin-bottom:12px;padding:12px 16px;border:1px solid #e5e7eb;border-radius:6px;background:#f9fafb}
.variation-title{font-weight:600;font-size:14px;margin-bottom:4px;color:#ea580c}
.variation-desc{font-size:13px;color:#374151}

.difficulty-box{padding:16px;border:1px solid #e5e7eb;border-radius:6px;background:#f9fafb}
.diff-row{display:flex;gap:24px;margin-bottom:6px;font-size:14px}
.diff-label{color:#6b7280;min-width:80px}
.diff-value{font-weight:600}

.print-btn{position:fixed;top:16px;right:16px;width:36px;height:36px;border-radius:50%;
  border:1px solid #e5e7eb;background:#fff;font-size:16px;cursor:pointer;
  display:flex;align-items:center;justify-content:center;box-shadow:0 1px 3px rgba(0,0,0,.1)}
.print-btn:hover{background:#f3f4f6}

@media print{
  body{padding:16px;font-size:11pt}
  .section{page-break-inside:avoid}
  .print-btn,.nav{display:none}
  .problem-image{max-height:300px}
  .header{border-bottom-width:1px}
}
```

---

## INDEX_TEMPLATE

汇总索引页面。深色主题，视觉丰富，用于备考诊断，通常不打印。

当 `~/Desktop/jiaxiai/xingce/index.html` 不存在时生成。`var problems = [];` 初始为空，每次分析题目后追加。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>行测备考 · 学习诊断</title>
<style>
:root{--bg:#0f172a;--card:#1e293b;--text:#e2e8f0;--text2:#94a3b8;
  --blue:#38bdf8;--gold:#fbbf24;--pink:#f472b6;--green:#34d399;
  --purple:#a78bfa;--orange:#fb923c;--fuchsia:#e879f9;--cyan:#22d3ee;
  --red:#f87171;--lime:#4ade80;--lpurple:#c084fc;--yellow:#fcd34d}
*{margin:0;padding:0;box-sizing:border-box}
html{scroll-behavior:smooth}
body{font-family:-apple-system,'PingFang SC','Microsoft YaHei',sans-serif;
  background:var(--bg);color:var(--text);line-height:1.7}

.header{padding:48px 40px 32px;text-align:center;
  background:linear-gradient(135deg,#0f172a,#1e3a5f,#0f172a);
  border-bottom:1px solid rgba(56,189,248,.1)}
.header h1{font-size:28px;font-weight:800}
.header h1 span{color:var(--blue)}
.header .sub{color:var(--text2);font-size:13px;margin-top:6px}

.tab-bar{display:flex;justify-content:center;gap:4px;margin:32px auto 0;
  background:var(--card);border-radius:12px;padding:4px;max-width:460px}
.tab{flex:1;padding:8px 16px;text-align:center;border-radius:10px;
  font-size:13px;font-weight:600;cursor:pointer;color:var(--text2);transition:all .2s}
.tab.on{background:var(--blue);color:#0f172a}
.tab:hover:not(.on){color:var(--text)}

.container{max-width:1000px;margin:0 auto;padding:32px 24px}
.view{display:none}.view.active{display:block}

.stats-bar{display:flex;gap:24px;justify-content:center;margin:24px 0 32px;flex-wrap:wrap}
.sb{text-align:center}
.sb .n{font-size:28px;font-weight:800;color:var(--blue)}
.sb .l{font-size:11px;color:var(--text2);margin-top:2px}

.problem-card{background:var(--card);border-radius:14px;margin-bottom:16px;overflow:hidden;
  border:1px solid rgba(255,255,255,.05);transition:border-color .2s}
.problem-card:hover{border-color:rgba(56,189,248,.2)}
.problem-head{padding:16px 20px;display:flex;align-items:center;justify-content:space-between;
  cursor:pointer;user-select:none}
.problem-head .left{display:flex;align-items:center;gap:14px}
.p-num{font-size:20px;font-weight:800;color:var(--blue);min-width:40px}
.p-title{font-size:15px;font-weight:700}
.p-date{font-size:12px;color:var(--text2);margin-top:2px}
.problem-head .right{display:flex;align-items:center;gap:8px}
.p-diff{font-size:11px;font-weight:600;padding:2px 10px;border-radius:8px}
.pd-1{background:rgba(34,197,94,.15);color:var(--lime)}
.pd-2{background:rgba(234,179,8,.15);color:var(--gold)}
.pd-3{background:rgba(249,115,22,.15);color:var(--orange)}
.pd-4{background:rgba(239,68,68,.15);color:var(--red)}
.p-modules{display:flex;gap:4px;flex-wrap:wrap}
.p-module-tag{font-size:10px;padding:1px 7px;border-radius:6px;
  background:rgba(255,255,255,.06);color:var(--text2)}
.open-btn{font-size:12px;color:var(--blue);text-decoration:none;padding:4px 12px;
  border:1px solid rgba(56,189,248,.3);border-radius:8px;transition:all .2s;white-space:nowrap}
.open-btn:hover{background:rgba(56,189,248,.1)}
.arrow{color:var(--text2);font-size:14px;transition:transform .2s;margin-left:8px}
.problem-card.expanded .arrow{transform:rotate(90deg)}
.p-detail{max-height:0;overflow:hidden;transition:max-height .3s ease;
  border-top:0 solid rgba(255,255,255,.04)}
.problem-card.expanded .p-detail{max-height:600px;border-top-width:1px}
.p-detail-inner{padding:12px 20px;font-size:13px;color:var(--text2)}
.p-detail-row{margin-bottom:6px}
.p-detail-label{color:var(--text2);margin-right:8px}
.p-detail-tags{display:flex;gap:4px;flex-wrap:wrap;margin-top:4px}
.p-thinking-tag{font-size:11px;padding:1px 8px;border-radius:8px;
  background:rgba(8,145,178,.12);color:var(--cyan)}

.module-group{margin-bottom:24px}
.module-group-head{display:flex;align-items:center;gap:10px;padding:12px 0;
  border-bottom:1px solid rgba(255,255,255,.06);margin-bottom:8px}
.module-group-head .m-bar{width:4px;height:22px;border-radius:2px;flex-shrink:0}
.module-group-head h3{font-size:15px;font-weight:700;flex:1}
.m-count{font-size:11px;font-weight:700;padding:2px 10px;border-radius:8px;
  background:rgba(255,255,255,.06);color:var(--text2)}
.m-problem{display:flex;align-items:center;gap:10px;padding:6px 0 6px 14px;font-size:13px}
.m-problem .m-link{color:var(--blue);text-decoration:none;font-size:12px}
.m-problem .m-link:hover{text-decoration:underline}
.m-problem .m-context{color:var(--text2);font-size:12px;flex:1}
.m-problem .m-diff{font-size:10px;padding:1px 6px;border-radius:4px}

.chart-row{display:flex;gap:24px;margin-bottom:24px;flex-wrap:wrap}
.chart-card{background:var(--card);border-radius:14px;padding:24px;flex:1;min-width:300px;
  border:1px solid rgba(255,255,255,.05)}
.chart-card.full{flex-basis:100%;min-width:100%}
.chart-title{font-size:14px;font-weight:700;margin-bottom:16px;display:flex;align-items:center;gap:8px}
.chart-subtitle{font-size:11px;color:var(--text2);margin-top:-10px;margin-bottom:16px}

.radar-wrap{display:flex;justify-content:center}
.radar-wrap svg{width:100%;max-width:420px;height:auto}

.cat-stat-row{display:flex;align-items:center;gap:10px;margin-bottom:10px}
.cat-stat-row .cs-dot{width:10px;height:10px;border-radius:50%;flex-shrink:0}
.cat-stat-row .cs-name{font-size:13px;width:72px}
.cat-stat-row .cs-bar-bg{flex:1;height:8px;border-radius:4px;background:rgba(255,255,255,.06);overflow:hidden}
.cat-stat-row .cs-bar{height:100%;border-radius:4px;transition:width .5s ease}
.cat-stat-row .cs-count{font-size:12px;color:var(--text2);min-width:28px;text-align:right}

.insight-row{display:flex;gap:16px;flex-wrap:wrap;margin-bottom:24px}
.ins-card{background:var(--card);border-radius:12px;padding:18px 20px;flex:1;min-width:140px;
  border:1px solid rgba(255,255,255,.05);text-align:center}
.ins-card .ins-val{font-size:24px;font-weight:800}
.ins-card .ins-label{font-size:11px;color:var(--text2);margin-top:4px}

.weakness-list{list-style:none;padding:0}
.weakness-item{display:flex;align-items:center;gap:10px;padding:8px 0;
  border-bottom:1px solid rgba(255,255,255,.04);font-size:13px}
.weakness-dot{width:8px;height:8px;border-radius:50%}
.weakness-name{flex:1}
.weakness-status{font-size:11px;padding:2px 8px;border-radius:6px}
.ws-none{background:rgba(239,68,68,.12);color:var(--red)}
.ws-weak{background:rgba(249,115,22,.12);color:var(--orange)}
.ws-ok{background:rgba(234,179,8,.12);color:var(--gold)}
.ws-good{background:rgba(34,197,94,.12);color:var(--lime)}

.thinking-cloud{display:flex;flex-wrap:wrap;gap:8px;justify-content:center;
  padding:16px;margin-bottom:16px}
.t-tag{padding:4px 14px;border-radius:16px;cursor:default;transition:all .2s;
  border:1px solid transparent;background:rgba(255,255,255,.04)}
.t-tag:hover{transform:scale(1.05);filter:brightness(1.2)}

.diff-bar-wrap{display:flex;align-items:flex-end;gap:16px;justify-content:center;
  height:120px;padding:0 20px}
.diff-col{display:flex;flex-direction:column;align-items:center;gap:4px}
.diff-bar{width:48px;border-radius:6px 6px 0 0;transition:height .5s ease;min-height:4px}
.diff-bar-label{font-size:11px;color:var(--text2)}
.diff-bar-count{font-size:14px;font-weight:700}

.heatmap-wrap{overflow-x:auto;padding-bottom:4px}
.heatmap-wrap svg{height:auto}
.hm-legend{display:flex;align-items:center;gap:6px;margin-top:12px;font-size:10px;
  color:var(--text2);justify-content:flex-end}
.hm-legend-cell{width:12px;height:12px;border-radius:2px}
.hm-tip{font-size:11px;color:var(--text2);margin-top:8px;text-align:center;font-style:italic}

.empty-state{text-align:center;color:var(--text2);padding:60px 20px;font-size:14px}
.footer{text-align:center;padding:32px;color:var(--text2);font-size:11px;
  border-top:1px solid rgba(255,255,255,.04);margin-top:40px}

.wechat-section{text-align:center;padding:28px 24px;border-top:1px solid rgba(255,255,255,.06);margin-top:8px}
.wechat-card{display:inline-flex;align-items:center;gap:20px;background:var(--card);border-radius:14px;
  padding:18px 28px;border:1px solid rgba(255,255,255,.08)}
.wechat-qr{width:110px;border-radius:8px;flex-shrink:0}
.wechat-label{font-size:11px;font-weight:700;color:var(--blue);margin-bottom:6px;letter-spacing:.5px;text-transform:uppercase}
.wechat-desc{font-size:13px;color:var(--text);line-height:1.9;font-weight:500}

@media(max-width:600px){
  .header{padding:36px 16px 24px}
  .header h1{font-size:22px}
  .container{padding:20px 12px}
  .problem-head{padding:14px 16px;flex-direction:column;align-items:flex-start;gap:8px}
  .problem-head .right{align-self:flex-end}
  .chart-row{flex-direction:column}
  .chart-card{min-width:auto}
  .insight-row{flex-direction:column}
  .ins-card{min-width:auto}
}
</style>
</head>
<body>

<div class="header">
  <svg viewBox="0 0 290 68" width="290" height="68" xmlns="http://www.w3.org/2000/svg" style="display:block;margin:0 auto">
    <defs>
      <linearGradient id="logo-pg" x1="0%" y1="0%" x2="100%" y2="100%">
        <stop offset="0%" stop-color="#1e3a5f"/>
        <stop offset="100%" stop-color="#0f172a"/>
      </linearGradient>
      <radialGradient id="logo-ig" cx="25%" cy="50%" r="40%" gradientUnits="objectBoundingBox">
        <stop offset="0%" stop-color="white" stop-opacity="0.18"/>
        <stop offset="100%" stop-color="white" stop-opacity="0"/>
      </radialGradient>
    </defs>
    <polygon points="50,4 56,12.3 155,11.1 155,2" fill="#c084fc"/>
    <polygon points="56,12.3 62,20.6 155,20.3 155,11.1" fill="#38bdf8"/>
    <polygon points="62,20.6 68,28.9 155,29.4 155,20.3" fill="#22d3ee"/>
    <polygon points="68,28.9 74,37.1 155,38.6 155,29.4" fill="#34d399"/>
    <polygon points="74,37.1 80,45.4 155,47.7 155,38.6" fill="#fbbf24"/>
    <polygon points="80,45.4 86,53.7 155,56.9 155,47.7" fill="#fb923c"/>
    <polygon points="86,53.7 92,62 155,66 155,56.9" fill="#f87171"/>
    <polygon points="50,4 8,62 92,62" fill="url(#logo-pg)"/>
    <polygon points="50,4 8,62 92,62" fill="url(#logo-ig)"/>
    <polygon points="50,4 8,62 92,62" fill="none" stroke="rgba(100,180,255,0.35)" stroke-width="1.5" stroke-linejoin="round"/>
    <line x1="0" y1="33" x2="29" y2="33" stroke="rgba(255,255,255,0.8)" stroke-width="2.5" stroke-linecap="round"/>
    <text x="165" y="50" font-family="'PingFang SC','Noto Sans SC','Microsoft YaHei',sans-serif" font-size="40" font-weight="900" fill="white" letter-spacing="6">嘉析</text>
  </svg>
  <div style="font-size:15px;color:rgba(226,232,240,0.65);margin-top:10px;font-weight:500;letter-spacing:2px">行测备考 · 学习诊断</div>
  <div class="sub">题目分析汇总 &middot; 模块覆盖 &middot; 考试策略诊断</div>
</div>

<div class="tab-bar">
  <div class="tab on" onclick="switchView('list',this)">&#128197; 题目列表</div>
  <div class="tab" onclick="switchView('module',this)">&#128218; 知识模块</div>
  <div class="tab" onclick="switchView('diag',this)">&#128202; 备考诊断</div>
</div>

<div class="container">

<div class="stats-bar">
  <div class="sb"><div class="n" id="st-total">0</div><div class="l">题目总数</div></div>
  <div class="sb"><div class="n" id="st-modules">0</div><div class="l">涉及模块</div></div>
  <div class="sb"><div class="n" id="st-thinking">0</div><div class="l">思维方式</div></div>
  <div class="sb"><div class="n" id="st-weak">0</div><div class="l">待加强模块</div></div>
</div>

<div class="view active" id="view-list"></div>

<div class="view" id="view-module">
  <div id="module-groups"></div>
</div>

<div class="view" id="view-diag">
  <div class="insight-row" id="insight-row"></div>
  <div class="chart-row">
    <div class="chart-card">
      <div class="chart-title">&#128171; 模块覆盖雷达</div>
      <div class="chart-subtitle">各模块练习量 — 形状越饱满覆盖越均衡</div>
      <div class="radar-wrap" id="radar-chart"></div>
    </div>
    <div class="chart-card">
      <div class="chart-title">&#128202; 难度分布</div>
      <div class="chart-subtitle">各难度等级题目数量</div>
      <div class="diff-bar-wrap" id="diff-chart"></div>
      <div style="margin-top:24px">
        <div class="chart-title" style="margin-bottom:12px">&#128161; 薄弱模块诊断</div>
        <ul class="weakness-list" id="weakness-list"></ul>
      </div>
    </div>
  </div>
  <div class="chart-row">
    <div class="chart-card full">
      <div class="chart-title">&#129504; 思维方式频率</div>
      <div class="chart-subtitle">出现次数越多说明越常用 — 关注低频思维，它们可能是你的盲区</div>
      <div class="thinking-cloud" id="thinking-cloud"></div>
      <div id="thinking-bars"></div>
    </div>
  </div>
  <div class="chart-row">
    <div class="chart-card full">
      <div class="chart-title">&#128293; 备考热力图</div>
      <div class="chart-subtitle">每个格子代表一天 — 颜色越深当天练习越多</div>
      <div class="heatmap-wrap" id="heatmap-chart"></div>
      <div class="hm-legend">
        <span>少</span>
        <div class="hm-legend-cell" style="background:rgba(255,255,255,.04)"></div>
        <div class="hm-legend-cell" style="background:rgba(56,189,248,.2)"></div>
        <div class="hm-legend-cell" style="background:rgba(56,189,248,.45)"></div>
        <div class="hm-legend-cell" style="background:rgba(56,189,248,.7)"></div>
        <div class="hm-legend-cell" style="background:rgba(56,189,248,.95)"></div>
        <span>多</span>
      </div>
      <div class="hm-tip">坚持每天刷题，让热力图亮起来</div>
    </div>
  </div>
</div>

</div>

<div class="wechat-section">
  <div class="wechat-card">
    <img class="wechat-qr" src="data:image/png;base64,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" alt="微信二维码">
    <div>
      <div class="wechat-label">扫码添加老师微信</div>
      <div class="wechat-desc">添加老师微信<br>获取学习情况的深度复盘总结</div>
    </div>
  </div>
</div>

<div class="footer">行测备考 · 学习诊断 &middot; 由 /gwyxingce 自动维护<div style="margin-top:8px"><a href="http://www.he321.com/xingce" style="color:inherit;text-decoration:none">嘉析AI</a>官网：<a href="http://www.he321.com/xingce" style="color:inherit;text-decoration:underline">www.he321.com/xingce</a></div></div>

<script>
var problems = [];

var moduleNames=['言语理解','数量关系','判断推理','资料分析','常识判断'];
var moduleColors={'言语理解':'#38bdf8','数量关系':'#fbbf24','判断推理':'#f472b6',
  '资料分析':'#34d399','常识判断':'#a78bfa'};
var diffLabels={1:'容易',2:'中等',3:'较难',4:'高难'};
var diffColors={1:'#22c55e',2:'#eab308',3:'#f97316',4:'#ef4444'};

var moduleCounts={},thinkingCounts={},diffCounts={1:0,2:0,3:0,4:0},allThinking=[];

function computeData(){
  moduleCounts={};thinkingCounts={};diffCounts={1:0,2:0,3:0,4:0};allThinking=[];
  moduleNames.forEach(function(m){moduleCounts[m]=0});
  problems.forEach(function(p){
    p.modules.forEach(function(m){moduleCounts[m]=(moduleCounts[m]||0)+1});
    p.thinking.forEach(function(t){
      thinkingCounts[t]=(thinkingCounts[t]||0)+1;
      if(allThinking.indexOf(t)===-1) allThinking.push(t);
    });
    diffCounts[p.difficulty]=(diffCounts[p.difficulty]||0)+1;
  });
}

function render(){
  computeData();
  var coveredModules=0,weakModules=0;
  moduleNames.forEach(function(m){
    if(moduleCounts[m]>0) coveredModules++;
    if(moduleCounts[m]<5) weakModules++;
  });
  document.getElementById('st-total').textContent=problems.length;
  document.getElementById('st-modules').textContent=coveredModules+'/'+moduleNames.length;
  document.getElementById('st-thinking').textContent=allThinking.length;
  document.getElementById('st-weak').textContent=weakModules;
  renderListView();
  renderModuleView();
  renderDiagView();
}

function renderListView(){
  var html='';
  problems.slice().reverse().forEach(function(p){
    html+='<div class="problem-card" onclick="toggleCard(this)">';
    html+='<div class="problem-head"><div class="left">';
    html+='<div class="p-num">#'+String(p.id).padStart(3,'0')+'</div>';
    html+='<div><div class="p-title">'+p.title+'</div>';
    html+='<div class="p-date">'+p.date+' &middot; '+p.source+'</div></div>';
    html+='</div><div class="right">';
    html+='<span class="p-diff pd-'+p.difficulty+'">'+p.diffLabel+'</span>';
    html+='<div class="p-modules">';
    p.modules.forEach(function(m){
      html+='<span class="p-module-tag" style="color:'+(moduleColors[m]||'#94a3b8')+'">'+m+'</span>';
    });
    html+='</div>';
    html+='<a class="open-btn" href="'+p.file+'" target="_blank" onclick="event.stopPropagation()">查看分析 &rarr;</a>';
    html+='<span class="arrow">&#9654;</span>';
    html+='</div></div>';
    html+='<div class="p-detail"><div class="p-detail-inner">';
    html+='<div class="p-detail-row" style="margin-bottom:10px"><span class="p-detail-label">跳转：</span>';
    ['思路','解法','知识点','思维','易错','变式','难度'].forEach(function(s){
      html+='<a href="'+p.file+'#'+s+'" target="_blank" style="font-size:11px;color:var(--blue);text-decoration:none;padding:2px 8px;border:1px solid rgba(56,189,248,.2);border-radius:6px;margin-right:4px;display:inline-block;margin-bottom:4px" onclick="event.stopPropagation()">'+s+'</a>';
    });
    html+='</div>';
    html+='<div class="p-detail-row"><span class="p-detail-label">知识点：</span>';
    (p.moduleDetails||[]).forEach(function(d){html+='<div style="margin-left:16px;font-size:12px">'+d+'</div>';});
    html+='</div>';
    html+='<div class="p-detail-row"><span class="p-detail-label">思维方式：</span>';
    html+='<div class="p-detail-tags">';
    p.thinking.forEach(function(t){html+='<span class="p-thinking-tag">'+t+'</span>';});
    html+='</div></div>';
    html+='</div></div></div>';
  });
  document.getElementById('view-list').innerHTML=html||'<div class="empty-state">暂无题目，发送行测题目开始分析</div>';
}

function renderModuleView(){
  var grouped={};
  moduleNames.forEach(function(m){grouped[m]=[]});
  problems.forEach(function(p){
    p.modules.forEach(function(m){
      if(!grouped[m]) grouped[m]=[];
      grouped[m].push(p);
    });
  });
  var sorted=moduleNames.slice().sort(function(a,b){return(grouped[b]||[]).length-(grouped[a]||[]).length});
  var html='';
  sorted.forEach(function(m){
    var ps=grouped[m]||[];
    var col=moduleColors[m]||'#94a3b8';
    html+='<div class="module-group">';
    html+='<div class="module-group-head">';
    html+='<div class="m-bar" style="background:'+col+'"></div>';
    html+='<h3 style="color:'+col+'">'+m+'</h3>';
    html+='<span class="m-count">'+ps.length+' 题</span>';
    html+='</div>';
    if(ps.length===0){
      html+='<div style="padding:8px 14px;font-size:12px;color:#64748b">暂无题目</div>';
    }
    ps.forEach(function(p){
      html+='<div class="m-problem">';
      html+='<a class="m-link" href="'+p.file+'#知识点" target="_blank">#'+String(p.id).padStart(3,'0')+' '+p.date+' &rarr;</a>';
      html+='<span class="m-context">'+p.title+'</span>';
      html+='<span class="m-diff p-diff pd-'+p.difficulty+'" style="font-size:10px;padding:1px 6px">'+p.diffLabel+'</span>';
      html+='</div>';
    });
    html+='</div>';
  });
  document.getElementById('module-groups').innerHTML=html;
}

function renderDiagView(){
  renderInsights();
  renderRadar();
  renderDiffChart();
  renderWeakness();
  renderThinkingCloud();
  renderHeatmap();
}

function renderInsights(){
  var topModule='',topVal=0;
  moduleNames.forEach(function(m){if(moduleCounts[m]>topVal){topVal=moduleCounts[m];topModule=m;}});
  var avgDiff=0;
  if(problems.length>0){
    var sum=0;problems.forEach(function(p){sum+=p.difficulty});
    avgDiff=(sum/problems.length).toFixed(1);
  }
  var dates=problems.map(function(p){return p.date}).sort();
  var span=0;
  if(dates.length>=2){
    var d0=new Date(dates[0]),d1=new Date(dates[dates.length-1]);
    span=Math.round((d1-d0)/86400000);
  }
  var html='';
  html+='<div class="ins-card"><div class="ins-val" style="color:'+(moduleColors[topModule]||'var(--blue)')+'">'+
    (topModule||'-')+'</div><div class="ins-label">练习最多模块</div></div>';
  html+='<div class="ins-card"><div class="ins-val" style="color:var(--gold)">'+avgDiff+'</div><div class="ins-label">平均难度 (1-4)</div></div>';
  html+='<div class="ins-card"><div class="ins-val" style="color:var(--green)">'+span+'</div><div class="ins-label">备考跨度 (天)</div></div>';
  html+='<div class="ins-card"><div class="ins-val" style="color:var(--blue)">'+allThinking.length+'</div><div class="ins-label">思维方式种类</div></div>';
  document.getElementById('insight-row').innerHTML=html;
}

function renderRadar(){
  var names=moduleNames,n=names.length;
  var cx=190,cy=190,r=130;
  var step=2*Math.PI/n,start=-Math.PI/2;
  var maxVal=1;
  names.forEach(function(m){if(moduleCounts[m]>maxVal) maxVal=moduleCounts[m]});

  var svg='<svg viewBox="0 0 380 400" xmlns="http://www.w3.org/2000/svg">';
  [0.25,0.5,0.75,1].forEach(function(scale){
    var pts=[];
    for(var i=0;i<n;i++){
      var a=start+i*step;
      pts.push((cx+r*scale*Math.cos(a)).toFixed(1)+','+(cy+r*scale*Math.sin(a)).toFixed(1));
    }
    svg+='<polygon points="'+pts.join(' ')+'" fill="none" stroke="rgba(255,255,255,.07)"/>';
  });
  for(var i=0;i<n;i++){
    var a=start+i*step;
    svg+='<line x1="'+cx+'" y1="'+cy+'" x2="'+(cx+r*Math.cos(a)).toFixed(1)+'" y2="'+(cy+r*Math.sin(a)).toFixed(1)+'" stroke="rgba(255,255,255,.06)"/>';
  }
  var dataPts=[];
  for(var i=0;i<n;i++){
    var a=start+i*step;
    var val=Math.max(moduleCounts[names[i]]/maxVal,0.06);
    dataPts.push((cx+r*val*Math.cos(a)).toFixed(1)+','+(cy+r*val*Math.sin(a)).toFixed(1));
  }
  svg+='<polygon points="'+dataPts.join(' ')+'" fill="rgba(56,189,248,.12)" stroke="rgba(56,189,248,.6)" stroke-width="2"/>';
  for(var i=0;i<n;i++){
    var a=start+i*step;
    var val=Math.max(moduleCounts[names[i]]/maxVal,0.06);
    var dx=cx+r*val*Math.cos(a),dy=cy+r*val*Math.sin(a);
    var col=moduleColors[names[i]]||'#94a3b8';
    svg+='<circle cx="'+dx.toFixed(1)+'" cy="'+dy.toFixed(1)+'" r="4" fill="'+col+'" stroke="#0f172a" stroke-width="2"/>';
    var lx=cx+(r+22)*Math.cos(a),ly=cy+(r+22)*Math.sin(a);
    var anchor='middle';
    if(Math.cos(a)>0.3) anchor='start';
    else if(Math.cos(a)<-0.3) anchor='end';
    svg+='<text x="'+lx.toFixed(1)+'" y="'+ly.toFixed(1)+'" text-anchor="'+anchor+'" fill="'+col+'" font-size="11" font-weight="700">'+names[i]+'</text>';
    svg+='<text x="'+lx.toFixed(1)+'" y="'+(ly+14).toFixed(1)+'" text-anchor="'+anchor+'" fill="#94a3b8" font-size="10">'+moduleCounts[names[i]]+' 题</text>';
  }
  svg+='</svg>';
  document.getElementById('radar-chart').innerHTML=svg;
}

function renderDiffChart(){
  var maxVal=1;
  for(var d=1;d<=4;d++){if(diffCounts[d]>maxVal) maxVal=diffCounts[d];}
  var html='';
  for(var d=1;d<=4;d++){
    var h=Math.max(diffCounts[d]/maxVal*100,4);
    html+='<div class="diff-col">';
    html+='<div class="diff-bar-count" style="color:'+diffColors[d]+'">'+diffCounts[d]+'</div>';
    html+='<div class="diff-bar" style="height:'+h+'px;background:'+diffColors[d]+'"></div>';
    html+='<div class="diff-bar-label">'+diffLabels[d]+'</div>';
    html+='</div>';
  }
  document.getElementById('diff-chart').innerHTML=html;
}

function renderWeakness(){
  var sorted=moduleNames.slice().sort(function(a,b){return moduleCounts[a]-moduleCounts[b]});
  var html='';
  sorted.forEach(function(m){
    var c=moduleCounts[m]||0;
    var status,cls;
    if(c===0){status='未涉及';cls='ws-none';}
    else if(c<=5){status='需加强';cls='ws-weak';}
    else if(c<=15){status='初步掌握';cls='ws-ok';}
    else{status='较熟练';cls='ws-good';}
    var col=moduleColors[m]||'#94a3b8';
    html+='<li class="weakness-item">';
    html+='<div class="weakness-dot" style="background:'+col+'"></div>';
    html+='<span class="weakness-name">'+m+'</span>';
    html+='<span style="font-size:12px;color:var(--text2)">'+c+' 题</span>';
    html+='<span class="weakness-status '+cls+'">'+status+'</span>';
    html+='</li>';
  });
  document.getElementById('weakness-list').innerHTML=html;
}

function renderThinkingCloud(){
  var sorted=allThinking.slice().sort(function(a,b){return thinkingCounts[b]-thinkingCounts[a]});
  var maxFreq=1;
  sorted.forEach(function(t){if(thinkingCounts[t]>maxFreq) maxFreq=thinkingCounts[t]});

  var cloudHtml='';
  sorted.forEach(function(t){
    var freq=thinkingCounts[t];
    var size=Math.round(13+12*(freq/maxFreq));
    var opacity=0.5+0.5*(freq/maxFreq);
    cloudHtml+='<span class="t-tag" style="font-size:'+size+'px;color:var(--cyan);opacity:'+opacity.toFixed(2)+
      ';border-color:rgba(34,211,238,'+((freq/maxFreq)*0.3).toFixed(2)+')">'+t+' <sup style="font-size:10px">'+freq+'</sup></span>';
  });
  document.getElementById('thinking-cloud').innerHTML=cloudHtml||'<div class="empty-state" style="padding:20px">暂无数据</div>';

  var barHtml='';
  sorted.slice(0,10).forEach(function(t){
    var pct=Math.round(thinkingCounts[t]/maxFreq*100);
    barHtml+='<div class="cat-stat-row">';
    barHtml+='<div class="cs-dot" style="background:var(--cyan)"></div>';
    barHtml+='<div class="cs-name" style="width:80px">'+t+'</div>';
    barHtml+='<div class="cs-bar-bg"><div class="cs-bar" style="width:'+pct+'%;background:var(--cyan)"></div></div>';
    barHtml+='<div class="cs-count">'+thinkingCounts[t]+'</div>';
    barHtml+='</div>';
  });
  document.getElementById('thinking-bars').innerHTML=barHtml;
}

function renderHeatmap(){
  var today=new Date();today.setHours(0,0,0,0);
  var start52=new Date(today);start52.setDate(start52.getDate()-52*7);
  var earliest=start52;
  problems.forEach(function(p){
    var d=new Date(p.date+'T00:00:00');
    if(d<earliest) earliest=new Date(d);
  });
  var startDate=new Date(earliest);
  startDate.setDate(startDate.getDate()-startDate.getDay());

  var dateMap={},topicMap={};
  problems.forEach(function(p){
    dateMap[p.date]=(dateMap[p.date]||0)+1;
    topicMap[p.date]=(topicMap[p.date]||'')+p.title+' ';
  });

  var cells=[];
  var d=new Date(startDate);
  while(d<=today){
    var key=d.getFullYear()+'-'+pad2(d.getMonth()+1)+'-'+pad2(d.getDate());
    cells.push({date:key,count:dateMap[key]||0,topic:topicMap[key]||'',dow:d.getDay()});
    d.setDate(d.getDate()+1);
  }

  var cellSize=13,gap=3,cellStep=cellSize+gap;
  var weeks=Math.ceil(cells.length/7);
  var labelW=28,labelH=18;
  var W=labelW+weeks*cellStep+10;
  var H=labelH+7*cellStep+4;
  var svg='<svg viewBox="0 0 '+W+' '+H+'" width="'+W+'" xmlns="http://www.w3.org/2000/svg" style="min-width:'+W+'px">';

  var dayNames=['','一','','三','','五',''];
  for(var i=0;i<7;i++){
    if(dayNames[i]) svg+='<text x="'+(labelW-4)+'" y="'+(labelH+i*cellStep+cellSize-2)+'" text-anchor="end" fill="#64748b" font-size="9">'+dayNames[i]+'</text>';
  }

  var lastMonth=-1,cellIdx=0;
  var monthNames=['1月','2月','3月','4月','5月','6月','7月','8月','9月','10月','11月','12月'];
  for(var w=0;w<weeks;w++){
    for(var dow=0;dow<7;dow++){
      if(cellIdx>=cells.length) break;
      var c=cells[cellIdx];
      var cDate=new Date(c.date+'T00:00:00');
      var mon=cDate.getMonth();
      if(mon!==lastMonth&&dow===0){
        svg+='<text x="'+(labelW+w*cellStep)+'" y="'+(labelH-5)+'" fill="#64748b" font-size="9">'+monthNames[mon]+'</text>';
        lastMonth=mon;
      }
      cellIdx++;
    }
  }
  cellIdx=0;
  for(var w=0;w<weeks;w++){
    for(var dow=0;dow<7;dow++){
      if(cellIdx>=cells.length) break;
      var c=cells[cellIdx];
      var x=labelW+w*cellStep,y=labelH+dow*cellStep;
      var fill=hmColor(c.count);
      var title=c.date+(c.count>0?' ('+c.count+'题) '+c.topic.trim():'');
      svg+='<rect x="'+x+'" y="'+y+'" width="'+cellSize+'" height="'+cellSize+'" rx="2" fill="'+fill+'"><title>'+title+'</title></rect>';
      cellIdx++;
    }
  }
  svg+='</svg>';
  document.getElementById('heatmap-chart').innerHTML=svg;
}

function hmColor(c){
  if(c===0) return 'rgba(255,255,255,.04)';
  if(c===1) return 'rgba(56,189,248,.25)';
  if(c<=3) return 'rgba(56,189,248,.5)';
  if(c<=5) return 'rgba(56,189,248,.75)';
  return 'rgba(56,189,248,.95)';
}
function pad2(n){return n<10?'0'+n:''+n}
function switchView(v,el){
  document.querySelectorAll('.tab').forEach(function(t){t.classList.remove('on')});
  el.classList.add('on');
  document.querySelectorAll('.view').forEach(function(vw){vw.classList.remove('active')});
  document.getElementById('view-'+v).classList.add('active');
}
function toggleCard(card){card.classList.toggle('expanded')}

render();
var first=document.querySelector('.problem-card');
if(first) first.classList.add('expanded');
</script>
</body>
</html>
```
<!-- 嘉析AI · 公考行测 · 7 维备考分析器 · v1.0.0 -->
