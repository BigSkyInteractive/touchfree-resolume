# Open questions

Seven things we could not settle from documentation. Answer one and ignore the
rest; there is no need to take the whole list. "You have this backwards" is a
complete and welcome answer.

Each question says what we currently believe and how confident we are, so you
can correct the belief rather than guess at what we meant.

---

## Q1. Can Arena receive an OSC address that is not its own?

**We believe: no.** Resolume's documentation says "The addresses are all fixed
and set up already", and we found no learn mode for an *incoming* address, the
way MIDI learn works. So we think `/touchfree/2d/right_wrist/y` cannot be
pointed at a parameter in plain Arena at all.

**Why it matters:** everything else here exists to work around this. If there
is a route we missed, most of this repository is unnecessary and we would
rather know now.

## Q2. Does a compiled Wire patch run without a Wire licence?

**We believe: yes, and with no watermark.** This is the least verified claim in
the repository and the one the whole approach rests on. Our source is a forum
summary we could not open directly.

**Why it matters:** if it is right, we buy one licence, compile the patches,
and anyone downloading them pays nothing. If it is wrong, every user needs
EUR 399 and we should be solving this a different way entirely.

## Q3. Does Wire have the arithmetic we need?

**We are assuming:** a range remap (map an input range onto 0 to 1), a clamp,
and a vector length or distance between two 3D points.

[MAPPINGS.md](MAPPINGS.md) assumes all three exist and we have not checked any
of them. The distance one is the least ordinary: mapping 4 needs the straight
line distance between two wrists in 3D.

## Q4. Can a Wire patch drive Arena's own parameters?

**We believe:** a patch can use `Write OSC` into `OSC Out` to send back into
Arena's own OSC input, on the same machine.

We have not confirmed the port or address mechanics, or whether Resolume
minds a loopback like that. Two of our five mappings drive an effect that *is*
the patch and need none of this. The other three move Arena's own controls,
which is where the question bites.

**Is there a more direct way?** If a patch can expose a parameter that an
operator links to a layer control without OSC in the middle, that is obviously
better and we would drop the loopback.

## Q5. What are the right addresses for a clip playhead and an effect amount?

We know to find these with Shortcuts, then Edit OSC, then click the control.
We have not been able to do that, so the destination addresses in
[MAPPINGS.md](MAPPINGS.md) are our best reading of the documentation and may
well be wrong.

Also: are those addresses stable across compositions, or do they move when
layers and clips are rearranged? For an unattended installation that matters
more than it does on a live rig.

## Q6. Does a compiled patch survive a Resolume update?

**We do not know.** We saw forum threads about Arena and Wire version
mismatches, which suggests it might need recompiling per release.

**Why it matters:** it is the difference between publishing a patch once and
maintaining one per Resolume version. If it is the latter we would rather say
so in the README than have someone find out during a show.

## Q7. Is Wire the right tool at all?

We arrived at Wire by reading documentation, not by experience. If an
experienced operator would reach for something else, we would like to hear it,
including answers like:

- Use Chataigne or another mapping host between the two
- Do it in TouchDesigner and send Resolume video instead of parameters
- There is a native Resolume feature that does this and you have missed it
- Nobody drives Resolume this way, and here is why

---

## Also useful, if you have opinions

**Calibration.** The ranges in MAPPINGS.md were chosen at a desk and never
tested on a person. If you have wired body tracking to Resolume before, the
values that actually feel right are worth more than our guesses.

**Smoothing.** Our stream is filtered before it leaves, and Resolume applies
none of its own. We do not know whether it wants more easing at the far end
for a large screen, or whether that just adds lag.

**Which parameters are worth driving.** We picked five gestures that sounded
good in a meeting. An operator's list of what actually reads well to an
audience would probably be a better five.
