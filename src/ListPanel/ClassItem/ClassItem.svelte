<script context="module" lang="ts">
  export function rmpScoreColor(score: number) {
    if (score >= 4.0) return "rgb(127, 246, 195)";
    if (score >= 3.0) return "rgb(255, 241, 112)";
    return "rgb(255, 156, 156)";
  }
  export function gpaDifficultyColor(difficulty: string) {
    switch (difficulty) {
      case "a breeze":
        return "#00E676";
      case "easy":
        return "#C6FF00";
      case "fine":
        return "var(--primary)";
      case "hard":
        return "#FF6E40";
      case "challenging":
        return "#FF5252";
      default:
        return "white";
    }
  }
</script>

<script lang="ts">
    import Star from "svelte-material-icons/Star.svelte";
    import StarOutline from "svelte-material-icons/StarOutline.svelte";
    import Plus from "svelte-material-icons/Plus.svelte";
    import Minus from "svelte-material-icons/Minus.svelte";
    import MapMarker from "svelte-material-icons/MapMarker.svelte";
    import Monitor from "svelte-material-icons/Monitor.svelte";
    import Account from "svelte-material-icons/Account.svelte";
    import Poll from "svelte-material-icons/Poll.svelte";

    import { db, detectTerm, focusedClass, listMode, scheduledClasses, shouldAnimate, starredClasses, term } from "../../mainStore";
    import { ClassStatus, type Class } from "../../DB/DB";
    import ClassAllocation from "../../assets/ClassAllocation.svelte";
    import DateChecker from "../../assets/DateChecker.svelte";
    import SectionPopup from "./SectionPopup.svelte";
    import { calculateAverageGPA, calculateDifficulty } from "../../MainPanel/Class/GradeDistribution/GradeDistribution.svelte";
  import { onDestroy, onMount } from "svelte";

    export let item: Class;

    $: code = item.code.split(" - ")[0];;
    let showSectionPopup = false;

    function toggleStar(e) {
        e.stopPropagation();
        if ($starredClasses.includes(item.number)) {
            $starredClasses = $starredClasses.filter(x => x !== item.number);
        } else {
            $starredClasses = [...$starredClasses, item.number];
        }
    }

    function toggleScheduled(e) {
        e.stopPropagation();
        if ($scheduledClasses.includes(item.number)) {
            $scheduledClasses = $scheduledClasses.filter(x => ![item.number, ...item.associatedClasses.map(y => y.number)].includes(x));
        } else {
            $scheduledClasses = [...$scheduledClasses, item.number];
            if (item.associatedClasses.length) showSectionPopup = true;
        }
    }

    function focusClass() {
        $shouldAnimate = true;
        if ($listMode === "scheduler") {
            if ($starredClasses.includes(item.number)) {
                $listMode = "starred";
            } else {
                // if the class isn't starred, it won't show up in the starred list...
                // so go to the main course search instead
                $listMode = "all";
            }
        }
        $focusedClass = item;
    }

    // This is sort of a hack, but I'm not sure how to trigger reactivity
    // in a better way here. The object is updated but reactivity is not
    // detected on it.
    let oldAvailability = {...item.availability};
    function checkForUpdate() {
        if (oldAvailability.status !== item.availability.status ||
            oldAvailability.enrolled !== item.availability.enrolled ||
            oldAvailability.capacity !== item.availability.capacity ||
            oldAvailability.waitlist !== item.availability.waitlist ||
            oldAvailability.waitlistCapacity !== item.availability.waitlistCapacity) {
            // availability has changed
            oldAvailability = {...item.availability};
            // trigger reactivity
            item = {...item};
        }
    }

    let updateInterval;
    onMount(() => {
        updateInterval = setInterval(checkForUpdate, 1000);
    })
    onDestroy(() => {
        clearInterval(updateInterval);
    })
</script>

{#if showSectionPopup}
  <SectionPopup {item} close={() => (showSectionPopup = false)} />
{/if}

{#key item.number}
  <div
    class="classItem"
    class:small={detectTerm() !== $db.term}
    class:focused={$focusedClass !== "home" &&
      $listMode !== "scheduler" &&
      $focusedClass?.code === item.code}
    class:open={item.availability.status === ClassStatus.Open}
    class:waitlist={item.availability.status === ClassStatus.Waitlist}
    class:closed={item.availability.status === ClassStatus.Closed}
    role="button"
    tabindex="0"
    on:click={focusClass}
    on:keydown={(e) => {
      if (e.key !== "Tab") focusClass();
    }}
  >
    <div class="topBar">
      <div class="header" title={`${code} - ${item.name}`}>
        <h3>{code}</h3>
        <h2>{item.name}</h2>
      </div>
      <button
        aria-label={$listMode === "scheduler"
          ? "Add/remove class to schedule"
          : "Star this class for later"}
        tabindex={1}
        class="roundBtn"
        on:click={$listMode === "scheduler" ? toggleScheduled : toggleStar}
      >
        {#if $listMode === "scheduler"}
          {#if $scheduledClasses.includes(item.number)}
            <Minus />
          {:else}
            <Plus />
          {/if}
        {:else if $starredClasses.includes(item.number)}
          <Star />
        {:else}
          <StarOutline />
        {/if}
      </button>
    </div>
    <div class="body">
      {#if item.meetingInfos.some((x) => x.location && x.location !== "N/A")}
        {#each item.meetingInfos as meetingInfo}
          <div style="min-width: calc(50% - 10px)" class="location">
            {#if meetingInfo.location === "Online" || meetingInfo.location === "Remote Instruction"}
              <Monitor />
            {:else}
              <MapMarker />
            {/if}
            {meetingInfo.location}
          </div>
        {/each}
      {/if}
      {#if item.meetingInfos.some((x) => x.dayAndTime && x.dayAndTime !== "Cancelled")}
        <div class="dayAndTime">
            <DateChecker background={false} number={item.number} meetingInfos={item.meetingInfos} />
        </div>
      {/if}
      {#if item.instructor.name && item.instructor.name !== "N/A"}
        <div class="instructor" style="min-width: calc(50%)">
          <Account />
          {item.instructor.name}
          {#if item.instructor.id && item.instructor.id !== "-1" && item.instructor.numRatings > 0}
            <div
              class="rmpScore"
              style={`background-color: ${rmpScoreColor(item.instructor.avgRating)}`}
            >
              {item.instructor.avgRating.toFixed(1)}
            </div>
            <div class="rmpScore" style={`background-color: #aaa`}>
              {item.instructor.avgDifficulty.toFixed(1)}
            </div>
            ({item.instructor.numRatings} ratings)
          {/if}
        </div>
      {/if}
      {#if item.gradeDistributions && item.gradeDistributions.length}
        {@const averageGPA = calculateAverageGPA(item.gradeDistributions)}
        {@const difficulty = calculateDifficulty(averageGPA)}
        <div class="gradeDistribution">
          <Poll /> GPA
          <div
            class="rmpScore"
            style={`background-color: ${gpaDifficultyColor(difficulty)}`}
          >
            {averageGPA.toFixed(1)}
          </div>
        </div>
      {/if}
    </div>
    {#if detectTerm() === $db.term}
      <ClassAllocation availability={item.availability} />
    {/if}
  </div>
{/key}

<style>
  .classItem {
    cursor: pointer;
    padding: 10px;
    background-color: var(--gray-900);
    margin: 10px;
    margin-bottom: 2px;
    border-radius: 10px;
    height: 170px;
    position: relative;
    overflow: hidden;
    outline: 2px solid var(--gray-600);

    display: flex;
    flex-direction: column;
    transition:
      transform 0.2s cubic-bezier(0.075, 0.82, 0.165, 1),
      border-radius 0.2s cubic-bezier(0.075, 0.82, 0.165, 1);
  }
  .classItem.small {
    height: 160px;
  }
  .classItem.small .topBar:before {
    background-color: var(--gray-600) !important;
  }
  .classItem.focused {
    outline: 2px solid var(--gray-500);
    background-color: var(--gray-700);
    transform: translateX(10px);
    border-radius: 10px 0 0 10px;
  }
  .classItem .topBar:before {
    content: "";
    position: absolute;
    top: -20px;
    left: -20px;
    width: calc(100% + 40px);
    height: calc(100% + 20px);
    z-index: -1;
    background-color: var(--gray-950);
    z-index: 1;
    transition: background-color 0.2s;
  }
  .classItem.focused .topBar:before {
    background-color: var(--gray-800);
  }
  .classItem.closed .topBar:before {
    background-color: var(--closed-dark);
  }
  .classItem.waitlist .topBar:before {
    background-color: var(--waitlist-dark);
  }
  .topBar {
    position: relative;
    display: flex;
    justify-content: space-between;
    align-items: center;
    transform: translateY(-5px); /* looks better on smaller text lens */
    padding-bottom: 5px;
  }
  .header {
    z-index: 2;
    font-weight: normal;
    margin: 0;
    margin-left: 5px;
    flex-grow: 1;
    overflow: hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 2;
    line-height: 1.25; /* fallback */
    max-height: 3;
  }
  .header h2 {
    margin: 0;
    font-size: 1.2rem;
    font-weight: 900;
  }
  .header h3 {
    margin: 0;
    font-size: 0.95rem;
    font-weight: 600;
  }
  .body {
    flex-grow: 1;
    display: flex;
    flex-direction: row;
    align-content: flex-start;
    justify-content: space-between;
    flex-wrap: wrap;
    gap: 5px;
  }
  .body > * {
    margin-left: 5px;
    font-size: 0.9rem;
  }
  .body > * {
    display: flex;
    align-items: center;
    gap: 5px;
  }
  :global(.rmpScore) {
    padding: 2px 5px;
    color: black;
    border-radius: 4px;
    text-align: center;
    width: 25px;
    font-weight: bold;
  }
</style>
