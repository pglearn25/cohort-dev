# Which hosted notification vendor fits Cadence?

Resolves issue #2 (map: #1). Researched 2026-11-11 from vendor docs and pricing pages only. Anything I could not confirm from a primary source is labelled "Guess:" or "Not verified".

## Recommendation

**Knock** (one sign-up), runner-up **Novu Cloud**.

Why Knock: a documented headless path (low-level JS client with read/seen/archived state, badge counts and a websocket) and a free tier whose in-app feed retention is unlimited. Novu's free tier has the shorter, murkier retention. Knock needs no extra service for the in-app feed. Email needs a downstream provider and web push needs Firebase, so those two channels cost extra sign-ups (see Channels).

## Comparison

| Need | Knock | Novu Cloud | Courier | MagicBell |
|---|---|---|---|---|
| Free tier | 10,000 messages/month, unlimited environments and stored profiles [K1] | 10,000 workflow runs/month, 2 environments (Dev + Prod), 3 team members [N1] | 10,000 sends/month [C1] | 1,000 deliveries/month, 1 project [M1] |
| Headless feed | Yes: "headless UI using our lower-level primitives"; `@knocklabs/client` [K2][K3] | Yes: `@novu/js` has list, read, unread, archive, snooze, preferences, realtime events [N2] | Not verified (Inbox docs page returned 404) | Not verified (docs overview does not mention headless) [M2] |
| Read/seen/archived, badge counts | Yes [K4] | Yes; seen must be called manually when headless [N2] | Not verified | Not verified |
| Per-user preferences | PreferenceSet of opted-out channels/categories/workflows, checked before send [K5] | `preferences.list()` / `update()` [N2] | Not verified | Not verified |
| Idempotency | `Idempotency-Key` on the trigger endpoint only; 24h window; only 2xx cached [K6] | `Idempotency-Key` on POST/PATCH; 24h cache; docs say it is not enabled for all orgs and you contact support to enable it [N3] | Not verified | Not verified |
| Retention | Feed: unlimited on Free. Message logs: 30 days (90 Enterprise) [K1][K7] | Pricing page: activity feed 24h Free, 7d Pro, 90d Team [N1]. Limits page also shows "Inbox feed retention" 30 days on Free, and does not say what it covers [N4] | Not stated [C1] | Not stated [M1] |
| Rate limits | Five tiers of 1/5/60/200/1000 req/s; tier-to-endpoint mapping not visible in the fetched page; 429 on exceed [K8] | Free: trigger 60 req/s, config 20, global 30; bulk costs 100 tokens [N5] | Not verified | Not verified |
| Pricing at growth | Starter $250/month incl. 50,000 messages, then $0.005 each [K1] | Pro from $30/month (30,000+ runs, 7d retention); Team from $250/month (250,000+ runs, 90d); extra runs $1.20 per 1K [N1] | Business $0.005 per send, pay as you go [C1] | Startup $249/month incl. 50,000 deliveries, then $0.0025 each [M1] |
| Node/TS SDK | `knock-node` official repo [K9] | `@novu/js` (client). Server SDK not checked | "SDKs" listed, not checked [C1] | Browser/Node client listed [M2] |

## Channels (optional per the issue)

- **Email.** Knock: downstream providers such as Postmark or Twilio are billed separately [K1], so email means a second account. Novu: "Email" is listed as a free-tier channel [N1]; Guess: it still needs an email provider integration for real delivery. Not verified.
- **Browser web push.** Knock has no first-party web push channel; it goes through Firebase Cloud Messaging with your own service worker, which means a Firebase project (extra sign-up) [K10]. Novu supports FCM web push the same way, plus Pushpad and OneSignal as providers [N6]; those pages look old, so the details may have changed. Neither vendor removes the need for a push provider.
- **Conclusion.** Web push and email are not free of extra sign-ups with any shortlisted vendor. The in-app feed alone needs exactly one service.

## Production notes

- Knock idempotency is limited to the trigger endpoint. Using a job or event ID as the key is the documented pattern for retried jobs [K6]. Replays carry an `idempotent-replayed: true` header.
- Novu idempotency works only with API-key auth and, per its docs, is not enabled for all organizations [N3]. This is a point against Novu for a production-grade bar.
- Knock Free has no hard cap; going over without a billing method means Knock contacts you [K1]. Novu's behaviour on exceeding the free runs is not stated [N1].
- Retention: Knock keeps feed items indefinitely even after the 30-day log window; custom retention policies are not supported [K7].

## Gaps and risks

- Courier and MagicBell were shortlisted but I could not verify headless feed, preferences, idempotency or retention from their docs in this pass. They are not recommended because of that gap, not because they were shown to fail.
- Knock web push, Knock preferences API details, and the Knock rate-limit tier mapping were not fully verified.
- Feed messages count toward Knock message usage [K7], so fan-out to many recipients multiplies usage. Guess: 10,000 free messages/month is enough for development.
- Knock Starter jumps to $250/month; Novu Pro at $30/month is the cheaper first paid step.

## Sources

- [K1] Knock pricing: https://knock.app/pricing
- [K2] Knock in-app UI overview: https://docs.knock.app/in-app-ui/overview
- [K3] Knock JS client: https://docs.knock.app/in-app-ui/javascript/overview
- [K4] Knock feeds: https://docs.knock.app/in-app-ui/feeds/overview
- [K5] Knock preferences: https://docs.knock.app/concepts/preferences
- [K6] Knock idempotent requests: https://docs.knock.app/api-reference/overview/idempotent-requests
- [K7] Knock data retention: https://docs.knock.app/manage-your-account/data-retention and https://docs.knock.app/concepts/messages
- [K8] Knock rate limits: https://docs.knock.app/api-reference/overview/rate-limits
- [K9] Knock Node SDK: https://github.com/knocklabs/knock-node
- [K10] Knock web push via FCM: https://docs.knock.app/tutorials/sending-web-push-notifications-with-fcm
- [N1] Novu pricing: https://novu.co/pricing
- [N2] Novu JS SDK: https://docs.novu.co/platform/sdks/javascript
- [N3] Novu idempotency: https://docs.novu.co/api-reference/idempotency
- [N4] Novu limits: https://docs.novu.co/platform/developer/limits
- [N5] Novu rate limiting: https://docs.novu.co/api-reference/rate-limiting
- [N6] Novu FCM push: https://docs.novu.co/platform/integrations/push/fcm
- [C1] Courier pricing: https://www.courier.com/pricing
- [M1] MagicBell pricing: https://www.magicbell.com/pricing
- [M2] MagicBell docs: https://www.magicbell.com/docs
