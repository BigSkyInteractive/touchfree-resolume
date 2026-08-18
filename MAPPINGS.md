# Five proposed mappings

**Status: a proposal, not a specification.** The structure below follows from
what TouchFree sends, which is Verified. Every constant is Assumption: we chose
them at a desk, and not one has been tested on a person in a room.

They are written as arithmetic rather than as Wire nodes so that they can be
argued with by anyone, and built in Wire, TouchDesigner, Chataigne, Max or a
script. If the arithmetic is right and only the numbers are wrong, that is a
good outcome and we would like to hear the better numbers.

Notation: `unlerp(a, b, v)` is `(v - a) / (b - a)`. `clamp01(v)` bounds to 0 to
1. Resolume takes 0.0 to 1.0 across each parameter's own range, so every
mapping ends in `clamp01` and none of them needs to know a parameter's real
units.

**The destination addresses in each section are our reading of Resolume's
documentation and are unconfirmed.** See Q5.

---

## Common gating

Applies to all five.

```
if person < 0.5:            hold, or ease to rest
if address silent > T_LOST: hold, or ease to rest
```

`T_LOST` **(Assumption: 250 ms)** is a timeout on the address itself, not a
check on `vis`. A joint the camera loses stops sending its position *and* its
`vis` at the same moment, so `vis` cannot tell you it has gone.

Whether holding is even the right response is worth an opinion. Easing to a
rest value may read better on a big screen than freezing.

---

## 1. Hand height to effect intensity

```
read  y = /touchfree/2d/right_wrist/y        # 0 at the top, 1 at the bottom

out = clamp01(unlerp(Y_LOW, Y_HIGH, y))      # reversed args, so it inverts
```

`Y_HIGH` **(Assumption: 0.15)** is where in the picture a raised hand sits.
`Y_LOW` **(Assumption: 0.70)** is where a lowered one sits. The whole frame
height is the wrong range: most of it is ceiling and floor that the gesture
never reaches, so the effect would barely move.

These two numbers depend entirely on how the camera is framed and where the
audience stands, so they are the clearest example of something we cannot
choose from here.

`+y` runs down, which is why the arguments are reversed. This is the mistake
we expect people to make most often with our stream.

**Destination:** the effect's own amount parameter, or the effect lives inside
the patch and no OSC is sent at all.

## 2. Left to right position to a layer position or clip playhead

```
read  x = /touchfree/2d/left_hip/x           # already 0 to 1 across the view

out = clamp01(x)                             # or 1 - x
```

This one is nearly free: the value is already the room as the camera frames it.

Whether it wants `x` or `1 - x` depends on how the camera is mounted, and our
stream is mirrored, so we would not trust anyone's reasoning about it including
our own. Walk across the room and look.

We use a hip rather than a wrist because hips stay in frame and move with the
whole body.

**Destination (unconfirmed):**
`/composition/layers/1/clips/1/transport/position`

## 3. Distance to opacity or blur

```
read  valid = /touchfree/3d/valid
      d     = /touchfree/3d/distance          # metres

if valid < 0.5: hold
out = clamp01(unlerp(D_FAR, D_NEAR, d))       # 1.0 close, 0.0 far
```

`D_NEAR` **(Assumption: 0.5 m)** and `D_FAR` **(Assumption: 4.0 m)** are the
working depth of the installation. This is the only mapping whose range is in
real units, and the only one that genuinely has to be configured, because
distance has no natural maximum the way a screen fraction does.

Reverse the last line if far should mean more.

Needs **Body 3D, metric** switched on in TouchFree, otherwise `valid` stays 0
and nothing under `/touchfree/3d/` arrives.

**Destination (unconfirmed):** `/composition/layers/1/video/opacity`, or a blur
effect's amount.

## 4. Arm spread to scale

```
read  lx,ly,lz = /touchfree/3d/left_wrist/x, /y, /z     # metres
      rx,ry,rz = /touchfree/3d/right_wrist/x, /y, /z

spread = sqrt((lx-rx)^2 + (ly-ry)^2 + (lz-rz)^2)        # metres
out    = clamp01(unlerp(S_MIN, S_MAX, spread))
```

**There is no wrist-to-wrist address in our stream**, so this has to be
computed at the far end. That is why Q3 asks whether Wire has a distance node.

`S_MIN` **(Assumption: 0.20 m)** is hands together, `S_MAX` **(Assumption:
1.60 m)** is arms wide. A person's armspan is roughly their height, so 1.60 m
is a guess at an average adult and will be wrong for a child.

**Use the 3D wrists, not the 2D pair.** This part is Verified reasoning about
our own data rather than a guess: a 2D spread shrinks as the person walks
backwards, so the same pose would scale differently at two distances. The 3D
wrists are in metres and hold their meaning at any range.

If either wrist goes silent, hold the whole mapping. A spread computed from one
live wrist and one stale one grows while the person stands still.

**Destination (unconfirmed):**
`/composition/video/effects/transform/scale`

We are told this parameter runs to 1000%, so the sensible thing is probably to
cap the output well below 1.0. We do not know what a normal ceiling is here.

## 5. Presence to opacity

```
read  p = /touchfree/person

out = ease(p, tau = T_FADE)                   # p is a hard 0.0 or 1.0
```

`T_FADE` **(Assumption: 400 ms)**. The value arrives as a hard 0 or 1, so
without easing the layer snaps.

The only mapping with no range to calibrate, and the one we are most confident
about. Easing this in Resolume rather than in the patch may well be the better
answer.

**Destination (unconfirmed):** `/composition/layers/2/video/opacity`
