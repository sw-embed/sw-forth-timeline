# sw-forth-timeline

A timeline of Forth, from the IBM 1130 to my own Forths, with live
demos. It is one wide strip that scrolls sideways. Each event is a
callout, and my own Forths have a picture that links to a live demo.
The timeline is written in Rust and drawn in the browser with Yew and
WebAssembly. It is forked from the APL timeline in
[sw-apl](https://github.com/sw-vibe-coding/sw-apl).

**[Open the live Forth Timeline](https://sw-embed.github.io/web-sw-cor24-forth/forth-timeline/)**

The timeline is published as part of the
[COR24 Forth live demo](https://sw-embed.github.io/web-sw-cor24-forth/)
([web-sw-cor24-forth](https://github.com/sw-embed/web-sw-cor24-forth)).

My Forths on the timeline, each with a live demo:

- [COR24 Forth](https://sw-embed.github.io/web-sw-cor24-forth/)
- [proto-forth-wasm](https://sw-vibe-coding.github.io/sw-fth-wasm/)
- [Forth in SWTOS](https://swtos.softwarewrighter.com/forth/)
- [reva-fork](https://softwarewrighter.github.io/reva-fork/)
- [GA144 Studio](https://sw-embed.github.io/sw-ga144/)

## Layout

```text
data/timeline.json      the events, compiled into the wasm
components/timeline/    the Rust workspace
  crates/forth-timeline       events, scale and callout layout (no browser)
  crates/forth-timeline-web   the Yew app that draws the strip
pages/                  the built site: index.html, machines/*.webp, timeline/
```

## Add an event

Add a line to `data/timeline.json`. Each event has these fields:

- `year`: where the event sits on the axis, as a year with a fraction
  for the month.
- `when`: the date as it is shown.
- `title` and an optional `text`.
- `kind`: `note` (a line of history), `key` (a key event), or
  `mine` (one of my Forths, at the end).
- Optional fields: `image` (a path under `pages/`, such as
  `machines/cor24.webp`), `href`, `demo` (the label under the
  picture), and `tip` (what the picture says on hover).

The `mine` events sit past 2026 on the axis, two years apart, so their
callouts do not overlap. Their `when` gives the real date.

Pictures go in `pages/machines/` as WebP files, 640 pixels wide.

```sh
magick screenshot.png -resize 640x pages/machines/name.webp
```

Then rebuild, because the events are compiled into the wasm.

## Build

You need a Rust toolchain with Edition 2024 support, the
WebAssembly target, a `wasm-bindgen` CLI whose version matches
`wasm-bindgen` in `components/timeline/Cargo.lock`, and `wasm-opt`
from [Binaryen](https://github.com/WebAssembly/binaryen).

```sh
rustup target add wasm32-unknown-unknown
cargo install --locked wasm-bindgen-cli --version <version from Cargo.lock>
```

Test, build, and write the site's wasm and JavaScript to
`pages/timeline/`.

```sh
cd components/timeline
cargo test
cargo build --release --target wasm32-unknown-unknown -p forth-timeline-web
wasm-bindgen --target web --no-typescript --out-dir ../../pages/timeline \
  target/wasm32-unknown-unknown/release/forth_timeline_web.wasm
cd ../..
wasm-opt -Oz --enable-bulk-memory --enable-nontrapping-float-to-int \
  --enable-sign-ext --enable-mutable-globals \
  pages/timeline/forth_timeline_web_bg.wasm \
  -o pages/timeline/forth_timeline_web_bg.wasm
```

Serve `pages/` with any static HTTP server to view it.

## Publish

The built `pages/` directory is published by web-sw-cor24-forth. Its
`scripts/update-forth-timeline.sh` copies `pages/` into its own
`pages/forth-timeline/`. Commit and push that repository to deploy.

## License

[MIT License](LICENSE). [Copyright (c) 2026 Michael A Wright](COPYRIGHT).
