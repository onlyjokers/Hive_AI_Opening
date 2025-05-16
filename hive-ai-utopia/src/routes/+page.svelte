<script lang="ts">
  import DraggableBox from './DraggableBox.svelte';
  import { onMount } from 'svelte';
import './styles.css';
  import { gsap } from 'gsap';
import { fade } from 'svelte/transition';

  let windowWidth = 0;
  let windowHeight = 0;
  let animationDone = false;

  // 定义可拖拽框的配置数组，仅包含 id/text/rotation
  const boxes = [
    { id: 'works', text: '██落在夜空生成以前', rotation: 0 },
    { id: 'name', text: '蔡宇潇  Justin Bortnick', rotation: 0 },
    { id: 'type', text: '文学游戏', rotation: 0 },
    { id: 'time', text: '2024-2025', rotation: 0 },
    { id: 'duration', text: '时长不限', rotation: 0 }
  ];

  // 盒子状态对象会在后面初始化

  // 存储每个框的最新状态，用于计算包裹轮廓，默认尺寸为0
  // 存储每个框的最新状态，用于计算包裹轮廓，默认全零
  let boxData = {} as Record<string, { x: number; y: number; width: number; height: number; rotation: number }>;
  for (const b of boxes) {
    boxData[b.id] = { x: 0, y: 0, width: 0, height: 0, rotation: b.rotation };
  }
  // 处理框移动事件，更新boxData
  function handleBoxMove(event) {
    const { id, x, y, width, height, rotation } = event.detail;
    boxData[id] = { x, y, width, height, rotation };
  }

  // 计算多边形凸包（Monotone Chain算法）
  function convexHull(points: { x: number; y: number }[]) {
    if (points.length <= 1) return points;
    const sorted = points.slice().sort((a, b) => a.x === b.x ? a.y - b.y : a.x - b.x);
    const cross = (o, a, b) => (a.x - o.x) * (b.y - o.y) - (a.y - o.y) * (b.x - o.x);
    const lower = [];
    for (const p of sorted) {
      while (lower.length >= 2 && cross(lower[lower.length-2], lower[lower.length-1], p) <= 0) lower.pop();
      lower.push(p);
    }
    const upper = [];
    for (let i = sorted.length - 1; i >= 0; i--) {
      const p = sorted[i];
      while (upper.length >= 2 && cross(upper[upper.length-2], upper[upper.length-1], p) <= 0) upper.pop();
      upper.push(p);
    }
    upper.pop(); lower.pop();
    return lower.concat(upper);
  }

  // 生成SVG路径字符串，仅对已显现的盒子计算凸包
  $: hullPath = (() => {
    const visibleBoxes = boxes.filter(b => boxStates[b.id].visible);
    if (visibleBoxes.length < 2) return '';
    const pts: { x: number; y: number }[] = [];
    for (const b of visibleBoxes) {
      const d = boxData[b.id];
      if (d && !isNaN(d.x) && !isNaN(d.y)) {
        // 计算四个角的坐标
        const rad = d.rotation * Math.PI / 180;
        const cx = d.x + d.width / 2;
        const cy = d.y + d.height / 2;
        const hw = d.width / 2;
        const hh = d.height / 2;
        const corners = [
          { dx: -hw, dy: -hh },
          { dx: hw, dy: -hh },
          { dx: hw, dy: hh },
          { dx: -hw, dy: hh }
        ];
        for (const { dx, dy } of corners) {
          const x = cx + dx * Math.cos(rad) - dy * Math.sin(rad);
          const y = cy + dx * Math.sin(rad) + dy * Math.cos(rad);
          pts.push({ x, y });
        }
      }
    }
    const hull = convexHull(pts);
    if (hull.length === 0) return '';
    return 'M ' + hull.map(p => `${p.x} ${p.y}`).join(' L ') + ' Z';
  })();

  // （保留单一handleBoxMove，已在上方定义）

  // 定义更多丰富的代码行
  const codeLines = [
    "共生 += random(-0.1, 0.1);",
    "class 生态系统 {",
    "  共生 += random(-0.1, 0.1);",
    "  this.共生 = 0.5;",
    "  let 平衡 = 共生 * 对抗;",
    "  constructor() {",
    "    关系.push(new 关系模型());",
    "  }",
    "  for (let i = 0; i < 3; i++) {",
    "    对抗 += random(-0.1, 0.1);",
    "  }",
    "  function 对抗与共生() {",
    "    class 生态系统 {",
    "      关系.push(new 关系模型());",
    "    }",
    "  }",
    "  function 更新状态() {",
    "    this.共生 = 0.5;",
    "    return 和谐;",
    "    constructor() {",
    "    }",
    "  }",
    "  共生 += random(-0.1, 0.1);",
    "  对抗 += random(-0.1, 0.1);",
    "  let 平衡 = 共生 * 对抗;",
    "  let 和谐 = 平衡 * random(0.5, 1.5);",
    "  let 关系 = [];",
    "  let 对抗 = 0;",
    "  let 和谐 = 0;",
    "  return 和谐;",
    "}",
  ]; 
  
  // 构造带动态插入盒子文本的行序列
  // 先将所有 codeLines 转为 code 类型行
  let staticLines: Array<{ type: 'code' | 'box'; id?: string; text: string }> = codeLines.map(text => ({ type: 'code', text }));
  // 为确保足够滚动内容，在前后各加 padCount 行空白注释
  const padCount = 20;
  const padLines = Array(padCount).fill({ type: 'code', text: '' });
  staticLines = [...padLines, ...staticLines, ...padLines];
  // 按要求：第一个插入在 padCount+1 行，最后一个插入在倒数 padCount+1 前，其余随机递增
  // 第一个盒子固定位置
  const firstIdx = padCount + 1;
  staticLines.splice(firstIdx, 0, { type: 'box', id: boxes[0].id, text: boxes[0].text });
  // 最后一个盒子固定位置
  const lastIdx = staticLines.length - padCount - 1;
  staticLines.splice(lastIdx, 0, { type: 'box', id: boxes[boxes.length - 1].id, text: boxes[boxes.length - 1].text });
  // 中间盒子随机插入，保持顺序递增
  let prevIdx = firstIdx + 1;
  for (let i = 1; i < boxes.length - 1; i++) {
    const b = boxes[i];
    const remaining = (boxes.length - 1) - i;
    const maxIdx = staticLines.length - padCount - remaining + 1;
    const idx = prevIdx + Math.floor(Math.random() * (maxIdx - prevIdx + 1));
    staticLines.splice(idx, 0, { type: 'box', id: b.id, text: b.text });
    prevIdx = idx + 1;
  }

  let codeDiv: HTMLDivElement;
  let linesContainer: HTMLDivElement;
  // 为每行代码元素准备引用数组
  let lineRefs: Array<HTMLDivElement> = Array(staticLines.length);
  /**
   * 自定义action：收集每行代码行的DOM引用
   * @param element 行元素
   * @param idx 行索引
   */
  function captureLineRef(element: HTMLDivElement, idx: number) {
    lineRefs[idx] = element;
    return {
      destroy() {
        delete lineRefs[idx];
      }
    };
  }
  // 记录每个盒子的显示状态与初始位置
  let boxStates = {} as Record<string, { visible: boolean; initialX: number; initialY: number }>;
  // 标记每个盒子是否已启动显示延时，避免重复计时
  let boxScheduled = {} as Record<string, boolean>;
  // 记录渲染后的盒子容器引用，使用action捕获
  let wrapperRefs: Record<string, HTMLDivElement> = {};
  /**
   * 自定义action，用于捕获wrapper div引用
   * @param element wrapper div元素
   * @param id 盒子id
   */
  function captureRef(element: HTMLDivElement, id: string) {
    wrapperRefs[id] = element;
    return {
      destroy() {
        delete wrapperRefs[id];
      }
    };
  }
  // 按出现顺序记录已显示的盒子id
  let visibleOrder: string[] = [];
  // 最后一个盒子id，用于延迟显示
  const lastId = boxes[boxes.length - 1].id;
  // 标记最后一个盒子已进入视口但等待其他盒子显示完成
  let pendingLast = false;
  for (const b of boxes) {
    boxStates[b.id] = { visible: false, initialX: 0, initialY: 0 };
    boxScheduled[b.id] = false;
  }

  onMount(() => {
    windowWidth = window.innerWidth;
    windowHeight = window.innerHeight;
    if (codeDiv && linesContainer) {
      const viewH = codeDiv.clientHeight;
      const fullH = linesContainer.scrollHeight;
      gsap.fromTo(
        linesContainer,
        { y: viewH },
        {
          // 滚动到内容底部与视口底对齐时停止：y = viewH - fullH
          // y: viewH - fullH,
          y: viewH/1.1 - fullH,
          duration: 10,
          ease: 'none',
          onUpdate: () => {
            // 检测视口区域
            const viewportRect = codeDiv.getBoundingClientRect();
            
            // 遍历检测所有盒子类型行
            staticLines.forEach((line, idx) => {
              if (line.type === 'box' && !boxStates[line.id].visible && !boxScheduled[line.id]) {
                const lineEl = lineRefs[idx];
                if (lineEl) {
                  const lineRect = lineEl.getBoundingClientRect();
                  // 当行进入视口后才触发显示
                  if (lineRect.top >= viewportRect.top && lineRect.bottom <= viewportRect.bottom) {
                    // 仅触发一次
                    if (line.id === lastId) {
                      // 延迟最后一个盒子的显示，等待其他盒子出现
                      pendingLast = true;
                    } else {
                      boxScheduled[line.id] = true;
                      const delay = 1500 + Math.random() * 2000;
                      setTimeout(() => {
                        const pos = lineEl.getBoundingClientRect();
                        boxStates[line.id] = {
                          visible: true,
                          initialX: pos.left,
                          initialY: pos.top
                        };
                      }, delay);
                    }
                  }
                }
              }
            });
          }
        }
      );
    }
  });
  // 延迟最后一个盒子显示，需等待其它盒子都已出现
  $: if (pendingLast &&
          boxes.filter(b => b.id !== lastId).every(b => boxStates[b.id].visible) &&
          !boxScheduled[lastId]) {
    // 标记已调度
    boxScheduled[lastId] = true;
    // 获取最后一行对应的 DOM 引用
    const idx = staticLines.findIndex(l => l.type === 'box' && l.id === lastId);
    const lineEl = idx >= 0 ? lineRefs[idx] : null;
    if (lineEl) {
      const pos = lineEl.getBoundingClientRect();
      // 直接显示最后一个盒子
      boxStates[lastId] = { visible: true, initialX: pos.left, initialY: pos.top };
    }
    // 重置标记
    pendingLast = false;
  }
  // 监测可见状态变化，触发移动动画
  $: {
    const currentVis = boxes.filter(b => boxStates[b.id].visible).map(b => b.id);
    if (currentVis.length > visibleOrder.length) {
      const newIds = currentVis.filter(id => !visibleOrder.includes(id));
      newIds.forEach(id => visibleOrder.push(id));
      // 每次有新box出现后0.2s触发移动
      setTimeout(() => {
        const latest = visibleOrder[visibleOrder.length - 1];
        const isLast = visibleOrder.length === boxes.length && latest === boxes[boxes.length - 1].id;
        // 目标id列表
        const targets = isLast ? visibleOrder.filter(id => id !== latest) : [...visibleOrder];
        // 定义最后布局时各盒子Y轴位置
        const finalYs = [20, 100, 150, 200, 250];

        targets.forEach(id => {
          const wrapper = wrapperRefs[id];
          if (wrapper) {
            const boxEl = wrapper.querySelector('.draggable-box') as HTMLElement;
            if (boxEl) {
              let newX: number;
              let newY: number;
              if (isLast) {
                newX = 20;
                // 根据boxes中原始顺序索引设定Y
                const idx = boxes.findIndex(b => b.id === id);
                newY = finalYs[idx] ?? (20 + idx * 30);
              } else {
                // 普通随机移动
                const w = boxEl.offsetWidth;
                const h = boxEl.offsetHeight;
                newX = Math.random() * (windowWidth - w);
                newY = Math.random() * (windowHeight - h);
              }
              // 移动后实时更新 boxData 以触发凸包重计算
              gsap.to(boxEl, {
                left: newX,
                top: newY,
                duration: 0.5,
                onUpdate: () => {
                  const rect = boxEl.getBoundingClientRect();
                  boxData[id] = {
                    x: rect.left,
                    y: rect.top,
                    width: boxEl.offsetWidth,
                    height: boxEl.offsetHeight,
                    rotation: boxData[id].rotation
                  };
                }
              });
            }
          }
        });
      }, 200);
    }
  }
</script>

<svelte:window bind:innerWidth={windowWidth} bind:innerHeight={windowHeight} />

<!-- 代码背景 -->
<div bind:this={codeDiv} class="code-background">
  <div bind:this={linesContainer} class="lines-container">
    {#each staticLines as line, i}
      <div
        use:captureLineRef={i}
        class:selected={line.type === 'box'}
      >
        {line.text}
      </div>
    {/each}
  </div>
</div>

<main>
  <!-- 创建连接三个框形成三角形的容器 -->
  <div class="triangle-container">
    <!-- 凸包多边形包裹效果 -->
  <svg class="triangle-svg" width="100%" height="100%" xmlns="http://www.w3.org/2000/svg">
      <path
        d={hullPath}
        fill="rgba(255, 246, 160, 0.25)"
        stroke="#e0d782"
        stroke-width="1.5"
      />
    </svg>
      
    <!-- 可拖拽的框 - 根据滚动触发显示 -->
    {#each boxes as b}
      {#if boxStates[b.id].visible}
        <div use:captureRef={b.id} in:fade={{ duration: 200 }}>
          <DraggableBox
            id={b.id}
            text={b.text}
            initialX={boxStates[b.id].initialX}
            initialY={boxStates[b.id].initialY}
            rotation={b.rotation}
            on:move={handleBoxMove}
          />
        </div>
      {/if}
    {/each}
  </div>
</main>

<style>
  /* 样式已移至styles.css */

  main {
    width: 100%;
    min-height: 100vh;
    position: relative;
    padding: 20px;
    box-sizing: border-box;
  }

  .triangle-container {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 5;
    pointer-events: none;
  }

  .triangle-svg {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 1;
    pointer-events: none;
  }
  
  /* 确保DraggableBox可以被拖动 */
  .triangle-container :global(.draggable-box) {
    pointer-events: auto;
  }
</style>
