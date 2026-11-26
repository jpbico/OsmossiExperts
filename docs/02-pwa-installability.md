# 2. Making the app installable (iPhone and Android)

Sources: MDN *Making PWAs installable* <https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Guides/Making_PWAs_installable>, MDN Web app manifest <https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Manifest>, MDN Service Worker API <https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API>, WebKit *Web Push for Web Apps on iOS and iPadOS* <https://webkit.org/blog/13878/web-push-for-web-apps-on-ios-and-ipados/>, Capacitor docs <https://capacitorjs.com/docs>.

## 2.1 Baseline requirements (MDN)

- Serve the app over **HTTPS**, or from `localhost` / `127.0.0.1` during development. Netlify serves HTTPS by default.
- Link a **web app manifest** (`<link rel="manifest" href="/manifest.webmanifest">`). Chromium browsers need:
  - `name` or `short_name`
  - `icons` with a **192px** and a **512px** icon
  - `start_url`
  - `display` and/or `display_override` (use `standalone`)
  - `prefer_related_applications` set to `false` or left out
- **Service worker.** MDN notes it is *not* required for installability, but we need it for an offline shell and caching. Register `/sw.js` with scope `/`.

Proposed manifest:

```json
{
  "id": "/",
  "name": "Osmossi Experts",
  "short_name": "Osmossi",
  "start_url": "/?source=pwa",
  "scope": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#0f172a",
  "icons": [
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png" },
    { "src": "/icons/maskable-512.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable" }
  ]
}
```

Plus for iOS, in `<head>`: `<link rel="apple-touch-icon" href="/icons/apple-touch-icon-180.png">` and `<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">`.

## 2.2 Android

- Chrome on devices with Google Mobile Services, and Samsung Internet on Samsung devices, install PWAs as **WebAPKs**, which behave like real apps (MDN). Firefox, Edge, Opera, and Chrome without GMS add a browser-badged home-screen shortcut instead.
- Custom install button: listen for `beforeinstallprompt`, call `preventDefault()`, keep the event, and call `prompt()` from our own **Install** button. Chromium only.

## 2.3 iPhone / iPad

- **No `beforeinstallprompt` on iOS** (MDN). Show a short "Share → Add to Home Screen" hint instead. Show it only in Safari or another iOS browser, only when the app is not already standalone (`matchMedia('(display-mode: standalone)')` / `navigator.standalone`), and let the user dismiss it.
- iOS 16.3 and earlier: install only from Safari. **iOS 16.4+**: from the Share menu in Safari, Chrome, Edge, Firefox, and Orion (MDN).
- **Web Push** (iOS 16.4+) works **only for Home Screen web apps**. The manifest must say `display: standalone` or `fullscreen`, and the permission request must follow a user tap (WebKit). The **Badging API** (`setAppBadge`) is also available, after notification permission.
- Practical limits to plan for: storage can be evicted (keep the source of truth on the server, in Blobs), and background sync is limited. For a posting app that is fine, because publishing runs on the server.

## 2.4 Service worker strategy

| Request | Strategy |
|---|---|
| App shell (HTML, CSS, JS, icons) | Precache on install, with a versioned cache name; delete old caches on `activate` |
| `/api/*` (functions) | **Network only**: never cache authenticated or POST responses |
| Generated preview images | Network-first with a short-lived cache, or none |
| Navigation while offline | Serve the cached shell with an "offline" banner; disable Publish |

Update flow: when a new service worker is waiting, show a "New version — Reload" toast, then `postMessage({type:'SKIP_WAITING'})`. This matters because Agent Console approvals cause frequent redeploys (doc 4).

## 2.5 When to use a native wrapper (Capacitor) instead

Capacitor is "a cross-platform native runtime" that wraps a web app in native iOS and Android shells, with plugins in Swift and Java (capacitorjs.com). **For now, a PWA is enough.** Move to Capacitor only if one of these becomes a requirement:

- Distribution through the **App Store / Google Play** (discoverability, MDM, store review).
- Native-only APIs: the Instagram **share sheet / Stories intent** to hand an image to the Instagram app, background tasks, reliable background push on older iOS, fuller photo-library access, or biometric unlock.
- iOS users before 16.4 who need push.

The same codebase can stay a PWA and later be wrapped with Capacitor, so designing for the web first loses nothing.

## 2.6 Install checklist (for testers)

- [ ] Lighthouse / Chrome DevTools → Application → Manifest shows no errors; icons are 192 and 512, and maskable is OK
- [ ] Android Chrome: the install prompt appears; the app opens standalone; the icon and splash are correct
- [ ] iOS Safari: Add to Home Screen; opens standalone; status bar and safe areas are OK (`viewport-fit=cover`, `env(safe-area-inset-*)`)
- [ ] Offline: the shell loads and Publish is disabled with a clear message
- [ ] After a deploy: the update toast appears and reloading gets the new version
