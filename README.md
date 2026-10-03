# cute-qr-code

A small Vue app for turning text into friendly, ready-to-use QR code exports.

Live site: [cute-qr-code.surge.sh](https://cute-qr-code.surge.sh/)

Every visit starts with a randomly generated, family-friendly `.club` URL. Replace it with any text or URL and the QR code updates immediately on every keypress.

## Outputs

- Fixed-width QR text that can be selected or copied with one click
- Black QR PNG on a transparent background
- Black QR PNG on white
- White QR PNG on a transparent background, with a dark-preview toggle

Each PNG can be downloaded directly from its tile.

## Run locally

```bash
npm install
npm run dev
```

Open the local URL printed by Vite, usually `http://localhost:5173`.

## Production build

```bash
npm run build
```

The compiled site is written to `dist/`.

## Stack

- Vue 3
- Vite
- [`qrcode`](https://www.npmjs.com/package/qrcode) for QR matrix and PNG generation
