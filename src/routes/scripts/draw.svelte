<script>
  import { onMount } from 'svelte';

  let canvas, ctx;
  let isDrawing = false;
  export let color = '#000000'; // Default color
  export let lineWidth = 4;     // Default line width

  let lastX, lastY; // Track the last points for smooth curves
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
      currentStateIndex--; // Adjust the index after shifting
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
    event.preventDefault();
    isDrawing = true;
    const {x, y} = getPosition(event);
    lastX = x;
    lastY = y;
    ctx.beginPath();
    ctx.moveTo(x, y);
  }

  function draw(event) {
    event.preventDefault();
    if (!isDrawing || !ctx) return;
    const {x, y} = getPosition(event);
    smoothLine(x, y);
  }

  function stopDrawing(event) {
    event.preventDefault();
    if (!isDrawing) return;
    isDrawing = false;
    ctx.lineTo(lastX, lastY);
    ctx.stroke(); // Draw the final segment
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

  function smoothLine(x, y) {
    ctx.quadraticCurveTo(lastX, lastY, (lastX + x) / 2, (lastY + y) / 2);
    ctx.stroke();
    lastX = x;
    lastY = y;
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
    canvas.width = window.innerWidth * devicePixelRatio;
    canvas.height = window.innerHeight * devicePixelRatio;
    canvas.style.width = `${window.innerWidth}px`;
    canvas.style.height = `${window.innerHeight}px`;
    ctx.scale(devicePixelRatio, devicePixelRatio);
    ctx.strokeStyle = color;
    ctx.lineWidth = lineWidth;
    ctx.lineCap = 'round'; // Smooth the end of the line
    ctx.lineJoin = 'round'; // Smooth the joint of the line
  }

  function handleResize() {
    const imgData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    setupCanvas();
    ctx.putImageData(imgData, 0, 0);
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
