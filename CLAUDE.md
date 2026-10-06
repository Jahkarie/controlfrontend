# On D' Road: guest site (`controlfrontend`)

On D' Road is an invite-only, 18+ Antigua Carnival band. This repo is the guest-facing site. The API, admin pages and door scanner are in the separate `control` repo, whose `CLAUDE.md` describes the backend, known issues and features not built yet.

## Stack

- One static page, `index.html`: HTML, CSS and plain JavaScript. No build step, no framework.
- Images next to it: `og.png` (1200×630 link preview), `favicon.svg`, `favicon-32.png`, `apple-touch-icon.png`. The `og:url` and `og:image` tags point at `https://ondroad.xyz/`.
- Only external script: `qrcodejs` from cdnjs, for the entry pass.
- Calls the backend at `BACKEND_URL` (set near the top of the `<script>`, currently `https://control-1-0baa.onrender.com`). The backend's CORS list must include wherever this page is served (`ondroad.xyz`, `www.ondroad.xyz`, `FRONTEND_URL`).

## Page flow

- **`login-view`:** enter email → `/api/login` emails a one-time link.
- **`?login=<code>`:** exchanged at `/api/login/verify` for a session token.
- **`accept-view` (`?invite=<token>`):** shows who invited them and their note. Accepting needs the matching email and the "I'm 18 or older" box ticked; it calls `/api/invite/accept` with `adult: true` and logs them in.
- **`portal-view`:** package picker (with the costume warning when `requires_compliance`), order status with pay-by deadline, payment instructions, cancel/refund buttons, the event card, RSVP, the QR entry pass once paid, and "invite your people" (2 invites).
- **Event card (`#event-card`):** filled from `/api/event`. Hidden when nothing is set. Paid-only details show once the order is paid; until then a note says more details unlock after payment.
- **Reserving:** the "I agree to the terms" box must be ticked (sent as `terms: true`), and packages with sizes need one picked from "Your size". If `/api/orders` answers `code: 'AGE_REQUIRED'` (guests who joined before the 18+ check), the page asks "Are you 18 or older?" with `ask()` and resends with `adult: true`. The order panel lets the guest change the size (`/api/orders/size`).
- **Footer and terms:** `/api/site` (public) is loaded on every visit. The footer shows the organizers' WhatsApp, Instagram and email when set, plus a "Terms" link that opens `#terms-dlg`. The login screen points people who don't get the email to the contact details.
- The session is kept in `localStorage` as `control_token` and `control_user_email`. Requests go through `authFetch()` with `Authorization: Bearer <token>`.
- The QR code encodes `ONDROAD:<reference>:<email>`; the door scanner in `control/door.js` depends on that format.

## Conventions

- Use `toast()` for notices and `ask()` for confirm/prompt dialogs, never `alert`/`confirm`/`prompt`.
- Set user data with `textContent`, not `innerHTML`.
- Colors and fonts are CSS variables in `:root`; reuse them.
- Animations respect `prefers-reduced-motion`. Keep `:focus-visible` outlines.
- Guest-facing text is short, plain and on-brand ("Get in", "Bring your people").
