---
name: shredder-theater
description: |
  Generate a self-contained HTML "shredder theater" page that theatrically
  destroys files on screen: drag files in, watch them get fed into a CSS-drawn
  shredder, hear a synthesized shredding sound, and end on an emotional payoff
  line (e.g. "主人，已处理完毕"). Use this skill when the user asks for a
  碎纸机 / 回收站销毁页 / 删除动效页面 / "把这些文件处理掉" 之类的演出效果, or
  wants a beautiful, shareable file-deletion animation. Do NOT use it for real
  disk deletion, secure data erasure, or generic landing pages and web apps
  (use web-design-engineer for those).
---

# Shredder Theater

把"删除文件"变成一场好看的演出：拖进去 → 碎掉 → 一句话收尾。

产出永远是**一个 HTML 文件**：双击就能看，离线可用，手机打开不塌。

## Inputs to collect

调用前确认这 4 项，缺省就走默认值，不要为了凑齐信息反复追问：

| 项 | 缺省 | 说明 |
|---|---|---|
| 收尾文案 | 从 `references/message-templates.md` 按场景挑一条 | 唯一不能省的项 |
| 氛围配色 | 从 `references/visual-language.md` 挑一套 | 用户给了风格就照做 |
| 交互形态 | 支持真实文件拖入 + 一个「演示删除」按钮 | 按钮是保底演示路径 |
| 落地尺寸 | 响应式单页，手机竖屏优先 | 用 `clamp()` + `dvh` |

- **收尾文案是整个页面的情绪落点**，必须落到具体的一句话。不要写「已删除」「操作成功」这类系统提示语。
- 用户说了对象（"给我妈做的""发给前任"），称呼就从用户给的来，不要自己改成"主人"。
- 用户没说文案、只说了"随便来一个"，按场景选一条最短的，别写长段落。

## Procedure

1. **先定文案，再定视觉**
   文案决定情绪基调，视觉跟着走。反过来做容易得到"好看但没感觉"的页面。
   从 `references/message-templates.md` 挑文案，从 `references/visual-language.md` 挑配色，两处都要在
   `references/visual-language.md` 里记一笔"为什么这么搭"，因为收尾句和背景明暗要互相成就。

2. **搭五层结构**（单文件 HTML，CSS 和 JS 内联，不引外部资源）
   - **舞台层**：背景。深色系用暗角 + 噪点，浅色系用柔和渐变，避免纯色大面积平铺。
   - **碎纸机层**：用 CSS 画机身（面板、指示灯、进纸口、滚轮），不要用图片素材，也不要用 emoji 当主视觉。
   - **托盘层**：待处理文件列表。拖真实文件进来时只取 `file.name` 展示，别读文件内容。
     图片类文件可以用 `URL.createObjectURL()` 给一张缩略图——"亲眼看着那张图被碎掉"才是情绪高点；
     碎完记得 `revokeObjectURL()` 释放内存，缩略图不离开本机。
   - **动效层**：纸屑粒子。纸被吸入进纸口后，从进纸口下沿喷出碎纸条，每条独立随机速度/旋转/落地延迟。
   - **文案层**：收尾那句话，最后出现。

3. **声音用 Web Audio 现场合成，不打包音频文件**
   - 机械嗡鸣：锯齿波 ~70Hz + 低通，白噪声做撕纸底噪
   - 碎纸声：bandpass 白噪声短脉冲，每条纸屑落一次，间隔随机 30–60ms
   - 收尾：一声 sine 叮（880→1320Hz 指数衰减）
   - 浏览器自动播放策略会拦掉首次播放，必须让第一次点击同时解锁音频；页面上给一句"点一下开始"提示。

4. **动效用 transform / opacity，别动 layout 属性**
   纸被吸入：`scaleY` 压缩 + `translateY` 进纸口 + 轻微 `rotate`，约 300ms。
   机身震动：高频小幅 `translateX` 抖动，用 `steps()` 做出机械顿挫感。
   文案出现：fade + `scale(0.96 → 1)` + 字距收拢，约 600ms ease-out。
   碎纸条：`translateY` 落下 + `rotate` 翻转 + 透明度衰减，落地后叠成一小堆。

5. **自检再交付**（这一步不能省）
   用浏览器打开生成的文件，逐条确认：
   - 点一下真的有声音（不是静音）
   - 纸真的被吸进去、碎纸条真的掉下来
   - 文案在碎完之后才出现
   - 375px 宽的手机视口下没有横向滚动、文案不溢出、中文标题没有被挤断成两行
   - 浅色主题下机身和背景能拉开（机器要"立"在背景上，不是浮在背景里）
   - 浏览器 console 无报错、**无未捕获的 Promise 拒绝**（`Uncaught (in promise)` 会让整场演出在中途静默死掉）
   任何一条不过就改完再交付，别把半成品丢给用户。

   演出流程用 `async function` 编排时，在调用处包一层 `try/catch`：出错时至少要把收尾文案显示出来
   并给出提示，不要让页面停在一个"什么都没发生"的状态。

6. **交付**
   给出 HTML 文件的绝对路径，并一句话说明：这页怎么用（拖文件进去 / 点演示按钮）、收尾是什么。
   如果顺手截了图，一并附上。

## Output contract

- **一个** `.html` 文件，自包含：无外部 CSS/JS/字体/图片依赖，断网可开
- 文案、配色写在文件顶部，便于二次改
- 至少支持两条路径：拖真实文件进来、点按钮演示
- 不产出 README、配置文件、构建脚本

## Failure handling

- **用户想真的删文件**：这个 skill 只做演出，不碰磁盘。直说"页面是模拟的，文件还在"，不要顺手去执行删除。
- **浏览器不响**：自动播放策略拦截。需要用户先点击页面；页面自带"点一下开始"提示做兜底。
- **演出中途卡住、什么都不发生**：多半是 `async` 演出函数抛错变成了未捕获的 Promise 拒绝。
  给入口加 `try/catch`，出错就把收尾文案和一句提示显示出来，别让页面停在半路。
- **文件名很长撑破布局**：文件名用 `text-overflow: ellipsis` 截断，完整名放 `title` 属性里。
- **移动端溢出**：`dvh` 比 `vh` 在移动端浏览器更准；长文案用 `clamp()` 控字号。
- **深色背景上的字看不清**：正文与背景对比度至少拉到 4.5:1，深底配奶白字、浅底配墨色字。

## Examples

**Input**："做个碎纸机页面，删完显示'主人，已处理完毕'"

**Output**：单文件 HTML。深色暖黄台灯配色，顶部居中放置 CSS 绘制的碎纸机，中部文件托盘（可拖入真实文件），底部演示按钮。点击后纸被吸入进纸口、机身抖动、Web Audio 合成碎纸声，纸屑从中段掉落堆积，600ms 后「主人，已处理完毕」淡入并伴随一声轻叮。

**Input**："想做一个狠一点的，前任发来的文件全碎掉"

**Output**：同上结构，但配色切到冷灰工业风，文案换成冷嘲语气的短句（如"已永久销毁，不接受复合"），碎纸声更重、机身震动幅度更大。

## Windows (win32) platform notes

- 打开文件：`start "" "<path>\file.html"`（macOS/Linux：`open "<path>/file.html"`）
- 生成文件路径用反斜杠-safe 写法，或在 Python/Node 里用 `path.join`；不要在 SKILL.md 正文里硬编码 `/tmp/`
- macOS/Linux 下拖拽可用 `DataTransfer`，Windows 浏览器同样支持；若某浏览器禁用了拖拽，保留演示按钮作为唯一路径，不要因此报错
