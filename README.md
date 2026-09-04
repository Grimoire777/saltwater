# Saltwater — night stills

Eighteen hand-generated night illustrations, graded and sized for the Saltwater
render pipeline. Drop this folder in the repo, push, and every file is live at

```
https://grimoire777.github.io/saltwater/stills/<slug>.jpg
```

which is a direct URL the render service can download. Nothing else to host.

## What was done to them

Every file is **2848×1600, exactly 16:9, sRGB, no EXIF, JPEG q88**.

2848 rather than 1920 on purpose. `stillToClip` scales a still to twice the
render target — 3840×2160 — before `zoompan` samples it, and starting closer to
that number is what keeps the slow push from stepping between pixels.

Brightness was corrected with a **gamma curve on luminance only**. Gamma is
anchored at both ends, so the moon stays the moon and the black stays black
while the midtones — pale sand, mist, the parts that actually make a frame too
bright — come down. Running the curve per channel instead looks fine until the
correction gets heavy, then it multiplies saturation and any faint cast with it;
at gamma 2.9 it turned moonlit sand into golden desert. Deriving one gain from
luma and applying it to all three channels keeps hue and saturation exactly
where the painting put them.

**Correction is capped at gamma 2.2.** Past that you are not dimming a picture,
you are deleting the half of it that was already dark. Two frames hit the cap
and stayed slightly above the ideal rather than being crushed to reach it.

## The set

Mean is average luma out of 255; the library gate is **under 55**. p95 is the
95th percentile — how bright the brightest twentieth of the frame is, which is
what a mean average hides.

| slug | title word | mean | p95 | use | scene |
|---|---|---|---|---|---|
| `palm-milkyway` | Palms and Stars | 42.7 → 42.6 | 86 | opener | Coconut palm silhouette under the Milky Way |
| `moon-clouds-bay` | Moon and Cloud | 45.5 → 45.0 | 198 | bright area | Full moon over a bay, cumulus banked low, long reflection |
| `lantern-stream` | Lantern Light | 18.9 → 19.1 | 62 | opener | Lanterns on palm trunks over a still stream, fireflies |
| `dunes-fireflies` | Night Dunes | 92.2 → 54.9 | 193 | bright area | Pale dunes with marram grass, two fireflies |
| `hazy-moon` | Hazy Moon | 41.4 → 41.4 | 74 | opener | Diffused full moon behind haze, broken reflection |
| `beach-lounger` | Quiet Shore | 57.2 → 45.2 | 159 | bright area | Empty lounger and closed parasol on dark pebbles |
| `hanging-lantern` | Lantern Light | 63.2 → 45.2 | 178 | bright area | Single lantern hanging from a palm trunk |
| `moored-sailboat` | Still Harbour | 29.8 → 29.8 | 89 | opener | Sailboat moored at a pier under a crescent |
| `lily-pond` | Lily Pond | 38.1 → 38.1 | 67 | opener | Crescent reflected in a lily pond, fireflies |
| `bioluminescence` | Glowing Water | 41.4 → 41.4 | 117 | opener | Cyan bioluminescent surf on black rock |
| `distant-ship` | A Distant Ship | 76.7 → 45.3 | 128 | opener | Lit ship far out, surf line curving across frame |
| `misty-lagoon` | Misty Lagoon | 58.4 → 45.1 | 126 | opener | Shallow lagoon under drifting mist, green cast |
| `pier-crescent` | Quiet Pier | 41.2 → 41.1 | 83 | opener | Pier running out to a crescent and harbour lights |
| `dark-water-star` | Dark Water | 49.9 → 45.0 | 127 | opener | Open sea, one bright star, low rolling foam |
| `sleeping-village` | Sleeping Shore | 94.9 → 48.7 | 130 | bright area | Curving beach, huts among palms with dim windows |
| `hammock-lantern` | Lantern Light | 43.6 → 43.6 | 159 | bright area | Empty hammock between palms, brass lantern on sand |
| `shells-waterline` | The Waterline | 48.9 → 45.0 | 106 | opener | Shells and pebbles at the waterline, moon out of shot |
| `moonpath` | Moonpath | 38.3 → 38.2 | 66 | opener | Moonpath across flat sea, palm islet at the edge |

**Twelve are marked `opener`** — calm across the whole frame, safe as the first
thing on screen when someone presses play at 1am. The other six are fine
mid-session but carry a large bright area: a moonlit cumulus bank, a white dune,
a lantern. Average brightness does not catch those; p95 does.

## Importing them

`manifest.json` holds one ready payload per still. POST each to the render
service:

```
POST /jobs/import
{ "slug": "moonpath", "aspect": "16x9",
  "url": "https://grimoire777.github.io/saltwater/stills/moonpath.jpg",
  "dim": 0, "source": "davinci" }
```

**`dim` must be 0.** The night grade is already in the pixels. The service's own
grade would cool an already-cool palette a second time and push these past
useful into murky.

Each import produces a 10-second palindromic slow zoom — fully in at the
midpoint, back out by the end, so the last frame matches the first and
`buildLoop`'s crossfade has nothing to hide.

## One workflow change

WF-A's `sceneWords` map turns a scene key into the middle slot of the session
title. These scenes are new, so add them or the titles fall back to the generic
phrase:

```js
  'dark-water': 'Dark Water',
  'distant-ship': 'A Distant Ship',
  'dunes': 'Night Dunes',
  'glowing-water': 'Glowing Water',
  'hazy-moon': 'Hazy Moon',
  'lantern-light': 'Lantern Light',
  'lily-pond': 'Lily Pond',
  'misty-lagoon': 'Misty Lagoon',
  'moon-clouds': 'Moon and Cloud',
  'moonpath': 'Moonpath',
  'palm-stars': 'Palms and Stars',
  'quiet-pier': 'Quiet Pier',
  'quiet-shore': 'Quiet Shore',
  'sleeping-shore': 'Sleeping Shore',
  'still-harbour': 'Still Harbour',
  'waterline': 'The Waterline',
```

## Regenerate

Two images arrived square (1024×1024) and are not in this set. Cropping a square
to 16:9 costs a third of the frame, and upscaling 1024 to 1920 is visibly soft on
a television — neither is worth doing to a good painting. Both are worth having,
so re-run their prompts with the aspect set to 16:9 before generating:

- **Palm and Milky Way over a bay** — palm silhouette, Milky Way, full moon low
  over the sea with a long reflection. The best composition in the whole batch.
- **Moonpath with islet** — already re-generated at 2848×1600 and included here
  as `moonpath.jpg`. Nothing to do.
