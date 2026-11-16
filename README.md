# NPO Player: Chapters

An experiment that adds chapters to the [NPO Player](https://cdn.npoplayer.nl/player/v2/npoplayer.js). It:

- loads a stream in the NPO Player with chapter markers (`timeBoundActions` of type `marker`);
- breaks the seek bar into segments at every chapter start and end;
- shows the chapter title in the seek bar's hover tooltip;
- lists the chapters below the player. Click one to jump to it.

Everything is in `index.html`. There is no build step and nothing to install. The player scripts and [`jose`](https://github.com/panva/jose) (used to sign the JWT) load from CDNs, so you need an internet connection.

## Requirements

- A modern browser. The page uses ES modules, top-level `await`, CSS masks and `Array.prototype.findLast`.
- Any static file server, for example Python 3 or Node.js (`npx`).
- The VPRO signing secret for the NPO Player (see [Configuration](#configuration)).

## Starting the project

Serve the folder over HTTP. Opening `index.html` directly as a `file://` URL is not supported, because module scripts and `localStorage` don't work reliably there.

```sh
# with Python
python3 -m http.server 8000

# or with Node.js
npx serve .
```

Then open <http://localhost:8000> (or the URL that `npx serve` prints).

## Configuration

### Signing secret (required)

The player needs a JWT that is signed with HS256 using the VPRO NPO Player secret. The page reads that secret from `localStorage` under the key `npo-player-key`, so it never has to be in the source code.

Set it once per browser and origin. Open the page, open the developer console and run:

```js
localStorage.setItem("npo-player-key", "<your-secret>");
```

Then reload the page.

`localStorage` is stored per origin, so set it again if you change host or port (for example `localhost:8000` versus `127.0.0.1:8000`).

To remove the secret:

```js
localStorage.removeItem("npo-player-key");
```

### Other settings in `index.html`

| What | Where | Default |
| --- | --- | --- |
| Media ID (MID) of the stream | `signJWT("VPWON_1351929", …)` | `VPWON_1351929` |
| JWT issuer | `.setIssuer("vpro")` in `signJWT` | `vpro` |
| Chapters | `timeBoundActions` array (`startTime`/`endTime` in seconds, `label`) | three sample chapters |
| Start position | `startOffset` in `player.load(…)` | `200` seconds |
| Maximum label length | `MAX_LABEL_LENGTH` | `20` characters |
| Gap between chapter segments in the seek bar | `--chapter-gap` in the `<style>` block | `4px` |
| NPO tag (analytics) properties | `npoTag.initprops` | placeholders (`your-brand`, `0`) |

## Troubleshooting

- **The player shows an error or doesn't load the stream:** check that `npo-player-key` is set for the origin you're on. Run `localStorage.getItem("npo-player-key")` in the console. If it returns `null`, the token is signed with the wrong key and the player rejects it.
- **Nothing happens at all:** check the console for errors loading the CDN scripts (`cdn.npoplayer.nl`, `esm.sh`).
