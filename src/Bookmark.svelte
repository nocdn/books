<script lang="ts">
  import { X } from "lucide-svelte";
  import { onMount } from "svelte";
  let { bookmark, onDeleteBookmark, index } = $props<{
    bookmark: Bookmark;
    onDeleteBookmark: (id: number) => void;
    index: number;
  }>();
  type Bookmark = {
    id: number;
    title: string;
    url: string;
    tags: string[];
    favicon: string;
    createdAt: string;
  };

  const date = new Date(bookmark.createdAt);
  const formattedDate = date.toLocaleString("en-US", {
    month: "short",
    day: "numeric",
  });

  const formattedUrl = bookmark.url
    .replace("https://", "")
    .replace("http://", "")
    .replace("www.", "");

  let isHovered = $state(false);
  let isOptionHeld = $state(false);

  function handleMouseEnter() {
    isHovered = true;
  }
  function handleMouseLeave() {
    isHovered = false;
  }

  function handleKeyDown(e: KeyboardEvent) {
    if (e.altKey) {
      isOptionHeld = true;
    }
  }
  function handleKeyUp(e: KeyboardEvent) {
    if (!e.altKey) {
      isOptionHeld = false;
    }
  }

  onMount(() => {
    window.addEventListener("keydown", handleKeyDown);
    window.addEventListener("keyup", handleKeyUp);
    return () => {
      window.removeEventListener("keydown", handleKeyDown);
      window.removeEventListener("keyup", handleKeyUp);
    };
  });
</script>

<div
  role="button"
  tabindex="0"
  class="motion-opacity-in-0 motion-duration-200 flex items-center gap-2"
  style="animation-delay: {index * 0.035}s"
  onmouseenter={handleMouseEnter}
  onmouseleave={handleMouseLeave}>
  <img src={bookmark.favicon} alt={bookmark.title} class="h-4 w-4" />
  <a href={bookmark.url} class="font-geist font-medium">{bookmark.title}</a>
  <p class="font-jetbrains-mono text-sm text-gray-400">
    {formattedUrl}
  </p>
  <X
    size={16}
    class="ml-auto cursor-pointer text-red-500 transition-opacity duration-200 hover:text-red-800"
    strokeWidth={2.25}
    style="opacity: {isHovered && isOptionHeld
      ? 1
      : 0}; pointer-events: {isHovered && isOptionHeld ? 'auto' : 'none'}"
    onclick={() => onDeleteBookmark(bookmark.id)} />
  <p class="font-jetbrains-mono text-sm text-gray-400">
    {formattedDate}
  </p>
</div>
