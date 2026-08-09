# 🎮 Galleria Live Map Overlay

A read-only browser-console JavaScript overlay for the **Galleria** web game that displays live player position, maze layout, coins, and movement trail in real-time.

![Game](https://img.shields.io/badge/Game-Galleria-blue)
![Language](https://img.shields.io/badge/Language-JavaScript-yellow)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)

---

## 🎯 What It Shows

- 🗺️ **Live maze/map layout** (97×97 grid)
- 🟡 **All 33 coin positions** with numbers
- 🔵 **Current player position** with direction indicator
- 🧭 **Player facing direction** (NORTH/SOUTH/EAST/WEST)
- 🔵 **Live movement trail** (blue line tracking your path)
- 📍 **Live X/Y world coordinates** (decimal precision)
- 🪙 **Collected coin status** (shown/hidden based on collection)
- 📊 **Live collected/total counter** (e.g., 12/33)

---

## ✨ Features

✅ **Fixed overlay** in top-left corner  
✅ **Read-only mode** - No game state modification  
✅ **No backend required** - Pure client-side JavaScript  
✅ **No React modification** - Works with existing game  
✅ **Live updates** - Real-time tracking  
✅ **Auto-reset** - Clears on page refresh  
✅ **High DPI support** - Works on retina displays  
✅ **Minimal footprint** - No dependencies  

---

## 🚀 Quick Setup (3 Simple Steps)

### **Step 1: Open the Game**

Navigate to: https://galleria.theflorentines.xyz/

Then:
1. Press `F12` (Open Developer Tools)
2. Click on the **Console** tab
3. Click **"Click to Enter"** in the game
4. Wait for gameplay to start

---

### **Step 2: Initialize Game State**

Copy and paste **CODE 1** into the Console:

```javascript
(() => {
  const canvas = document.querySelector("canvas");

  if (!canvas) {
    console.error("❌ Canvas নেই");
    return;
  }

  const fiberKey = Object.keys(canvas).find(k =>
    k.startsWith("__reactFiber")
  );

  if (!fiberKey) {
    console.error("❌ React Fiber নেই");
    return;
  }

  let fiber = canvas[fiberKey];

  for (let i = 0; i < 3 && fiber; i++) {
    fiber = fiber.return;
  }

  if (!fiber) {
    console.error("❌ fiber[3] পাওয়া যায়নি");
    return;
  }

  let hook = fiber.memoizedState;

  for (let i = 0; i < 16 && hook; i++) {
    hook = hook.next;
  }

  const state = hook?.memoizedState?.current;

  if (!state) {
    console.error(
      "❌ Game state পাওয়া যায়নি। আগে Click to Enter করে আবার চালাও।"
    );
    return;
  }

  window.GalleriaGameState = state;

  console.log(
    "%c✅ GalleriaGameState READY",
    "color:#00ff88;font-size:16px;font-weight:bold"
  );

  console.log("cam:", state.cam);
  console.log("map:", state.map?.length);
  console.log("powerups:", state.powerups?.length);
})();
```

Press **Enter** and wait for:

```
✅ GalleriaGameState READY
cam: {...}
map: 97
powerups: 33
```

---

### **Step 3: Start the Live Map**

Once you see `✅ GalleriaGameState READY`, paste **CODE 2** into the Console:

```javascript
(() => {
  "use strict";

  const ID = "__galleria_live_map_overlay__";

  const old = document.getElementById(ID);
  if (old) old.remove();

  if (!window.GalleriaGameState) {
    console.error(
      "❌ window.GalleriaGameState পাওয়া যায়নি। " +
      "আগে Code 1 চালাও।"
    );
    return;
  }

  const state = window.GalleriaGameState;

  if (!state.cam) {
    console.error("❌ state.cam পাওয়া যায়নি");
    return;
  }

  if (!Array.isArray(state.map)) {
    console.error("❌ state.map array নয়");
    return;
  }

  const MAP_ROWS = state.map.length;
  const MAP_COLS = state.map[0]?.length || MAP_ROWS;

  const MAP_SIZE = 300;
  const HEADER_HEIGHT = 38;

  const TRAIL_DISTANCE = 0.12;
  const MAX_TRAIL_POINTS = 1800;

  const root = document.createElement("div");

  root.id = ID;

  root.style.cssText = `
    position:fixed;
    top:12px;
    left:12px;
    width:${MAP_SIZE + 18}px;
    padding:8px;
    box-sizing:border-box;
    z-index:2147483647;
    background:rgba(3,18,15,.94);
    border:1px solid rgba(130,255,190,.65);
    border-radius:7px;
    box-shadow:
      0 0 0 1px rgba(0,0,0,.5),
      0 8px 30px rgba(0,0,0,.45);
    font-family:
      ui-monospace,
      SFMono-Regular,
      Menlo,
      Consolas,
      monospace;
    color:#d9ffe8;
    pointer-events:none;
    user-select:none;
  `;

  const header = document.createElement("div");

  header.style.cssText = `
    height:${HEADER_HEIGHT}px;
    display:flex;
    align-items:center;
    justify-content:space-between;
    padding:0 8px;
    box-sizing:border-box;
    font-size:12px;
    font-weight:700;
    letter-spacing:.4px;
    color:#baffd4;
  `;

  const title = document.createElement("span");
  title.textContent = "GALLERIA LIVE MAP";

  const counter = document.createElement("span");
  counter.textContent = "0 / 33";

  header.appendChild(title);
  header.appendChild(counter);

  root.appendChild(header);

  const canvas = document.createElement("canvas");

  canvas.style.cssText = `
    display:block;
    width:${MAP_SIZE}px;
    height:${MAP_SIZE}px;
    border:1px solid rgba(130,255,190,.45);
    border-radius:3px;
    background:#081712;
  `;

  root.appendChild(canvas);

  document.documentElement.appendChild(root);

  const ctx = canvas.getContext("2d");

  const DPR = Math.min(window.devicePixelRatio || 1, 2);

  canvas.width = MAP_SIZE * DPR;
  canvas.height = MAP_SIZE * DPR;

  ctx.scale(DPR, DPR);

  const status = document.createElement("div");

  status.style.cssText = `
    padding:6px 7px 1px 7px;
    font-size:9px;
    line-height:13px;
    color:#9bdcb5;
    white-space:nowrap;
    overflow:hidden;
  `;

  root.appendChild(status);

  const trail = [];

  let lastX = null;
  let lastY = null;

  function worldToMap(x, y) {
    return {
      x: (x / MAP_COLS) * MAP_SIZE,
      y: (y / MAP_ROWS) * MAP_SIZE
    };
  }

  function drawCircle(x, y, radius, fill, stroke, lineWidth = 1) {
    ctx.beginPath();
    ctx.arc(x, y, radius, 0, Math.PI * 2);

    if (fill) {
      ctx.fillStyle = fill;
      ctx.fill();
    }

    if (stroke) {
      ctx.strokeStyle = stroke;
      ctx.lineWidth = lineWidth;
      ctx.stroke();
    }
  }

  function drawMaze(map) {
    const cellW = MAP_SIZE / MAP_COLS;
    const cellH = MAP_SIZE / MAP_ROWS;

    ctx.fillStyle = "#081712";
    ctx.fillRect(0, 0, MAP_SIZE, MAP_SIZE);

    for (let y = 0; y < MAP_ROWS; y++) {
      const row = map[y];

      if (!row) continue;

      for (let x = 0; x < MAP_COLS; x++) {
        const value = Number(row[x]);

        const px = x * cellW;
        const py = y * cellH;

        if (value === 1) {
          ctx.fillStyle = "#183d34";

          ctx.fillRect(
            px,
            py,
            Math.ceil(cellW + 0.15),
            Math.ceil(cellH + 0.15)
          );
        } else {
          ctx.fillStyle = "#a7d58c";

          ctx.fillRect(
            px,
            py,
            Math.ceil(cellW + 0.15),
            Math.ceil(cellH + 0.15)
          );
        }
      }
    }
  }

  function drawTrail() {
    if (trail.length < 2) return;

    ctx.beginPath();

    for (let i = 0; i < trail.length; i++) {
      const p = trail[i];
      const m = worldToMap(p.x, p.y);

      if (i === 0) {
        ctx.moveTo(m.x, m.y);
      } else {
        ctx.lineTo(m.x, m.y);
      }
    }

    ctx.strokeStyle = "rgba(0,120,255,.72)";
    ctx.lineWidth = 2;
    ctx.lineJoin = "round";
    ctx.lineCap = "round";

    ctx.stroke();
  }

  function drawCoins(powerups) {
    if (!Array.isArray(powerups)) return;

    for (let i = 0; i < powerups.length; i++) {
      const coin = powerups[i];

      if (
        !coin ||
        !Number.isFinite(coin.x) ||
        !Number.isFinite(coin.y)
      ) {
        continue;
      }

      const m = worldToMap(coin.x, coin.y);
      const taken = !!coin.taken;

      if (taken) {
        drawCircle(
          m.x,
          m.y,
          2.1,
          "rgba(80,110,95,.45)"
        );

        continue;
      }

      drawCircle(
        m.x,
        m.y,
        4.1,
        "#ffd84a",
        "#5b3e00",
        1
      );

      drawCircle(
        m.x,
        m.y,
        1.25,
        "#fff4a3"
      );

      ctx.fillStyle = "rgba(20,30,20,.9)";
      ctx.font = "bold 6px monospace";
      ctx.textAlign = "center";
      ctx.textBaseline = "middle";

      ctx.fillText(
        String(i + 1),
        m.x,
        m.y + 0.1
      );
    }
  }

  function drawPlayer(cam) {
    if (
      !cam ||
      !Number.isFinite(cam.posX) ||
      !Number.isFinite(cam.posY)
    ) {
      return;
    }

    const m = worldToMap(
      cam.posX,
      cam.posY
    );

    let dx = Number(cam.dirX) || 0;
    let dy = Number(cam.dirY) || 0;

    const len = Math.hypot(dx, dy);

    if (len > 0) {
      dx /= len;
      dy /= len;
    } else {
      dx = 0;
      dy = -1;
    }

    const px = -dy;
    const py = dx;

    const tipX = m.x + dx * 9;
    const tipY = m.y + dy * 9;

    const leftX = m.x - dx * 5 + px * 5;
    const leftY = m.y - dy * 5 + py * 5;

    const rightX = m.x - dx * 5 - px * 5;
    const rightY = m.y - dy * 5 - py * 5;

    ctx.shadowColor = "#00e5ff";
    ctx.shadowBlur = 9;

    ctx.beginPath();

    ctx.moveTo(tipX, tipY);
    ctx.lineTo(leftX, leftY);
    ctx.lineTo(rightX, rightY);

    ctx.closePath();

    ctx.fillStyle = "#00d9ff";
    ctx.fill();

    ctx.shadowBlur = 0;

    ctx.strokeStyle = "#003d4a";
    ctx.lineWidth = 1;

    ctx.stroke();

    drawCircle(
      m.x,
      m.y,
      2.5,
      "#ffffff"
    );
  }

  function updateTrail(cam) {
    if (
      !cam ||
      !Number.isFinite(cam.posX) ||
      !Number.isFinite(cam.posY)
    ) {
      return;
    }

    const x = cam.posX;
    const y = cam.posY;

    if (lastX === null) {
      lastX = x;
      lastY = y;

      trail.push({
        x,
        y
      });

      return;
    }

    const distance = Math.hypot(
      x - lastX,
      y - lastY
    );

    if (distance >= TRAIL_DISTANCE) {
      trail.push({
        x,
        y
      });

      lastX = x;
      lastY = y;

      if (trail.length > MAX_TRAIL_POINTS) {
        trail.splice(
          0,
          trail.length - MAX_TRAIL_POINTS
        );
      }
    }
  }

  function render() {
    const s = window.GalleriaGameState || state;

    const cam = s?.cam;
    const map = s?.map;
    const powerups = s?.powerups || [];

    if (!cam || !Array.isArray(map)) {
      requestAnimationFrame(render);
      return;
    }

    updateTrail(cam);

    ctx.clearRect(
      0,
      0,
      MAP_SIZE,
      MAP_SIZE
    );

    drawMaze(map);
    drawTrail();
    drawCoins(powerups);
    drawPlayer(cam);

    const takenCount = powerups.filter(
      p => p && p.taken
    ).length;

    counter.textContent =
      `${takenCount} / ${powerups.length || 33}`;

    const x = Number(cam.posX);
    const y = Number(cam.posY);

    const grabbed =
      Number.isFinite(Number(s.grabbed))
        ? Number(s.grabbed)
        : takenCount;

    const facing =
      Number(cam.dirX) || Number(cam.dirY)
        ? (
            Math.abs(Number(cam.dirX)) >
            Math.abs(Number(cam.dirY))
              ? (
                  Number(cam.dirX) > 0
                    ? "EAST"
                    : "WEST"
                )
              : (
                  Number(cam.dirY) > 0
                    ? "SOUTH"
                    : "NORTH"
                )
          )
        : (s.facing || "?");

    status.textContent =
      `X ${x.toFixed(2)}  Y ${y.toFixed(2)}  ` +
      `• ${grabbed}/${powerups.length || 33}  ` +
      `• ${facing}`;

    requestAnimationFrame(render);
  }

  console.log(
    "%c🗺️ GALLERIA LIVE MAP STARTED",
    "color:#00ff88;font-size:16px;font-weight:bold"
  );

  console.log(
    "Map:",
    MAP_COLS,
    "×",
    MAP_ROWS
  );

  console.log(
    "Player:",
    state.cam.posX,
    state.cam.posY
  );

  console.log(
    "Coins:",
    state.powerups?.length
  );

  console.log(
    "%cRead-only overlay — game state is not modified.",
    "color:#8ee8b0"
  );

  requestAnimationFrame(render);
})();
```

Press **Enter**.

---

## ✅ Success!

You should see in the Console:

```
🗺️ GALLERIA LIVE MAP STARTED
Map: 97 × 97
Player: (x, y)
Coins: 33
Read-only overlay — game state is not modified.
```

**The Live Map will appear in the top-left corner!** 🎉

Move around the game and watch the overlay update in real-time:
- ✅ Player position
- ✅ X/Y coordinates
- ✅ Direction facing
- ✅ Movement trail
- ✅ Coin counter

---

## 📊 Status Display

The status bar shows:

```
X 48.25  Y 52.10  • 12/33  • NORTH
```

| Component | Meaning |
|-----------|---------|
| `X 48.25` | Player X coordinate |
| `Y 52.10` | Player Y coordinate |
| `12/33` | 12 coins collected out of 33 total |
| `NORTH` | Facing direction |

---

## 🗺️ Map Elements

| Element | Color | Meaning |
|---------|-------|---------|
| **Green Cell** | `#a7d58c` | Walkable path |
| **Dark Cell** | `#183d34` | Wall/obstacle |
| **Blue Line** | `rgba(0,120,255,.72)` | Your movement trail |
| **Yellow Circle** | `#ffd84a` | Uncollected coin |
| **Gray Circle** | `rgba(80,110,95,.45)` | Collected coin |
| **Cyan Triangle** | `#00d9ff` | Your player + direction |

---

## 🔄 After Page Refresh

The overlay is temporary. After refreshing, repeat:

```
1. Click "Click to Enter"
2. Paste CODE 1 → Press Enter
3. Paste CODE 2 → Press Enter
```

---

## 🧩 Technical Architecture

```
Galleria Game (React)
        ↓
React Fiber Tree (Canvas Component)
        ↓
GalleriaGameState
├── cam (player position & direction)
│   ├── posX, posY
│   ├── dirX, dirY
├── map (97×97 maze grid)
└── powerups (33 coin objects)
        ↓
Live Map Canvas Overlay
├── Maze rendering
├── Trail drawing
├── Coin visualization
└── Player indicator
```

---

## ⚙️ Performance

| Metric | Value |
|--------|-------|
| **Canvas Size** | 300×300px |
| **Map Size** | 97×97 cells |
| **Max Trail Points** | 1800 |
| **Trail Update Distance** | 0.12 units |
| **Refresh Rate** | 60 FPS (requestAnimationFrame) |
| **DPI Support** | Up to 2x (Retina) |

---

## ⚠️ Important Notes

🔒 **Read-only mode** - Game state is never modified  
🌐 **No backend** - 100% client-side JavaScript  
📴 **No dependencies** - Works standalone  
🎮 **Game controls** - Use normal game controls to move  
🔄 **Auto-reset** - Clears on every page refresh  
⚡ **Performance** - Lightweight and efficient  

---

## 🐛 Troubleshooting

| Issue | Solution |
|-------|----------|
| `❌ Canvas নেই` | Canvas not found - Wait for game to fully load |
| `❌ React Fiber নেই` | React structure not accessible - Game may have updated |
| `❌ GalleriaGameState পাওয়া যায়নি` | Run CODE 1 before CODE 2 |
| No overlay appearing | Check Console tab for error messages |
| Coins not showing | Refresh page and run both codes again |
| Overlay flickering | This is normal during map rendering |

---

## 📜 License

For educational and personal experimentation only.

---

## 🎮 Game Link

**Galleria** by The Florentines  
https://galleria.theflorentines.xyz/

---

## ❤️ Made with Love

Created for Web3 game enthusiasts and blockchain explorers.

**Enjoy exploring Galleria!** 🗺️✨
