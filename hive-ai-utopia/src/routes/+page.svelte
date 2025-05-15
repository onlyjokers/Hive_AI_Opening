<script lang="ts">
  import DraggableBox from './DraggableBox.svelte';
  import { onMount } from 'svelte';
  import './styles.css';

  let windowWidth = 0;
  let windowHeight = 0;

  // 三个框的位置数据
  let boxPositions = {
    shengsheng: { center: { x: 0, y: 0 }, edges: null, edgeCenters: null },
    duikang: { center: { x: 0, y: 0 }, edges: null, edgeCenters: null },
    yu: { center: { x: 0, y: 0 }, edges: null, edgeCenters: null }
  };

  // 更新三角形连线
  function updateTriangle() {
    const svg = document.querySelector('.triangle-svg');
    if (!svg) return;

    // 清空现有的路径
    while (svg.firstChild) {
      svg.removeChild(svg.firstChild);
    }

    // 检查是否有足够的位置数据来绘制三角形
    if (!boxPositions.shengsheng?.edges || 
        !boxPositions.duikang?.edges || 
        !boxPositions.yu?.edges) {
      return;
    }

    // 计算连接点（找到最合适的边缘点以形成三角形）
    const points = calculateOptimalConnectionPoints();
    
    // 创建三角形路径
    const path = document.createElementNS('http://www.w3.org/2000/svg', 'path');
    
    // 计算路径数据
    const d = `
      M ${points.shengsheng.x} ${points.shengsheng.y}
      L ${points.duikang.x} ${points.duikang.y}
      L ${points.yu.x} ${points.yu.y}
      Z
    `;
    
    // 设置路径属性
    path.setAttribute('d', d);
    path.setAttribute('fill', 'rgba(255, 246, 160, 0.7)'); // 淡黄色半透明
    path.setAttribute('stroke', '#e0d782');
    path.setAttribute('stroke-width', '1.5');
    
    // 添加路径到SVG
    svg.appendChild(path);
  }
  
  // 计算三角形最优连接点（找到每个框最合适的边缘点）
  function calculateOptimalConnectionPoints() {
    // 准确计算框之间的连接点
    const getOptimalPoint = (from, to1, to2) => {
      // 获取当前框的位置信息
      const box = boxPositions[from];
      if (!box || !box.position) return { x: 0, y: 0 }; // 默认值
      
      // 计算三个框的质心
      const allCenters = [
        boxPositions.shengsheng.center,
        boxPositions.duikang.center,
        boxPositions.yu.center
      ];
      
      const centroid = {
        x: (allCenters[0].x + allCenters[1].x + allCenters[2].x) / 3,
        y: (allCenters[0].y + allCenters[1].y + allCenters[2].y) / 3
      };
      
      // 根据框的位置相对于质心的方向，确定连接点
      // 这个算法会使连接点位于框的外边缘
      const { x, y, width, height } = box.position;
      const centerX = x + width / 2;
      const centerY = y + height / 2;
      
      // 计算框中心到质心的向量
      const dx = centroid.x - centerX;
      const dy = centroid.y - centerY;
      
      // 根据框到质心的方向确定边缘上的点
      // 选择一个最接近方向向量的点
      let connX, connY;
      
      // 计算方向向量的角度
      const angle = Math.atan2(dy, dx);
      const PI = Math.PI;
      
      // 根据角度选择适当的边缘点
      // 将圆周划分为8个部分，选择最接近的边缘
      if (angle > -PI/8 && angle <= PI/8) {
        // 右
        connX = x + width;
        connY = y + height/2;
      } else if (angle > PI/8 && angle <= 3*PI/8) {
        // 右下
        connX = x + width;
        connY = y + height;
      } else if (angle > 3*PI/8 && angle <= 5*PI/8) {
        // 下
        connX = x + width/2;
        connY = y + height;
      } else if (angle > 5*PI/8 && angle <= 7*PI/8) {
        // 左下
        connX = x;
        connY = y + height;
      } else if ((angle > 7*PI/8 && angle <= PI) || (angle <= -7*PI/8 && angle > -PI)) {
        // 左
        connX = x;
        connY = y + height/2;
      } else if (angle > -7*PI/8 && angle <= -5*PI/8) {
        // 左上
        connX = x;
        connY = y;
      } else if (angle > -5*PI/8 && angle <= -3*PI/8) {
        // 上
        connX = x + width/2;
        connY = y;
      } else {
        // 右上
        connX = x + width;
        connY = y;
      }
      
      return { x: connX, y: connY };
    };
    
    // 计算每个框的最佳连接点
    return {
      shengsheng: getOptimalPoint('shengsheng', 'duikang', 'yu'),
      duikang: getOptimalPoint('duikang', 'shengsheng', 'yu'),
      yu: getOptimalPoint('yu', 'shengsheng', 'duikang')
    };
  }
  
  // 处理框移动事件
  function handleBoxMove(event) {
    // 获取完整的位置数据
    const { id, center, edges, edgeCenters, position, width, height } = event.detail;
    boxPositions[id] = {
      center,
      edges,
      edgeCenters,
      position,
      width,
      height
    };
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
