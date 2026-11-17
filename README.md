# webdev-scroll-site

A scroll-scrubbed landing page for my web development service: scrolling flies a camera through four pre-rendered scenes (idea, build, launch, open for business). It runs on `scrub-engine.js`, a zero-dependency vanilla JS scrub engine.

Live: https://webdev-scroll-site.vercel.app/

## Why it exists

I wanted a landing page that shows the process of buying a website instead of describing it, and that still behaves on a phone in Low Power Mode or on a slow connection. Video scroll-scrubbing usually breaks in exactly those places.

## How it works

- **The page is config, the engine is the product.** `index.html` is mostly one `mountScrollWorld(container, config)` call: each section declares its still, poster, desktop and mobile clips, accent color and copy. The engine builds its own DOM and injects namespaced CSS into the container, so it drops into plain HTML or any framework with no build step.
- **Clip tier follows device class, not input type.** A screen whose short side is 600 CSS px or less gets the mobile encode. An iPad Pro reports a coarse pointer but has a desktop-class screen and decoder, so it gets the full master and still receives the touch hardening.
- **Touch hardening.** On coarse-pointer or narrow viewports the engine coalesces seeks (never sets `currentTime` while the decoder is still seeking), keeps the poster up until the clip actually paints, primes each video with a muted play and pause on first touch (an iOS blank-video fix), and ignores URL-bar-only resizes.
- **A degradation ladder that never upgrades on a guess.** `prefers-reduced-motion` or data-saver switches to stills mode: the stills cross-dissolve on scroll with no video loaded. If iOS Low Power Mode refuses `play()`, the page drops to stills mode at runtime. Chromium's network hints can only downgrade the experience, since Safari exposes none of them.
- **No pop on the still-to-video swap.** Each poster is the extracted first frame of its own encoded clip, so the handoff is pixel identical.
- **Pacing.** A per-section `linger` value remaps clip time with an eased curve so the camera settles mid-scene where the copy peaks. Phones get a longer scroll run per scene because a small viewport makes the same flight read as faster.
- **Crawlable copy.** A `data-sw-seo` block holds a plain h1/h2/p version of the copy for crawlers and screen readers. The engine hides it on mount.
- Clips are fetched lazily as blobs near the viewport, with posters shown first.

## Stack

Vanilla JavaScript, HTML and CSS. No dependencies, no build step. Hosted on Vercel. The scenes are AI-generated camera flights, encoded in desktop and mobile tiers. Built with Claude Code.

## Run it locally

The posters and stills are in the repo. The eight MP4 clips (about 35 MB) are not; download them from the live deployment:

```bash
mkdir -p assets/vid
for n in studio workshop launchpad storefront; do
  for s in "" "-m"; do
    curl -fsSo "assets/vid/$n$s.mp4" "https://webdev-scroll-site.vercel.app/assets/vid/$n$s.mp4"
  done
done
python3 -m http.server 8000   # the engine fetches clips, so serve over http, not file://
```

Then open http://localhost:8000. Without the clips the page still loads and shows each scene's poster.

## Status and limits

- Live and used as the demo for my web development work.
- The engine supports connector clips (flights between scenes). This deployment leaves them empty, so scenes cross-dissolve into each other.
- Network-based downgrades rely on `navigator.connection`, which only Chromium browsers expose.
