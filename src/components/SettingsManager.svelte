<script lang="ts">
  import { createEventDispatcher } from "svelte";
  import { fade } from "svelte/transition";
  import Close from "./svg/Close.svelte";
  import Spinner from "./svg/Spinner.svelte";
  import Toggle from "./common/Toggle.svelte";
  import { settings } from "../store";
  import { get } from "svelte/store";
  import Import from "./svg/Import.svelte";
  import Export from "./svg/Export.svelte";
  import { migrateSettings } from "../store/settings";
  const { ipcRenderer } = require("electron");
  export let visible = false;

  const dispatchEvent = createEventDispatcher();

  let isSavingSettings = false;
  let currentSettings = get(settings);

  const saveSettings = () => {
    isSavingSettings = true;
    $settings = currentSettings;
    isSavingSettings = false;
    closeSettingsManager();
  };

  const importSettings = () => ipcRenderer.send("import-settings");
  const exportSettings = () => ipcRenderer.send("export-settings", $settings);

  ipcRenderer.on("import-settings", (e, imported) => {
    if (imported.settings) {
      $settings = migrateSettings(imported.settings);
      currentSettings = get(settings);
    } else {
      $settings = imported;
      currentSettings = get(settings);
    }
  });

  const closeSettingsManager = () => {
    dispatchEvent("close-settings-manager");
  };
</script>

{#if visible}
  <div class="modal-container" transition:fade={{ duration: 100 }}>
    <div class="modal">
      <div class="header">
        <div class="title">Settings</div>
        <button class="close" on:click={() => closeSettingsManager()}>
          <Close />
        </button>
      </div>
      <div class="settings">
        {#each Object.keys(currentSettings) as key}
          <div class="setting">
            <div class="info">
              <label class="name" for={key}>{currentSettings[key].name}</label>
              <div class="description">{currentSettings[key].description}</div>
            </div>
            <Toggle id={key} bind:value={currentSettings[key].value} />
          </div>
        {/each}
      </div>
      <div class="controls">
        <button on:click={saveSettings} class:spinning={isSavingSettings}>
          {#if isSavingSettings}
            <Spinner />
          {:else}
            Save
          {/if}
        </button>
        <button
          title="Import Settings"
          class="outlined"
          on:click={importSettings}>
          <Import />
        </button>
        <button
          title="Export Settings"
          class="outlined"
          on:click={exportSettings}>
          <Export />
        </button>
      </div>
    </div>
    <div class="overlay" on:click={() => closeSettingsManager()} />
  </div>
{/if}

<style>
  .modal-container {
    z-index: 3;
  }
  .modal {
    width: max(385px, 35vw);
    height: max(400px, 60vh);
  }
  .settings {
    flex-grow: 1;
    overflow-y: auto;
    overflow-x: hidden;
    margin-bottom: 0.75rem;
  }
  .settings::-webkit-scrollbar {
    width: 5px;
  }
  .settings::-webkit-scrollbar-thumb {
    background: #36475d;
  }
  .setting {
    display: flex;
    justify-content: space-between;
    padding: 0.1rem 0.45rem 0 0rem;
    width: 100%;
    margin-bottom: 1rem;
  }
  .info {
    width: 70%;
  }
  .name {
    display: block;
    font-size: 1.25rem;
    font-weight: 550;
    margin-bottom: 0.35rem;
  }
  .description {
    font-size: 0.9rem;
    font-weight: 300;
  }
  .controls {
    display: flex;
  }
  .controls > * {
    margin-right: 0.5rem;
  }
</style>
