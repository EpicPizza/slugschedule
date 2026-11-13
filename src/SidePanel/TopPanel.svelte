<script lang="ts" context="module">
    export function prettyTerm(term: number) {
        switch(term % 10) {
            case 0:
                return "❄ 20" + term.toString().slice(1, 3)
            case 2:
                return "🌸 20" + term.toString().slice(1, 3)
            case 4:
                return "☀ 20" + term.toString().slice(1, 3)
            case 8:
                return "🍁 20" + term.toString().slice(1, 3)
        }
    }
</script>
<script lang="ts">
    import SlugSchedule from "../assets/slugschedule.svg";
    import Star from "svelte-material-icons/Star.svelte";
    import Magnify from "svelte-material-icons/Magnify.svelte";
    import Harddisk from "svelte-material-icons/Harddisk.svelte";
    import Calendar from "svelte-material-icons/Calendar.svelte";
    import { db, focusedClass, listMode } from "../mainStore";
    import TermMenu from "./TermMenu.svelte";

    let termMenuOpen = false;

    $: term = prettyTerm($db.term);
</script>
<div class="topRow">
    <button class="logo" on:click={() => { $focusedClass = "home"; $listMode = "all"}}>
        <img src={SlugSchedule} alt="Slug Schedule Logo. Yellow Slug in front of Calendar."/>
    </button>
    <div class="title">
        <button on:click={() => { $focusedClass = "home"; $listMode = "all"}}>
            <h2>SlugSchedule</h2>
        </button>
        <button on:click={() => termMenuOpen = !termMenuOpen} class:active={termMenuOpen} class="term" style="text-align: center">{term} ({$db.term})</button>
    </div>
    {#if termMenuOpen}
        <TermMenu bind:term={term} bind:termMenuOpen={termMenuOpen} />
    {/if}
</div>
<div class="tabRow">
    <button title="Search all classes" on:click={() => $listMode = "all"} class="iconBtn" class:active={$listMode === "all"}>
        <div>
            <Harddisk size="2em" />
            <span>Catalog</span>
        </div>
    </button>
    <button title="Starred classes" on:click={() => $listMode = "starred"} class="iconBtn" class:active={$listMode === "starred"}>
        <div>
            <Star size="2em" />
            <span>Starred</span>
        </div>
    </button>
    <button title="Schedule classes" on:click={() => $listMode = "scheduler"} class="iconBtn" class:active={$listMode === "scheduler"}>
        <div>
            <Calendar size="2em" />
            <span>Schedule</span>
        </div>
    </button>
    <!--
    <button title="Smart class search" on:click={() => $listMode = "smart"} class="iconBtn smart" class:active={$listMode === "smart"}>
        <AutoFix size="2em" />
    </button>
    -->
</div>

<style>
    .topRow {
        position: relative;
        flex-shrink: 0;
        height: 72px;
        display: flex;
        flex-direction: row;
        gap: 10px;
        align-items: center;
        padding: 10px;
        padding-left: 15px;
        padding-bottom: 5px;
    }
    .title {
        display: flex;
        flex-direction: column;
        gap: 4px;
        align-items: start;
        width: 100%;
    }
    .title > button > h2 {
        margin: 0px;
        font-size: 1.4em;
        margin-bottom: 0px;
    }
    .title > button {
        background-color: transparent;
        border: none;
        padding: 0px !important;
        margin: 0px !important;
        font-size: 1rem;
    }
    .tabRow {
        width: 390px;
        flex-shrink: 0;
        height: 54px;
        display: flex;
        flex-direction: row;
        align-items: center;
        background-color: var(--gray-950);
        padding: 0px 5px 0px 5px;
        border-bottom: 2px solid var(--gray-600);
    }
    .tabRow > *:not(.indicator):not(.term):not(span) {
        border-radius: 10px;
        width: 62px;
        height: 62px;
        overflow: hidden;
        background-color: var(--gray-600);
        user-select: none;
        cursor: pointer;
    }
    .iconBtn {
        padding: 0px !important;
        width: 100% !important;
        padding: 0px !important;
        height: 100% !important;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        border-radius: 10px 10px 0px 0px !important;
        background-color: var(--gray-950) !important;
        border: #00000000 solid 2px;
        color: #ddd;
        
        transition: border-color 0.1s ease-in-out, transform 0.1s ease-in-out, border-radius 0.1s ease-in-out, color 0.1s ease-in-out;
    }
    .iconBtn:hover {
        background-color: var(--gray-800) !important;
        transition: background-color 0.1s ease-in-out, transform 0.1s ease-in-out, border-radius 0.1s ease-in-out, color 0.1s ease-in-out;
    }
    .iconBtn > div {
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: center;
        gap: 4px;
        font-size: 0.9em;
    }
    .logo {
        padding: 0px;
        background-color: transparent !important;
        border-width: 0;
        transition: filter 0.1s ease-in-out, transform 0.1s ease-in-out;
        outline: none;
    }
    .logo:hover, .logo:focus-visible {
        filter: hue-rotate(-55deg);
        transform: scale(1.1);
    }
    .iconBtn > div > span {
        font-size: 0.9em;
        font-weight: 800;
        padding-right: 2px;
    }
    .iconBtn:focus:not(:focus-visible) {
        outline: none;
    }
    .iconBtn:focus-visible {
        outline: 2px solid var(--primary);
    }
    .iconBtn.active {
        color: var(--primary);
        background-color: var(--gray-800) !important;
        border: var(--gray-600) solid 2px;
        border-bottom: #00000000 solid 2px;
        transform: translateY(2px);
    }
    .term {
        background-color: transparent;
        color: unset;
        border: none;
        font-weight: unset;
        font-size: unset;
        padding: 0px;
    }
    .logo > img {
        width: 60px;
    }
    .term:focus:not(:focus-visible) {
        outline: none;
    }
    .term:hover, .term.active, .term:focus-visible {
        font-weight: 600;
        text-shadow:#444;
    }

    .grower:not(.mobile) {
        flex-grow: 1;
    }

    @media screen and (max-width: 1000px) {
        .tabRow {
            width: calc(100% - 10px);
        }

        .topRow {
            height: 40px;
        }

        .logo > img {
            width: 30px;
        }

        .title > button > h2 {
            font-size: 1.2em;
            padding-bottom: 3px;
        }

        .title {
            display: flex;
            justify-content: space-between;
            flex-direction: row;
            width: 100%;
        }
    }
</style>