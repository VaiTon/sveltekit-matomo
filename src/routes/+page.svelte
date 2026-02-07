<script lang="ts">
	import Matomo from '$lib/Matomo.svelte';
	import { tracker } from '$lib/tracker';
</script>

<h1>Matomo Test Page</h1>

<p>This is a test page for the Matomo component.</p>

<Matomo url="https://example.matomo.cloud" siteId={1} />

{#if $tracker}
	<button onclick={() => $tracker.trackEvent('test', 'track me')}>track me</button>
{:else}
	<p>Tracker not initialized yet...</p>
{/if}
