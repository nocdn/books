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

  let bookmarks = $state<Bookmark[]>([]);

  async function fetchBookmarks() {
    const response = await fetch("http://localhost:5570/api/list");
    const data = await response.json();
    console.log("fetched bookmark", data);
    bookmarks = data.bookmarks;
  }

  fetchBookmarks();

  async function deleteBookmark(id: number) {
    const response = await fetch(`http://localhost:5570/api/delete/${id}`, {
      method: "DELETE",
    });
    const data = await response.json();
    console.log(data);
    fetchBookmarks();
  }

  type submittedBookmark = {
    url: string;
    tags: string[];
    createdAt: string;
  };

  async function handleCreateBookmark(bookmark: submittedBookmark) {
    const response = await fetch("http://localhost:5570/api/create", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(bookmark),
    });
    const data = await response.json();
    console.log(data);
    fetchBookmarks();
  }
</script>

<main>
  <Navbar onGroupChange={handleGroupChange} />
  <div class="mt-24 grid w-full place-items-center">
    <Bookmarks
      bookmarks={bookmarks}
      onCreateBookmark={handleCreateBookmark}
      onDeleteBookmark={deleteBookmark} />
  </div>
</main>
