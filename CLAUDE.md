# On D' Road: guest site (`controlfrontend`)

On D' Road is an invite-only, 18+ Antigua Carnival band. This repo is the guest-facing site. The API, admin pages and door scanner are in the separate `control` repo, whose `CLAUDE.md` describes the backend, known issues and features not built yet.

## Stack

- One static page, `index.html`: HTML, CSS and plain JavaScript. No build step, no framework.
- Images next to it: `og.png` (1200×630 link preview), `favicon.svg`, `favicon-32.png`, `apple-touch-icon.png`. The `og:url` and `og:image` tags point at `https://ondroad.xyz/`.
- External scripts: `qrcodejs` from cdnjs, for the entry pass, and PayPal's button script from `www.paypal.com`, loaded only when the guest's reserved order can be paid with PayPal.
- Calls the backend at `BACKEND_URL` (set near the top of the `<script>`, currently `https://control-1-0baa.onrender.com`). The backend's CORS list must include wherever this page is served (`ondroad.xyz`, `www.ondroad.xyz`, `FRONTEND_URL`).

## Page flow

- **`login-view`:** enter email → `/api/login` emails a one-time link.
- **`?login=<code>`:** exchanged at `/api/login/verify` for a session token.
- **`accept-view` (`?invite=<token>`):** shows who invited them and their note. Accepting needs the matching email and the "I'm 18 or older" box ticked; it calls `/api/invite/accept` with `adult: true` and logs them in.
- **`portal-view`:** a header (`#portal-title`, `#portal-sub`) that says "You're in." (no order), "Spot held." (reserved, with the pay-by date) or "You're going." (paid, with package, size and the event's "when"); the package picker (with the costume warning when `requires_compliance`), order status with pay-by deadline, payment instructions, cancel/refund buttons, the event card, RSVP, the QR entry pass once paid, and "invite your people" (2 invites).
- **Package cards:** the price is `price_cents` from `/api/packages`, which is the current tier's price. With a `tier`, the card shows its name, "Until …", "N left at this price" and "Then …" (`next_tier`). `sales_closed` packages show "Sales closed" and can't be picked.
- **Codes (`#promo-box`):** "Have a code?" checks it with `/api/promo/check` and shows what the guest pays; the code goes to `/api/orders` as `promo_code` along with the `tier_id` and `price_cents` the card showed. A free code turns the button into "Claim my free pass" and the order comes back `PAID`. On a reserved order, "Have a code?" uses `/api/orders/promo` (hidden once a PayPal checkout has started). Codes: `PROMO_INVALID` (shown at the code), `PRICE_CHANGED` (reload packages and show the new price), `SOLD_OUT` (reload packages). The code UI only appears when the backend sends `sales_closed`, so an older backend never shows it.
- **Order panel:** the amount is the order's own (`amount_cents`); `#order-price-note` shows the tier and code; the size select is disabled when `size_locked`; "How to pay" shows the general instructions and the package's `pay_note`; `#order-pickup` shows when the package was collected. Free passes (`paid_via === 'COMP'`) get their own wording, and the refund button reads "Cancel my free pass".
- **Event card (`#event-card`):** filled from `/api/event`. Hidden when nothing is set. Paid-only details show once the order is paid; until then a note says more details unlock after payment.
- **Reserving:** the "I agree to the terms" box must be ticked (sent as `terms: true`), and packages with sizes need one picked from "Your size". If `/api/orders` answers `code: 'AGE_REQUIRED'` (guests who joined before the 18+ check), the page asks "Are you 18 or older?" with `ask()` and resends with `adult: true`. The order panel lets the guest change the size (`/api/orders/size`).
- **Pay now (`#paypal-box`):** on a reserved order, `/api/paypal/checkout` says whether PayPal is on for this guest and the USD amount. PayPal's buttons call `/api/paypal/create-order` and, once the guest approves, `/api/paypal/capture`. Codes: `ALREADY_PAID` (an earlier payment was found; reload the order), `PENDING` (PayPal is still processing; show the message instead of buttons), `CHECK` (problem with an earlier payment; contact us), `DECLINED` (`actions.restart()` so the guest can pick another card). When PayPal shows, the cash instructions are labelled "Other ways to pay".
- **Footer, terms, FAQ and sponsors:** `/api/site` (public) is loaded on every visit. The footer shows the organizers' WhatsApp, Instagram and email when set, a "Terms" link that opens `#terms-dlg`, an "FAQ" link, and a "With" row of sponsor logos (`#sponsors`; only `https:` links or relative paths such as `sponsors/acme.png`, which need a `sponsors/` folder in this repo). The FAQ is the `#faq` section after the account area, one `<details>` per `# ` question; it's hidden when the FAQ is empty. Terms and FAQ text go through `renderRich()` (`# ` headings, `- ` bullets, all set as text). The login screen points people who don't get the email to the FAQ and the contact details.
- The session is kept in `localStorage` as `control_token` and `control_user_email`. Requests go through `authFetch()` with `Authorization: Bearer <token>`.
- The QR code encodes `ONDROAD:<reference>:<email>`; the door scanner in `control/door.js` (entry and package pickup) depends on that format.
- The page must keep working with the previous backend, since this repo deploys first: treat missing fields as "feature off" and a 404 from a new endpoint as "not available".

## Conventions

- Use `toast()` for notices and `ask()` for confirm/prompt dialogs, never `alert`/`confirm`/`prompt`.
- Set user data with `textContent`, not `innerHTML`.
- Colors and fonts are CSS variables in `:root`; reuse them.
- Animations respect `prefers-reduced-motion`. Keep `:focus-visible` outlines.
- Guest-facing text is short, plain and on-brand ("Get in", "Bring your people").
