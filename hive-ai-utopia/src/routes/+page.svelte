<script lang="ts">
  import DraggableBox from './DraggableBox.svelte';
  import { onMount, onDestroy } from 'svelte';
  import './styles.css';
  import { gsap } from 'gsap';
  import { fade } from 'svelte/transition';
  import { goto } from '$app/navigation';

  let windowWidth = 0;
  let windowHeight = 0;
  let animationDone = false;
  
  // 是否使用固定尺寸（832H×1728W）
  let useFixedSize = false;

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
  function handleBoxMove(event: CustomEvent) {
    const { id, x, y, width, height, rotation } = event.detail;
    boxData[id] = { x, y, width, height, rotation };
  }

  // 计算多边形凸包（Monotone Chain算法）
  function convexHull(points: { x: number; y: number }[]) {
    if (points.length <= 1) return points;
    const sorted = points.slice().sort((a, b) => a.x === b.x ? a.y - b.y : a.x - b.x);
    const cross = (o: {x: number; y: number}, a: {x: number; y: number}, b: {x: number; y: number}) => (a.x - o.x) * (b.y - o.y) - (a.y - o.y) * (b.x - o.x);
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
        // 在固定尺寸模式下，需要转换坐标系统
        let baseX = d.x;
        let baseY = d.y;
        
        if (useFixedSize && typeof document !== 'undefined') {
          // 在固定尺寸模式下，需要将viewport坐标转换为SVG坐标系
          const triangleContainer = document.querySelector('.triangle-container');
          const svg = document.querySelector('.triangle-svg');
          
          if (triangleContainer && svg) {
            const containerRect = triangleContainer.getBoundingClientRect();
            const svgRect = svg.getBoundingClientRect();
            
            // 将DraggableBox的viewport坐标转换为相对于SVG的坐标
            baseX = d.x - svgRect.left;
            baseY = d.y - svgRect.top;
            
            console.log('Fixed size coordinate conversion:', {
              boxId: b.id,
              original: { x: d.x, y: d.y },
              containerRect: { left: containerRect.left, top: containerRect.top },
              svgRect: { left: svgRect.left, top: svgRect.top },
              converted: { x: baseX, y: baseY }
            });
          } else {
            // 如果找不到容器元素，回退到直接使用坐标
            baseX = d.x;
            baseY = d.y;
            console.log('Fixed size mode - fallback to direct coordinates');
          }
        }
        
        // 计算四个角的坐标
        const rad = d.rotation * Math.PI / 180;
        const cx = baseX + d.width / 2;
        const cy = baseY + d.height / 2;
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
    // 如果是测试模式，立即应用fixed-size-mode
    if (useFixedSize) {
      document.body.classList.add('fixed-size-mode');
      console.log('Test mode: Applied fixed-size-mode class');
    }
    
    // 添加调试信息：代码背景元素状态
    console.log('Code background element:', codeDiv);
    console.log('Lines container element:', linesContainer);
    console.log('Static lines count:', staticLines.length);
    console.log('Static lines sample:', staticLines.slice(0, 5));
    
    // 延迟检查元素可见性
    setTimeout(() => {
      if (codeDiv) {
        const computedStyle = window.getComputedStyle(codeDiv);
        console.log('Code background computed styles:', {
          display: computedStyle.display,
          position: computedStyle.position,
          zIndex: computedStyle.zIndex,
          opacity: computedStyle.opacity,
          visibility: computedStyle.visibility
        });
      }
    }, 100);
    
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
      console.log('shouldRecord from localStorage:', shouldRecord);
      
      if (shouldRecord === 'true') {
        console.log('启用固定尺寸模式');
        
        // 立即移除标记，防止页面刷新时重新录制
        localStorage.removeItem('hive-ai-should-record');
        
        // 设置使用固定尺寸
        useFixedSize = true;
        console.log('useFixedSize set to:', useFixedSize);
        
        // 添加body class
        document.body.classList.add('fixed-size-mode');
        console.log('Added fixed-size-mode class to body');
        
        // 延迟一小段时间后开始录制，等待页面完全渲染
        setTimeout(() => {
          startScreenRecording();
        }, 500);
      } else {
        console.log('未启用录制模式');
      }
    } catch (e) {
      console.error('加载自定义盒子数据时出错:', e);
    }
    
    windowWidth = window.innerWidth;
    windowHeight = window.innerHeight;
    if (codeDiv && linesContainer) {
      const viewH = codeDiv.clientHeight;
      const fullH = linesContainer.scrollHeight;
      
      // 根据固定尺寸模式调整动画参数
      const animationDuration = useFixedSize ? 4 : 8; // 固定尺寸模式下缩短动画时间
      const boxDelay = useFixedSize ? 750 : 1500; // 固定尺寸模式下缩短盒子出现延迟
      
      console.log('Animation settings:', { 
        useFixedSize, 
        animationDuration, 
        boxDelay, 
        viewH, 
        fullH 
      });
      
      gsap.fromTo(
        linesContainer,
        { y: viewH },
        {
          // 滚动到内容底部与视口底对齐时停止：y = viewH - fullH
          y: (viewH/1 - (boxes.length * 35)) - fullH,
          duration: animationDuration,
          ease: 'none',
          onUpdate: () => {
            // 检测视口区域
            const viewportRect = codeDiv.getBoundingClientRect();
            
            // 遍历检测所有盒子类型行
            staticLines.forEach((line, idx) => {
              if (
                line.type === 'box' &&
                line.id &&                               // 确保有id
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
                    setTimeout(() => {
                      const showBox = () => {
                        const pos = lineEl.getBoundingClientRect();
                        if (line.id) {
                          boxStates[line.id] = {
                            visible: true,
                            initialX: pos.left,
                            initialY: pos.top
                          };
                        }
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
                    }, boxDelay);
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
        const finalYs = [20, 100, 150, 200, 250, 300, 350, 400, 450, 500];

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
      // 获取屏幕共享
      const stream = await navigator.mediaDevices.getDisplayMedia({
        video: {
          width: { ideal: 1728 },
          height: { ideal: 832 },
          frameRate: { ideal: 30 }
        },
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

  // 重新启动动画的函数
  function restartAnimation() {
    if (!codeDiv || !linesContainer) return;
    
    // 重置所有盒子状态
    initBoxStates();
    
    // 停止当前动画
    gsap.killTweensOf(linesContainer);
    
    // 重新计算视口和动画参数
    const viewH = codeDiv.clientHeight;
    const fullH = linesContainer.scrollHeight;
    
    // 根据固定尺寸模式调整动画参数
    const animationDuration = useFixedSize ? 4 : 8;
    const boxDelay = useFixedSize ? 750 : 1500;
    
    console.log('Restarting animation with settings:', { 
      useFixedSize, 
      animationDuration, 
      boxDelay, 
      viewH, 
      fullH 
    });
    
    // 重新启动动画
    gsap.fromTo(
      linesContainer,
      { y: viewH },
      {
        y: (viewH/1 - (boxes.length * 35)) - fullH,
        duration: animationDuration,
        ease: 'none',
        onUpdate: () => {
          // 检测视口区域
          const viewportRect = codeDiv.getBoundingClientRect();
          
          // 遍历检测所有盒子类型行
          staticLines.forEach((line, idx) => {
            if (
              line.type === 'box' &&
              line.id &&
              boxStates[line.id] &&
              !boxStates[line.id].visible &&
              !boxScheduled[line.id]
            ) {
              const lineEl = lineRefs[idx];
              if (lineEl) {
                const lineRect = lineEl.getBoundingClientRect();
                if (lineRect.top >= viewportRect.top && lineRect.bottom <= viewportRect.bottom) {
                  boxScheduled[line.id] = true;
                  setTimeout(() => {
                    const showBox = () => {
                      const pos = lineEl.getBoundingClientRect();
                      if (line.id) {
                        boxStates[line.id] = {
                          visible: true,
                          initialX: pos.left,
                          initialY: pos.top
                        };
                      }
                    };
                    if (line.id === lastId) {
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
                  }, boxDelay);
                }
              }
            }
          });
        }
      }
    );
  }

  // 监听useFixedSize变化，重新启动动画
  $: if (typeof useFixedSize !== 'undefined') {
    // 当useFixedSize变化时，更新body class
    if (useFixedSize) {
      document.body.classList.add('fixed-size-mode');
    } else {
      document.body.classList.remove('fixed-size-mode');
    }
    
    // 延迟重新启动动画，等待DOM更新
    setTimeout(() => {
      restartAnimation();
    }, 100);
  }

  // 组件销毁时清理动画和定时器
  onDestroy(() => {
    // 停止所有GSAP动画
    gsap.killTweensOf("*");
    
    // 清除所有定时器（如果有全局引用的话）
    // 注意：setTimeout 创建的定时器在组件销毁时会自动清理
    
    // 移除body类
    if (typeof document !== 'undefined') {
      document.body.classList.remove('fixed-size-mode');
    }
  });
</script>

<svelte:window bind:innerWidth={windowWidth} bind:innerHeight={windowHeight} />

<!-- 外层容器，根据useFixedSize控制尺寸 -->
<div 
  class="app-container" 
  class:fixed-size={useFixedSize}
  style={useFixedSize ? 'width: 1728px; height: 832px; border: 3px solid #e0d782; margin: 0 auto; overflow: hidden; position: relative; background-color: #000;' : ''}
>
  <!-- 代码背景 -->
  <div bind:this={codeDiv} class="code-background">
    <!-- 测试静态文本 -->
    <div style="color: #ff0000; font-size: 20px; margin: 20px;">测试代码背景显示</div>
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
    
    <!-- 调试信息 -->
    <div class="debug-info">
      <p>useFixedSize: {useFixedSize}</p>
      <p>容器类: {useFixedSize ? 'fixed-size' : 'normal'}</p>
      {#if useFixedSize}
        <p>固定尺寸: 1728×832</p>
        <p>录制模式: 开启</p>
        <p>动画时长: 4秒</p>
        <p>盒子延迟: 750ms</p>
      {:else}
        <p>正常模式</p>
        <p>动画时长: 8秒</p>
        <p>盒子延迟: 1500ms</p>
      {/if}
      <p>可见盒子数: {boxes.filter(b => boxStates[b.id]?.visible).length}/{boxes.length}</p>
      <p>凸包路径: {hullPath ? '已生成' : '未生成'}</p>
      <button on:click={() => useFixedSize = !useFixedSize}>
        切换模式 (会重启动画)
      </button>
      <p style="font-size: 10px; margin-top: 5px;">
        固定尺寸模式下动画加速，盒子出现更快
      </p>
    </div>
  </main>
</div>

<style lang="css">
  /* 样式已移至styles.css */

  /* 外层容器 */
  .app-container {
    width: 100vw;
    height: 100vh;
    overflow: hidden;
    position: relative;
    background-color: #000;
  }
  
  /* 固定尺寸模式：832H×1728W */
  :global(.app-container.fixed-size) {
    width: 1728px !important;
    height: 832px !important;
    max-width: 1728px !important;
    max-height: 832px !important;
    min-width: 1728px !important;
    min-height: 832px !important;
    margin: 0 auto !important;
    border: 3px solid #e0d782 !important;
    box-sizing: border-box !important;
    background-color: #000 !important;
    position: relative !important;
    overflow: hidden !important;
    display: block !important;
  }
  
  /* 固定尺寸模式下的代码背景样式调整 */
  .app-container.fixed-size :global(.code-background) {
    position: absolute;
    width: 1728px;
    height: 832px;
    z-index: 1 !important;
    opacity: 0.4 !important;
  }

  main {
    width: 100%;
    min-height: 100vh;
    position: relative;
    padding: 20px;
    box-sizing: border-box;
  }
  
  /* 固定尺寸模式下的main样式调整 */
  .app-container.fixed-size main {
    min-height: 832px;
    height: 832px;
  }

  .triangle-container {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 20;
    pointer-events: none;
  }

  .triangle-svg {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 21;
    pointer-events: none;
  }
  
  /* 确保DraggableBox可以被拖动 */
  .triangle-container :global(.draggable-box) {
    pointer-events: auto;
    z-index: 22;
  }
  
  /* 调试信息样式 */
  .debug-info {
    position: fixed;
    top: 10px;
    right: 10px;
    background-color: rgba(224, 215, 130, 0.9);
    color: #000;
    padding: 10px;
    border-radius: 5px;
    font-size: 12px;
    z-index: 1000;
  }
</style>
