# Free accounts and phone access

For the newer password signup/sign-in flow and hosting while the computer is off, use [Free server hosting and password accounts](FREE_SERVER_HOSTING.md). The email-code and local-tunnel setup below remains available for existing accounts and local testing.

Account support uses Supabase Free for verified email sign-in and a private workspace per account. No billing plan, service-role key, booking or payment integration is needed. The server keeps session cookies HttpOnly; browser caches are separated by account. Existing device trips require an explicit import in Account. Different-device edits use revision checks; a conflict preserves the local copy until reviewed.

## Finish Supabase setup

1. Add the Free project's `SUPABASE_URL` and `SUPABASE_PUBLISHABLE_KEY` to `.env.local`. A legacy anon key also works. Never use a secret/service-role key.
2. Open the project's SQL Editor and run [`supabase/workspaces.sql`](../supabase/workspaces.sql). This creates the workspace table, owner-only row policies and revision-checked save function. It does not import device trips.
3. Enable Email under Authentication → Sign In / Providers. Keep email confirmation enabled.
4. Configure custom SMTP before editing templates on Supabase Free. Create a **Brevo Free** account, verify a sender, generate an SMTP key and enter Brevo's SMTP host, port, Login and SMTP key in Supabase Authentication → Emails → SMTP Settings. Use your verified sender address and sender name Wayfarer. Brevo's observed settings were `smtp-relay.brevo.com`, port `587`; use the values shown in your own account. Keep the key in Supabase, out of this app and chat. Brevo Free currently allows 300 emails/day; sender verification/account validation and delivery restrictions can apply. Do not upgrade or supply payment details for this setup.
5. Under Authentication → Emails → Templates, replace **Magic link or OTP** with a numeric-code email. Use the same code-only body for **Confirm sign up**, so new accounts receive a code too:

```html
<h2>Your Wayfarer code</h2>
<p>Enter this code to sign in: {{ .Token }}</p>
<p>If you did not request this email, ignore it.</p>
```

Supabase's default SMTP only sends to project-team email addresses, at two emails per hour. The current Free dashboard also restricts template editing until custom SMTP is configured. Its initial custom-SMTP rate limit is 30 emails/hour. Leave this in place for personal testing.

The **exact From address** must appear as Verified in Brevo → Senders, domains, IPs. Creating a Gmail account or saving its address in Supabase does not verify it in Brevo. Add the sender in Brevo and complete its inbox verification first. If the app accepts a code-send request but nothing arrives, check Brevo → Transactional → Email → Logs for an asynchronous rejection. An accepted SMTP message is not proof of delivery.

6. Open `/sign-in`, request a code and enter it. Open Account → Import device trips to move existing browser trips into your account. Check Account says **Trips synced**, then sign in on the phone to confirm they appear.

Supabase's Free tier currently includes 50,000 monthly active users and 500 MB of database storage. Idle free projects can pause. This app limits a workspace to 2 MB and 200 saved trips plus 200 recoverable deletions, and reserves at most 200 code-send attempts/day locally. These limits are safeguards, not a promise of unlimited free service. Existing search-provider quotas still apply.

## Access from outside your Wi-Fi

Use the free temporary Cloudflare tunnel for personal testing. It needs no domain or Cloudflare account. The app uses a **production build on loopback port 3001**, separate from the development server. Production always requires verified sign-in for planner pages and search APIs, and rejects cross-origin writes. The URL is public, so do not disable authentication.

In PowerShell, in the project directory:

Stop the development server with Ctrl+C first; Next.js shares a generated TypeScript file across build modes. The remote launcher checks port 3000 and asks you to stop it before building.

```powershell
powershell -File scripts/install-cloudflared.ps1
npm run remote
```

The installer downloads Cloudflare's official Windows binary and verifies the release SHA-256 digest. `npm run remote` builds the app, starts the private server and prints an HTTPS phone link. Keep the terminal open and computer awake. Open the link over cellular data and sign in. Ctrl+C stops access. No router port forwarding or broad firewall rule is required.

On this Windows setup, development uses Webpack and dual-stack `::` binding so Codex's browser can reach `localhost` over IPv6. Production builds also use Webpack after a reproducible Turbopack CSS-worker startup failure. Phone access still binds its production server only to `127.0.0.1` behind the tunnel. To import existing local trips, sign in using the same browser and `http://localhost:3000` origin where you created them, then use Account → Import device trips.

The temporary hostname changes every restart and has no uptime guarantee. Quick Tunnels allow 200 concurrent in-flight requests and do not support SSE. Flight progress uses NDJSON; verify streaming on your network. Persistent public hosting with the computer off is a separate deployment, and this setup does not provide it. Never tunnel `next dev` or port 3000.

## Recovery and testing

If sync fails, trips remain in that account's local cache. Account provides retry and a JSON backup. On a revision conflict, download a backup before loading the account copy; an additional recovery copy remains on the same device and can be restored as separate trips. Guest trips are never silently uploaded into a signed-in account. Sign-out clears the account cache only when fully synced; unsynced edits remain in its account-specific cache for recovery. Avoid shared browsers for sensitive trip data, or download a backup and clear site data after signing out.

Room details are displayed only when a source actually supplies them. Structured room data supports labels such as **2 double beds · Sleeps 4**, private/shared rooms and multiple units. Xotelo's current free listings do not include verified bed layouts or room capacity: those cards explicitly mark the information unavailable and show requested guests/rooms separately. A double bed alone does not establish a room's allowed occupancy.

Sources: [Supabase passwordless email](https://supabase.com/docs/guides/auth/auth-email-passwordless), [SMTP restrictions](https://supabase.com/docs/guides/auth/auth-smtp), [Supabase pricing](https://supabase.com/pricing), [Brevo Free limits](https://help.brevo.com/hc/en-us/articles/208589409-About-Brevo-s-pricing-plans), [Cloudflare Quick Tunnels](https://developers.cloudflare.com/tunnel/get-started/quick-tunnels/).
