<script lang="ts">
	import { onMount } from 'svelte';
	import { env } from '$env/dynamic/public';
	import { afterNavigate } from '$app/navigation';
	import { tracker, type Tracker } from '$lib/tracker';
	import { page } from '$app/state';

	interface Props {
		url?: string;
		siteId?: number;
		disableCookies?: boolean;
		requireConsent?: boolean;
		doNotTrack?: boolean;
		enableCrossDomainLinking?: boolean;
		domains?: string[];
		heartBeat?: number | null;
		linkTracking?: boolean | null;
	}

	let {
		url = env.PUBLIC_MATOMO_URL,
		siteId = env.PUBLIC_MATOMO_SITE_ID == null ? undefined : parseInt(env.PUBLIC_MATOMO_SITE_ID),
		disableCookies = false,
		requireConsent = false,
		doNotTrack = false,
		enableCrossDomainLinking = false,
		domains = [],
		heartBeat = 15,
		linkTracking = null
	}: Props = $props();

	interface TMatomo {
		getTracker: (url: string, siteId: number) => Tracker;
		set: (tracker: Tracker) => void;
	}

	interface TWindow {
		Matomo: TMatomo;
	}

	async function initializeMatomo() {
		// gymnastics to silence svelte-check
		const matomo = (window as unknown as TWindow).Matomo;
		if (!matomo) return;

		if (!url || !siteId) {
			console.warn('Matomo URL or Site ID is not set. Skipping Matomo initialization.');
			return;
		}

		const track = matomo.getTracker(`${url}/matomo.php`, siteId);
		if (!track) return;

		if (disableCookies) track.disableCookies();
		if (requireConsent) track.requireConsent();
		if (doNotTrack) track.setDoNotTrack(true);
		if (heartBeat) track.enableHeartBeatTimer(heartBeat);
		if (enableCrossDomainLinking) track.enableCrossDomainLinking();
		if (domains.length) track.setDomains(domains);
		if (linkTracking !== null) track.enableLinkTracking(linkTracking);

		tracker.set(track);

		track.setCustomDimension(1, page.data.user ? 'true' : 'false');
		track.setCustomUrl(page.url.href);
		track.trackPageView();
	}

	onMount(async () => {
		setTimeout(initializeMatomo, 100);
	});

	afterNavigate(async ({ to }) => {
		if (!$tracker) {
			await initializeMatomo();
			return;
		}

		if (to?.url.href && $tracker) {
			$tracker.setCustomDimension(1, page.data.user ? 'true' : 'false');
			$tracker.setCustomUrl(to.url.href);
			$tracker.trackPageView();
		}
	});
</script>

<svelte:head>
	{#if url && siteId}
		<script async defer src={`${url}/matomo.js`}></script>
	{/if}
</svelte:head>
