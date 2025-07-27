<script lang="ts">
  let {
    onGroupChange,
    onBackendChange,
    backendAddress,
  }: {
    onGroupChange: (group: string) => void;
    onBackendChange: (backend: string) => void;
    backendAddress: string;
  } = $props();
  import hand from "./assets/peace-hand.svg";
  import arrows from "./assets/arrow-separate-vertical.svg";
  import { Check } from "lucide-svelte";
  import Spinner from "./lib/Spinner.svelte";

  let groupColors = {
    main: "bg-blue-500",
    work: "bg-red-500",
  };

  let showingOptions = $state(false);

  type Status = "idle" | "typing" | "saved";
  let status: Status = $state("idle");
  let debounceTimer: number;

  let localBackendAddress = $state(backendAddress);

  $effect(() => {
    localBackendAddress = backendAddress;
  });

  function handleAddressInput() {
    status = "typing";
    clearTimeout(debounceTimer);
    debounceTimer = setTimeout(() => {
      let addressToSave = localBackendAddress.trim().replace(/\/+$/, "");
      if (addressToSave && !/^https?:\/\//i.test(addressToSave)) {
        addressToSave = `http://${addressToSave}`;
      }
      onBackendChange(addressToSave);
      status = "saved";
    }, 500);
  }

  async function handleExport() {
    const response = await fetch(`${backendAddress}/api/export`);
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
    {#if showingOptions}
      <div class="motion-preset-blur-left-sm flex items-center gap-2.5">
        <p class="text-sm font-medium text-gray-500">backend:</p>
        <input
          type="text"
          placeholder="https://books.address.com/"
          bind:value={localBackendAddress}
          class="w-52 rounded-md border border-gray-200 px-2 py-1 text-sm font-medium text-gray-500"
          oninput={handleAddressInput} />
        {#if status === "typing"}
          <Spinner size={16} />
        {:else if status === "saved"}
          <Check size={16} class="text-green-600" />
        {/if}
      </div>
      <button
        class="motion-preset-blur-left-sm flex cursor-pointer items-center gap-2 px-3 py-1.5 text-sm font-medium text-gray-500"
        onclick={handleExport}>
        export
      </button>
    {/if}
    <button
      class="flex cursor-pointer items-center gap-2 py-1.5 pr-3 pl-0 text-sm font-medium text-gray-500 transition-opacity {showingOptions
        ? 'opacity-60'
        : ''}"
      onclick={() => {
        showingOptions = !showingOptions;
      }}>
      options
    </button>
  </div>
</nav>
