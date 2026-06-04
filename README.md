# Blob → Image

A tiny single-file utility that turns raw image bytes — or a base64 / hex / `data:image/…` dump of them — back into a viewable, downloadable image. Useful when you've captured an image payload off the wire, pulled one out of a database BLOB column, or copied a base64 string from a log and you want to see what it actually is.

Runs **entirely in your browser**. No server, no upload, no telemetry. Open the HTML, drop a file, see the image.

## What it does

- **Magic-byte sniff** for PNG, JPEG, GIF, WebP, BMP, TIFF, ICO, HEIC, AVIF, SVG (and a few non-image formats so the output makes sense).
- **Auto-decode** base64, hex, and `data:image/…;base64,…` URLs — paste any of those forms into the textarea and it'll find the image inside.
- **Encryption check** — flags well-known encrypted containers (OpenSSL `enc`, AES Crypt, AxCrypt, age, PGP, encrypted ZIP/RAR/7z/PDF, KeePass KDBX, BitLocker volumes) and measures Shannon entropy + byte uniformity for the unknowns.
- **Inline preview** with image dimensions, megapixels, aspect ratio.
- **Multi-output panels** — Image, readable Text, and structured Other (JSON, XML/HTML, PEM, CSV/TSV, URL lists) when the bytes contain any of those.
- **Downloads** — original bytes with the correct extension, or re-encoded as PNG / JPEG.
- **SHA-256 hash** of the decoded bytes so you can compare two blobs without eyeballing them.

## Try it locally

Just open `index.html` in any modern browser. That's it. No build step.

```sh
open index.html       # macOS
xdg-open index.html   # Linux
start index.html      # Windows
```

Or serve it locally if you want it on a phone in the same network:

```sh
python3 -m http.server 8000
# then visit http://<your-ip>:8000/
```

## Host it on GitHub Pages (recommended)

This is the easiest way to share a link with people. Three steps.

1. Create a new public repo on GitHub (e.g. `blob-to-image`).
2. Put `index.html` (and this README) in the repo root, commit, push.
3. In the repo: **Settings → Pages → Source: Deploy from branch → Branch: `main` / root → Save**.

Within ~1 minute your page is live at:

```
https://<your-github-username>.github.io/blob-to-image/
```

Shareable, free, no infrastructure.

### One-time setup, from scratch

```sh
cd blob-to-image
git init
git add index.html README.md
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<your-username>/blob-to-image.git
git push -u origin main
```

Then enable Pages in the repo settings as above.

## Host with Docker (optional)

GitHub Pages is simpler for a static page like this — Docker is overkill. But if you'd rather self-host (intranet, air-gapped, behind a VPN), you can do it with three lines using nginx:

**`Dockerfile`**

```dockerfile
FROM nginx:1.27-alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

**Build & run**

```sh
docker build -t blob-to-image .
docker run --rm -p 8080:80 blob-to-image
# visit http://localhost:8080
```

**Share the image**

```sh
docker save blob-to-image:latest | gzip > blob-to-image.tar.gz
# send the file; the recipient runs:
gunzip -c blob-to-image.tar.gz | docker load
docker run --rm -p 8080:80 blob-to-image
```

Or push to a registry:

```sh
docker tag blob-to-image <your-dockerhub-user>/blob-to-image:latest
docker push <your-dockerhub-user>/blob-to-image:latest
```

## Privacy

Everything runs locally in the browser. The page makes **zero network requests** once loaded — no analytics, no font CDNs, no JavaScript libraries. Drop the most sensitive blob you have; the bytes never leave your machine.

## Browser support

Anything from the last few years. The page uses standard Web APIs (`FileReader`, `Blob`, `URL.createObjectURL`, `crypto.subtle.digest`, `<canvas>.toBlob`). HEIC decoding is browser-dependent — Safari and recent Chrome on macOS handle it, others can still download the original `.heic` bytes but the preview will show "Browser couldn't decode this format".

## License

Public domain / Unlicense — do whatever you want with it.
