# Levels

## Format (`levelNN.json`)

- `grid`: `[cols, rows]`. Cells are `[x, y]` with `origin: bottom-left`.
- `start` / `goal`: cells.
- `pieces[]`: `id`, `fabric` (a tile name in `assets/extracted/fabrics/`; the game draws the real tile, embedded by `tools/bundle_fabrics.py`), `color` (the palette base; if it differs from the fabric's base, the tile is tinted to it, as for the binding), `rect` (`[x, y, w, h]` in cells), and `z` (stacking only, no gameplay role).
  - Later, irregular shapes can use `cells: [[x,y],...]` in place of `rect`.
- `frame` (optional): the binding, a one-cell ring around the quilt (cells with x or y = -1 or = cols/rows). `enter` / `exit` are `"free"` or `"stitch"`.
- Zhuldyz needs no level data: it links pieces with the same `fabric` (see rule 7). Which pieces share a cloth is part of the level design. Give the binding a fabric no piece uses, and make same-fabric pieces the same `color` too.

## Each round a different quilt

The game varies the authored level on every load (`varyQuilt` in `build/index.html`) from a random seed. Opening `?seed=N` replays one quilt (the current seed is `quraqDebug.seed`), and `?seed=0` plays the level exactly as written here:

- **Layout:** one of the 8 symmetries of the square (transpose, then mirror either axis). Start and goal move with the pieces, so she can begin in any corner. No rule depends on direction.
- **Cloths:** each same-cloth group (pieces sharing a `fabric`) gets another of the level's fabrics, as a permutation, and `color` becomes that fabric's base. Twins stay twins and pieces with no twin still have none. A deal is kept only if touching pieces are at least 55 apart in RGB (borders must read) and the goal piece is light (luma ≥ 120, so the scissors on it read). It takes the best of up to 300 tries.
- The binding never changes. Restart deals a new quilt; with an explicit `?seed=N`, restart keeps it. The maps below show the authored quilt.

## Rules

1. **Walkable cells:** a piece owns the cells where it is the top visible layer. The girl walks 4-neighbour inside the owned cells of her current piece.
2. **Border:** every orthogonal edge between cells owned by different pieces blocks her (`DESIGN.md`). Diagonal corners are not borders.
3. **Sewing:** a seam is made of one-cell segments. With Koshkar muiyz picked, the player drags along a seam, or taps it, to sew it segment by segment. Sewing any segment re-pieces the two pieces either side as the **koshkar staircase** of `assets/extracted/ornaments/koshkar_muiyz.svg`: a square centred on the seam, as long as the stretch of seam the two pieces share, is split corner to corner by the staircase, with interlocking hooks at every step (tall and short steps alternate). One piece's fabric fills each side. Steps stay square, about 2 cells each. The **whole seam between those two pieces** opens: every edge on it becomes a bridge she can cross both ways, jumping along a parabola. The staircase only replaces cloth still showing its own piece's fabric, so earlier ornaments stay.
   - **Which seams:** any seam touching a cell she can currently walk to (her piece, pieces linked by bridges, and the frame).
   - **Resuming:** when she stops at a seam, she carries on toward the tapped cell as soon as that seam is sewn or another bridge makes the cell reachable.
   - **Panel:** the ornament panel stays docked. It wiggles when she stops at a seam. (`DESIGN.md` now matches this.)
4. **Tap she can reach:** she takes the shortest path, choosing the one closest to the straight line toward the tap. **She never goes the long way:** if the only route is longer than walking straight (a detour round a piece to a path sewn on another side), she treats the cell as unreachable and stops at the first unsewn border on the straight line, so every side of a piece can still be sewn. Steps along the frame cost nothing here, so the frame route still works; bands and stars are shortcuts, not detours. (Walking to the start of an Ala zhip band you just drew is exempt: you chose that cell.)
5. **Tap she can't reach:** she walks along the straight line toward the tap and stops at the first border on it. The ornament panel opens.
6. **Ala zhip:** the player presses on a reachable cell and drags diagonally (snapped to the nearest of the 4 diagonals). The band covers cells `(x±k, y±k)` and stops where the drag ends, before a 4th new piece, or at the canvas edge (never on the frame). It must cross at least one seam. Each diagonal step becomes a two-way link, and each step can be sewn once. **She takes the band at once:** she walks to its start if she is elsewhere, then glides along it to the far end. If the cell she was heading for is now reachable she carries on there; otherwise she rests at the band's end and that old target is dropped. If she is moving when the band is sewn, she turns onto it at the next cell. Bands may cross other bands and Koshkar seams. **Look:** the band cuts a strip-pieced triangle out of the piece it heads to (the piece of its end cell), like the half-square strip blocks on the pillowcases. That piece is cut along its own corner-to-corner diagonal, slanting the same way as the band, and the half her path runs through becomes the triangle. Its strips run parallel to the cut, in the three fabrics bordering the triangle: the main neighbour along each outer edge, and the piece itself across the cut (undyed fills in for a repeat). Only her path (the diagonal cells) is walkable.
7. **Zhuldyz (same cloth is linked):** the star takes her from the piece she stands on to another piece cut from the **same cloth** (same `fabric`, else same `color`), even one she could already walk to. Hidden pieces and the binding never count. The panel shows Zhuldyz only if some two visible pieces share a cloth.
   - **Trigger:** if she is stopped at a border, picking Zhuldyz uses it at once. Otherwise picking it makes the twin pieces shimmer, with a small silver star where she would land, and tapping a shimmering piece sends her there (landing on the tapped cell). No twin: the star icon nods.
   - **Where she lands:** the cell, in the twin piece nearest the cell you last tapped (or the goal; twins she can't walk to yet come first), that is nearest to that cell. She then carries on toward the cell you tapped.
   - **Portal:** her cell and the landing cell become a two-way portal (one step in pathfinding). Standing on a portal cell and picking Zhuldyz (or tapping her own cell) sends her through it.
   - **Star block:** at both ends, the 3×3 cells round the end (shifted inward at the canvas edge, so the whole star shows) are re-pieced as the pillowcase's **sawtooth star** (`assets/extracted/ornaments/zhuldyz.svg`: a 4×4 grid over the 3 cells, 2×2 centre, background corners, flying-geese sides whose small triangles are the 8 points). Both ends use the same three fabrics, from round where she **leaves**: the centre is her piece, the background the piece bordering the block most (the binding counts here), the points the remaining bordering piece that contrasts most with the background. Every edge between cells of the block becomes a bridge.
8. **The win ends the sewing:** reaching the goal plays the ending by itself: she unravels, her thread is knotted on the goal cell, and the silver scissors that mark the goal cut it, then rise and fade. The panel leaves, taps do nothing, and the save button appears and restart pulses.
9. **Frame (level 01):** the frame is free: stepping onto it from any piece, walking along it, and stepping from it onto any piece all cost nothing. Stitches are needed only between pieces inside the canvas. It is an escape from dead ends, and also the plainest route: start → frame → straight into any piece on the edge, the goal included (changed 2026-11-13; before, stepping off the frame needed a stitch).

## Level 01 — "The quilt" (the only level: Quraq is a mini game)

12×12. Visible owner of each cell (top row is y=11):

```
y11 I I I K K K K K L G G G   <- goal (11,11) in G
y10 I I I K K K K K L G G G
y9  I I I J J J J J L L L L
y8  I I I J J J J J L L L L
y7  D D D J J J J J F F F F
y6  D D P P P E E E F F F F
y5  D D P P P E E E F F F F
y4  D D D E E E E E F F F F
y3  S S S S S S S S S S S S   <- strip
y2  A A A A B B B B C C C C
y1  A A A A B B B B C C C C   (H hidden under B at 5-6,1)
y0  A A A A B B B B C C C C
    ^ start (0,0)
```

- **Wordless tutorial:** the strip S blocks everything to the north, so the first tap upward stops her at a border right away and the panel wiggles. *(Since the frame became free (2026-11-13), a tap on a piece that touches the frame sends her round by the frame instead, so this lesson only works for taps on inner cells.)*
- **All three ornaments:** a short Koshkar route is A → S → E → J → L → G, and a diagonal Ala zhip band can cut it shorter. Zhuldyz links the same cloth: A ↔ J (madder), B ↔ E ↔ I (indigo), C ↔ K (ochre), S ↔ L (undyed). From the start, one star takes her to J. G (saffron), P (arabesque) and D have no twin, so the last step into G from inside the canvas always needs Koshkar or Ala zhip (from the frame it is free).
- **Frame route:** start → frame → G, with no stitch (G touches the frame). Every edge piece is one free walk away.
- **Tests:** a thin strip (S), an overlap patch (P), an L-shaped piece created by an overlap (L under G), and one fully hidden piece (H) for the later lift-corner step.
