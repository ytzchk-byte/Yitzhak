# Napkin CAD

A pocket CAD app for your phone or tablet. Sketch a part with your finger, write the dimensions by hand, and turn it into a 3D model you can export for 3D printing.

Everything runs in the browser, works offline once loaded, and can be installed to your home screen.

## What it does

**Sketch**
- **Draw** freehand. Wobbly strokes are cleaned up into straight lines, rectangles, polygons and circles. Lines snap to 0°/45°/90°, and an outline closes when you finish where you started. Separate strokes join at their ends.
- **Dimension** by handwriting: write `80` next to an edge (or inside/next to a circle for its diameter). The number is read automatically and applied.
  - The first dimension scales the whole freehand sketch to real millimetres.
  - Later dimensions reshape the outline while keeping its angles (a small constraint solver keeps the loop closed).
  - If a number is misread, tap **Fix** and type it. The app saves your handwriting and reads it better next time.
  - You can also just tap an edge or circle and type the value.
- **Select** to drag points and shapes, or type exact positions. Holes show their position measured from the outline's left and bottom edges.
- **Line / Rect / Circle** for precise, grid-snapped drawing.
- **Photo** puts a picture of a paper sketch under the grid so you can trace it.
- Undo / redo, pinch to zoom, two-finger pan. With a stylus, the pen draws and your finger pans.

**3D**
- **Make 3D → Extrude** turns closed outlines into solids; shapes inside an outline become holes.
- **Make 3D → Revolve** spins a half-profile around its left edge (cups, knobs, bottles).
- Add **boxes, cylinders, spheres, cones**; edit sizes, position, 90° rotations, colour.
- Drag a selected part to move it. **Cut away / Join / Keep overlap** combine two parts (boolean CSG).
- Export **STL** (Z-up, millimetres, ready for slicers) or **OBJ**. Save and open projects as `.json`.

## Use it

Open `index.html` through any static web server, for example:

```sh
python3 -m http.server 8000
# then open http://localhost:8000 on your phone (same Wi-Fi: http://<your-computer-ip>:8000)
```

### Offline on your phone (GitHub Pages)

1. GitHub Pages needs a **public** repository on a free account (private repositories need GitHub Pro or a paid plan).
2. Repository **Settings → Pages → Build and deployment**: Source **Deploy from a branch**, pick the branch with the app and the **/ (root)** folder, then **Save**.
3. After a minute the site is live at `https://<user>.github.io/<repo>/`.
4. Open that link on your phone once while online:
   - **iPhone (Safari):** Share → **Add to Home Screen**.
   - **Android (Chrome):** ⋮ menu → **Install app** / **Add to Home screen**.
5. Launch it from the home-screen icon. It now works with no connection; your sketches and parts are saved on the phone.

When the app is updated, opening it once while online fetches the new version.

## Files

| Path | What it is |
| --- | --- |
| `index.html` | The whole app: UI, sketch engine, handwriting reader, 3D workspace, exporters |
| `sw.js`, `manifest.webmanifest`, `icon*.png`, `icon.svg` | Offline support and home-screen install |
| `vendor/` | three.js r160, OrbitControls, BufferGeometryUtils, three-mesh-bvh, three-bvh-csg (MIT licences included) |

## How the pieces work

- **Shape recognition**: the stroke is resampled, simplified (Ramer–Douglas–Peucker), tested for a circle by radius variance, then edge directions are snapped and the loop is re-closed with a least-squares length adjustment.
- **Dimensions**: edge directions stay fixed; dimensioned edge lengths are pinned and the remaining edges absorb the change with the smallest weighted adjustment that keeps the outline closed.
- **Handwriting**: a $P point-cloud recognizer with built-in digit templates. Strokes are grouped into characters by horizontal overlap; a small low dot is a decimal point. Corrections are stored as personal templates in the browser.
