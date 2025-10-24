# 3D Sunglasses Viewer for Netlify

A fast, mobile-optimized 3D model viewer using Google's `<model-viewer>` with HDRI support for realistic glass reflections and refractions.

## 📁 File Structure

```
web3d/
├── index.html          ← Main viewer page
├── _headers            ← Netlify config (MIME types, CORS, caching)
├── assets/
│   ├── model.glb       ← Your sunglasses model (you provide)
│   ├── studio_1k.hdr   ← Environment map for reflections (you provide)
│   └── poster.webp     ← Loading poster image (optional, you provide)
└── README.md           ← This file
```

---

## 🚀 Quick Start (Deploy in 5 Minutes)

### Step 1: Prepare Your Assets

You need three files in the `assets/` folder:

#### 1.1 Convert your HDRI (EXR → HDR)

Your 500 MB EXR will destroy mobile performance. Convert it to a lightweight 1K–2K `.hdr`:

**Option A: Using Blender**
1. Open Blender
2. Shading tab → World → Add your EXR as Environment Texture
3. UV/Image Editor → Open your EXR
4. Image menu → Resize → 1024 × 512 (or 2048 × 1024 for high-quality)
5. Image → Save As → Select "Radiance HDR" format
6. Save as `studio_1k.hdr`

**Option B: Using OpenImageIO (CLI)**
```bash
# Install (macOS)
brew install openimageio

# Convert & resize
oiiotool your-hdri.exr --resize 1024x512 -o studio_1k.hdr
```

**Result:** A 1–2 MB `.hdr` file that provides realistic reflections without destroying bandwidth.

#### 1.2 Place your model

Copy your current 10 MB GLB into `assets/`:
```
assets/model.glb
```

*(We'll optimize this to ~1–3 MB later, but it works fine as-is for testing)*

#### 1.3 Create a poster image (optional but recommended)

Render a nice still frame of your sunglasses in Blender:
- Resolution: 1920 × 1080 or similar
- Export as PNG/JPG
- Convert to WebP (smaller):

```bash
# Using cwebp (install via brew/apt)
cwebp -q 80 poster.png -o poster.webp
```

Or use an online converter: https://squoosh.app

**Result:** ~100–200 KB poster that loads instantly before the 3D.

---

### Step 2: Deploy to Netlify

**Option A: Drag-and-Drop (easiest)**
1. Go to [Netlify](https://app.netlify.com/)
2. Sites → "Add new site" → "Deploy manually"
3. Drag the entire `web3d/` folder into the drop zone
4. Wait ~10 seconds
5. Open your new site URL (e.g., `https://random-name-123.netlify.app`)

**Option B: Netlify CLI**
```bash
# Install CLI
npm install -g netlify-cli

# Login
netlify login

# Deploy
cd web3d
netlify deploy --prod
# When prompted, select the web3d folder as the publish directory
```

---

### Step 3: Test on Mobile & Desktop

1. Open your Netlify URL on desktop → You should see the poster
2. **Tap/click** the poster → 3D loads with HDRI reflections
3. Open on your phone → Should load instantly (poster first), then 3D after tap
4. Check Developer Tools → Console for any errors

**Common issues:**
- **404 on model.glb**: Check file name matches exactly `model.glb` in `assets/`
- **CORS error**: Make sure `_headers` file is at the root (next to `index.html`)
- **Black/flat materials**: HDRI might not be loading; check `studio_1k.hdr` path
- **Slow load**: See optimization section below

---

## 🎨 Blender Material Setup for Sunglasses

For realistic glass lenses with HDRI reflections, use these settings in Blender **before** exporting:

### Lens Material (Principled BSDF)

```
Transmission: 1.0               ← Makes it glass, not plastic
IOR: 1.5                        ← Realistic refraction (glass)
Roughness: 0.0–0.1              ← 0 = mirror-smooth, 0.1 = slightly frosted
Alpha: 1.0                      ← DO NOT use alpha-blend for glass!
Metallic: 0.0
```

### Lens Tint (Colored Lenses)

Enable **KHR_materials_volume** (glTF extension):
1. Shader Editor → Add "Volume Absorption" → connect to Volume output
2. Or in Principled BSDF (if using glTF exporter):
   - **Thickness**: 2.0 mm (actual lens thickness in your scene units)
   - **Attenuation Color**: Your tint (e.g., brown, green, dark gray)
   - **Attenuation Distance**: Controls how strong the tint is

### Optional: AR Coating Sheen

For that subtle rainbow reflection on anti-reflective coatings:
- Enable **KHR_materials_iridescence** (glTF extension)
- Iridescence: 0.3–0.5
- Iridescence IOR: 1.3
- Iridescence Thickness: 400 nm

### Frame Material

Typical plastic/acetate:
```
Transmission: 0.0
Roughness: 0.2–0.4              ← Slight sheen, not mirror
Metallic: 0.0
Base Color: Your frame color
```

Metal frames:
```
Metallic: 1.0
Roughness: 0.1–0.3
Base Color: White/light gray (tint via roughness/env reflections)
```

### Export Settings (Blender → GLB)

1. Select all objects
2. **File → Export → glTF 2.0 (.glb)**
3. Settings:
   - **Format**: glTF Binary (.glb)
   - **Include**: Selected Objects (or Visible Objects)
   - **Transform**: +Y Up
   - **Materials**: Export
   - **Images**: Automatic (or "Embed" if textures aren't packing)
   - **Compression**: None (we'll compress later with gltf-transform)
   - **Lighting**: Disable "Punctual Lights" unless you need them
4. Export

**Before export checklist:**
- [ ] Apply all transforms: Ctrl+A → All Transforms
- [ ] All meshes have UVs
- [ ] Normal maps are in a "Normal Map" node (Tangent space, Non-Color data)
- [ ] All texture images set to correct color space (sRGB for Base Color, Non-Color for everything else)
- [ ] No exotic shader nodes (only Principled BSDF)

---

## ⚡ Optimize Your Model (Shrink 10 MB → 1–3 MB)

Once the basic viewer works, compress your GLB for faster mobile loading:

### Install glTF Transform CLI
```bash
npm install -g @gltf-transform/cli
```

### Compress the Model
```bash
cd assets

# Full optimization: Draco geometry + KTX2 textures
gltf-transform optimize model.glb model-opt.glb \
  --draco \
  --meshopt \
  --ktx2 \
  --texture-compress etc1s \
  --slots "normalTexture:uastc" \
  --uastc 2 \
  --etc1s-quality 255 \
  --etc1s-compression 9
```

**What this does:**
- `--draco`: Compresses geometry (vertices, normals, UVs) → ~80% smaller
- `--meshopt`: Further optimizes mesh data for GPU
- `--ktx2`: Compresses textures to GPU-native format
- `--slots normalTexture:uastc`: Keeps normal maps high-quality (UASTC)
- `etc1s` for color/roughness/metallic: good compression, slight quality trade-off

**Result:** Usually 70–90% size reduction (10 MB → 1–3 MB).

### Update HTML to Use Optimized Model
In `index.html`, change:
```html
src="/assets/model.glb"
```
to:
```html
src="/assets/model-opt.glb"
```

Redeploy to Netlify.

---

## 🔧 Advanced Customization

### Adjust Camera Position
Add these attributes to `<model-viewer>`:
```html
camera-orbit="45deg 75deg 2m"
min-camera-orbit="auto auto 0.5m"
max-camera-orbit="auto auto 10m"
```

### Change Exposure/Tone
```html
exposure="1.2"          <!-- Brighter (default 1) -->
tone-mapping="commerce" <!-- Options: commerce, neutral, aces -->
```

### Disable Skybox (Solid Background)
Remove:
```html
skybox-image="/assets/studio_1k.hdr"
```
Keep only `environment-image` for reflections.

### Auto-Rotate
```html
auto-rotate
auto-rotate-delay="0"
rotation-per-second="30deg"
```

### AR (Augmented Reality)
Already enabled in the template:
```html
ar
ar-modes="webxr scene-viewer quick-look"
```
Works on iOS (Quick Look) and Android (Scene Viewer).

---

## 📊 Performance Targets

| Asset | Target Size | Notes |
|-------|-------------|-------|
| **GLB** | 0.5–3 MB | 10 MB works but optimize later |
| **HDR** | 1–2 MB | 1K (1024×512) is sweet spot for product |
| **Poster** | 100–300 KB | WebP format, ~1920×1080 |
| **Total (first tap)** | ~200 KB | Just the poster + HTML |
| **Total (after tap)** | 2–5 MB | Model + HDR load on-demand |

With `reveal="interaction"`, the page loads instantly and only fetches the 3D when the user taps.

---

## 🐛 Troubleshooting

### Model doesn't load (404)
- Check file name: must be exactly `model.glb` in `assets/`
- Check path in HTML: `src="/assets/model.glb"`
- Redeploy to Netlify

### Materials look flat/wrong
- HDRI not loading → Check `studio_1k.hdr` exists in `assets/`
- Check console for CORS errors → Make sure `_headers` is deployed
- Re-export from Blender with embedded textures

### CORS errors
- Make sure `_headers` file is at the root (same level as `index.html`)
- Redeploy to Netlify
- Open in private/incognito window to clear cache

### Textures are black/missing
- Re-export GLB with "Images: Automatic" or "Embed"
- In Blender: File → External Data → Pack Resources
- Make sure texture image color space is correct (sRGB for color, Non-Color for data)

### Glass looks weird/opaque
- Check Transmission = 1.0 in Blender
- Make sure IOR is set (~1.5)
- Don't use Alpha < 1.0 for glass
- Make sure HDRI is loading (glass needs reflections to look real)

### Slow on mobile
- Optimize GLB with `gltf-transform` (see above)
- Use 1K HDR, not 2K or EXR
- Keep poster small (WebP, ~200 KB)

---

## 📚 Additional Resources

- [Model-Viewer Docs](https://modelviewer.dev/)
- [glTF Spec (KHR Extensions)](https://github.com/KhronosGroup/glTF/tree/main/extensions)
- [glTF Transform CLI](https://gltf-transform.dev/)
- [Blender glTF Export Tips](https://docs.blender.org/manual/en/latest/addons/import_export/scene_gltf2.html)
- [Free HDRIs](https://polyhaven.com/hdris) (download 1K–2K .hdr)

---

## ✅ Deployment Checklist

- [ ] Convert EXR to 1K `.hdr` (1–2 MB)
- [ ] Place `model.glb` in `assets/`
- [ ] Place `studio_1k.hdr` in `assets/`
- [ ] (Optional) Place `poster.webp` in `assets/`
- [ ] Ensure `_headers` file is at root
- [ ] Deploy `web3d/` folder to Netlify
- [ ] Test on desktop (tap to load)
- [ ] Test on mobile (should be fast, tap to load 3D)
- [ ] Optimize GLB with `gltf-transform` if needed
- [ ] Enjoy realistic sunglasses with HDRI reflections!

---

## 💡 Next Steps

1. **Get it working first** → Deploy with your 10 MB GLB + 1K HDR
2. **Optimize** → Compress to ~1–3 MB with `gltf-transform`
3. **Polish** → Adjust exposure, camera, add annotations
4. **Embed** → Use an `<iframe>` to embed this page in your Framer site

Example iframe for Framer:
```html
<iframe
  src="https://your-site.netlify.app"
  width="100%"
  height="600px"
  frameborder="0"
  allow="xr-spatial-tracking"
  style="border:none;">
</iframe>
```

---

**Questions?** Check the console (F12) for error messages, or share your Netlify URL and I can spot-check the setup.
