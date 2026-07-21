# mocview.app — VPx share-link handling (deploy to the GitHub Pages repo)

These files make a shared VPx link (`https://mocview.app/v/<code>`) open the VPx.
They belong in the **mocview.app GitHub Pages repository**, not this app repo —
they're authored here as deliverables. Copy them over and commit.

## What each file does

- **`404.html`** — GitHub Pages serves this for any path it doesn't have a file
  for, so it catches every `/v/<code>`. When the path is a share link it shows a
  branded landing and opens the native app via the `mocview://v/<code>` custom
  scheme (with a manual "Open in the MOCVIEW app" button + an App Store
  fallback). Any other path shows the normal "Page Not Found". Drop it at the
  **root** of the Pages site (replacing the current 404 page, or merge its
  `/v/` logic into yours).

## How the flow works today (custom scheme — live now)

1. Someone taps `https://mocview.app/v/<code>` (LinkedIn, browser, etc.).
2. GitHub Pages renders `404.html`; it detects `/v/<code>` and redirects to
   `mocview://v/<code>`.
3. iOS opens the MOCVIEW app (the `mocview` URL scheme is registered in
   `ios/Runner/Info.plist`); the app resolves the code through the signer and
   plays the VPx in-app (MOCVIEW-only — never a portable file).
4. No app installed / desktop → the landing stays, offering the App Store link.

## The later "Both" upgrade (seamless universal links)

To make the `https://` link open the app **directly** (no landing interstitial)
on iOS, three things are needed — all already prepared except the Apple-portal step:

1. **`.well-known/apple-app-site-association`** (included here) — serve it at
   `https://mocview.app/.well-known/apple-app-site-association` over HTTPS with
   **no redirect**. It declares `J4V2LADZMP.com.mocview-air.app` for `/v/*`.
   (GitHub Pages serves extensionless files fine; iOS parses it as JSON
   regardless of content-type.)
2. **Enable "Associated Domains"** on the App ID `com.mocview-air.app` in the
   Apple Developer portal, and regenerate the provisioning profile.
3. **Add the entitlement** `applinks:mocview.app` to the iOS app (a one-line
   `Runner.entitlements` addition) — deferred for now because it touches the CI
   signing/provisioning. The app's deep-link handler **already accepts the
   `https://mocview.app/v/*` form**, so once (1)–(3) land, seamless open works
   with no further app-code change.

Team ID: `J4V2LADZMP` · Bundle ID: `com.mocview-air.app`
