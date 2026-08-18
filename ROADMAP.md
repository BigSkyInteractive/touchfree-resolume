# Roadmap

## What is here now

- `README.md`, the setup guide and address reference
- `MAPPINGS.md`, the five mappings written as maths, buildable in any environment
- `addresses.json`, the same address reference machine-readable, generated from
  TouchFree's source so it cannot drift

That is a specification. Everything in it has been checked against the sender,
and none of it has been run against Resolume.

## What is missing

**The compiled Wire patches.** They are the point of the repository and they
are not built yet.

Plain Resolume cannot receive `/touchfree/*` at all: its OSC input listens only
on Resolume's own fixed addresses, with no learn mode. Wire's `OSC In` into
`Read OSC` reads an address you type, which is what closes the gap. A compiled
patch then runs in any Arena or Avenue with no Wire licence and no watermark,
so nobody downloading this needs to buy anything.

Building them needs, in order:

1. A Wire licence, to compile. Authoring runs on the trial.
2. Confirmation that Wire has range remap, clamp, and a vector length node.
   `MAPPINGS.md` assumes all three.
3. Confirmation of the `Write OSC` loopback into Arena's own parameters.
   Mappings 1 and 3 can live entirely inside the patch; 2, 4 and 5 move
   Arena's own controls and need the loopback.
4. A camera, a room and a screen, to set the ranges. The numbers in
   `MAPPINGS.md` are starting points, not measurements. Where "hand up" begins
   and what counts as arms wide are decided by watching them.
5. Finding out whether a compiled patch survives a Resolume version bump.

Steps 2 and 3 are an hour. Step 4 is the real work.

## If you are not waiting for us

`MAPPINGS.md` is complete enough to build these yourself today, in Wire or in
anything else that speaks OSC and can do arithmetic. If you do, and something
in the specification is wrong, an issue saying so is worth more to us than a
patch.
