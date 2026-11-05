# ERANFREE – PWA + Trusted Web Activity kit (Next.js on Vercel)

This kit adds install support, app icons and safe offline support to the
existing ERANFREE Next.js site **without changing its pages or features**.
It has **not** been deployed, and **no APK has been built**.

## 1. Before anything: replace the stand-in logo

The circular ERANFREE logo was not received, and the live site's current icons
(`/icon.svg`, `/apple-icon.png`) are the default v0.app logo, not ERANFREE.
So every icon in `public/` was made from a **stand-in** (`source-logo/stand-in-logo.svg`: blue circle, white
"EF"). Regenerate all icons from the real logo (square, ideally 1024×1024 PNG or SVG):

```bash
./scripts/make-icons.sh path/to/eranfree-logo.png   # needs ImageMagick 7
```

## 2. Copy into the Next.js project

| Kit file | Goes to |
| --- | --- |
| `public/*` (incl. `public/.well-known/`) | project `public/` |
| `components/pwa-register.tsx` | project `components/` |
| `app-layout-snippet.tsx` | merge into `app/layout.tsx` (metadata, viewport, `<PwaRegister />`) |
| `vercel.json` | project root (merge `headers` if one exists) |

Remove the old `app/manifest.*`, `app/icon.*`, `app/apple-icon.*` files if
present — Next.js would otherwise emit duplicate icon/manifest tags.
The old `public/icon-light-32x32.png`, `icon-dark-32x32.png`, `icon.svg`,
`apple-icon.png` can stay until the new ones are live, then be deleted.

Deploy with `vercel --prod` (or push to the production branch).

## 3. After deploying — URLs to check

- Manifest: https://earnfree-blush.vercel.app/manifest.webmanifest
- Service worker: https://earnfree-blush.vercel.app/sw.js
- Offline page: https://earnfree-blush.vercel.app/offline.html
- Icons:
  - https://earnfree-blush.vercel.app/icons/icon-192.png
  - https://earnfree-blush.vercel.app/icons/icon-512.png
  - https://earnfree-blush.vercel.app/icons/maskable-192.png
  - https://earnfree-blush.vercel.app/icons/maskable-512.png
  - https://earnfree-blush.vercel.app/apple-touch-icon.png
  - https://earnfree-blush.vercel.app/favicon.ico, /favicon-32x32.png, /favicon-16x16.png
- Digital Asset Links: https://earnfree-blush.vercel.app/.well-known/assetlinks.json

Validate in Android Chrome → menu → "Install app", and in desktop Chrome
DevTools → Application → Manifest / Service workers. Run Lighthouse (PWA
checks moved to the Application panel in recent Chrome).

## 4. What the service worker does

- **Caches only**: offline page, icons, manifest, and hashed build files in `/_next/static/`.
- **Never caches**: any page HTML, Next.js RSC/data requests, `/api/*`, login/account/
  wallet/points/rewards/customer paths, anything with an `Authorization` header,
  any non-GET request, and anything on another domain (database, auth, payments).
  So account data, tokens, customer records and loyalty balances are never stored.
- **Offline**: page loads that fail show `/offline.html`.
- **Updates**: a new version waits; users see an "Update" toast and the app reloads once.
  Bump `VERSION` in `sw.js` whenever you change it.
- **Kill switch**: open any page with `?sw=off` to unregister.

## 5. Mobile / Android Chrome fixes included

- Viewport no longer blocks pinch-zoom (`maximum-scale=1, user-scalable=no` removed —
  flagged by Lighthouse accessibility audits).
- `viewport-fit=cover` + brand `theme-color` `#1765D1` for the Android status bar / TWA.
- Recommended in your CSS: `padding-bottom: env(safe-area-inset-bottom)` on fixed bottom
  nav bars; tap targets ≥ 48px; inputs ≥ 16px font-size to stop zoom-on-focus.

## 6. Content changes required (not just wording)

The live site currently says: *"Invest, Earn & Grow Daily"*, *"deposit, activate VIP
plans, earn daily income, refer friends and withdraw to your UPI"*, *"Activate a plan,
claim daily income and withdraw straight to your UPI."*

Those describe deposit-for-profit / guaranteed-return features. To be a legitimate
loyalty platform (and to pass Google Play review, which rejects these under its
Financial Services and deceptive-behaviour policies), remove the **features** as well
as the text:

- [ ] Remove deposits, "VIP plans" / paid plan activation and "daily income" claims
- [ ] Remove cash withdrawals to UPI; rewards should be vouchers, discounts, free items
- [ ] Remove cash referral commissions / multi-level referral earnings
- [ ] Remove any "guaranteed", "returns", "profit", "invest", "double" wording
- [ ] Add Terms, Privacy Policy and a clear "points have no cash value" statement

Suggested replacement copy:
- Title: **ERANFREE - Business Loyalty & Rewards**
- Hero: **Earn points every time you shop** — "Collect points at participating
  businesses and redeem them for vouchers, discounts and free items."
- Feature chips: **Points on purchases · Partner rewards · Easy redemption**

## 7. Next step toward an APK (not done)

1. Pick a package name (e.g. `app.eranfree.twa`) and create a signing key.
2. Use Bubblewrap (`bubblewrap init --manifest https://earnfree-blush.vercel.app/manifest.webmanifest`)
   or PWABuilder.com to generate the Android project.
3. Put the package name and SHA-256 fingerprint(s) into `public/.well-known/assetlinks.json`
   and redeploy — without this the TWA shows a browser address bar.
4. Build, test on a real device, then submit.
