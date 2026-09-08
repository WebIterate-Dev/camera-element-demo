# camera element demo

A minimal demo of Chrome's declarative `<camera>` element — a browser-owned
capture control that replaces the `navigator.mediaDevices.getUserMedia()` call
with markup:

```html
<camera id="cam">
  <button>Turn on camera</button>
</camera>
```

The browser renders the control, owns the permission prompt, and hands the page
a video track. Because the user agent controls the button, it gets a trusted
signal of user intent — which is what lets it offer better recovery when
permission was previously denied. `<camera>` is not a settled standard yet.

## Requirements

Chrome 153+ (the element shipped alongside `<microphone>`; the multi-capability
`<usermedia>` landed first, in Chrome 151). No other browser implements it.

The pages are served over HTTP because permission on a `file://` origin behaves
inconsistently:

## Notes

**1. `<camera>` fires `track`, not `stream`.** It's a _single-capability_
element, so it yields a `MediaStreamTrack`. (`<usermedia>` is the
multi-capability variant, and that one gives you a `MediaStream` via a `stream`
event.) `srcObject` only accepts a `MediaStream`, so the track has to be
wrapped:

```js
cam.addEventListener("track", (event) => {
  const track = cam.track ?? event.track;
  preview.srcObject = new MediaStream([track]);
  preview.play();
});
```

**2. The element is subject to PEPC activation rules.** It's built on Page
Embedded Permission Control, which refuses to activate — silently, apart from a
DevTools issue — unless the element is _fully_ inside the viewport, has been
visible for ~500 ms, and has not recently moved or been inserted into the DOM.
In practice: put it high in the page and make sure nothing above it reflows
after load. `debug.html` includes a watchdog that mirrors this check and tells
you whether a click will be accepted.

To bypass it while iterating:

```bash
open -na "Google Chrome" --args --runtime-enabled-features=BypassPepcSecurityForTesting
```

**3. Don't style the element.** The user agent validates and overrides its
presentation to keep the permission entry point legible; custom styling is
itself an activation blocker. Style a wrapper, never the `<camera>`.

**4. Feature-detect the capability, not the name.** The interface can be exposed
while the element isn't upgraded to it, leaving a control that renders but does
nothing:

```js
const supported =
  "HTMLCameraElement" in window &&
  cam instanceof HTMLCameraElement &&
  typeof cam.setConstraints === "function";
```

## References

- [Introducing the `<usermedia>` HTML element](https://developer.chrome.com/blog/usermedia-html-element)
- [WICG/PEPC — permission element spec](https://github.com/WICG/PEPC/blob/main/permission-element.bs)
- [Chrome 153 release notes](https://chromestatus.com/release-notes/153)
- [<camera> and <microphone> elements land in Google Chrome](https://webiterate.dev/capability-elements-camera-126/)

## License

[MIT](LICENSE) © 2026 Web Iterate
