<script lang="ts">
  import { Plus } from "lucide-svelte";
  import Bookmark from "./Bookmark.svelte";
  import Spinner from "./lib/Spinner.svelte";

  // ----------------- Types -----------------
  type BookmarkType = {
    id: number;
    title: string;
    url: string;
    tags: string[];
    favicon: string;
    createdAt: string;
  };

  type SubmittedBookmark = {
    url: string;
    tags: string[];
    createdAt: string;
  };

  // ----------------- Props -----------------
  let { bookmarks, onCreateBookmark, onDeleteBookmark, onEditBookmark } =
    $props<{
      bookmarks: BookmarkType[];
      onCreateBookmark: (bookmark: SubmittedBookmark) => Promise<any>;
      onDeleteBookmark: (id: number) => void;
      onEditBookmark: (id: number, title: string, url: string) => void;
    }>();

  let input = $state("");
  let isLoading = $state(false);
  // While request is pending, input text is gray
  let isPending = $state(false);
  // After success, trigger fade-out animation
  let isFading = $state(false);
  let inputElement: HTMLInputElement;

  const tags = $derived(
    input.match(/#([\w-]+)/g)?.map((t) => t.substring(1)) ?? [],
  );
  const url = $derived(input.replace(/#([\w-]+)/g, "").trim());

  function handleKeydown(event: KeyboardEvent) {
    if (event.key === "Enter" && (event.metaKey || event.ctrlKey)) {
      let submittedUrl = url;
      if (!/^https?:\/\//i.test(submittedUrl)) {
        submittedUrl = `https://${submittedUrl}`;
      }
      // Start spinner & mark as pending
      isLoading = true;
      isPending = true;

      // Trigger creation and await its completion to stop spinner
      onCreateBookmark({
        url: submittedUrl,
        tags: tags,
        createdAt: new Date().toLocaleString("en-GB", {
          timeZone: "Europe/London",
        }),
      })
        .then(() => {
          // Stop spinner and pending state
          isLoading = false;
          isPending = false;

          // Trigger fade-out of the input text
          isFading = true;

          // After the fade animation completes, clear input & reset
          setTimeout(() => {
            input = "";
            isFading = false;
          }, 120); // slightly longer than motion-duration-100
        })
        .catch(() => {
          // Stop spinner even if creation fails; error handling could be added here
          isLoading = false;
          isPending = false;
          isFading = false;
        });
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

  $effect(() => {
    const handleGlobalKeydown = (event: KeyboardEvent) => {
      if (event.key === "/" && inputElement) {
        const target = event.target as HTMLElement;
        if (target.tagName !== "INPUT") {
          event.preventDefault();
          inputElement.focus();
        }
      } else if (event.key === "Escape" && inputElement) {
        inputElement.blur();
      }
    };

    window.addEventListener("keydown", handleGlobalKeydown);

    return () => {
      window.removeEventListener("keydown", handleGlobalKeydown);
    };
  });

  let filteredBookmarks = $derived(
    bookmarks.filter(
      (bookmark: BookmarkType) =>
        bookmark.title.toLowerCase().includes(input.toLowerCase()) ||
        bookmark.tags.some((tag) =>
          tag.toLowerCase().includes(input.toLowerCase()),
        ) ||
        bookmark.url.toLowerCase().includes(input.toLowerCase()) ||
        bookmark.createdAt.toLowerCase().includes(input.toLowerCase()),
    ),
  );
</script>

<main class="flex w-3xl flex-col gap-4">
  <div
    class="group flex items-center gap-2 rounded-lg border border-gray-200 p-3 focus-within:outline-2 focus-within:outline-offset-1 focus-within:outline-gray-700/45">
    <div
      role="button"
      tabindex="0"
      class="mr-0.5 grid h-5 w-5 cursor-pointer place-items-center opacity-50 transition-opacity duration-200 group-focus-within:opacity-100 group-hover:opacity-100"
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
      class="font-jetbrains-mono w-full bg-transparent text-[15px] leading-none font-medium outline-none {isFading
        ? 'motion-opacity-out-0 motion-duration-100'
        : ''} {isPending ? 'text-gray-500' : ''}"
      bind:value={input}
      onkeydown={handleKeydown}
      bind:this={inputElement} />
  </div>
  <div
    class="flex h-0 items-center gap-2 transition-all duration-200 {tags.length >
    0
      ? 'h-6'
      : 'h-0'}">
    {#each tags as tag}
      <p
        class="font-jetbrains-mono motion-opacity-in-0 -motion-translate-y-in-[10%] motion-duration-300 rounded-sm bg-[#EAF8EF] px-1.5 py-0.5 text-[15px] font-medium text-[#00C460]">
        {tag}
      </p>
    {/each}
  </div>
  <div
    class="font-geist-mono flex items-center justify-between border-b border-gray-200 py-3 text-sm text-gray-400">
    <p class="font-medium">TITLE</p>
    <p>CREATED AT</p>
  </div>
  <div class="flex flex-col gap-3">
    {#each filteredBookmarks as bookmark, index (bookmark.id)}
      <Bookmark
        bookmark={bookmark}
        onDeleteBookmark={onDeleteBookmark}
        onEditBookmark={onEditBookmark}
        index={index} />
    {/each}
  </div>
</main>
