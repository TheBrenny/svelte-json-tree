<script lang="ts">
  /* eslint-disable @typescript-eslint/no-empty-function */
  import { getContext, onMount } from 'svelte';
  import JSONArrow from './JSONArrow.svelte';
  import { useState } from './utils/context';
  import { writable } from 'svelte/store';
  import Summary from './Summary.svelte';
  import Expandable from './Expandable.svelte';

  export let keys: string[];
  export let shouldShowColon: (key: string) => boolean = undefined;
  export let expandKey = (key: string) => key;

  export let defaultExpanded = false;
  const { isParentExpanded, displayMode, root, expanded, expandable, keyPath, level, shouldExpandNode } = useState(
    { root: false },
    { expandable: true }
  );
  $expandable = true;

  const ctx: { labelsCanBeClicked: boolean; contenteditable: boolean } = getContext('json_tree');

  if (displayMode !== 'summary') {
    // if not internally control to open
    if (!defaultExpanded) {
      const controlled = shouldExpandNode({ keyPath, level });
      if (controlled !== undefined) {
        defaultExpanded = controlled;
      }
    }

    onMount(() => {
      return isParentExpanded.subscribe((value) => {
        if (!value) expanded.set(false);
        else expanded.set(defaultExpanded);
      });
    });
  }
  function toggleExpand() {
    if (!ctx.labelsCanBeClicked) return;
    $expanded = !$expanded;
  }
  function toggleExpandLabel(index) {
    if (!ctx.labelsCanBeClicked) return;
    child_expanded[index].update((value) => !value);
  }

  $: child_expanded = keys.map(() => writable(false));
</script>

{#if displayMode === 'summary'}
  <slot name="summary" />
{:else}
  <!-- svelte-ignore a11y-click-events-have-key-events a11y-no-static-element-interactions -->
  <span class="root" on:click={toggleExpand}>
    {#if root}
      <JSONArrow {expanded} />
    {/if}
    <Summary>
      <slot name="preview" {root} />
    </Summary>
  </span>

  {#if $expanded}
    <!-- svelte-ignore a11y-click-events-have-key-events a11y-no-noninteractive-element-interactions -->
    <ul on:click|stopPropagation={toggleExpand}>
      {#each keys as key, index}
        <!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
        <li class:indent={$expanded} on:click|stopPropagation={() => {}}>
          <Expandable key={expandKey(key)} expanded={child_expanded[index]}>
            <!-- svelte-ignore a11y-no-static-element-interactions -->
            <span class="label" on:click={() => toggleExpandLabel(index)}>
              <JSONArrow /><slot name="item_key" {key} {index} />{#if !shouldShowColon || shouldShowColon(key)}<span class="operator"
                  >{': '}</span
                >{/if}
            </span><slot name="item_value" {key} {index} />
          </Expandable>
        </li>
      {/each}
    </ul>
  {/if}
{/if}

<style>
  .root {
    display: inline-block;
    position: relative;
  }
  .indent {
    padding-left: var(--li-identation);
  }
  .label {
    position: relative;
  }
</style>
