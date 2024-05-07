<script>
    import DrawingTool from "./scripts/draw.svelte";

    function copyText() {
    navigator.clipboard.writeText('8P8rSfV6uYkkPwxwPTYXBsvea2ZntzZ1c6M3Y2hwKJ7U').then(() => {
      // Success: Display a success message
      alert('CA: 8P8rSfV6uYkkPwxwPTYXBsvea2ZntzZ1c6M3Y2hwKJ7U (Copied to clipboard)');
    }, (err) => {
      // Error: Log and alert the user of an error
      console.error('Failed to copy text: ', err);
      alert('Failed to copy text.');
    });
  }
</script>

<main class="main-content">
    <DrawingTool />
    <div class="ca-copy" on:click={copyText}>
        CA
      </div>
    <div class="social-links">
        <a href="https://twitter.com/noobcto/" target="_blank" rel="noopener" class="social-link">
            <img src="https://github.com/scaryskeeletons/images/raw/main/twitter.webp" alt="Twitter"/>
        </a>
        <a href="https://dexscreener.com/solana/8P8rSfV6uYkkPwxwPTYXBsvea2ZntzZ1c6M3Y2hwKJ7U" target="_blank" rel="noopener" class="social-link">
            <img src="https://github.com/scaryskeeletons/images/raw/main/dexscreener.webp" alt="Dex Screener"/>
        </a>
        <a href="https://t.me/noobcto" target="_blank" rel="noopener" class="social-link">
            <img src="https://github.com/scaryskeeletons/images/raw/main/telegram.webp" alt="Telegram"/>
        </a>
    </div>
    <div class="meme-images">
        <img src="https://github.com/scaryskeeletons/images/raw/main/noobimg.webp" alt="noob"/>
    </div>
</main>

<style>
    @import "./styles/styles.css";
</style>
