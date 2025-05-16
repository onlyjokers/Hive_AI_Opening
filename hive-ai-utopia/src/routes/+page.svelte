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
    "  return 和谐;",
    "  共生 += random(-0.1, 0.1);",
    "}",
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
    "  return 和谐;",
    "  共生 += random(-0.1, 0.1);",
    "}"
  ]; 
  
  // 构造带动态插入盒子文本的行序列
  // 先将所有 codeLines 转为 code 类型行
  let staticLines: Array<{ type: 'code' | 'box'; id?: string; text: string }> = codeLines.map(text => ({ type: 'code', text }));
  // 随机将 boxes 插入到 staticLines 中
  boxes.forEach(b => {
    const idx = Math.floor(Math.random() * (staticLines.length + 1));
    staticLines.splice(idx, 0, { type: 'box', id: b.id, text: b.text });
  });
  // 为确保足够滚动内容，可在前后各加 10 行空白注释
  const padLines = Array(10).fill({ type: 'code', text: '' });
  staticLines = [...padLines, ...staticLines, ...padLines];

  let codeDiv: HTMLDivElement;
  let linesContainer: HTMLDivElement;
  // 为每行代码元素准备引用数组
  let lineRefs: Array<HTMLDivElement> = Array(staticLines.length);
  // 记录每个盒子的显示状态与初始位置
  let boxStates = {} as Record<string, { visible: boolean; initialX: number; initialY: number }>;
  for (const b of boxes) {
    boxStates[b.id] = { visible: false, initialX: 0, initialY: 0 };
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
          y: viewH - fullH,
          duration: 30,
          ease: 'none',
          onUpdate: () => {
            // 检测视口区域
            const viewportRect = codeDiv.getBoundingClientRect();
            
            // 遍历检测所有盒子类型行
            staticLines.forEach((line, idx) => {
              if (line.type === 'box' && !boxStates[line.id].visible) {
                const lineEl = lineRefs[idx];
                if (lineEl) {
                  const lineRect = lineEl.getBoundingClientRect();
                  
                  // 当行进入视口后才触发显示
                  if (lineRect.top >= viewportRect.top && 
                      lineRect.bottom <= viewportRect.bottom && 
                      lineRect.top <= viewportRect.bottom) {
                    
                    // 行完全进入视口后，等待2-6秒的随机时间
                    const delay = 2000 + Math.random() * 4000; // 2-6秒
                    
                    setTimeout(() => {
                      // 重新获取当前位置，确保使用最新位置
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
            });
          }
        }
      );
    }
  });
</script>

<svelte:window bind:innerWidth={windowWidth} bind:innerHeight={windowHeight} />

<!-- 代码背景 -->
<div bind:this={codeDiv} class="code-background">
  <div bind:this={linesContainer} class="lines-container">
    {#each staticLines as line, i}
      <div
        bind:this={lineRefs[i]}
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
        <div in:fade={{ duration: 200 }}>  <!-- wrapper for fade transition -->
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
