<svelte:head>
  <title>Orrery</title>
</svelte:head>

<script>
import Scene from './Scene.svelte';
</script>

<canvas></canvas>

<style>
:global(html, body) {
  height: 100%;
  overflow: hidden;
}

canvas {
  display: block;
  width: 100%;
  height: 100%;
}
</style>