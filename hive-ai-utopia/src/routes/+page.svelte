<script lang="ts">
  import DraggableBox from './DraggableBox.svelte';
  import { onMount } from 'svelte';
  import './styles.css';

  let windowWidth = 0;
  let windowHeight = 0;

  // 定义可拖拽框的配置数组
  const boxes = [
    { id: 'shengsheng', text: '共生', initialX: 150, initialY: 450, width: 120, height: 100, rotation: 0 },
    { id: 'duikang',   text: '对抗', initialX: 500, initialY: 200, width: 120, height: 100, rotation: 10 },
    { id: 'yu',        text: '与',   initialX: 350, initialY: 400, width:  80, height:  80, rotation: 0 }
  ];
  // 存储每个框的最新状态，用于计算包裹轮廓
  let boxData = {} as Record<string, { x: number; y: number; width: number; height: number; rotation: number }>;
  // 初始化boxData默认值，避免NaN
  for (const b of boxes) {
    boxData[b.id] = { x: b.initialX, y: b.initialY, width: b.width, height: b.height, rotation: b.rotation };
  }
  // 初始化 boxData，确保初始值存在
  for (const b of boxes) {
    boxData[b.id] = { x: b.initialX, y: b.initialY, width: b.width, height: b.height, rotation: b.rotation };
  }

  // 初始化 boxData 的默认值，避免 NaN
  for (const b of boxes) {
    boxData[b.id] = { x: b.initialX, y: b.initialY, width: b.width, height: b.height, rotation: b.rotation };
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
    "}"
  ];

  onMount(() => {
    windowWidth = window.innerWidth;
    windowHeight = window.innerHeight;
    // 初始化 boxData 为初始框配置
    for (const b of boxes) {
      boxData[b.id] = { x: b.initialX, y: b.initialY, width: b.width, height: b.height, rotation: b.rotation };
    }
  });
</script>

<svelte:window bind:innerWidth={windowWidth} bind:innerHeight={windowHeight} />

<!-- 代码背景 -->
<div class="code-background">
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
    
    <!-- 可拖拽的框 - 放在三角形的位置上模拟图片中的布局 -->
    <DraggableBox 
      id="shengsheng" 
      text="共生" 
      initialX={150} 
      initialY={450} 
      width={120} 
      height={100} 
      on:move={handleBoxMove} 
    />
    <DraggableBox 
      id="duikang" 
      text="对抗" 
      initialX={500} 
      initialY={200} 
      width={120} 
      height={100} 
      rotation={10}
      on:move={handleBoxMove} 
    />
    <DraggableBox 
      id="yu" 
      text="与" 
      initialX={350} 
      initialY={400} 
      width={80} 
      height={80}
      on:move={handleBoxMove} 
    />
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
