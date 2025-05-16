<script lang="ts">
  import { createEventDispatcher } from 'svelte';

  export let text: string;
  export let initialX: number;
  export let initialY: number;
  export let width: number = 100;
  export let height: number = 100;
  export let rotation: number = 0;
  export let id: string;

  const dispatch = createEventDispatcher();
  let isDragging = false;
  let startX: number;
  let startY: number;
  let x = initialX;
  let y = initialY;
  
  // 当位置改变时发送事件，包含左上角坐标及大小
  $: {
    // 当位置或大小改变时发送事件，包括旋转属性
    dispatch('move', {
      id,
      x,
      y,
      width,
      height,
      rotation
    });
  }

  function handleMouseDown(event: MouseEvent) {
    isDragging = true;
    startX = event.clientX - x;
    startY = event.clientY - y;
    event.preventDefault();
  }

  function handleMouseMove(event: MouseEvent) {
    if (!isDragging) return;
    x = event.clientX - startX;
    y = event.clientY - startY;
    event.preventDefault();
  }

  function handleMouseUp() {
    isDragging = false;
  }
</script>

<svelte:window on:mousemove={handleMouseMove} on:mouseup={handleMouseUp} />

<div
  class="draggable-box"
  style="
    left: {x}px;
    top: {y}px;
    width: {width}px;
    height: {height}px;
    transform: rotate({rotation}deg);
  "
  on:mousedown={handleMouseDown}
>
  <span>{text}</span>
</div>

<style>
  .draggable-box {
    position: absolute;
    background-color: #fff6a0;
    border: 1px solid #e0d782;
    cursor: move;
    user-select: none;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 28px;
    box-sizing: border-box;
    box-shadow: 2px 2px 5px rgba(0, 0, 0, 0.1);
    z-index: 10;
    transition: box-shadow 0.2s ease, transform 0.05s ease;
    padding: 5px;
  }

  span {
    font-weight: bold;
    font-family: "SimHei", "黑体", "Microsoft YaHei", "微软雅黑", sans-serif;
    color: #333;
  }

  /* 当正在拖动时修改样式 */
  .draggable-box:active {
    box-shadow: 4px 4px 10px rgba(0, 0, 0, 0.25);
    transform: scale(1.02);
  }
</style>