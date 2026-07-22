# PMapping — Raw Data Report

**Source repo:** [maamora/PMapping](https://github.com/maamora/PMapping), `data/raw/`
**Scraping method:** Bing Maps local-business search results, extracted with the **BusinessScraper.com** browser extension
**Purpose of the scrape:** building a list of delivery / courier / logistics providers across major Moroccan cities

---

## 1. What's in `data/raw/`

The folder holds 13 CSV files, one per Moroccan city, each named `bing<City>.csv`. Every file is a raw export of a Bing Maps "local pack" search (the extension scrapes the business cards Bing shows for a query like *"livraison" / "delivery"* in a given city).

| File | City | Rows |
|---|---|---|
| `bingcasa.csv` | Casablanca | 129 |
| `bingrabat.csv` | Rabat | 75 |
| `bingKenitra.csv` | Kénitra | 77 |
| `bingMohammedia.csv` | Mohammedia | 17 |
| `bingAgadir.csv` | Agadir | 16 |
| `bingTanger.csv` | Tanger | 15 |
| `bingMarrakesh.csv` | Marrakesh | 14 |
| `bingBeni Mellal.csv` | Béni Mellal | 14 |
| `bingNador.csv` | Nador | 13 |
| `bingOujda.csv` | Oujda | 12 |
| `bingEl Jadida.csv` | El Jadida | 10 |
| `bingFes.csv` | Fès | 10 |
| `bingMeknes.csv` | Meknès | 7 |
| **Total** | **13 cities** | **409 rows** |

A `.gitkeep` placeholder is also present, keeping the folder tracked before real data existed.

Note on the Béni Mellal filename: it contains a literal `#U00e9` instead of the encoded character `é` — a leftover artifact from however the export/URL-encoding was handled when the file was saved, rather than a clean UTF‑8 filename.

---

## 2. Column structure

The CSVs are raw scraper dumps, not a clean schema — column **names and order differ slightly file to file**, because BusinessScraper.com names columns after the CSS/DOM elements it pulled from Bing's page rather than after semantic fields. Reconstructed meaning of the recurring columns:

| Raw column | Actual meaning |
|---|---|
| `l_magTitle` | Business name |
| `b_factrow` | Category (e.g. "Courier & delivery services", "Pizza", "Fast food") |
| `b_factrow 2` | Street address |
| `b_factrow 3` | Separator artifact (`·`) — not real data |
| `opHours` | Closing time snippet (e.g. "· Closes 18:00") |
| `e_green` | Open/closed status ("Open", "Open 24 hours") |
| `nowrap` | Phone number (`+212 …`) |
| `l_rev_pirs` / `l_rev_rc` | Star rating / review count (only present when Bing showed reviews) |
| `rms_img src` | Thumbnail image URL from Bing's CDN |

Because column order isn't fixed, any downstream cleaning script needs to detect columns by content pattern (e.g. "contains `+212`" → phone) rather than by column name/index.

---

## 3. Data quality observations

- **Casablanca file is the least clean**: it has an extra `rms_img src` column and, in 7 rows across the dataset, the category field (`b_factrow`) actually contains a street address instead of a category (e.g. `"22 Boulevard des Almohades, Casablanca"`) — a sign the scraper occasionally lost column alignment on listings with a different card layout (no rating, or an image present/absent).
- **22 rows** across all files have `·` as their category value — the separator character leaked into the category column when Bing's card had no category text to show.
- **Column sets differ per city file** (Fès and Casablanca have an extra image column; Tanger has `opHours` and `nowrap` swapped in order relative to most other files) — confirming this is an unprocessed, per-session export rather than a merged/normalized dataset.
- No unique ID, coordinates, or timestamp column exists — only what Bing's card visibly renders, so re-scraping the same query later could return different results with no way to reconcile them against this snapshot.

---

## 4. What the scrape actually captured

The scrape wasn't restricted purely to courier companies — because it's a Bing Maps local-pack pull, it captured **whatever businesses Bing ranked for the delivery-related query**, which is a mix of dedicated delivery/courier operators, restaurants that offer delivery, and some unrelated local businesses that happened to rank nearby.

Category breakdown across all 409 rows:

| Category | Count |
|---|---|
| Courier & delivery services | 122 |
| Pizza (restaurant, not courier) | 67 |
| Food delivery service | 31 |
| Freight & cargo / forwarding | 33 |
| Fast food | 8 |
| Restaurant | 8 |
| Convenience store / Supermarket | 21 |
| Public service & government | 6 |
| Everything else (30+ scattered categories: clothing, pharmacy, florist, bookstore, etc.) | ~113 |

**By city, delivery/courier-relevant rows vs. other:**

| City | Total rows | Courier & Delivery | Food Delivery Svc | Freight/Cargo | Pizza (restaurant) | Other/unrelated |
|---|---|---|---|---|---|---|
| Casablanca | 129 | 12 | 7 | 9 | 4 | 97 |
| Kénitra | 77 | 27 | 7 | 7 | 18 | 18 |
| Rabat | 75 | 22 | 4 | 11 | 26 | 12 |
| Mohammedia | 17 | 8 | 1 | 2 | 5 | 1 |
| Agadir | 16 | 5 | 2 | 0 | 2 | 7 |
| Tanger | 15 | 10 | 2 | 0 | 3 | 0 |
| Marrakesh | 14 | 9 | 2 | 0 | 3 | 0 |
| Béni Mellal | 14 | 8 | 3 | 0 | 2 | 1 |
| Nador | 13 | 11 | 0 | 1 | 0 | 1 |
| Oujda | 12 | 1 | 1 | 1 | 1 | 8 |
| El Jadida | 10 | 4 | 1 | 2 | 0 | 3 |
| Fès | 10 | 2 | 2 | 0 | 2 | 4 |
| Meknès | 7 | 3 | 1 | 0 | 1 | 2 |

Takeaway: **Casablanca and Oujda are dominated by "Other"** (mostly unrelated local businesses/stores that surfaced in the search), meaning those two files need the heaviest filtering before use, while **Kénitra, Rabat, Tanger, Marrakesh, Béni Mellal and Nador** returned mostly on-topic courier/delivery listings.

---

## 5. Notable recurring brands

A handful of recognizable national/international operators recur across cities, confirming the scrape did surface real delivery infrastructure and not just noise:

- **Pizza Hut** — 21 listings (restaurant delivery, not courier)
- **Jumia** (agencies / relay points) — 11 listings
- **DHL Service Point** — 6 listings
- **Domino's Pizza** — 7 listings
- **McDonald's** — 7 listings
- **Chrono** (Chrono Pizza / Chrono-branded delivery) — 3 listings
- **Glovo** — 1 listing

Most "Courier & delivery services" entries, however, are **local/independent operators** with informal names (e.g. *"Livreur Agadir Livraison Agadir"*, *"One delivery nador"*, *"Marrakech Delivery"*), suggesting the last-mile delivery market in these secondary cities is fragmented and dominated by small local providers rather than national chains — DHL and Jumia are the only consistently present formal networks outside Casablanca/Rabat.

---

## 6. Fields useful vs. missing for a delivery-provider directory

**Usable as-is:** business name, category, city/address text, phone number, open/closed status and closing time.

**Missing / not captured by this scraper:**
- Geographic coordinates (lat/long) — nothing here can be plotted on a map without a follow-up geocoding step
- Full opening hours (only the closing time snippet is captured, not full weekly hours)
- Star rating / review count (only present for a subset of rows, mostly Fès/Casablanca where the extra rating columns appear)
- A category taxonomy — categories are Bing's own free-text labels, not a fixed enum, so they need manual grouping (as done in Section 4) before analysis

---

## 7. Suggested next steps

1. **Normalize schema**: map every file to a fixed set of columns (`name, category, address, city, phone, hours_closing, status`) regardless of source column order.
2. **Filter noise**: drop rows whose category is `·` or clearly unrelated to delivery/courier/logistics (especially in the Casablanca and Oujda files).
3. **Geocode addresses** to enable actual mapping (the stated goal of "PMapping").
4. **De-duplicate** cross-city chains (DHL, Jumia, Pizza Hut, Domino's) if the goal is a count of *unique* delivery networks rather than raw listings.
5. Re-scrape Casablanca and Rabat with a more targeted query — their low "on-topic" ratio suggests the search term used there returned too broad a result set compared to the smaller cities.
