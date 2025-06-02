<script lang="ts">
  import { Button } from "$lib/components/ui/button";
  import { Github } from "@lucide/svelte";
  import "../app.css";

  let { children } = $props();
</script>

<svelte:head>
  <title>Unity Package Manifest Generator</title>
</svelte:head>

<main class="bg-background min-h-dvh pt-4 pb-12 dark space-y-3">
  <header class="mx-4">
    <Button
      variant="link"
      onclick={() =>
        window.open(
          "https://github.com/2394425147/unity-package-manifest-generator"
        )}
    >
      <Github />
      GitHub
    </Button>
  </header>
  <div id="content" class="mx-4 lg:not-3xl:mx-auto not-3xl:max-w-5xl">
    {@render children()}
  </div>
</main>

<style>
  /* inter-latin-wght-normal */
  @font-face {
    font-family: "Inter Variable";
    font-style: normal;
    font-display: swap;
    font-weight: 100 900;
    src: url(https://cdn.jsdelivr.net/fontsource/fonts/inter:vf@latest/latin-wght-normal.woff2)
      format("woff2-variations");
    unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6, U+02DA,
      U+02DC, U+0304, U+0308, U+0329, U+2000-206F, U+20AC, U+2122, U+2191,
      U+2193, U+2212, U+2215, U+FEFF, U+FFFD;
  }

  main {
    font-family: "Inter Variable", sans-serif;
  }
</style>
