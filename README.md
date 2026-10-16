# local+ protocol demo

A no-install browser demo for Windows and macOS app developers exploring `local+` URL schemes.

**[Open the demo](https://victorhuangwq.github.io/local-plus-protocol-demo/)**

The page explains when to use `local+`, what changes for your app and its callers, and how to try it
with Chrome Canary or Edge Canary. Browser setup, expected results, and limitations are all on the page.

## Edit and preview

Everything is in [index.html](index.html). No dependencies, build step, or backend.

To preview locally, run this from the repo folder with Python installed:

```sh
python -m http.server 8000 --bind 127.0.0.1
```

Open `http://127.0.0.1:8000/`. Stop the server with `Ctrl+C`.

GitHub Pages publishes the root of `main`; [.nojekyll](.nojekyll) disables Jekyll processing.
