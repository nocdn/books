<script lang="ts">
  import { X } from "lucide-svelte";
  import { onMount } from "svelte";

  let { bookmark, onDeleteBookmark, onEditBookmark, index } = $props<{
    bookmark: Bookmark;
    onDeleteBookmark: (id: number) => void;
    onEditBookmark: (id: number, title: string, url: string) => void;
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

  const formattedUrl = new URL(bookmark.url).hostname.replace("www.", "") + "/";

  let isHovered = $state(false);
  let isOptionHeld = $state(false);
  let editingState = $state<"none" | "title" | "url">("none");
  let editableTitle = $state(bookmark.title);
  let editableUrl = $state(bookmark.url);
  let isUrlHovered = $state(false);

  function handleMouseEnter() {
    isHovered = true;
  }
  function handleMouseLeave() {
    isHovered = false;
  }

  function enterTitleEditMode(e: Event) {
    if (isOptionHeld) {
      e.preventDefault();
      editableTitle = bookmark.title;
      editableUrl = bookmark.url;
      editingState = "title";
    }
  }

  function enterUrlEditMode(e: Event) {
    if (isOptionHeld) {
      e.preventDefault();
      editableTitle = bookmark.title;
      editableUrl = bookmark.url;
      editingState = "url";
    }
  }

  function handleEdit(e: KeyboardEvent) {
    if (e.key === "Enter") {
      onEditBookmark(bookmark.id, editableTitle, editableUrl);
      editingState = "none";
    } else if (e.key === "Escape") {
      editingState = "none";
    }
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

  function displayUrlOnHover(url: string) {
    return url.replace(/^(https?:\/\/)?(www\.)?/, "");
  }
</script>

<div>
  <div
    role="button"
    tabindex="0"
    class="motion-opacity-in-0 motion-duration-400 flex items-center gap-2"
    style="animation-delay: {index * 0.035}s"
    onmouseenter={handleMouseEnter}
    onmouseleave={handleMouseLeave}>
    <img src={bookmark.favicon} alt={bookmark.title} class="h-4 w-4" />
    {#if editingState === "title"}
      <!-- svelte-ignore a11y_autofocus -->
      <input
        type="text"
        bind:value={editableTitle}
        onkeydown={handleEdit}
        onblur={() => (editingState = "none")}
        class="font-geist w-full truncate bg-transparent font-medium focus:outline-none"
        autofocus />
    {:else if editingState === "url"}
      <!-- svelte-ignore a11y_autofocus -->
      <input
        type="text"
        bind:value={editableUrl}
        onkeydown={handleEdit}
        onblur={() => (editingState = "none")}
        class="font-jetbrains-mono w-full bg-transparent py-0.5 text-sm text-gray-400 focus:outline-none"
        autofocus />
    {:else}
      <a
        href={bookmark.url}
        class="font-geist max-w-1/2 truncate font-medium {!isOptionHeld
          ? 'hover:text-[#e11d48]'
          : 'cursor-text'}"
        style="max-width:50%;overflow:hidden;text-overflow:ellipsis;white-space:nowrap;"
        onclick={enterTitleEditMode}>{bookmark.title}</a>

      <div
        role="button"
        tabindex="0"
        class="font-jetbrains-mono text-sm text-gray-400"
        onclick={enterUrlEditMode}
        onkeydown={(e) => {
          if (e.key === "Enter" || e.key === " ") enterUrlEditMode(e);
        }}
        onmouseenter={() => (isUrlHovered = true)}
        onmouseleave={() => (isUrlHovered = false)}>
        <span
          class="max-w-1/2 overflow-hidden text-ellipsis whitespace-nowrap"
          style="max-width:50%;">
          {isUrlHovered ? displayUrlOnHover(bookmark.url) : formattedUrl}
        </span>
      </div>

      <X
        size={16}
        class="ml-auto cursor-pointer text-red-500 transition-opacity duration-200 hover:text-red-800"
        strokeWidth={2.25}
        style="opacity:{isHovered && isOptionHeld
          ? 1
          : 0};pointer-events:{isHovered && isOptionHeld ? 'auto' : 'none'}"
        onclick={() => onDeleteBookmark(bookmark.id)} />

      <p class="font-jetbrains-mono text-sm text-gray-400">{formattedDate}</p>
    {/if}
  </div>

  <div
    class={`flex w-full items-center gap-2 transition-all ${
      isHovered && isOptionHeld && bookmark.tags.length > 0 ? "h-8" : "h-0"
    }`}>
    {#if bookmark.tags.length > 0 && isHovered && isOptionHeld}
      {#each bookmark.tags as tag}
        <div
          class="font-jetbrains-mono motion-opacity-in-0 -motion-translate-y-in-[10%] motion-duration-300 mt-1 w-fit rounded-sm bg-[#FFEEED] px-2 py-1 text-xs font-medium text-[#FF574B]">
          {tag.toUpperCase()}
        </div>
      {/each}
    {/if}
  </div>
</div>
