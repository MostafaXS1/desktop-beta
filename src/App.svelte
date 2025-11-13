<script>
  import vscode from "./assets/vscode.png"
  import Taskbar from "./ui/taskbar.svelte";
  import Desktop from "./ui/desktop.svelte";
  import Window from "./ui/window.svelte";
  import wallpaper from "./assets/wallpaper.jpg";
</script>

<main style={`background-image: url(${wallpaper});`}>
  <Taskbar />
  <Desktop />
  <!-- <Window icon={vscode} url="https://svelte.dev" title="Svelte Docs" /> -->
</main>

<style>
  main {
    position: relative;
    width: 100vw;
    height: 100vh;
    background-size: cover;
    background-position: center;
    margin: 0;
    overflow: hidden;
  }
</style>
