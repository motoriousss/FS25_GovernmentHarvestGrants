# Government Harvest Grants — Community Test Script (v1.0.0.0)

Thanks for helping test this mod! It adds a "Government Harvest Grants" tab
to the pause menu: each in-game year the government picks 2-3 crops and
pays a bonus on top of market price when you give them crop from your silo.

This document is split into two parts:
1. **Functional tests** — does it do what it's supposed to do?
2. **Try to break it** — edge cases and abuse that might reveal bugs.

You don't need to do these in order, and you don't need to do all of them
solo — if a few of you split up sections that's great. Tick off what you
tried, note anything that didn't match the expected result, and see
"Reporting a bug" at the bottom for what to include.

---

## 0. Setup

- [ ] Install the mod, start (or load) a savegame with at least a few
      fields and a silo/storage building.
- [ ] Check the mod shows up correctly in the mod selection screen with its
      icon intact (not a broken/placeholder image).
- [ ] If your map doesn't have money to spare, use the in-game money cheat
      (`~` console, or Settings if enabled) so you're not blocked from
      testing large give-amounts by a low balance.
- [ ] Note your map name and mod list (or at least "vanilla map, no other
      mods") in case something turns out to be map/mod-specific.

---

## 1. Functional tests

### Year-start popup
- [ ] At the start of a new farming year (period 1), a popup appears
      listing 2-3 crops.
- [ ] At least one of the crops is always a staple grain (wheat, barley,
      canola, oat, maize, soybean, sunflower, or sorghum).
- [ ] The other 1-2 crops are from the specialty list (potato, sugar beet,
      carrot, parsnip, green bean, pea, spinach, grape, olive, cotton, rice)
      — only ones that actually exist on your map.
- [ ] Each crop has a plausible one-line reason under it (not garbled text,
      not a repeated `%s`, no missing crop name).
- [ ] Over a few years, you should see a mix of generic reasons (e.g. "a
      disease outbreak reduced last year's harvest...") and crop-specific
      ones (e.g. wheat/grapes/potatoes etc. each have their own tailored,
      sometimes humorous reasons) — it's roughly a coin flip each time, so
      don't worry if you see the same style twice in a row, just flag it if
      it's *always* one or the other after many years.
- [ ] The grant terms line reads correctly, e.g. "Grant: €X extra per tonne
      (Y% above market price). Reach Z t given to the scheme this year for
      a one-off bonus of €W."
- [ ] Percentage shown is 20-35% for staples, 30-50% for specialty crops.
- [ ] One-off bonus is roughly 50,000-80,000 for staples, 80,000-120,000
      for specialty crops (some rounding is fine).

### The menu tab
- [ ] Open the pause menu (Esc) — a "Government Harvest Grants" tab
      appears, with an icon, next to Statistics.
- [ ] The intro text at the top matches what the popup said (crop names +
      reasons).
- [ ] The table lists exactly the crops from the popup, no extras, no
      duplicates.
- [ ] For each crop: Stock (t) matches what you'd expect from your silo
      contents; Max Price/t roughly matches that crop's **MAX PRICE** on
      the game's own Prices page (Esc → Prices tab); Bonus/t looks like the
      advertised percentage of that max price; Bonus Target and Target
      Bonus match the popup.
- [ ] Potential Earnings column updates sensibly (should be roughly
      `stock × (max price + bonus)`, plus the threshold bonus if your
      current stock alone would cross the target).

### Giving crop to the scheme
- [ ] Click a row — "Selected: [crop] (X t in silo)" text updates at the
      bottom, and the amount box pre-fills with your full stock number.
- [ ] Type a number directly into the amount box — it should accept typed
      input and let you edit/clear it freely.
- [ ] Click **Give All** with stock > 0 — money is added to your balance,
      a confirmation popup shows the tonnes/crop/amount paid, and the
      Stock (t) column drops to (or near) 0 afterwards.
- [ ] Type a smaller custom number into the box and click **Claim Amount**
      — only that amount leaves your silo, and the payout matches
      `amount × (max price + bonus)`.
- [ ] Press Enter after typing an amount instead of clicking Claim Amount
      — same result as clicking the button.
- [ ] Give enough of a crop across one or more transactions to cross its
      Bonus Target — the one-off Target Bonus is added on top exactly
      once, and a `*` marker appears next to Target Bonus in the table
      afterwards.
- [ ] Give slightly more again after already crossing the threshold — you
      are **not** paid the Target Bonus a second time.
- [ ] Selling the same crop normally at a real selling station (not through
      the mod) works exactly as vanilla — no grant bonus is applied there,
      and it doesn't interfere with the mod's tracked stock/threshold
      numbers.

### Persistence
- [ ] Save the game, fully quit, and reload — the same crops, reasons,
      tonnes-given-so-far, and threshold status are all exactly as you
      left them.
- [ ] Let (or force, via console date/time cheats if you have them) a new
      year start — a new popup appears with a new crop selection; previous
      year's crops are no longer in the table.

### Console command fallback
- [ ] `hgShowGrants` prints/opens the same info as the menu tab.
- [ ] `hgGiveCrop WHEAT 50` (swap in a crop actually in your scheme) opens
      a confirmation dialog with the right numbers, and completing it
      matches doing the same thing from the menu.
- [ ] `hgGiveAll BARLEY` (or whichever crop) behaves like the Give All
      button.

### Multiplayer (if you can test with a friend)
- [ ] Host + one client: the host can open the menu tab and give crop
      normally.
- [ ] The client's balance updates correctly after the host (or the client
      themselves, if on the same farm) gives crop.
- [ ] Note: the year-start popup and live menu data are currently only
      guaranteed correct for whoever is hosting / for singleplayer — if
      you're testing as a remote client, that's specifically worth
      reporting on either way (works fine / doesn't show / shows wrong
      numbers).

### Language (if you play in German, French, Spanish, or Italian)
- [ ] The mod title and description show correctly in your language in the
      mod selection screen.
- [ ] Note: only the title/description are translated so far — the in-game
      menu tab, popup, and dialog text are English-only for now regardless
      of game language. That's expected, not a bug, but worth confirming
      it doesn't look broken/mixed in an odd way.

---

## 2. Try to break it

This is the important part — please be creative and mean to it.

- [ ] Type `0`, a negative number, or letters/symbols into the amount box,
      then hit Claim Amount.
- [ ] Clear the amount box completely and hit Claim Amount.
- [ ] Type a number way bigger than your stock (e.g. `999999`) and claim —
      it should clamp to what you actually have, not go negative or crash.
- [ ] Type a decimal like `12.345` — see what it actually gives/pays.
- [ ] Hit **Give All** or **Claim Amount** rapidly multiple times in a row
      (double/triple click) — make sure you're not paid twice for the same
      stock.
- [ ] Try giving crop with 0 stock of that crop.
- [ ] Select a crop, then quickly reselect a different one before the
      amount box updates — make sure the wrong crop doesn't get charged.
- [ ] Open the Government Harvest Grants tab with **no** active grant crops
      (shouldn't normally happen, but if a map has none of the possible
      crops at all, check it fails gracefully with a message rather than
      an empty/broken screen).
- [ ] Leave the menu open, switch to another pause-menu tab and back —
      table should still be correct, no leftover/stale data.
- [ ] Try it on a farm with **multiple players / multiple people managing
      the same farm's silos** if you can — give crop from two different
      client machines back-to-back.
- [ ] Try it on a very small/early map with few crop types available —
      confirm it still always finds at least a staple, and doesn't error
      out if fewer than 2 specialty crops exist.
- [ ] Try changing economic difficulty (new save) — thresholds should stay
      the same (50k-120k range), but the per-tonne bonus should scale with
      the difficulty's price multiplier.
- [ ] Check `log.txt` (Documents\My Games\FarmingSimulator2025\log.txt)
      after a play session for anything mentioning `GovernmentHarvestGrants`,
      `GrantSystem`, or `InGameMenuGovernmentHarvestGrants` — even
      warnings, not just hard errors.
- [ ] Uninstall mid-save, reload without it, then reinstall and reload
      again — savegame shouldn't corrupt (some data loss is expected/fine
      since the mod isn't active in between, just shouldn't crash).
- [ ] If you're upgrading from an older test build (pre-rename), confirm it
      just starts a fresh grant year rather than erroring or duplicating
      crops — losing last build's progress is expected, crashing isn't.

---

## Reporting a bug

Please include:
1. What you did, step by step.
2. What you expected to happen vs. what actually happened.
3. A screenshot if it's visual.
4. Any relevant lines from `log.txt` (search for "GovernmentHarvestGrants").
5. Your map name and full mod list.
6. Singleplayer or multiplayer (and if MP, host or client).
7. Please send the above information on the discussions tab on my github.

Thanks again for testing — this is genuinely useful and every report,
including "couldn't break this part," helps.
