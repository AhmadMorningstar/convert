# [Convert website](https://ahmadmorningstar.github.io/convert/)
**Truly universal online file converter.**

This is my personal fork of [p2r3/convert](https://github.com/p2r3/convert), a fantastic open-source project by [p2r3](https://github.com/p2r3). All credit for the original design, architecture, and the vast majority of the format handlers goes to them and their contributors. I'm running this fork for my own use, hosted at [ahmadmorningstar.github.io/convert](https://ahmadmorningstar.github.io/convert/).

> This fork is not affiliated with, endorsed by, or intended to contribute back to the upstream project. If you're looking for the original, actively maintained version, go to [convert.to.it](https://convert.to.it/) or [github.com/p2r3/convert](https://github.com/p2r3/convert).

Many online file conversion tools are **boring** and **insecure**. They only allow conversion between two formats in the same medium (images to images, videos to videos, etc.), and they require that you _upload your files to some server_.

This is not just terrible for privacy, it's also incredibly lame. What if you _really_ need to convert an AVI video to a PDF document? Try to find an online tool for that, I dare you.

This tool aims to "just work". You're almost _guaranteed_ to get an output — perhaps not always the one you expected, but it'll try its best to not leave you hanging.

For a semi-technical overview of the original tool this is forked from, check out p2r3's video: https://youtu.be/btUbcsTbVA8

## Usage

1. Go to [ahmadmorningstar.github.io/convert](https://ahmadmorningstar.github.io/convert/)
2. Click the big blue box to add your file (or just drag it on to the window).
3. An input format should have been automatically selected. If it wasn't, try searching for it.
4. Select an output format from the second list. If you're on desktop, that's the one on the right side. If you're on mobile, it'll be somewhere lower down.
5. Click **Convert**!
6. Hopefully, after a bit (or a lot) of thinking, the program will spit out the file you wanted.

## License

This project is licensed under **GPL-2.0**, same as upstream. See [LICENSE](LICENSE) for the full text.

In short: you're free to use, modify, and host this fork, but any public copy has to stay open-source under GPL-2.0 with the original copyright and license intact.

## Deployment

### Local development (Bun + Vite)

1. Clone this repository ***WITH SUBMODULES***: `git clone --recursive https://github.com/AhmadMorningstar/convert`. Omitting submodules will leave you missing a few dependencies.
2. Install [Bun](https://bun.sh/).
3. Run `bun install` to install dependencies.
4. Run `bunx vite` to start the development server.

_The following steps are optional, but recommended for performance:_

When you first open the page, it'll take a while to generate the list of supported formats for each tool. If you open the console, you'll see it complaining a bunch about missing caches.

After this is done (indicated by a `Built initial format list` message in the console), use `printSupportedFormatCache()` to get a JSON string with the cache data. You can then save this string to `cache.json` to skip that loading screen on startup.

If you run into issues where your changes seem to not be applying, try disabling this cache.

### Production build

```bash
bun run build       # builds the site into dist/
bun run cache:build  # optional: pre-generates dist/cache.json (needs Chrome/Puppeteer)
bun run preview      # serve the built dist/ locally to sanity-check before pushing
```

### GitHub Pages (automatic)

Every push to `master` triggers `.github/workflows/pages.yml`, which builds, caches, tests, and deploys automatically to [ahmadmorningstar.github.io/convert](https://ahmadmorningstar.github.io/convert/). No manual deploy steps needed.

### Docker (prebuilt image)

Docker compose files live in the `docker/` directory, so run compose with `-f` from the repository root:

```bash
docker compose -f docker/docker-compose.yml up -d
```

This runs the container on `http://localhost:8080/convert/`.

### Docker (local build for development)

```bash
docker compose -f docker/docker-compose.yml -f docker/docker-compose.override.yml up --build -d
```

The first Docker build is expected to be slow because Chromium and related system packages are installed in the build stage (needed for puppeteer in `buildCache.js`). Later builds are usually much faster due to Docker layer caching.

## Adding or changing handlers (personal notes)

Each conversion "tool" is wrapped in a handler under [src/handlers](src/handlers/). A barebones starting point:

```ts
// file: dummy.ts

import type { FileData, FileFormat, FormatHandler } from "../FormatHandler.ts";
import CommonFormats, { Category } from "src/CommonFormats.ts";

class dummyHandler implements FormatHandler {

  public name: string = "dummy";
  public supportedFormats: FileFormat[] = [
    CommonFormats.PNG.builder("png")
      .markLossless()
      .allowFrom(false)
      .allowTo(false),

    {
      name: "CompuServe Graphics Interchange Format (GIF)",
      format: "gif",
      extension: "gif",
      mime: "image/gif",
      from: false,
      to: false,
      internal: "gif",
      category: [Category.IMAGE, Category.VIDEO],
      lossless: false
    },
  ];
  public ready: boolean = false;

  async init () {
    this.ready = true;
  }

  async doConvert (
    inputFiles: FileData[],
    inputFormat: FileFormat,
    outputFormat: FileFormat
  ): Promise<FileData[]> {
    const outputFiles: FileData[] = [];
    return outputFiles;
  }

}

export default dummyHandler;
```

Notes for myself:

- Naming: a tool called `dummy` → class `dummyHandler` → file `dummy.ts`.
- The handler sets the output file's name (usually just swapping the extension).
- Never mutate byte buffers that enter/exit a handler — clone with `new Uint8Array()` if needed.
- Run MIME types through [normalizeMimeType](src/normalizeMimeType.ts) first.
- Treat files as the media they represent, not their raw underlying data (e.g. SVG is an *image*, not just XML).
- Avoid CDNs — install via `npm`/Bun, or add a git submodule under `src/handlers`, or as a last resort a local folder with the needed assets.
- WebAssembly binaries get registered in [vite.config.js](vite.config.js) and served under `/convert/wasm/`. Don't link to `node_modules` directly.

### Testing

- Broad project-level tests live in `test/` (graph traversal, end-to-end conversion smoke tests).
- Handler-specific unit tests live in `test/handlers/`, named `<handlerName>.test.ts`.

## Credit

Original project and the vast majority of the code: [p2r3/convert](https://github.com/p2r3/convert) by [p2r3](https://github.com/p2r3) and contributors, licensed under GPL-2.0.
