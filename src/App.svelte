<script lang="ts">
  import Navbar from "./Navbar.svelte";
  import { DownloadCloud, Plus } from "lucide-svelte";
  import Bookmark from "./Bookmark.svelte";
  import Spinner from "./lib/Spinner.svelte";
  import Palette from "./Palette.svelte";
  let backendAddress = $state("");

  $effect(() => {
    const savedAddress = localStorage.getItem("backendAddress");
    if (savedAddress) {
      backendAddress = savedAddress;
    }
  });

  function handleGroupChange(group: string) {
    console.log(group);
  }

  function handleBackendChange(newAddress: string) {
    backendAddress = newAddress;
    localStorage.setItem("backendAddress", newAddress);
  }

  type Bookmark = {
    id: number;
    title: string;
    url: string;
    tags: string[];
    favicon: string;
    createdAt: string;
  };

  let bookmarks = $state<Bookmark[]>([]);
  let tags = $state<string[]>([]);

  async function fetchBookmarks() {
    isLoading = true;
    if (!backendAddress) return;
    const response = await fetch(`${backendAddress}/api/list`);
    const data = await response.json();
    console.log("fetched bookmark", data);
    bookmarks = data.bookmarks;
    isLoading = false;
  }

  async function fetchTags() {
    const response = await fetch(`${backendAddress}/api/tags`);
    const data = await response.json();
    console.log("fetched tags", data);
    tags = data.tags;
  }

  $effect(() => {
    fetchBookmarks();
    fetchTags();
  });

  async function deleteBookmark(id: number) {
    // Store the bookmark in case we need to restore it
    const bookmarkToDelete = bookmarks.find((b) => b.id === id);
    if (!bookmarkToDelete) {
      console.error("Bookmark not found");
      return;
    }

    // Optimistically remove from UI immediately
    bookmarks = bookmarks.filter((b) => b.id !== id);

    try {
      const response = await fetch(`${backendAddress}/api/delete/${id}`, {
        method: "DELETE",
      });

      if (!response.ok) {
        // If delete failed, restore the bookmark
        bookmarks = [...bookmarks, bookmarkToDelete].sort(
          (a, b) => b.id - a.id,
        );
        const errPayload = await response.json().catch(() => null);
        console.error("Failed to delete bookmark", errPayload);
        throw new Error(errPayload?.message ?? "Failed to delete bookmark");
      }

      const data = await response.json();
      console.log(data);

      // On success, we don't need to do anything since bookmark is already removed
      // But we might want to refresh tags if they've changed
      await fetchTags();
    } catch (error) {
      // If there was a network error or other exception, restore the bookmark
      if (bookmarks.find((b) => b.id === id) === undefined) {
        bookmarks = [...bookmarks, bookmarkToDelete].sort(
          (a, b) => b.id - a.id,
        );
      }
      console.error("Error deleting bookmark:", error);
    }
  }

  type submittedBookmark = {
    url: string;
    tags: string[];
    createdAt: string;
  };

  async function handleCreateBookmark(bookmark: submittedBookmark) {
    const response = await fetch(`${backendAddress}/api/create`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(bookmark),
    });

    // Only proceed when the server confirms success
    if (!response.ok) {
      // Attempt to parse JSON error, but ignore if it fails
      const errPayload = await response.json().catch(() => null);
      console.error("Failed to create bookmark", errPayload);
      throw new Error(errPayload?.message ?? "Failed to create bookmark");
    }

    const data = await response.json();
    console.log(data);

    // Refresh list after successful creation
    await fetchBookmarks();
    await fetchTags();

    // Return created bookmark payload so caller can await
    return data;
  }

  async function editBookmark(id: number, title: string, url: string) {
    // Store the original bookmark in case we need to restore it
    const originalBookmark = bookmarks.find((b) => b.id === id);
    if (!originalBookmark) {
      console.error("Bookmark not found");
      return;
    }

    // Optimistically update the bookmark in the UI immediately
    bookmarks = bookmarks.map((b) => (b.id === id ? { ...b, title, url } : b));

    try {
      const response = await fetch(`${backendAddress}/api/update/${id}`, {
        method: "PUT",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ title, url }),
      });

      if (!response.ok) {
        // If update failed, restore the original bookmark
        bookmarks = bookmarks.map((b) => (b.id === id ? originalBookmark : b));
        const errPayload = await response.json().catch(() => null);
        console.error("Failed to update bookmark", errPayload);
        throw new Error(errPayload?.message ?? "Failed to update bookmark");
      }

      const data = await response.json();
      console.log(data);

      // On success, we don't need to do anything since bookmark is already updated
      // But we might want to refresh to get any additional server-side changes
      // For now, we'll skip the full refresh to keep the optimistic behavior
    } catch (error) {
      // If there was a network error or other exception, restore the original bookmark
      if (
        bookmarks.find(
          (b) =>
            b.id === id &&
            (b.title !== originalBookmark.title ||
              b.url !== originalBookmark.url),
        )
      ) {
        bookmarks = bookmarks.map((b) => (b.id === id ? originalBookmark : b));
      }
      console.error("Error updating bookmark:", error);
    }
  }

  let input = $state("");
  let isLoading = $state(false);
  // While request is pending, input text is gray
  let isPending = $state(false);
  // After success, trigger fade-out animation
  let isFading = $state(false);
  let inputElement: HTMLInputElement;

  // parsedTags is the array of tags in the input
  const parsedTags = $derived(
    input.match(/#([\w-]+)/g)?.map((t) => t.substring(1)) ?? [],
  );
  const url = $derived(input.replace(/#([\w-]+)/g, "").trim());

  const currentTagQuery = $derived(input.match(/#([\w-]*)$/)?.[1] ?? null);

  // when `currentTagQuery` is not null, are currently typing a tag so show suggestions.
  const showTagSuggestions = $derived(currentTagQuery !== null);

  function handleKeydown(event: KeyboardEvent) {
    // If only one tag suggestion remains, Tab autocompletes it.
    if (
      event.key === "Tab" &&
      showTagSuggestions &&
      filteredTags.length === 1
    ) {
      event.preventDefault();
      const suggestion = filteredTags[0];
      // Replace the current partial tag (after the last '#') with the full suggestion plus a trailing space
      input = input.replace(/#([\w-]*)$/, `#${suggestion} `);
      return; // Skip further handling
    }

    if (event.key === "Enter") {
      let submittedUrl = url;
      if (!/^https?:\/\//i.test(submittedUrl)) {
        submittedUrl = `https://${submittedUrl}`;
      }
      // Start spinner & mark as pending
      isLoading = true;
      isPending = true;

      handleCreateBookmark({
        url: submittedUrl,
        tags: parsedTags,
        createdAt: new Date().toLocaleString("en-GB", {
          timeZone: "Europe/London",
        }),
      })
        .then(() => {
          isLoading = false;
          isPending = false;

          isFading = true;

          setTimeout(() => {
            input = "";
            isFading = false;
          }, 120);
        })
        .catch(() => {
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

  function handleTagClick(suggestion: string) {
    // Replace current partial tag (after last '#') with full suggestion + space
    input = input.replace(/#([\w-]*)$/, `#${suggestion} `);
    // Keep focus in the input so user can continue typing
    inputElement?.focus();
  }

  async function addTag(bookmarkId: number, tag: string) {
    const bookmark = bookmarks.find((b) => b.id === bookmarkId);
    if (!bookmark) {
      console.error("Bookmark not found");
      return;
    }

    if (bookmark.tags.includes(tag)) {
      return;
    }

    const originalTags = [...bookmark.tags];
    const updatedTags = [...bookmark.tags, tag];

    // Optimistically update the bookmark tags in the UI immediately
    bookmarks = bookmarks.map((b) =>
      b.id === bookmarkId ? { ...b, tags: updatedTags } : b,
    );

    try {
      const response = await fetch(
        `${backendAddress}/api/update/${bookmarkId}`,
        {
          method: "PUT",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({ tags: updatedTags }),
        },
      );

      if (!response.ok) {
        // If add tag failed, restore the original tags
        bookmarks = bookmarks.map((b) =>
          b.id === bookmarkId ? { ...b, tags: originalTags } : b,
        );
        console.error("Failed to add tag");
        return;
      }

      // On success, refresh tags to get any new tags that might have been created
      await fetchTags();
    } catch (error) {
      // If there was a network error, restore the original tags
      bookmarks = bookmarks.map((b) =>
        b.id === bookmarkId ? { ...b, tags: originalTags } : b,
      );
      console.error("Error adding tag:", error);
    }
  }

  async function removeTag(bookmarkId: number, tag: string) {
    const bookmark = bookmarks.find((b) => b.id === bookmarkId);
    if (!bookmark) {
      console.error("Bookmark not found");
      return;
    }

    const originalTags = [...bookmark.tags];
    const updatedTags = bookmark.tags.filter((t) => t !== tag);

    // Optimistically update the bookmark tags in the UI immediately
    bookmarks = bookmarks.map((b) =>
      b.id === bookmarkId ? { ...b, tags: updatedTags } : b,
    );

    try {
      const response = await fetch(
        `${backendAddress}/api/update/${bookmarkId}`,
        {
          method: "PUT",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({ tags: updatedTags }),
        },
      );

      if (!response.ok) {
        // If remove tag failed, restore the original tags
        bookmarks = bookmarks.map((b) =>
          b.id === bookmarkId ? { ...b, tags: originalTags } : b,
        );
        console.error("Failed to remove tag");
        return;
      }

      // On success, refresh tags to update the global tags list
      await fetchTags();
    } catch (error) {
      // If there was a network error, restore the original tags
      bookmarks = bookmarks.map((b) =>
        b.id === bookmarkId ? { ...b, tags: originalTags } : b,
      );
      console.error("Error removing tag:", error);
    }
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
    bookmarks.filter((bookmark: Bookmark) => {
      const lowerInput = input.toLowerCase();

      // If the input is just "#" (optionally followed by spaces), don’t filter anything.
      if (/^#\s*$/.test(lowerInput)) {
        return true;
      }

      // Split the query into whitespace-separated tokens
      const tokens = lowerInput.trim().split(/\s+/);

      // Every token must match somewhere in the bookmark
      return tokens.every((token) => {
        if (token === "") return true;

        if (token.startsWith("#")) {
          // Tag token → match against bookmark tags (strip leading '#')
          const tagQuery = token.slice(1);
          if (tagQuery === "") return true; // token is just '#'
          return bookmark.tags.some((tag) =>
            tag.toLowerCase().includes(tagQuery),
          );
        }

        // Non-tag token → match against title, url, date, or tags
        return (
          bookmark.title.toLowerCase().includes(token) ||
          bookmark.url.toLowerCase().includes(token) ||
          bookmark.createdAt.toLowerCase().includes(token) ||
          bookmark.tags.some((tag) => tag.toLowerCase().includes(token))
        );
      });
    }),
  );

  let filteredTags = $derived(
    currentTagQuery === null
      ? []
      : currentTagQuery === ""
        ? tags
        : tags.filter((tag) =>
            tag.toLowerCase().includes(currentTagQuery.toLowerCase()),
          ),
  );

  $inspect(parsedTags);
  $inspect(filteredTags);

  let showingPalette = $state(false);
</script>

<main>
  <Navbar
    onOptionPress={() => (showingPalette = true)}
    onGroupChange={handleGroupChange}
    onBackendChange={handleBackendChange}
    backendAddress={backendAddress} />
  <div class="mt-24 grid w-full place-items-center">
    <div class="flex w-3xl flex-col gap-4">
      <div
        class="group flex h-11 items-center gap-2 rounded-md border border-gray-200 px-3 shadow-xs transition-all focus-within:outline-2 focus-within:outline-offset-1 focus-within:outline-gray-700/45">
        <div
          role="button"
          tabindex="0"
          class="mr-0.5 grid h-5 w-5 cursor-pointer place-items-center opacity-50 transition-opacity duration-200 group-focus-within:opacity-100"
          onmousedown={handleFileUpload}>
          {#if isLoading}
            <Spinner size={22} />
          {:else}
            <Plus size={17} class="mt-[1px]" />
          {/if}
        </div>
        <input
          type="text"
          placeholder="Insert a link, or just plain text"
          class="font-geist w-full bg-transparent text-[14px] leading-none font-[450] outline-none {isFading
            ? 'motion-opacity-out-0 motion-duration-100'
            : ''} {isPending ? 'text-gray-500' : ''}"
          bind:value={input}
          onkeydown={handleKeydown}
          bind:this={inputElement} />
        {#if input.length > 0}
          <div
            class="font-jetbrains-mono motion-opacity-in-0 motion-blur-in-[2px] motion-duration-300 rounded-sm bg-gray-100 px-2 py-0.5 text-[13px] font-medium text-gray-500">
            ENTER
          </div>
        {/if}
      </div>
      <div
        class="flex h-0 items-center gap-2 transition-all duration-200 {showTagSuggestions
          ? 'h-6'
          : 'h-0'}">
        {#each filteredTags as tag}
          <button
            tabindex="0"
            class="font-jetbrains-mono motion-opacity-in-0 -motion-translate-y-in-[10%] motion-duration-300 cursor-pointer rounded-sm bg-[#F1F1F1] px-1.5 py-0.5 text-[15px] font-medium text-[#787879] hover:bg-[#e2e2e2]"
            onclick={() => handleTagClick(tag)}>
            {tag}
          </button>
        {/each}
      </div>
      <div
        class="font-geist-mono ml-[3px] flex items-center justify-between border-b border-gray-200 py-3 text-sm text-gray-400">
        <p class="font-sf-pro-text font-[450]">Title</p>
        <p class="font-sf-pro-text font-[450]">Date</p>
      </div>
      <div class="ml-[3px] flex flex-col gap-3">
        {#if filteredBookmarks.length === 0}
          <p class="font-sf-pro-rounded text-left font-medium text-gray-400">
            how empty...
          </p>
        {/if}
        {#each filteredBookmarks as bookmark, index (bookmark.id)}
          <Bookmark
            onAddTag={addTag}
            onRemoveTag={removeTag}
            bookmark={bookmark}
            onDeleteBookmark={deleteBookmark}
            onEditBookmark={editBookmark}
            index={index} />
        {/each}
      </div>
    </div>
  </div>
</main>
