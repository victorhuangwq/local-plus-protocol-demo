# local+ protocol demo

A minimal, no-install website demonstrating Chromium's `local+` external-protocol block.

**[Open the demo](https://victorhuangwq.github.io/local-plus-protocol-demo/)**

No native app, extension, JavaScript, dependencies, or backend. The page contains two ordinary links:

| URL | Feature enabled, no registered handler (Windows/macOS) |
| --- | --- |
| `toydemo:hello` | Normal external-handler lookup; console reports no registered handler |
| `local+toydemo:hello` | Blocked before handler lookup; console reports `Not allowed to launch 'local+toydemo:hello'.` |

These are expected results under the conditions below, not outcomes detected by the website.
The page cannot read browser-generated console messages or reliably detect an external app launch.

## Browser setup

Use a desktop Chromium-based build containing [Chromium change 8382199](https://chromium-review.googlesource.com/c/chromium/src/+/8382199),
merged September 23, 2026, as commit `51e74b2fa6a88d7da47643160bb990468ff82c02`
(main position `1703676`). It introduced `LocalOnlyAppProtocolPrefix`, disabled by default.
Landing upstream does not establish availability in any particular Chrome or Edge release.
An unknown `--enable-features` name can be silently accepted by an older build, so the flag alone is not proof of support.

Launch your compatible browser with a fresh, dedicated test profile. This avoids reusing an existing process
that was started without the feature flag and keeps your normal browser profile separate.
Do not disable browser security protections.

### Windows (PowerShell)

Replace the executable path with the compatible browser you intend to test:

```powershell
$browser = 'C:\path\to\compatible-browser.exe'
$profile = Join-Path $env:TEMP ('local-plus-demo-' + [guid]::NewGuid())
& $browser "--user-data-dir=$profile" '--enable-features=LocalOnlyAppProtocolPrefix' 'https://victorhuangwq.github.io/local-plus-protocol-demo/'
```

### macOS (Terminal)

Replace the executable path with the compatible browser you intend to test.
For example, a Chrome Canary installation normally uses the path shown below, but its version must still contain the change:

```sh
BROWSER='/Applications/Google Chrome Canary.app/Contents/MacOS/Google Chrome Canary'
PROFILE="$(mktemp -d -t local-plus-demo)"
"$BROWSER" --user-data-dir="$PROFILE" --enable-features=LocalOnlyAppProtocolPrefix \
  'https://victorhuangwq.github.io/local-plus-protocol-demo/'
```

These commands launch a browser, not an app handler or installer. They create a test profile in your temporary
directory. Close that test browser when finished; you can then remove its specific profile folder.
Do not delete your normal browser profile.

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

For an A/B comparison, launch the same compatible browser in another fresh profile, replacing
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
