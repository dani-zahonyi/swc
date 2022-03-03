<svelte:options tag="my-slider" />

<input type="number" value={_value[0]} name={name[0]} />
{#if range}
  <input type="number" value={_value[1]} name={name[1]} />
{/if}

<div class="track">
  <div
    class="progress"
    style={progress} />
    <div
    class="thumb"
    style={`left: ${pos[0] * 100}%;`}
    use:handle
    on:dragstart={() => ((active[0] = true))}
    on:drag={({ detail: v }) => (pos[0] = v)}
    on:dragend={() => ((active[0] = false))}>
    <div  class="{active[0] ? 'active thumb-content':'thumb-content'}">
      
    </div>
  </div>
  {#if range}
  <div
  class="thumb"
  style={`left: ${pos[1] * 100}%;`}
  use:handle
  on:dragstart={() => ((active[1] = true))}
  on:drag={({ detail: v }) => (pos[1] = v)}
  on:dragend={() => ((active[1] = false))}>
  <div  class="{active[1] ? 'active thumb-content':'thumb-content'}">
  </div>
</div>
  {/if}
</div>

<script>
  import { createEventDispatcher } from "svelte";
import { get_current_component } from 'svelte/internal';

const component = get_current_component();
const svelteDispatch = createEventDispatcher();

const dispatch = (name, detail) => {
  svelteDispatch(name, detail);
  component.dispatchEvent?.(new CustomEvent(name, { detail }));
  
};

  let name = [];
  let range = false;
  let min = 0;
  let max = 100;
  let step = 1;
  let _value = [min, max];
  let pos;
  let active = [false, false];
  let order = false;
  let value;
  export { name, range, min, max, step, value, order };
  $: if (typeof value ==="string") _value = JSON.parse(value);
  $: if (active[0] || active[1]) setValue(pos);
  $: if (!active[0] && !active[1]) setPos(_value);
  $: if (range && order && active && pos) pos = checkPos(pos);
  $: if(typeof min==="string") min = parseInt(min);
  $: if(typeof max==="string") max = parseInt(max);
  $: min, max, clamp();
  $: progress = `
    left: ${range ? Math.min(pos[0], pos[1]) * 100 : 0}%;
    right: ${100 - Math.max(pos[0], (range ? pos[1] : pos[0])) * 100}%;
  `;

  function setValue(pos) {
      
    const offset = min % step;
    const width = max - min
    _value = pos
      .map(v => min + v * width)
      .map(v => Math.round((v - offset) / step) * step + offset);
    

  }

  function setPos(value) {
      
    pos = value
      .map(v => Math.min(Math.max(v, min), max))
      .map(v => (v - min) / (max - min));
      dispatch("input", _value);
    
  }
  function checkPos(pos) {
    return [Math.min(...pos), Math.max(...pos)];
  }
  function clamp() {
      
    setPos(_value);
    setValue(pos);
  }


  // thumb

function handle(node) {
  const onDown = getOnDown(node);

  node.addEventListener("touchstart", onDown);
  node.addEventListener("mousedown", onDown);
  return {
    destroy() {
      node.removeEventListener("touchstart", onDown);
      node.removeEventListener("mousedown", onDown);
    }
  };
}

function getOnDown(node) {
  const onMove = getOnMove(node);

  return function (e) {
    e.preventDefault();
    node.dispatchEvent(new CustomEvent("dragstart"));

    const moveevent = "touches" in e ? "touchmove" : "mousemove";
    const upevent = "touches" in e ? "touchend" : "mouseup";

    document.addEventListener(moveevent, onMove);
    document.addEventListener(upevent, onUp);

    function onUp(e) {
      e.stopPropagation();

      document.removeEventListener(moveevent, onMove);
      document.removeEventListener(upevent, onUp);

      node.dispatchEvent(new CustomEvent("dragend"));
    };
  };
}

function getOnMove(node) {
  const track = node.parentNode;

  return function (e) {
    const { left, width } = track.getBoundingClientRect();
    const clickOffset = "touches" in e ? e.touches[0].clientX : e.clientX;
    const clickPos = Math.min(Math.max((clickOffset - left) / width, 0), 1) || 0;
    node.dispatchEvent(new CustomEvent("drag", { detail: clickPos }));
  };
}
  
</script>

<style>
  input {
    display: none;
  }
  .track {
    margin: 16px 8px;
    position: relative;
    height: 4px;
    width: calc(100% - 16px);
    border-radius: 100vh;
    background: var(--track-bg, #ebebeb);
  }
  .progress {
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    border-radius: 100vh;
    background: var(--progress-bg, #8abdff);
  }
  .thumb {
    width: 16px;
    height: 16px;
    border-radius: 100vh;
    background: var(--thumb-bg, #5784fd);
  }
  .thumb {
    position: absolute;
    top: 50%;
    width: 0;
    height: 0;
  }
  .thumb-content {
    position: relative;
    width: fit-content;
    height: fit-content;
    width: 15px;
    height: 15px;
    transform: translate(-50%, -50%);
  }
  .thumb-content::before {
    content: "";
    position: absolute;
    width: 200%;
    height: 200%;
    transform: translate(-25%, -25%) scale(0.5);
    border-radius: 100vh;
    background: var(--thumb-bg, #5784fd);
    opacity: 30%;
    transition: transform 100ms ease-in-out;
  }
  .thumb-content.active::before {
    transform: translate(-25%, -25%) scale(1);
  }
</style>