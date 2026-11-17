<script lang="ts">
    import AlertCircleOutline from "svelte-material-icons/AlertCircleOutline.svelte";
    import AsteriskCircleOutline from "svelte-material-icons/AsteriskCircleOutline.svelte";
    import InformationOutline from "svelte-material-icons/InformationOutline.svelte";
    import AlertOutline from "svelte-material-icons/AlertOutline.svelte";

    import RichText from "../../../assets/RichText.svelte";
    import PrereqNode from "./PrereqNode.svelte";
    import { parsePrerequisites } from "./prerequisites";
    import { onMount } from "svelte";

    export let content = "";

    let showOriginal: boolean = false;
    let init: boolean = false;

    $: init && localStorage.setItem("show-original-prereqs", showOriginal ? "true" : "false");

    onMount(() => {
        showOriginal = localStorage.getItem("show-original-prereqs") == "true";
        init = true;
    });

    $: parsedContent = parsePrerequisites(content);
</script>

<div class="prerequisites">
    {#if showOriginal}
        <div class="sectionTitle">
            <div class="icon">
                <AsteriskCircleOutline></AsteriskCircleOutline>
            </div>
            <p class="text">Prerequisites</p>
        </div>

        <div class="section originalInfo">
            <InformationOutline size="24px"></InformationOutline>
            <p>Showing original prerequisites. <button on:click={() => { showOriginal = !showOriginal; }}>show expanded prerequisites</button></p>
        </div>

        <div class="section originalPrereqs sectionRichText">
            <RichText content={content} />
        </div>
    {/if}

    {#if showOriginal == false}
        <div class="section expandedAlert">
            <AlertOutline size="24px"></AlertOutline>
            <p>Always double-check with original prerequisites. Our expanded prerequisites may not be 100% accurate. <button on:click={() => { showOriginal = !showOriginal; }}>show original prerequisites</button></p>
        </div>
    {/if}

    {#if parsedContent.prerequisites != null && showOriginal == false}
        <div class="sectionTitle">
            <div class="icon">
                <AsteriskCircleOutline></AsteriskCircleOutline>
            </div>
            <p class="text">Prerequisites</p>
        </div>

        <PrereqNode node={parsedContent.prerequisites}></PrereqNode>
    {/if}

    {#if parsedContent.restrictions.length != 0 && showOriginal == false}
        <div class="sectionTitle">
            <div class="icon">
                <AlertCircleOutline></AlertCircleOutline>
            </div>
            <p class="text">Restrictions</p>
        </div>
        
        <div class="section sectionRichText">
            <RichText content={parsedContent.restrictions.join("\n\n")} />
        </div>
    {/if}
</div>

<style>
    .prerequisites {
        position: relative;
    }

    .originalInfo {
        display: flex;
        align-items: center;
        gap: 12px;
        padding: 8px 4px 4px 8px;
        background-color: #0000002d;
        border-radius: 8px;
        margin-bottom: 3px;
        border-radius: 8px 8px 4px 4px !important;
    }

    .originalInfo button {
        background: none !important;
        border: none !important;
        margin: 0;
        padding: 0;
        text-decoration: underline;
        color: inherit;
    }

    .originalPrereqs {
        border-radius: 4px 4px 8px 8px !important;
    }

    .originalInfo p {
        margin: 0;
    }

    .expandedAlert {
        display: flex;
        align-items: center;
        gap: 12px;
        padding: 8px 4px 4px 8px;
        background-color: #ffc171 !important;
        color: black;
        border-radius: 8px;
        margin-top: 24px;
    }

    .expandedAlert button {
        background: none !important;
        border: none !important;
        margin: 0;
        padding: 0;
        text-decoration: underline;
        color: inherit;
        font-weight: 800;
    }

    .expandedAlert p {
        margin: 0;
        line-height: 1.5em;
    }

    p {
        margin-top: 5px;
        line-height: 1.75em;
        white-space: pre-line;
    }

    .sectionTitle {
        display: flex; 
        align-items: center;
        gap: 4px;
        margin-top: 16px;
        margin-bottom: 4px;
    }
    .sectionTitle .icon {
        display: flex;
        align-items: center;
        font-size: 18px
    }
    .sectionTitle .text {
        margin: 0px;
        font-size: 16px;
        font-weight: 600;
    }
    .section {
        background-color: #0000002d;
        padding: 16px 20px 16px 20px;
        border-radius: 8px;
    }
</style>