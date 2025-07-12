<script lang="ts">
  import Navbar from "./Navbar.svelte";
  import Bookmarks from "./Bookmarks.svelte";
  function handleGroupChange(group: string) {
    console.log(group);
  }

  type Bookmark = {
    id: number;
    title: string;
    url: string;
    tags: string[];
    favicon: string;
    createdAt: string;
  };

  let bookmarks = $state<Bookmark[]>([
    {
      id: 1,
      title: "Google",
      url: "https://www.google.com",
      tags: ["search", "web"],
      favicon: "https://www.google.com/s2/favicons?domain=google.com&sz=128",
      createdAt: "2025-07-11T20:14:00+01:00",
    },
  ]);

  const secondBookmark = {
    id: 2,
    title: "Svelte",
    url: "https://svelte.dev",
    tags: ["framework", "web"],
    favicon: "https://www.google.com/s2/favicons?domain=svelte.dev&sz=128",
    createdAt: "2025-07-10T10:00:00+01:00",
  };

  setTimeout(() => {
    bookmarks = [...bookmarks, secondBookmark];
  }, 1000);

  async function handleCreateBookmark(bookmark: Bookmark) {}
</script>

<main>
  <Navbar onGroupChange={handleGroupChange} />
  <div class="mt-24 grid w-full place-items-center">
    <Bookmarks bookmarks={bookmarks} onCreateBookmark={handleCreateBookmark} />
  </div>
</main>
