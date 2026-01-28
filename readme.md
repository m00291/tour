# 360° — A-Frame 360 Virtual Tour

An interactive 360° virtual tour built with A-Frame. It features smooth progressive image loading, tap/click hotspots for navigation, touch gestures (drag + pinch), mouse zoom, image prefetching and hotspot editing.

Demo: [https://m00291.github.io/tour/](https://m00291.github.io/tour/)

## Key Features

- Progressive panorama loading
  - Instant low-res preview (480px) that swaps to high-res (3040px) when ready.
  - Texture swap without reapplying attributes when possible, minimizing flicker.
  - Caches images to avoid redundant requests.
- Hotspot-based navigation
  - Clickable/tappable cones act as navigation hotspots between rooms.
  - Hotspots are auto-shown/hidden based on the current panorama.
  - A short lift animation gives visual feedback on tap.
  - Hotspot rig rotates to keep hotspot directions consistent with the panorama's phi-start.
- Smooth controls
  - Touch drag to look around (tuned sensitivity, clamped vertical rotation).
  - Two-finger pinch to zoom on mobile, with min/max bounds.
  - Mouse wheel zoom on desktop with gentle increments.
  - Magic window permission UI disabled for a clean UI.
- Prefetching for snappy transitions
  - When a view's hotspots become visible, their target images are prefetched in the background.
- Robust texture application
  - Tries to update the existing THREE.js texture map directly for speed.
  - Falls back to A-Frame's setAttribute + polling when necessary.
  - Preserves texture parameters (filters, encoding, mipmaps) on swap.
- In-scene hotspot editing (DevTools)
  - Unlock a hotspot and drag it directly in the scene to fine-tune its position.
  - Constrains motion along the camera ray with adjustable distance.
  - Temporarily disables other hotspots to avoid accidental navigation while editing.

## In-Scene Hotspot Editing

DevTools helper lets you reposition an existing hotspot directly in the scene, without guessing coordinates.

- Unlock one hotspot for editing:
  - In DevTools Console:
    ```js
    unlockHotspot('e_k')
    ```
- Drag the hotspot:
  - Mouse/touch drag across the canvas to move it along your current view ray.
  - While dragging, Press ArrowUp/ArrowDown to push/pull the hotspot (change distance).
- Finish editing:
  - Release pointer to show the final position to the console.
  - Copy `position` value back into the HTML.
  - Re-enable all hotspots and exit edit mode:
    ```js
    lockHotspot()
    ```

> [!NOTE]
> - Only one hotspot can be unlocked at a time.
> - Other hotspots are disabled while editing to prevent navigation.
> - Min/Max distance is clamped.

## How It Works

- Scene setup
  - `a-sky` displays the current equirectangular panorama.
  - `a-entity#hotspots` holds all cones used as hotspots.
  - Each hotspot has:
    - `data-visible-on`: which panorama it should appear in (e.g., `#e`).
    - `data-target`: the panorama to switch to (e.g., `#k`).
    - `target-phi-start`: the starting yaw alignment for the target panorama.
- Progressive loading flow
  - `progressiveSkyLoad(imgId, phiStart)`:
    1. Tries to apply cached low-res quickly.
    2. Loads high-res and swaps it in when ready.
    3. Sets `phi-start` only once on the first successful apply.
    4. Calls `updateHotspots` to show the correct hotspots and rotate the hotspot rig.
- Hit testing
  - Uses a `THREE.Raycaster` against hotspot `object3D`s for precise click/tap detection.
  - Resolves the corresponding A-Frame element via `hit.object.el`.
- Touch/mouse input
  - Custom `onTouchMove` increases look sensitivity and clamps vertical rotation.
  - Pinch zoom via `touchstart`/`touchmove`/`touchend`.
  - Mouse wheel zoom handled globally with bounds.

## File Structure

- `index.html` — main A-Frame scene and scripts
- `room/`
  - `{id}-480.jpg` — low-res preview panoramas
  - `{id}-3040.jpg` — high-res panoramas
- `ico.jpg` — favicon

Example IDs used: `e` (Entrance), `k` (Kitchen), `l` (Living), `c` (Center), `d` (Dining), `cor` (Corridor), `r1` (Room1), `m` (MasterRoom), `mt` (Master Room Toilet), `mtc1`/`mtc2` (Toilet Ceiling 1/2), `r2` (Room2), `lk` (Leak), `t` (Toilet)

## How to Add New Hotspot

1. Add the target panoramas in `room/` as both `-480` and `-3040` jpgs, e.g., `room/x-480.jpg` and `room/x-3040.jpg`.  
2. Create a hotspot cone under `#hotspots`:
   - `id`: use `from_to` format
   - `class`: `hotspot` (required).
   - `position`: approximate direction in the current panorama.
   - `data-visible-on`: the current panorama id (e.g., `#e`).
   - `data-target`: the target panorama id (e.g., `#x`).
   - `target-phi-start`: yaw alignment for the target panorama (degrees).

Example:
```html
<a-cone id="e_x" class="hotspot" radius-top="0.4" radius-bottom="0" transparent="true" color="#fff" opacity="0.3"
  position="1.5 -1.5 3"
  data-target="#x"
  target-phi-start="45"
  data-visible-on="#e">
</a-cone>
```

> [!TIP]
> Quick check for hotspot `target-phi-start`
>  1. In DevTools Console, lower the zoom floor:
>     ```js
>     minZoom = 0.1
>     ```
>  2. Drag the scene to face straight down (absolute downward).
>  3. Zoom out and Rotate until you're aligned to a neat square angle as your reference.
>  4. Go to the scene and click the hotspot.
>  5. In Console, adjust sky yaw:
>     ```js
>     sky.setAttribute('phi-start', 'DEGREES')
>     ```
>  6. Copy the final `phi-start` value into the hotspot.

> [!TIP]
> Use radius-top/radius-bottom to indicate arrow direction:
> - `radius-bottom="0"` (point-down cone) lifts up on click
> - `radius-top="0"` (point-up cone) lifts down on click.

## Development

- Requirements
  - Static hosting is enough (GitHub Pages works).
  - A-Frame 1.7.1 is loaded via CDN.
- Run locally (Visual Studio Code "Go Live")
  - Install "Live Server" extension.
  - Open project folder in VS Code and click "Go Live" button in status bar.

## Configuration

- Zoom limits
  - `minZoom`: 0.5, `maxZoom`: 2 (adjust in wheel and pinch handlers).
- Initial view
  - Initial panorama: `progressiveSkyLoad("#e", "0");`
  - Adjust initial sky id and phi as needed.
- Performance tips
  - Keep low-res around 480px height for instant swaps.
  - Ensure high-res are optimized JPGs to balance detail and load time.
  - mid-res for a three-step transition is not developed.

## Troubleshooting

- Blank sky or slow swap
  - Check image paths and file names match id scheme (`e`, `k`, `l`, …).
  - Verify images are accessible over HTTP(S) and not blocked by CORS.
  - When testing locally, use VS Code Live Server "Go Live" instead of opening the file directly (`file://`).

- Hotspots not clickable
  - Ensure `class="hotspot"`.
  - Confirm `data-visible-on` matches the current panorama id (e.g., `#e`).
  - Check that the hotspot is not moved to position `999` (hidden state).

- Misaligned hotspots after switching
  - Verify `target-phi-start` for that hotspot's target panorama.
  - Hotspot cone rotates by `rotation="x y z"`.

- Texture not updating
  - The code falls back to `setAttribute` when direct THREE texture update fails.
  - Open console to see warnings/errors for clues.

- DevTools helper not working
  - Ensure the scene has loaded (wait for A-Frame loaded event).
  - Check that #mainCamera exists and has a camera/object3D.
  - Make sure you unlocked an existing hotspot id (use DOM inspector to confirm).