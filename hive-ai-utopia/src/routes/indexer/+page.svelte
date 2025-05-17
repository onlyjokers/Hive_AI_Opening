<script lang="ts">
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  
  // 定义默认盒子配置
  const defaultBoxes = [
    { id: 'works', text: '██落在夜空生成以前', rotation: 0 },
    { id: 'name', text: '蔡宇潇  Justin Bortnick', rotation: 0 },
    { id: 'type', text: '文学游戏', rotation: 0 },
    { id: 'time', text: '2024-2025', rotation: 0 },
    { id: 'duration', text: '时长不限', rotation: 0 }
  ];
  
  // 默认代码行
  const defaultCodeLines = [
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
  
  // 自定义输入文本
  let inputText = '';
  
  // 自定义代码行文本
  let codeText = '';
  
  // 是否显示代码编辑器
  let showCodeEditor = false;
  
  // 是否录制视频的选项
  let shouldRecord = false;
  
  // 视频录制相关变量
  let mediaRecorder: MediaRecorder | null = null;
  let recordedChunks: BlobPart[] = [];
  
  // 从本地存储中加载或使用默认值
  onMount(() => {
    const savedTexts = localStorage.getItem('hive-ai-texts');
    if (savedTexts) {
      inputText = savedTexts;
    } else {
      // 默认使用原始文本，以逗号分隔
      inputText = defaultBoxes.map(box => box.text).join(',');
    }
    
    const savedCode = localStorage.getItem('hive-ai-code');
    if (savedCode) {
      codeText = savedCode;
    } else {
      codeText = defaultCodeLines.join('\n');
    }
  });
  
  // 启动视频录制
  async function startRecording() {
    try {
      // 设置要录制的窗口大小
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
      // 出错时仍然跳转到主页
      goto('/');
    }
  }
  
  // 处理构建按钮点击事件
  async function handleBuild() {
    // 保存输入的文本
    localStorage.setItem('hive-ai-texts', inputText);
    localStorage.setItem('hive-ai-code', codeText);
    
    // 解析输入文本并保存为自定义盒子数据
    const texts = inputText.split(',').map(text => text.trim()).filter(text => text);
    
    if (texts.length > 0) {
      // 创建新的盒子数组
      const newBoxes = defaultBoxes.map((box, index) => {
        // 如果有对应索引的文本，则使用自定义文本；否则保留默认文本
        return {
          id: box.id,
          text: index < texts.length ? texts[index] : box.text,
          rotation: box.rotation
        };
      });
      
      // 保存新的盒子配置
      localStorage.setItem('hive-ai-boxes', JSON.stringify(newBoxes));
    } else {
      // 如果输入为空，则使用默认值
      localStorage.removeItem('hive-ai-boxes');
    }
    
    // 判断是否需要录制
    if (shouldRecord) {
      // 记录录制选项，方便主页面使用
      localStorage.setItem('hive-ai-should-record', 'true');
      
      // 启动录制（会包括跳转后的页面）
      await startRecording();
    } else {
      localStorage.removeItem('hive-ai-should-record');
    }
    
    // 跳转到主页面
    goto('/');
  }
</script>

<main>
  <h1>蜂巢AI理想国 文本编辑器</h1>
  
  <div class="instruction">
    <p>在下方输入文本，用逗号(,)分隔多个文本。点击"构建"按钮跳转到主页面。</p>
    <p>如果不输入任何内容，将使用默认文本。</p>
  </div>
  
  <div class="input-container">
    <textarea 
      bind:value={inputText} 
      rows="6"
      placeholder="输入文本，用逗号分隔，例如：文本一,文本二,文本三"
    ></textarea>
  </div>
  
  <div class="option-container">
    <label class="record-option">
      <input type="checkbox" bind:checked={shouldRecord} />
      <span>生成时调整窗口尺寸为 832H×1728W 并录制 10 秒视频</span>
    </label>
    
    <label class="record-option" style="margin-top: 0.75rem;">
      <input type="checkbox" bind:checked={showCodeEditor} />
      <span>自定义代码内容</span>
    </label>
  </div>
  
  {#if showCodeEditor}
    <div class="code-editor-container">
      <h3>代码编辑器</h3>
      <p class="hint">自定义滚动背景中的代码内容，每行一条语句。</p>
      <textarea 
        bind:value={codeText} 
        rows="15"
        class="code-editor"
        placeholder="输入代码，每行一条语句"
      ></textarea>
      <div class="code-button-group">
        <button class="reset-button" on:click={() => codeText = defaultCodeLines.join('\n')}>
          重置为默认代码
        </button>
      </div>
    </div>
  {/if}
  
  <div class="button-group">
    <button class="reset-button" on:click={() => inputText = defaultBoxes.map(box => box.text).join(',')}>
      重置为默认内容
    </button>
    <button class="main-button" on:click={handleBuild}>构建</button>
  </div>
</main>

<style>
  main {
    max-width: 800px;
    margin: 0 auto;
    padding: 2rem;
    font-family: "SimHei", "黑体", "Microsoft YaHei", "微软雅黑", sans-serif;
    background-color: #111;
    color: #f0f0f0;
    min-height: 100vh;
    box-sizing: border-box;
  }
  
  h1 {
    color: #f0f0a0;
    text-align: center;
    margin-bottom: 2rem;
  }
  
  .instruction {
    background-color: rgba(255, 246, 160, 0.1);
    padding: 1rem;
    margin-bottom: 2rem;
    border-radius: 4px;
    border: 1px solid #e0d782;
  }
  
  .input-container {
    margin-bottom: 1.5rem;
  }
  
  .option-container {
    margin-bottom: 2rem;
  }
  
  .record-option {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    cursor: pointer;
    color: #e0d782;
    font-size: 0.9rem;
  }
  
  .record-option input[type="checkbox"] {
    appearance: none;
    -webkit-appearance: none;
    width: 18px;
    height: 18px;
    border: 1px solid #e0d782;
    border-radius: 3px;
    background-color: rgba(255, 246, 160, 0.1);
    display: grid;
    place-content: center;
    cursor: pointer;
  }
  
  .record-option input[type="checkbox"]::before {
    content: "";
    width: 10px;
    height: 10px;
    transform: scale(0);
    transition: 120ms transform ease-in-out;
    box-shadow: inset 1em 1em #e0d782;
    transform-origin: center;
    clip-path: polygon(14% 44%, 0 65%, 50% 100%, 100% 16%, 80% 0%, 43% 62%);
  }
  
  .record-option input[type="checkbox"]:checked::before {
    transform: scale(1);
  }
  
  .record-option input[type="checkbox"]:focus {
    outline: 2px solid rgba(255, 246, 160, 0.5);
    outline-offset: 2px;
  }
  
  textarea {
    width: 100%;
    padding: 1rem;
    background-color: #222;
    color: #fff;
    border: 1px solid #e0d782;
    border-radius: 4px;
    font-family: "SimHei", "黑体", "Microsoft YaHei", "微软雅黑", sans-serif;
    font-size: 1rem;
    resize: vertical;
    box-sizing: border-box;
  }
  
  .button-group {
    display: flex;
    justify-content: space-between;
    gap: 1rem;
    margin-top: 2rem;
  }
  
  button {
    padding: 0.75rem 1.5rem;
    font-family: inherit;
    font-size: 1rem;
    cursor: pointer;
    border-radius: 4px;
    border: none;
    transition: all 0.2s ease;
  }
  
  .reset-button {
    background-color: rgba(255, 246, 160, 0.1);
    color: #e0d782;
    border: 1px solid #e0d782;
  }
  
  .reset-button:hover {
    background-color: rgba(255, 246, 160, 0.2);
  }
  
  .main-button {
    background-color: #f0f0a0;
    color: #333;
    font-weight: bold;
  }
  
  .main-button:hover {
    background-color: #e0d782;
  }
  
  .code-editor-container {
    margin-bottom: 2rem;
    background-color: rgba(255, 246, 160, 0.05);
    padding: 1rem;
    border-radius: 4px;
    border: 1px solid rgba(224, 215, 130, 0.3);
  }
  
  .code-editor-container h3 {
    color: #e0d782;
    margin-top: 0;
    margin-bottom: 0.5rem;
  }
  
  .hint {
    font-size: 0.9rem;
    margin-bottom: 1rem;
    color: rgba(255, 246, 160, 0.7);
  }
  
  .code-editor {
    font-family: monospace;
    font-size: 0.9rem;
    background-color: #1a1a1a;
    border-color: rgba(224, 215, 130, 0.5);
  }
  
  .code-button-group {
    margin-top: 1rem;
    display: flex;
    justify-content: flex-end;
  }
</style>