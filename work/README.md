# Work tile screenshots

Drop a file here and the matching tile on the homepage picks it up
automatically — no code change needed. Until a file exists the tile falls
back to a designed placeholder, so a missing screenshot never renders as a
broken image.

| File              | Client              | Live site               |
|-------------------|---------------------|-------------------------|
| `armvet.jpg`      | ArmVet LLC          | armvet-llc.com          |
| `eclaire.jpg`     | Éclaire             | eclaire.store           |
| `blackstar.jpg`   | Blackstar The Brand | blackstarthebrand.com   |
| `tracytha.jpg`    | Tracy Tha Barber    | tracythabarber.com      |

## Capture settings

- **Aspect ratio 16:10** — tiles crop to this from the top, so frame the
  hero. Anything below the fold gets cropped off.
- **1600×1000** is plenty. The tile renders at ~420px wide, so this covers
  2x displays with room to spare.
- **JPEG, quality ~80.** Keep each file under ~180KB; they are lazy-loaded
  but four of them still add up.
- Capture at a **desktop viewport (1440 wide)**, not mobile — a phone-width
  screenshot letterboxes badly in a 16:10 tile.

To add a fifth or sixth client, copy one `<a class="work">` block in
`index.html`, change the href, name, slug and the two-letter `data-initial`.

## Note on the console until files land

Each tile requests its screenshot up front, so while these files are missing
the browser console logs four 404s and the fallback ground shows instead.
That is the tradeoff for a zero-edit handoff: drop the four files in and the
tiles fill and the 404s stop, with no change to `index.html`.
