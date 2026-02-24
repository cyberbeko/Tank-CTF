# 🏜️ Tank CTF — Desert Sandbox with Neural Network AI

> A fully browser-based 3D Capture the Flag tank game built with Three.js, featuring procedural desert maps, custom tank shapes, paint skins, and an in-browser Neural Network + Reinforcement Learning enemy AI system — all in a single HTML file.

---

## 🎮 Play It Live

**[▶ Launch Game](https://dede3phc22dgx.cloudfront.net/creao2/492ad92a-018b-4b1d-b921-558e8da2a3d4/f8714340-e0f1-7095-02a8-f4632fda3cf2/535c8a10-c281-4bb6-beba-6ec05a205d39/tank_game.html)**

No install. No server. Just open and play.

---

## 📸 Features at a Glance

- **4v4 Capture the Flag** — Blue team (you + 3 AI allies) vs 4 Red AI enemies
- **3 Tank Types** — Hornet (fast/fragile), Mammoth (slow/armored), Viking (balanced)
- **4 Procedural Paint Skins** — Leopard, Alligator, Orange Blaze, Carpet Royal Pattern
- **Desert Sandbox Map** — sand texture, dunes, rocky outcrops, craters, palm trees, sandbag bunkers, ruined walls
- **Neural Network AI** — 16-input → 3 hidden layers → 5-output policy network
- **Reinforcement Learning** — epsilon-greedy exploration, experience replay, ES policy gradient
- **Infinite Respawns** — no permanent deaths, 3-second player countdown
- **Live NN HUD** — action probabilities, reward graph, weight heat map
- **Zero dependencies** — one `.html` file, Three.js loaded from CDN

---

## 🛠️ How I Built It — Step by Step

### Step 1 — Basic 3D Tank Game

The foundation was a standard Three.js scene rendered inside a `<canvas>` element. I set up:

- A **perspective camera** following the player tank from behind using `lerp()` for smooth movement
- **Directional + ambient + hemisphere lighting** for depth and shadow
- A flat green ground plane with `receiveShadow`
- **WASD + Arrow Key** movement with tank rotation
- **Pointer Lock API** (`requestPointerLock`) so the mouse controls turret aim without leaving the screen
- A click-to-shoot system that fires `THREE.SphereGeometry` bullets with `PointLight` attached for a glow effect
- Simple circle-box collision detection for tanks vs walls

```js
// Core movement loop
player.angle += rotDir * player.rotSpeed * dt;
player.group.rotation.y = player.angle;
player.group.position.x += Math.sin(player.angle) * mv * player.speed * dt;
player.group.position.z += Math.cos(player.angle) * mv * player.speed * dt;
```

---

### Step 2 — Control Remapping

Remapped controls to feel more natural for tank gameplay:

| Key | Action |
|-----|--------|
| W / ↑ | Move forward |
| S / ↓ | Move backward |
| A / ← | Turn left |
| D / → | Turn right |
| Z | Turret left |
| X | Turret right |
| C | Center turret |
| Click | Shoot |

---

### Step 3 — 4v4 Capture the Flag

This was the biggest design change. I restructured the entire game around two teams:

**Team System**
- `blueTeam[]` — player + 3 AI allies
- `redTeam[]` — 4 AI enemies

**Flag Mechanics**
- Two flags spawned at opposite ends of the arena (`z = ±78`)
- Each flag has a pole, cloth mesh, ring base, and a colored `PointLight` beacon
- Pickup radius: `5 units` — when a tank drives within range of an uncarried flag, it's picked up
- Capture: carrier must return to their own base (`8 units`) while own flag is still at base
- **F key** — `tryPassFlag()` transfers the flag to the nearest alive ally within 18 units

```js
// Flag pickup detection
if (!tank.hasFlag && !ef.carrier && ef.mesh.position.distanceTo(tank.group.position) < 5) {
  ef.carrier = tank;
  tank.hasFlag = true;
}

// Capture detection
if (tank.hasFlag && ownFlag.atBase && tank.group.position.distanceTo(ownFlag.base) < 8) {
  score++;
}
```

**Respawn System**
- On death: tank goes invisible, `respawnTimer` set (3s player, 4–5s AI)
- On respawn: full HP restored, random team spawn point, visible again
- Player sees a countdown overlay: `💀 DESTROYED — RESPAWNING IN 3s`

---

### Step 4 — Tank Shapes (3 Hull Types)

Each tank is built as a `THREE.Group` hierarchy:
```
group (root)
  ├── body mesh
  ├── track meshes (×2)
  ├── nose / skirt / fin details
  └── turretGroup
        ├── turret mesh
        ├── cheek / mantlet details
        └── barrel mesh(es)
```

| Tank | Speed | HP | Signature |
|------|-------|----|-----------|
| **Hornet** | 24 | 75 | Slim wedge, twin barrels, fast rotation |
| **Mammoth** | 11 | 160 | Wide armored hull, heavy single barrel, side skirts |
| **Viking** | 17 | 110 | Angular fins, hexagonal-ish turret, balanced |

Tank-specific gameplay differences:
- **Bullet speed:** Hornet 65, Viking 55, Mammoth 42
- **Damage taken:** Hornet 30, Viking 25, Mammoth 16
- **Shoot cooldown:** Hornet 0.25s, Viking 0.4s, Mammoth 0.7s

---

### Step 5 — Procedural Paint Skins

All paint textures are generated at runtime using the **Canvas 2D API** → `THREE.CanvasTexture`. No image files needed.

```js
function makeLeopardTexture(teamColor) {
  const canvas = document.createElement('canvas');
  canvas.width = canvas.height = 128;
  const ctx = canvas.getContext('2d');
  // Tan base
  ctx.fillStyle = '#c8a45a';
  ctx.fillRect(0, 0, 128, 128);
  // Dark spots with golden centers
  spots.forEach(s => {
    ctx.fillStyle = '#2d1a00';
    ctx.arc(s.x, s.y, s.r, 0, Math.PI * 2);
    ctx.fill();
  });
  // Team color tint overlay (e.g. #2563eb28 for blue)
  ctx.fillStyle = `#${teamHex}28`;
  ctx.fillRect(0, 0, 128, 128);
  return new THREE.CanvasTexture(canvas);
}
```

| Paint | Technique |
|-------|-----------|
| **Leopard** | Tan base + dark spot circles with golden inner circles |
| **Alligator** | Dark green + elliptical scale grid with row offset |
| **Orange Blaze** | Linear flame gradient + diagonal stripe overlay |
| **Carpet** | Dark red + repeating diamond pattern in gold/purple |

Each paint receives a semi-transparent team color tint so blue team tanks look blue-tinted and red team tanks look red-tinted — same skin, different teams.

---

### Step 6 — Desert Sandbox Map

The old flat green arena was replaced with a fully procedural desert environment:

**Ground**
- `PlaneGeometry(240, 240, 80, 80)` — subdivided for vertex displacement
- Gentle sine-wave height noise applied to vertices: `sin(x*0.08) * cos(z*0.09) * 0.6`
- Procedural sand texture: radial gradient base + 6000 grain dots + sine wind ripple lines, tiled 10×10

**Environment Objects**
| Element | Details |
|---------|---------|
| Sand dunes | Scaled `SphereGeometry`, half-buried, non-blocking |
| Rocky outcrops | `BoxGeometry` with stacked rock chips, blocking |
| Sandbag bunkers | `BoxGeometry` with burlap-weave canvas texture |
| Craters | `RingGeometry` rim + `CircleGeometry` dark center |
| Palm trees | Cylinder trunk + cone fronds + sphere coconuts |
| Ruined walls | Sandstone blocks + rotated crumbled chunks |
| Pebbles | 60 scaled spheres scattered randomly |
| Boundary walls | Sandstone walls with randomized crenellations |

**Lighting**
```js
// Warm desert sun
const sun = new THREE.DirectionalLight(0xffcc77, 1.6);
sun.position.set(60, 100, 30);

// Warm sky + hot sand fill
scene.add(new THREE.HemisphereLight(0xfff0c0, 0xc8a060, 0.45));

// Exponential distance haze
scene.fog = new THREE.FogExp2(0xd4b87a, 0.008);
```

---

### Step 7 — Neural Network + Reinforcement Learning AI

The final step replaced the old scripted AI with a fully in-browser trained neural network.

#### Network Architecture

```
Input Layer  (16 neurons)
      ↓  He-initialized weights
Hidden Layer 1  (24 neurons, ReLU)
      ↓
Hidden Layer 2  (20 neurons, ReLU)
      ↓
Hidden Layer 3  (16 neurons, ReLU)
      ↓  Softmax
Output Layer  (5 neurons = 5 actions)
```

**Input features (state vector):**
```
[0-1]  Enemy flag direction (dx/100, dz/100)
[2-3]  Own base direction (dx/100, dz/100)
[4-5]  Nearest enemy direction (dx/100, dz/100)
[6]    Nearest enemy distance (normalized)
[7-8]  Tank facing vector (sin/cos of angle)
[9-12] Wall proximity — 4 raycasts (front/back/left/right)
[13]   Own HP (normalized 0–1)
[14]   Is carrying flag? (0 or 1)
[15]   Is enemy carrying flag? (0 or 1)
```

**Output actions:**
```
0 = Move Forward
1 = Turn Left
2 = Turn Right
3 = Shoot (aim + fire at nearest enemy)
4 = Flag Run (navigate to flag or base)
```

#### Reinforcement Learning Loop

```
Every 80ms per tank:
  1. Observe current state (16-vector)
  2. Select action (epsilon-greedy: random OR argmax(NN output))
  3. Execute action in game
  4. Observe next state + compute reward
  5. Store (state, action, reward, nextState) in replay buffer
  6. Sample 28 random transitions and train NN
```

**Reward shaping:**
| Event | Reward |
|-------|--------|
| Flag captured | +20.0 |
| Flag picked up | +8.0 |
| Moving toward enemy flag | +0.12 × distance closed |
| Moving toward own base (with flag) | +0.18 × distance closed |
| Shooting near enemy | +0.6 |
| Shooting with no enemy nearby | -0.3 |
| Wall collision (stuck) | -0.5 |
| Death | -10.0 |
| HP loss | -2.0 × (damage / maxHP) |
| Alive (step cost) | -0.01 |

#### Training Algorithm

Used **Evolution Strategies (ES)** with antithetic sampling — a lightweight alternative to full backpropagation that runs efficiently in real-time:

```js
// Generate two mirrored perturbations
const noise = weights.map(() => Math.random() * 2 - 1);
const nn1_weights = weights.map((v, i) => v + noise[i] * sigma);
const nn2_weights = weights.map((v, i) => v - noise[i] * sigma);  // antithetic

// Evaluate both on mini-batch
const f1 = batchLogLikelihood(nn1, batch);
const f2 = batchLogLikelihood(nn2, batch);

// Update in direction of higher fitness
newWeights = weights.map((v, i) => v + lr * (f1 * noise[i] + f2 * noise[i]) / batchSize);
```

**Additional RL features:**
- **Experience Replay Buffer** — 1200 transitions, prevents correlated updates
- **Target Network** — copy of main network, synced every 80 steps for training stability
- **Epsilon decay** — starts at 35% random exploration, decays to 4%
- **Centralized training** — all 7 AI tanks share one network (parameter sharing)

#### Wall Avoidance

Tanks cast 4 rays (front/back/left/right) each step:
```js
// If wall within 24 units in front, rotate toward more open side
if (wallDists[0] > 0.6) {
  turnLeft  = wallDists[2] < wallDists[3]; // rotate toward open side
  turnRight = !turnLeft;
}
```

---

## 📂 Project Structure

```
tank_game.html          ← Entire game (single file, ~1300 lines)
│
├── <style>             CSS HUD, overlay, NN HUD, selection cards
├── <canvas>            Three.js rendering target
├── <div id="hud">      HP bar, score, flag status, minimap, feed
├── <div id="nn-hud">   Live NN visualization panel
├── <div id="overlay">  Start/end screen with tank+paint selection
│
└── <script>
    ├── TANK_TYPES      3 hull builders (Hornet/Mammoth/Viking)
    ├── PAINTS          4 canvas texture generators
    ├── buildSelectionUI()  Start screen cards with 2D previews
    ├── init()          Scene, lighting, camera, renderer
    ├── makeSandTexture()   Procedural sand (grain + ripple)
    ├── makeRockTexture()   Procedural rock (noise + cracks)
    ├── makeSandbagTexture() Procedural burlap weave
    ├── buildArena()    Full desert map construction
    ├── NeuralNetwork   FC network class (forward, toFlat, fromFlat)
    ├── RLAgent         RL controller (selectAction, remember, train)
    ├── buildState()    16-input state vector builder
    ├── wallRayCast()   4-direction wall proximity sampling
    ├── computeReward() Reward shaping function
    ├── updateAI()      NN+RL decision loop per tank
    ├── executeAction() Translates action → movement + shoot
    ├── updateNNHud()   Live NN visualization updater
    ├── updateFlags()   Flag pickup / carry / capture logic
    ├── killTank()      Death + infinite respawn scheduling
    └── animate()       Main game loop (requestAnimationFrame)
```

---

## 🧠 Technical Notes

### Why a single HTML file?
This keeps the project fully portable — share the URL, open the file, done. No build tools, no npm, no webpack.

### Why ES (Evolution Strategies) instead of backprop?
True backpropagation requires storing the computation graph and implementing gradient flow through every operation. ES is a clean approximation that works well for low-parameter networks and runs in real-time without slowing the game loop.

### Why shared weights across all AI tanks?
Parameter sharing means all 7 AI tanks contribute experience to one network simultaneously. This dramatically speeds up training — 7 tanks × 12.5 decisions/second = ~87 training samples per second.

### Why Canvas textures instead of image files?
Single-file constraint. Canvas 2D gives enough expressiveness for tileable procedural patterns and avoids any cross-origin image loading issues.

---

## 🚀 Technologies

| Tech | Use |
|------|-----|
| [Three.js r128](https://threejs.org) | 3D rendering, geometry, lighting, shadows |
| Canvas 2D API | Procedural texture generation |
| Pointer Lock API | Mouse-based turret control |
| Vanilla JS (ES6) | Game loop, physics, AI, RL |
| HTML/CSS | HUD, overlay, selection UI |

---

## 📋 Controls

| Key / Action | Effect |
|---|---|
| **W** / **↑** | Move forward |
| **S** / **↓** | Move backward |
| **A** / **←** | Turn left |
| **D** / **→** | Turn right |
| **Z** | Rotate turret left |
| **X** | Rotate turret right |
| **C** | Center turret |
| **Mouse move** | Aim turret (after clicking) |
| **Left click** | Shoot |
| **F** | Pass flag to nearest ally |

---

## 💡 What I Learned

- **Three.js scene graph** — using `Group` hierarchies to separate hull rotation from turret rotation cleanly
- **Procedural textures** — Canvas 2D can generate surprisingly convincing materials with just math
- **Reinforcement learning without libraries** — a neural network and policy gradient fit in ~200 lines of vanilla JS
- **Reward shaping is hard** — getting the AI to actually capture flags (not just shoot) required carefully tuning the reward magnitudes
- **Wall avoidance** is better done with raycasts than learned behavior — RL is slow to learn spatial awareness; explicit checks are faster and more reliable

---

## 🔮 Ideas for Future Updates

- [ ] Save/load trained NN weights to `localStorage` — persistent learning between sessions
- [ ] Blue AI allies also use RL — self-play between two NN teams
- [ ] Destructible sandbag walls
- [ ] Sandstorm weather event (dynamic fog + speed penalty)
- [ ] Supply crates with HP, speed boost, rapid-fire pickups
- [ ] Training speed slider (2×/4× simulation speed)
- [ ] Leaderboard via simple backend

---

## 📄 License

MIT — use it, fork it, learn from it.

---

*Built entirely in the browser. No build step. No framework. Just HTML, CSS, JavaScript, and Three.js.*
