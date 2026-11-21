# Napkin CAD

A pocket CAD app for your phone or tablet. Sketch a part with your finger, write the dimensions by hand, and turn it into a 3D model you can export for 3D printing.

Everything runs in the browser, works offline once loaded, and can be installed to your home screen.

## What it does

**Draw freely, the app cleans it up**
- Draw with your finger. Wobbly strokes become straight lines, rectangles, polygons and circles, snapped to 0°/45°/90°. End where you started to close an outline; shapes drawn inside an outline become holes.
- Separate strokes join at their ends. Line / Rect / Circle tools are there for precise, grid-snapped drawing.
- **Photo** puts a picture of a paper sketch under the grid so you can trace it.

**Dimensions, the intuitive way**
- **Write a number** right beside an edge (or next to a hole) while drawing. The app tells numbers from shapes by size and position, reads them, and applies them.
- **Tap** any edge, circle or dimension to open a number sheet with a **handwriting pad** (or the keyboard).
- The first dimension scales the whole freehand sketch to real millimetres; later ones reshape it while keeping its angles.
- In Dimension mode, **drag from a hole to an edge** to set its distance.
- **Angles between lines**: tap a corner, write the angle with a degree sign inside it (e.g. `60°`), or in Dimension mode drag from one line to the line it meets. Angles are drawn as arcs; other corners and sizes stay put where they can.
- Misread number? Tap **Fix** (the app learns your handwriting) or **Not a number** to turn the ink back into lines.

**Fully-defined check, then confirm**
- A status chip shows **Not to scale / Needs N dims / Fully defined / Over-defined**. Edges that still need a size are amber; sizes worked out from other dimensions show in grey brackets; duplicates are red.
- **Review** shows a live 3D preview, what is still missing, and **Add missing dimensions**, which walks you through each one. Nothing becomes 3D until you **Confirm**.

**3D and sketching on faces**
- Extrude (holes cut through) or Revolve (half-profile around its left edge).
- Tap a part, then tap one of its **flat faces** and choose **Sketch on this face**. The face outline appears as a reference; draw, dimension (positions are measured from the face edges), review, then **Add material**, **Cut** or **Cut through**.
- Boxes, cylinders, spheres, cones; drag to move; 90° rotations; **Cut away / Join / Keep overlap** between parts.
- Export **STL** (Z-up, millimetres) or **OBJ**; save and open projects as `.json`.

**Gestures**
- **Scribble** back and forth over a shape or a dimension to delete it.
- **Two-finger tap** = undo, **three-finger tap** = redo. Pinch to zoom, two fingers to pan. With a stylus, the pen draws and your finger pans.

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
- **Corner angles**: when a shape has corner angles, a small Newton solver moves the vertices as little as possible so every length, kept direction (horizontal/vertical sides) and corner angle holds at once. A corner angle between two square sides lets one of them turn (the non-horizontal one, else the shorter).
- **Fully defined**: each outline's side lengths and free side directions are unknowns; the closure equation Σ L·d = 0, every length and every corner angle are rows. The rank gives the missing count, a projection test tells which sides are pinned (derived), and a size or angle that adds no rank, in the order you added them, is flagged as set twice. Holes and face features need two distances to non-parallel reference edges.
- **Sketch on face**: the tapped triangle's coplanar neighbours form the face; their boundary edges become the reference outline in a 2D frame on that plane. The new feature is extruded along the face normal and combined with the part by CSG.
- **Handwriting**: a $P point-cloud recognizer with built-in digit templates. Strokes are grouped into characters by horizontal overlap; a small low dot is a decimal point. Corrections are stored as personal templates in the browser.
