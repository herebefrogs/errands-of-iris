# unidrill — implementation TODO

Ordered roughly by dependency, not necessarily by priority. See `DESIGN.md`
for the reasoning behind each of these; this is just the sequencing.
Completed items are moved to `CHANGELOG.md` (an archive, not read on startup)
as they land — this list stays scoped to open work.

## Bugs

(none open)

## Playtest / gameplay balancing

(none open)

## Later / revisit

- [ ] Byte-golf pass (near submission, once features are frozen). Mechanical
      only — see the "no premature byte-golfing" memory. Known dead weight to
      clear:
      - `src/js/sound.js` still carries ZzFXM (`zzfxM`, `loadSongs`, `playSong`)
        and the game.js imports `loadSongs, playSong` — unused since music moved
        to voxby/`player.js`. Drop them unless the SFX work (dust collect, tally
        tick, stall, rainbow) ends up wanting a ZzFXM cue. `playSound` (single
        `zzfx`) stays regardless.
      - the debug overlays below.
      - whatever else esbuild-metafile / a bundle diff flags at the time.

- [ ] Delete the debug overlays — the `DEBUG_CAMERA` flag + ring/crosshair
      block and the `DEBUG_POINTER` flag + its branch of the D-pad draw block,
      both at the end of `render()` in game.js. **Only if we're over the 13 KB
      budget at submission time.** As of the camera-tracking commit there's
      ~40–50% headroom, so keep them for now — handy for re-tuning the
      `CAMERA_*` constants and the pointer `RAMP`/`DEAD`. (Keep the plain
      base+knob D-pad overlay — that's the shipped control, not debug.)

## Ideas — not yet designed

Half-formed; each needs a design pass before it becomes a build item.

- [ ] Upgrade picks. Every X dust collected, pause the game and offer 2–3
      upgrade options to choose from (roguelite-style). X, the option pool,
      and what the upgrades do are all TBD. Open: does a mid-dive pause
      break the one-decision-per-second push-your-luck tension, or add a
      welcome second layer of choice? Run-scoped or meta-progression across
      runs?
- [ ] Underground creatures. Would baddies improve the game — worms,
      centipedes, beetles, leprechauns? Completely unformed: brainstorm how
      they weave into the momentum loop (obstacle that costs momentum?
      steals dust? chases you on the ascent? a leprechaun guarding a dense
      patch?) before it's worth prototyping. Risk: the game's pleasure is
      carving your own path — anything that demands twitch dodging could
      fight that.

## Won't do

- Bingo-fuel warning. Was: a HUD alert when the player likely can't climb
  back out (reachable distance ≈ `momentum^2 / (2*(entropy + sandDrag))`,
  warn when that drops below `depth`). Dropped with the objective change —
  the player now wins whether they resurface or not (the run just ends when
  momentum hits 0, wherever they are, and a rainbow sprouts back up the
  tunnel). There's no failed-return-trip to warn about any more; shaft
  length carved feeds the score, not depth. See "Sprout a rainbow on run
  end".

- Rainbow dust — carry penalty. DESIGN.md's old core-loop line said
  "carrying more dust drains momentum faster"; the idea was to scale
  momentum decay by the `dust` count so a greedy deep run is more
  precarious on the return. Prototyped and dropped. What we found:
  - A naive per-cell drag term made the game brutally hard — the terrain
    already decays you continuously, so this was a second, always-on drag
    stacked on top ("double drag").
  - Zeroing SAND drag to make room for it helped, but turned the game into
    a different, more punishing thing and made clay patches near-instant
    death.
  - It actively fought the dense-dust boost: a dense patch sped you up and
    permanently loaded you down in the same pass, so the boost stopped
    reading as a reward.
  - Capping the carry term kept it playable but exposed the real problem —
    a capped penalty is just a one-time tax on your first ~40 dust, after
    which it's the no-penalty game again, so the mechanic isn't even
    load-bearing.
  Fundamentally a carry penalty punishes the player for collecting, when
  the whole game is about wanting them to collect as much as possible. The
  no-penalty version — pure momentum management, drag from terrain only,
  speed from dense dust — is simpler and more intuitive: one source of
  slowdown (terrain), one source of speed (dust). Keeping that.

- ROCK as a third material. Was: a solid, undrillable material (own rare
  blob field, independent of clay) that deflects the drill's heading on
  contact instead of stopping it dead. Prototyped and dropped. What we
  found:
  - The reflection is axis-aligned (probe the blocked cell's two neighbors
    to tell a vertical face from a horizontal one apart) rather than a true
    surface-normal bounce — cheap, but it means only a shallow-angle hit
    naturally deflects clear. Anything close to a head-on hit reflects
    almost straight back the way it came, which is very hard to land on
    purpose with keyboard's 8 fixed directions (more manageable with the
    analog D-pad on mobile) — in practice most hits read as "close to
    normal," so most hits felt like a wall, not a deflection.
  - Tried a momentum penalty on bounce (`*= 0.8`) to make ramming feel
    costly — combined with the near-head-on problem above, a couple of
    bounces in a row (steering re-aiming back into the rock before the
    reflection could carry the drill clear, even after adding a brief
    post-bounce steering lock) drained most of the run's speed. Dropped the
    penalty entirely (pure direction change, no speed cost) as a last try —
    still didn't read as fun, just as an annoyance layered on the aiming
    problem above.
  - Root issue is the deflection geometry, not the tuning: a fair, fun bounce
    needs the reflection to actually feel aimable, which likely means a real
    surface-normal reflection (derived from the blob's shape, not axis
    probes) rather than more constant-tweaking. Not worth the time to chase
    for the payoff — dropping the feature. Revisit only with a genuinely
    different deflection approach in mind, not a retune of this one.
