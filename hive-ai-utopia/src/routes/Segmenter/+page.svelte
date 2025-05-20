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
    timeValues?: TimeInputValues; // 分解后的时间值
  }

  // 时间输入组件接口
  interface TimeInputValues {
    hours: number;
    minutes: number;
    seconds: number;
  }

  let segments: TimeSegment[] = [{ id: '', time: '', seconds: 0, timeValues: { hours: 0, minutes: 0, seconds: 0 } }];
  let totalTime = '01:00:00'; // 默认总时间为1小时
  let totalTimeValues: TimeInputValues = { hours: 1, minutes: 0, seconds: 0 };
  let fairnessLevel = 50; // 公平程度，默认50%
  let allowRepeat = false; // 是否允许重复视频
  let outputSegments: TimeSegment[] = [];
  
  // 在组件挂载时初始化时间值
  onMount(() => {
    updateTotalTimeValues();
    // 初始化所有时间段的时间值
    segments.forEach((segment, index) => {
      if (segment.time) {
        updateSegmentTimeValues(index);
      }
    });
  });

  // 根据总时间字符串更新时间值对象
  function updateTotalTimeValues() {
    const parts = totalTime.split(':').map(Number);
    if (parts.length === 3) {
      totalTimeValues = {
        hours: parts[0],
        minutes: parts[1],
        seconds: parts[2]
      };
    } else if (parts.length === 2) {
      totalTimeValues = {
        hours: 0,
        minutes: parts[0],
        seconds: parts[1]
      };
    }
  }

  // 根据时间值对象更新总时间字符串
  function updateTotalTimeString() {
    const h = Math.max(0, Math.min(99, totalTimeValues.hours || 0));
    const m = Math.max(0, Math.min(59, totalTimeValues.minutes || 0));
    const s = Math.max(0, Math.min(59, totalTimeValues.seconds || 0));
    
    totalTime = `${h.toString().padStart(2, '0')}:${m.toString().padStart(2, '0')}:${s.toString().padStart(2, '0')}`;
  }

  // 处理时间输入的变化
  function handleTimeInput(field: keyof TimeInputValues, value: number) {
    // 确保值在有效范围内
    if (field === 'hours') {
      totalTimeValues.hours = Math.max(0, Math.min(99, value));
    } else if (field === 'minutes' || field === 'seconds') {
      totalTimeValues[field] = Math.max(0, Math.min(59, value));
    }
    
    // 更新时间字符串
    updateTotalTimeString();
  }

  // 处理分钟或秒数增加，并处理进位
  function incrementTime(field: 'hours' | 'minutes' | 'seconds') {
    if (field === 'seconds') {
      totalTimeValues.seconds++;
      if (totalTimeValues.seconds > 59) {
        totalTimeValues.seconds = 0;
        totalTimeValues.minutes++;
      }
    } else if (field === 'minutes') {
      totalTimeValues.minutes++;
      if (totalTimeValues.minutes > 59) {
        totalTimeValues.minutes = 0;
        totalTimeValues.hours++;
      }
    } else if (field === 'hours') {
      totalTimeValues.hours = Math.min(99, totalTimeValues.hours + 1);
    }
    
    updateTotalTimeString();
  }

  // 处理分钟或秒数减少，并处理借位
  function decrementTime(field: 'hours' | 'minutes' | 'seconds') {
    if (field === 'seconds') {
      totalTimeValues.seconds--;
      if (totalTimeValues.seconds < 0) {
        totalTimeValues.seconds = 59;
        totalTimeValues.minutes--;
      }
    } else if (field === 'minutes') {
      totalTimeValues.minutes--;
      if (totalTimeValues.minutes < 0) {
        totalTimeValues.minutes = 59;
        totalTimeValues.hours--;
      }
    } else if (field === 'hours') {
      totalTimeValues.hours = Math.max(0, totalTimeValues.hours - 1);
    }
    
    // 确保时间不为负
    totalTimeValues.hours = Math.max(0, totalTimeValues.hours);
    totalTimeValues.minutes = Math.max(0, totalTimeValues.minutes);
    totalTimeValues.seconds = Math.max(0, totalTimeValues.seconds);
    
    updateTotalTimeString();
  }

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
    segments = [...segments, { 
      id: '', 
      time: '', 
      seconds: 0,
      timeValues: { hours: 0, minutes: 0, seconds: 0 }
    }];
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

  // 为时间段更新分解的时间值
  function updateSegmentTimeValues(index: number) {
    const segment = segments[index];
    const parts = segment.time.split(':').map(Number);
    
    if (parts.length === 3) {
      segment.timeValues = {
        hours: parts[0],
        minutes: parts[1],
        seconds: parts[2]
      };
    } else if (parts.length === 2) {
      segment.timeValues = {
        hours: 0,
        minutes: parts[0],
        seconds: parts[1]
      };
    } else {
      segment.timeValues = {
        hours: 0,
        minutes: 0,
        seconds: 0
      };
    }
    
    segments = [...segments]; // 触发更新
  }

  // 根据分解的时间值更新时间字符串
  function updateSegmentTimeString(index: number) {
    const segment = segments[index];
    if (!segment.timeValues) {
      segment.timeValues = { hours: 0, minutes: 0, seconds: 0 };
    }
    
    const h = Math.max(0, Math.min(99, segment.timeValues.hours || 0));
    const m = Math.max(0, Math.min(59, segment.timeValues.minutes || 0));
    const s = Math.max(0, Math.min(59, segment.timeValues.seconds || 0));
    
    if (h > 0) {
      segment.time = `${h.toString().padStart(2, '0')}:${m.toString().padStart(2, '0')}:${s.toString().padStart(2, '0')}`;
    } else {
      segment.time = `${m.toString().padStart(2, '0')}:${s.toString().padStart(2, '0')}`;
    }
    
    // 更新秒数
    segment.seconds = timeToSeconds(segment.time);
    segments = [...segments]; // 触发更新
  }

  // 处理时间段时间输入的变化
  function handleSegmentTimeInput(index: number, field: keyof TimeInputValues, value: number) {
    const segment = segments[index];
    if (!segment.timeValues) {
      segment.timeValues = { hours: 0, minutes: 0, seconds: 0 };
    }
    
    // 确保值在有效范围内
    if (field === 'hours') {
      segment.timeValues.hours = Math.max(0, Math.min(99, value));
    } else if (field === 'minutes' || field === 'seconds') {
      segment.timeValues[field] = Math.max(0, Math.min(59, value));
    }
    
    // 更新时间字符串
    updateSegmentTimeString(index);
  }

  // 处理分钟或秒数增加，并处理进位
  function incrementSegmentTime(index: number, field: 'hours' | 'minutes' | 'seconds') {
    const segment = segments[index];
    if (!segment.timeValues) {
      updateSegmentTimeValues(index);
    }
    
    if (field === 'seconds') {
      segment.timeValues!.seconds++;
      if (segment.timeValues!.seconds > 59) {
        segment.timeValues!.seconds = 0;
        segment.timeValues!.minutes++;
      }
    } else if (field === 'minutes') {
      segment.timeValues!.minutes++;
      if (segment.timeValues!.minutes > 59) {
        segment.timeValues!.minutes = 0;
        segment.timeValues!.hours++;
      }
    } else if (field === 'hours') {
      segment.timeValues!.hours = Math.min(99, segment.timeValues!.hours + 1);
    }
    
    updateSegmentTimeString(index);
  }

  // 处理分钟或秒数减少，并处理借位
  function decrementSegmentTime(index: number, field: 'hours' | 'minutes' | 'seconds') {
    const segment = segments[index];
    if (!segment.timeValues) {
      updateSegmentTimeValues(index);
    }
    
    if (field === 'seconds') {
      segment.timeValues!.seconds--;
      if (segment.timeValues!.seconds < 0) {
        segment.timeValues!.seconds = 59;
        segment.timeValues!.minutes--;
      }
    } else if (field === 'minutes') {
      segment.timeValues!.minutes--;
      if (segment.timeValues!.minutes < 0) {
        segment.timeValues!.minutes = 59;
        segment.timeValues!.hours--;
      }
    } else if (field === 'hours') {
      segment.timeValues!.hours = Math.max(0, segment.timeValues!.hours - 1);
    }
    
    // 确保时间不为负
    segment.timeValues!.hours = Math.max(0, segment.timeValues!.hours);
    segment.timeValues!.minutes = Math.max(0, segment.timeValues!.minutes);
    segment.timeValues!.seconds = Math.max(0, segment.timeValues!.seconds);
    
    updateSegmentTimeString(index);
  }

  // 智能解析时间输入
  function smartParseTime(input: string): string {
    // 已经是有效格式的不处理
    if (/^\d{1,2}:\d{2}(:\d{2})?$/.test(input)) {
      return input;
    }
    
    // 处理简单的数字（假设是分钟）
    if (/^\d+$/.test(input)) {
      const minutes = parseInt(input);
      if (minutes < 100) {
        return `${minutes.toString().padStart(2, '0')}:00`;
      } else {
        // 如果超过99分钟，转换为小时:分钟格式
        const hours = Math.floor(minutes / 60);
        const mins = minutes % 60;
        return `${hours.toString().padStart(2, '0')}:${mins.toString().padStart(2, '0')}:00`;
      }
    }
    
    // 处理分:秒格式，但没有前导零
    if (/^\d{1,2}:\d{1,2}$/.test(input)) {
      const parts = input.split(':').map(Number);
      return `${parts[0].toString().padStart(2, '0')}:${parts[1].toString().padStart(2, '0')}`;
    }
    
    // 处理小时:分:秒格式，但没有前导零
    if (/^\d{1,2}:\d{1,2}:\d{1,2}$/.test(input)) {
      const parts = input.split(':').map(Number);
      return `${parts[0].toString().padStart(2, '0')}:${parts[1].toString().padStart(2, '0')}:${parts[2].toString().padStart(2, '0')}`;
    }
    
    // 处理如 "1h30m"、"45m20s"、"1h20s" 格式
    const hourPattern = /(\d+)h/i;
    const minutePattern = /(\d+)m/i;
    const secondPattern = /(\d+)s/i;
    
    let hours = 0;
    let minutes = 0;
    let seconds = 0;
    
    const hourMatch = input.match(hourPattern);
    const minuteMatch = input.match(minutePattern);
    const secondMatch = input.match(secondPattern);
    
    if (hourMatch) hours = parseInt(hourMatch[1]);
    if (minuteMatch) minutes = parseInt(minuteMatch[1]);
    if (secondMatch) seconds = parseInt(secondMatch[1]);
    
    if (hours > 0 || minutes > 0 || seconds > 0) {
      // 处理进位
      minutes += Math.floor(seconds / 60);
      seconds %= 60;
      hours += Math.floor(minutes / 60);
      minutes %= 60;
      
      if (hours > 0) {
        return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
      } else {
        return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
      }
    }
    
    // 无法解析，返回原始输入
    return input;
  }
  
  // 处理时间段输入失焦事件
  function handleTimeSegmentBlur(index: number) {
    // 智能解析并格式化输入
    segments[index].time = smartParseTime(segments[index].time);
    // 更新秒数
    updateSeconds(index);
    // 更新分解的时间值
    updateSegmentTimeValues(index);
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
      
      // 首先确保每个唯一ID至少被使用一次
      let lastUsedId: string | null = null;
      
      // 复制一份原始片段用于确保至少使用一次
      const allUniqueIds = new Set(idsArray);
      const unusedIds = new Set(idsArray);
      
      // 第一阶段：确保每个唯一ID至少使用一次
      while (unusedIds.size > 0) {
        // 找到所有未使用的ID中与上一个不同的ID
        const availableIds = Array.from(unusedIds).filter(id => id !== lastUsedId);
        
        // 如果没有可用ID（只剩下一个ID且与上一个相同），则打破规则使用相同ID
        const selectedId = availableIds.length > 0 
          ? availableIds[Math.floor(Math.random() * availableIds.length)]
          : Array.from(unusedIds)[0];
        
        // 获取并添加片段
        const segment = segmentsByIdMap[selectedId].shift()!;
        finalSegments.push({...segment});
        
        // 标记ID已使用并更新最后使用的ID
        unusedIds.delete(selectedId);
        lastUsedId = selectedId;
        
        // 如果该ID已经用完所有片段，移除该ID
        if (segmentsByIdMap[selectedId].length === 0) {
          // 将原始片段重新填充回来用于后续的重复使用
          segmentsByIdMap[selectedId] = validSegments
            .filter(s => (s.id || 'unknown') === selectedId)
            .map(s => ({...s}));
        }
      }
      
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
      
      // 计算第一阶段已使用的时间
      const usedTime = finalSegments.reduce((sum, segment) => sum + segment.seconds, 0);
      
      // 第二阶段：填充剩余时间
      let remainingTime = totalAvailableSeconds - usedTime;
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
      <label for="total-time" class="setting-label">总时间:</label>
      <div class="time-input-wrapper">
        <div class="time-unit-input">
          <button class="time-button" on:click={() => incrementTime('hours')} aria-label="增加小时">
            <svg viewBox="0 0 24 24" width="16" height="16">
              <path d="M7 14l5-5 5 5z" fill="currentColor" />
            </svg>
          </button>
          <input 
            type="number" 
            min="0" 
            max="99" 
            bind:value={totalTimeValues.hours}
            on:input={() => handleTimeInput('hours', totalTimeValues.hours)}
            class="time-input hours"
          />
          <button class="time-button" on:click={() => decrementTime('hours')} aria-label="减少小时">
            <svg viewBox="0 0 24 24" width="16" height="16">
              <path d="M7 10l5 5 5-5z" fill="currentColor" />
            </svg>
          </button>
          <span class="time-label">小时</span>
        </div>
        <span class="time-separator">:</span>
        <div class="time-unit-input">
          <button class="time-button" on:click={() => incrementTime('minutes')} aria-label="增加分钟">
            <svg viewBox="0 0 24 24" width="16" height="16">
              <path d="M7 14l5-5 5 5z" fill="currentColor" />
            </svg>
          </button>
          <input 
            type="number" 
            min="0" 
            max="59" 
            bind:value={totalTimeValues.minutes}
            on:input={() => handleTimeInput('minutes', totalTimeValues.minutes)}
            class="time-input minutes"
          />
          <button class="time-button" on:click={() => decrementTime('minutes')} aria-label="减少分钟">
            <svg viewBox="0 0 24 24" width="16" height="16">
              <path d="M7 10l5 5 5-5z" fill="currentColor" />
            </svg>
          </button>
          <span class="time-label">分钟</span>
        </div>
        <span class="time-separator">:</span>
        <div class="time-unit-input">
          <button class="time-button" on:click={() => incrementTime('seconds')} aria-label="增加秒钟">
            <svg viewBox="0 0 24 24" width="16" height="16">
              <path d="M7 14l5-5 5 5z" fill="currentColor" />
            </svg>
          </button>
          <input 
            type="number" 
            min="0" 
            max="59" 
            bind:value={totalTimeValues.seconds}
            on:input={() => handleTimeInput('seconds', totalTimeValues.seconds)}
            class="time-input seconds"
          />
          <button class="time-button" on:click={() => decrementTime('seconds')} aria-label="减少秒钟">
            <svg viewBox="0 0 24 24" width="16" height="16">
              <path d="M7 10l5 5 5-5z" fill="currentColor" />
            </svg>
          </button>
          <span class="time-label">秒钟</span>
        </div>
      </div>
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
        <div class="segment-id-container">
          <svg class="segment-icon" viewBox="0 0 24 24" width="18" height="18">
            <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z" fill={getColorForId(segment.id)} />
          </svg>
          <input 
            bind:value={segment.id}
            placeholder="视频ID"
            class="segment-id"
            aria-label={`视频ID ${i + 1}`}
          />
        </div>
        <div class="segment-time-container">
          <svg class="time-icon" viewBox="0 0 24 24" width="18" height="18">
            <path d="M11.99 2C6.47 2 2 6.48 2 12s4.47 10 9.99 10C17.52 22 22 17.52 22 12S17.52 2 11.99 2zM12 20c-4.42 0-8-3.58-8-8s3.58-8 8-8 8 3.58 8 8-3.58 8-8 8zm.5-13H11v6l5.25 3.15.75-1.23-4.5-2.67z" fill="#666" />
          </svg>
          
          <!-- 现代化的时间输入控件 -->
          <div class="segment-time-input-wrapper">
            <div class="segment-time-unit-input">
              <button class="time-button" on:click={() => incrementSegmentTime(i, 'hours')} aria-label={`增加小时 ${i + 1}`}>
                <svg viewBox="0 0 24 24" width="16" height="16">
                  <path d="M7 14l5-5 5 5z" fill="currentColor" />
                </svg>
              </button>
              <input 
                type="number" 
                min="0" 
                max="99" 
                value={segment.timeValues?.hours || 0}
                on:input={(e) => handleSegmentTimeInput(i, 'hours', parseInt(e.currentTarget.value) || 0)}
                class="time-input hours"
                aria-label={`小时 ${i + 1}`}
              />
              <button class="time-button" on:click={() => decrementSegmentTime(i, 'hours')} aria-label={`减少小时 ${i + 1}`}>
                <svg viewBox="0 0 24 24" width="16" height="16">
                  <path d="M7 10l5 5 5-5z" fill="currentColor" />
                </svg>
              </button>
              <span class="time-label">时</span>
            </div>
            <span class="time-separator">:</span>
            <div class="segment-time-unit-input">
              <button class="time-button" on:click={() => incrementSegmentTime(i, 'minutes')} aria-label={`增加分钟 ${i + 1}`}>
                <svg viewBox="0 0 24 24" width="16" height="16">
                  <path d="M7 14l5-5 5 5z" fill="currentColor" />
                </svg>
              </button>
              <input 
                type="number" 
                min="0" 
                max="59" 
                value={segment.timeValues?.minutes || 0}
                on:input={(e) => handleSegmentTimeInput(i, 'minutes', parseInt(e.currentTarget.value) || 0)}
                class="time-input minutes"
                aria-label={`分钟 ${i + 1}`}
              />
              <button class="time-button" on:click={() => decrementSegmentTime(i, 'minutes')} aria-label={`减少分钟 ${i + 1}`}>
                <svg viewBox="0 0 24 24" width="16" height="16">
                  <path d="M7 10l5 5 5-5z" fill="currentColor" />
                </svg>
              </button>
              <span class="time-label">分</span>
            </div>
            <span class="time-separator">:</span>
            <div class="segment-time-unit-input">
              <button class="time-button" on:click={() => incrementSegmentTime(i, 'seconds')} aria-label={`增加秒钟 ${i + 1}`}>
                <svg viewBox="0 0 24 24" width="16" height="16">
                  <path d="M7 14l5-5 5 5z" fill="currentColor" />
                </svg>
              </button>
              <input 
                type="number" 
                min="0" 
                max="59" 
                value={segment.timeValues?.seconds || 0}
                on:input={(e) => handleSegmentTimeInput(i, 'seconds', parseInt(e.currentTarget.value) || 0)}
                class="time-input seconds"
                aria-label={`秒钟 ${i + 1}`}
              />
              <button class="time-button" on:click={() => decrementSegmentTime(i, 'seconds')} aria-label={`减少秒钟 ${i + 1}`}>
                <svg viewBox="0 0 24 24" width="16" height="16">
                  <path d="M7 10l5 5 5-5z" fill="currentColor" />
                </svg>
              </button>
              <span class="time-label">秒</span>
            </div>
          </div>
        </div>
        <button class="remove-button" on:click={() => removeSegment(i)} aria-label={`删除时间段 ${i + 1}`}>
          <svg viewBox="0 0 24 24" width="16" height="16">
            <path d="M6 19c0 1.1.9 2 2 2h8c1.1 0 2-.9 2-2V7H6v12zM19 4h-3.5l-1-1h-5l-1 1H5v2h14V4z" fill="currentColor" />
          </svg>
          <span>删除</span>
        </button>
      </div>
    {/each}
    
    <div class="buttons-container">
      <button class="add-button" on:click={addSegment}>
        <svg viewBox="0 0 24 24" width="16" height="16">
          <path d="M19 13h-6v6h-2v-6H5v-2h6V5h2v6h6v2z" fill="currentColor" />
        </svg>
        添加时间段
      </button>
      <button class="distribute-button" on:click={distributeSegments}>
        <svg viewBox="0 0 24 24" width="16" height="16">
          <path d="M3 17v2h6v-2H3zM3 5v2h10V5H3zm10 16v-2h8v-2h-8v-2h-2v6h2zM7 9v2H3v2h4v2h2V9H7zm14 4v-2H11v2h10zm-6-4h2V7h4V5h-4V3h-2v6z" fill="currentColor" />
        </svg>
        计算公平分配
      </button>
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
    max-width: 900px;
    margin: 0 auto;
    padding: 30px;
    font-family: 'SF Pro Text', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
    color: #333;
    background-color: #fff;
    border-radius: 10px;
    box-shadow: 0 10px 25px rgba(0, 0, 0, 0.05);
  }
  
  h1 {
    text-align: center;
    color: #2c3e50;
    margin-bottom: 30px;
    font-weight: 600;
    font-size: 2.2rem;
  }
  
  h2 {
    color: #2c3e50;
    margin: 25px 0 15px;
    font-weight: 500;
    font-size: 1.5rem;
    display: flex;
    align-items: center;
  }
  
  h2::before {
    content: '';
    display: inline-block;
    width: 4px;
    height: 18px;
    background-color: #3498db;
    margin-right: 10px;
    border-radius: 2px;
  }
  
  .settings-container {
    background-color: #f8f9fa;
    padding: 25px;
    border-radius: 12px;
    margin-bottom: 30px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  }
  
  .setting-label {
    font-weight: 500;
    color: #2c3e50;
    margin-bottom: 8px;
    display: block;
  }
  
  .total-time-container {
    margin-bottom: 20px;
  }
  
  /* 时间输入控件样式 */
  .time-input-wrapper {
    display: flex;
    align-items: center;
    margin-top: 8px;
  }
  
  .time-unit-input {
    display: flex;
    flex-direction: column;
    align-items: center;
    position: relative;
  }
  
  .time-input {
    width: 60px;
    height: 40px;
    text-align: center;
    font-size: 1.1rem;
    font-weight: 500;
    border: 1px solid #ddd;
    border-radius: 6px;
    padding: 0 5px;
    appearance: textfield;
    -moz-appearance: textfield;
  }
  
  .time-input::-webkit-inner-spin-button, 
  .time-input::-webkit-outer-spin-button { 
    -webkit-appearance: none;
    margin: 0;
  }
  
  .time-button {
    background-color: #f0f0f0;
    border: none;
    width: 24px;
    height: 24px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    color: #555;
    transition: all 0.2s;
    padding: 0;
    margin: 2px 0;
  }
  
  .time-button:hover {
    background-color: #e0e0e0;
    color: #333;
  }
  
  .time-separator {
    font-size: 1.5rem;
    margin: 0 8px;
    color: #666;
    font-weight: 300;
  }
  
  .time-label {
    font-size: 0.7rem;
    color: #777;
    margin-top: 4px;
  }
  
  .fairness-slider-container {
    margin: 25px 0 20px;
    position: relative;
  }
  
  .fairness-slider-container label {
    display: block;
    margin-bottom: 15px;
    font-weight: 500;
  }
  
  .fairness-slider-container input {
    width: 100%;
    height: 6px;
    border-radius: 3px;
    background: linear-gradient(to right, #3498db, #2ecc71);
    -webkit-appearance: none;
    appearance: none;
    -moz-appearance: none;
    outline: none;
    margin: 10px 0;
  }
  
  .fairness-slider-container input::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 18px;
    height: 18px;
    border-radius: 50%;
    background: #fff;
    border: 2px solid #3498db;
    cursor: pointer;
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.2);
  }
  
  .fairness-slider-container input::-moz-range-thumb {
    width: 18px;
    height: 18px;
    border-radius: 50%;
    background: #fff;
    border: 2px solid #3498db;
    cursor: pointer;
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.2);
  }
  
  .slider-label-left, .slider-label-right {
    font-size: 0.8em;
    color: #666;
    position: absolute;
    bottom: -20px;
  }
  
  .slider-label-left {
    left: 0;
  }
  
  .slider-label-right {
    right: 0;
  }
  
  .allow-repeat-container {
    margin-top: 25px;
    padding-top: 15px;
    border-top: 1px solid #eee;
  }
  
  .checkbox-label {
    display: flex;
    align-items: center;
    gap: 10px;
    cursor: pointer;
    font-weight: 500;
  }
  
  .checkbox-label input[type="checkbox"] {
    appearance: none;
    -webkit-appearance: none;
    width: 18px;
    height: 18px;
    border: 2px solid #3498db;
    border-radius: 4px;
    background-color: white;
    cursor: pointer;
    position: relative;
    transition: all 0.2s;
  }
  
  .checkbox-label input[type="checkbox"]:checked {
    background-color: #3498db;
  }
  
  .checkbox-label input[type="checkbox"]:checked::after {
    content: '';
    position: absolute;
    top: 2px;
    left: 5px;
    width: 5px;
    height: 10px;
    border: solid white;
    border-width: 0 2px 2px 0;
    transform: rotate(45deg);
  }
  
  .option-description {
    margin-top: 8px;
    font-size: 0.85em;
    color: #666;
    margin-left: 28px;
    line-height: 1.4;
  }
  
  .segments-container {
    background-color: #f8f9fa;
    padding: 25px;
    border-radius: 12px;
    margin-bottom: 30px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  }
  
  .segment-input {
    display: flex;
    gap: 15px;
    margin-bottom: 15px;
    align-items: center;
  }
  
  .segment-id-container, .segment-time-container {
    display: flex;
    align-items: center;
    background-color: white;
    border-radius: 8px;
    padding: 0 10px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
    transition: all 0.2s;
  }
  
  .segment-id-container:focus-within, .segment-time-container:focus-within {
    box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.3);
  }
  
  .segment-icon, .time-icon {
    margin-right: 8px;
  }
  
  .segment-id {
    padding: 12px 8px;
    border: none;
    border-radius: 8px;
    font-size: 0.95rem;
    flex: 1;
    outline: none;
    background: transparent;
    appearance: none;
    -webkit-appearance: none;
    -moz-appearance: none;
  }
  
  .segment-id-container {
    flex: 1;
  }
  
  .segment-time-container {
    width: 280px;
    position: relative;
  }
  
  /* 时间段时间输入样式 */
  .segment-time-input-wrapper {
    display: flex;
    align-items: center;
    padding: 0 5px;
  }
  
  .segment-time-unit-input {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  
  /* 修复 appearance 兼容性 */
  .time-input {
    width: 50px;
    height: 35px;
    text-align: center;
    font-size: 1rem;
    font-weight: 500;
    border: 1px solid #ddd;
    border-radius: 6px;
    padding: 0 5px;
    appearance: textfield;
    -webkit-appearance: textfield;
    -moz-appearance: textfield;
  }
  
  .time-input::-webkit-inner-spin-button, 
  .time-input::-webkit-outer-spin-button { 
    -webkit-appearance: none;
    appearance: none;
    margin: 0;
  }
  
  
  .remove-button {
    background-color: #ff5252;
    color: white;
    border: none;
    border-radius: 8px;
    padding: 10px 15px;
    cursor: pointer;
    font-weight: 500;
    display: flex;
    align-items: center;
    gap: 5px;
    transition: all 0.2s;
    height: 45px;
  }
  
  .remove-button:hover {
    background-color: #e04141;
  }
  
  .buttons-container {
    display: flex;
    gap: 15px;
    margin-top: 25px;
  }
  
  .add-button, .distribute-button {
    padding: 12px 18px;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    font-weight: 500;
    display: flex;
    align-items: center;
    gap: 8px;
    transition: all 0.2s;
  }
  
  .add-button {
    background-color: #4caf50;
    color: white;
  }
  
  .add-button:hover {
    background-color: #43a047;
  }
  
  .distribute-button {
    background-color: #3498db;
    color: white;
    margin-left: auto;
  }
  
  .distribute-button:hover {
    background-color: #2980b9;
  }
  
  .output-container {
    background-color: #f8f9fa;
    padding: 25px;
    border-radius: 12px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  }
  
  /* 时间线相关样式 */
  .timeline-container {
    margin: 30px 0;
    position: relative;
  }
  
  .timeline-header {
    display: flex;
    justify-content: space-between;
    margin-bottom: 8px;
    color: #666;
    font-size: 0.9em;
    font-weight: 500;
  }
  
  .timeline {
    height: 60px;
    background-color: #eaeff3;
    border-radius: 8px;
    position: relative;
    overflow: hidden;
    box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.1);
  }
  
  .timeline-segment {
    position: absolute;
    height: 100%;
    top: 0;
    border-radius: 4px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.15);
    transition: all 0.3s ease;
    overflow: hidden;
    cursor: pointer;
  }
  
  .timeline-segment:hover {
    transform: scaleY(1.08);
    z-index: 10;
    box-shadow: 0 3px 8px rgba(0, 0, 0, 0.25);
  }
  
  .segment-label {
    position: absolute;
    bottom: 5px;
    left: 5px;
    right: 5px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    font-size: 0.8em;
    color: white;
    text-shadow: 0 1px 2px rgba(0, 0, 0, 0.6);
    text-align: center;
    padding: 2px 6px;
    background-color: rgba(0, 0, 0, 0.3);
    border-radius: 4px;
  }
  
  .timeline-markers {
    position: relative;
    height: 30px;
    margin-top: 8px;
  }
  
  .timeline-marker {
    position: absolute;
    transform: translateX(-50%);
  }
  
  .marker-line {
    width: 2px;
    height: 10px;
    background-color: #888;
    margin: 0 auto;
  }
  
  .marker-label {
    font-size: 0.75em;
    color: #555;
    font-weight: 500;
    text-align: center;
    margin-top: 4px;
  }
  
  .output-list {
    display: flex;
    flex-direction: column;
    gap: 12px;
    margin-top: 25px;
  }
  
  .output-item {
    display: flex;
    align-items: center;
    gap: 15px;
    padding: 15px;
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.08);
    transition: transform 0.2s;
  }
  
  .output-item:hover {
    transform: translateY(-2px);
    box-shadow: 0 3px 8px rgba(0, 0, 0, 0.12);
  }
  
  .output-number {
    font-weight: bold;
    min-width: 30px;
    height: 30px;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #f0f0f0;
    border-radius: 50%;
    font-size: 0.85rem;
  }
  
  .output-id {
    flex: 1;
    font-weight: 500;
  }
  
  .output-time {
    font-weight: 600;
    color: #333;
    background-color: #f5f5f5;
    padding: 5px 10px;
    border-radius: 6px;
    font-size: 0.9em;
  }
  
  .output-range {
    color: #666;
    font-size: 0.9em;
    background-color: #f5f5f5;
    padding: 5px 10px;
    border-radius: 6px;
  }
  
  @media (max-width: 768px) {
    .segment-input {
      flex-direction: column;
      gap: 10px;
      align-items: stretch;
    }
    
    .segment-time-container {
      width: 100%;
    }
    
    .segment-time-input-wrapper {
      justify-content: center;
    }
    
    .time-input-wrapper, .segment-time-input-wrapper {
      flex-wrap: wrap;
      justify-content: center;
      gap: 5px;
    }
    
    .output-item {
      flex-wrap: wrap;
    }
    
    .time-input {
      width: 45px;
      height: 35px;
      font-size: 0.9rem;
    }
  }
</style>