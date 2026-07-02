# Jcloset Setup Guide

Jcloset is a fully client-side digital wardrobe that runs entirely in your browser — no server needed. All image processing (background removal, classification, color extraction) happens locally via WebML models.

---

## Prerequisites

- **Node.js 18+** with `npm`

---

## 1. Quick Start (Local Dev)

```bash
cd frontend
npm install
npm run dev
```

Opens at `http://localhost:5173`.

---

## 2. Deploy to GitHub Pages

### One-time setup

1. **Create a GitHub repository** named `Jcloset` (or your preference).
2. Push the project:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/<your-username>/Jcloset.git
   git push -u origin main
   ```
3. **Enable GitHub Pages**:
   - Go to repo → **Settings** → **Pages**
   - Under "Source", select **GitHub Actions**

### Automatic deployment

Every push to `main` triggers the workflow in `.github/workflows/deploy.yml` which:

1. Installs dependencies
2. Runs `npm run build` → outputs `frontend/dist/`
3. Uploads the artifact
4. Deploys to Pages

Your site will be live at: `https://<your-username>.github.io/Jcloset/`

### Manual deploy (from local)

```bash
cd frontend
npm run build

# Option A: push dist/ to gh-pages branch
npx gh-pages -d dist

# Option B: serve locally to preview
npm run preview
```

---

## 3. How It Works

Everything runs in your browser — zero servers, zero API keys.

| Step | Technology | Model Size |
|------|-----------|------------|
| Background removal | `@imgly/background-removal` (U²-Net via ONNX) | ~170 MB |
| Classification | `@huggingface/transformers` (CLIP ViT-B/32) | ~600 MB |
| Color extraction | Canvas pixel sampling + KMeans (pure JS) | — |
| Storage | IndexedDB via Dexie (no SQLite) | — |

**Important:** The first time you upload a clothing item, the browser downloads the background removal model (~170 MB) and the CLIP model (~600 MB). These are cached in the browser's IndexedDB / Cache API for future use. The first upload may take 30–60 seconds; subsequent uploads are much faster.

---

## 4. Project Structure

```
Jcloset/
├── frontend/
│   ├── src/
│   │   ├── App.tsx / App.css            # Main app shell + styles
│   │   ├── types.ts                     # TypeScript types
│   │   ├── services/
│   │   │   ├── backgroundRemover.ts     # @imgly/background-removal wrapper
│   │   │   ├── classifier.ts            # CLIP zero-shot classification
│   │   │   ├── colorExtractor.ts        # Canvas-based color analysis
│   │   │   └── storage.ts               # IndexedDB via Dexie
│   │   └── components/
│   │       ├── ClosetGrid.tsx           # Item grid
│   │       ├── ClothingCard.tsx         # Individual card
│   │       ├── UploadModal.tsx          # Upload with drag-drop + processing
│   │       └── ItemDetail.tsx           # Full item view
│   ├── index.html
│   ├── package.json
│   ├── vite.config.ts
│   └── tsconfig.json
├── .github/workflows/deploy.yml         # GitHub Actions deploy
├── package.json                         # Root convenience scripts
└── setup.md
```

---

## 5. Customization

### Change the base URL

If your repo has a different name, update the `base` in `frontend/vite.config.ts`:

```ts
export default defineConfig({
  base: '/YourRepoName/',
  plugins: [react()],
})
```

### Add items manually

Data is stored in your browser's IndexedDB. Clearing browser data will remove your closet. Export/import functionality can be added if needed.

---

## 6. Browser Support

Requires a modern browser with:
- WebGL support (for ONNX Runtime Web)
- IndexedDB
- ES2020

**Supported:** Chrome, Edge, Firefox, Safari 16+

---

## 7. Troubleshooting

| Problem | Likely Fix |
|---------|-----------|
| First upload is very slow | Models are downloading (~770 MB total). Progress is shown in the modal. |
| "Failed to load image for classification" | The image file may be corrupted. Try a different image. |
| Build fails with TypeScript errors | Run `npm install` first. If issues persist, check `@huggingface/transformers` compatibility. |
| Nothing appears after deploy | Wait 1–2 minutes for Pages to provision. Check **Actions** tab for build status. |
| Blank page after deploy | The `base` in `vite.config.ts` must match your repo name. See section 5. |
| Processing fails on mobile | Mobile browsers have limited memory for large ML models. Desktop recommended. |
