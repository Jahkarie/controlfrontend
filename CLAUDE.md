# On D' Road: guest site (`controlfrontend`)

On D' Road is an invite-only, 18+ Antigua Carnival band. This repo is the guest-facing site. The API, admin pages and door scanner are in the separate `control` repo, whose `CLAUDE.md` describes the backend, known issues and features not built yet.

## Stack

- One static file, `index.html`: HTML, CSS and plain JavaScript. No build step, no framework.
- Only external script: `qrcodejs` from cdnjs, for the entry pass.
- Calls the backend at `BACKEND_URL` (set near the top of the `<script>`, currently `https://control-1-0baa.onrender.com`). The backend's CORS list must include wherever this page is served (`ondroad.xyz`, `www.ondroad.xyz`, `FRONTEND_URL`).

## Page flow

- **`login-view`:** enter email → `/api/login` emails a one-time link.
- **`?login=<code>`:** exchanged at `/api/login/verify` for a session token.
- **`accept-view` (`?invite=<token>`):** shows who invited them and their note; accepting with the matching email calls `/api/invite/accept` and logs them in.
- **`portal-view`:** package picker (with the costume warning when `requires_compliance`), order status with pay-by deadline, payment instructions, cancel/refund buttons, RSVP, the QR entry pass once paid, and "invite your people" (2 invites).
- The session is kept in `localStorage` as `control_token` and `control_user_email`. Requests go through `authFetch()` with `Authorization: Bearer <token>`.
- The QR code encodes `ONDROAD:<reference>:<email>`; the door scanner in `control/door.js` depends on that format.

## Conventions

- Use `toast()` for notices and `ask()` for confirm/prompt dialogs, never `alert`/`confirm`/`prompt`.
- Set user data with `textContent`, not `innerHTML`.
- Colors and fonts are CSS variables in `:root`; reuse them.
- Animations respect `prefers-reduced-motion`. Keep `:focus-visible` outlines.
- Guest-facing text is short, plain and on-brand ("Get in", "Bring your people").
