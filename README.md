# webhook-inspector

Local webhook inspector. Registers a Service Worker that intercepts requests to a configured path on the same origin and displays them in real time. Single HTML file. Browser-only.

**Live demo:** https://0xelitesystem.github.io/webhook-inspector/

## What this is, and what it isn't

**Is**: a local development tool. You point your own client-side code or a same-origin tool at the configured path (e.g. `https://0xelitesystem.github.io/webhook-inspector/inspect`) and inspect the request in your browser without spinning up a backend.

**Isn't**: a public-internet webhook receiver. Service Workers only intercept same-origin requests. External services like Stripe, GitHub, or Slack can't send to a Service Worker URL, those need an actual server, and tools like [ngrok](https://ngrok.com) or [webhook.site](https://webhook.site) are the right answer.

So this is for testing your own browser-side fetch code, or for piping local CLI tools at a same-origin URL during dev.

## Use it

Open `index.html` in a browser, or visit the hosted demo at `https://0xelitesystem.github.io/webhook-inspector/` once Pages is enabled.

1. Set a capture path (default: `/inspect`)
2. Click Activate. The Service Worker registers and starts intercepting.
3. Send any request to that path. Use the built-in test sender, your own JS code, curl, or any same-origin source.
4. Watch captured requests appear in the list. Click to expand and see headers and body.

## How it works

When you click Activate, the page registers a Service Worker built from an inline JS Blob URL. That worker:

1. Listens for `fetch` events from any client of the same origin
2. Filters to requests whose path starts with your configured prefix
3. Clones the request, reads method + URL + headers + body
4. Posts that data to a `BroadcastChannel` that the inspector page listens on
5. Replies to the original request with `{"ok": true, "captured": true}`, status 200

The inspector page receives the broadcast and adds the request to the visible list. All in-memory, no storage.

When you click Deactivate, the worker is unregistered. Active requests in flight may still be intercepted until the worker fully detaches; refresh if it lingers.

## Limitations

- **Same-origin only.** This is by Service Worker design.
- **HTTPS or localhost.** Service Workers don't run on `http://` non-localhost origins.
- **Only intercepts after activation.** Requests sent before the worker is active reach the network normally (or 404).
- **Up to 200 captured requests** are kept in memory. Older ones drop off.
- **Closing or refreshing the tab** drops the captured list. State is intentionally non-persistent.

## Why no localStorage for captures

Persisting webhook bodies to localStorage would surface them on next page load even after the user thought they cleared everything. Better to keep them in memory and force re-capture on refresh. If you need a record of captures, add an Export JSON button (a future addition; currently just copy-paste from the visible body).

## Tech

- Single HTML file, ~700 lines
- Service Worker built inline as a Blob URL (no separate `sw.js` file)
- BroadcastChannel for SW-to-page messaging
- Vanilla JS, no frameworks, no dependencies
- Light and dark themes, OS preference honored
- WCAG AA contrast on both themes

## When to use this vs alternatives

| Use case | Tool |
|---|---|
| Receive webhooks from Stripe, GitHub, etc. on the public internet | ngrok, webhook.site, Cloudflare Tunnel |
| Test your own browser-side fetch code's request shape | this tool |
| Inspect requests from a Postman-style client pointed at your origin | this tool |
| Mock-server a development build's API calls | this tool, with the SW returning canned responses |

## License

MIT. See [LICENSE](LICENSE).

## Related

- [csp-builder](https://github.com/0xelitesystem/csp-builder), build Content-Security-Policy headers visually
- [byok-patterns](https://github.com/0xelitesystem/byok-patterns), three deeply-built BYOK reference implementations
- [single-file-saas-template](https://github.com/0xelitesystem/single-file-saas-template), ship a SaaS in one HTML file
