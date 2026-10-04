# Church Work Code Review 

Notes: ran Codex agent through to find issues - my write-ups have been put in italics. 

## Issues / Concerns 

### High — Firestore writes contradict policy.

An unauthenticated POST endpoint calls createReviewItem(), which writes to Firestore. It currently also depends on firebase-admin, absent from declared dependencies. [Endpoint (line 5)](./app/api/review-queue/items/route.ts) · [Write implementation (line 100)](./lib/firebase/review-items.ts).

### High — “Private pilot access” is bypassable. 

The access gate trusts an editable browser sessionStorage value. Middleware only adds cache/search-index headers; it does not authenticate users. This gate cannot protect private information. [Access gate (line 20)](./components/ChurchWorkAccessGate.tsx).

### High — Public requests bypass the advertised facility review sequence. 

Guest requests persist directly as submitted; partner queries include that status. This differs from the sandbox’s “facility approves before partner assignment” flow. [Guest endpoint (line 18)](./app/api/guest-requests/route.ts) · [Partner endpoint (line 7)](./app/api/partner-requests/route.ts).

*BV NOTES: assuming this is just for demo purposes*

### Medium — Privacy checks give false assurance. 
The MVP uses a short keyword blacklist, which cannot detect names, SSNs, or many medical details. Notes remain editable after approval, so approved partner context can subsequently change. [MVP page (line 21)](./app/mvp/page.tsx).

*BV NOTES: a classic that I teach my students - if something doesn't allow you the first time around, try again! Will need to hard check wherever this is allowed.*

### Medium — Public endpoint can incur API costs. 

Demo audio calls OpenAI when configured, without endpoint authentication or rate limiting. [Audio endpoint (line 68)](./app/api/churchwork-demo-audio/route.ts).

*BV NOTES: `churchWorkCareLedgerDashboard.tsx` and `churchworkDemoAudioClient.ts` are being called client-side, and hitting the `/api/churchwork-demo-audio`. How often will this demo change? Could be worth putting an admin option to regenerate it, instead of requiring it to reset every 24 hours.*

## Overall: 
Maintainability — Significant scope/documentation drift. CRCF funding tools, SpiritualSolace demos, and ChurchWork live portals coexist. The README says local-only and Cloudflare work is parked, while database integrations and Cloudflare dependencies/scripts are present.