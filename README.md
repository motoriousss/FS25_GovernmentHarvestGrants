# Government Harvest Grants (FS25)

A government crop grant scheme with a real in-game menu tab. Each in-game
year, the Ministry of Agriculture picks 2-3 crops in short supply - always
at least one staple grain, plus one or two rarer/late-game crops - and pays
your farm a bonus for supplying them.

## What it does

- **A per-tonne bonus** on top of the crop's *yearly peak market price*
  (the same "MAX PRICE" figure the base game's own Prices page shows).
  Staple grains get a random **20-35%** bonus, rarer/late-game crops get
  **30-50%**, fixed for that crop for the whole year.
- **A one-off threshold bonus** once you've given enough tonnes of that
  crop to the scheme in the year: randomised between **50,000-80,000** for
  staple grains, **80,000-120,000** for rarer/late-game crops. The tonnage
  needed to earn it is scaled per crop (sugar beet needs ~400t, grapes only
  ~30t).
- **A year-start popup** explaining which crops were picked and why, with a
  short in-character reason (disease outbreak, import crisis, factory
  shortage, or something more tongue-in-cheek) - see `src/CropPool.lua` for
  the full list.
- **A "Government Harvest Grants" tab** in the pause menu showing, per active
  crop: your current silo stock, max price, bonus/t, bonus target, target
  bonus, and potential earnings.
- **Give crop straight from your silo**: select a crop, type a tonnage (or
  hit Give All), hit Claim Amount (or press Enter) - it's removed from your
  silo and you're paid market price + bonus immediately, no need to drive
  to a selling station.
- **Console command fallback**: `hgShowGrants`, `hgGiveCrop <crop> <t>`,
  `hgGiveAll <crop>` all work independently of the menu.

## Crop pool

**Staples** (at least one guaranteed every year): wheat, barley, canola,
oat, maize, soybean, sunflower, sorghum.

**Specialty/rarer** (1-2 chosen alongside the staple, only from crops that
actually exist on the loaded map): potato, sugar beet, carrot, parsnip,
green bean, pea, spinach, grape, olive, cotton, rice.

## Installation

1. Download the latest `FS25_GovernmentHarvestGrants.zip` release zip file.
2. Put the zip in `...\Documents\My Games\FarmingSimulator2025\mods\`.
3. Enable it from the in-game mod selection screen when starting/loading a
   savegame.

## File guide

| File | What it does |
|---|---|
| `modDesc.xml` | Mod metadata, description (multi-language), source file list |
| `languages/l10n_en.xml` | All in-game text (menu labels, dialogs, popup wording) |
| `src/CropPool.lua` | Which crops can be picked, bonus %/threshold ranges, all the reason text |
| `src/GrantSystem.lua` | Core logic: pricing, crop selection, silo stock reading/removal, the give-crop transaction, save/load |
| `src/GovernmentHarvestGrants.lua` | Main entry point: menu tab registration, year-start trigger, console commands |
| `src/GrantsDialog.lua` | The InfoDialog/YesNoDialog popups (year-start notice, console-command confirmations) |
| `src/Utils.lua` | Small helpers (rounding, weighted random crop pick) |
| `src/Settings.lua` | `HGSettings.DEBUG_LOG` toggle for extra console/log output |
| `src/events/GiveCropEvent.lua` | Client→server "give crop" request, multiplayer-safe |
| `src/events/GiveCropResultEvent.lua` | Server→other-clients payout broadcast |
| `gui/InGameMenuGovernmentHarvestGrants.xml` / `.lua` | The actual pause-menu tab: layout + table/button logic |
| `gui/guiProfiles.xml` | Custom GUI styling profiles for that tab |
| `icon_GovernmentHarvestGrants.dds` | Mod list icon (512x512, no mipmaps) |
| `images/menuIcon.dds` | Pause-menu tab icon |

## Tweaking

- Crop lists, threshold tonnages, reason text: `src/CropPool.lua`
- Bonus % ranges (staple 20-35%, specialty 30-50%) and threshold bonus
  ranges (staple 50k-80k, specialty 80k-120k): also in `src/CropPool.lua`
- Number of specialty crops per year (currently 1-2, randomised):
  `HGGrantSystem:selectCropsForFarm` in `src/GrantSystem.lua`
- Table columns/layout: `gui/InGameMenuGovernmentHarvestGrants.xml`
- Menu tab position (currently just above Statistics):
  `GovernmentHarvestGrants.injectMenuPage` in `src/GovernmentHarvestGrants.lua`

## Multiplayer note

Crop selection, pricing, and the give-crop transaction are all server-
authoritative and safe in multiplayer. The year-start popup and the menu
tab's live data reflect whichever machine opens them (correct for
singleplayer and the host in a listen server); a remote client on a
dedicated server can still use the console commands themselves, but won't
automatically see the host's year-start popup.

## If something looks off

- Set `HGSettings.DEBUG_LOG = true` in `src/Settings.lua` for extra
  diagnostic output (search `log.txt` for `GovernmentHarvestGrants`).
- Check `log.txt` (`Documents\My Games\FarmingSimulator2025\log.txt`) for
  anything mentioning `GovernmentHarvestGrants` or
  `InGameMenuGovernmentHarvestGrants` after a play session.
- A community test script (functional checklist + "try to break it" cases)
  is available separately if you want a structured way to have others test
  it - just ask.

---

## Change history

A rough log of what's changed since the first version, in case you're
comparing notes with testers on an older build.

1. **v0.1 - Popup-only concept.** Year-start popup naming 2-3 crops with a
   per-tonne bonus and a threshold bonus; claim via sell-then-collect.

2. **Real silo transaction.** Replaced "sell normally, then claim" with a
   direct transaction: give crop straight from your silo, paid instantly
   (market price + bonus), using the real FS25 storage API to read and
   remove stock.

3. **Unit bug fix.** An early version converted litres to tonnes using
   each fill type's real density (`massPerLiter`), which produced wildly
   inflated numbers (millions per tonne) because that field is stored in
   tonnes-per-litre internally, not kg-per-litre. Fixed by switching to
   the same "1000 litres = 1 unit" convention the game's own Prices page
   uses everywhere.

4. **Pricing accuracy fix.** Even with units fixed, the mod was using the
   *live current* best station price, which reads lower than the Prices
   page's "MAX PRICE" column most of the time (that column is the
   theoretical peak across the crop's full yearly seasonal price curve).
   Fixed by replicating the game's actual calculation: the highest of the
   12 seasonal price multipliers, scaled by economic difficulty and the
   best station-specific price premium available.

5. **Threshold bonus fix.** Was a flat 50k/80k that got tripled by the
   save's economic difficulty multiplier (unintended). Fixed to a
   randomised 50k-80k (staple) / 80k-120k (specialty) range with no
   difficulty scaling.

6. **Real in-game menu tab.** Replaced the original dialog-only interface
   with an actual "Harvest Grants" tab in the pause menu (table of crops,
   stock, prices, bonuses), including a working free-text amount input box
   (element type `TextInput`, not the `TextInputElement` first assumed).

7. **More varied reasons.** Expanded from 5 generic reasons per category to
   10, and added 3 crop-specific reasons for all 19 possible crops, with a
   50/50 chance of a generic vs. crop-specific reason each time a crop is
   picked.

8. **Icon fixes.** Mod list icon resized to the required 512x512 with
   mipmaps stripped (both were flagged by the GIANTS test runner).

9. **Renamed to Government Harvest Grants.** Mod name, main files, class
   names, GUI page ID, and all user-facing text updated throughout.
   Console commands (`hgShowGrants`/`hgGiveCrop`/`hgGiveAll`) and internal
   `HG`-prefixed helper class names were deliberately left as-is.
   **Note:** this changed the savegame data key, so saves from before the
   rename will start with a fresh grant year rather than carrying over
   progress - not a crash risk, just a reset.
