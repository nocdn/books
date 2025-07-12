<script lang="ts">
  import { Plus } from "lucide-svelte";
  import Bookmark from "./Bookmark.svelte";
  import Spinner from "./lib/Spinner.svelte";
  let { bookmarks, onCreateBookmark } = $props<{
    bookmarks: BookmarkType[];
    onCreateBookmark: (bookmark: BookmarkType) => void;
  }>();
  type BookmarkType = {
    id: number;
    title: string;
    url: string;
    tags: string[];
    favicon: string;
    createdAt: string;
  };

  let input = $state("");
  let isLoading = $state(false);
  let inputSending = $state(false);

  function handleKeydown(event: KeyboardEvent) {
    if (event.key === "Enter" && (event.metaKey || event.ctrlKey)) {
      onCreateBookmark({
        url: input,
      });
      isLoading = true;
      inputSending = true;
      setTimeout(() => {
        isLoading = false;
        inputSending = false;
      }, 500);
    }
  }

  function handleFileUpload() {
    const input = document.createElement("input");
    input.type = "file";
    input.accept = "text/plain, image/*, audio/*";
    input.onchange = (e) => {
      const file = (e.target as HTMLInputElement).files?.[0];
      console.log(file);
    };
    input.click();
  }
</script>

<main class="flex w-3xl flex-col gap-4">
  <div
    class="group flex items-center gap-2 rounded-lg border border-gray-200 p-3 focus-within:outline-2 focus-within:outline-offset-1 focus-within:outline-gray-700/45">
    <div
      role="button"
      tabindex="0"
      class="mr-0.5 grid h-5 w-5 cursor-pointer place-items-center"
      onmousedown={handleFileUpload}>
      {#if isLoading}
        <Spinner size={22} />
      {:else}
        <Plus size={18} />
      {/if}
    </div>
    <input
      type="text"
      placeholder="DREAM BIG, START SMALL"
      class="font-jetbrains-mono w-full bg-transparent text-[15px] leading-none font-medium outline-none {inputSending
        ? 'motion-opacity-out-0 motion-duration-100'
        : ''}"
      bind:value={input}
      onkeydown={handleKeydown} />
  </div>
  <div
    class="font-geist-mono flex items-center justify-between border-b border-gray-200 py-3 text-sm text-gray-400">
    <p class="font-medium">TITLE</p>
    <p>CREATED AT</p>
  </div>
  <div class="flex flex-col gap-2">
    {#each bookmarks as bookmark}
      <Bookmark bookmark={bookmark} />
    {/each}
  </div>
</main>
