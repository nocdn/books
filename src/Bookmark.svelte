<script lang="ts">
  import { Plus, X } from "lucide-svelte";
  import { onMount, tick } from "svelte";

  let {
    bookmark,
    onDeleteBookmark,
    onEditBookmark,
    onAddTag,
    onRemoveTag,
    index,
  } = $props<{
    bookmark: Bookmark;
    onDeleteBookmark: (id: number) => void;
    onEditBookmark: (id: number, title: string, url: string) => void;
    onAddTag: (bookmarkId: number, tag: string) => void;
    onRemoveTag: (bookmarkId: number, tag: string) => void;
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

  let formattedDate = $derived(getFormattedDate(bookmark.createdAt));

  function getFormattedDate(dateStr: string) {
    const [datePart, timePart] = dateStr.split(", ");
    const [day, month, year] = datePart.split("/");
    const [hours, minutes, seconds] = timePart.split(":");
    const date = new Date(
      Number(year),
      Number(month) - 1,
      Number(day),
      Number(hours),
      Number(minutes),
      Number(seconds),
    );
    return date.toLocaleString("en-US", {
      month: "short",
      day: "numeric",
    });
  }

  let formattedUrl = $derived(
    new URL(bookmark.url).hostname.replace("www.", "") + "/",
  );

  let isExpanded = $state(false);
  let isAddingTag = $state(false);
  let isOptionHeld = $state(false);
  let editingState = $state<"none" | "title" | "url">("none");
  let editableTitle = $state(bookmark.title);
  let editableUrl = $state(bookmark.url);
  let isUrlHovered = $state(false);

  function handleMouseEnter() {
    isExpanded = true;
  }
  function handleMouseLeave() {
    if (!isAddingTag) {
      isExpanded = false;
    }
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
    } else {
      navigator.clipboard.writeText(bookmark.url);
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
    const cleanUrl = url.replace(/^(https?:\/\/)?(www\.)?/, "");
    try {
      const urlObject = new URL(url);
      if (urlObject.pathname === "/" && !cleanUrl.endsWith("/")) {
        return `${cleanUrl}/`;
      }
    } catch (e) {
      // Invalid URL, return cleaned string as is
    }
    return cleanUrl;
  }

  let newTagValue: string = $state("");
  let newTagInputElement = $state<HTMLInputElement | null>(null);

  function handleNewTag() {
    const tag = newTagValue.trim();
    if (tag) {
      onAddTag(bookmark.id, tag);
    }
    newTagValue = "";
    isAddingTag = false;
  }
</script>

<div
  role="button"
  tabindex="0"
  onmouseenter={handleMouseEnter}
  onmouseleave={handleMouseLeave}>
  <div
    role="button"
    tabindex="0"
    class="motion-opacity-in-0 motion-duration-400 flex items-center gap-2"
    style="animation-delay: {index * 0.035}s">
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
        class="font-geist truncate font-medium {!isOptionHeld
          ? 'hover:text-[#c11a3f]'
          : 'cursor-text'}"
        onclick={enterTitleEditMode}>{bookmark.title}</a>

      <div
        role="button"
        tabindex="0"
        class="font-jetbrains-mono min-w-0 flex-1 text-sm text-gray-400"
        onclick={enterUrlEditMode}
        onkeydown={(e) => {
          if (e.key === "Enter" || e.key === " ") enterUrlEditMode(e);
        }}
        onmouseenter={() => (isUrlHovered = true)}
        onmouseleave={() => (isUrlHovered = false)}>
        <span class="block overflow-hidden text-ellipsis whitespace-nowrap">
          {isUrlHovered ? displayUrlOnHover(bookmark.url) : formattedUrl}
        </span>
      </div>

      <X
        size={16}
        class="ml-auto cursor-pointer text-red-500 transition-opacity duration-200 hover:text-red-800"
        strokeWidth={2.25}
        style="opacity:{isExpanded && isOptionHeld
          ? 1
          : 0};pointer-events:{isExpanded && isOptionHeld ? 'auto' : 'none'}"
        onclick={() => onDeleteBookmark(bookmark.id)} />

      <p
        class="font-sf-pro-text flex-shrink-0 text-sm whitespace-nowrap text-gray-400">
        {formattedDate}
      </p>
    {/if}
  </div>

  <div
    class={`flex w-full items-center gap-2 transition-all ${
      isExpanded && (isOptionHeld || isAddingTag) && bookmark.tags.length > 0
        ? "h-8"
        : "h-0"
    }`}>
    {#if bookmark.tags.length > 0 && isExpanded && (isOptionHeld || isAddingTag)}
      {#each bookmark.tags as tag}
        <button
          onclick={() => {
            onRemoveTag(bookmark.id, tag);
          }}
          class="font-jetbrains-mono motion-opacity-in-0 -motion-translate-y-in-[10%] motion-duration-300 mt-1 w-fit cursor-pointer rounded-sm bg-[#F1F1F1] px-2 py-1 text-xs font-medium text-[#787879] hover:bg-[#FFEEED] hover:text-[#FF574B]">
          {tag.toUpperCase()}
        </button>
      {/each}
      {#if isAddingTag}
        <input
          type="text"
          class="font-jetbrains-mono mt-1 w-fit py-1 pl-1.5 text-xs font-medium uppercase focus:outline-gray-300"
          bind:value={newTagValue}
          bind:this={newTagInputElement}
          onkeydown={(e) => {
            if (e.key === "Enter") {
              handleNewTag();
            } else if (e.key === "Escape") {
              isAddingTag = false;
              newTagValue = "";
            }
          }}
          onblur={() => {
            isAddingTag = false;
            newTagValue = "";
          }} />
      {:else}
        <div class="">
          <Plus
            size={16}
            class="cursor-pointer text-[#787879]"
            onclick={async () => {
              isAddingTag = true;
              await tick();
              newTagInputElement?.focus();
            }} />
        </div>
      {/if}
    {/if}
  </div>
</div>
