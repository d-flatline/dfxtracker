# DFX Tracker — test build

A pattern tracker for Android with two views onto one song: a RAD-style VGA
pattern editor and an FL-style step sequencer. Imports and exports ProTracker
`.MOD`, ScreamTracker 3 `.S3M` and FastTracker II `.XM`, and renders to `.WAV`.

This repository exists to hand out a build for testing. **Grab the APK from
[Releases](../../releases).**

## Status

**It runs, and it makes sound.** Confirmed on an Android phone: a ProTracker
`M.K.` module loads through the system picker, plays through Oboe, and the
pattern grid follows it.

That was not a given. The C++ core is heavily tested on a host — 6,161
assertions, a mutation fuzzer over all three loaders, a threading check, and a
byte-exact export round trip for MOD, S3M and XM alike — and the Android layer
compiles clean against the NDK with its JNI surface verified symbol by symbol,
55 exports against 55 declarations. None of that says the app starts, and until
someone ran it, nobody knew.

What the build could not catch was the interface. Two rounds of screenshots
found a grid that spent the whole screen height on five rows, chrome that
wrapped onto four lines, and a stray caret drawn along the top edge — none of
which is a compile error. See the release notes for what changed.

**As of v0.9 your work also keeps.** There is a project format that holds a
song exactly, an autosave that survives the process being killed, and a song
that can be named, timed and told where to loop back to.

**As of v0.13 the editor can type everything the loaders can read.** An S3M's
effect column used to stop at `O` and an XM could not be given an `Fxx` at all,
because a command was taken from a single hex nibble while the formats store up
to thirty-four of them. The pad now offers exactly the commands the song's
format has. XM's volume column — which is a second effect column, not a level —
can be typed in as well.

**As of v0.12 the instruments are yours too.** A note key now *sounds* as you
press it — on the piano pad, a hardware keyboard, or a sequencer pad, with the
transport stopped or running. And an instrument can finally be edited rather
than only imported and played: name, volume, panning, tuning, loop points and
loop mode, plus normalise, reverse and fades. On a phone this is the only way
instruments are reachable at all — the list beside the grid is the first thing
a small screen drops.

**As of v0.11 you can write a song with it.** Blocks can be marked, copied,
pasted, cleared and transposed; a song's channel count can be changed after it
exists; and the app finally asks for the audio output rather than playing over
your phone calls. That was the last of what the roadmap called Phase 1 — the
gate between a module *editor* and a tracker.

**As of v0.10 it reads FastTracker II.** `.XM` loads, plays and saves —
including the parts of XM that are not in any MOD: instruments that are a
keymap over as many as sixteen samples, volume and panning envelopes with a
fadeout (so a note-off *releases* instead of cutting), patterns of different
lengths in one song, and a volume column that is a second effect column rather
than a level. A hardware keyboard works now too, on the usual Z-M / Q-P tracker
layout.

### What has never run on a device

Everything in v0.9 is verified on a host, which is exactly what was true of the
interface bugs above before someone took a photograph. So if you touch any of
these, what happened is worth reporting **even when it worked**:

- saving a project, opening it again, and the recovery prompt after a kill
- the song-properties strip — name, speed, tempo, volume — and the `LOOP`/`ONCE`
  toggle with the `ORD LOOP→` button beside it
- **pressing keys with nothing playing** — every note key should sound the
  moment you press it, and a chord on a hardware keyboard should not cut itself
  off
- **audio focus** — play something, then take a call, then get a notification,
  then pull the headphones out. Those are three different behaviours and only
  the middle one should leave the song audible. None of it can be checked in a
  host build: it is a conversation with the rest of the phone.
- opening an `.xm`, especially a big one: the format is verified against a
  fixture and a fuzzer here, not against a shelf of real songs
- a hardware keyboard, if you have one to pair
- the step sequencer view

## Installing

- **Android 7.0 (API 24) or newer**, `arm64-v8a` or `x86_64`.
- Debug build, signed with the standard Android debug key. Your phone will ask
  you to allow installs from whatever app you downloaded it with.
- No permissions are requested. There is no `INTERNET` permission and no
  storage permission — files are opened through the system picker, so the app
  only ever sees what you hand it.
- The app is **landscape-locked**; a tracker grid needs the width.

## Using it

Open a module with the file button, or tap a `.mod`, `.s3m` or `.xm` in any
file manager and pick DFX Tracker. If you have nothing to hand, anything from
[modarchive.org](https://modarchive.org) will do — and an `.xm` from there is
the most useful thing you could throw at this build.

Or start from nothing: `NEW…` makes a blank MOD or S3M whose first eight
instrument slots hold a synthesised drum kit, so a new song makes a noise as
soon as you type into it.

Working: MOD, S3M and XM import, both editor views, cell editing with undo,
playback, pattern and order editing, block copy/paste/clear/transpose, changing
a song's channel count, WAV sample import, song properties and the loop point,
project save/load with autosave and crash recovery, same-format export, and WAV
render. With a hardware keyboard: the Z-M / Q-P layout, arrows to move,
shift-arrows to mark a block, space to play, and ctrl-Z/Y/C/V.

`INST…` opens the instrument strip for the selected slot. It also reports what
an XM instrument holds that no control here reaches yet: how many samples hang
off its keymap, which envelopes it carries, and its fadeout.

Blocks are marked rather than dragged: `MARK` sets one corner and the cursor is
the other, because the grid's three gestures are already spent — tap places the
cursor, one finger scrolls, two zoom.

Saving a project (`.dfxp`) is lossless and separate from exporting a module,
which is not: export tells you what the format cannot carry — a MOD has no
volume column — *before* it writes, rather than after.

Not built yet: IT import, cross-format conversion, OPL2 synthesis, and the
parts of the sample editor that need a waveform on screen — trimming to a
selection, and dragging loop points. The operations exist and are tested; what
is missing is somewhere to see them. AdLib/OPL
instruments in an S3M are parsed and preserved but **silent** — there is no
OPL2 emulator yet, and the UI says so rather than pretending. XM's
volume-column commands are loaded, played, written and shown, but cannot yet be
typed: the editor's cell has five fields and an XM cell has six.

## If something goes wrong

The useful things to report, roughly in order:

1. **It did not install** — the phone's exact wording.
2. **It installed but died on launch** — `adb logcat` around the crash, or a
   screenshot of the dialog. A failure to load the native library shows up as
   `UnsatisfiedLinkError`.
3. **It runs but the audio is wrong** — crackling, stuttering, silence, or
   wrong notes. Which, and on which module.
4. **It runs and the audio is fine** — then say so, because that is the single
   most informative outcome available right now.
