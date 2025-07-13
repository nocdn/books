<script lang="ts">
  import Navbar from "./Navbar.svelte";
  import Bookmarks from "./Bookmarks.svelte";

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

  async function fetchBookmarks() {
    if (!backendAddress) return;
    const response = await fetch(`${backendAddress}/api/list`);
    const data = await response.json();
    console.log("fetched bookmark", data);
    bookmarks = data.bookmarks;
  }

  $effect(() => {
    fetchBookmarks();
  });

  async function deleteBookmark(id: number) {
    const response = await fetch(`${backendAddress}/api/delete/${id}`, {
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
    const response = await fetch(`${backendAddress}/api/create`, {
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

  async function editBookmark(id: number, title: string, url: string) {
    const response = await fetch(`${backendAddress}/api/update/${id}`, {
      method: "PUT",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ title, url }),
    });
    const data = await response.json();
    console.log(data);
    fetchBookmarks();
  }
</script>

<main>
  <Navbar
    onGroupChange={handleGroupChange}
    onBackendChange={handleBackendChange}
    backendAddress={backendAddress} />
  <div class="mt-24 grid w-full place-items-center">
    <Bookmarks
      bookmarks={bookmarks}
      onCreateBookmark={handleCreateBookmark}
      onDeleteBookmark={deleteBookmark}
      onEditBookmark={editBookmark} />
  </div>
</main>
