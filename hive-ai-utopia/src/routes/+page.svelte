<script lang="ts">
  import DraggableBox from './DraggableBox.svelte';
  import { onMount } from 'svelte';
  import './styles.css';

  let windowWidth = 0;
  let windowHeight = 0;

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
  
  <div class="content-container">
    <div class="header">
      <div class="title-section">
        <h1 class="title">蜂巢AI理想国</h1>
        <h2 class="subtitle">Hive - AI Utopia</h2>
        <div class="title-number">I</div>
      </div>
      
      <div class="topics-section">
        <h3>对话主题</h3>
        <ul>
          <li>智能体与自动化</li>
          <li>控制性与黑箱</li>
          <li>蒙太奇与数据库美学</li>
          <li>代替性算法与系统</li>
          <li>对齐与偏见</li>
          <li>演绎与归纳</li>
        </ul>
      </div>
    </div>
    
    <div class="info-section">
      <div class="info-row">
        <span class="role">总策划 | </span>
        <span class="name">高世强</span><br/>
        <span class="role-en">Chief Curator | </span>
        <span class="name-en">Gao Shiqiang</span>
      </div>
      
      <div class="info-row">
        <span class="role">策展人 | </span>
        <span class="name">武子扬、张立、赵子琦</span><br/>
        <span class="role-en">Curators | </span>
        <span class="name-en">Wu Ziyang, Zhang Li, Zhao Ziqi</span>
      </div>
      
      <div class="info-row">
        <span class="role">展览执行 | </span>
        <span class="name">夏小燕</span><br/>
        <span class="role-en">Coordinator | </span>
        <span class="name-en">Xia Xiaoyan</span>
      </div>
      
      <div class="info-row">
        <span class="role">出品人 | </span>
        <span class="name">夏季风</span><br/>
        <span class="role-en">Producer | </span>
        <span class="name-en">Xia Jifeng</span>
      </div>
      
      <div class="info-row">
        <span class="role">开幕时间 | </span>
        <span class="name">5月22日下午4点</span><br/>
        <span class="role-en">Opening | </span>
        <span class="name-en">May 22nd, 4pm</span>
      </div>
      
      <div class="info-row">
        <span class="role">展览时间 | </span>
        <span class="name">2025年5月22日至8月22日</span><br/>
        <span class="role-en">Exhibition Dates | </span>
        <span class="name-en">2025.5.22-8.22</span>
      </div>
      
      <div class="info-row">
        <span class="role">地点 | </span>
        <span class="name">蜂巢北京</span><br/>
        <span class="role-en">Venue | </span>
        <span class="name-en">Hive Beijing</span>
      </div>
      
      <div class="info-row">
        <span class="role">地址 | </span>
        <span class="name">北京市朝阳区酒仙桥路4号798艺术区E06</span><br/>
        <span class="role-en">ADD | </span>
        <span class="name-en">E06, 798 Art Zone, Chaoyang District, Beijing</span>
      </div>
    </div>
    
    <div class="logos">
      <div class="organizer">主办单位</div>
      <div class="logo-images">
        <!-- 这里可以放SIMA和蜂巢logo -->
      </div>
    </div>
  </div>
  
  <!-- 创建连接三个框形成三角形的容器 -->
  <div class="triangle-container">
    <!-- 可拖拽的框 - 放在三角形的位置上模拟图片中的布局 -->
    <DraggableBox text="共生" initialX={150} initialY={450} width={120} height={100} />
    <DraggableBox text="对抗" initialX={500} initialY={200} width={120} height={100} />
    <DraggableBox text="与" initialX={350} initialY={400} width={80} height={80} />
    
    <!-- 三角形连线 -->
    <svg class="triangle-svg" width="100%" height="100%" xmlns="http://www.w3.org/2000/svg">
      <!-- 三角形将在JS中动态绘制 -->
    </svg>
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

  /* 不需要这个类了，使用code-background代替 */

  .content-container {
    display: flex;
    flex-direction: column;
    max-width: 1200px;
    margin: 0 auto;
  }

  .header {
    display: flex;
    justify-content: space-between;
    margin-bottom: 60px;
  }

  .title-section {
    position: relative;
  }

  .title {
    font-size: 36px;
    margin: 0;
    font-weight: bold;
  }

  .subtitle {
    font-size: 20px;
    margin: 5px 0 0 0;
    font-weight: normal;
  }

  .title-number {
    position: absolute;
    right: -80px;
    top: -10px;
    font-size: 72px;
    font-weight: bold;
  }

  .topics-section {
    max-width: 300px;
  }

  .topics-section h3 {
    text-align: center;
    margin-bottom: 20px;
  }

  .topics-section ul {
    list-style-type: none;
    padding: 0;
  }

  .topics-section li {
    margin-bottom: 10px;
    font-size: 16px;
    text-align: center;
    position: relative;
  }

  .topics-section li::before {
    content: "•";
    position: absolute;
    left: -20px;
  }

  .info-section {
    margin-bottom: 50px;
  }

  .info-row {
    margin-bottom: 15px;
    line-height: 1.6;
  }

  .role, .role-en {
    font-weight: normal;
  }

  .name, .name-en {
    font-weight: bold;
  }

  .logos {
    display: flex;
    align-items: center;
    justify-content: flex-end;
    gap: 20px;
  }

  .organizer {
    font-size: 14px;
  }

  .logo-images {
    display: flex;
    gap: 30px;
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

  @media (max-width: 768px) {
    .header {
      flex-direction: column;
    }
    
    .title-number {
      position: static;
      text-align: right;
    }
    
    .topics-section {
      margin-top: 40px;
      max-width: none;
    }
  }
</style>
