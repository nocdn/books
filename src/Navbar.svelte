<script lang="ts">
  let { onGroupChange }: { onGroupChange: (group: string) => void } = $props();
  import hand from "./assets/peace-hand.svg";
  import arrows from "./assets/arrow-separate-vertical.svg";
  let groupColors = {
    main: "bg-blue-500",
    work: "bg-red-500",
  };

  let selectedGroup = $state("main");
</script>

<nav class="font-geist flex items-center justify-between p-4 px-6">
  <div class="flex items-center gap-2">
    <img src={hand} alt="hand" class="h-5 w-5" />
    <p class="font-geist-mono mx-3 opacity-25">/</p>
    <div
      class="mt-[1.5px] mr-1 h-3.5 w-3.5 rounded-full {groupColors.work} transition-colors duration-300">
    </div>
    <select
      name="groups"
      id="groups"
      class="flex cursor-pointer appearance-none items-center gap-2 border-none bg-transparent outline-none"
      onchange={(e) => onGroupChange((e.target as HTMLSelectElement).value)}>
      <option value="main">main</option>
      <option value="work">work</option>
    </select>
    <img src={arrows} alt="arrows" class="h-3.5 w-3.5" />
  </div>
  <div class="flex items-center gap-2"></div>
</nav>
