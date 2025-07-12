<script lang="ts">
  let { onGroupChange }: { onGroupChange: (group: string) => void } = $props();
  import hand from "./assets/peace-hand.svg";
  import arrows from "./assets/arrow-separate-vertical.svg";
  let groupColors = {
    main: "bg-blue-500",
    work: "bg-red-500",
  };

  let selectedGroup = $state("main");

  async function handleExport() {
    const response = await fetch("http://localhost:5570/api/export");
    const contentDisposition = response.headers.get("Content-Disposition");

    const now = new Date();
    const year = now.getFullYear();
    const month = String(now.getMonth() + 1).padStart(2, "0");
    const day = String(now.getDate()).padStart(2, "0");
    const hours = String(now.getHours()).padStart(2, "0");
    const minutes = String(now.getMinutes()).padStart(2, "0");
    const seconds = String(now.getSeconds()).padStart(2, "0");

    let filename = `bookmarks_export_${year}${month}${day}_${hours}${minutes}${seconds}.sql`;

    if (contentDisposition) {
      const matches = contentDisposition.match(/filename="(.+)"/);
      if (matches && matches[1]) {
        filename = matches[1];
      }
    }

    const data = await response.blob();
    const url = window.URL.createObjectURL(data);
    const a = document.createElement("a");
    a.href = url;
    a.download = filename;
    a.click();
  }
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
      class="flex cursor-pointer appearance-none items-center gap-2 border-none bg-transparent text-sm font-medium opacity-90 outline-none"
      onchange={(e) => onGroupChange((e.target as HTMLSelectElement).value)}>
      <option value="main">main</option>
      <option value="work">work</option>
    </select>
    <img src={arrows} alt="arrows" class="h-3.5 w-3.5" />
  </div>
  <div class="flex items-center gap-2">
    <button
      class="flex cursor-pointer items-center gap-2 px-3 py-1.5 text-sm font-medium text-gray-500"
      onclick={handleExport}>
      export
    </button>
  </div>
</nav>
