# PMP Practice Exam Simulator

A static, no-build web app that simulates a full PMP practice exam.

- **Payment page** first (price + Pay button that opens your hosted checkout), then the login page (username `pmpractice01`) gates access to the practice session.
- **Instructions page** explains the format and timing.
- **180 questions** (case studies, single and multiple-select) with a **NEXT** button, Previous, flagging and a question navigator.
- **4-hour exam timer** and one **20-minute break** (the exam clock pauses during the break).
- Auto-submit at zero, score + full answer review at the end, progress saved in the browser.

## Run locally
Open `index.html` in a browser (no server needed), or run `python -m http.server` and visit http://localhost:8000.

## Deploy on GitHub Pages
1. Push this folder to a GitHub repository.
2. Settings -> Pages -> Deploy from branch -> `main` / root.
3. Open the published URL.

## Update the questions
```bash
pip install openpyxl
python tools/convert_xlsx.py PMP_practice_Q_A.xlsx questions.js
```
The converter reads the `180 Question PMP` sheet (questions, options, answer key, case studies).

## Payment setup
Open `index.html` and edit the two lines under "Payment settings": `PAYMENT_URL` (your Stripe Payment Link / PayPal / Gumroad checkout URL) and `PRICE_LABEL` (e.g. "$29").
After a buyer pays, send them the exam's username and password (most providers can include this in the confirmation email). Until `PAYMENT_URL` is set, the Pay button is disabled.

## Security note
GitHub Pages is static hosting, so the login is a client-side gate only: credentials are stored as SHA-256 hashes, which keeps casual visitors out but is **not real security**. Anyone who can read the source can bypass it, and the answer key ships with `questions.js`. For genuine access control use a private repo or a backend/auth service.

To change the credentials, replace `USER_HASH` and `PASS_HASH` in `index.html` with new SHA-256 hashes.

## Payment (PayPal, one-time $9.99)
The first screen shows the price and a PayPal hosted checkout button.
1. In PayPal: Pay Links and Buttons -> create a **$9.99** button for Exam 1 and copy its **Hosted Button ID**.
2. In `index.html` set `HOSTED_BUTTON_ID = "..."` (the PayPal client ID is already filled in). Commit and push.
3. The button only works on a normal website (GitHub Pages, your own domain), not inside sandboxed preview frames.

## Emailing the username and password automatically
PayPal cannot send custom credentials by itself, and a static page cannot safely send email. Use one of these:
- **No code (recommended):** a Zapier or Make scenario - trigger "PayPal: new payment for this button/amount", action "send email" (Gmail/Outlook/Mailgun) to the payer's email with this exam's username and password.
- **Own server:** a small webhook (Cloudflare Worker, Netlify/Vercel function) that receives PayPal's `PAYMENT.CAPTURE.COMPLETED` event, verifies it, and emails the credentials with Resend/SendGrid.

This exam's login is `pmpractice01` / `Passw0rd123` (hashed inside `index.html`). Everyone who pays receives the same login, and the
page cannot tell who paid, so the payment screen is a convenience, not a lock: anyone who has the login (or reads the page source) can get in.
For per-buyer accounts you need a backend that creates a unique login per payment.
