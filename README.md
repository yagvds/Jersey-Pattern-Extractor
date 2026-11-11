# Jersey Pattern Extractor — Phase 2

## What this phase implements

1. Upload a jersey mockup (PNG/JPG/WEBP, up to 4 MB).
2. Send the image from a Next.js server route to an Anthropic-compatible Messages API gateway.
3. Ask Claude Vision to identify visible fabric panels and return normalized polygon coordinates through a forced tool schema.
4. Preview the polygons over the source mockup and select front/back/sleeve panel.
5. Clip the selected polygon, fit its bounding box to a 600 × 900 viewBox (2:3 ratio), trace the raster pixels to SVG, and export SVG or 2400 × 3600 PNG.

## Setup

Node.js 20.9+.

```bash
cp .env.example .env.local
# Edit .env.local and set a rotated API key and a model ID supported by your gateway.
npm install
npm run dev
```

Open http://localhost:3000.

## Gateway configuration

The server calls `${ANTHROPIC_BASE_URL}/v1/messages` and sends `x-api-key` and `anthropic-version`. Configure:
- `ANTHROPIC_BASE_URL`
- `ANTHROPIC_API_KEY`
- `ANTHROPIC_VERSION`
- `ANTHROPIC_MODEL`

The model ID is deliberately configurable because gateway model aliases can differ. Never expose the API key through `NEXT_PUBLIC_*`.

## Important technical boundary

Claude returns **approximate visible panel polygons**, not pixel-perfect segmentation masks. This is a human-reviewable AI proposal. The current extraction clips to the polygon and stretches its bounding box to a 2:3 canvas. It does **not** yet perform true perspective/cloth dewarping, remove internal wrinkles/shadows, infer hidden fabric, or reconstruct manufacturing pattern pieces. A polygon crop may therefore distort angled/curved panels.

For production quality, next add:
1. Editable polygon vertices / brush mask refinement.
2. Four-corner or mesh-based perspective/cloth dewarping (OpenCV or a dedicated vision model).
3. Per-pixel segmentation mask rather than polygon-only detection.
4. Shadow/lighting normalization and texture reconstruction.
5. Separate output templates for front, back, and sleeves.
6. Physical export metadata: 600 mm × 900 mm page size and configurable DPI/bleed.

## Physical size

The SVG export is assigned physical dimensions of 600 mm × 900 mm with a 600 × 900 viewBox. PNG export is 2400 × 3600 pixels (nominally 101.6 DPI at 60 × 90 cm). Confirm the required print DPI, bleed, and color profile with the print provider.

## Privacy and security

The uploaded image is transmitted to the configured AI gateway for analysis. Use images you have rights to process. Add authentication, rate limiting, and retention controls before public deployment. Rotate any API key that has been exposed in chat or source code.


## Deploy as a public website (no local runtime for end users)

This project is prepared for Vercel deployment. End users only need the deployed URL; they do not install Node.js or run a local server.

### Deploy with GitHub + Vercel

1. Create a GitHub repository and upload the contents of this project.
2. Sign in to Vercel and choose **Add New → Project**.
3. Import the GitHub repository.
4. Keep the framework preset as **Next.js**.
5. In **Environment Variables**, add:
   - `ANTHROPIC_BASE_URL` = `https://claude.dramakuy.id/anthropic`
   - `ANTHROPIC_API_KEY` = your newly rotated secret key
   - `ANTHROPIC_VERSION` = `2023-06-01`
   - `ANTHROPIC_MODEL` = the exact model ID/alias enabled by your gateway
6. Click **Deploy**. Vercel builds and publishes the site and provides a public `*.vercel.app` URL.
7. Open the URL and test with a jersey mockup.

### Hosting note

The API route currently caps uploads at 4 MB to stay below the typical Vercel serverless request-body limit when using multipart form uploads. If production mockups are larger, add direct-to-object-storage uploads (for example Vercel Blob or Cloudflare R2) before sending the image to the AI route.

### Important

I cannot publish this into your Vercel account or create a live public URL without you connecting/authorizing that hosting account. The ZIP is deployment-ready; deployment requires the account steps above. Keep the API key in Vercel Environment Variables only, never in frontend code or GitHub.
