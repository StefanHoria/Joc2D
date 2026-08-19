# Joc2D — A 2D Tile-Based Game Engine in Java

A personal learning project: a top-down 2D game written from scratch in **Java**, with no game
engine and no external libraries — only the JDK's own `Swing` and `AWT`. The point of the project
was to understand what a game engine actually does underneath: the game loop, frame timing, sprite
animation, tile maps and keyboard input.

| | |
|---|---|
| **Author** | Ștefan Horia-Eusebiu |
| **Language** | Java (Swing / AWT, no external dependencies) |
| **Type** | Personal project, in progress |
| **Started** | October 2024 |

---

## What is implemented

**Fixed-timestep game loop.** `GamePanel` implements `Runnable` and runs on its own thread. Each
iteration calls `update()` and `repaint()`, then sleeps for exactly the time left until the next
frame:

```java
double drawInterval = 1000000000 / FPS;          // 60 FPS → 16.67 ms per frame
double nextDrawTime = System.nanoTime() + drawInterval;
// ... update(); repaint();
Thread.sleep((long) remainingTime);
nextDrawTime += drawInterval;
```

Accumulating `nextDrawTime` rather than sleeping for a fixed duration keeps the frame rate from
drifting when a frame takes longer than expected.

**Rendering with double buffering.** `setDoubleBuffered(true)` on the panel, drawing through
`Graphics2D` in the overridden `paintComponent`, tiles first and the player on top.

**Tile system.** `TileManager` loads the tile textures and reads the map from a text file where each
number is an index into the tile array:

```
4 4 4 4 4 4 4 4 4 4 4 4 4 4 4 4 …
4 4 4 4 4 4 4 4 4 4 0 2 2 2 2 0 …
```

**Sprite animation.** The player has 8 sprites (2 animation frames × 4 directions). A counter swaps
frames every 12 game ticks, i.e. roughly 5 times per second at 60 FPS — fast enough to read as
walking, slow enough not to flicker.

**Keyboard input.** `KeyHandler` implements `KeyListener` and keeps a boolean per direction (`W`,
`A`, `S`, `D`), set on `keyPressed` and cleared on `keyReleased`. Reading state instead of reacting
to events means movement is smooth and continuous while a key is held.

**Screen geometry.** 16×16 px tiles scaled 3× → 48 px on screen; a 16×12 tile viewport → a 768×576
window.

## Project structure

```
.
├── src/main/java/
│   ├── joc2d/
│   │   ├── Main.java          entry point — builds the JFrame and starts the thread
│   │   ├── GamePanel.java     the game loop, screen geometry, rendering
│   │   └── KeyHandler.java    keyboard input (WASD)
│   ├── entity/
│   │   ├── Entity.java        base class: world position, speed, sprites, direction
│   │   └── Player.java        the player — movement, animation, drawing
│   └── tile/
│       ├── Tile.java          one tile: texture + collision flag
│       └── TileManager.java   loading textures, reading the map, drawing
└── res/
    ├── player/                8 player sprites (2 frames × 4 directions)
    ├── tiles/                 38 tile textures
    └── maps/                  the maps, as text files of tile indices
```

## Running it

There is no build file in the repository yet, so the project is compiled directly with the JDK
(17 or newer). The `res/` folder is the resources root and has to be on the classpath, since the
code loads its assets through `getResourceAsStream`:

```bash
javac -d out $(find src/main/java -name "*.java")
java -cp out:res joc2d.Main
```

On Windows, use `;` instead of `:` as the classpath separator:

```bash
java -cp "out;res" joc2d.Main
```

Alternatively, open the folder in IntelliJ IDEA, mark `src/main/java` as *Sources Root* and `res` as
*Resources Root*, then run `Main`.

**Controls:** `W` `A` `S` `D` to move.

## Current state and what comes next

This is a project in progress, and the code shows it honestly. What is not yet implemented:

- **Collisions.** `Tile` already carries a `collision` boolean, but nothing reads it yet — the
  player walks through everything. This is the next step.
- **The camera.** The player is drawn at a fixed point on screen and the tiles are drawn at fixed
  screen coordinates, without a camera offset. The player's world coordinates do change, but the
  view does not follow them yet, so the world does not scroll.
- **Loading the full map.** The world is declared as 50×50 tiles, but `loadMap()` and `draw()` are
  bounded by `maxScreenCol` / `maxScreenRow` (16×12), so only the visible corner of `world01.txt` is
  read. The map file itself already contains the full 50-column layout.
- **`worldHeight`** is computed as `tileSize * maxScreenRow` instead of `maxWorldRow` — it will need
  fixing along with the camera.

## What I learned

- A game loop is not just "an infinite `while`" — without controlled timing, the game runs at a
  different speed on every machine.
- Reading input as *state* (a boolean per key) rather than as *events* is what makes movement feel
  continuous instead of stuttering.
- Separating `update()` from `draw()` is the distinction that keeps game logic independent of
  rendering.
- A tile map as a plain text file of indices is a surprisingly effective format: it can be edited in
  any text editor and read with a handful of lines of code.

---

Academic and personal work, published for portfolio purposes.
