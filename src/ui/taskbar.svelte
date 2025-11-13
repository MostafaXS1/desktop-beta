<script>
  import { fly } from "svelte/transition";
  import { onMount, onDestroy } from "svelte";
  import tippy from "tippy.js";
  import "tippy.js/dist/tippy.css";

  import chrome from "../assets/chrome.png";
  import linkedin from "../assets/linkedin.png";
  import vscode from "../assets/vscode.png";
  import explorer from "../assets/explorer.png";
  import photopea from "../assets/photopea.png";
  import start from "../assets/start.jpg";
  import Window from "./window.svelte";

  let time = $state(new Date().toLocaleTimeString());
  let date = $state(
    new Intl.DateTimeFormat("en-US", {
      weekday: "short",
      month: "short",
      day: "numeric",
    }).format(new Date())
  );

  const timer = setInterval(() => {
    time = new Date().toLocaleTimeString();
    date = new Intl.DateTimeFormat("en-US", {
      weekday: "short",
      month: "short",
      day: "numeric",
    }).format(new Date());
  }, 100);

  let instances = [];

  onMount(() => {
    instances = tippy(".taskbar-button", {
      delay: [750, 0],
      duration: [200, 200],
      animation: "fade",
      allowHTML: false,
      offset: [0, 8],
      arrow: false,
    });
  });

  onDestroy(() => {
    clearInterval(timer);
    instances.forEach((inst) => inst.destroy && inst.destroy());
  });

  let startmenuvisible = $state(false);
  let langmenuvisible = $state(false);
  let timemenuvisible = $state(false);

  function handleStart() {
    startmenuvisible = !startmenuvisible;
    if (langmenuvisible) {
      langmenuvisible = !langmenuvisible;
    }
    if (timemenuvisible) {
      timemenuvisible = !timemenuvisible;
    }
  }
  function handleLang() {
    langmenuvisible = !langmenuvisible;
    if (startmenuvisible) {
      startmenuvisible = !startmenuvisible;
    }
    if (timemenuvisible) {
      timemenuvisible = !timemenuvisible;
    }
  }
  function handleTime() {
    timemenuvisible = !timemenuvisible;
    if (startmenuvisible) {
      startmenuvisible = !startmenuvisible;
    }
    if (langmenuvisible) {
      langmenuvisible = !langmenuvisible;
    }
  }

  export function focusDesktop() {
    if (startmenuvisible) {
      startmenuvisible = !startmenuvisible;
    }
    if (langmenuvisible) {
      langmenuvisible = !langmenuvisible;
    }
    if (timemenuvisible) {
      timemenuvisible = !timemenuvisible;
    }
  }

  let windows = $state([]);
  let lastId = 0;
</script>

{#if startmenuvisible}
  <div
    in:fly={{ y: 1200, duration: 300 }}
    out:fly={{ y: 1200, duration: 300 }}
    class="start"
  >
    Working on it ¯\_(ツ)_/¯<br />Working on it ¯\_(ツ)_/¯<br />Working on it
    ¯\_(ツ)_/¯<br />Working on it ¯\_(ツ)_/¯<br />Working on it ¯\_(ツ)_/¯<br
    />Working on it ¯\_(ツ)_/¯<br />Working on it ¯\_(ツ)_/¯<br />Working on it
    ¯\_(ツ)_/¯<br />Working on it ¯\_(ツ)_/¯<br />Working on it ¯\_(ツ)_/¯<br
    />Working on it ¯\_(ツ)_/¯<br />Working on it ¯\_(ツ)_/¯<br />Working on it
    ¯\_(ツ)_/¯<br />Working on it ¯\_(ツ)_/¯<br/>Working on it ¯\_(ツ)_/¯<br/>Working on it ¯\_(ツ)_/¯<br/>Working on it ¯\_(ツ)_/¯<br/>Working on it ¯\_(ツ)_/¯<br/>Working on it ¯\_(ツ)_/¯<br/>Working on it ¯\_(ツ)_/¯<br/>Working on it ¯\_(ツ)_/¯<br/>Working on it ¯\_(ツ)_/¯<br/>Working on it ¯\_(ツ)_/¯<br />Working on it ¯\_(ツ)_/¯<br
    />Working on it ¯\_(ツ)_/¯
  </div>
{/if}

{#if langmenuvisible}
  <div
    in:fly={{ y: 200, duration: 200 }}
    out:fly={{ y: 200, duration: 200 }}
    class="lang-menu"
  >
    Working on it too ツ
  </div>
{/if}

{#if timemenuvisible}
  <div
    in:fly={{ y: 200, duration: 200 }}
    out:fly={{ y: 200, duration: 200 }}
    class="lang-menu"
  >
    Working on it three :3
  </div>
{/if}

{#each windows as w (w.id)}
  <Window url={w.url} title={w.title} icon={w.icon} posx={w.posx} posy={w.posy} width={w.width} height={w.height} on:close={() => (windows = windows.filter(x => x.id !== w.id))} />
{/each}

<nav class="bottom-nav">
  <button
    class="taskbar-button"
    aria-label="start"
    data-tippy-content="Start"
    style="margin-left: 4px"
    onclick={handleStart}
  >
    <img src={start} width="24px" alt="start-icon" />
  </button>

  <button
    class="taskbar-button"
    aria-label="chrome"
    data-tippy-content="Chromium"
>
    <img src={chrome} width="24px" alt="chrome-icon" />
  </button>

  <button
    class="taskbar-button"
    aria-label="vscode"
    data-tippy-content="VSCode"
  >
    <img src={vscode} width="24px" alt="vscode-icon" />
  </button>

  <button
    class="taskbar-button"
    aria-label="explorer"
    data-tippy-content="Explorer"
  >
    <img src={explorer} width="24px" alt="explorer-icon" />
  </button>

  <button
    class="taskbar-button"
    aria-label="photopea"
    data-tippy-content="photopea"
    onclick={() => (windows = [...windows, { id: ++lastId, icon: photopea, width: 1200, height: 800, posx: 360, posy: 110, url: "https://photopea.com/", title: "Photopea" }])}
  >
    <img src={photopea} width="24px" alt="photopea-icon" />
  </button>

  <button
    class="taskbar-button"
    aria-label="linkedin"
    data-tippy-content="LinkedIn"
  >
    <img src={linkedin} width="24px" alt="linkedin-icon" />
  </button>
  <div class="spacer"></div>
  <button class="lang-hitbox" onclick={handleLang}
    ><span class="lang">ENG</span></button
  >
  <button class="lang-hitbox" onclick={handleTime}
    ><span class="time">{time}<br />{date}</span></button
  >
</nav>

<style>
  nav.bottom-nav {
    height: 46px;
    border-top: 1px solid #404040;
    position: fixed;
    left: 0;
    right: 0;
    bottom: 0;
    display: flex;
    justify-content: left;
    gap: 12px;
    padding: 0px;
    background: #1c1c1c;
    box-shadow: 0 -1px 4px rgba(0, 0, 0, 0.08);
    z-index: 9999;
  }
  button.taskbar-button {
    margin-top: auto;
    margin-bottom: auto;
    width: 40px;
    height: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    margin-left: -5px;
    margin-right: -5px;
    padding: 8px;
    border-radius: 6px;
    border: 1px solid transparent;
    background: transparent;
    cursor: pointer;
  }
  button.taskbar-button:hover {
    transition: 0.1s ease-in;
    background-color: #292929;
  }
  button.taskbar-button:focus {
    outline: none;
    border-color: #999;
  }
  .time {
    font-weight: 499;
    font-size: 12px;
    margin: auto 12px auto 0;
    margin-left: auto;
    text-align: right;
    color: white;
    padding: 5px;
    border-radius: 6px;
    cursor: pointer;
  }
  .time:hover {
    transition: 0.1s ease-in;
    background-color: #292929;
  }
  .lang {
    font-weight: 499;
    font-size: 12px;
    margin: auto 12px auto 0;
    color: white;
    padding-top: 14px;
    padding-bottom: 14px;
    padding-left: 8px;
    padding-right: 8px;
    margin-right: -6px;
    border-radius: 6px;
    cursor: pointer;
  }
  .lang:hover {
    transition: 0.1s ease-in;
    background-color: #292929;
  }
  .lang-hitbox {
    color: transparent;
    background-color: transparent;
    margin: 0px;
    padding: 0px;
    border: none;
    display: flex;
    align-items: flex-start;
    justify-content: center;
  }
  .spacer {
    flex-grow: 1;
  }
  .start {
    position: fixed;
    bottom: 56px;
    left: 10px;
    width: 600px;
    color: white;
    background: #242424;
    border-radius: 6px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
    z-index: 1000;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .lang-menu {
    position: fixed;
    bottom: 56px;
    right: 10px;
    width: 200px;
    height: 100px;
    color: white;
    background: #242424;
    border-radius: 6px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
    z-index: 1000;
    display: flex;
    align-items: center;
    justify-content: center;
  }
</style>
