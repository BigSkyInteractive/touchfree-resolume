# TouchFree to Resolume: a working document

We build [TouchFree](https://bigskyinteractive.com), a camera-based body
tracker. It streams a person's skeleton over OSC at about 30 updates a second.

We want that to drive Resolume Arena and Avenue well. **We are not Resolume
experts, and this repository is us saying so in public.** It is a working
document, not a finished integration, and it is open because we would rather
be corrected early than ship something confidently wrong.

If you know Resolume, the things we most need are in
**[QUESTIONS.md](QUESTIONS.md)**. Seven of them, numbered so you can answer one
and ignore the rest.

---

## How to read the claims in here

Everything in this repository is tagged, because the two halves of this problem
are known to very different standards.

| Tag | Means |
|---|---|
| **Verified** | We read it in our own source, or measured it. Trust it, and tell us if it is wrong anyway |
| **Documented** | Quoted from Resolume's own documentation. We have not run it |
| **Assumption** | We guessed. It may be wrong. This is what we are asking about |

The TouchFree half is Verified. Most of the Resolume half is Documented and
some of it is Assumption. Nothing in here has been run against Resolume by us.

---

## What we are trying to build

Five installation gestures, in plain language:

1. Raise a hand, an effect intensifies
2. Walk across the room, a clip scrubs or a layer moves
3. Step closer, the visuals sharpen or brighten
4. Spread your arms, the composition scales
5. Someone walks up, a layer fades in

None of these are exotic. We expect a Resolume operator has done all five with
other input sources and knows the sane way to wire them.

---

## The problem as we understand it

**Documented.** Resolume's OSC input listens on Resolume's own fixed addresses.
Its documentation says "The addresses are all fixed and set up already", and we
found no learn mode for an incoming address. So `/touchfree/2d/right_wrist/y`
appears to have nowhere to go in Arena.

**If that is wrong, please say so first**, because everything below is built on
it and a simpler route would be very welcome.

**Documented.** Resolume Wire looks like the bridge. Its `OSC In` node feeding
a `Read OSC` node takes an address you type in, and Resolume's documentation
says a patch loaded in a host "will use the host and its settings for
communicating".

**Assumption, and the one we would most like checked.** We believe a *compiled*
Wire patch runs in Arena and Avenue with no Wire licence and no watermark,
which is what would make this free for anyone downloading it. Our source for
that is a forum summary we could not open directly. If it is wrong, this whole
approach costs every user EUR 399 and we should be doing something else.

---

## What TouchFree sends

**Verified.** All of this was read out of our sender source, and
[addresses.json](addresses.json) is generated from it so it cannot drift.

One float per address, always. Everything is a float, flags included. About 30
updates a second, 25 with hand tracking on, packed into bundles under 1472
bytes so nothing fragments.

| Address | Value |
|---|---|
| `/touchfree/person` | `1.0` while a person is tracked |
| `/touchfree/2d/<joint>/x` `/y` | **already 0 to 1** across the picture, mirrored. `+x` right, **`+y` down** |
| `/touchfree/2d/<joint>/vis` | tracker confidence, 0 to 1 |
| `/touchfree/hand/left/<0-20>/x` `/y` | 21 hand points, 0 to 1, mirrored |
| `/touchfree/hand/right/<0-20>/x` `/y` | the same for the other hand |
| `/touchfree/3d/valid` | `1.0` when the metric solve is good |
| `/touchfree/3d/position/x` `/y` `/z` | hip midpoint, **metres** |
| `/touchfree/3d/rotation/x` `/y` `/z` | body orientation, **degrees** |
| `/touchfree/3d/distance` | **metres** from the camera |
| `/touchfree/3d/<joint>/x` `/y` `/z` | per-joint position, **metres** |
| `/touchfree/3d/<joint>/vis` | confidence, 0 to 1 |

Two things that catch people:

**The 2D family is already normalised.** It is not pixels. If you have read
anywhere that TouchFree sends screen pixels, that was us being wrong in a draft.

**`+y` runs down**, screen convention. Invert it whenever up should mean more.

**A joint the camera cannot see is not sent.** Its `x`, `y`, `z` and `vis` all
stop together and your receiver holds its last value. So watch for an address
going quiet, not for a flag: `vis` stops too. Gate on `/touchfree/person`.

The 33 joint names are in [addresses.json](addresses.json). The first 17 are
the standard COCO set.

---

## Where we have got to

- **[QUESTIONS.md](QUESTIONS.md)** is the ask. Start there.
- **[MAPPINGS.md](MAPPINGS.md)** is our first attempt at the five mappings,
  written as arithmetic rather than as nodes. **The numbers in it are
  uncalibrated and were chosen at a desk**, not measured in a room. Treat them
  as a shape to argue with.
- **[addresses.json](addresses.json)** is the address reference, generated from
  our source.

There are no Wire patches yet. We did not want to build them on top of
assumptions we had not checked.

## How to help

Open an issue. Answering a single numbered question in QUESTIONS.md is worth
more to us than a full solution, and "you have this backwards" is worth most of
all. If you have built something like this and want to say how you did it, that
is exactly the thing we are missing.

We will keep this document updated as answers come in, and mark what changed.

---

MIT, see [LICENSE](LICENSE). TouchFree is made by
[Big Sky Interactive](https://bigskyinteractive.com).

Our other public repositories, which document the same wire from other angles:
[touchfree-receiver-kit](https://github.com/BigSkyInteractive/touchfree-receiver-kit),
[touchfree-fluid-body](https://github.com/BigSkyInteractive/touchfree-fluid-body),
[touchfree-puppet-2d](https://github.com/BigSkyInteractive/touchfree-puppet-2d).
