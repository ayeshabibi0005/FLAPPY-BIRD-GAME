# 🐦 Flappy Bird — Enhanced Python Edition

>A simple remake of the classic game, built from scratch in pure Python with custom visuals, moving skies, and a small but detailed bird.

---

## What Is This?

This is a simple Flappy Bird clone I made using Python and tkinter. I didn’t use any external libraries like Pygame or Pillow — everything you see is drawn directly on the canvas using basic shapes and a bit of math.

I originally just wanted to get the game working, but I ended up spending more time on how it looks. So instead of keeping it plain, I added things like a soft sky gradient, a glowing sun, and clouds that move at different speeds to give a bit of depth. There are also some rolling hills in the background, and the pipes have a bit of shading to make them look less flat.

The ground scrolls with a bit of texture, and the bird isn’t just a static shape — its wings actually move using a sine wave, which makes it feel a little more alive. It’s still a small project, but I tried to make it look nicer than a typical basic clone.

---

## Demo at a Glance

| Feature | Details |
|---|---|
| Language | Python 3 (standard library only) |
| GUI Framework | `tkinter` Canvas |
| Window Size | 800 × 600 px |
| Target FPS | ~62 fps (16 ms tick) |
| Controls | `SPACE` to flap · `ENTER` to restart |

---

## Getting Started

### Prerequisites

All you need is Python 3. `tkinter` ships with almost every Python installation. To double-check:

```bash
python -m tkinter
```

A small window should pop up. If it does, you're good to go.

### Running the Game

```bash
python flappy_bird.py
```

That's it. No `pip install`, no setup.

---

## How to Play

- **Press `SPACE`** to make the bird flap upward.
- Navigate through the gap between the top and bottom pipes.
- **Every pipe you clear** adds 1 to your score.
- Your **best score** is tracked for the whole session.
- **Crash into a pipe, the ceiling, or the ground** and it's game over.
- **Press `ENTER`** on the Game Over screen to try again.

---

## Project Structure

The entire game lives in a single file:

```
flappy_bird.py
│
├── Constants (top of file)
│   └── Window size, physics values, pipe dimensions, color palettes
│
└── class FlappyBirdGame
    ├── __init__()          ← Sets up the window, binds keys, spawns clouds
    ├── reset_game_state()  ← Resets score, bird position, pipe position
    ├── update()            ← The game loop — physics, movement, collision, scoring
    ├── _collision()        ← AABB collision detection
    │
    └── Drawing Methods
        ├── draw()              ← Master render call (clears + redraws everything)
        ├── _draw_sky()         ← 10-band gradient background
        ├── _draw_sun()         ← Glowing sun with highlight ring
        ├── _draw_clouds()      ← Two-layer parallax cloud system
        ├── _draw_hills()       ← Sine-curve rolling hills
        ├── _draw_pipes()       ← 3D pipes with cap and shading
        ├── _draw_ground()      ← Layered scrolling ground with grass tufts
        ├── _draw_bird()        ← Detailed bird with wing animation
        ├── _draw_hud()         ← Score and keybind display
        └── _draw_gameover()    ← Styled Game Over panel
```

---

## Under the Hood — The Interesting Bits

### The Game Loop

The game runs on `root.after(16, self.update)`, which schedules the `update()` method to run every 16 milliseconds — roughly 62 frames per second. Each frame, `update()` does four things in order:

1. Applies gravity to the bird's velocity and moves it downward
2. Moves the pipe leftward at a fixed speed
3. Checks whether the bird just cleared a pipe (scoring)
4. Checks for a collision (game over)

Then it calls `draw()` to repaint the entire screen from scratch.

### Physics

The bird's movement is modelled with a simple velocity accumulator:

```
velocity += GRAVITY      # 0.55 per frame — pulls the bird down
bird_y   += velocity     # position updates each tick
```

When you press `SPACE`, velocity is snapped to `JUMP_VELOCITY` (`-10.5`) — negative because tkinter's Y-axis points downward, so a negative value moves the bird *up*.

### Collision Detection

Collision uses **AABB (Axis-Aligned Bounding Box)** — treating the bird as a square region around its centre, then checking four conditions:

- Is the bird horizontally overlapping with the pipe? (`in_x`)
- Is it above the top pipe's gap? (`hit_t`)
- Is it below the bottom pipe's gap? (`hit_b`)
- Has it hit the ground or ceiling? (`hit_g`, `hit_c`)

If `in_x` is true *and* either `hit_t` or `hit_b` is true — or either boundary is hit — the game ends.

### Parallax Clouds

There are two independent cloud layers — `clouds_far` (7 slower clouds) and `clouds_near` (4 faster clouds). Each frame, far clouds move at `0.5 px/frame` and near clouds at `1.3 px/frame`. When a cloud scrolls off the left edge, it teleports back to the right with a freshly randomized Y position and scale, creating a seamless infinite scroll.

The far layer is drawn with desaturated bluish-grey tones to simulate depth; the near layer uses brighter whites.

### Wing Animation

The bird's wing droop is computed each frame using:

```python
droop = math.sin(self.wing_angle) * 15
```

`wing_angle` increments by `0.20` each frame and reverses direction when it goes beyond `±0.5`, giving the wing a natural flapping rhythm. When you jump (press `SPACE`), `wing_angle` is forced to `-0.7` to trigger a sharp upstroke — a small touch that makes the jump feel responsive.

### 3D Pipes

The pipes aren't just plain green rectangles. Each pipe segment draws:

- A **mid-green body**
- A **bright left strip** (simulating light hitting the left face)
- A **narrow transition band** (where light meets shadow)
- A **dark right strip** (shadow)
- A **near-black edge** on the far right

The cap at the mouth of each pipe is slightly wider and gets the same treatment, plus a small horizontal shine line across its face. It's about 10 draw calls per pipe, but it sells the illusion well.

### Ground

The ground is built in layers:

- **Dark soil** band at the very bottom
- **Mid-tone dirt** layer above it
- **Grass strip** at the top
- **Bright green highlight** on the upper edge of the grass
- **Scrolling grass tufts** — small ovals that move left in sync with the pipes, wrapping around to create the illusion of continuous ground movement

---

## Tweakable Constants

All the numbers that control how the game *feels* are declared at the top of the file, making them easy to tune:

| Constant | Default | What It Does |
|---|---|---|
| `GRAVITY` | `0.55` | How fast the bird falls each frame |
| `JUMP_VELOCITY` | `-10.5` | How hard the bird jumps (more negative = higher) |
| `PIPE_SPEED` | `4` | How fast pipes scroll left |
| `PIPE_GAP` | `185` | Vertical gap between top and bottom pipe |
| `BIRD_RADIUS` | `20` | Collision radius of the bird |
| `PIPE_WIDTH` | `78` | Width of each pipe |

Try raising `PIPE_SPEED` to `6` or lowering `PIPE_GAP` to `140` for a serious challenge.

---

## Known Limitations

- **Single pipe at a time** — only one pair of pipes is active on screen. A multi-pipe system could be added by replacing the single `pipe_x`/`pipe_cy` state with a list.
- **No sound** — `tkinter` doesn't have built-in audio. Adding `pygame.mixer` or `playsound` would handle this.
- **High score resets on close** — there's no persistence. A simple `json` or `shelve` write on exit would fix it.
- **Collision is square, bird is round** — the AABB check is slightly generous at the corners. A circular collision check would be more precise but trickier to implement without an external library.

---

## Why tkinter?

Because constraints breed creativity. Using a full game library like Pygame is the obvious choice, but building this with only `tkinter.Canvas` means every visual effect — gradients, 3D shading, parallax, wing animation — had to be achieved with rectangles, ovals, and polygons. That's what makes this project genuinely interesting to read.

---

## License

Do whatever you want with it. Change the colors, rewrite the bird, add more pipes, ship it as your own game. It's a learning project — learn from it.

---

*Built with Python 3 · Zero dependencies · One file · Pure fun.*

