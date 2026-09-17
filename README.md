# VIC-II-spector

An interactive, cycle-by-cycle map of one VIC-II raster line, for C64 coders.

**→ [elysium64.github.io/vicspector](https://elysium64.github.io/vicspector/)**

Linus Åkesson's timing chart, turned on its side so cycles run left to right the
way the beam does, plus a code planner that tells you what your instructions
really cost once the VIC starts stealing cycles.

Single self-contained HTML file. No build step, no dependencies, no network
access — open it from disk and it works.

## What it shows

Hover or click any cycle column to get the full picture for that cycle:

- **VIC bus access** in both half-cycles — p, s, r, c, g and idle, with the
  address each one forms
- **BA / AEC**, and what the 6510 can still do: free, write-only, or stalled
- **Sprite DMA** lanes, one per sprite, showing the fetch and the three-cycle
  BA lead-in
- **Screen zones** — display window, border and horizontal blanking, tracking
  the 38/40 column switch
- **Internal state** — VC, VCBASE, RC, VMLI, MC, MCBASE, the expansion
  flip-flop and the border flip-flops, on the cycles they actually change
- **Sprite X coordinate** at the start of every cycle

Selectable chip: 6569 (PAL, 63 cycles), 6567R8 and 6567R56A (NTSC, 65 and 64).
The parts share everything up to cycle 55 and differ in the idle cycles and
where sprites 0–2 fetch.

## Code planner

Drop instructions onto the line and see where they really land.

- Each instruction carries its own cycle — drag the pills to place them,
  shift-click to group several and drag them together
- Two lanes: what you'd count on paper, and where it lands once the VIC has
  taken its cycles, with stalls hatched
- Cost is modelled on **read/write**, not cycle count. RDY low halts the 6510
  on reads only; writes complete regardless, which is why the VIC drops BA
  three cycles before AEC follows. R/W patterns are derived from the mnemonic
  (`STA` gets a trailing write, RMW ops two, `JSR` writes on cycles 4–5) and
  can be flipped by hand per cycle
- A running budget: free cycles, what the code spends, what's left — going
  negative when the code outruns the line
- State persists in `localStorage`

## Trick reference

Cycle-anchored notes for FLI, forced bad lines, FPP and 1px char lines, opening
the side borders, sprite crunch and stretch, FLD, DMA delay, linecrunch and
stable raster.

The sprite crunch tab carries the offset graph: give it an origin and it finds
every loop back to it, so you can pick a sprite height and read off the
schedule, with the crunch steps marked.

## Accuracy

Timings follow Christian Bauer's documentation, cross-checked where possible:

- Free-cycle counts match the canonical figures — 63 on a clear line, 23 on a
  bad line, 2 cycles per sprite, 1 free plus 3 write-only on a bad line with
  all eight sprites
- The X-coordinate model reproduces Bauer's lightpen example (LP in cycle 20
  gives `$1e` in LPX, sprite coordinate `$03c`)
- The scheduler reproduces every cycle annotation in Åkesson's MISC listing —
  8 sprites, no bad line: the `nop` at 54 freezing across the sprite DMA and
  resuming at 11, `sty $d017` at 12 with its write landing on cycle 15, then
  16, 20, 24, 26, 32
- The sprite crunch function `Cr(x) = (0x2a & (x & (x+3))) | (0x15 & (x | (x+3)))`
  agrees with Åkesson's `((MC | MCBASE) & 0x15) | ((MC & MCBASE) & 0x2a)` on all
  64 offsets, and the graph reproduces Crossbow's 17-line minimum

Known rough edges: the border compares are pixel-level events shown at
cycle granularity; the cycle-exact border and register timings are documented
for the 6569, so treat the NTSC border cycles as approximate; and the crunch
graph finds 15- and 16-line loops from `$35` that Åkesson doesn't list, which
may reflect a constraint specific to MISC's encoding.

## Sources

- Linus Åkesson, [VIC 6569/8565 timing chart](https://www.linusakesson.net/programming/vic-timing/)
- Linus Åkesson, [MISC — technical notes](https://www.linusakesson.net/scene/lunatico/misc.php) (sprite crunch)
- Christian Bauer, [The MOS 6567/6569 video controller (VIC-II) and its application in the Commodore 64](https://ist.uwaterloo.ca/~schepers/MJK/ascii/VIC-Article.txt), 1996

Built with Claude.
