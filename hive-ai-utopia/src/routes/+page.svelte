<script lang="ts">
  import DraggableBox from './DraggableBox.svelte';
  import { onMount } from 'svelte';
  import './styles.css';
  import { gsap } from 'gsap';
  import { fade } from 'svelte/transition';
  import { goto } from '$app/navigation';

  let windowWidth = 0;
  let windowHeight = 0;
  let animationDone = false;

  // 定义默认可拖拽框的配置数组
  const defaultBoxes = [
    { id: 'works', text: '██落在夜空生成以前', rotation: 0 },
    { id: 'name', text: '蔡宇潇  Justin Bortnick', rotation: 0 },
    { id: 'type', text: '文学游戏', rotation: 0 },
    { id: 'time', text: '2024-2025', rotation: 0 },
    { id: 'duration', text: '时长不限', rotation: 0 }
  ];

  // 初始化盒子，将在onMount中尝试从本地存储加载
  let boxes = [...defaultBoxes];

  // 盒子状态对象会在后面初始化

  // 存储每个框的最新状态，用于计算包裹轮廓，默认尺寸为0
  // 存储每个框的最新状态，用于计算包裹轮廓，默认全零
  let boxData = {} as Record<string, { x: number; y: number; width: number; height: number; rotation: number }>;
  
  // 初始化boxData函数，在boxes更新后需要调用
  function initBoxData() {
    boxData = {} as Record<string, { x: number; y: number; width: number; height: number; rotation: number }>;
    for (const b of boxes) {
      boxData[b.id] = { x: 0, y: 0, width: 0, height: 0, rotation: b.rotation };
    }
  }
  
  // 初始化boxData
  initBoxData();

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
    const visibleBoxes = boxes.filter(b => boxStates[b.id]?.visible);
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
  // 最后一个盒子固定位置，等待所有中间插入完成后计算
  const lastIdx = staticLines.length - padCount - 1;
  staticLines.splice(lastIdx, 0, { type: 'box', id: boxes[boxes.length - 1].id, text: boxes[boxes.length - 1].text });

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

  // 初始化盒子状态
  function initBoxStates() {
    // 重新初始化 boxStates 和 boxScheduled
    boxStates = {};
    boxScheduled = {};
    
    // 为每个盒子设置初始状态
    for (const b of boxes) {
      boxStates[b.id] = { visible: false, initialX: 0, initialY: 0 };
      boxScheduled[b.id] = false;
    }
    
    // 重置可见顺序
    visibleOrder = [];
  }

  onMount(() => {
    // 尝试从本地存储加载自定义盒子数据
    try {
      // 首先检查是否有来自 indexer 的文本输入
      const savedTexts = localStorage.getItem('hive-ai-texts');
      if (savedTexts) {
        const texts = savedTexts.split(',').map(t => t.trim()).filter(t => t);
        if (texts.length > 0) {
          // 根据 texts 动态生成盒子
          boxes = texts.map((text, idx) => ({
            id: `box${idx}`,
            text,
            rotation: 0
          }));
          localStorage.setItem('hive-ai-boxes', JSON.stringify(boxes));
          initBoxData();
          initBoxStates(); // 添加这一行初始化 boxStates
          rebuildStaticLines();
        }
      } else {
        // 检查是否有已保存的完整盒子配置
        const savedBoxes = localStorage.getItem('hive-ai-boxes');
        if (savedBoxes) {
          boxes = JSON.parse(savedBoxes);
          // 重新初始化boxData
          initBoxData();
          initBoxStates(); // 添加这一行初始化 boxStates
          // 重新构建静态行
          rebuildStaticLines();
        }
      }
      
      // 检查是否需要录制视频
      const shouldRecord = localStorage.getItem('hive-ai-should-record');
      if (shouldRecord === 'true') {
        // 立即移除标记，防止页面刷新时重新录制
        localStorage.removeItem('hive-ai-should-record');
        
        // 延迟一小段时间后开始录制，等待页面完全渲染
        setTimeout(() => {
          startScreenRecording();
        }, 500);
      }
    } catch (e) {
      console.error('加载自定义盒子数据时出错:', e);
    }
    
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
          y: (viewH/1 - 100) - fullH,
          duration: 8,
          ease: 'none',
          onUpdate: () => {
            // 检测视口区域
            const viewportRect = codeDiv.getBoundingClientRect();
            
            // 遍历检测所有盒子类型行
            staticLines.forEach((line, idx) => {
              if (
                line.type === 'box' &&
                boxStates[line.id] &&                   // 确保已有该盒子状态
                !boxStates[line.id].visible &&
                !boxScheduled[line.id]
              ) {
                const lineEl = lineRefs[idx];
                if (lineEl) {
                  const lineRect = lineEl.getBoundingClientRect();
                  // 当行进入视口后才触发显示
                  if (lineRect.top >= viewportRect.top && lineRect.bottom <= viewportRect.bottom) {
                    // 仅触发一次
                    boxScheduled[line.id] = true;
                    const delay = 1500;
                    setTimeout(() => {
                      const showBox = () => {
                        const pos = lineEl.getBoundingClientRect();
                        boxStates[line.id] = {
                          visible: true,
                          initialX: pos.left,
                          initialY: pos.top
                        };
                      };
                      if (line.id === lastId) {
                        // 等待其它盒子可见后再显示
                        if (boxes.filter(b => b.id !== lastId).every(b => boxStates[b.id].visible)) {
                          showBox();
                        } else {
                          const checkInterval = setInterval(() => {
                            if (boxes.filter(b => b.id !== lastId).every(b => boxStates[b.id].visible)) {
                              showBox();
                              clearInterval(checkInterval);
                            }
                          }, 100);
                        }
                      } else {
                        showBox();
                      }
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

  // 重新构建包含盒子文本的静态行序列
  function rebuildStaticLines() {
    // 重置静态行
    staticLines = codeLines.map(text => ({ type: 'code', text }));
    // 添加填充行
    staticLines = [...padLines, ...staticLines, ...padLines];
    
    // 第一个盒子固定位置
    const firstIdx = padCount + 1;
    staticLines.splice(firstIdx, 0, { type: 'box', id: boxes[0].id, text: boxes[0].text });
    
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
    
    // 最后一个盒子固定位置
    const lastIdx = staticLines.length - padCount - 1;
    staticLines.splice(lastIdx, 0, { type: 'box', id: boxes[boxes.length - 1].id, text: boxes[boxes.length - 1].text });
  }

  // 视频录制相关变量
  let mediaRecorder: MediaRecorder | null = null;
  let recordedChunks: BlobPart[] = [];
  
  // 开始屏幕录制的函数
  async function startScreenRecording() {
    try {
      // 调整窗口大小
      window.resizeTo(1728, 832);
      
      // 获取屏幕共享
      const stream = await navigator.mediaDevices.getDisplayMedia({
        video: { width: 1728, height: 832 },
        audio: false
      });
      
      // 创建 MediaRecorder 对象
      mediaRecorder = new MediaRecorder(stream, { mimeType: 'video/webm' });
      
      // 收集录制的数据
      mediaRecorder.ondataavailable = (event) => {
        if (event.data.size > 0) {
          recordedChunks.push(event.data);
        }
      };
      
      // 录制结束后下载视频
      mediaRecorder.onstop = () => {
        // 创建视频 Blob
        const blob = new Blob(recordedChunks, { type: 'video/webm' });
        
        // 创建下载链接
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        document.body.appendChild(a);
        a.style.display = 'none';
        a.href = url;
        a.download = '蜂巢AI理想国_' + new Date().toISOString() + '.webm';
        a.click();
        
        // 清理
        setTimeout(() => {
          document.body.removeChild(a);
          window.URL.revokeObjectURL(url);
          recordedChunks = [];
        }, 100);
      };
      
      // 开始录制
      mediaRecorder.start();
      
      // 10秒后停止录制
      setTimeout(() => {
        if (mediaRecorder && mediaRecorder.state !== 'inactive') {
          mediaRecorder.stop();
          
          // 停止所有视频轨道
          stream.getTracks().forEach(track => track.stop());
        }
      }, 10000);
      
    } catch (err) {
      console.error('无法录制屏幕:', err);
      alert('无法开始录制，请确保已授予屏幕共享权限。');
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
  
  <!-- 返回编辑器按钮 -->
  <!-- <a href="/indexer" class="edit-button">编辑文本</a> -->
</main>

<style lang="css">
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
  
  /* 编辑按钮样式 */
  .edit-button {
    position: fixed;
    bottom: 20px;
    right: 20px;
    padding: 8px 16px;
    background-color: rgba(255, 246, 160, 0.15);
    color: #e0d782;
    border: 1px solid #e0d782;
    border-radius: 4px;
    text-decoration: none;
    font-size: 14px;
    transition: all 0.2s ease;
    z-index: 100;
  }
  
  .edit-button:hover {
    background-color: rgba(255, 246, 160, 0.3);
  }
</style>
