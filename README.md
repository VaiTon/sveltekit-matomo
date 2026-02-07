# Matomo for SvelteKit

This package provides a Svelte component to easily integrate Matomo into your SvelteKit app.

> [!WARNING]
> This package is a fork of [the original sveltekit-matomo](https://github.com/sinnwerkstatt/sveltekit-matomo).

## How to use

1. Include the `<Matomo />` component in your `+layout.svelte` and configure it with the URL of your Matomo instance and the site ID of your website.

   ```svelte
   <Matomo url="https://matomo.mysite.com" siteId={13} />
   ```

   You can also use environment variables via `$env/static/public` (recommended) or `$env/dynamic/public` to configure it. See [SvelteKit docs](https://svelte.dev/docs/kit/$env-static-public) for more information.

   `.env`:

   ```ini
     PUBLIC_MATOMO_URL="https://matomo.mysite.com"
     PUBLIC_MATOMO_SITE_ID=13
   ```

   `+layout.svelte`:

   ```svelte
   <script lang="ts">
   import {(PUBLIC_MATOMO_URL, PUBLIC_MATOMO_SITE_ID)} from '$env/static/public';
   </script>

   <Matomo url={PUBLIC_MATOMO_URL} siteId={PUBLIC_MATOMO_SITE_ID} />
   ```

2. Optional: track custom events using the `tracker` store provided by the package.
   ```svelte
   <script lang="ts">
   	import { tracker } from '@sinnwerkstatt/sveltekit-matomo';
   	function onSomeEvent() {
   		if ($tracker) $tracker.trackEvent('my-cateogry', 'my-action', 'my-name');
   	}
   </script>
   ```
