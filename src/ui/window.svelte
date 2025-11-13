<script>
  import { createEventDispatcher, onDestroy } from "svelte";

  let { url, title, icon, posx, posy, width, height } = $props();

  const dispatch = createEventDispatcher();

  // geometry as runes
  let visible = $state(true);
  let x = $state(posx);
  let y = $state(posy);
  // let width = $state(560);
  // let height = $state(320);

  const MIN_W = 200;
  const MIN_H = 120;
  const TASKBAR_HEIGHT = 46; // adjust to match your taskbar

  // maximize state + previous geometry store
  let maximized = $state(false);
  let prevGeom = {
    x: posx,
    y: posy,
    width: 560,
    height: 320,
  };

  // drag state
  let dragging = $state(false);
  let dragClientStart = { x: 0, y: 0, pointerId: null, target: null };
  let dragWinStart = { x: 0, y: 0 };

  // resize state
  let resizing = $state(false);
  let resizeDir = null;
  let resizeStart = {
    clientX: 0,
    clientY: 0,
    winX: 0,
    winY: 0,
    width: 0,
    height: 0,
    pointerId: null,
    target: null,
  };

  let winEl = $state(null);

  export function close() {
    visible = false;
    dispatch("close");
  }

  // toggle maximize / restore
  function toggleMaximize() {
    const vw = Math.max(
      document.documentElement.clientWidth,
      window.innerWidth || 0
    );
    const vh = Math.max(
      document.documentElement.clientHeight,
      window.innerHeight || 0
    );

    if (!maximized) {
      // save current geometry
      prevGeom = { x, y, width, height };
      // maximize (fill width, leave room for taskbar)
      x = 0;
      y = 0;
      width = Math.max(0, vw);
      height = Math.max(0, vh - TASKBAR_HEIGHT);
      maximized = true;
    } else {
      // restore
      let nx = prevGeom.x;
      let ny = prevGeom.y;
      let nw = prevGeom.width;
      let nh = prevGeom.height;

      if (nx + nw > vw) nx = Math.max(0, vw - nw);
      if (ny + nh > vh - TASKBAR_HEIGHT)
        ny = Math.max(0, vh - TASKBAR_HEIGHT - nh);

      x = nx;
      y = ny;
      width = Math.max(MIN_W, nw);
      height = Math.max(MIN_H, nh);
      maximized = false;
    }
  }

  // --- Drag handlers (titlebar) with pointer capture ---
  function titlePointerDown(e) {
    if (e.button !== 0) return;
    // don't start dragging when clicking controls
    if (e.target.closest && e.target.closest(".controls")) return;
    if (maximized) return; // don't drag when maximized

    dragging = true;
    dragClientStart = { x: e.clientX, y: e.clientY, pointerId: e.pointerId, target: e.currentTarget };
    dragWinStart = { x, y };

    // try to capture pointer on the titlebar so moves/up are delivered even when cursor leaves
    try {
      if (dragClientStart.target && dragClientStart.target.setPointerCapture) {
        dragClientStart.target.setPointerCapture(e.pointerId);
      }
    } catch (err) {}

    window.addEventListener("pointermove", onDragMove);
    window.addEventListener("pointerup", endDrag, { once: true });
    e.preventDefault();
  }

  function onDragMove(e) {
    if (!dragging) return;
    const dx = e.clientX - dragClientStart.x;
    const dy = e.clientY - dragClientStart.y;
    let newX = dragWinStart.x + dx;
    let newY = dragWinStart.y + dy;

    // clamp to viewport so right side never goes out of bounds
    const vw = Math.max(document.documentElement.clientWidth, window.innerWidth || 0);
    const vh = Math.max(document.documentElement.clientHeight, window.innerHeight || 0);

    // ensure window fully visible on right and bottom (leave taskbar height)
    const maxX = Math.max(0, vw - width);
    const maxY = Math.max(0, vh - TASKBAR_HEIGHT - height);

    if (newX < 0) newX = 0;
    if (newX > maxX) newX = maxX;
    if (newY < 0) newY = 0;
    if (newY > maxY) newY = maxY;

    x = Math.round(newX);
    y = Math.round(newY);
  }

  function endDrag(e) {
    dragging = false;
    // release pointer capture if set
    try {
      if (
        dragClientStart &&
        dragClientStart.target &&
        dragClientStart.pointerId != null &&
        dragClientStart.target.releasePointerCapture
      ) {
        dragClientStart.target.releasePointerCapture(dragClientStart.pointerId);
      }
    } catch (err) {}
    dragClientStart.pointerId = null;
    dragClientStart.target = null;
    window.removeEventListener("pointermove", onDragMove);
  }

  // --- Resize handlers with pointer capture ---
  function resizePointerDown(e, dir) {
    if (e.button !== 0) return;
    if (maximized) return; // can't resize while maximized

    resizing = true;
    resizeDir = dir;
    resizeStart = {
      clientX: e.clientX,
      clientY: e.clientY,
      winX: x,
      winY: y,
      width,
      height,
      pointerId: e.pointerId,
      target: e.currentTarget,
    };

    try {
      if (resizeStart.target && resizeStart.target.setPointerCapture) {
        resizeStart.target.setPointerCapture(e.pointerId);
      }
    } catch (err) {}

    window.addEventListener("pointermove", onResizeMove);
    window.addEventListener("pointerup", endResize, { once: true });
    e.preventDefault();
  }

  function onResizeMove(e) {
    if (!resizing) return;

    const dx = e.clientX - resizeStart.clientX;
    const dy = e.clientY - resizeStart.clientY;

    let newW = resizeStart.width;
    let newH = resizeStart.height;
    let newX = resizeStart.winX;
    let newY = resizeStart.winY;

    if (resizeDir.includes("e")) {
      newW = Math.max(MIN_W, resizeStart.width + dx);
    }
    if (resizeDir.includes("w")) {
      newW = Math.max(MIN_W, resizeStart.width - dx);
      newX = resizeStart.winX + dx;
      if (newW === MIN_W) newX = resizeStart.winX + (resizeStart.width - MIN_W);
    }

    if (resizeDir.includes("s")) {
      newH = Math.max(MIN_H, resizeStart.height + dy);
    }
    if (resizeDir.includes("n")) {
      newH = Math.max(MIN_H, resizeStart.height - dy);
      newY = resizeStart.winY + dy;
      if (newH === MIN_H) newY = resizeStart.winY + (resizeStart.height - MIN_H);
    }

    const vw = Math.max(
      document.documentElement.clientWidth,
      window.innerWidth || 0
    );
    const vh = Math.max(
      document.documentElement.clientHeight,
      window.innerHeight || 0
    );

    if (newX < 0) {
      newW += newX;
      newX = 0;
    }
    if (newY < 0) {
      newH += newY;
      newY = 0;
    }
    if (newX + newW > vw) newW = Math.min(newW, vw - newX);
    if (newY + newH > vh) newH = Math.min(newH, vh - newY);

    x = Math.round(newX);
    y = Math.round(newY);
    width = Math.round(newW);
    height = Math.round(newH);
  }

  function endResize() {
    resizing = false;
    try {
      if (
        resizeStart &&
        resizeStart.target &&
        resizeStart.pointerId != null &&
        resizeStart.target.releasePointerCapture
      ) {
        resizeStart.target.releasePointerCapture(resizeStart.pointerId);
      }
    } catch (err) {}
    resizeDir = null;
    resizeStart.pointerId = null;
    resizeStart.target = null;
    window.removeEventListener("pointermove", onResizeMove);
  }

  onDestroy(() => {
    window.removeEventListener("pointermove", onDragMove);
    window.removeEventListener("pointermove", onResizeMove);
  });
</script>

{#if visible}
  <div
    bind:this={winEl}
    class="window"
    role="dialog"
    aria-label={title}
    style="top: {y}px; left: {x}px; width: {width}px; height: {height}px"
  >
    <div class="titlebar" onpointerdown={titlePointerDown}>
      <div class="title"><img style="display: inline-block;margin-right:8px" alt={icon} width="14px" src={icon}>{title}</div>
      <div class="controls">
        <button class="ctrl" type="button" aria-label="minimize">—</button>
        <button class="ctrl" type="button" aria-label="maximize" onclick={toggleMaximize}>
          {#if maximized}⧉{:else}▢{/if}
        </button>
        <button class="ctrl close" type="button" aria-label="close" onclick={close}>✕</button>
      </div>
    </div>

    <div class="content">
      <iframe src={url} title={title}></iframe>
    </div>

    {#if !maximized}
      <!-- resize handles -->
      <div class="resize handle-n" onpointerdown={(e) => resizePointerDown(e, "n")}></div>
      <div class="resize handle-s" onpointerdown={(e) => resizePointerDown(e, "s")}></div>
      <div class="resize handle-e" onpointerdown={(e) => resizePointerDown(e, "e")}></div>
      <div class="resize handle-w" onpointerdown={(e) => resizePointerDown(e, "w")}></div>

      <div class="resize handle-ne" onpointerdown={(e) => resizePointerDown(e, "ne")}></div>
      <div class="resize handle-nw" onpointerdown={(e) => resizePointerDown(e, "nw")}></div>
      <div class="resize handle-se" onpointerdown={(e) => resizePointerDown(e, "se")}></div>
      <div class="resize handle-sw" onpointerdown={(e) => resizePointerDown(e, "sw")}></div>
    {/if}
  </div>
{/if}

<style>
  .window {
    position: absolute;
    width: 560px;
    height: 320px;
    border: 1px solid #202020;
    background: #202020;
    display: flex;
    flex-direction: column;
    overflow: hidden;
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.18);
    color: #ffffff;
    font-size: 13px;
    min-width: 200px;
    min-height: 120px;
    user-select: none;
    touch-action: none;
  }
  .titlebar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    background: #202020;
    cursor: grab;
  }
  .titlebar:active {
    cursor: grabbing;
  }
  .title {
    flex: 1;
    text-align: left;
    font-weight: 600;
    padding-left: 6px;
    pointer-events: none;
  }
  .controls {
    display: flex;
    gap: 6px;
    align-items: center;
    pointer-events: auto;
  }
  .ctrl {
    width: 28px;
    height: 22px;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    border: none;
    background: #202020;
    color: #ffffff;
    cursor: pointer;
    padding-top: 14px;
    padding-bottom: 14px;
    padding-left: 18px;
    padding-right: 18px;
  }
  .ctrl:hover {
    background: #373737;
  }
  .ctrl.close {
    background: #202020;
    color: #ffffff;
  }
  .ctrl.close:hover {
    background: #e81123;
  }
  .content {
    flex: 1;
    min-height: 0;
    background: #202020;
  }
  .content iframe {
    width: 100%;
    height: 100%;
    border: 0;
    display: block;
  }
  .resize {
    position: absolute;
    z-index: 6;
    background: transparent;
  }
  .handle-n,
  .handle-s {
    left: 10px;
    right: 10px;
    height: 12px;
  }
  .handle-n {
    top: -6px;
    cursor: ns-resize;
  }
  .handle-s {
    bottom: -6px;
    cursor: ns-resize;
  }
  .handle-e,
  .handle-w {
    top: 10px;
    bottom: 10px;
    width: 12px;
  }
  .handle-e {
    right: -6px;
    cursor: ew-resize;
  }
  .handle-w {
    left: -6px;
    cursor: ew-resize;
  }
  .handle-ne,
  .handle-nw,
  .handle-se,
  .handle-sw {
    width: 16px;
    height: 16px;
  }
  .handle-ne {
    right: -8px;
    top: -8px;
    cursor: nesw-resize;
  }
  .handle-nw {
    left: -8px;
    top: -8px;
    cursor: nwse-resize;
  }
  .handle-se {
    right: -8px;
    bottom: -8px;
    cursor: nwse-resize;
  }
  .handle-sw {
    left: -8px;
    bottom: -8px;
    cursor: nesw-resize;
  }
  .handle-n,
  .handle-s {
    left: 10px;
    right: 10px;
  }
  .handle-e,
  .handle-w {
    top: 10px;
    bottom: 10px;
  }
</style>