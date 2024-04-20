<script>
  import { onMount } from 'svelte';

  let canvas, ctx;
  let isDrawing = false;
  export let color = '#000000'; // Default color
  export let lineWidth = 4;     // Default line width

  let states = [];
  let currentStateIndex = -1;

  function saveState() {
    if (currentStateIndex + 1 < states.length) {
      states = states.slice(0, currentStateIndex + 1);
    }
    states.push(canvas.toDataURL());
    currentStateIndex++;
    if (states.length > 10) {
      states.shift();
      currentStateIndex--; // current state minus 1
    }
  }

  function undo() {
    if (currentStateIndex > 0) {
      currentStateIndex--;
      restoreState();
    }
  }

  function redo() {
    if (currentStateIndex < states.length - 1) {
      currentStateIndex++;
      restoreState();
    }
  }

  function restoreState() {
    const img = new Image();
    img.onload = () => {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.drawImage(img, 0, 0);
    };
    img.src = states[currentStateIndex];
  }

  function startDrawing(event) {
    event.preventDefault(); // Prevent default to stop any browser default behavior
    if (!ctx) return;
    isDrawing = true;
    ctx.beginPath();
    const {x, y} = getPosition(event);
    ctx.moveTo(x, y);
  }

  function draw(event) {
    event.preventDefault();
    if (!isDrawing || !ctx) return;
    const {x, y} = getPosition(event);
    ctx.lineTo(x, y);
    ctx.stroke();
  }

  function stopDrawing(event) {
    event.preventDefault();
    if (!ctx || !isDrawing) return;
    isDrawing = false;
    saveState();
  }

  function getPosition(event) {
    let clientX, clientY;
    if (event.touches && event.touches.length) {
      clientX = event.touches[0].clientX;
      clientY = event.touches[0].clientY;
    } else {
      clientX = event.clientX;
      clientY = event.clientY;
    }

    return {
      x: clientX - canvas.getBoundingClientRect().left,
      y: clientY - canvas.getBoundingClientRect().top
    };
  }

  onMount(() => {
    ctx = canvas.getContext('2d');
    setupCanvas();
    saveState();
    window.addEventListener('resize', handleResize);
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  });

  function setupCanvas() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    ctx.strokeStyle = color;
    ctx.lineWidth = lineWidth;
  }

  function handleResize() {
    setupCanvas();
    restoreState();
  }

  $: if (ctx) {
    ctx.strokeStyle = color;
    ctx.lineWidth = lineWidth;
  }
</script>


<div class="toolbar">
  <input type="color" bind:value={color}/>
  <input type="range" min="1" max="80" bind:value={lineWidth}/>
  <button on:click={undo}>Undo</button>
  <button on:click={redo}>Redo</button>
</div>

<canvas bind:this={canvas}
        on:mousedown={startDrawing}
        on:mousemove={draw}
        on:mouseup={stopDrawing}
        on:mouseout={stopDrawing}
        on:blur={stopDrawing}
        on:touchstart={startDrawing}
        on:touchmove={draw}
        on:touchend={stopDrawing}
        style="border: none; touch-action: none;">
</canvas>
