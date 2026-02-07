# Matomo for SvelteKit

This package provides a Svelte component to easily integrate Matomo into your SvelteKit app.

> [!WARNING]
> This package is a fork of [the original sveltekit-matomo](https://github.com/sinnwerkstatt/sveltekit-matomo).

## Installation

```bash
npm install @sinnwerkstatt/sveltekit-matomo
```

## Basic Usage

1. Include the `<Matomo />` component in your `+layout.svelte` and configure it with the URL of your Matomo instance and the site ID of your website.

   ```svelte
   <script lang="ts">
   	import { Matomo } from '@sinnwerkstatt/sveltekit-matomo';
   </script>

   <Matomo url="https://matomo.mysite.com" siteId={13} />
   ```

2. You can also use environment variables via `$env/static/public` (recommended) or `$env/dynamic/public` to configure it. See [SvelteKit docs](https://svelte.dev/docs/kit/$env-static-public) for more information.

   `.env`:

   ```ini
   PUBLIC_MATOMO_URL="https://matomo.mysite.com"
   PUBLIC_MATOMO_SITE_ID=13
   ```

   `+layout.svelte`:

   ```svelte
   <script lang="ts">
   	import { Matomo } from '@sinnwerkstatt/sveltekit-matomo';
   	import { PUBLIC_MATOMO_URL, PUBLIC_MATOMO_SITE_ID } from '$env/static/public';
   </script>

   <Matomo url={PUBLIC_MATOMO_URL} siteId={Number(PUBLIC_MATOMO_SITE_ID)} />
   ```

## Configuration Options

All available props for the `<Matomo />` component:

| Prop                       | Type                         | Required | Default | Description                                                      |
| -------------------------- | ---------------------------- | -------- | ------- | ---------------------------------------------------------------- |
| `url`                      | `string`                     | Yes      | -       | URL of your Matomo instance (e.g., `https://matomo.example.com`) |
| `siteId`                   | `number`                     | Yes      | -       | Site ID from your Matomo configuration                           |
| `disableCookies`           | `boolean`                    | No       | `false` | Disable all first-party cookies                                  |
| `requireConsent`           | `boolean`                    | No       | `false` | Require user consent before tracking                             |
| `doNotTrack`               | `boolean`                    | No       | `false` | Respect the browser's Do Not Track setting                       |
| `enableCrossDomainLinking` | `boolean`                    | No       | `false` | Enable cross-domain linking                                      |
| `domains`                  | `string[]`                   | No       | `[]`    | Array of domains to be treated as local                          |
| `heartBeat`                | `number \| null`             | No       | `15`    | Heart beat timer interval in seconds (set to `null` to disable)  |
| `linkTracking`             | `boolean \| null`            | No       | `null`  | Enable link tracking (auto-enabled if not specified)             |
| `onTrackerReady`           | `(tracker: Tracker) => void` | No       | -       | Callback when tracker is initialized, useful for custom setup    |
| `onError`                  | `(error: Error) => void`     | No       | -       | Error handler callback                                           |

## Advanced Usage

### Custom Tracker Setup

Use the `onTrackerReady` callback to perform custom initialization:

```svelte
<script lang="ts">
	import { Matomo, type Tracker } from '@sinnwerkstatt/sveltekit-matomo';
	import { page } from '$app/stores';

	function handleTrackerReady(tracker: Tracker) {
		// Set custom dimensions
		tracker.setCustomDimension(1, $page.data.user ? 'logged-in' : 'anonymous');

		// Set user ID if available
		if ($page.data.userId) {
			tracker.setUserId($page.data.userId);
		}
	}
</script>

<Matomo url="https://matomo.mysite.com" siteId={13} onTrackerReady={handleTrackerReady} />
```

### Tracking Custom Events

Use the `tracker` store to track custom events anywhere in your app:

```svelte
<script lang="ts">
	import { tracker } from '@sinnwerkstatt/sveltekit-matomo';

	function onButtonClick() {
		if ($tracker) {
			$tracker.trackEvent('button', 'click', 'cta-signup');
		}
	}

	function onFormSubmit() {
		if ($tracker) {
			$tracker.trackEvent('form', 'submit', 'newsletter', 1);
		}
	}
</script>

<button on:click={onButtonClick}>Sign Up</button>
```

### Cookie Consent Management

```svelte
<script lang="ts">
	import { Matomo, tracker } from '@sinnwerkstatt/sveltekit-matomo';

	let consentGiven = $state(false);

	function giveConsent() {
		consentGiven = true;
		if ($tracker) {
			$tracker.setConsentGiven();
		}
	}
</script>

<Matomo url="https://matomo.mysite.com" siteId={13} requireConsent={true} />

{#if !consentGiven}
	<div class="cookie-banner">
		<p>We use cookies for analytics.</p>
		<button on:click={giveConsent}>Accept</button>
	</div>
{/if}
```

### Error Handling

```svelte
<script lang="ts">
	import { Matomo } from '@sinnwerkstatt/sveltekit-matomo';

	function handleMatomoError(error: Error) {
		console.error('Matomo error:', error);
		// Report to your error tracking service
	}
</script>

<Matomo url="https://matomo.mysite.com" siteId={13} onError={handleMatomoError} />
```

### Tracking Site Search

```svelte
<script lang="ts">
	import { tracker } from '@sinnwerkstatt/sveltekit-matomo';

	function handleSearch(keyword: string, category: string, resultsCount: number) {
		if ($tracker) {
			$tracker.trackSiteSearch(keyword, category, resultsCount);
		}
	}
</script>
```

### Cross-Domain Tracking

```svelte
<Matomo
	url="https://matomo.mysite.com"
	siteId={13}
	enableCrossDomainLinking={true}
	domains={['*.example.com', '*.example.org']}
/>
```

## Available Tracker Methods

When using the `tracker` store or `onTrackerReady` callback, you have access to these methods:

- `trackPageView(customTitle?: string)` - Log a page view
- `trackEvent(category: string, action: string, name?: string, value?: number)` - Log an event
- `trackSiteSearch(keyword: string, category?: string, resultsCount?: number)` - Log a site search
- `enableHeartBeatTimer(activeTimeInSeconds?: number)` - Enable heart beat timer for better time tracking
- `enableLinkTracking(enable?: boolean)` - Enable automatic link tracking
- `setCustomUrl(url: string)` - Override the page's reported URL
- `setDoNotTrack(bool: boolean)` - Respect Do Not Track browser setting
- `setCustomDimension(dimensionID: number, value: string)` - Set a custom dimension
- `requireConsent()` - Require user consent before tracking
- `disableCookies()` - Disable all first-party cookies
- `enableCrossDomainLinking()` - Enable cross-domain linking
- `setDomains(domains: string[])` - Set domains to be treated as local

## License

AGPL-3.0-or-later

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
