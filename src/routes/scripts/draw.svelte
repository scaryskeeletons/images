<script>
  import { onMount, onDestroy } from 'svelte';

  let canvas, ctx;
  let isDrawing = false;
  export let color = '#000000'; // Default color
  export let lineWidth = 4;     // Default line width

  let states = [];
  let currentStateIndex = -1;

  function saveState() {
    if (currentStateIndex + 1 < states.length) {
      states = states.slice(0, currentStateIndex + 1); // Remove all states after current index
    }
    states.push(canvas.toDataURL()); // Save current canvas as an image
    currentStateIndex++;
    if (states.length > 5) { // Only keep last 5 states
      states.shift();
      currentStateIndex--;
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
    if (!ctx) return;
    isDrawing = true;
    ctx.beginPath();
    const {x, y} = getPosition(event);
    ctx.moveTo(x, y);
    console.log('Drawing started with color:', color, 'and line width:', lineWidth);
  }

  function draw(event) {
    if (!isDrawing || !ctx) return;
    const {x, y} = getPosition(event);
    ctx.lineTo(x, y);
    ctx.stroke();
    console.log('Drawing');
  }

  function stopDrawing() {
    if (!ctx) return;
    isDrawing = false;
    console.log('Drawing stopped');
    saveState(); // Save state when drawing stops
  }

  function getPosition(event) {
    let clientX = event.clientX;
    let clientY = event.clientY;

    if (event.touches) {
      clientX = event.touches[0].clientX;
      clientY = event.touches[0].clientY;
    }

    return {
      x: clientX - canvas.getBoundingClientRect().left,
      y: clientY - canvas.getBoundingClientRect().top
    };
  }

  onMount(() => {
    ctx = canvas.getContext('2d');
    setupCanvas();
    saveState(); // Save the initial blank state
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
    console.log('Canvas setup with initial styles');
  }

  function handleResize() {
    setupCanvas();
    restoreState(); // Redraw the current state to the resized canvas
  }

  // Reactively update the styles directly
  $: if (ctx) {
    ctx.strokeStyle = color;
    ctx.lineWidth = lineWidth;
    console.log(`Reactive update - Color: ${color}, Line Width: ${lineWidth}`);
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
