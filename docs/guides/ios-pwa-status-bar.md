# iOS PWA top-edge rendering

NAN-162 / [#5772](https://github.com/HKUDS/nanobot/issues/5772) tracks washed-out
controls at the top of an installed iOS PWA. This is separate from the mobile
sidebar focus and double-tap fixes in #5839.

## Compatibility surface

The initial HTML includes one empty, non-interactive `.pwa-status-bar-surface`
outside React's root. It only renders in standalone mode on engines supporting
`-webkit-touch-callout` and text background clipping. Normal browser tabs and
Chromium PWAs keep it `display: none`.

WebKit's [fixed-container sampling implementation](https://github.com/WebKit/WebKit/blob/d620932d8ce37cf83f60d5c50dace2d50fed4f80/Source/WebCore/page/LocalFrameView.cpp)
looks for fixed/sticky elements near the viewport edge, covering at least 90% of
its width. A primary background must exceed the 10px thin-border threshold and
must not be hidden or nearly transparent. Sampling can ignore `pointer-events`.
Nanobot's relative/absolute header and small button backgrounds do not meet that
contract across the top edge.

The surface is 11px high, viewport-fixed, full-width, and above the application
layers. It inherits the body's background color, including startup and live theme
changes. Clipping that background to empty text prevents it from painting over
controls while retaining the background style for WebKit's sampler. It has no
focusable content and cannot intercept taps. It does not add padding, move the
header, change viewport fitting, or animate an artificial overlay.

This is a compatibility workaround, **not a supported Apple status-bar API**.
Independent device experiments describing the same mechanism:

- [Empty-text fixed surface on iOS 27](https://qiita.com/na-trium-144/items/0add98a80ca2391e3f17)
- [iOS PWA blur experiments, CSS-only update](https://tips.ojapp.app/en/ios-27-pwa-top-blur-workaround-2/)

Keep `viewport-fit=auto` and the existing status-bar metadata. Do not replace this
with a guessed safe-area inset or claim that a desktop screenshot verifies iOS
system compositing.

## Device acceptance gate

HTML/CSS contract tests and desktop layout checks verify scoping, color inheritance,
non-interference and safe-area preservation. They cannot prove the reported iOS
system blur is gone. Before marking NAN-162 resolved, record the actual device,
iOS version and nanobot commit, then compare the same device before/after:

1. Completely quit and relaunch the installed PWA; verify the latest HTML is served.
2. Check sidebar/theme controls on new and existing chats, before and after scrolling.
3. Switch light/dark themes, rotate portrait/landscape, and open/close the keyboard.
4. Open and close the sidebar and settings; verify top controls and one-tap navigation.
5. Compare an ordinary Safari tab and an unaffected browser to rule out added space,
   painted strips, blocked input or zoom regressions.

If the workaround stops matching WebKit behavior, re-check the upstream sampling
implementation and device results rather than increasing padding or layer sizes.
The separately reported portrait `@` selection failure is not addressed here.
