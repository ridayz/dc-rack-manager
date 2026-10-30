# AGENTS.md — Panduan Design Model 3D (Three.js)

> Sumber acuan: `https://github.com/mrdoob/three.js` — JavaScript 3D Library (MIT, 117k+ stars).
> Docs: `https://threejs.org/docs/` · Examples: `https://threejs.org/examples/` · Manual: `https://threejs.org/manual/`

File ini WAJIB dibaca setiap agen sebelum bikin/ubah tampilan 3D di repo ini.
Skill global **`threejs`** (`skills/threejs/SKILL.md`) WAJIB diload tiap sentuh 3D — panduan di bawah ini ringkasannya.

## 1. Cara pakai Three.js yang benar

- Install: `npm install three` — import sebagai module: `import * as THREE from 'three'`
- Jangan pakai CDN `<script>` global untuk project baru. Pakai ES module + bundler (Vite/Next).
- Addons (OrbitControls, GLTFLoader, dll) import dari `three/addons/`:
  `import { OrbitControls } from 'three/addons/controls/OrbitControls.js'`
- Versi pin di `package.json`. Cek Migration Guide tiap upgrade mayor:
  `https://github.com/mrdoob/three.js/wiki/Migration-Guide`

## 2. Struktur scene minimal (pola baku)

```js
import * as THREE from 'three';
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
renderer.setSize(innerWidth, innerHeight);
document.body.appendChild(renderer.domElement);

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x0b1020);

const camera = new THREE.PerspectiveCamera(60, innerWidth/innerHeight, 0.1, 1000);
camera.position.set(0, 2, 5);

const light = new THREE.DirectionalLight(0xffffff, 2);
light.position.set(3, 5, 4);
scene.add(light, new THREE.AmbientLight(0xffffff, 0.5));

const mesh = new THREE.Mesh(
  new THREE.BoxGeometry(1, 1, 1),
  new THREE.MeshStandardMaterial({ color: 0xa855f7, roughness: 0.4, metalness: 0.1 })
);
scene.add(mesh);

renderer.setAnimationLoop((t) => {
  mesh.rotation.y = t / 1000;
  renderer.render(scene, camera);
});
```

## 3. Aturan design 3D di project ini

1. **Renderer**: selalu `antialias: true`, batasi `pixelRatio max 2`, pakai `setAnimationLoop` (bukan `requestAnimationFrame` manual).
2. **Kamera**: default `PerspectiveCamera(fov 45-60, near 0.1, far 1000)`. Jangan taruh kamera di dalam mesh.
3. **Lighting wajib**: minimal 1 directional + 1 ambient/hemisphere. Material `MeshStandardMaterial` tanpa cahaya = hitam.
4. **Kontrol**: pakai `OrbitControls` dengan `enableDamping = true` + `controls.update()` di loop.
5. **Resize**: selalu handle `window resize` → update `camera.aspect` + `renderer.setSize`.
6. **Cleanup**: dispose `geometry`, `material`, `texture`, `renderer` saat component unmount (React/Next).
7. **Asset**: model `.glb/.gltf` via `GLTFLoader`, texture via `TextureLoader`. Taruh di `public/models/`, jangan inline base64 besar.
8. **Perf**: reuse geometry/material, gabung mesh statis, hindari bikin object baru di dalam loop animasi.
9. **A11y + state**: sediakan loading (progress loader), error (pesan + retry), empty state, dan fallback 2D kalau WebGL tidak tersedia.
10. **Gaya visual**: ikuti `archify` — dark glassmorphism, aksen gradient ungu→pink, canvas 3D sebagai fokus utama, UI overlay minimal.

## 4. Referensi cepat API

| Butuh | Pakai |
|---|---|
| Scene, fog, background | `THREE.Scene`, `scene.fog = new THREE.Fog(...)` |
| Kamera | `PerspectiveCamera` (umum), `OrthographicCamera` (isometrik) |
| Geometri | `BoxGeometry`, `SphereGeometry`, `PlaneGeometry`, `CylinderGeometry`, `TorusGeometry` |
| Material | `MeshStandardMaterial` (realistis), `MeshBasicMaterial` (flat/unlit), `MeshNormalMaterial` (debug) |
| Cahaya | `DirectionalLight`, `AmbientLight`, `HemisphereLight`, `PointLight`, `SpotLight` |
| Load model | `GLTFLoader` (`three/addons/loaders/GLTFLoader.js`) |
| Kontrol kamera | `OrbitControls` (`three/addons/controls/OrbitControls.js`) |
| Post-processing | `EffectComposer` + passes (`three/addons/postprocessing/`) |
| WebGPU (eksperimental) | `WebGPURenderer` (`three/webgpu`) — default tetap WebGL |

## 6. Standar Data Hall 3D (project ini)

- **Floor**: putih raised-tile (`0xf8fafc` / `0xe8edf3`), tile 2×2 m + perforated tile di cold aisle.
- **Rak**: bodi + pintu **silver metalik** (`rack 0xc0c8d4 metal .7`, `door 0xaeb8c6 metal .75`); unit server di dalam **gelap** (`0x272c36`) + LED hijau/biru/kuning/merah + LCD biru; PDU oranye di sisi hot.
- **Aisle**: cold aisle glow biru `0x38bdf8` (supply 22 °C), hot aisle glow oranye `0xf97316` (return 35 °C); containment kaca `MeshPhysicalMaterial opacity .14`.
- **Overhead**: fiber trough kuning `0xeab308`, cable tray abu, busway biru `0x2563eb` + drop rod per baris.
- **CRAC**: putih `0xf1f5f9` + vent abu + layar hijau + pipa atas; 4 unit di ujung hall.
- **UPS**: dark `0x1f2937` + layar biru, di sisi selatan hall.
- **Pintu masuk**: kaca bening full (frame aluminium tipis), sign `DATA HALL • MASUK` biru dongker `0x00008B` background transparan + outline/stroke ijo `0x00cc44`.
- **Kamera awal**: dari pintu masuk `(0, 1.7, 10.8)` target `(0, 1.4, -4)`; auto-rotate mati default.
- **Tembok**: dilepas (open hall) agar terang & rotasi bebas; sisakan pilar + pipa fire merah + EXIT sign + CCTV.
- **Lighting**: `toneMappingExposure 1.9`, hemisphere 2.2 + ambient .9 + directional 2.2 + downlight pintu + strip plafon (tanpa plafon solid).
- **File**: `data-hall-3d/v3.html` (aktif), `v2.html`, `index.html` (arsip).

## 5. Checklist sebelum selesai

- [ ] Jalan tanpa error di console, tidak ada layar hitam (cek lighting + kamera)
- [ ] Resize browser tidak merusak aspect ratio
- [ ] FPS stabil, `pixelRatio` dibatasi, tidak ada alokasi di loop
- [ ] Ada loading / error / fallback WebGL
- [ ] Dispose dipanggil saat unmount, tidak ada memory leak
- [ ] Ikuti standar `archify` (kontras, radius, gap, mobile 360px untuk UI overlay)
