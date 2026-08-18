# TouchFree to Resolume

Drive Resolume Arena or Avenue from body movement. TouchFree tracks the person and
streams the skeleton over OSC; a Resolume Wire patch turns that into parameter
movement.

Every TouchFree address in this document was read from the sender source, not from
memory. Resolume behaviour is from Resolume's own documentation.

---

## Read this first: why a Wire patch, and not just OSC

Resolume's OSC input listens **only on Resolume's own fixed addresses**. There is no
learn mode and no way to point an incoming address at a parameter. From Resolume's
documentation: "The addresses are all fixed and set up already."

So sending `/touchfree/2d/right_wrist/y` straight at Arena does nothing at all. It is
not a range problem or a units problem. Arena has nowhere to put it.

**Resolume Wire is the piece that closes the gap.** Its `OSC In` node feeding a
`Read OSC` node takes an address you type in, so it can read TouchFree's stream
directly. And a patch running inside Arena uses the host's own OSC settings: "When you
have your patch loaded in a host like Arena or Avenue, the Midi and OSC inputs and
outputs you have in the patch will use the host and its settings for communicating."

**A compiled Wire patch runs in Arena and Avenue with no Wire licence and no
watermark.** You only need Wire if you want to open the patch and change it.

---

## Status of this repository

| Piece | State |
|---|---|
| This guide, and the address reference below | Complete, checked against the TouchFree sender source |
| Compiled `.wire` patches for the five mappings | **Not yet published** |

The patches are being built and tested against real hardware before release. Until
they land, this document is enough to build your own in Wire, and the mapping
sections below spell out the maths each one needs.

---

## What you need

- Resolume Arena or Avenue 7
- TouchFree Desktop, with a camera working and Camera Setup showing green
- Both on the same machine, or on the same network
- Resolume Wire only if you want to author or edit patches

---

## Step 1: Turn on OSC input in Resolume

Preferences, then the OSC tab. Enable OSC Input and note the port.

Keep the message window in that panel open while you set up. It shows the last 200
messages received and it is the fastest way to tell a network problem from a mapping
problem.

## Step 2: Point TouchFree at Resolume

In TouchFree, open **Output**, switch on **Body Data Stream (OSC)** and set:

- Destination: `127.0.0.1` on one machine, otherwise Resolume's IP address
- Port: whatever you set in Resolume
- **2D screen positions** on for anything driven by where a joint is on screen
- **3D metric positions** on for anything driven by real distance in metres

There is no network discovery in TouchFree. Type the address.

On the **Control** page, **Body landmarks** must be on or nothing is sent at all. The
`/touchfree/3d/*` family additionally needs **Body 3D, metric** on; while it is off,
`/touchfree/3d/valid` reads 0 and the per-joint 3D addresses are absent.

Stand in front of the camera and watch Resolume's message window. TouchFree addresses
should appear. If they do not, see Troubleshooting.

## Step 3: Load the patch

Drop the compiled patch into your Resolume effects folder and add it to a layer or
clip like any other effect. Its parameters appear in Arena's own interface.

---

## What TouchFree sends

One float per address, always. No message ever carries more than one argument, so
every address is one channel everywhere.

**Everything is a float**, including the flags. Nothing here is an integer.

| Address | Value |
|---|---|
| `/touchfree/person` | `1.0` while a person is tracked, `0.0` otherwise |
| `/touchfree/2d/<joint>/x` `/y` | **0 to 1** across the picture, mirrored. `+x` right, **`+y` down** |
| `/touchfree/2d/<joint>/vis` | tracker confidence, 0 to 1 |
| `/touchfree/hand/left/<0-20>/x` `/y` | 21 hand points, 0 to 1, mirrored |
| `/touchfree/hand/right/<0-20>/x` `/y` | the same for the other hand |
| `/touchfree/3d/valid` | `1.0` when the 3D solve is good. Gate the rest of `/3d` on it |
| `/touchfree/3d/position/x` `/y` `/z` | hip midpoint in **metres** |
| `/touchfree/3d/rotation/x` `/y` `/z` | body orientation in **degrees** |
| `/touchfree/3d/distance` | **metres** from the camera |
| `/touchfree/3d/quality` | solve fit residual in pixels, lower is better |
| `/touchfree/3d/<joint>/x` `/y` `/z` | per-joint position in **metres** |
| `/touchfree/3d/<joint>/vis` | confidence, 0 to 1 |

Note what this means for mapping work: **the 2D family is already normalised 0 to 1.**
It needs no conversion, only a useful range picked out of it. The `/3d` family is in
metres and degrees and does need converting.

### Joint names

The `<joint>` above. 33 of them, the first 17 being the standard COCO set.

```
nose              left_eye          right_eye         left_ear
right_ear         left_shoulder     right_shoulder    left_elbow
right_elbow       left_wrist        right_wrist       left_hip
right_hip         left_knee         right_knee        left_ankle
right_ankle       left_eye_inner    left_eye_outer    right_eye_inner
right_eye_outer   mouth_left        mouth_right       left_pinky
right_pinky       left_index        right_index       left_thumb
right_thumb       left_heel         right_heel        left_foot_index
right_foot_index
```

---

## What Resolume expects

**Floats between 0.0 and 1.0**, mapped across the parameter's real range. Two examples
from Resolume's own documentation:

- `/composition/video/effects/transform/scale` takes 0.0 to 1.0, meaning 0% to 1000%
- `/composition/video/effects/transform/rotationz` takes 0.0 to 1.0, meaning -180 to +180 degrees

So `0.5` is the middle of the parameter, not the number 0.5.

Because Resolume normalises across each parameter's own range, a patch only has to
decide **its own** range. You never need to know a parameter's real units.

**To find an address:** Shortcuts, then Edit OSC, then click the control. The address
appears and can be copied. Do not type one from memory.

**Absolute and relative forms.** Sending the string `"a"` as the first argument sets an
absolute value, `/composition/layers/1/clips/1/video/effects/transform/positionx "a" 320`.
`"+"`, `"-"` and `"*"` add, subtract and multiply against the current value. These are
available inside a Wire patch's `Write OSC` node; TouchFree itself only ever sends a
single float per address.

**Absolute versus selected.** `/composition/layers/1/...` always targets layer 1;
`/composition/selectedlayer/...` follows whatever the operator has selected. Prefer the
absolute form for anything running unattended.

---

## Five mappings

Each one lists the TouchFree address to read, what the patch does with it, and where it
goes in Resolume. Confirm every Resolume address with Shortcuts, Edit OSC before wiring
it.

### Raise a hand to intensify an effect

- **Read** `/touchfree/2d/right_wrist/y`
- **Patch** invert it, because `+y` runs **down**, so a raised hand is a *smaller*
  number. Then remap your chosen band of the frame to 0 to 1 and clamp. Using the whole
  frame height wastes most of the travel on ceiling and floor; a band from roughly head
  height to waist height gives a usable throw.
- **Send** the effect's amount parameter, or drive the effect inside the patch itself

The most legible mapping for an audience, because cause and effect are obvious.

### Walk across the room to scrub a clip

- **Read** `/touchfree/2d/left_hip/x`
- **Patch** remap and clamp. Invert if the direction feels backwards; the stream is
  mirrored, so which way is which depends on how the camera is mounted. This value is
  already 0 to 1 across the camera's view, which is naturally "the room as framed", so
  it often needs nothing but a clamp.
- **Send** `/composition/layers/1/clips/1/transport/position`

The floor becomes a timeline. Strong for installations.

### Step closer to bring visuals into focus

- **Read** `/touchfree/3d/distance`, in metres
- **Patch** remap your working range to 0 to 1 and clamp. A near of 0.5 m and a far of
  4.0 m is a sensible starting point. Invert so closer means sharper.
- **Send** a blur effect's amount, or `/composition/layers/1/video/opacity`

Gate this on `/touchfree/3d/valid`, and remember it needs **Body 3D, metric** on.

### Spread arms to scale the composition

- **Read** `/touchfree/3d/left_wrist/x` `/y` `/z` and `/touchfree/3d/right_wrist/x` `/y` `/z`
- **Patch** the Euclidean distance between the two points, then remap and clamp. There
  is no wrist-to-wrist address; the patch computes it.
- **Send** `/composition/video/effects/transform/scale`

**Use the 3D wrists, not the 2D ones.** A 2D spread shrinks as the person walks
backwards, so the same pose would zoom differently at two distances. The 3D wrists are
in metres and hold their meaning at any distance.

Scale runs to 1000%. Cap your output well below 1.0 until you have seen it on the real
screen.

### Presence to opacity

- **Read** `/touchfree/person`
- **Patch** nothing, it is already 0.0 or 1.0. Ease it, or the layer will snap.
- **Send** `/composition/layers/2/video/opacity`

A layer fades up when somebody walks over. The attract-loop handoff, done in Resolume.

---

## Known limits, read before building a show on this

**A joint the camera cannot see stops sending, it does not send a guess.** The pose
model returns all 33 points every frame and estimates the ones outside the picture, but
those estimates are withheld rather than published. When someone's hand leaves the top
of frame, `/touchfree/2d/right_wrist/x`, `/y` and `/vis` all go quiet together, and your
receiver holds the last value it got. That is the OSC convention and it is what you
want: an estimate of an unseen joint moves 6 to 8 times as far per frame as a tracked
one, and driving a parameter from it looks broken.

Two consequences for a patch:

- **Watch for silence, not for a flag.** `vis` stops arriving along with the position,
  so it cannot tell you the joint has gone. Use a timeout on the address itself, and
  gate the whole mapping on `/touchfree/person`.
- **Values stay inside 0 to 1**, because a landmark outside the picture is exactly what
  gets withheld. Clamp anyway; it costs nothing.

This matters most for hand height and arm spread, where joints leave frame often.
Distance is unaffected, being gated by `/touchfree/3d/valid`.

Measured on a real take, a person framed from the chest up had 16 of 33 landmarks
withheld, both wrists among them.

> Fixed 2026-08-17. Builds before that published the estimates on OSC, with screen
> fractions running past 1.0. If you are on an older build, clamp in the patch and gate
> each joint on its own `vis`, which did keep arriving there.

**`+y` runs down** on the 2D family. Screen convention, not a mistake. Invert it
whenever "up" should mean "more".

**Body rotation can flip.** `/touchfree/3d/rotation/*` is solved from a nearly flat set
of points and can occasionally jump to the mirror-image answer, reading as if the person
spun around. Position and distance do not have this problem. Prefer them.

**Smoothing.** TouchFree filters its landmarks before sending, so the stream is already
calm. Resolume applies no filtering of its own. If a mapping still looks nervous, map it
to a slow parameter such as opacity or scale before trying a fast one such as position
or playhead, and narrow the range you are driving rather than fighting the data.

---

## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| Nothing in Resolume's message window | Firewall blocking UDP | Allow inbound UDP on the OSC port |
| Nothing, both apps on one machine | Wrong destination | Use `127.0.0.1`, not the LAN address |
| Nothing, and the port is right | Body landmarks off | Control page, switch **Body landmarks** on |
| 2D addresses arrive, 3D ones do not | Body 3D off | Control page, switch **Body 3D, metric** on |
| Messages arrive, nothing moves | The address is not a Resolume address | Arena only acts on its own addresses. A Wire patch has to read TouchFree's and write Resolume's |
| Parameter pinned at maximum | Metres or degrees sent into a 0 to 1 parameter | Remap and clamp in the patch |
| A mapping freezes when someone reaches off screen | The joint left the frame, so its address stopped | Working as intended. Gate on `/touchfree/person` and ease toward a rest value on a timeout |
| Works, then stops | Tracking lost the person | Check Camera Setup |

If messages appear in Resolume's window, the network is fine and the problem is the
address or the range. If they do not, it is the network or the firewall.

---

## Licence

MIT. See [LICENSE](LICENSE).

TouchFree is made by [Big Sky Interactive](https://bigskyinteractive.com). This
repository documents only what arrives on the wire and how to consume it.

Related: [touchfree-receiver-kit](https://github.com/BigSkyInteractive/touchfree-receiver-kit),
[touchfree-fluid-body](https://github.com/BigSkyInteractive/touchfree-fluid-body),
[touchfree-puppet-2d](https://github.com/BigSkyInteractive/touchfree-puppet-2d).
