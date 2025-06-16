<script lang="ts">
  import Matomo from "$lib/Matomo.svelte"
  import { tracker } from "$lib/tracker"
</script>

Matomo test

<Matomo />

{#if $tracker}
  <button on:click={() => $tracker.trackEvent('test', 'track me')}>track me</button>
{/if}
