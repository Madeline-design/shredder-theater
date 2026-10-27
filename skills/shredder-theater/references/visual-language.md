# Visual Language — 碎纸机页面的视觉配方

写页面时从这里挑，不要每次现编配色。配色用 CSS 变量声明在 `:root`，改一处即可整体换肤。

## 五套配色

### 1. 深夜机房（默认）

冷蓝底 + 琥珀指示灯。最稳、最"高级感"的一版，适合大多数场合。

```css
:root{
  --bg:#0b0f14;
  --bg-2:#141b24;
  --panel:#1b232e;
  --line:#2a3646;
  --ink:#e8edf3;
  --ink-dim:#8a99ab;
  --accent:#f0a94c;   /* 琥珀：指示灯、按钮 */
  --accent-2:#4aa3df; /* 冷蓝：碎屑高光 */
}
```

### 2. 暖黄台灯

深炭底 + 一束暖光从上方洒下。最有"深夜房间里对着屏幕"的私密感，适合恋人、朋友、家人。

```css
:root{
  --bg:#14110e;
  --bg-2:#1f1a15;
  --panel:#2a231c;
  --line:#3a3026;
  --ink:#f5ece0;
  --ink-dim:#a89a86;
  --accent:#e8b464;
  --accent-2:#c47a4a;
}
```

### 3. 赛博荧光

近黑底 + 青/品红双撞色。适合"狠一点""删得彻底"的表达。

```css
:root{
  --bg:#07070a;
  --bg-2:#0e0e14;
  --panel:#16161f;
  --line:#26263a;
  --ink:#eaf6ff;
  --ink-dim:#7c89a0;
  --accent:#00f0ff;
  --accent-2:#ff2d78;
}
```

### 4. 奶油治愈

浅色系，柔和粉米。最不像"删除"，最适合自我关怀、告别、疗愈主题。

```css
:root{
  --bg:#faf5f0;
  --bg-2:#f3e9e2;
  --panel:#ffffff;
  --line:#e7d9cd;
  --ink:#3a2f28;
  --ink-dim:#9b8a7c;
  --accent:#e0917f;
  --accent-2:#9fb5a0;
}
```

### 5. 工业冷灰

钢灰 + 信号红。冷、硬、机械，像真机器。

```css
:root{
  --bg:#1a1c1e;
  --bg-2:#22262a;
  --panel:#2b3035;
  --line:#3c434a;
  --ink:#dfe4e8;
  --ink-dim:#8792a0;
  --accent:#d94f45;
  --accent-2:#8fa3b8;
}
```

## 字体

主文案用衬线（情绪），信息层用黑体（清晰），装饰小字用宽字距。

```css
--font-serif:"Songti SC","STSong","SimSun",Georgia,"Times New Roman",serif;
--font-sans:"PingFang SC","Hiragino Sans GB","Microsoft YaHei",-apple-system,"Segoe UI",sans-serif;
--font-mono:"SF Mono","JetBrains Mono",Consolas,monospace;
```

- 收尾文案：`--font-serif`，字号 `clamp(1.6rem, 6vw, 2.6rem)`，字重 400–500，字距 `0.06em`
- 辅助标签：`--font-sans`，字号 `0.75rem`，字距 `0.18em`，可全大写
- 文件名：`--font-mono`，`0.8rem`
- 一页最多两种字体家族，第三种会让排版显脏

## 动效参数

| 动作 | 时长 | 缓动 | 说明 |
|---|---|---|---|
| 纸吸入进纸口 | 280–320ms | `cubic-bezier(.4,0,.2,1)` | `scaleY` 压缩 + `translateY` + 轻微 `rotate` |
| 机身震动 | 480–600ms | `steps(8, end)` | `translateX` ±1.5px，制造机械顿挫 |
| 碎屑下落 | 900–1600ms | `cubic-bezier(.4,0,.7,1)` | 重力加速感，越晚落越慢 |
| 碎屑堆积 | 持续 | 线性淡出 | 落地后堆叠不消失，是"战果" |
| 文案淡入 | 600ms | `cubic-bezier(.16,1,.3,1)` | fade + `scale(.96→1)` + 字距收拢 |
| 指示灯呼吸 | 1.6s 循环 | `ease-in-out` | `opacity` + `box-shadow` 扩散 |

碎屑每条的 `x` 速度、旋转角、`animation-delay`、宽度全部随机（`Math.random()`），且**延迟要在生成时就给**，不然会齐刷刷一起掉。

## 质感技巧

噪点叠层（`data:` URI 的 SVG `feTurbulence`），给页面加"胶片颗粒"：

```css
.grain::after{
  content:"";position:absolute;inset:0;pointer-events:none;opacity:.05;
  background-image:url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='.8' numOctaves='3'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
}
```

暗角（深色系必加）：

```css
.vignette{position:absolute;inset:0;pointer-events:none;
  background:radial-gradient(120% 90% at 50% 40%,transparent 35%,rgba(0,0,0,.55) 100%);}
```

扫描线（赛博风用）：`repeating-linear-gradient(0deg, rgba(255,255,255,.03) 0 1px, transparent 1px 3px)`

## 碎纸机画法

机身 = 一个带 `border-radius` 的面板 + 上方一条进纸口（细高亮条）+ 一个指示灯（小圆点，`box-shadow` 做辉光）+ 下方出纸口（碎屑从这里喷出）。

- 面板材质：`linear-gradient(160deg, var(--panel), var(--bg-2))` + `1px solid var(--line)`
- 进纸口：`height:6px` 的深色槽 + `inset 0 1px 0 rgba(255,255,255,.06)` 的高光边
- 指示灯默认暗，进纸时点亮为 `--accent`，碎完缓慢熄灭
- 碎屑从出纸口**下方**生成，落在机身底部形成一小堆

## 避坑

- 不用 emoji 当主视觉（🗑️ 一下就把氛围拉回玩具感）
- 不用「删除成功」「操作完成」这类系统提示语
- 不用渐变文字 + 发光按钮的通用 SaaS 风格
- 不做"居中一个标题 + 一个按钮"的空布局，页面至少要有三层信息：机身、托盘、落点文案
- 深底正文对比度 ≥ 4.5:1；不要用 `--ink-dim` 当正文色
