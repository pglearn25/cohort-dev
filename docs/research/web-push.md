# Is browser web push workable for Cadence without a mobile app?

Resolves issue #3 (map: #1). Researched 2026-11-11. Primary sources only; unverified claims are labelled "Guess:".

## Verdict: workable with caveats

Web push works on every major engine, but only reaches iPhone/iPad users who install Cadence to the Home Screen as a web app and opt in per device. Use it as an opt-in supplement to email, never the only channel.

## Findings

### Browser support
- Push API is Baseline "widely available", across browsers since March 2023. [MDN Push API](https://developer.mozilla.org/en-US/docs/Web/API/Push_API)
- caniuse: Chrome 50+, Edge 79+, Firefox 44+, Safari macOS 16.1+ (partial until 17.6, full from 18.0), iOS Safari 16.4+ (marked partial in every listed version). [caniuse push-api](https://caniuse.com/push-api)
- Guess: caniuse's "partial" for iOS reflects the Home Screen requirement below; the page did not say why.

### iOS / iPadOS: installed web app required
- Web Push arrived in iOS/iPadOS 16.4. The site must be added to the Home Screen with a manifest whose `display` is `standalone` or `fullscreen`; otherwise it is only a bookmark. [WebKit: Web Push for Web Apps on iOS and iPadOS](https://webkit.org/blog/13878/web-push-for-web-apps-on-ios-and-ipados/)
- Permission can only be requested in response to direct user interaction (e.g. tapping a button); users manage it per web app in Settings, and it integrates with Focus. Same source.
- No Apple Developer Program membership needed; push endpoints must allow `*.push.apple.com`. Same source.
- Plain Safari tabs on iPhone therefore cannot receive push; Cadence would need an "Add to Home Screen" onboarding step.
- Not verified: whether iOS 18.4 declarative push removes the install requirement; the WebKit post does not say. [WebKit: Declarative Web Push](https://webkit.org/blog/16535/meet-declarative-web-push/)

### Safari on macOS
- Safari 16 / macOS Ventura+ supports Web Push without the Developer Program; subscribing needs an explicit user gesture. [WebKit: Meet Web Push](https://webkit.org/blog/12945/meet-web-push/)
- Delivery goes through APNs via `webpushd`; Safari need not be running. Same source.
- Not verified: whether macOS requires installing to the Dock; the post does not say it does.

### Rules that bite
- Every push must show a visible notification (`userVisibleOnly: true`); silent background push is not allowed and violating it gets the subscription revoked (Safari). [WebKit: Meet Web Push](https://webkit.org/blog/12945/meet-web-push/) A badge update alone does not count. [WebKit: Safari 17.0 features](https://webkit.org/blog/14112/webkit-features-for-safari-17-0/)
- Declarative Web Push (iOS/iPadOS 18.4, macOS 15.5 beta at the time of the post) lets the browser display a JSON notification with no service worker and avoids the revocation penalty. [WebKit: Declarative Web Push](https://webkit.org/blog/16535/meet-declarative-web-push/) Guess: too new and Safari-only to design around; classic service-worker push is the baseline.

### Permission prompt UX norms
- Request only in response to a user gesture; browsers disallow requests not triggered by one (Firefox since 72, Safari). Chrome and Firefox require a secure context (HTTPS). [MDN: Using the Notifications API](https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API/Using_the_Notifications_API)
- Don't spam; prompt from an "Enable notifications" button, not on page load; give users a way to change their mind after denial. Same source.
- Guess: a denied state cannot be re-prompted by the site and the user must change it in browser settings; not confirmed in the fetched text.
- Chrome's quiet-prompt/auto-revocation docs could not be fetched (developer.chrome.com URLs returned 404). Not verified.

### Service worker and protocol requirements
- An active service worker is required to subscribe via `PushManager.subscribe()`. The subscription endpoint must be kept secret and subscription handling protected against CSRF. [MDN Push API](https://developer.mozilla.org/en-US/docs/Web/API/Push_API)
- The server sends encrypted POSTs to each subscription endpoint, identifying itself with VAPID (JWT signed with a P-256 key, `exp` at most 24h); 404/410 means delete the subscription. [web.dev: Web Push Protocol](https://web.dev/articles/push-notifications-web-push-protocol); [RFC 8292 (VAPID)](https://datatracker.ietf.org/doc/html/rfc8292)
- Battery: no standard mechanism governs service worker wake-ups; Firefox quotas only pushes that don't show a notification. [MDN Push API](https://developer.mozilla.org/en-US/docs/Web/API/Push_API)

### What a vendor must provide
No vendor is strictly required: the push services are run by the browser makers and reached through the standard protocol (sources above). Cadence's own server or a vendor needs to: hold a VAPID key pair, store per-device subscriptions per user, encrypt and send payloads, and prune 404/410 subscriptions. Guess: hosted vendors package this; an open-source web-push library would also do. Not verified against any vendor documentation.

### Realism for Cadence's audience
- Students on desktop (Chrome/Edge/Firefox/Safari macOS): straightforward.
- Students on iPhone: Home Screen install first, a real friction point. Guess: a minority will do it.
- Android: supported by Chrome/Firefox per caniuse; no install requirement found in the sources.
- Guess: subscriptions are per browser/device, so Cadence must handle multiple subscriptions per user.
- Guess: instructor/admin alerts are a lower-friction fit than student reminders on iOS.

## Open items for #6
- Confirm the iOS 18.4+ Home Screen requirement on a real device.
- Chrome's permission-quieting/auto-revocation behaviour (source not retrieved).
- Choose library vs vendor; opt-in rates are unmeasured.
