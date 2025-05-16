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
  
  // 自定义输入文本
  let inputText = '';
  
  // 从本地存储中加载或使用默认值
  onMount(() => {
    const savedTexts = localStorage.getItem('hive-ai-texts');
    if (savedTexts) {
      inputText = savedTexts;
    } else {
      // 默认使用原始文本，以逗号分隔
      inputText = defaultBoxes.map(box => box.text).join(',');
    }
  });
  
  // 处理构建按钮点击事件
  function handleBuild() {
    // 保存输入的文本
    localStorage.setItem('hive-ai-texts', inputText);
    
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
    margin-bottom: 2rem;
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
</style>