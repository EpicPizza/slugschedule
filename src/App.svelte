<script lang="ts">
  import ListPanel from "./ListPanel/ListPanel.svelte";
  import Loading from "./Loading/Loading.svelte";
  import MainPanel from "./MainPanel/MainPanel.svelte";
  import BottomPanel, { prettyTerm } from "./SidePanel/BottomPanel.svelte";
  import TopPanel from "./SidePanel/TopPanel.svelte";
  import { db } from "./mainStore";
  import TermMenu from "./SidePanel/TermMenu.svelte";

  let termMenuOpen = false;

  $: term = $db ? prettyTerm($db.term) : "";
</script>

{#if !$db}
<Loading />
{:else}
<div class="app">
  <div class="sidePanel">
    <BottomPanel />
    <ListPanel />
  </div>
  <div class="mainPanel">
    <MainPanel />
  </div>
</div>
{/if}

<style>
  .app {
    display: flex;
    flex-direction: row;
    width: 100%;
    height: 100dvh;
    overflow: hidden;
  }

  .sidePanel {
    display: flex;
    flex-direction: column;
    max-height: 100%;
    overflow: hidden;
    min-width: 350px;
    max-width: 350px;
  }

  .mainPanel {
    display: flex;
    flex-direction: column;
    overflow: hidden;
  }

  @media screen and (max-width: 1000px) {
    div {
      flex-direction: column;
    }

    .sidePanel {
      min-width: 100%;
      max-width: 100%;
    }
  }
</style>