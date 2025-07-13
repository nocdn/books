<script lang="ts">
  let { appearing = true } = $props<{ appearing: boolean }>();
</script>

<span
  class="absolute bottom-8 left-1/2 flex -translate-x-1/2 transform items-center gap-3.5 rounded-full border border-gray-200 px-4 py-2 {appearing
    ? 'motion-preset-focus-sm motion-translate-y-in-[10%]'
    : 'motion-opacity-out-0 motion-blur-out-[3px]'}">
  <div
    class="h-1.5 w-1.5 rounded-full bg-green-400"
    title="You are here!"
    id="pulsating-dot">
  </div>
  <p class="text-sm font-medium text-gray-500">bookmark created</p>
</span>

<style>
  #pulsating-dot {
    animation: pulse 3s infinite ease-in-out;
  }

  @keyframes pulse {
    0%,
    100% {
      /* normal state */
      transform: scale(1);
      box-shadow: 0 0 6px 1px rgba(74, 222, 128, 0.5); /* Green shadow */
      opacity: 1;
    }
    50% {
      /* pulsed state */
      transform: scale(1.25);
      box-shadow: 0 0 8px 3px rgba(74, 222, 128, 0.7); /* Green shadow */
      opacity: 0.8;
    }
  }
</style>
