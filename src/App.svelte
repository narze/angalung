<script lang="ts">
  import logo from "./assets/svelte.png"
  import Counter from "./lib/Counter.svelte"

  import { appWindow } from "@tauri-apps/api/window"
  import { onMount } from "svelte"
  import { io } from "socket.io-client"

  let messages = []
  let autoread = false
  let clicked = false

  onMount(() => {
    const socket = io("ws://streamie.narze.live")

    socket.on("message", ({ message, username, language }) => {
      messages = [...messages, `${username}: ${message} (${language ?? "th"})`]

      if (autoread) {
        read(message, language)
      }
    })
  })

  onMount(() => {
    document
      .getElementById("titlebar-minimize")
      .addEventListener("click", () => appWindow.minimize())
    document
      .getElementById("titlebar-maximize")
      .addEventListener("click", () => appWindow.toggleMaximize())
    document
      .getElementById("titlebar-close")
      .addEventListener("click", () => appWindow.close())
  })

  function read(message: string, language: string = "th") {
    if (!canRead) {
      return
    }

    new Audio(
      `https://tts-api.vercel.app/api/tts?text=${message}&lang=${language}`
    ).play()
  }

  function canRead(message: string): boolean {
    return message.length <= 200
  }
</script>

<div data-tauri-drag-region class="titlebar">
  <div class="titlebar-button" id="titlebar-minimize">
    <img
      src="https://api.iconify.design/mdi:window-minimize.svg"
      alt="minimize"
    />
  </div>
  <div class="titlebar-button" id="titlebar-maximize">
    <img
      src="https://api.iconify.design/mdi:window-maximize.svg"
      alt="maximize"
    />
  </div>
  <div class="titlebar-button" id="titlebar-close">
    <img src="https://api.iconify.design/mdi:close.svg" alt="close" />
  </div>
</div>

<main>
  <h1 class="text-3xl">AUNGALUNG</h1>

  <p>พิมพ์ !say ตามด้วยข้อความใน Twitch</p>
  <p>
    {#if !clicked}
      <button on:click={() => (clicked = autoread = true)} class="btn btn-sm">
        Click me to activate autoreading
      </button>
    {:else}
      <button on:click={() => (autoread = !autoread)} class="btn btn-sm">
        Autoread : {autoread ? "🔊" : "🔇"}
      </button>
    {/if}
  </p>

  {#each messages as message}
    <p>
      {message}
      <button on:click={() => read(message)}>🔉</button>
    </p>
  {/each}
</main>

<style>
  :root {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
      Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
  }

  main {
    text-align: center;
    padding: 1em;
    margin: 2rem auto;
  }

  img.logo {
    height: 16rem;
    width: 16rem;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4rem;
    font-weight: 100;
    line-height: 1.1;
    margin: 2rem auto;
    max-width: 14rem;
  }

  p {
    max-width: 14rem;
    margin: 1rem auto;
    line-height: 1.35;
  }

  @media (min-width: 480px) {
    h1 {
      max-width: none;
    }

    p {
      max-width: none;
    }
  }

  .titlebar {
    height: 30px;
    background: #329ea3;
    user-select: none;
    display: flex;
    justify-content: flex-end;
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
  }
  .titlebar-button {
    display: inline-flex;
    justify-content: center;
    align-items: center;
    width: 30px;
    height: 30px;
  }
  .titlebar-button:hover {
    background: #5bbec3;
  }
</style>
