<script lang="ts">
  let { bookmark } = $props<{ bookmark: Bookmark }>();
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
</script>

<div class="motion-opacity-in-0 motion-duration-200 flex items-center gap-2">
  <img src={bookmark.favicon} alt={bookmark.title} class="h-4.5 w-4.5" />
  <a href={bookmark.url} class="font-geist font-medium">{bookmark.title}</a>
  <p class="font-jetbrains-mono text-sm text-gray-400">
    {formattedUrl}
  </p>
  <p class="font-jetbrains-mono ml-auto text-sm text-gray-400">
    {formattedDate}
  </p>
</div>
