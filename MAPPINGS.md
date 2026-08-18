# The five mappings, precisely

Each mapping is written as maths rather than as nodes, so it can be built in
Resolume Wire, TouchDesigner, Chataigne, Max, or a script. The Wire patches in
this repository, when they land, are these specifications and nothing more.

Notation: `lerp(a, b, t)` is `a + (b - a) * t`. `unlerp(a, b, v)` is
`(v - a) / (b - a)`. `clamp01(v)` bounds to 0 to 1. Every mapping ends in
`clamp01`, without exception.

Every mapping is also gated:

```
if person < 0.5:            hold, or ease to rest
if address silent > 250 ms: hold, or ease to rest
```

`vis` stops arriving at the same moment the position does, so it cannot serve
as the gate. Time the address itself.

---

## 1. Hand height to effect intensity

```
read   y    = /touchfree/2d/right_wrist/y      # 0 at top, 1 at bottom
       near = 0.15                             # top of useful travel
       far  = 0.70                             # bottom of useful travel

out = clamp01(unlerp(far, near, y))            # note the order: inverts
```

`near` and `far` are the band of the picture the gesture actually uses. The
full frame height wastes most of the throw on ceiling and floor. Tune them by
standing where the audience will stand.

`+y` runs down, which is why `unlerp` takes `far` first. Getting this backwards
is the single most common mistake with this stream.

## 2. Left to right position to a layer position or clip playhead

```
read   x = /touchfree/2d/left_hip/x            # already 0..1 across the view

out = clamp01(x)                               # or 1 - x if the sense is wrong
```

The value is already the room as the camera frames it, so this usually needs
nothing but the clamp. The stream is mirrored, so which direction increases
depends on how the camera is mounted. Decide it by walking, not by reasoning.

Use a hip rather than a wrist: hips stay in frame and move with the whole body.

## 3. Distance to opacity or blur

```
read   valid = /touchfree/3d/valid
       d     = /touchfree/3d/distance          # metres

if valid < 0.5: hold
near = 0.5                                     # metres, closest useful
far  = 4.0                                     # metres, furthest useful

out = clamp01(unlerp(far, near, d))            # 1.0 when close, 0.0 when far
```

Reverse the last line if you want far to mean more. Needs **Body 3D, metric**
switched on in TouchFree; without it `valid` stays 0.

This is the only mapping whose range is in real units, and the only one that
genuinely needs configuring, because distance has no natural maximum.

## 4. Arm spread to scale

```
read   lx,ly,lz = /touchfree/3d/left_wrist/x, /y, /z      # metres
       rx,ry,rz = /touchfree/3d/right_wrist/x, /y, /z

spread = sqrt((lx-rx)^2 + (ly-ry)^2 + (lz-rz)^2)          # metres

min = 0.20                                                 # hands together
max = 1.60                                                 # arms wide
out = clamp01(unlerp(min, max, spread))
```

There is no wrist-to-wrist address. This is why the patch computes it.

**Use the 3D wrists, never the 2D pair.** A 2D spread shrinks as the person
walks backwards, so the same pose scales differently at two distances. Metres
hold their meaning at any range.

`max` is a person's armspan, roughly their height. 1.60 m suits an average
adult; measure yours.

If either wrist goes silent, hold. A spread computed from one live wrist and
one stale one grows without the person moving.

## 5. Presence to opacity

```
read   p = /touchfree/person

out = ease(p, tau = 400 ms)                    # 0.0 or 1.0, smoothed
```

The only mapping with no range to set. It arrives as a hard 0 or 1, so ease it
or the layer will snap.

---

## Where these go in Resolume

Confirm every address with Shortcuts, then Edit OSC, then click the control.
Do not trust this list, including where it agrees with you.

| Mapping | Typical destination |
|---|---|
| 1 | the effect's own amount parameter |
| 2 | `/composition/layers/1/clips/1/transport/position` |
| 3 | `/composition/layers/1/video/opacity`, or a blur amount |
| 4 | `/composition/video/effects/transform/scale` |
| 5 | `/composition/layers/2/video/opacity` |

Resolume takes 0.0 to 1.0 across each parameter's own range, which is why every
mapping above ends in `clamp01` and never needs to know a parameter's real
units. Scale in particular runs to 1000%, so cap mapping 4 well below 1.0 until
you have seen it on the real screen.
