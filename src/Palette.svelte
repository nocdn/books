<script lang="ts">
  let { onDismiss = () => {}, children } = $props();

  let dismissing = $state(false);
  function dismissPalette() {
    dismissing = true;
    setTimeout(() => {
      onDismiss();
    }, 300);
  }
</script>

<container
  onmousedown={dismissPalette}
  role="button"
  tabindex="0"
  class="absolute top-0 left-0 z-50 grid h-full w-full place-content-center bg-black/40 {dismissing
    ? 'animate-bg-fade-out'
    : 'animate-bg-fade-in'}">
  <palette
    role="dialog"
    tabindex="0"
    onmousedown={(e: MouseEvent) => e.stopPropagation()}
    class="h-96 w-md bg-white font-mono {dismissing
      ? 'animate-palette-out'
      : 'animate-palette-in'} flex flex-col overflow-hidden rounded-lg">
    {@render children()}
  </palette>
</container>

<style>
  .animate-palette-in {
    animation: palette-in 0.4s cubic-bezier(0.175, 0.885, 0.32, 1);
  }
  .animate-palette-out {
    animation: palette-out 0.3s cubic-bezier(0.215, 0.61, 0.355, 1);
  }

  .animate-bg-fade-in {
    animation: bg-fade-in 0.3s ease forwards;
  }
  .animate-bg-fade-out {
    animation: bg-fade-out 0.3s ease forwards;
  }

  @keyframes palette-in {
    0% {
      opacity: 0;
      filter: blur(4px);
      transform: translateY(30px) scale(1.1);
    }
    100% {
      opacity: 1;
      filter: blur(0);
      transform: translateY(0) scale(1);
    }
  }

  @keyframes palette-out {
    0% {
      opacity: 1;
      filter: blur(0);
      transform: translateY(0) scale(1);
    }
    100% {
      opacity: 0;
      filter: blur(4px);
      transform: translateY(15px) scale(1.05);
    }
  }

  @keyframes bg-fade-in {
    0% {
      opacity: 0;
    }
    100% {
      opacity: 1;
    }
  }

  @keyframes bg-fade-out {
    0% {
      opacity: 1;
    }
    100% {
      opacity: 0;
    }
  }
</style>
