<script lang="ts">
  import DraggableBox from './DraggableBox.svelte';
  import { onMount } from 'svelte';
  import './styles.css';

  let windowWidth = 0;
  let windowHeight = 0;

  // 三个框的中心点位置数据，初始化为组件的初始位置中心
  let boxPositions = {
    shengsheng: { x: 150 + 120 / 2, y: 450 + 100 / 2 },
    duikang:   { x: 500 + 120 / 2, y: 200 + 100 / 2 },
    yu:        { x: 350 +  80 / 2, y: 400 +  80 / 2 }
  };

  // 更新三角形连线
  function updateTriangle() {
    const svg = document.querySelector('.triangle-svg');
    if (!svg) return;

    // 清空现有的路径
    while (svg.firstChild) {
      svg.removeChild(svg.firstChild);
    }

    // 检查是否有足够的中心点数据来绘制三角形
    if (!boxPositions.shengsheng.x || !boxPositions.shengsheng.y ||
        !boxPositions.duikang.x || !boxPositions.duikang.y ||
        !boxPositions.yu.x || !boxPositions.yu.y) {
      return;
    }

    // 使用中心点直接绘制三角形路径
    const path = document.createElementNS('http://www.w3.org/2000/svg', 'path');
    const d = `M ${boxPositions.shengsheng.x} ${boxPositions.shengsheng.y}
L ${boxPositions.duikang.x} ${boxPositions.duikang.y}
L ${boxPositions.yu.x} ${boxPositions.yu.y}
Z`;
    
    // 设置路径属性
    path.setAttribute('d', d);
    path.setAttribute('fill', 'rgba(255, 246, 160, 0.7)');
    path.setAttribute('stroke', '#e0d782');
    path.setAttribute('stroke-width', '1.5');
    
    // 添加路径到SVG
    svg.appendChild(path);
  }
  
  // 不再使用复杂连接点算法，直接连接中心点
  
  // 处理框移动事件：计算中心点并更新位置
  function handleBoxMove(event) {
    const { id, x, y, width, height } = event.detail;
    boxPositions[id] = { x: x + width / 2, y: y + height / 2 };
    updateTriangle();
  }

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
    
    // 初始化三角形
    setTimeout(updateTriangle, 100);
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
    <!-- 三角形连线 -->
    <svg class="triangle-svg" width="100%" height="100%" xmlns="http://www.w3.org/2000/svg">
      <!-- 三角形将在JS中动态绘制 -->
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
