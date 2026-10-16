# local+ protocol demo

A short guide and no-install browser demo for Windows and macOS app developers.

**[Open the demo](https://victorhuangwq.github.io/local-plus-protocol-demo/)**

## Should websites be able to open this entry point in your app?

Your desktop app may handle custom URLs so other native apps and websites can open it.
Use `local+` for entry points that other native apps need to call, but websites should not launch.
In supporting browsers with the feature enabled, the prefix blocks web-initiated launches of that scheme.
Native apps can still call it through the OS.

## What to register and what callers change

| | Ordinary entry point | Local-only entry point |
| --- | --- | --- |
| Protocol your app registers | `myapp:` | `local+myapp:` |
| URL another native app opens through the OS | `myapp:hello` | `local+myapp:hello` |
| Website link in a supporting browser with the feature enabled | Normal app-opening flow | Blocked |

On **Windows**, register `local+myapp` using your usual protocol-registration mechanism.
On **macOS**, declare `local+myapp` in your app's URL schemes.
Registration uses the scheme name without the trailing colon.

Update callers to open the full URL, such as `local+myapp:hello`, and make sure your handler accepts the new scheme.
The OS dispatches that URL to your app as usual.

**It's a new protocol name, not a setting on the old one.** There is no automatic mapping between the two names.
Keeping the old `myapp:` scheme wired to the same functionality leaves that entry point reachable from websites.

**Keep validating incoming URLs.** This is a browser restriction, not authentication or OS-level protection.
Unsupported browsers may not enforce it, and administrator policies can override it.

## See the browser block, no app installation needed

The demo uses `toydemo:` and `local+toydemo:`. No native app, extension, JavaScript, dependencies, or backend.
Compare a request that reaches app lookup with one the browser rejects before lookup:

| URL | Feature enabled, no registered handler (Windows/macOS) |
| --- | --- |
| `toydemo:hello` | The browser looks for an installed app. This demo hasn't installed one, so the console reports no registered handler. |
| `local+toydemo:hello` | The browser refuses the website's request before looking for an app. The console reports `Not allowed to launch 'local+toydemo:hello'.` |

These are expected results under the conditions below, not outcomes detected by the website.
The page cannot read browser-generated console messages or reliably detect an external app launch.

## Browser setup

<details>
<summary>Set up your test browser</summary>

Use the latest **Chrome Canary** or **Edge Canary** and explicitly enable `LocalOnlyAppProtocolPrefix`.

Run one command below, then open the demo in the new browser window.
The extra `--user-data-dir` flag keeps the demo separate from your normal browser.
Close any previous demo window before running the command again.

### Windows (PowerShell)

Chrome Canary:

```powershell
& "$env:LOCALAPPDATA\Google\Chrome SxS\Application\chrome.exe" --user-data-dir="$env:TEMP\local-plus-chrome" --enable-features=LocalOnlyAppProtocolPrefix
```

Edge Canary:

```powershell
& "$env:LOCALAPPDATA\Microsoft\Edge SxS\Application\msedge.exe" --user-data-dir="$env:TEMP\local-plus-edge" --enable-features=LocalOnlyAppProtocolPrefix
```

### macOS (Terminal)

Chrome Canary:

```sh
open -na "Google Chrome Canary" --args --user-data-dir="$TMPDIR/local-plus-chrome" --enable-features=LocalOnlyAppProtocolPrefix
```

Edge Canary:

```sh
open -na "Microsoft Edge Canary" --args --user-data-dir="$TMPDIR/local-plus-edge" --enable-features=LocalOnlyAppProtocolPrefix
```

The commands use standard Canary installation locations and store demo profiles in your temporary directory.
Nothing else is installed.

</details>

## Walkthrough

1. Open the demo in the test browser. Open DevTools Console with `Ctrl+Shift+J` on Windows or
   `Command+Option+J` on macOS. Keep error messages visible.
2. Click **Try ordinary protocol**. With no registered handler, expect a console error containing
   `because the scheme does not have a registered handler`.
3. Click **Try local-only protocol**. Expect `Not allowed to launch 'local+toydemo:hello'.`

The page can remain visually unchanged in both cases. Compare the console, not whether a window opens.
Click each link separately so ordinary user-gesture and launch-throttling behavior does not confound the comparison.
If an app happens to be registered for a demo scheme, the ordinary flow may show an app-opening prompt instead;
cancel it unless you intend to open that app.

For an A/B comparison, close the demo browser, change the profile folder name to end in `-off`, and replace
`--enable-features=LocalOnlyAppProtocolPrefix` with `--disable-features=LocalOnlyAppProtocolPrefix`.
Both links should then reach normal handler lookup, absent other restrictions.

## Interpretation and limits

- `local+` is part of the protocol's actual name. There is no automatic mapping from `toydemo:` to `local+toydemo:`.
- Clicking a link is still web-initiated, even though it has a user gesture.
- No installed handler is necessary to observe the new block. An installed handler is necessary to demonstrate
  a successful native-app launch, including the address-bar exception, end to end.
- Browser-initiated navigation such as an address-bar entry retains the existing prompt flow.
  Native app-to-app launches through the OS do not pass through this Chromium check.
- Remembered user allowances do not override the block. Administrator `AutoLaunchProtocolsFromOrigins`
  and applicable `URLAllowlist` policies can. The landed tests also cover management-state restrictions on
  `URLAllowlist` for Windows and macOS. A fresh browser profile does not remove machine policies.
- This is not OS-level access control or a guarantee about unsupported browsers.
- Failure to see the expected message can indicate an older build, a missing feature flag, enterprise policy,
  hidden console errors, or another browser restriction. Do not interpret absence of an app window as proof.

The expected behavior is based on the [landed implementation and tests](https://chromium-review.googlesource.com/c/chromium/src/+/8382199),
not a claim that every currently distributed browser supports the feature.

## Hosting and local preview

GitHub Pages serves [index.html](index.html) directly from the root of `main`. No build step is needed.
The [.nojekyll](.nojekyll) file disables Jekyll processing.

To preview over HTTP locally, if Python is installed:

```sh
python -m http.server 8000 --bind 127.0.0.1
```

Open `http://127.0.0.1:8000/` in the test browser. Stop the server with `Ctrl+C`.
Localhost is still web content for this check.
