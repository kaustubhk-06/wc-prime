# Soft Pearl

The approved wc-prime identity is option 01 / A — Soft Pearl from the dark logo exploration.

- `wordmark.svg`: scalable wordmark with a transparent background; place on charcoal.
- `icon.svg` and `../icon.svg`: square project icon with the approved dark background.
- `icon-192.png` and `icon-512.png`: app icons.
- `apple-touch-icon.png`: 180 px touch icon.
- `favicon.ico`: 16, 32, and 48 px favicon.
- `banner.svg` and `social-preview.png`: 1280 × 640 dark brand banner.
- `site.webmanifest`: app icon metadata for a future website.

The source SVGs preserve the approved geometry and opal gradient. Background: `#0d0e14`. Preserve the lowercase wc lettering, proportions, and clear space. Do not substitute a font or add cloud illustrations.

There is no website implementation yet. When adding one, serve these assets and include:

```html
<link rel="icon" type="image/svg+xml" href="/assets/icon.svg">
<link rel="icon" href="/assets/brand/favicon.ico" sizes="any">
<link rel="apple-touch-icon" href="/assets/brand/apple-touch-icon.png">
<link rel="manifest" href="/assets/brand/site.webmanifest">
<meta name="theme-color" content="#0d0e14">
```

For GitHub's social preview setting, upload `social-preview.png` under repository Settings → General → Social preview. Committing the image alone does not change that setting.
