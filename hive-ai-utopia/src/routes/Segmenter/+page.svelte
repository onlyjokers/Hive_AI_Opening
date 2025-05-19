<script lang="ts">
  import { onMount } from 'svelte';

  interface TimeSegment {
    id: string;
    time: string; // 格式为 "mm:ss" 或 "hh:mm:ss"
    seconds: number; // 转换后的总秒数
    startPosition?: number; // 在总时间线中的开始位置（秒）
    endPosition?: number; // 在总时间线中的结束位置（秒）
    startTime?: string; // 开始时间的字符串表示
    endTime?: string; // 结束时间的字符串表示
  }

  let segments: TimeSegment[] = [{ id: '', time: '', seconds: 0 }];
  let totalTime = '01:00:00'; // 默认总时间为1小时
  let fairnessLevel = 50; // 公平程度，默认50%
  let allowRepeat = false; // 是否允许重复视频
  let outputSegments: TimeSegment[] = [];
  
  // 颜色映射缓存
  const colorCache: Record<string, string> = {};
  
  // 为ID生成颜色
  function getColorForId(id: string): string {
    if (!id) return '#999999'; // 默认灰色
    
    // 如果已经有缓存的颜色，直接返回
    if (colorCache[id]) return colorCache[id];
    
    // 根据ID生成一致的颜色（基于ID的哈希值）
    let hash = 0;
    for (let i = 0; i < id.length; i++) {
      hash = ((hash << 5) - hash) + id.charCodeAt(i);
      hash = hash & hash; // Convert to 32bit integer
    }
    
    // 生成明亮的颜色，避免太暗的颜色
    const hue = Math.abs(hash) % 360;
    const saturation = 65 + (Math.abs(hash) % 20); // 65%-85%
    const lightness = 55 + (Math.abs(hash) % 10); // 55%-65%
    
    const color = `hsl(${hue}, ${saturation}%, ${lightness}%)`;
    colorCache[id] = color;
    
    return color;
  }

  // 转换时间字符串为秒数
  function timeToSeconds(time: string): number {
    if (!time) return 0;
    
    const parts = time.split(':').map(Number);
    if (parts.length === 2) {
      // 分:秒格式
      return parts[0] * 60 + parts[1];
    } else if (parts.length === 3) {
      // 时:分:秒格式
      return parts[0] * 3600 + parts[1] * 60 + parts[2];
    }
    return 0;
  }

  // 将秒数转换为时间字符串 (mm:ss 或 hh:mm:ss)
  function secondsToTime(seconds: number): string {
    const hours = Math.floor(seconds / 3600);
    const minutes = Math.floor((seconds % 3600) / 60);
    const secs = Math.floor(seconds % 60);
    
    if (hours > 0) {
      return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
    } else {
      return `${minutes.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
    }
  }

  // 添加新的时间段输入框
  function addSegment() {
    segments = [...segments, { id: '', time: '', seconds: 0 }];
  }

  // 删除时间段
  function removeSegment(index: number) {
    if (segments.length > 1) {
      segments = segments.filter((_, i) => i !== index);
    }
  }

  // 更新时间段的秒数
  function updateSeconds(index: number) {
    segments[index].seconds = timeToSeconds(segments[index].time);
    segments = [...segments]; // 触发更新
  }

  // 公平分配时间段
  function distributeSegments() {
    // 首先确保所有段都有时间
    const validSegments = segments.filter(s => s.time && s.seconds > 0);
    if (validSegments.length === 0) {
      alert('请至少添加一个有效的时间段！');
      return;
    }

    // 计算总可用时间（秒）
    const totalAvailableSeconds = timeToSeconds(totalTime);
    if (totalAvailableSeconds <= 0) {
      alert('请输入有效的总时间！');
      return;
    }

    // 计算所有段的总时间
    const segmentsTotalTime = validSegments.reduce((sum, segment) => sum + segment.seconds, 0);
    
    // 如果所有段的总时间超过可用时间，则无法分配
    if (segmentsTotalTime > totalAvailableSeconds) {
      alert('所有时间段总和超过了总可用时间！');
      return;
    }

    // 分析有多少不同的ID
    const uniqueIds = new Set(validSegments.map(s => s.id || 'unknown'));
    
    // 如果只有一种ID，无法保证前后不是同一ID
    if (uniqueIds.size === 1 && validSegments.length > 1) {
      alert('只有一种视频ID，无法满足前后不是同一ID的条件！至少需要两种不同的ID。');
      return;
    }

    // 创建最终的分配结果
    let finalSegments: TimeSegment[] = [];

    // 如果允许重复，且总时间超过了视频总长度
    if (allowRepeat && totalAvailableSeconds > segmentsTotalTime) {
      // 将视频按ID分组
      const segmentsByIdMap: Record<string, TimeSegment[]> = {};
      validSegments.forEach(segment => {
        const id = segment.id || 'unknown';
        if (!segmentsByIdMap[id]) {
          segmentsByIdMap[id] = [];
        }
        segmentsByIdMap[id].push({...segment});
      });
      
      const idsArray = Object.keys(segmentsByIdMap);
      
      // 为了实现公平，创建一个队列，包含所有ID，每次循环都会轮流使用不同ID
      let idQueue: string[] = [];
      
      // 根据公平度决定是否打乱ID顺序
      if (fairnessLevel > 50) {
        // 高公平度：随机排序ID
        idQueue = [...idsArray].sort(() => Math.random() - 0.5);
      } else {
        // 低公平度：保持原始ID顺序
        idQueue = [...idsArray];
      }
      
      // 记录上一次使用的ID，确保不连续使用相同ID
      let lastUsedId: string | null = null;
      
      // 持续添加片段直到填满总时间
      let remainingTime = totalAvailableSeconds;
      let iterationLimit = 1000; // 防止无限循环
      
      while (remainingTime > 0 && iterationLimit > 0) {
        iterationLimit--;
        
        // 找到可以使用的ID（与上一个不同）
        const availableIds = idQueue.filter(id => 
          id !== lastUsedId && // 不能与上一个相同
          segmentsByIdMap[id].length > 0 && // 该ID还有可用片段
          segmentsByIdMap[id][0].seconds <= remainingTime // 片段长度不超过剩余时间
        );
        
        if (availableIds.length === 0) {
          // 如果没有合适的ID可用，需要重新填充片段
          for (const id of idsArray) {
            // 只有当原始片段集非空时才重新填充
            if (validSegments.filter(s => (s.id || 'unknown') === id).length > 0) {
              segmentsByIdMap[id] = validSegments
                .filter(s => (s.id || 'unknown') === id)
                .map(s => ({...s}));
            }
          }
          
          // 再次检查是否有可用ID
          const refreshedAvailableIds = idQueue.filter(id => 
            id !== lastUsedId && 
            segmentsByIdMap[id].length > 0 &&
            segmentsByIdMap[id][0].seconds <= remainingTime
          );
          
          // 如果依然没有可用ID，或者剩余时间太少，则退出循环
          if (refreshedAvailableIds.length === 0) {
            break;
          }
          
          // 使用新的可用ID列表
          const selectedIdIndex = Math.floor(Math.random() * refreshedAvailableIds.length);
          const selectedId = refreshedAvailableIds[selectedIdIndex];
          
          // 获取并添加片段
          const segment = segmentsByIdMap[selectedId].shift()!;
          finalSegments.push({...segment});
          
          // 更新状态
          remainingTime -= segment.seconds;
          lastUsedId = selectedId;
        } else {
          // 随机选择一个可用ID
          const selectedIdIndex = Math.floor(Math.random() * availableIds.length);
          const selectedId = availableIds[selectedIdIndex];
          
          // 获取并添加片段
          const segment = segmentsByIdMap[selectedId].shift()!;
          finalSegments.push({...segment});
          
          // 更新状态
          remainingTime -= segment.seconds;
          lastUsedId = selectedId;
        }
      }
    } else {
      // 不允许重复，使用原有片段
      
      // 按ID对片段进行分组
      const segmentsByIdMap: Record<string, TimeSegment[]> = {};
      validSegments.forEach(segment => {
        const id = segment.id || 'unknown';
        if (!segmentsByIdMap[id]) {
          segmentsByIdMap[id] = [];
        }
        segmentsByIdMap[id].push({...segment});
      });
      
      const idsArray = Object.keys(segmentsByIdMap);
      
      // 记录最后使用的ID
      let lastUsedId: string | null = null;
      
      // 创建结果数组
      let iterationLimit = 1000; // 防止无限循环
      
      // 持续添加片段直到使用完所有片段
      while (Object.values(segmentsByIdMap).some(group => group.length > 0) && iterationLimit > 0) {
        iterationLimit--;
        
        // 找到所有还有片段的ID
        const availableIds = idsArray.filter(id => 
          segmentsByIdMap[id].length > 0 && // 该ID还有可用片段
          id !== lastUsedId // 不能与上一个相同
        );
        
        if (availableIds.length === 0) {
          // 如果没有不同于上一个的ID，但还有片段，则必须使用相同ID
          const anyAvailableIds = idsArray.filter(id => segmentsByIdMap[id].length > 0);
          
          if (anyAvailableIds.length === 0) {
            // 已经没有可用片段
            break;
          }
          
          // 随机选择一个ID（即使与上一个相同）
          const selectedIdIndex = Math.floor(Math.random() * anyAvailableIds.length);
          const selectedId = anyAvailableIds[selectedIdIndex];
          
          // 获取并添加片段
          const segment = segmentsByIdMap[selectedId].shift()!;
          finalSegments.push({...segment});
          
          // 更新最后使用的ID
          lastUsedId = selectedId;
        } else {
          // 根据公平度决定ID选择策略
          let selectedId: string;
          
          if (fairnessLevel > 70) {
            // 高公平度：随机选择不同ID
            const randomIndex = Math.floor(Math.random() * availableIds.length);
            selectedId = availableIds[randomIndex];
          } else if (fairnessLevel > 30) {
            // 中等公平度：优先选择数量多的ID组
            availableIds.sort((a, b) => 
              segmentsByIdMap[b].length - segmentsByIdMap[a].length
            );
            selectedId = availableIds[0];
          } else {
            // 低公平度：按原始顺序选择
            selectedId = availableIds[0];
          }
          
          // 获取并添加片段
          const segment = segmentsByIdMap[selectedId].shift()!;
          finalSegments.push({...segment});
          
          // 更新最后使用的ID
          lastUsedId = selectedId;
        }
      }
    }
    
    // 计算时间线位置信息
    let currentPosition = 0;
    const output = finalSegments.map(segment => {
      const startPosition = currentPosition;
      currentPosition += segment.seconds;
      
      return {
        ...segment,
        startPosition,
        endPosition: currentPosition,
        startTime: secondsToTime(startPosition),
        endTime: secondsToTime(currentPosition)
      };
    });
    
    console.log('Final output segments:', output);
    
    // 验证结果中是否有连续的相同ID
    let hasConsecutiveSameId = false;
    for (let i = 1; i < output.length; i++) {
      if (output[i].id === output[i-1].id) {
        hasConsecutiveSameId = true;
        console.warn(`发现连续相同ID: 位置 ${i-1} 和 ${i}, ID: ${output[i].id}`);
      }
    }
    
    if (hasConsecutiveSameId) {
      console.warn('警告：结果中存在连续相同ID的情况');
    }
    
    // 分配输出
    outputSegments = output;
  }
</script>

<svelte:head>
  <title>时间段公平分配器</title>
</svelte:head>

<div class="segmenter-container">
  <h1>时间段公平分配器</h1>

  <div class="settings-container">
    <div class="total-time-container">
      <label for="total-time">总时间:</label>
      <input 
        id="total-time" 
        type="text"
        bind:value={totalTime}
        placeholder="hh:mm:ss"
      />
    </div>

    <div class="fairness-slider-container">
      <label for="fairness-slider">公平程度: {fairnessLevel}%</label>
      <input 
        id="fairness-slider" 
        type="range" 
        min="0" 
        max="100" 
        bind:value={fairnessLevel}
      />
      <span class="slider-label-left">原始顺序</span>
      <span class="slider-label-right">最大分散</span>
    </div>
    
    <div class="allow-repeat-container">
      <label for="allow-repeat" class="checkbox-label">
        <input 
          id="allow-repeat" 
          type="checkbox" 
          bind:checked={allowRepeat}
        />
        允许重复视频（填充整个时间段）
      </label>
      <div class="option-description">
        启用后，当总时间超过所有视频总长度时，会重复使用视频以填满时间段（视频前后不会是同一ID）
      </div>
    </div>
  </div>

  <div class="segments-container">
    <h2>时间段输入</h2>
    
    {#each segments as segment, i}
      <div class="segment-input">
        <input 
          bind:value={segment.id}
          placeholder="视频ID"
          class="segment-id"
        />
        <input 
          bind:value={segment.time}
          placeholder="mm:ss 或 hh:mm:ss"
          class="segment-time"
          on:blur={() => updateSeconds(i)}
        />
        <button class="remove-button" on:click={() => removeSegment(i)}>删除</button>
      </div>
    {/each}
    
    <div class="buttons-container">
      <button class="add-button" on:click={addSegment}>添加时间段</button>
      <button class="distribute-button" on:click={distributeSegments}>计算公平分配</button>
    </div>
  </div>

  {#if outputSegments.length > 0}
    <div class="output-container">
      <h2>分配结果</h2>
      
      <!-- 时间线视图 -->
      <div class="timeline-container">
        <div class="timeline-header">
          <span>0:00</span>
          <span>{totalTime}</span>
        </div>
        <div class="timeline">
          {#each outputSegments as segment, i}
            <div 
              class="timeline-segment" 
              style="
                left: {((segment.startPosition || 0) / timeToSeconds(totalTime)) * 100}%; 
                width: {(segment.seconds / timeToSeconds(totalTime)) * 100}%;
                background-color: {getColorForId(segment.id)};
              "
              title="ID: {segment.id || '未指定'} | 时长: {segment.time} | 开始: {segment.startTime || '0:00'} | 结束: {segment.endTime || segment.time}"
            >
              <div class="segment-label">{segment.id || '未指定'}</div>
            </div>
          {/each}
        </div>
        <div class="timeline-markers">
          {#each Array(5) as _, i}
            <div class="timeline-marker" style="left: {i * 25}%">
              <div class="marker-line"></div>
              <div class="marker-label">
                {secondsToTime(Math.floor(timeToSeconds(totalTime) * (i / 4)))}
              </div>
            </div>
          {/each}
        </div>
      </div>
      
      <!-- 详细列表 -->
      <div class="output-list">
        {#each outputSegments as segment, i}
          <div class="output-item" style="border-left: 4px solid {getColorForId(segment.id)}">
            <span class="output-number">{i + 1}.</span>
            <span class="output-id">ID: {segment.id || '未指定'}</span>
            <span class="output-time">时长: {segment.time}</span>
            <span class="output-range">位置: {segment.startTime || '0:00'} - {segment.endTime || segment.time}</span>
          </div>
        {/each}
      </div>
    </div>
  {/if}
</div>

<style>
  .segmenter-container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    font-family: Arial, sans-serif;
  }
  
  h1 {
    text-align: center;
    color: #333;
    margin-bottom: 30px;
  }
  
  .settings-container {
    display: flex;
    flex-direction: column;
    gap: 20px;
    background-color: #f5f5f5;
    padding: 20px;
    border-radius: 8px;
    margin-bottom: 30px;
  }
  
  .total-time-container {
    display: flex;
    align-items: center;
    gap: 10px;
  }
  
  .total-time-container input {
    padding: 8px;
    border: 1px solid #ddd;
    border-radius: 4px;
    width: 120px;
  }
  
  .fairness-slider-container {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }
  
  .fairness-slider-container input {
    width: 100%;
  }
  
  .slider-label-left, .slider-label-right {
    font-size: 0.8em;
    color: #666;
  }
  
  .slider-label-left {
    align-self: flex-start;
  }
  
  .slider-label-right {
    align-self: flex-end;
  }
  
  .allow-repeat-container {
    margin-top: 10px;
    padding-top: 10px;
    border-top: 1px solid #ddd;
  }
  
  .checkbox-label {
    display: flex;
    align-items: center;
    gap: 8px;
    cursor: pointer;
    font-weight: bold;
  }
  
  .option-description {
    margin-top: 5px;
    font-size: 0.8em;
    color: #666;
    margin-left: 24px;
  }
  
  .segments-container {
    margin-bottom: 30px;
  }
  
  .segment-input {
    display: flex;
    gap: 10px;
    margin-bottom: 10px;
  }
  
  .segment-id, .segment-time {
    padding: 8px;
    border: 1px solid #ddd;
    border-radius: 4px;
  }
  
  .segment-id {
    flex: 1;
  }
  
  .segment-time {
    width: 150px;
  }
  
  .remove-button {
    background-color: #ff5555;
    color: white;
    border: none;
    border-radius: 4px;
    padding: 8px 12px;
    cursor: pointer;
  }
  
  .buttons-container {
    display: flex;
    gap: 10px;
    margin-top: 20px;
  }
  
  .add-button, .distribute-button {
    padding: 10px 15px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-weight: bold;
  }
  
  .add-button {
    background-color: #4caf50;
    color: white;
  }
  
  .distribute-button {
    background-color: #2196f3;
    color: white;
    margin-left: auto;
  }
  
  .output-container {
    background-color: #f9f9f9;
    padding: 20px;
    border-radius: 8px;
  }
  
  /* 时间线相关样式 */
  .timeline-container {
    margin: 30px 0;
    position: relative;
  }
  
  .timeline-header {
    display: flex;
    justify-content: space-between;
    margin-bottom: 5px;
    color: #666;
    font-size: 0.9em;
  }
  
  .timeline {
    height: 50px;
    background-color: #eee;
    border-radius: 4px;
    position: relative;
    overflow: hidden;
  }
  
  .timeline-segment {
    position: absolute;
    height: 100%;
    top: 0;
    border-radius: 2px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
    transition: all 0.2s ease;
    overflow: hidden;
    cursor: pointer;
  }
  
  .timeline-segment:hover {
    transform: scaleY(1.05);
    z-index: 10;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
  }
  
  .segment-label {
    position: absolute;
    bottom: 2px;
    left: 2px;
    right: 2px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    font-size: 0.75em;
    color: white;
    text-shadow: 0 0 2px rgba(0, 0, 0, 0.5);
    text-align: center;
  }
  
  .timeline-markers {
    position: relative;
    height: 20px;
    margin-top: 5px;
  }
  
  .timeline-marker {
    position: absolute;
    width: 1px;
  }
  
  .marker-line {
    width: 1px;
    height: 8px;
    background-color: #aaa;
  }
  
  .marker-label {
    font-size: 0.75em;
    color: #666;
    transform: translateX(-50%);
    margin-top: 2px;
  }
  
  .output-list {
    display: flex;
    flex-direction: column;
    gap: 10px;
    margin-top: 20px;
  }
  
  .output-item {
    display: flex;
    align-items: center;
    gap: 15px;
    padding: 10px 15px;
    background-color: white;
    border-radius: 4px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  }
  
  .output-number {
    font-weight: bold;
    min-width: 25px;
  }
  
  .output-id {
    flex: 1;
  }
  
  .output-time {
    font-weight: bold;
    color: #333;
    margin-right: 10px;
  }
  
  .output-range {
    color: #666;
    font-size: 0.9em;
  }
</style>