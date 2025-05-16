<script lang="ts">
  import DraggableBox from './DraggableBox.svelte';
  import { onMount } from 'svelte';
  import './styles.css';
  import { gsap } from 'gsap';

  let windowWidth = 0;
  let windowHeight = 0;

  // 定义可拖拽框的配置数组，支持动态增删
  const boxes = [
    { id: 'shengsheng', text: '共生', initialX: 150, initialY: 450, rotation: 0 },
    { id: 'duikang',   text: '对抗', initialX: 500, initialY: 200, rotation: 0 },
    { id: 'yu',        text: '与',   initialX: 350, initialY: 400, rotation: 0 },
    { id: 'ni',        text: '你',   initialX: 30,  initialY: 400, rotation: 0 }
  ];
  // 存储每个框的最新状态，用于计算包裹轮廓，默认尺寸为0
  let boxData = {} as Record<string, { x: number; y: number; width: number; height: number; rotation: number }>;
  for (const b of boxes) {
    boxData[b.id] = { x: b.initialX, y: b.initialY, width: 0, height: 0, rotation: b.rotation };
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

  // 生成SVG路径字符串
  $: hullPath = (() => {
    // 只有当所有框都有数据时才计算凸包
    if (!boxes.every(b => boxData[b.id])) return '';
    const pts: { x: number; y: number }[] = [];
    for (const b of boxes) {
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

  // 背景代码
  const codeSnippets = [
    "共生 += random(-0.1, 0.1);",
    "class 生态系统 {",
    "  共生 += random(-0.1, 0.1);",
    "  this.共生 = 0.5;",
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

  let codeDiv: HTMLDivElement;
  onMount(() => {
    windowWidth = window.innerWidth;
    windowHeight = window.innerHeight;
    // 使用 GSAP 逐行向下滚动代码背景
    if (codeDiv) {
      // 从顶部开始
      codeDiv.scrollTop = 0;
      gsap.to(codeDiv, {
        scrollTop: codeDiv.scrollHeight,
        duration: 30,
        ease: 'none',
        repeat: -1
      });
    }
  });
</script>

<svelte:window bind:innerWidth={windowWidth} bind:innerHeight={windowHeight} />

<!-- 代码背景 -->
<div bind:this={codeDiv} class="code-background">
  {#each codeSnippets as snippet}
    {snippet}
    <br>
  {/each}
</div>

<main>
  <!-- 创建连接三个框形成三角形的容器 -->
  <div class="triangle-container">
    <!-- 凸包多边形包裹效果 -->
    <svg class="triangle-svg" width="100%" height="100%" xmlns="http://www.w3.org/2000/svg">
      <path
        d={hullPath}
        fill="rgba(255,246,160,0.7)"
        stroke="#e0d782"
        stroke-width="1.5"
      />
    </svg>
    
    <!-- 可拖拽的框 - 动态渲染所有框 -->
    {#each boxes as b}
      <DraggableBox
        id={b.id}
        text={b.text}
        initialX={b.initialX}
        initialY={b.initialY}
        rotation={b.rotation}
        on:move={handleBoxMove}
      />
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
