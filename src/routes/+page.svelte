<script lang="ts">
	import Matomo from '$lib/Matomo.svelte';
	import { tracker } from '$lib/tracker';
</script>

Matomo test

<Matomo />

{#if $tracker}
	<button onclick={() => $tracker.trackEvent('test', 'track me')}>track me</button>
{/if}
