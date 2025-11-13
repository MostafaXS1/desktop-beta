<script>
  import DesktopIcons from "./desktop-icons.svelte";
  import vscode from "../assets/vscode.png";
  import chrome from "../assets/chrome.png";
  import photopea from "../assets/photopea.png";
  import linkedin from "../assets/linkedin.png";
  import explorer from "../assets/explorer.png";
  import nes from "../assets/nes.png";
  import wikipedia from "../assets/wikipedia.png";
  import Window from "./window.svelte";

  let windows = $state([]);
  let lastId = 0;
</script>

{#each windows as w (w.id)}
  <Window
    url={w.url}
    title={w.title}
    icon={w.icon}
    posx={w.posx}
    posy={w.posy}
    width={w.width}
    height={w.height}
    on:close={() => (windows = windows.filter((x) => x.id !== w.id))}
  />
{/each}

<main>
  <div class="desktop-grid">
    <div class="icon-wrapper">
      <DesktopIcons src={wikipedia} name="Wikipedia" />
      <button
        type="button"
        aria-label="wikipedia-desktop-icon"
        class="icon-cover"
        onclick={() =>
          (windows = [
            ...windows,
            {
              id: ++lastId,
              icon: wikipedia,
              width: 800,
              height: 900,
              posx: 560,
              posy: 85,
              url: "https://www.wikipedia.org/",
              title: "Wikipedia",
            },
          ])}
      ></button>
    </div>
    <div class="icon-wrapper">
      <DesktopIcons src={nes} name="Games" />
      <button
        type="button"
        aria-label="games-desktop-icon"
        class="icon-cover"
        onclick={() =>
          (windows = [
            ...windows,
            {
              id: ++lastId,
              icon: nes,
              width: 1200,
              height: 800,
              posx: 360,
              posy: 110,
              url: "https://emu-sepia.vercel.app/",
              title: "Games",
            },
          ])}
      ></button>
    </div>
    <div class="icon-wrapper">
      <DesktopIcons src={photopea} name="Photopea" />
      <button
        type="button"
        aria-label="photopea-desktop-icon"
        class="icon-cover"
        onclick={() =>
          (windows = [
            ...windows,
            {
              id: ++lastId,
              icon: photopea,
              width: 1200,
              height: 800,
              posx: 360,
              posy: 110,
              url: "https://photopea.com/",
              title: "Photopea",
            },
          ])}
      ></button>
    </div>
  </div>
</main>

<style>
  .desktop-grid {
    height: calc(100vh - 46px);
    width: 100%;
    padding: 8px;
    display: flex;
    flex-direction: column;
    flex-wrap: wrap;
    align-content: flex-start;
    box-sizing: border-box;
  }
  .icon-wrapper {
    position: relative;
    display: inline-block;
  }
  .icon-cover {
    position: absolute;
    inset: 0;
    width: 85px;
    height: 100%;
    background: transparent;
    border: none;
    padding: auto;
    margin: auto;
    cursor: pointer;
    user-select: none;
    z-index: 3;
    pointer-events: auto;
  }
  .icon-cover:hover {
    background-color: #a7a7a760;
    border-radius: 6px;
  }
</style>
