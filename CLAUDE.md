# Trouser Finder

When the user says "run a search" (or similar), find smart trousers that fit the spec below,
verify each one on the retailer's live product page, and write the results to the tracker.

- Tracker page: https://claude.ai/artifact/GD8wym6q9pmcGWmo9RyD6P (source: `finder.html`)
- The user never pastes anything. Claude does all searching, reading of size charts and checking.

## The spec (agreed with the user, do not reinterpret)

**What:** normal, versatile smart trousers. Not jeans, not soft/fluffy cotton chinos.

| Point (garment, flat) | Target | Hard rule |
|---|---|---|
| Waist | 16–16.5" | |
| Thigh | 11.5–12.5" | reject < 11" |
| Knee | 8.25–9" | |
| Leg opening | 7–7.75" | reject > 8.25" |
| Front rise | 10–11.5" | |
| Inseam | 70–72 cm (27.5–28.5") | use the product's stated figure; never assume "typical" lengths. If not stated, mark "unverified". |

Reference: user's perfect H&M jeans — waist 16, thigh 11, knee 8.5, hem 6.5, rise 10, inseam 28 (flat, inches).
Body: waist 33, seat 37, thigh 21, knee 15, calf 14.5. Height 5'10".
Size hints (starting points only — measurements decide): Zara EU 38 (length fine) · Next waist 30 (snug) / 32, Regular or Short (no hemming needed) · H&M S · Levi's/Edwin W29 L30.

- **Fabric:** wool preferred; drape matters most. Blends/synthetics fine if smart. Stretch fine, not needed.
- **Colours:** navy, brown, charcoal/dark grey, black ONLY. Never light grey, beige, khaki, green, pink, red, purple, white.
- **Style:** single pleat favoured; flat front OK if thigh passes. Mid rise, well below belly button (higher OK if fit perfect). Break: touching or just above shoe (trainers, Solovair Gibsons, brogues, boots).
- **Budget:** ≤ £110. **New only.** **Location:** Wales, UK.
- **Returns pass:** start online → QR code → drop at Post Office. Free preferred; a small fee is OK (show it).
  Fail: print-your-own label, email-to-request, store-only, high fees.
- **Ranking:** fit → drape/fabric → returns → price.

## Search procedure

1. Search UK retailers (e.g. Spoke, M&S, Next, Zara, Uniqlo, COS, Arket, Massimo Dutti, Charles Tyrwhitt,
   Percival, Reiss/Suitsupply sale, John Lewis, ASOS, Brook Taverner, Hawes & Curtis, Moss, T.M.Lewin).
2. For each candidate, open the live product page. Record: price, colour, fabric %, stock for the
   recommended size AND length, size-chart garment measurements, returns method/fee.
3. Pick the best size by measurements; score every point good / warn / bad against the table.
4. `verdict: "pass"` only if no hard rule fails, colour/fabric/budget/returns pass, and the size is in stock.
   Fails one check narrowly → `verdict: "near"` with a plain `reason`. Otherwise drop it.
5. Write to the artifact db with ArtifactData (batch). Never overwrite `status` or `notes` the user set.

### db shape

`items/<retailer-slug>-<product-slug>`:
```
retailer, name, url, price, wasPrice?, colour, fabric, pleat ("Single pleat" | "Flat front" ...),
size (e.g. "W32 Short"), inStock (bool), verdict ("pass" | "near"), reason?, rank (1 = best),
fit: { waist|thigh|knee|hem|rise|inseam: { value, target, verdict: "good"|"warn"|"bad", note? } },
returns (one line incl. fee), why (one line), flags: ["New", "Price drop £x", "Sold out", ...],
checkedAt (ISO), status (user-owned), notes (user-owned)
```
`meta/lastRun`: `{ at (ISO), checked (pages opened), retailers (string), note? }`

On re-runs: flag new items, price changes, sold-out sizes; keep user `status`/`notes`.

### Retailer access notes (from run 2026-10-03)

- **Zara:** `<product-url>&ajax=true` via curl returns JSON with composition + per-size stock. Browser and size guide blocked (Akamai). Sizes are waist inches (30/31/32/34).
- **M&S:** headless Chromium works; `__NEXT_DATA__` → `props.pageProps.productDetails` has composition and live per-SKU `inventory.quantity`. Inside leg: Short 29″, Regular 31″. No thigh/hem published.
- **Uniqlo:** API works with header `x-fr-clientid: uq.gb.web-spa` (`/uk/api/commerce/v5/en/products/<id>/price-groups/00/details` and `/l2s?withStocks=true`). Size-chart pages are blocked. UK lengths are 32″/34″ (too long).
- **Shopify feeds (`/products.json`)** work for T.M.Lewin, Percival, Kit Blake (stock per variant).
- **Spoke:** Next.js `__NEXT_DATA__` on collection pages has prices; almost everything is over £110.
- **Blocked:** Next, Reiss, H&M, COS, Arket, ASOS, John Lewis, Massimo Dutti, Suitsupply, Hawes & Curtis.
- Under £110 nobody reachable publishes garment thigh/hem, so items go to `near` with that reason unless measurements are found.
