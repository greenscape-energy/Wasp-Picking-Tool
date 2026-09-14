# Greenscape Warehouse Pick List Generator

Turns an EasyPV design and a customer quote into a warehouse pick list and a
Wasp import file. Drag the two PDFs in, check what it read, generate.

**Internal Greenscape tool. Not for distribution outside the company** — it has
our full Wasp item catalogue, stock figures and picking logic built into it.

---

## Which build have you got?

**The current build is `2026-08-27`.**

The **top right of the page** shows the build date, and it is repeated in the footer
along with the date of the Wasp item export behind it. The build date is also printed
on every pick list, so a paper copy says which version produced it.

Every build is called the same thing and looks the same, so **check that stamp before
reporting a fault** — the fix is very often already in a newer build. A copy showing
no stamp at all predates 26 August and should be deleted.

> **This has already caused a real problem.** A pick generated on a pre-19-August
> build went to Wasp with blank Site and Location columns *and no mounting kit at all*
> — no clamps, no rails, no end caps, no splices, no hooks. Keep one copy in
> circulation and make sure it is this one.

## Using it

**From this repo:** open `index.html` above and click **Download raw file**. GitHub
will say the file is too big to display — that is normal, it is 2.4MB.

> **Before switching on GitHub Pages, read this.** A Pages site published from a
> private repo is still reachable by anyone on the internet who has the URL —
> restricting it to repo members needs GitHub Enterprise Cloud. Because this file
> carries our item catalogue and stock figures, a Pages link would put all of it
> in public view. For an internal always-current link, put the file on SharePoint
> or Teams instead, where Microsoft 365 sign-in protects it.

**Offline copy:** it is a single self-contained file — no install, no internet needed,
and it can be emailed. Rename it to something recognisable if you like; the name does
not matter. Everything runs in your own browser. No quote, design or customer detail
is uploaded anywhere.

## What you do

1. **Drop in the two PDFs** — the EasyPV design and the customer quote. It reads the
   customer name, panel count and model, string count, roof layouts, inverter and
   battery from them.
2. **Check what it read.** Every field is editable. It is a reader, not an oracle — if
   a quote is worded unusually it can miss something.
3. **Check the mounting figures.** Mid clamps, end clamps, rail end caps, rail splices
   and rails are always editable and always hold the final quantity that gets picked.
   Clamps are calculated from the layout and the panel orientation; caps, splices and
   rails fall back to an estimate from the rows × panels layout. If no layout is given,
   one is inferred from the panel and string count and written into the layout box so
   you can correct it. This works on **all three roofs**, whether the job came from PDFs
   or was typed in by hand. Type over any figure and yours is kept.
4. **Fill in what the PDFs do not say** — roof hook counts, tile type, whether the
   fuseboard goes indoors or outdoors, EV chargers, Tigo optimisers.
5. **Generate.** You get the pick list on screen, plus two downloads: the pick list as
   a spreadsheet, and the Wasp import file.

## Read the flags

Any line the tool is not certain about is marked. Three kinds:

- **CHECK** on a pick line — a substituted or assumed part. The reason is on the row.
- **NEEDS MANUAL CHECK** — a list at the bottom of things it could not work out,
  usually because an input was blank or a quote said something unexpected.
- **WHAT THE TOOL DECIDED** — a separate, quieter list of your own rules being applied.
  Nothing here needs doing; it is there so no decision is hidden.

Neither of the first two stops you generating. Both mean look before you pick.

It will never invent a Wasp item number. If it cannot find a code for something it
lists the item by description and flags it, rather than guessing.

## How the logic works

`Greenscape_Pick_List_Logic.xlsx` is the full picking logic written out in plain
English — every inverter kit, battery band, roof type, fuseboard ladder and quantity
rule, with a Correct? column. Fifteen sheets:

| Sheet | What it holds |
|---|---|
| Read me | How to review the workbook |
| Rules | The 37 rules the tool follows |
| Check these | 26 judgement calls that still need a decision |
| Quantity rules | All 46 "how many" rules and how they resolve |
| Inverters / Batteries / Roof types | The kit for every model |
| Fuseboards | Way counting and the board ladders |
| Car chargers / Tigo / Additional items | The optional sections |
| Stock locations | Which site and zone every pickable item comes from |
| Warnings | Every warning it can raise |
| Applied | What each review changed |

`Greenscape_Manual_Checks.xlsx` lists all 74 warnings the tool can raise, grouped by
the kind of answer each needs, with what has been settled so far.

If something picks wrong, those workbooks are where to find out why. Mark the row and
send it back.

## Stock locations

Wasp requires a **Site** and **Location** on every check-out line — it is decrementing
one specific pile, so a blank cell fails the whole import. The tool fills both from the
Wasp item export, per item:

- **one place** — written straight onto the line
- **several places** — the one holding the most stock, with the alternatives flagged
- **nowhere** (zero stock) — left out of the import file and flagged, because there is
  nothing to pick
- **non-inventory** (2219 Pick Order Issued) — kept in, borrowing the place most of the
  pick comes from, since Wasp holds no stock for it but still demands the column

Every line also shows its site and location on screen, so the pick can be walked rather
than hunted. Typing a Site and Location on the form overrides all of it.

Built from the **Items export of 21 August 2026** — **re-run the build when stock
moves**, or the zones will drift out of date.

## Mounting — Renusol

Changed over from Fastensol on 27 August:

| Part | Code |
|---|---|
| Mid clamps | 10186 |
| End clamps | 10184 |
| End caps | 2515 |
| Rail splice | 10187 |
| Rail (3.6m) | 10190 |

The single universal-clamp line is now **two lines**, mid and end, calculated from the
layout and the way the panels are laid:

- **two end clamps for every panel at the edge of a run**
- **two mid clamps for every joint between panels along that run**

Two rails per panel is where both twos come from — the EasyPV report's own wind-loading
section states it. Which way a run goes depends on the orientation: **portrait** means a
run is a row, so R runs of N panels; **landscape** means a run is a column, so N runs of
R panels. That gives `mids = 2 × runs × (run length − 1)` and `ends = 4 × runs`. A single
panel in a run takes 4 end clamps and no mids. Every line carries its own arithmetic.

**Orientation is set per roof**, so one portrait roof and one landscape roof on the same
job are each worked out their own way. It is read from the EasyPV report where possible:
the words "portrait" or "landscape" if present, otherwise the installed module height the
wind-loading section states — over 1.5m means the panel is standing up. Our reports never
use either word (their "Orientation" fields are compass degrees from South), so the module
height is what resolves it in practice. Override any roof by hand.

A roof type only gets the lines it actually carries. **Metal Roof keeps the universal
clamp 10201** — the spreadsheet's own note against it says "Not Mids or End Clamps" — and
flat, in-roof and ground mount get none of them.

**Roof hooks:** Pantile 1321 · Slate 10189 · Rosemary 10189. Slate also takes 1901 Genius
flashing and Rosemary 10174 Hookstops, both at the hook quantity. Peg Tiles and Cambrian
Tiles were not specified and keep what the spreadsheet gives them.

## Isolators

- **10015** per inverter up to 6kW, single phase
- **10017** per inverter over 6kW, **and always for three phase** — 10017 is the 4-pole
  unit, so a flat kW rule would have put a single-pole isolator on a three-phase supply
- **10144** one per panel string on the job

This changed the AC isolator on 7 Solis three-phase String inverters (5–20kW) that were
holding a single-pole 10015, and replaced the spreadsheet's fixed DC counts of 1 to 6
with the string count.

## Settled rules

- **Hook uplift** — landscape panels take the ×1.2, portrait do not. Decided by the
  orientation, not a tickbox. This was the oldest open question in the tool and it
  changed hook counts on every job.
- **Due date is mandatory** — the tool will not generate without one.
- **No panels** — reads as an AC coupled job and picks nothing for the array or roof.
- **Fox batteries cap at 4** — more than that is reported as a data-entry error, not
  multiplied into extra junction boxes.
- **One EV supply per install**, regardless of charger count.
- **Tigo kit carries one TAP**; additional arrays get their own.
- **Both fuseboards come from the location's range.**
- **2219 stays in the import** on every pick, borrowing the site and location most of
  the pick comes from.

Settled rules report under **What the tool decided** rather than **Needs manual check**,
so the check list only ever holds things that need a person.

## Known open items

- **Which RCBO goes with a charger.** Two answers on the same review named different
  parts: 2397 as "the only additional part", and 2237 as "correct". Both are RCBOs, so
  applying both would put two protection devices on every charger. 2237 is picked and
  the clash is flagged until it is settled.
- **The 8 way busbar has no Wasp code yet.** It prints by description and stays out of
  the import file until one exists.
- **The Tesla Wall Connector rule cannot fire** — 1898 is not in the charger list.
- **2219 in the import needs a live test.** It is non-inventory, so the site and location
  on its line are borrowed from the rest of the pick. Wasp may still refuse it.
- **12 items are held in more than one place.** The tool takes the one with the most
  stock. Confirm that is the right call for each.
- **Peg Tiles and Cambrian Tiles** hooks were never specified.

The import is **format-confirmed**: Wasp auto-matched every column and read the item
numbers and quantities correctly on 21 August.

## Rebuilding it

`index.html` is a **built artefact**, not source. It is generated by a Python pipeline
from the Wasp picking spreadsheet and the item master. Editing the 2.4MB HTML by hand is
a bad idea and will be overwritten by the next build.

The pipeline (`build.py`, the HTML template, the source spreadsheet, ~33 browser tests)
is **not in this repo**. Ask Dan for it before attempting any change to the logic.

## Data in here

Item codes, descriptions and stock figures for 1,850 Wasp items, and the picking recipe
for every inverter and battery we fit. **Keep the repo private.**
