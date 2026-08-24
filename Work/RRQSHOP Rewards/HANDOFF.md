# RRQSHOP Rewards — Front-End Handoff

**Read this file first. It is the map; everything else is detail.**

Program: **RRQSHOP Rewards** — a cashback/loyalty program for www.rrqtopup.com (RRQSHOP).
Currency unit = **poin**, pegged **1 poin = Rp 1**.
Scope of this repo: customer-facing web, mobile-first, Bahasa Indonesia (MY/PH via i18n).
State as of **2026-08-03**. Design lead = the owner + design sessions; this repo is design + standalone code, **no application code**.

---

## 0a. Decisions locked 2026-08-03 — read these before the sections below

Eight items that were open when the rest of this file was written. All owner-decided. Where a section below repeats one of these, the decision here wins.

| # | Decision | Consequence |
|---|---|---|
| 1 | **C1 CLOSED — the poin balance lives in the sandwich menu's poin block.** No header chip, no tab badge | The 132px header gap stays empty. Do not build a header balance chip. `Sandwich Menu / M` `1036:22248` logged-in variant is the only balance-in-chrome surface. **Wire it to `/balance`** |
| 2 | **T&C (#20) extends the LIVE T&C page** — a Rewards/PRIME section plus the reseller statement. Not a new page | One legal document, one review cycle. No new page shell to design |
| 3 | **Emails: only E2 ships in V1.** E1, E4, E6, E7 are all out | ⚠️ **Redeem has no receipt.** A user who pays fully with poin gets the success page and nothing in their inbox. Order-status + the wallet ledger are the only record. Accepted knowingly |
| 4 | **Clawback (#21): the "rare edge case" ruling extends.** Debt wallet state stays as-is; add exactly one new state — the **checkout blocked-redemption reason** | Ledger reversal and debt-netting rows follow the existing sign convention, no new frames. The one thing that must be built is a reason string when a user with debt meets the redeem toggle |
| 5 | **Desktop wallet `972:9896` balance → `12.000`** | One-node fix. The equivalence line and sidebar cell already read 12.000 and do not change |
| 6 | **New-member payoff = louder delivered sticky banner.** No new surface | Build it as already specified: `Bonus member baru!` copy in the existing sticky slot. No celebration animation, no success-page hero |
| 7 | **Form states: no frames will be drawn.** Dev uses house patterns | Because "house pattern" was undocumented, the rules are written out in §7 instead. **Read them — they are binding, and the contrast floor has already been broken twice on this exact page** |
| 8 | **Compliance: STANDARD on `murah`.** Keep `murah` as a plain adjective, kill `termurah` and every superlative | This is what the 11 audit fixes in §2 already assume, so that table needs no change. `murah` in titles stays |
| 9 | **The new-member bonus is upsold BEFORE purchase**, as a 4th variant of the game-page strip set | Built → §5a. Not to be confused with decision 6, which is the post-purchase acknowledgement. Both surfaces are needed |
| 10 | **A partial-redeem receipt now exists** — an order paid partly with poin shows what the poin covered | Built `1223:23548` → §5b. Closes a gap `650:3049` had left open |
| 11 | **`Total Bayar`'s amount is Barlow BOLD** on the receipt; the rows above stay SemiBold | So the total reads as the conclusion of the money column, not another row |
| 12 | **No separate "poin terpakai" line.** `1 poin = Rp 1` is locked, so `−Rp 1.250` already states 1.250 poin | A sub-line was built and cut as redundant. Do not re-add it — and do not let the money column carry poin units, it must stay pure rupiah so it visibly adds up |
| 13 | **Near-expiry state = recolour only.** Same string, amber instead of grey, at ≤14 days | Built `1272:23548`. One conditional token, no copy variant, no layout change. Amber is **`#FF8A3D`**, NOT `#F6AC37` — see §2 |
| 14 | **Guests get a PRIME row in the sandwich menu** | Built `1284:23742` in `1033:22285`. Closes consequence 2b, open since 07-31. ⚠️ The route ends at a login wall — see §10 |
| 15 | **The order-success poin banner IS tappable → wallet.** No chevron | Owner added the route, then removed the chevron: the whole band is the tap target with no visible affordance. Applied to all three receipts |
| 16 | **All `RRQ Topup` branding → `RRQSHOP`, and `rrqshop.com` is the correct domain** | 35 nodes swept. Footer + search-bar strings are library-locked — see §10. Promo code `JOINRRQTOPUP` deliberately untouched |
| 17 | **Stale PRIME session is intercepted at `Bayar Sekarang`** | Built `1330:23953`, a BOTTOM SHEET (not the login pop-up). See §5e |
| 18 | **The re-auth sheet pre-fills the identifier**, editable, no "Bukan kamu?" escape | One field to fill: password only. Editing the field IS the escape |
| 19 | **A 3X badge IS allowed on the re-auth sheet** — a scoped exception to "no chips in drawers" | That drawer is a receipt; this one is a conversion moment. Scoped to re-auth only — see §5e |
| 20 | **V1 emails: E2 only** — restated because it keeps getting assumed otherwise | Redeem has no receipt in any channel. Accepted knowingly (decision 3) |

---

## 0. What is in this repo

| File | What it is | Who needs it |
|---|---|---|
| **HANDOFF.md** | this file — the map, the laws, the data contract, open questions | you, first |
| `cashback-design-todo.md` | **the master design log and backlog** — phases, numbered items (#1–#27, C1–C6, E1–E5), session logs, every decision and what was rejected. **Division of labour set 2026-08-20: that doc plans the work, THIS file specifies it.** It no longer restates node IDs, hex values or the laws — it points here. If a session log there ever disagrees with §9 or §2 here, **this file wins** | designer-facing; read it for *why*, read this for *what* |
| `publisher-compliance-copy-rule.md` | the copy rule for price claims + a live-site audit naming **11 failing i18n keys** + a CI lint spec | you (locale files + CI) |
| `prime-landing-figma-vs-prod.md` | PRIME landing: exact copy diff Figma-vs-production, string by string | you (when you build #16) |
| `rewards-hero-motion-handoff.md` | hero motion integration: 5 host requirements, presets, a11y, perf, 7 traps, 14-item QA list | you (hero) |
| `rewards-button-motion-handoff.md` | Rewards nav-tab token motion: production CSS, React hook, gating/decay, QA | you (nav tab) |
| `cashback-research-R1-R3.md` | naming + guest-escrow research. Guest escrow is **deferred**, kept for later | background only |
| `hapus-akun-handoff.md` | **Hapus Akun (delete account)** — the Flutter app's delete flow ported to web, mobile + desktop. Node index, copy strings, states, and 5 open items incl. what happens to poin and PRIME. Outside the Rewards program but it touches the same account cluster | you (when you build it) |
| `rewards-hero-*.{js,css,html}` | **shipping code**, 5 files, zero deps, zero build step | you |
| `email-E2-*.{html,gif,py}` | **shipping code** — E2 expiring-soon email, complete | you / whoever sends |
| `Asset-Landing-Animation/` | hero assets: 3 full-size PNGs + 6 icon PNGs | you |
| `Logo RRQSHOP Vector.svg` | brand vector (has padding baked in — see Traps) | you |
| `logo-rrqshop-email.svg` + `@2x.png` | trimmed email logo + non-transparent PNG fallback | you (email) |

**Figma:** file `RWkIM1Q8o5pqt6eK3O8WBe` (RRQ-Topup-2.5), main page **"RRQSHOP Rewards"** `362:12714`. Some component sets live on page "2.5". You need **Dev Mode** access — several sets carry variant properties and auto-layout that a view-only link won't expose.

---

## 1. Program mechanics — the model you are coding against

These are locked. Backend spec (`cashback-rewards.md`, held by Tech) is authoritative for mechanics; design is authoritative for UI/UX. Any conflict = Tech↔Design sync, no silent override.

### Earning
- **Base rate = flat % per game / product-group**, with **separate member and prime columns**. NOT tiered by rupiah amount. "Bigger top-up earns more" happens because higher denominations are their own product groups with higher rates.
- **Events REPLACE the base** with an absolute per-tier % for a targeted game/product-group and period. **"2x" is a CMS display label only** — the operator inputs a percentage. Never compute the multiplier client-side from a rate ratio (member and prime ratios can differ).
- **Events apply per GAME** — a whole game page boosts uniformly. No mixed state inside one page.
- **PRIME = configurable per-game `prime_rate`** (fallback `prime_percentage` setting). Whether an event boosts PRIME is an operator choice per event.
- **Full pipeline:** `base rate (game/product-group, member vs prime column) → event REPLACES base → + payment-method event additive (pp) → + promo cashback_rate additive → floor → per-user event cap → global max-rate ceiling (config, e.g. 4%) → debt netting → event budget`
- **PRIME + coupon = automatic downgrade to member rules.** Not a user choice. UI must **warn on coupon-apply** for PRIME members — built as popup `629:15489`, never a radio picker.
- **New-member first transaction = flat override %** that ignores the entire pipeline. **Rule 1.10b: must NOT fire for users who were already PRIME before their first transaction.** This is a dev state-gate, not a separate frame.
- **No rate row configured → the game earns nothing.** This is the eligibility gate.
- Poin are earned **on delivery**, not on payment.
- **Earn base = net amount** (after poin redeemed). Consequence: a coins-only order (total Rp 0) earns **zero**.
- **Rounding = floor.** Rp 10.550 @ 1% = 105 poin.

### Expiry
- **Rolling, wallet-level.** The whole balance shares **one** `expires_at`. Default **60 days**, admin-tunable.
- Every earn **activation** (i.e. delivery) resets it to **+60d at 23:59:59 WIB**.
- **Redemption never extends it.** API returns `expires_at`; the UI shows **one date**, never per-lot dates.

### Redeeming
- **Redeem eligibility is a per-product/game flag owned by the backend** (owner, 2026-08-20) — not a hardcoded list, and **not a payment-method property**. Some products/games cannot redeem; which ones is manageable server-side and can change without a design change. The front end reads the flag and expresses it as `state=non-redeemable`.
- **Payment-method minimums are backend-managed, per method** (owner, 2026-08-20). Confirmed against production: the carriers are **all 11 bank Virtual Accounts** (BNI · BRI · Mandiri · Permata · ATM Bersama · BSI · CIMB Niaga · Danamon · Maybank · Bank Neo Commerce · BCA) **and both Convenience Stores** (Alfamart · Indomaret) — every one at `Minimal Rp 10.000` today. E-wallets and QRIS carry none. `OVO`/`Minimal Rp 10.000` in the mock is only an illustration; OVO is not a carrier. A minimum is checked against the **remainder after poin**, so redeeming can push an order under it.
- 🚨 **There is NO payment-method restriction on redeeming.** No method is barred from being combined with poin — the "DANA cannot redeem" rule this doc carried until 2026-08-20 **never existed** and has been removed here and in Figma. A method only drops out when it cannot accept the resulting remainder (next bullet).
- Poin are spent first, remainder goes to the other payment method.
- **Balance rule (locked 2026-07-31):** every headline balance = **`balance.available` only**, never `available + pending`. Pending lives in the ledger. This protects the 1:1 peg — including pending would make "Setara dengan Rp N" overstate what is spendable. Applies to: wallet SALDO card, account Poin stat cell, desktop poin card `972:9896`, and the E2 email's `{{poin_balance}}` merge field.

### Refund / clawback
- Unspent poin → lot canceled, ledger shows the reversal.
- Already-spent poin → **clawback debt**: wallet shows outstanding debt, redemption blocked until cleared, future earns net against the debt (`+1.000 → issued 0, debt −1.000` ledger rows).

### API
- Backend calls the currency **"coins"** internally; display name is always **poin**.
- `/api/cashback` — `balance {available, pending, expires_at}` · `history` (merged ledger) · `preview`.
- **Every number on screen comes from `/preview`. Never client math.** This is a law, not a preference.
- `/preview` returns a **per-source breakdown** (mirrors lot `calc_breakdown`) — needed for the drawer's dim origin lines.
- Guest `/preview` is public and returns `potential_cashback` + new-member potential + eligibility flag. That is what backs the guest login-nudge.
- Confirmed built by backend: per-source breakdown · `promo_code.cashback_rate` write path · expiry-reminder job (`cashback:wallet:expiry-reminder`, single fire at CMS `days_before`) · CMS display-label field per event · single max-rate ceiling enforcement.
- Redeem fields: `coins_to_use`, `balance_after_redeem`.
- ⚠️ **Needed and not confirmed: per-method availability against the remainder.** Minimums are a backend property per method, so the client must not hardcode them — `/preview` (or the method list) has to return each method's minimum, or better, an `available` + `reason` per method computed against `balance_after_redeem`. Without it the front end cannot draw the disabled-with-reason state at all. See §10.

---

## 2. The laws — break these and the design stops making sense

### Colour law
| Colour | Means | Notes |
|---|---|---|
| **purple** `#9A5CF5` / `#AB5CEB` / lilac `#D9A6FC` | **poin** | the program's identity colour |
| **gold** `#F6AC37` | **PRIME** — and primary CTA, and selected | knowingly overloaded; do not add a 4th job |
| **green** `#40E74C` | **temporary boost** — event OR new-member bonus | never permanent |
| muted `#ADADAD` | spend | |
| very muted `#464646` | expired / canceled | must be more muted than spend |
| red `#FF526F` | destructive / failure only | **superseded the old "red = spend/expired" rule.** Red is unused in the poin system and is free for destructive actions |
| **amber `#FF8A3D`** | **time-sensitive** — near-expiry only | **New token, added 2026-08-03.** Deliberately NOT `#F6AC37`: PRIME gold appears 13× inside the wallet frame alone, so reusing it would put an expiry warning shoulder-to-shoulder with PRIME. 6.93:1 on `#212020`; the grey it replaces is 7.24:1 |

Poin glyph is purple in every state, and always sits on a dark cell — **never directly on gold**.

**The law now exists as Figma styles (built 2026-08-20) — as an ADDITION to the RRQ Topup 2.0 library, not a parallel system.** The 2.0 file (`Qge4kS2ihFEm4900iZgzkD`, palette set `383:12517`) publishes **18 flat paint styles** and they are **already bound throughout the 2.5 file** — nodes here reference the REMOTE styles `Primary`, `Surface 1/2/3/4/5/6/7/8`, `Masking Gradient`. So the base system owns the greys, the gold and the green:

| Library style | Value | Rewards uses it for |
|---|---|---|
| `Primary` | `#F6AC37` | PRIME · primary CTA · selected — the 3 jobs the law calls overloaded |
| `Primary Gradient` | `#F6AC37 → #FDE74A` | PRIME logo, PRIME multiplier text, gold arrow glyph |
| `Success` | `#40E74C` | **is** the Rewards boost green — same hex, and it also carries the confirmation job |
| `Surface 7` / `Surface 6` | `#ADADAD` / `#464646` | spend / expired |
| `Surface 3` | `#262525` | the glyph's outer ring |
| `Surface 1` / `2` / `5` / `8` | `#191716` / `#1F1F1F` / `#0D0D0D` / `#DDDDDD` | page, rows, hairlines, labels |

**Rewards adds 15 local styles — only where the library has no token.** The purples are a **numbered scale, `Rewards/Rewards 1–5`, dark → light, mirroring the library's `Surface 1–8`** (owner, 2026-08-20: the scale is named after the *program*, not after "poin"). **The library has no purple at all** — that is the whole reason the scale exists. Because numbers carry no meaning, the role is stated in each style's description and here:

| Style | Value | Role |
|---|---|---|
| `Rewards/Rewards 1` | `#8536C7` | glyph shading — art-internal to Rewards Token, never a surface or text colour |
| `Rewards/Rewards 2` | `#9A5CF5` | program identity purple — the one the law names first |
| `Rewards/Rewards 3` | `#AB5CEB` | glyph body — every Rewards Token, every pure-poin tile |
| `Rewards/Rewards 4` | `#D9A6FC` | **poin text** — every amount, banner line and pill number, base weight only |
| `Rewards/Rewards 5` | `#D9C2FF` | poin text on a filled poin card (redeem toggle), where `Rewards 4` loses separation |

The tinted grounds follow the same convention as **`Rewards/Surface 1–7`**, grouped **1–3 poin · 4–5 boost · 6–7 PRIME** (they are namespaced by the `Rewards` folder, so `Rewards/Surface 1` never collides with the library's `Surface 1` — but always write them as **"Rewards Surface N"** in prose):

| Style | Value | Role |
|---|---|---|
| `Rewards/Surface 1` | `#241F2E` | poin **section** ground — redeem toggle card, drawer poin block, sandwich poin band |
| `Rewards/Surface 2` | `#29242E` | poin **cell** ground — nominal-card pills, game-page banner, glyph containers. Tinted twin of library `Surface 2` |
| `Rewards/Surface 3` | `#3A2F52` | poin **border** — marks member/poin identity. A PRIME row on this is the §7 defect; it belongs on Rewards Surface 6 |
| `Rewards/Surface 4` | `#263027` | boost chip ground (event) — chip colour = boost **source** |
| `Rewards/Surface 5` | `#325435` | boost chip border |
| `Rewards/Surface 6` | `#2A2418` | PRIME tile — the correct ground for a PRIME-sourced wallet/ledger row |
| `Rewards/Surface 7` | `#393329` | PRIME chip ground — gold-sourced counterpart of Rewards Surface 4 |

Three **single-purpose** tokens keep names, because they are not scales: `Rewards/Green Highlight` `#97FF97` (boosted-glyph arrow, event only) · `Rewards/Near Expiry` `#FF8A3D` · `Rewards/On Gold` `#141518` (ink on library `Primary`). Each description carries its rule, the library style it pairs with, and a CSS custom-property name (`--rewards-4`, `--rewards-surface-6`, `--rewards-on-gold`…). Swatch board **`1911:26193`** shows both halves, with the inherited chips bound to the real remote styles.
- **Six styles were created and then deleted** because they duplicated the library: Gold (=`Primary`), Gold Gradient (=`Primary Gradient`), Boost Green (=`Success`), Spend (=`Surface 7`), Expired (=`Surface 6`), and Destructive — see the red conflict below. **Do not re-add them.** Rule going forward: if the library already has the value, reference it; `Rewards/` is only for what the program introduces.
- 🚨 **The law's red does not exist in the library.** §2 names `#FF526F` for destructive/failure; the library's `Error` is **`#FF6135`**. Two different reds for one job. Red is unused in the poin system, so nothing is blocked — but the law should either adopt `Error` or the owner should replace `Error`. Owner decision; no Rewards style was shipped for it.
- ⚠️ **`Rewards/Near Expiry` `#FF8A3D` is a THIRD warm token** alongside `Primary` `#F6AC37` and `Error` `#FF6135`. The §2 rationale only argues it against gold. Library owner should bless it or fold it into `Error`.
- ⚠️ **Nothing is bound to the new styles yet.** Components still hold raw hex; these are documentation until a rebind sweep runs. Values were sampled from live nodes, so a rebind is value-for-value with **two exceptions**: `Rewards/Surface 6` `#2A2418` is the *correct* value for PRIME rows currently drawn on `Rewards/Surface 3` (the §7 defect — rebinding fixes it), and `Rewards 2` `#9A5CF5` is law-named but appears on no Rewards node.
- ⚠️ **Local to the 2.5 file.** Publishing into the 2.0 library is the library owner's call (§10) — same blocker as the nav tabs.
- ℹ️ **Pre-existing drift, not ours:** a second remote style `RRQ/Surface/Surface-2` `#1F1F1F` is also reachable in this file — a duplicate of `Surface 2` from another library. Worth raising with the library owner while the palette is being discussed.

⚠️ **Green carries TWO jobs, and the law above only names one.** Besides "temporary boost" it is also the **confirmation** signal — the PRIME success modal's check, the `Aktif` chip, the post-cancel toast. Both uses are live and neither is wrong; just don't read a green tick as a boost.

### Copy law
- **Unit word is `poin`**, lowercase, on all ID surfaces. One form everywhere: checkout, wallet, email, T&C. MY/PH localise to `points` **via i18n, never hardcoded**.
- **Multiplier framing everywhere: "2x" / "3x".** Percentages are banned outside the T&C and technical tier tables.
- Compact surfaces (card pills, chips, balances) = glyph + number. The word `poin` appears on nominal-card pills by owner decision (`+35 poin`) because the program is new.
- Never use "Rewards" as a countable noun.
- Program name: settle on **RRQSHOP Rewards** (the docs still contain "RRQ Shop Rewards" drift — treat RRQSHOP as correct).
- **Rebrand sweep, 2026-08-03: 35 nodes changed** — `Chat CS RRQ Topup` ×6 · `Terima kasih telah berbelanja bersama RRQ Topup` ×4 · `Semua pesanan kamu di RRQ Topup…` · the login-drawer sub-copy · the PRIME success modal · `Langganan RRQ Topup Prime` ×13 and `Dengan berlangganan RRQ TOPUP Prime` ×9 (both → `RRQSHOP PRIME`, also normalising `Prime` → `PRIME`). Nothing was flattened — multi-style runs were skipped and none were found.
- **Deliberately NOT changed:** the promo code **`JOINRRQTOPUP`** (`1176:23985`). It is a code value — renaming it in a mock implies renaming the real thing, which breaks anything already printed or shared. Marketing/backend call.
- Every error/edge string **names the cause before the consequence, and none of them apologise.**

### Compliance law (price claims)
Full rule + audit: [publisher-compliance-copy-rule.md](publisher-compliance-copy-rule.md). One line:

> **Never make a price claim whose comparison set is unbounded or includes the publisher's official channel.**

Superlatives are unbounded by construction, so superlatives always fail.

- **Banned:** `termurah` / `paling murah` / `harga terendah` · `harga terbaik` / `best price` · `lebih murah dari [anything unnamed]` · any reference to the official/in-game store price · **`resmi` + a price claim in one sentence** · price-match promises · `diskon` on a base game-currency price.
- **Allowed:** `murah` as a plain adjective · `hemat` when the saving is named as ours ("hemat dengan RRQSHOP Rewards") · non-price superlatives (`tercepat`, `terpercaya`) · absolute facts (`1 poin = Rp 1`, `3x poin`) · `resmi` about RRQSHOP itself · customer reviews saying "Harga Terbaik" (UGC is exempt but **must never be promoted into first-party copy**).
- **The reframe:** say what we give back, not what the price beats. That is Rewards' whole job — so compliance and the stronger retention story are the same edit.

**Your work item: 11 keyed failures, all in the i18n message bundle → a locale-file edit, lintable in CI.** Priority F1–F3 are `<title>`/meta/og and already indexed.

| # | key | current | fix |
|---|---|---|---|
| F1 | game `seo_title` (also the visible H2 above the nominal grid) | `Top Up Mobile Legends Termurah \| Banyak Promo Menarik!` | `Top Up Mobile Legends Murah, Aman & Cepat` |
| F2 | game `seo_description` | `…Termurah di RRQ Topup…` | drop `Termurah` |
| F3 | `app.title` | `RRQ Topup - Layanan Topup Termurah` | `RRQSHOP — Layanan Top Up Game Murah dan Terpercaya` |
| F4 | `benefits_best_price` | `Harga Terbaik` | `Harga Transparan` |
| F5 | `benefits_best_price_desc` | `Harga terbaik, transparan…` | `Harga transparan, tanpa biaya tersembunyi…` |
| F6 | `faq_answer_1` | `resmi` + `harga murah` in one sentence | split into two sentences |
| F7 | `suggest_title` | `Dapatkan Harga Lebih Murah` | `Dapatkan 3x Poin dengan PRIME` |
| F8 | `suggests_desc` | `…diskon terus-terusan` | `Langganan PRIME, dapatkan 3x poin tiap transaksi` |
| F9 | `benefit_flashsale_desc` | `…bikin belanja lebih murah lagi…` | `Diskon tambahan saat Flash Sale di RRQSHOP` |
| F10 | `landing_benefit_desc` | `…Flash Sale makin murah…` | see the doc |
| F11 | `benefit_discount_title` / `_desc` | `Selalu ada diskon` | **delete the claim** → poin-multiplier benefit |

**Also owed, and no grep will catch it:** the **`TERMURAH` product tag renders from a CMS image**, not the message bundle. Recommended replacement is a 4th `tag` value **`HEMAT`** in nominal-card set `529:895` (same overhang slot, same 7-char budget; `PROMO` collides with the existing tag state).

**Locale-key note:** rename `benefit_discount_*` rather than re-valuing it. A key called `discount` holding poin copy is exactly how F11 gets reintroduced.

**PASS, leave alone:** `footer_marquee` "Top Up Murah" · home `seo_title` · `Hemat` / `Hemat sampai` · the T&C's `platform resmi RRQSHOP`.

### Accessibility floor
- Interactive surfaces **≥3:1** against their background.
- Any text the user must read **≥4.5:1**.
- In-system answers: gold `#F6AC37` for primary; a `#5A5A5A`-or-lighter border for secondary.
- Known open violation you will meet: the house **disabled treatment is opacity 0.32**, which puts `#DDDDDD` at ~2:1. The OVO row follows it (DANA did too until the fictional poin rule was removed, 2026-08-20). The pattern itself should be re-cut — a `#5A5A5A`-class ink at full opacity clears 3:1 and reads the same.

### Motion law
Figma holds **no** motion. Motion source of truth = the two handoff docs and the code files in this repo. Never rebuild motion from a Figma frame.

---

## 3. The two big product decisions that shape everything

### 3.1 PRIME's blanket discount is RETIRED (owner, 2026-07-29)
Replaced by the **poin multiplier** (3x). Scoped discounts survive: Flash Sale PRIME and exclusive coupons are still real.

**Design is done. The consequences that remain are yours and PM's:**
- **`saved_price_product` = "Diskon PRIME" must die.** Replacement already built: nominal-card set `529:895` shows PRIME as a **gold 3X chip + strikethrough poin pill** (`◈ +105 ~~+35~~`).
- **F7 / F8 are now false, not merely non-compliant** — they promise a discount that no longer exists.
- **Check `saved_price_order_detail`** ("Hemat" line at checkout) no longer computes a PRIME discount component.
- **Two live casualties on the production `/prime` account page** (the landing-page diff did not cover this surface): a **"Rp 12.000 / Total penghematan"** stat cell that is now meaningless, and the **"Benefit berlangganan" 3×2 panel still listing the old 6 benefits including "Selalu ada diskon"** (= F11). Both need the new copy from `858:17959`.
- **Two live FAQ answers become wrong:** FAQ 3 ("no limits on PRIME benefits" — false; there is a global max-rate ceiling, per-user event caps, and the PRIME+coupon downgrade) and FAQ 9 (cancellation — silent on whether 3x-earned poin are kept; **they are** — earning is per-transaction, expiry is wallet-level).
- **PM/legal, not you:** this is a material change to a live auto-renewing paid subscription. Needs a subscriber notice before the next renewal, a PRIME T&C update, and a grandfathering decision. Also: verify 3x poin actually beats the old discount rate — even at nominal parity poin are worse in practice (they expire in 60 days, are RRQSHOP-only, and vanish if the user stops topping up).

### 3.1b 🚨 The reposition stopped at the landing page

Discovered while building the PRIME journeys. The discount → multiplier change was applied to the pitch and **not** to the screens where money changes hands:

| Surface | Post-reposition state |
|---|---|
| PRIME landing `858:18217` | ✅ fully rewritten — `poin 3x lipat` |
| Subscribe form `1:1156` / `1:1255` | ❌ untouched · `Rp 20.000` · no poin mention at all |
| Success modal `1184:29401` | ❌ untouched · `Manfaatkan benefitmu` without ever naming the benefit |

**So a user can read the new pitch, pay, get confirmed, and never once be told they now earn 3x** — which is the entire thing they bought. The success modal is the moment of maximum receptiveness and it says nothing.

The subscribe forms also still live on page **"2.5"** — they were never migrated into the Rewards scope. Four defects in both stages, listed in §7.

### 3.2 Global nav is LOCKED (owner, 2026-07-31)
Rewards leaves the header and becomes the **centre bottom tab**. Chat moves into the sandwich. The PRIME header button becomes **state-conditional**.

| State | Header | Bottom bar | PRIME entry |
|---|---|---|---|
| **Guest** | logo · search · sandwich | Beranda · Pesanan · **Rewards** · Promo · Login | inside sandwich menu |
| **Member (non-PRIME)** | logo · search · **PRIME** · sandwich | Beranda · Pesanan · **Rewards** · Promo · Akun | header gold circle |
| **PRIME** | logo · search · sandwich | Beranda · Pesanan · **Rewards** · Promo · Akun | none — already subscribed |

- Rewards sits at **position 3 of 5, the centre slot, deliberately.** Guest taps Rewards → Rewards landing (guest variant, Login CTA). Member taps Rewards → wallet.
- **No header Rewards circle in any state.** One entry per program. Two permanent entries would split tap data.
- Why PRIME is state-conditional: since the discount was retired, PRIME is an accelerator on Rewards, not a parallel program. A guest can't earn poin so a guest PRIME button can only reach a login wall; a PRIME member has nothing to buy. Only logged-in non-PRIME has a true pitch.
- **Chat → sandwich.** Support was already triplicated. ⚠️ **Pull Chat-tab usage before shipping** — MinBot is still self-labelled Eksperimental and "top up belum masuk" is the #1 user anxiety.

**Layout contract for the chrome components (build to this):**
- Right-side controls in a **hug auto-layout, 8px gap, pinned right** (constraints MAX). `Header / M` 139w↔101w with the right edge always at x359. `Header / D` 228w↔134w, right edge always x1320.
- Desktop `Nav` is a hug AL, 375w, centred.
- Bottom-bar `Tabs` is 375w with every tab `FILL` so they distribute at any width. **`clipsContent` must stay `false`** or the active-tab caret (it overflows above y0) gets cut.
- Sandwich: every row has a chevron except Negara & Bahasa (which keeps its `Ubah` pill). All chevrons at x219/y16 with MAX constraints so they form one vertical line 24px from the row top whether the row is 49h or 62h.
- Sandwich **icon convention — do not "unify" it:** generic actions get a grey ring + monochrome line glyph (Blog, FAQ); brand marks go bare and full-colour (flag, WhatsApp, MinBot mascot at 28.8 to optically match the flag).
- Sandwich **poin block** sits in the identity slot above the divider (the slot guest uses for the Masuk prompt): token + "12.000 poin" + "RRQSHOP Rewards" + chevron → wallet, on a `#241F2E` band (`Poin Section BG` `1043:22261`), **logged-in only**. Balance is placeholder — **wire to `/balance`**.
- The Rewards tab icon is a **filled purple token among monochrome line icons.** Deliberate. If it must join the icon system, someone owes a line-art token for nav use only.

---

## 4. Desktop shell — canonical measurements

Confirmed twice, against production `/prime` and against the owner's Figma frames. **Account-cluster pages slot into this shell; do not invent one.**

```
header        64, centred icon+label tabs
container     1196, gutters 122
left column   375   (x122–497)
gap           35
right panel   786   (x532–1318)
page bg       #191716      panel  #1D1D1D
gold marquee  y1320   then Footer / D
breakpoints   1 column below 1024, sidebar at ≥1024
```
Reference frames: `970:7473` (member Ubah Profil) · `970:9121` (PRIME). ⚠️ `970:7093` (PRIME Status Active) no longer resolves — renamed or deleted; locate by name before relying on it.

- **Wallet becomes a 5th sidebar item, "Poin Saya."** Not a new page shell.
- **Do not stretch mobile modules blindly to 784+.** Corner rivets are 8px at fixed 8px insets, so the rivet-to-width ratio collapses past ~2× mobile width; hazard-stripe headers turn from accent into banner (this happened for real at only 520 building the Panduan Poin modal); and body copy past 784 blows through 65–75ch. **784 ≈ 2.2× the 351 mobile module is the practical ceiling.** Use the three patterns that already exist on the live PRIME page instead: a **two-cell stat row**, a **3×2 icon grid**, and a **full-width list**.
- Landing-style pages are different: a gradient hero band + centred title is the **landing** grammar and is wrong for the account cluster.
- ⚠️ **Mobile nav pressure:** mobile replaces the sidebar with a segmented control ("Akun Saya | Langganan PRIME | Akun Game"). A 4th segment at 375 will be tight — decide whether Poin becomes a segment or stays reached from the account card's Poin cell.

**Site chrome to clone, not rebuild:** dark `#212020` panels, corner-rivet dots, hazard-stripe Barlow-Bold titles. Fonts: **Barlow** (Black/Bold/SemiBold/Medium) for headings and numbers, **Inter** for body.

---

## 5. Surface-by-surface: what's designed, what you wire

Legend: ✅ designed and settled · 🔶 partial · ❌ not designed · 🚧 blocked on someone else

### Earn flow
| Surface | State | Node | Dev notes |
|---|---|---|---|
| Homepage tile badge | ✅ 3 states | `556:1271`–`1273` (set `556:1274`) | Non-eligible plain · eligible = purple frame + plain glyph · event = green frame + rescaled green arrow, **arrow always faces the card interior**. Badge is **top-right, locked** (top-left collides with ML flagship logos). Tiles mark **game** properties only — **no PRIME state on tiles** (PRIME is a user property; it would light every tile identically = zero information) |
| Game page strip | ✅ **4 variants** | set `451:14444` "Rewards" | `Property 1 = nominal banner` × `Property 2 = guest \| member \| prime \| new-member`. All 343×32, fill `#29242E`, radius 8, horizontal AL, clips. **One strip slot, never two banners.** Guest always owns the strip during events: `⚡ Lagi event 2x poin — login biar kebagian. [Login]` — claim and requirement in one sentence, so guests never see a naked "2x" they can't earn. Variants: guest `451:14418` (gold Login pill, `#F6AC37` with a lilac `#D9A6FC` stroke, label Barlow Bold 12 `#141518`) · member `451:14419` (`❯` chevron) · prime `451:14420` (divider + PRIME logo) · **new-member `1187:23295`** — see §5a. Type is **Inter Semi Bold 12, base lilac `#D9A6FC`; only the multiplier changes weight and colour** (Inter Black, source-coloured — gold gradient for PRIME, green for a temporary boost). Do not colour whole clauses |
| Game page strip — **desktop** | ✅ **4 variants NEW 2026-08-20** | set **`1873:26031`** "Rewards / Desktop" | Same props as mobile (`Property 1 = nominal banner` × `Property 2 = guest \| member \| prime \| new-member`) so the two sets stay swappable: guest `1873:25964` · member `1873:25979` · prime `1873:25993` · new-member `1873:26016`. **Different placement, not a resized mobile strip** — desktop banners are the **top edge of the step-1 "Pilih nominal layanan" card**, not a free-floating strip: 793×36 (fixed height across all four), radius **`12 12 0 0`** so it fuses with the card corners, fill `#29242E`, horizontal AL, padding 6/8, gap 12, **content centred** (mobile is left-aligned with the Login pill pushed right). Text, glyph, PRIME logo, chevron and Login pill are cloned from the mobile variants — Inter Semi Bold 12 lilac `#D9A6FC`, multiplier Inter Black in the boost-source colour, fractional mobile sizes (12.2 / 11.5) normalised to 12. Built from Serling's sample inside `Game Page` `1869:31245` (raw frame `1869:32603`); that sample has been **replaced by a live instance `1875:14181`** (`state=guest`) at the same position and z-index inside step-1 panel `1869:32496`, constraints `CENTER / MIN` |
| Nominal card | ✅ **24 variants** | `529:895` | props `tag: none\|promo\|flash` × `poinState: none\|normal\|event\|prime` × `selected: false\|true`. Card 164w; `poinState=none` is 105h (no pill), others 133h. **Division of labour: tag slot = price benefits only (PROMO/FLASH); pill = poin.** Purple never competes for the tag slot. Boosted pill = `◈↑ [2X] +70 poin` — the chip replaces the strikethrough; chip colour = boost source (green event / gold PRIME). Glyph has 3 variants: normal / green arrow (event) / **gold arrow on all PRIME pills** (deliberate "always boosted" retention device). **No PRIME+event state exists** — PRIME is immune to events. Chip label and all numbers are text properties fed by the CMS label + `/preview` |
| Checkout confirmation drawer | ✅ **9 variants** | set `587:12570` | `state: guest\|member\|prime\|event\|new-member` × **`pakaiPoin: false\|true`** (guest has `false` only). New variant IDs: member `1123:12586` · prime `1123:12685` · event `1123:12784` · new-member `1123:12888`. Drawer order = **game header → poin section → payment box → disclaimer → buttons** (reward before bill). Poin is **its own section above the payment box** so the money box stays pure Rupiah. Section border carries state identity: purple member / gold PRIME / green event. `pakaiPoin=true` adds a purple **`Pakai Poin −Rp N`** row and earn recomputes on the **net** amount. **No multiplier chips in the drawer at all** — border + glyph + sub-line already carry source and multiplier; chips are a nominal-card device only. Sub-line grammar: LEFT = label + one dim origin line per boost source (Inter Regular 11), pipeline order multiplier → payment channel → promo; RIGHT = total. Worst case 3 lines; PRIME max 1 |
| Checkout confirmation — **desktop** | ✅ **overlay only, NEW 2026-08-20** | **`1894:26082`** "Konfirmasi Pesanan - D (modal overlay)" — 1440×900, `#191716` page bg + `Scrim` `#000` 60%, instance `1894:26084` centred at **x533** | **The drawer is breakpoint-agnostic — there is no desktop variant set, and that is the finding, not an omission.** Checked against the 2.0 file (`Qge4kS2ihFEm4900iZgzkD`): **every** confirmation frame there is 375 wide, including the one inside the 1440 desktop frame `164:7734` (modal `Frame 115745` at x533 y184 = dead centre). Desktop already renders the same 375 card centred over a dimmed page, so **use `587:12570` as-is on both breakpoints** and flip `state` / `pakaiPoin` on the instance; only the presentation differs (centred modal + scrim instead of a mobile sheet). Same convention as `Panduan Poin - D (modal overlay)` `964:19269`. Vertical fit is comfortable at every state: tallest is `member/pakaiPoin=true` at 590, which centres at y155 in 900 and **y110 in a real 810 viewport** — no desktop scroll, no `max-height` needed. ⚠️ Do **not** widen this modal to 786 to "use the desktop grid" — production doesn't, and the detail table is a 2-column key/value list that gains nothing from width |
| Redeem toggle (payment step) | ✅ **5 states in one set** | set **`632:12570`** `state = on \| off \| non-redeemable \| coins-only \| payment-extra` → on `631:12571` · off `631:12570` · non-redeemable `631:12573` · coins-only `631:12572` · payment-extra `631:12574` | Single-line card **above** the method list: `Gunakan ◈ 1.250 poin` + purple switch (`596:16875`). **ON re-renders every method price to the remainder (Rp 12.104 → 10.854) — that re-render IS the key interaction**, and it is how the "poin first, remainder on another method" rule gets communicated without prose. Coins-only: **the payment method section is hidden entirely** (hidden beats 8 dead options) + subtitle `Pesanan ini dibayar sepenuhnya pakai poin`; toggling OFF re-expands it with a smooth expand. Non-redeemable: the toggle card is hidden entirely, no reason shown — **absence is the explanation and it produces a true belief** ("this game can't use poin") |
| Redeem toggle — **desktop** | ✅ **5 states NEW 2026-08-20** | set **`1888:26536`** "Pilih Pembayaran / Desktop", same prop `state` → on `1888:26058` · off `1888:26172` · non-redeemable `1888:26289` · coins-only `1888:26397` · payment-extra `1888:26418` | **Desktop is mobile's layout with two columns, nothing else** (owner, 2026-08-20, against production). Built from the mobile variants, so the accordion grammar is identical — expanded `E-Wallet dan QRIS`, collapsed `Virtual Account` / `Convenience Store`, same headers and chevrons — and only the method rows reflow **two per line**. Panel 793 · toggle card **761** in the same y74 slot · accordion 761 · section 753 · row list is a **WRAP auto-layout 736 wide** with rows **362** and gaps 12 (column) / 8 (row). Set `primaryAxisAlignItems=MIN` on that list: the wrap default centres a lone 5th row in the last line, which reads as a bug. Rows keep mobile's internals verbatim — `Minimal Rp 10.000` sub-line, the OVO and DANA disabled classes, **the DANA reason in its own sub-line slot** (no desktop adaptation needed), price right-aligned at rowW−8 (absolute rows: `x = 362−8−w`; auto-layout rows: grow the label instead, 171w). Heights: on/off **461** · non-redeemable **405** (accordion moves up to y74) · coins-only **150** (no accordion at all) · payment-extra **427**. payment-extra grows the ShopeePay cell only (44 → 62, `#241F2E` base showing as the strip, `TERMURAH` at y−4 with 4px right overhang) — a wrapped grid absorbs that cleanly because the cell is its own layout item. ⚠️ **Do not rebuild this from `1869:31735`** (the game-page step-2 panel): that is a flat hazard-stripe grid with `Tampilkan semua` and matches neither production nor mobile — see §7 |
| **Method unavailable — below minimum** | ✅ **already live in production; the mock just illustrates it on the wrong row** | production reference: owner screenshot 2026-08-20 | **This state is not something to invent — production ships it.** When no nominal is selected, or the amount is under `Rp 10.000`, every VA row and both Convenience Store rows render **dimmed, with a persistent `Minimal Rp 10.000` sub-line, and the price still shown** (`Rp 0` in that state). **The sub-line IS the reason** — production adds no event copy, and none is needed: a standing property of the method states the fact without pretending something just happened. Two corrections this forces on earlier entries in this doc: (a) **"the price is gone, not restated" was wrong** — it came from the fictional DANA rule; production keeps the price and dims the row, exactly as the mock's OVO row does. (b) **The group is NOT collapsed in this state** — production shows QRIS, E-Wallet, Virtual Account and Convenience Store all expanded, so the affected rows are visible when the amount changes; no group-level signal is required. What poin adds on top: the minimum is evaluated against the **remainder after poin**, so flipping the toggle can move a row into this state mid-session. Distinct from an **account-state block** (clawback debt, §5 #21), which is not a standing property and does need its own reason string |
| Payment-extra cashback | ✅ | `621:2204` / `621:2279` | Footer strip under the method row: dark-purple bg + **green** `1.5X` accent. ⚠️ **The number is only true at the 1% base tier** — it must be a **CMS display field per campaign**, never client-computed. Safe numberless alternative: `Bonus poin dengan metode pembayaran ini` |
| Coupon banners | ✅ | combo `640:12572` (set `641:12570`, `state = discount \| cashback \| combo`) | GREEN + discount icon = money benefit · PURPLE + boosted token = cashback code · combo stacks green above purple in pipeline order. Icon disambiguates, colour reinforces |
| PRIME coupon-downgrade warning | ✅ | `629:15489` | Popup at the **"Cek"** moment. `Konfirmasi` + trade-off body + `Hapus Kode` / gold `Ya, Tetap Pakai` → purple banner. Gold always sits on the constructive option |
| Order status | ✅ two placements | desktop `596:14892` (the left-col poin card inside it is unnamed) | **(a)** inline `Poin Rewards` row inside the PRODUK panel = **in-progress only** ("akan dapat"); **(b)** sticky bottom banner above `Beli Lagi` = **delivered only**. Mutually exclusive, never both on screen. Banner is **dark + purple accent, not solid purple** — solid purple competes with the gold `Beli Lagi` CTA and solid purple is celebration-burst language for one-time moments, not a persistent sticky. Copy: pending `[N] poin didapatkan setelah pesanan selesai` · terkirim `Kamu dapat [N] poin dari pesanan ini 🎉` — **🎉 only on terkirim, never celebrate un-earned poin.** Desktop has no sticky (mobile affordance): the poin card rides in the **left column under the QR/status/success card**, across all states. New-member celebration folds into the delivered banner with louder copy, no separate hero |
| Coins-only success page | ✅ | `1138:22227` | PRODUK panel goes 2 rows → 4: ID User · **Harga Layanan Rp 2.659** · **Pakai Poin −Rp 2.659** (purple `#D9A6FC`, label and value) · **Total Bayar Rp 0**. `Metode Pembayaran` = **`Poin RRQSHOP Rewards`** (a receipt must say how it was paid). The old single `Total` row was relabelled `Harga Layanan` so it reads in the same grammar as the drawer — "Total" directly above "Total Bayar" would be two meanings of one word. ⚠️ **That is probably right on the normal receipt too** — the base frame still says `Total`. Sticky banner is **restated, not deleted**: `Pesanan ini dibayar penuh pakai 2.659 poin`, no 🎉, because a coins-only order earns nothing |
| **Partial-redeem receipt** | ✅ **NEW 2026-08-03** | **`1223:23548`** | Built this session to close a real gap — `650:3049` shows only `Total`, so an order paid partly with poin had no record of what the poin covered. PRODUK panel: ID User · `Harga Layanan Rp 12.104` · **`Pakai Poin −Rp 1.250`** (purple `#D9A6FC`, label AND value) · **`Total Bayar Rp 10.854`** with the amount in Barlow **Bold** while the rows above stay SemiBold. `Metode Pembayaran` = a real method (`QRIS`) — coins-only says `Poin RRQSHOP Rewards` only because there is no other method. Banner keeps the earn message **with 🎉**: earn base is net, Rp 10.854 @1% floors to 108 poin, so this order genuinely earns — unlike coins-only, which earns nothing and drops the 🎉. **No separate poin-terpakai line** (decision 12). Cloned from `1138:22227` with text edits only; geometry untouched |

### 5a. New-member bonus upsell on the game page (built 2026-08-03)

The new-member first-transaction bonus is now **upsold before purchase**, on the game page, not only acknowledged after it. Built as a 4th variant of the existing strip set — **not a new element**, because the strip slot rule is one banner per page.

**`Property 2 = new-member` · `1187:23295`**

| | |
|---|---|
| Copy | `Transaksi pertamamu dapat 2x poin.` |
| Type | Inter Semi Bold 12, base lilac `#D9A6FC`; `2x` in Inter **Black** green `#40E74C` |
| Glyph | `Rewards Token / Small / Multiply` `507:1467` — the green-arrow boost token |
| CTA | inherits the guest gold `Login` pill → **so this variant is guest-facing** |
| Contrast | lilac 7.8:1 · green 9.2:1 on `#29242E`. Both clear the 4.5 floor |

**Placement:** instance `1188:23303` on `Game Page Upsell - M` `1176:23490`, inside step-1 panel `1176:23739` at **x16 y89** — the slot between the step header and the first product section, 343 wide at 16px gutters. Panel grows 287 → 331, nominal grid moves y89 → 133, bottom rivets → y315, siblings shift +44, artboard → 1917. **The panel is not auto-layout, so all of that is manual** — budget it if the strip height ever changes.

**Four rules that are binding, and why:**

1. **No rupiah figure on the strip, ever.** The strip sits *above* the product grid, so no nominal is selected and `/preview` has nothing to price. The number already has two built homes downstream — the guest drawer dual-number nudge and drawer `state=new-member` `1123:12888` — both of which fire *after* a nominal exists. The strip's only job is to create the reason to log in.
2. **A single multiplier is only honest if the mechanic is a true multiplier.** Today the bonus is a **flat override % that replaces the whole pipeline**, and base rates vary per product-group (5 Diamonds 1%, 1450 Diamonds 2%) — so one override yields a different multiple on every nominal. `2x` is a **placeholder**; see the open item in §10.
3. **`pertama` must appear in every version of this copy.** The bonus is once-ever and lands at roughly 8× normal earn in the built drawer (11.916 vs 1.489). The risk is not that users miss it — it is that they expect it again.
4. **`new-member > event`.** This variant *replaces* the event swap rather than stacking with it, because the override ignores the event rate entirely. Both are green under the colour law, so **`pertama` is the only disambiguator** between "first-transaction bonus" and "event running". That is what rule 3 is protecting.

**Suppression matrix — get this wrong and it misfires:**

| User | Purchased before | PRIME before 1st txn | Strip variant |
|---|---|---|---|
| Guest | unknown | — | **`new-member`** (this one) |
| Member | No | No | `new-member` copy, but **Login pill must become the `❯` chevron** — not yet built, see §10 |
| Member | Yes | — | `member` |
| PRIME | No | **Yes** | `member`/`prime` — **never `new-member`** (Rule 1.10b) |
| PRIME | Yes | — | `prime` |

The rule also lives in the variant's **Figma description**, so it survives without this file — same practice as the poin-availability rules on `632:12570`.

⚠️ **Not to be confused with decision 6 in §0a.** That decision (louder delivered sticky banner, no new surface) is the **post-purchase acknowledgement**. This is the **pre-purchase upsell**. Two different surfaces, both needed, neither replaces the other.

### Account / wallet
| Surface | State | Node | Dev notes |
|---|---|---|---|
| Account dashboard | ✅ both states | M reg `666:3416` · M PRIME `670:3918` | Unified layout. **Poin + Voucher as two side-by-side stat cells** in the profile card (replaced a dead "Total hemat"). Regular = silver card + PRIME upsell row; PRIME = **entire profile card gold gradient** + a gold `3X` chip on the Poin cell. Poin glyph purple in both, in a dark cell. Poin cell taps → wallet. Non-PRIME Voucher cell reads **`Khusus PRIME`** (gold) — a count of "8" there was wrong and confusing. ⚠️ **Propagate that to mobile `666:3416`**, and **declare its tap target** (it is now an upsell surface, so it should route to the PRIME page) |
| Wallet (mobile) | ✅ + 3 alt states | main `681:16812` · empty `706:12570` · debt `708:12570` · **all-expired `1127:22108`** | Body sections live in **one vertical AL wrapper** `729:18298`, 351w @ x12, itemSpacing 12, order: SALDO card `694:17538` → Informasi Poin `732:16577` → Riwayat `683:17326`. SALDO card is GoPay-Coins style: glyph + label + big Barlow Black number + `⇄ Setara dengan Rp 12.000` + dashed divider + gold expiry line + `Cara kerja` pill. **Expiry copy is conditional: >2 weeks → a calm fixed date (`30 Des 2026`); ≤2 weeks → amber urgency (`hangus dalam 14 hari`).** Debt state: balance `−500` in **purple** `#D9A6FC`, deliberately **not red** — the alarm is carried by the banner, not the number; owner declared debt a rare edge case, **do not elaborate further** |
| Wallet ledger | ✅ sign convention | — | **earn** = `+` prefix, purple `#D9A6FC`, Barlow SemiBold 15 · **spend** = `−` muted `#ADADAD` · **expired** = `−` very muted `#464646` · **canceled** = `0` `#464646` · **pending IS shown**, amount purple + `Pending • [date]` subtitle. Dates `#8B8B96`. Row title Inter Medium 14 `#DDDDDD`. Footer `Tampilkan lebih banyak`. Row icon rule: earn/spend rows carry the **game thumbnail** (40px, left); pure-poin rows with no game (expiry, bonus) get a **muted glyph tile**. **A pending lot that cancels updates IN PLACE** — same row, amount → `0` `#464646`, subtitle → `Pesanan Dibatalkan • [date]`. Do **not** delete the row and do **not** append a second one; keeping it is what makes the balance arithmetic explainable |
| Wallet "Informasi Poin" | ✅ 2 states | expanded `732:16577` · collapsed `729:18311` | 6 icon-rows, distinct glyph per row + footer pill `Info lebih lanjut seputar Poin` with a green `?`. **Populated wallet starts expanded; empty wallet starts collapsed** (no history to scroll → keep it tidy). The `?` pill opens the Panduan Poin drawer |
| Wallet (desktop) | 🔶 built, has bugs | `972:9896` | Balance card as a **two-cell row** with a purple glow border · Informasi Poin as a **2×3 grid, row-major, preserving mobile order 1–6** · Riwayat as a full-width list with thumbnails. `Panduan Poin` pill → wire to the desktop modal `964:19271`. ❗ **Open bugs listed in §7.** Still owed: the near-expiry ≤2wk amber variant on desktop, and the PRIME variant (gold left card + purple-glow right card = two glowing cards competing — check it when built) |
| Panduan Poin | ✅ both breakpoints | drawer-M `725:16668` · **modal-D `964:19271`** in overlay `964:19269` | 5 sections: Cara Kerja · Berapa Poin (prose, **no table** — the tier table lives in the T&C) · Member PRIME (3x) · Masa Berlaku (60 hari) · Pakai Poin. The desktop modal was **cloned from the mobile sheet, not rebuilt**, so copy is byte-identical and cannot drift. Modal reflow spec: 520w · gutter 24 (content 472) · title bar 52 · content top 68 · section gap 20 · gap-to-button 32 · bottom pad 24 · radius **16 on all four corners** · Tutup 166×44 centred. **Dev wiring: `Tentang poin ⓘ` → modal at ≥768, → bottom-sheet below.** Pills: desktop `961:19344`, mobile `962:19345` |
| Transaction history | ✅ both breakpoints | M `825:17404` · D `829:17404` | props `Status` (5: Belum Dibayar / Diproses / Terkendala / Selesai / Dibatalkan) · **`Pakai Poin` boolean** → toggles the `– Pakai N poin` line · `Beli Lagi` boolean. Per-status poin logic: Selesai + eligible = purple `+N poin` (earned) · Belum Dibayar / Diproses / Terkendala = muted `+N poin setelah pesanan selesai` (pending) · Dibatalkan = nothing. Voucher/non-game = nothing (eligibility). Already swapped into both layouts (mobile list `767:17597`, desktop `767:4297`) |
| PRIME subscription management | ✅ mobile | page `967:19481` · cancel dialog `967:19975` · status block `1089:12588` · account row `1097:22032` · history row `1112:12586` | Entry = the "Aktif" row on the PRIME account page. **Cancel dialog:** body states all four facts (`Benefit PRIME tetap aktif hingga 8 Jan 2026, lalu akunmu jadi member biasa. Poin yang sudah kamu dapat tetap aman. Periode yang sedang berjalan nggak bisa direfund.`), buttons = quiet **`Ya, Berhenti`** (left) / gold **`Nanti Lagi`** (right) — gold is on the safe action, and `Batalkan` was rejected because in Indonesian UI it is the *dismiss* verb. **Do NOT add friction to cancellation** — de-emphasize, don't obstruct; one tap, plain label, no guilt copy. Consumer-protection bodies flag dark patterns here and the site's own footer links Ditjen PKTN. `PRIME Status Block / M` `1089:12588` variant `state = aktif \| dibatalkan`, both 62h so the page doesn't shift; **swap on `subscription.cancelAtPeriodEnd`**, and after the end date passes replace the block with the normal PRIME upsell row. `PRIME History Row / M` variant `status = sukses\|pending\|gagal` → `#40E74C` / `#ADADAD` / `#FF526F`; only the status token changes between variants — date, period, method and amount stay, because a failed charge is still a real attempt against that period. 🚨 **`gagal` is a dead end** — a failed renewal is exactly when a subscriber must act and the row offers no retry, no "update payment method", no statement that PRIME lapses. **Open: decide inline retry vs a banner vs a push/email deep-link.** No desktop variant of this page |
| PRIME account/desktop pages | 🔶 built by owner | Ubah Profil `970:7473` / `970:9121` · ⚠️ Status `970:7093` DEAD · Riwayat Langganan `972:12084` · Voucher `972:13189` | Member vs PRIME differ **only in the left column**; the right panel is identical. Upsell copy `Upgrade ke PRIME, dan dapatkan 3X poin lebih banyak daripada member biasa` **passes compliance** — the comparative names an internal set. ❗ Open bugs in §7 |
| Error & edge states | ✅ 3 states | board `1158:23243` | See §6 |
| My Coupons (#24) + coupon card (C6) | ⏸ **PARKED** | — | Built on 07-31, then **all deleted at the owner's request: "don't change this area at all."** Nothing remains on canvas. **Do not rebuild without an explicit go-ahead.** The PRIME Voucher page `972:13189` is untouched and already solves the layout if it is ever picked up |

### Landing pages
| Surface | State | Node | Notes |
|---|---|---|---|
| Rewards landing | ✅ both breakpoints | D `870:19298` · M `887:12619` | Owner's desktop is canonical. Structure: header → hero (gradient `#D9A6FC → #AB5CEB → #500EC1 → #191716` + floating 3D props + big Rewards Token + glow; headline **Barlow Black Italic**, gold on "RRQSHOP Rewards"; gold Login CTA) → 3 feature cards → "Cara mendapatkan poin" panel (lilac Barlow Black title, 01/02/03 purple token badges) → "Menggunakan poin" panel (+ payment-toggle mock `631:12572`) → "Sering Ditanyakan" FAQ (9 questions) → hazard marquee → footer. Panels `#241F2E`, cards `#191716` |
| Rewards hero band | ✅ **built as code** | D `870:20216` · M `883:19494` | See §8. Figma holds no motion |
| PRIME landing | ✅ both breakpoints | D `858:17959` · M `858:18217` | This is the reposition **as copy**. Structure untouched — same sections, grid, CTA. Headline, subcopy, 3/6 card titles and **6/6 card descriptions** rewritten; one FAQ deleted. Copy parity between breakpoints is **exact**. Full string-by-string diff → [prime-landing-figma-vs-prod.md](prime-landing-figma-vs-prod.md). Card 1's gold 3X badge + purple `Tentang poin ⓘ` pill looks like a colour-law violation and **isn't** — gold is the PRIME multiplier, purple is poin, matching the account-dashboard precedent |
| Cashback event landing (#22) | 🚧 **do not build yet** | — | Recommendation: **not a standalone page** — reframe as a section or filter inside Promo. It would be the **third** promotional surface (Promo tab + Rewards tab already exist), and the decision point is already covered by the nominal card event state + strip swap + tile badge. 🚧 **Blocking question is ops/CMS, not design: how many cashback events run concurrently, and are they already listed on `/promo`?** Nothing should be drawn before that is answered. If it is built, **countdown is the expensive part** — needs a real `ends_at` from the API (never client-computed), WIB timezone, and defined behaviour at zero |
| T&C (#20) | ❌ not built · **decision 2: extends the LIVE T&C page, not a new page** | — | Add a Rewards/PRIME section to the existing T&C carrying: the product-group rate table + full 8-bullet S&K (**product-group % framing, never rupiah amount-bands**) · **which product groups cannot redeem** (a backend-managed property — write it as a category rule, never as a payment-method exclusion) · **the reseller/pricing statement the live T&C is missing**: `RRQSHOP adalah penyedia layanan top up pihak ketiga. Harga yang tercantum ditentukan sendiri oleh RRQSHOP dan tidak mewakili harga resmi dari penerbit game.` · PRIME benefit terms rewritten for the retired discount, including where the grandfathering decision gets written down. **No new page shell** |
| Refund → clawback (#21) | 🔶 debt wallet done · **decision 4: one state to add** | `708:12570` | The debt wallet state stays as-is — the owner has twice ruled debt a rare edge case. **Build exactly one thing: the checkout blocked-redemption reason**, so a user with outstanding debt who meets the redeem toggle is told why it won't work rather than facing a dead control. Follow the disabled-row treatment (§5, *Method unavailable — below minimum*) but note the class difference: a minimum is a standing property so its sub-line is always present, while debt is an account state that must state its own reason. The reason occupies the sub-line slot, `Barlow Regular 12`, `#ADADAD`, full opacity, one line. Ledger reversal rows and the future-earns-net rows (`+1.000 → issued 0, debt −1.000`) follow the **existing sign convention** and need no new frames |

### Emails (V1 = 5)
### Emails — **V1 = E2 only** (decision 3, 2026-08-03)

| # | Subject | State |
|---|---|---|
| **E2** | **Expiring soon** | ✅ **CLOSED — production HTML in this repo.** See §8. **The only email in V1** |
| ~~E1~~ | ~~Points earned~~ | **out of V1** — partly covered by the delivered sticky banner + wallet ledger |
| ~~E4~~ | ~~Redeemed receipt~~ | **out of V1.** ⚠️ Known consequence: **redeem has no receipt in any channel.** A user who pays fully with poin gets the success page `1138:22227` and nothing in their inbox. Accepted knowingly — if it gets reopened, this is the first one back |
| ~~E6~~ | ~~Welcome + first point~~ | out of V1 |
| ~~E7~~ | ~~Event / coupon granted~~ | out of V1 |
| ~~E3~~ | ~~Expired~~ | dropped 2026-07-21 |
| ~~E5~~ | ~~Guest claim~~ | deferred with guest escrow |

### 5b. The three receipts — pick by payment shape

| Order paid | Receipt | PRODUK panel |
|---|---|---|
| entirely with money | `650:3049` | `Total` only |
| **partly with poin** | **`1223:23548`** | `Harga Layanan` · `Pakai Poin` · `Total Bayar` (Bold) |
| entirely with poin (Rp 0) | `1138:22227` | same 3 money rows, `Total Bayar Rp 0`, method = `Poin RRQSHOP Rewards` |

The money column stays **pure rupiah** in all three so it visibly adds up. Poin units never enter it — the 1:1 peg means `−Rp 1.250` already states 1.250 poin.
⚠️ `650:3049` still labels its single row `Total` where the other two say `Harga Layanan` / `Total Bayar`. Worth aligning: "Total" directly above "Total Bayar" would be two meanings of one word.

### 5c. Login pop-up — a modal over the page, not a route (asset arrived 2026-08-03)

**`Login Drawer` `1184:25281`, 375×496 — despite the name this is the POP-UP, not a bottom sheet (see §5e).** Opens **over the current page**; the page behind stays mounted and ✕ dismisses it. So a guest tapping `Login` on the strip is **one page re-rendering, not two navigations** — that changes the implementation.

Contents: title `Masuk` · sub `Selamat datang kembali di RRQ TopUp` · `Email/Nomor Handphone` (one combined field, placeholder `Contoh: xxx@mail.com / +62xxx, +60xxx`) · `Password` with a show/hide eye · `Lupa password?` · gold full-width `Masuk` CTA · `Belum punya akun? Daftar Akun`.

Five discrepancies, all real work:
1. 🚨 **A market is missing.** The placeholder offers `+62` and `+60` — Indonesia and Malaysia. Markets are **ID + MY + PH**, so `+63` is absent and a Filipino user cannot tell whether their number is accepted. Not a copy nit.
2. **Figma and production disagree on 4 strings.** Figma: `Password` · `Lupa password?` · CTA `Masuk`. Production: `KATA SANDI` · `Lupa Kata Sandi?` · CTA `Login`. Someone must say which wins.
3. **Mixed ID/EN inside one form** — `Email/Nomor Handphone` beside `Password`.
4. **Stale brand** — sub-copy reads `RRQ TopUp`; rebrand sweep to RRQSHOP.
5. **The guest strip CTA says `Login` while this says `Masuk`** — same drift the unit-word law exists to stop. Align one way.

### 5d. Pesanan Diproses — `Pembayaran Diterima` (asset arrived 2026-08-03)

**Set `1184:25559`, REMOTE (RRQ Topup 2.0 library).** Three **fulfilment** variants — note `Top Up with login` means products needing the player's game credentials, *not* user login:

| Variant | Node | Copy |
|---|---|---|
| Instant | `1184:25726` | `Siap-siap ya, pesanan kamu akan segera masuk.` |
| Non-instant | `1184:25560` | `Proses pesanan ini membutuhkan waktu ± 2 jam…` |
| Top Up with login | `1184:25883` | `…silakan Chat CS RRQ Topup untuk kami lanjutkan…` |

- **None of the three carry poin content.** They are status headers only, so the pending row `[N] poin didapatkan setelah pesanan selesai` must live in the PRODUK panel below — muted, no 🎉, and mutually exclusive with the sticky banner.
- ⚠️ **No frame anywhere shows status header + PRODUK panel + pending poin row together.** Still a documentation gap.
- 🚨 **`Top Up with login` tells the user to Chat CS — and the nav decision moved Chat into the sandwich, with guests losing in-app chat entirely.** So the one order state that *requires* contacting support points at a door being closed. Same shape as the `gagal` renewal dead end. This string also still says `RRQ Topup`.
- Being remote, **any edit needs the library owner** — a third item on the consolidated ask in §10.

### 5e. Stale PRIME session — the re-auth bottom sheet (built 2026-08-03)

**`Login Lagi Drawer` `1330:23953`, 375×549.** A user sits on the game detail page with cached PRIME chrome; the session dies; they tap `Bayar Sekarang`. Every poin figure on screen is now a promise the server will not honour.

🚨 **This is a BOTTOM SHEET and it is NOT the login pop-up. Two different components:**

| | Chrome |
|---|---|
| Login pop-up `1184:25281` (misleadingly named "Login Drawer") | `WELCOME BACK` hazard tab · corner rivets · 40pt **centred** title · all four corners rounded · opens as a modal |
| **Re-auth sheet `1330:23953`** | flat `#212020` · 52px title bar, **left-aligned** Barlow Black 20 + `✕` at x343 · **top corners 12 only** · anchored to the viewport bottom · same sheet chrome as Panduan Poin `725:16998` |

Re-auth is a sheet because it **interrupts a task in progress** — the page behind must stay visibly present. A centred modal reads as a new destination, exactly wrong when there is a half-finished order underneath.

**Adapted from the 2.0 flow** (file `Qge4kS2ihFEm4900iZgzkD`, node `6206:56960`). What changed and why:

- The 2.0 block sold a **cheaper price** — `Rp 149.955` struck against `Rp 152.755` plus `Diskon PRIME 2RB`. Retired. There is now **no price difference at all**, so the strikethrough is gone and the card shows **one** price. That is the reposition headline made literal: *Top up seperti biasa, poin 3x lipat.*
- Benefit moved entirely to poin: PRIME token + `Poin didapat setelah login` + gold **3X badge** + `4.467 poin` lilac. Figures match the confirmation drawer `586:12572` the user is about to see.
- Card carries a **thin gold border** — border colour is state identity (purple member / gold PRIME / green event).
- **Identifier is pre-filled and editable** (`081234561776`, full opacity, not a 0.4 placeholder). One field to fill. Owner declined an explicit "Bukan kamu?" — editing the field is the escape. ⚠️ Open: full vs masked (`08*******1776`) is a production call.
- `Belanja Sebagai Guest` carries a muted **`tanpa poin`** beneath it. A silent guest button would produce a FALSE belief — the user would assume the 4.467 still applies.

**The 3X badge is a scoped exception.** The confirmation drawer has no multiplier chips by decision (07-10: border + glyph + sub-line already carry it). This sheet breaks that on purpose — it is a *conversion* moment, not a receipt, and the nominal pills directly behind it carry the same gold chip. Cloned from the real chip `517:2635` (in `526:637`), scaled 1.35×. **Do not read this as licence to add chips elsewhere.**

🚨 **BINDING DEV RULE — DO NOT LOSE THE FORM.** By the time they tap `Bayar Sekarang` they have selected a nominal and entered a game **User ID**, a WhatsApp number, possibly a promo code. **Re-auth must happen in place — token refresh, no navigation — and preserve all of it.** If re-login reloads the page and wipes that input, this sheet becomes worse than the bug it exists to fix: the user is punished for a timeout they did not cause. This is the highest-risk implementation detail in the flow.

⚠️ Also: **Rule 1.10b must key on transaction history, not session state.** The guest drawer's dual-number nudge includes a new-member bonus, and a returning PRIME subscriber is emphatically not a new member.

### 5f. How a user reaches the wallet — four routes

| Route | Node | Notes |
|---|---|---|
| **Rewards bottom tab** | `1026:22271` | Tab 3 of 5, centre slot. The primary route |
| **Account → Poin stat cell** | `666:3416` | `12.000` over `Poin` in the profile card |
| **Sandwich → poin block** | `1034:22238` | `12.000 poin` + chevron. Where C1 landed (decision 1) |
| **Order-success poin banner** | `650:3049` banner `650:3279` | Tappable → wallet (decision 15). **No chevron** — the whole band is the target |

🚧 **Placement gap:** the new `Bottom Bar / M` is only instanced on **four** frames — `Akun - Regular`, `Akun - PRIME`, `Riwayat Pesanan - M`, `Rewards Landing - M`. **Home `458:14753` and the game page still carry the old bar or none**, so the primary route to the wallet is invisible on the two highest-traffic screens. Placement work, not design work — the component is correct.

⚠️ **Not a route: the E2 email.** Its CTA is `Belanja dengan poin`, which sends the user shopping rather than to the wallet — arguably right, since the goal is spending.

📐 **The success banner is STICKY in production** — fixed to the viewport bottom above the `Beli Lagi` bar. The mock draws it in document flow at the page bottom, which is why the footer appears above it in the journey cards. Build it sticky, not as a trailing element.

### 5g. Near-expiry wallet state (built 2026-08-03)

**`Riwayat Poin / Near Expiry (≤14 hari)` `1272:23548`.** Closes the gap that disabled the only V1 email.

The wallet previously carried **only** the calm variant. E2's send is `days_before = 7`, and the documented reason for 7 was that the wallet flips amber at ≤14 days so the click-through lands somewhere urgent. That state did not exist — so the email shouted `Poin kamu hangus dalam 7 hari!` and delivered the user to a screen saying their poin expire in December.

Implementation is a two-line conditional on one token:
- `> 14 days` → `#ADADAD` (7.24:1)
- `≤ 14 days` → **`#FF8A3D`** (6.93:1)

Threshold off `expires_at` — and since the whole balance shares **one** `expires_at`, there is a single date to test.

⚠️ **Colour is the only signal.** E2 does the arithmetic (`7 hari`) and the wallet undoes it (`30 Des 2026` in amber). Colour-only state also fails WCAG 1.4.1, and this file has rejected colour-alone before — the voucher cards got explicit badges for exactly this reason. Cheapest fix if wanted, same slot: `Poin kedaluwarsa 12 hari lagi · 30 Des 2026`.

### Deferred to a later phase
Guest auto-account escrow + OTP claim (R3) · #10 guest order-tracking claim CTA · E5 guest claim email · #11 login page and #12 register form (**V2 — V1 uses the existing pages as-is**). ⚠️ **But a `Login Drawer` design DOES now exist — `1184:25281`, 375×496.** See §5c. V1 consequence: guests simply don't earn, and get a `Login/register to earn` nudge instead.

### Dropped, with reasons — do not reopen
- **#18 first-time cashback explainer modal.** Owner: *"my users don't read."* A first-visit modal fires before the user has any poin, hits guests who can't earn, and its only action is a dead-end OK. Every mechanic it would have explained is already shown by the interface: "poin first, remainder on another method" → the toggle-ON price re-render · "some methods can't use poin" → the disabled state · "spent poin can't come back" → only relevant at refund (#21).
- **Event countdown.** Stakeholder call. The urgency lever is intentionally unused in V1. If event conversion underperforms, countdown is the first knob to revisit.
- **Per-card event 2X tag.** Redundant — events are game-level so a whole page boosts uniformly, making a per-card tag × 20 pure noise. The boosted pill carries the proof.
- **Section-scoped events / countdown chip.** Superseded by game-level events.

---

## 6. Error & edge states (#19) — and the pattern rule behind them

Reference board `1158:23243` (page "RRQSHOP Rewards", 1333×884). Each state is drawn **in its real surface**, not as a loose component — you need to see where the string lands.

> **The rule: a failed *action* gets a dialog; a changed *fact* gets an inline note; and a fact the user would infer correctly on their own gets nothing at all.**

1. **Event 2x ends mid-session** (`1158:23250`, member drawer) — **re-fetch and explain.** The card corrects itself on return; if the number changed between tap and confirm, the drawer's poin section carries `Event 2x sudah berakhir, jadi poin dihitung dengan rate normal.` directly under the earn row. *Hold-the-rate was rejected* — it is a promise the UI cannot keep alone (the rate would have to be frozen per session server-side, and the locked law is that no number is client-held). *Silent was rejected* — it is the trust bug the item was raised for.
2. **Poin expire during checkout** (`1158:23463`, payment step) — balance is 0, so **the toggle card is removed entirely** and its slot states `Poin kamu kedaluwarsa sebelum pembayaran selesai, jadi nggak bisa dipakai di pesanan ini.` Prices return to Rp 12.104, and any method that had dropped out for failing the remainder minimum becomes available again.
3. **Gagal pakai poin** (`1158:23574`, dialog) — cloned from the `629:15489` chrome. Body leads with reassurance because "did my poin just disappear?" is the actual anxiety: `Poin kamu nggak jadi kepakai di pesanan ini dan saldonya tetap utuh. Kamu bisa coba lagi, atau lanjut bayar tanpa poin.` Buttons: quiet `Lanjut Tanpa Poin` (left) / gold `Coba Lagi` (right).

🚫 **Balance changed between page load and pay — NO UI. This is a dev rule, not a screen.**
**Auto-correct, never block:** the toggle silently re-renders to the real number, method prices re-render with it, and redeem succeeds with whatever is actually there. The explanatory note was built and then cut, because auto-correcting produces a **true** belief and most users never registered the old number.
⚠️ **The one condition that brings it back:** if the client re-fetches `/preview` **while the payment screen is already visible**, prices visibly jump (Rp 10.854 → 11.304) under the user's eyes — that *is* the price-reversal case and needs a word. If the fetch happens on entering the payment step, or only at submit, nothing needs saying. **Tech owns this choice; design follows it.** The cut frame is gone (`1158:23352` no longer resolves), but it can be rebuilt from the pattern in one pass.

⚠️ **Owed by Tech, and it decides whether the specificity above survives:** does `/preview` expose *why* a number changed (event ended vs balance changed vs expired), or must the client infer it by diffing? These states each need a distinguishable signal, otherwise the front end will pick one generic string.

**Not drawn:** network failure on `/preview` itself (the poin section can't render a number at all), and **the remainder-below-method-minimum case — which is now the ONLY valid instance of a method disabled by a redeem** (it inherited that job when the DANA rule was retired on 2026-08-20). **Correction (2026-08-20): this state already exists in production and needs no new invention** — dim row + persistent `Minimal Rp 10.000` sub-line + price retained, on all 11 VA rows and both Convenience Store rows, with every group expanded. It needs **no separate reason string**: the standing sub-line carries the fact. What the mock lacks is the *rows* — it has no VA or Convenience Store methods drawn at all, so it cannot show the state. The only genuinely poin-specific part is that the minimum is tested against the **remainder after poin**, so the toggle can push a row into this state mid-session.

---

## 7. Known defects in the mocks — do not copy these into code

The mocks contain placeholder-data bugs and a few real design defects. A dev will faithfully reproduce whatever the mock shows, so read this list before building.

**Fixed already** (noted so you don't re-report them): the `31/2/2025` dates, `Rp 20,000` → `Rp 25.000`, out-of-order history rows, the misspelling `kadaluwarsa` → **`kedaluwarsa`** (KBBI-correct), the invisible `Simpan` button, the unreadable EMAIL field, voucher state badges, and the debug string `--- Semua data sudah ditampilkan ---`.

**Still open — the payment step is behind production (found 2026-08-20, checked against the live desktop page):**
- 🚨 **The DANA disabled row asserted a rule that does not exist — removed 2026-08-20.** Both `state=on` variants carried DANA at opacity 0.32 with `Poin tidak berlaku untuk pembayaran ini` in place of its price, and `632:12570`'s description called it "DANA RULE (locked)". Per the owner there is **no** payment-method restriction on redeeming; eligibility is a per-product/game backend flag. Fixed in mobile `631:12571` and desktop `1888:26058` (DANA is a normal `Rp 10.854` row) and in the set description. **The S7 journey card `1353:19821` still shows the old lane B and needs re-shooting.**
- ⚠️ **The OVO row illustrates the right treatment on the wrong method.** Production (owner screenshot, 2026-08-20) dims a below-minimum row, keeps a persistent `Minimal Rp 10.000` sub-line and **keeps the price** — which is exactly what the mock's OVO row does, so **the treatment is faithful and my earlier "keeps its price while disabled" note was wrong**; the doc's "price gone, not restated" rule was the anomaly and came from the fictional DANA rule. What remains wrong: (a) OVO is an **e-wallet**, and e-wallets carry no minimum — the carriers are the 11 VA rows and the 2 Convenience Store rows; (b) OVO is dim in `state=on` AND `state=off`, at a `Rp 10.854` remainder that clears `Rp 10.000`, so it fails no minimum in either state. Fix by moving the example onto a VA row, which first requires drawing the VA and Convenience Store groups at all.
- 🚨 **The game-page step-2 panel `1869:31735` is not the production layout.** It draws a **flat hazard-stripe section + a 2-column grid + `Tampilkan semua`**. Production uses the **accordion**: separate group cards, each with a header and chevron, rows two-per-line inside. Mobile `632:12570` already uses the accordion, so the mock is inconsistent with *itself* as well as with production. The new desktop set `1888:26536` follows production/mobile; the game-page panel still needs redrawing (owner's call — it also changes the Rewards banner slot, which currently rides that panel's top edge).
- 🚨 **The mock has no Virtual Account or Convenience Store rows at all — only group headers.** Production (owner screenshot, 2026-08-20) ships **11 VA rows** (BNI · BRI · Mandiri · Permata · ATM Bersama · BSI · CIMB Niaga · Danamon · Maybank · Bank Neo Commerce · BCA) and **2 Convenience Store rows** (Alfamart · Indomaret), every one with a `Minimal Rp 10.000` sub-line. These are the *only* methods that carry minimums, so **the mock cannot draw the below-minimum state at all** — which is why it ended up illustrated on OVO. Drawing them is the prerequisite for both the poin below-minimum state and the §6 edge case.
- ⚠️ **Production also has a no-nominal-selected state the mocks never show:** every price renders `Rp 0`, VA and Convenience Store rows are dimmed below minimum, and the `TERMURAH` badge still sits on ShopeePay. The poin sets all assume a chosen nominal.
- ❓ **Open: what governs group expansion.** One production screenshot shows `Virtual Account` / `Convenience Store` **collapsed with logo chips + `+7 lainnya`**; another, at the same `Rp 0` state, shows **all four groups expanded** with full rows. Both are production. Worth pinning down before the mock commits to one, since the expanded case is what makes the below-minimum rows visible.
- ⚠️ **Four production details are missing from BOTH poin sets, mobile and desktop** — they are not desktop-only gaps, so fixing them is a change to `632:12570` first: (1) **QRIS is its own accordion group** in production, not folded into `E-Wallet dan QRIS`; (2) production lists **six e-wallets incl. `Doku e-Wallet`** and `ShopeePay/SPayLater` is the full label; (3) **collapsed groups preview their logos** (`BNI` `BRI` `mandiri` `Permata` `+7 lainnya`, `Alfamart` `Indomaret`) instead of being bare title rows; (4) the step is numbered **③ `Pilih metode bayar`**, not ② `Pilih pembayaran`. Every poin rule (toggle card, price re-render, disabled DANA, coins-only collapse, cashback strip) is unaffected by all four.

**Still open — desktop wallet `972:9896`:**
- ✅ **RESOLVED (decision 5, 2026-08-03): the balance reads `12.000`.** The card was showing `128.000` against its own equivalence line `Setara dengan Rp 12.000` and a sidebar cell of `12.000` — three numbers for one locked 1:1 peg. **Balance → 12.000**; the equivalence line and sidebar cell are already correct and do not change. One-node fix, still owed in Figma.
- 🚨 **The PRIME bonus row breaks gold = PRIME on BOTH breakpoints — and this doc was wrong about it.** It previously recorded mobile `709:12578` as the correct **gold** reference and desktop as the lone violation. Checked directly 2026-08-03: mobile is tile **`#3A2F52`** with the **default purple** token (`#AB5CEB` / `#D9A6FC` / `#8536C7`). There is no gold on it at all, so the reference implementation this doc pointed at does not exist. Fix both: tile → `#2A2418`, glyph → `Rewards Token / Small / PRIME` `554:1663`. As drawn the row is indistinguishable from `Poin kedaluwarsa`, which is the confusion the law exists to prevent.
- ❗ **The expiry tile is the brightest thing in the list** (`#AB5CEB`) while its amount (`−120`) is correctly the most muted. Icon and amount pull in opposite directions. Rule: pure-poin non-game rows get a **muted** tile, and expired is the most muted row overall.
- ⚠️ **`Pending` is rendered gold** — gold is reserved for PRIME, and here it sits inches from real PRIME gold in the same panel. Either render Pending in muted grey, or document explicitly that amber/gold now means "time-sensitive" and make the two golds visually distinct.
- ⚠️ **Date format split:** ledger rows use full months (`3 Juli 2026`), the balance card abbreviates (`30 Des 2026`), the PRIME pages use `8 Jan 2026`. **Standardise on abbreviated** — it is what the rest of the product uses and it keeps rows compact.

**Still open — PRIME/voucher desktop:**
- ❗ **The voucher count contradicts its grid** — the sidebar cell says `Voucher 8`, the grid shows 10. The fix depends on whether that cell means *available* or *total* — a semantics decision, not a design one.
- ⚠️ `Lihat` vs production's `Pakai` on the voucher button — different verb, different promise. **The live PRIME page uses `Pakai`.** Confirm which wins.
- ⚠️ All four active vouchers are identical duplicates; it reads as a rendering bug in a 2-col grid. Vary the placeholders.
- ⚠️ The history table has no sort affordance — add `aria-sort` and tabular figures on Total while you are in there. **Reverse-chronological is the whole contract of a history table.**
- ⚠️ **472–576px empty voids** on several desktop frames. Needs a call: does the footer ride up, or do the pages get filler? A 4-field form is legitimately short — bringing the footer up is the honest answer.
- ⚠️ `+62` in the phone field — unclear whether that is a value or a prefix placeholder. Needs a separated country-code affordance or a real value.
- ⚠️ `Pending` status is still absent from the desktop history table.

### PRIME subscribe form — four defects, both stages (`1:1156` and `1:1255`, page "2.5")

Identical strings on both, and a dev will copy them:

| # | Defect |
|---|---|
| ① | **`Periode : 20 Jan 2026 - 19 Feb 2025`** — the **end year precedes the start year**. A broken range on the screen where the user commits to a recurring charge |
| ② | **`Rp 20.000`** — PRIME is **Rp 25.000/bulan** on the landing page and in production. The same stale price already fixed ×10 on the PRIME history page; nobody fixed it here |
| ③ | **Three brand casings** — `Langganan RRQ Topup Prime`, `Dengan berlangganan RRQ TOPUP Prime` (the T&C consent line), `RRQTOPUP` in the footer. ① and ② of these were swept 2026-08-03 → `RRQSHOP PRIME`; the footer is library-locked |
| ④ | **`Rp 192,000`** uses a **comma** where the whole site uses periods, and it is Inter Bold where other money on the page is Barlow. Note this is the user's **GoPay balance**, not a price |

Also: 2.0 writes prices as **`Rp. 149.955`** with a period after `Rp`. The 2.5 convention is `Rp 154.955`. Do not carry the old form over.

### Two more traps, both hit during this session

**Hidden text nodes inside CTAs.** Gold buttons in this file frequently contain a *hidden* leftover label (usually `Lanjut Bayar`) alongside the visible one. `findAll(TEXT)[0]` returns the hidden one, so an edit appears to succeed and the button still shows the old label. **Filter on `.visible`.**

**Frames overlapping on canvas.** `Riwayat Poin / Debt` `708:12570` (x8764) and `Riwayat Poin / Hangus` `1127:22108` (x8769) sit **5px apart** — two 375-wide frames overlapping by 370px. Whichever renders on top hides the other. File hygiene, but it will confuse anyone browsing the wallet states.

### Form states — no frames will be drawn (decision 7, 2026-08-03). These rules are the spec instead.

Owner's call: the dev uses house patterns. The problem is that this file documents **no** house form patterns, and the contrast floor has already been broken twice on this exact page — so "house pattern" would otherwise mean "whatever gets invented". These rules are binding in place of frames:

- **Contrast floor applies to every state.** Interactive surfaces ≥3:1, any text the user must read ≥4.5:1. **A disabled control still has to be legible** — this is where opacity 0.32 keeps failing. Use a `#5A5A5A`-class ink at full opacity instead of dimming the whole node.
- **Primary action is gold `#F6AC37`** with a `#0D0D0D` Barlow Black label. `Simpan` is the only action on the profile page — it must not be a `#1F1F1F` pill on a `#1D1D1D` panel (that was a 1.02:1 boundary, i.e. an invisible button).
- **Secondary/quiet action** = `#2B2B2B` fill with a `#5A5A5A` border. That pair is already used on `Berhenti berlangganan` and `Riwayat`.
- **Error strings follow the #19 copy convention:** name the cause before the consequence, and do not apologise. `#ADADAD` on the surface they sit on, which clears 7:1 on every panel in this system.
- **Follow the #19 pattern rule for severity:** a failed *action* gets a dialog; a changed *fact* gets an inline note; a fact the user would infer correctly on their own gets nothing. **A form validation error is a failed action, but it is inline, not modal** — never interrupt a form the user is mid-way through with a dialog.
- **Read-only ≠ disabled, and say which.** The EMAIL field is non-editable, not broken: `#1F1F1F` fill, `#3A3A3A` border, `#ADADAD` ink (~6:1), plus a lock icon and a one-line reason (`Email nggak bisa diubah — hubungi CS kalau perlu ganti`). Silent dimming makes the user think it's a bug.
- **Loading:** disable the action, keep the label readable, do not remove the button from the layout. Nothing in this system may shift height on state change — the same reason both `PRIME Status Block` variants are locked to 62h.
- **Success:** production already uses a **green toast** for a confirmed state change (`✓ Langganan PRIME tidak akan diperpanjang otomatis` on the live `/prime` page). Reuse it. Do not invent a second success language.
- ⚠️ `+62` in the phone field is ambiguous — a separated country-code affordance or a real value, not a bare prefix in the input.

**Still open — PRIME landing:**
- ❗ **M3 — the FAQ chevron y is `h/2 + 10` where centring wants `(h − 20)/2`**, so every row drifts **+20px on mobile / +17 on desktop**, and on the single-line row (`858:18264`, h38) the chevron overflows the row bottom by 11px. Taller rows drift further, so the list reads progressively misaligned. Targets: h38→y9 · h64→y22 · h84→y32 · h104→y42 · desktop h58→y19. Almost certainly fallout from the 07-27 FAQ responsive pass — **check the desktop FAQ panel `883:6642` for the same drift.**
- Row heights 251/237/228 are hand-tuned, not auto-layout, so editing body copy means a manual re-tune.

**🚨 Layer names are NOT unique, and matching on them will break things.** There are at least two different nodes named `Rectangle 4642` in `Pesanan Selesai` alone — one is a 343×297 panel background, the other a 72×9 detail. A `findAll(name === …)` picks whichever comes first. This corrupted a receipt clone during this session's work: the wrong node got resized and the panel background broke.

**Resolve a row's value from its LABEL, never from a name.** Find the `TEXT` whose characters are `Total Bayar`, walk to its parent row, take the sibling `TEXT`. That is stable across clones and refactors. The same applies to any script or Code Connect mapping you write against this file.

**Layer-naming hygiene — this matters to you specifically:** many text layers are still named `Game Name`, and **mobile PRIME card 1 `858:18317` is still named `Selalu ada diskon`** — which *is* the F11 string, so **a grep to confirm F11's removal will return a false hit.** Several CTA frames are named `Pilih pembayaran` and contain a hidden `Lanjut Bayar` text node. Do not trust layer names as specs; read the rendered text.

---

## 8. Shipping code in this repo

### 8.1 Rewards hero motion — 5 files, zero dependencies, zero build step
Full integration doc: **[rewards-hero-motion-handoff.md](rewards-hero-motion-handoff.md)** — read it end to end before touching this; it has 5 host requirements, both presets, the responsive contract, a11y, perf numbers, browser support, **7 traps that each cost real time**, the 6 deliberate deviations from Figma, and a 14-item QA checklist.

| File | Role |
|---|---|
| `rewards-hero-warp.js` | ray-field engine, ES module. Exports `initWarp`, `PRESET`, `PRESET_MOBILE` |
| `rewards-hero-icons.js` | floating-props engine. Exports `initIcons`, `PRESET` |
| `rewards-hero-warp.css` | canvas layer, dome mask, token cut-out, icon layer |
| `rewards-hero-copy.css` | title / subtitle / CTA, desktop + mobile |
| `rewards-hero-warp.html` | **preview harness only — DO NOT SHIP.** 7 layer toggles, 3 slider panels, drag-to-place |

Preview (must be HTTP — `file://` blocks the ES module import as cross-origin; add `?tune=1` for sliders):
```bash
cd "/Users/serling/RRQ Topup" && python3 -m http.server 8080 --bind 127.0.0.1
```

**7-layer stack**, back to front: `Background.png` · ray canvas · `Token.png` · 4 CSS side-glow lobes · `Pattern.png` · 6 floating icons · copy.

**Non-negotiables:**
- **Band height is 445, not 368.** Token.png is bottom-anchored (alpha runs to art row 893 of 894) so cropping to 368 deletes 10.4% of the token; and the vanishing point is band-relative (`cy = H*vpy`), so at 368 the rays converge *inside* the mark. Figma node `870:20218` still needs resizing — owner TODO.
- **Page background must be `#191716`, and the hero's own background must match it.** The art dissolves at the bottom (mean alpha 21.7 on the last row), so the hero background is part of what you see there. Both must read one `--page-bg` var. Deleting that "redundant" declaration puts a visible 13/255 seam band straight back.
- **The title's stroke + inner shadow is ONE inline SVG filter** (`#hero-title-inner-shadow`). It must live in the document — CSS cannot express an inner shadow on text, and Chrome will not resolve a filter from an external file or a `data:` URI. Filter primitives scale off **title font-size**, not band width.
- **The star warp is owner-tuned — do not "fix" it.** 79 rays inward, `bow 0.08 / bowEase 2.55`, polar path. A chord-path model was built and rolled back on the owner's call (still reachable via `arcMode: 'chord'`). Verified: 0px mirror symmetry across 25 pairs, monotonic inward, 60fps.
- **Icon placements are exact Figma measurements, not estimates.** Depth comes from the artwork: `key`/`coin` have maxAlpha 151/178 (drawn faded) so they float slower and shorter. Periods 4.6–6.8s, deliberately non-harmonic so they never re-align. **Icons punch themselves out of the ray canvas every frame** via a `cutout` hook — a static CSS mask can't track a moving shape.
- **Icon PNGs are 2x exports sized from source pixels. Do not resize or re-export at a different scale without updating `w` in the preset.**
- **Mobile is the same art centre-cropped — no new assets.** Hero `883:19494` is 375×290. Breakpoint is **767px on band width** using `@container`, so a hero in a narrow column gets it too. Copy goes **in flow** on mobile so the band can grow (absolutely positioned children contribute nothing to parent height, so the desktop approach overflowed at 320). Type does **not** scale uniformly — title 54→32, sub 16→14, CTA label 16→16 — so mobile is restated, not multiplied.
- Visible CTA label is **`Login Sekarang`** (the layer is *named* `Pilih pembayaran` and holds a hidden `Lanjut Bayar` — ignore both).

**Still open on the hero:** dial `PRESET_MOBILE` on a real phone-width window (count 44, fan 0.95 are reasoned, **not measured** — no mobile reference clip exists) · verify time-based timing on a 120Hz screen (the toolbar shows `fps · field clock /s`; want ~0.96/s at 120fps) · `Pattern.png` will moiré on 1x displays (16px pitch downscaled 2x) · convert `Background.png` to WebP/AVIF (256KB, **but it is the live mask source — verify the alpha survives**) · lift `.layer` / `.glow*` / `@keyframes sideGlow` out of the harness into shipped CSS · **self-host Barlow + Inter** (currently loaded from Google Fonts in the harness; production must not depend on a third party for first paint) · member + PRIME hero variants (deferred) · owner still owes an updated `Token.png` (asked 3×, md5 unchanged).

### 8.2 Rewards nav-tab token motion
Full doc: **[rewards-button-motion-handoff.md](rewards-button-motion-handoff.md)** — production CSS, React component + hook, gating and decay, token SVG, backend flag, QA checklist. Live reference for the **motion character only** (pop timing, rake velocity, easing): `https://claude.ai/code/artifact/2d8bc33e-436d-4378-bb75-2a0ca867435d` — **its 30px dark circle container is obsolete; ignore the chrome, keep the motion.**

The motion was authored for a header circle button that no longer exists, then re-targeted to the nav tab (mobile `Bottom Bar / M` `1026:22496` tab 3 of 5, 75×52, token 24×24 at (25,8); desktop `Header / D` `1020:22750` centre tab, 75×64, token at (25,15)). **The pop and the facet rake survive intact; everything that dies is container styling.**

Four rules the header button never needed:
1. **Never animate while the tab is Active** — the gold caret and gold label already own that state.
2. **Arriving on the Rewards route counts as discovery**, not just tapping the tab — the sandwich poin block, the wallet and deep links all reach it.
3. **Only the nav tab animates.** The token now renders in up to 4 places at once (tab, sandwich poin block, wallet SALDO, nominal pills); glinting them together reads as a rendering bug. **The token component ships inert.**
4. **Glint budget widened 15s → 30s and capped 15 → 8**, because a fixed bottom bar is permanently on-screen and the old cadence spent the whole lifetime budget in under 4 minutes.

Other constraints: all energy in a **<1.8s** intro, steady-state **provably free** (no timer, no RAF, no canvas) · **purple/lilac/white only, never gold** (gold = PRIME) · glint = a gradient clipped by `overflow:hidden`, **NOT an SVG mask** (it smears on low-end Android WebView) · **not Lottie or Rive** (they hold a surface at idle) · `prefers-reduced-motion` → a static lilac dot with `aria-label` suffix **`baru`** (the old spec said English "new" in an Indonesian UI) · **decay flag `rewardsDiscovered` lives on the USER PROFILE**, not just localStorage; session 4+ is silent even if untapped, and a tap retires it permanently · desktop gains an optional pre-discovery hover glint.

### 8.3 E2 "expiring soon" email — complete
Production HTML: **[email-E2-expiring-soon.html](email-E2-expiring-soon.html)** — tables, inline CSS, VML button, dark-mode meta, merge fields, a plain-text alternative, and all build/send notes in the trailing comment. Figma: email-safe build `1004:21751`. ⚠️ The owner-design frame `998:9377` **no longer resolves** — locate by name if you need it.

Hero animation: **`email-E2-hourglass-tick-dropshadow.gif`** (transparent, 230×364 shown at 115×182, 3 frames, exactly 1000ms, 8.2KB) — carries Figma's drop shadow, baked as a uniform 1-in-4 subpixel pattern because GIF alpha is 1-bit. Generator: `email-E2-hourglass-tick-dropshadow.py`.

⚠️ **Asset provenance, read before deleting anything.** Every original source plate (`on_white.png`, `on_black.png`, `hg_full.png`, `hg_empty.png`, `tile480.png`) has been deleted from the repo, so `email-E2-hourglass-tick.py` and `email-E2-token-shine.py` can never run again. Consequences:
- **`email-E2-hourglass-tick.gif` must not be deleted** — it is the only surviving copy of the silhouette, and the dropshadow generator reads it as input. It is also the shadowless fallback.
- `email-E2-hourglass-tick.py` is kept as HISTORY ONLY, with a header saying so. Use the dropshadow generator instead.
- `email-E2-hourglass-tick-shadow.gif` + `.py` and `email-E2-token-shine.py` were **deleted 2026-08-04** as unusable. Recoverable from git.
- `email-E2-token-shine.gif` is **orphaned but kept** — unregenerable, and E1/E6/E7 may want an animated token.

**Owed to whoever sends it:**
- `days_before` = **7**. The wallet flips to amber urgency at ≤14 days, so a 7-day send lands the click-through on the matching amber state.
- **Set a minimum-balance floor.** "40 poin kamu hangus" cheapens the program and burns deliverability — and since E3 was dropped, this is the **only** expiry email, which is also why the exact date is stated rather than "segera".
- **The purple card cannot be CSS** (2 gradients + 3 inner shadows + grain + r28). Ship it as a **background image behind live text** — VML for Outlook, `background-image` elsewhere — and **set `bgcolor="#D9A6FC"` on that cell**, or Outlook's image blocking leaves `#4C1894` text on nothing.
- **Every rounded corner renders square in Outlook.** VML roundrect for the CTA; ship the 24×24 step badges as PNGs; the two big cards are fine square.
- **Barlow won't load in Outlook/Gmail** → Arial Bold. Check the 36px headline still wraps to 2 lines at Arial's wider metrics.
- Social: **Instagram only.** WhatsApp is CS and sits on its own labelled line.
- Bind `{{poin_balance}}` to **`balance.available`**, never `available + pending`.
- ⚠️ **One accepted inaccuracy:** `Buruan pakai poinmu` implies that *spending* saves the balance, but **redemption never extends `expires_at` — only earning does.** Someone who spends part still loses the rest on the same date. The owner chose utilisation framing over preservation deliberately; the extension rule sits in the footer fine print.
- The animation's design logic, if you ever regenerate it: **only the neck stream moves** — a grain crossing the waist is the one cue that reads as "falling"; blinking arbitrary pixels reads as a glitch. Cadence **600ms rest + 2×200ms** — long stillness then a quick discrete change reads as *ticking* rather than a loading spinner. **Frame 1 is the artwork as drawn, so Outlook's first-frame-only rendering degrades to the correct static hero.**
- **Two GIF regeneration traps:** `disposal=2` is **required** for the transparent version (with `disposal=1` the previous frame's grain persists through transparent areas and nothing appears to move); and the flood fill must be **4-connected** — the grain's bottom-left corner is diagonally adjacent to the waist outline, so 8-connectivity leaks out and eats the hourglass.

### 8.4 Logo assets
- `Logo RRQSHOP Vector.svg` — brand file. **Two traps:** the SVG canvas is `1418×743` (aspect 1.909) but the **artwork inside is `1374×568` (aspect 2.420)** — there is ~98px top / 74px bottom / 32px left padding baked in. **Scale to the ink, not the frame**, or the logo lands ~23% undersized and sits low in its box. And **SVG imports carry SCALE constraints on every child**, so any resize or reparent scales them non-uniformly and silently distorts the artwork (this turned the coin into an ellipse twice before it was caught — and the size readout looks correct even when the artwork is distorted).
- `logo-rrqshop-email.svg` — a **trimmed** copy, not the brand file: same paths, `viewBox` cropped to the real ink box `32.27 97.99 1374.29 568.01`. Re-cut the trim if brand reissues the SVG.
- `logo-rrqshop-email@2x.png` — 146×64, shown 73×32, served as the `<img>` fallback inside `<picture>`. **The PNG fallback is not optional:** Gmail, every Outlook and Yahoo strip or fail SVG in image tags — they ignore `<source>` and render the `<img>`, so the PNG is what most subscribers actually see. It is **deliberately not transparent** (`#0D0D0D` baked in, because old Outlook handles PNG alpha badly) — **the bar hex and the PNG backing must stay identical.**
- The email header bar went `#FFFFFF` → `#0D0D0D` because the rebrand vector has **white `#FEFEFE` "RRQ"** — it is a dark-background lockup and would have vanished on white. **If brand supplies a light-background lockup, revert the bar and re-export on white — do not recolour the artwork to fit the bar.**
- ⚠️ Open: the old email logo was aspect **4.4** (single-line horizontal wordmark); the rebrand lockup is **2.42** (coin + stacked RRQ/SHOP), so it now displays 73×32 instead of 132×30 and the bar reads lighter. **A horizontal RRQSHOP lockup does not exist in the SVG** — brand would need to supply one.

---

## 8.5. User Journey maps (Figma page "User Journeys" `1191:12586`)

Seven screen-by-screen flows with labelled arrows, built 2026-08-03. The point is that a dev can see **where a string lands** rather than reading a spec cold.

| Section | Node | Contents |
|---|---|---|
| **S1 — Guest earns first poin** | `1191:12587` | 10 steps: Home → game page w/ new-member strip → login drawer → game page (member) → payment → drawer `state=new-member, pakaiPoin=false` → awaiting payment → Diproses → delivered → wallet |
| **S2 — Member redeems poin** | `1208:14161` | 7 steps: game page → toggle OFF → **toggle ON** → drawer `pakaiPoin=true` → awaiting payment → partial receipt → wallet spend row. **Branches:** coins-only · non-redeemable · gagal-redeem |
| **S3 — PRIME subscribe + cancel** | `1231:14977` | **Entry row (5 routes)** → 10 steps: account upsell → PRIME landing → link GoPay → pay → success modal → account PRIME → game page PRIME → management → unsubscribe dialog → post-cancel. **Branches:** failed renewal · coupon collision · account-row cancelled |
| **S4 — Wallet + expiry** | `1248:16675` | **Entry row (4 routes)** → 6 steps: wallet → Informasi Poin → Panduan Poin → **near-expiry amber** → E2 email → return + top up (expiry resets). **Branches:** empty · debt · all-expired |
| **S5 — PRIME earns 3x** | `1295:18320` | 6 steps + a **Comparison row**: member vs PRIME at the card pill (`+35` → `3X +105`), the drawer earn (`1.489` → `4.467`, bonus `+2.978`), and the ledger (boosted purchase earn vs subscription bonus lot) |
| **S6 — Stale PRIME session** | `1315:30206` | 4 steps: cached PRIME UI → **re-auth bottom sheet** → fork: logged in (`state=prime`) **or** guest (`state=guest`, earns nothing) |
| **S7 — Poin unavailable** | `1353:19821` | Two **lanes**, decided oppositely. **A: game not eligible** — card with no pill → toggle absent → the Informasi Poin safety net. **B: a method that cannot take the remainder** — toggle OFF → toggle ON (prices drop) → row detail. ⚠️ **Lane B is drawn with DANA, which was wrong** (no method is barred from poin, owner 2026-08-20). The lane's *rule* stands; its example must be re-shot as a remainder-below-minimum case |

**S7 carries the reusable rule** the others depend on: *absence is acceptable only when the belief it produces is TRUE.* Lane A says nothing because the user's conclusion is correct and nothing on screen moved. Lane B must speak because the user's own action just re-rendered every price. **The difference is not the missing control — it is whether the user acted first.**

**Conventions:** cards are 375×600 clipping viewports (S6 uses 700) holding a clone focused on the relevant region. Solid `#3A3A3A` = designed in this file · **dashed red = asset missing** · green = the primary route. Gold arrows are user actions; a red arrow marks a broken link. Every caption prints its **source node ID** and the rule binding at that step.

**Entry rows.** S3 and S4 open with one, because both start mid-flow. Worth knowing: in S3 only the **Akun tab** reaches step 1 — the other four routes (header PRIME circle, game-page strip, drawer nudge, guest sandwich) jump straight to step 2, so the PRIME landing must stand alone as an entry.

⚠️ **These are MIRRORS.** Clones do not track their sources. Truth lives at the node IDs printed under each card — edit those, not the journey. Each section carries a banner saying so.

**Layout is cursor-based**, not fixed-offset: each group's position derives from the measured max height of the group above it. Captions can grow without colliding. Verified overlap-free with `clashes: []` on every section. Re-run the reflow if captions change.

## 9. Figma node index

**This is the single source of truth for node IDs** (rule set 2026-08-20). `cashback-design-todo.md` used to keep a second copy of this index; it now points here, and the IDs remaining there are historical session-log entries. If they disagree, this section wins.

File `RWkIM1Q8o5pqt6eK3O8WBe` (RRQ-Topup-2.5) · main page **"RRQSHOP Rewards" `362:12714`** · some sets on page "2.5".

**Chrome (built 2026-07-31)**
| Component | Node | Structure |
|---|---|---|
| `Header / M` | `1014:21813` | 375×106, **one component**, boolean `Show PRIME` (guest false · member true · prime false) |
| `Header / D` | `1020:22750` | 1440×64, variant `state = guest \| member \| prime` (all three differ — the account tab changes too) |
| `Bottom Bar / M` | `1026:22496` | 375×76, variant `state = guest \| logged-in` (member ≡ prime) |
| `Sandwich Menu / M` | `1036:22248` | 271w opened, variant `state = guest \| logged-in`. guest `1033:22285` 350h · logged-in `1034:22238` 424h |
| `Logo / RRQSHOP` | `1017:21865` | vector, final 67.75 × 28 |
| `Header Button / PRIME` | `1012:21825` | 30px `#1F1F1F` circle + 20px gold P monogram |
| `Header Menu / D / Rewards` | `1019:22165` | **local stand-in** — see the library blocker in §10 |
| Rewards tab, mobile | `1025:12597` / `1025:12570` | real variants on the **local** set `92:9393` |
| `Poin Section BG` | `1043:22261` | `#241F2E` band, logged-in only |

**Component sets** — Nominal Card `529:895` · Home tile `556:1274` · Drawer `587:12570` (+ kitchen-sink ref `583:12570`, + **D overlay `1894:26082`**) · Payment M `632:12570` / **D `1888:26536`** · Voucher `641:12570` · Txn card M `825:17404` / D `829:17404` · `Empty State / M` `1120:22067` · **Game-page strip `451:14444` "Rewards"** — guest `451:14418` · member `451:14419` · prime `451:14420` · new-member `1187:23295` · **Game-page strip D `1873:26031` "Rewards / Desktop"** — guest `1873:25964` · member `1873:25979` · prime `1873:25993` · new-member `1873:26016` (793×36, radius `12 12 0 0`, content centred, sits on the step-1 card's top edge)

**Game page** — `Game Page Upsell - M` **`1176:23490`** (guest/normal, carries the new-member strip instance `1188:23303`; step-1 panel `1176:23739`, nominal grid wrapper `1176:23742`, header instance `1176:23797`) · `Game Page Prime - M` `596:16105` (PRIME reference, deliberately untouched)

**Poin glyph component keys** (for instancing) — Default `507:1468` · **Multiply/boost `507:1467`** · PRIME `554:1663`. The boosted pills that consume them: default `448:14187` · events `526:636` · prime `526:637` — container `#29242E`, chip `#263027`+`#325435` stroke (event) / `#393329` (PRIME), amount always lilac `#D9A6FC`, green arrow highlight `#97FF97`

**Wallet** — main `681:16812` · AL wrapper `729:18298` · SALDO card `694:17538` · Riwayat `683:17326` · empty `706:12570` · debt `708:12570` · **all-expired `1127:22108`** · Informasi Poin expanded `732:16577` / collapsed `729:18311` · PRIME bonus-lot row `709:12578`

**Panduan Poin** — drawer-M `725:16668` · modal-D `964:19271` in overlay `964:19269` · pills desktop `961:19344` / mobile `962:19345`

**Account** — M regular `666:3416` · M PRIME `670:3918` · D Ubah Profil member `970:7473` / PRIME `970:9121` · ⚠️ D PRIME Status Active `970:7093` DEAD · D Poin section `972:9896` (balance card `972:10742`) · D Riwayat Langganan `972:12084` · D Voucher PRIME `972:13189`

**PRIME subscription** — management M `967:19481` · unsubscribe confirm `967:19975` · `PRIME Status Block / M` `1089:12588` · `PRIME Account Row` `1097:22032` · `PRIME History Row / M` `1112:12586`

**Landing** — Rewards D `870:19298` / M `887:12619` · hero D `870:20216` (band node `870:20218`) / M `883:19494` · PRIME D `858:17959` / M `858:18217` · FAQ panel D `883:6642`

**Checkout** — set **`632:12570`** `state = on | off | non-redeemable | coins-only | payment-extra` → on `631:12571` · off `631:12570` · non-redeemable `631:12573` · coins-only `631:12572` · payment-extra `631:12574` · **desktop set `1888:26536`** → on `1888:26058` · off `1888:26172` · non-redeemable `1888:26289` · coins-only `1888:26397` · payment-extra `1888:26418` · ~~DANA disabled row `601:12635`~~ **removed 2026-08-20** (the rule was fictional; DANA is a normal row in both `state=on` variants) · payment-extra `621:2204` / `621:2279` · coupon combo `640:12572` (set `641:12570`, `state = discount \| cashback \| combo`) · PRIME-coupon popup `629:15489`

**Order** — awaiting payment M `596:15180` / D `596:14892` · **Pesanan Diproses set `1184:25559` [REMOTE]** (instant `1184:25726` · non-instant `1184:25560` · top-up-with-login `1184:25883`) · receipts: money-only `650:3049` · **partial redeem `1223:23548`** · coins-only `1138:22227` · mobile txn list `767:17597` · desktop txn layout `767:4297`

**Login** — pop-up `Login Drawer` `1184:25281` (375×496, modal) · **re-auth bottom sheet `Login Lagi Drawer` `1330:23953`** (375×549). Two different components — see §5e

**Wallet states** — main `681:16812` · empty `706:12570` · debt `708:12570` · all-expired `1127:22108` · **near-expiry `1272:23548`** (⚠️ debt and all-expired overlap on canvas, see §7)

**Sandwich** — set `1036:22248` · guest `1033:22285` (**now carries `PRIME Row` `1284:23742`**) · member `1034:22238` · `Poin Section BG` `1043:22261`

**PRIME** — landing M `858:18217` / D `858:17959` · subscribe form `1:1156` + `1:1255` (page "2.5", four defects — §7) · **success modal `1184:29401`** · management M `967:19481` · unsubscribe `967:19975` · `PRIME Status Block / M` `1089:12588` · `PRIME Account Row` `1097:22032` · `PRIME History Row / M` `1112:12586` · bonus ledger row `709:12578` (⚠️ purple, should be gold — §7) · pill `526:637` with chip `517:2635`

**Journeys** — page `1191:12586` · S1 `1191:12587` · S2 `1208:14161` · S3 `1231:14977` · S4 `1248:16675` · S5 `1295:18320` · S6 `1315:30206` · S7 `1353:19821`

**Error board** — `1158:23243` (states `1158:23250`, `1158:23463`, `1158:23574`; the cut saldo-berubah frame `1158:23352` is DELETED)

**Colour styles** — **15** local paint styles under `Rewards/`: **Rewards 1–5** (purple scale) · **Surface 1–7** (tinted grounds) · `Green Highlight` · `Near Expiry` · `On Gold` **on top of the 2.0 library's 18** (`Primary`, `Primary Gradient`, `Success`, `Surface 1–8`, … — palette set `383:12517` in `Qge4kS2ihFEm4900iZgzkD`, already bound as REMOTE styles in this file). Swatch board **`1911:26193`** — see §2 for the inheritance table, the deleted duplicates and the red conflict

**Glyphs** — Default/Small `507:1468` · Multiply/event `507:1467` · PRIME/gold `554:1663` · Switch purple `596:16875`

**Footer** — M `1:2568` (includes the marquee) · D instance in `858:18206`

**Read-only, in the RRQ Topup 2.0 library** — nav tab set `99:9986` · `FAQ / D / Closed` `858:17954` · **Pesanan Diproses set `1184:25559`** · legacy logo masters `1:917` · `1:844` · `99:10220`. ⚠️ Two logo-master IDs previously listed here, `767:4235` (8 uses) and `767:4194` (1), **no longer resolve** — the library has moved since they were recorded. The library owner should locate them by usage, not by ID.

**Superseded** — drawer sample `839:12570` is still on canvas and safe to delete. The exploratory guest mobile landing `861:12570` and the absolute txn card set `776:17369` are **already deleted**

---

## 10. Open questions, by owner — none of these are yours to decide alone

### 🚧 Blocked on the Figma library owner (send as ONE request — **three** items wait on the same person)
1. **Add `Property 2 = Rewards`** (and ideally `Login`) to the nav tab set **`99:9986`**. It lives in the RRQ Topup 2.0 library and is read-only from this file — `appendChild` returns `Cannot move node. New parent is a internal, read-only node`. Desktop currently uses the local `Header Menu / D / Rewards` as a stand-in, and Login is faked as an Akun instance with an overridden label on both breakpoints. **Mobile was unaffected** (`92:9393` is local, so mobile got a real variant).
2. **Replace the legacy logo in the header masters** — `1:917`, `1:844`, `99:10220`, plus the two whose IDs no longer resolve (`767:4235`, 8 uses, and `767:4194`, 1). Locate those two **by usage, not by ID** — the library has moved since they were recorded. Note the framing: these are **not** "PNG logos", they are the **pre-rebrand TOPUP vector lockup**, so swapping them is a rebrand of existing mocks, not a format fix. 9 legacy pages were already fixed with one edit to the local master `740:18931` (38 usages resolved to 6 masters; 107 → 67.75 wide at x20 y65).
3. 🚨 **Fix the `Top Up with login` order state** in `Pembayaran Diterima` `1184:25559` (variant `1184:25883`). It tells the user to "Chat CS RRQ Topup" — but the locked nav decision moves Chat into the sandwich and leaves guests with no in-app chat at all, so the one order state that requires support points at a closing door. The string also still says "RRQ Topup" rather than RRQSHOP. Both need the library owner; design cannot reach a remote component.
4. 🚧 **Rebrand the footer** — `Footer / M` **`99:6783`** and `Footer / D` **`99:6947`**. Three strings × 21 instances: `RRQ Topup Link`, `RRQ Topup adalah fasilitas terbaru…`, and the copyright line. The local `Footer / M` `1:2568` on page "2.5" has the same strings but **nothing instances it**, so editing it fixes nothing. ⚠️ The copyright line reads `© 2025 RRQTOPUP by PT MID DIGITAL INTERACTIVE (formerly PT QEON INTERACTIVE)` — whether the trading name changes there is a legal question, not a brand-sweep one.
5. 🚧 **`Search Bar` `648:14739`** — 13 instances of `rrqtopup.com` in the mock browser address bars. Owner has confirmed `rrqshop.com` is correct, so these need updating too.

### 🚨 Product / PM
- **Guests have NO in-app route to PRIME.** The locked nav table puts the guest PRIME entry in the sandwich menu, but the specified menu contents (both variants) contain **no PRIME row**. Header: removed. Bottom bar: never had it. Sandwich: not there. A guest's only path to a paid product is now a direct `/prime` link from ads or SEO. Either add a PRIME row at the **top** of the guest menu (gold, visually separated from the utility group — buried between FAQ and Butuh Bantuan is worse than nothing), or accept ads/SEO-only acquisition as a deliberate call. **Also: guests get no MinBot row and Chat has left the bottom bar, so guests lose in-app chat entirely** and fall back to the WhatsApp CS link.
- **`Cek Pesanan Manual` was dropped** from the sandwich in the 07-31 spec. It exists in the live menu today but appears in neither new variant. Guest order-tracking is the use case (guests have no account). Confirm this is intentional and not an omission.
- **Paying entirely with poin earns zero poin — and therefore does not extend `expires_at` either.** So the most engaged redeemer, the one who spends their whole balance, is the one user who gets no expiry reset. Options: floor the earn base at the pre-redemption amount for coins-only orders, earn on the gross for every order, or accept it. **The answer changes the coins-only success page's copy.**
- **The `gagal` renewal row is a dead end** — decide inline retry vs a banner above the list vs a push/email deep-linking to payment.
- The three PRIME reposition items from §3.1: subscriber notice, T&C update, grandfathering decision, plus verifying 3x poin beats the old discount rate.
- 🚧 **How many cashback events run concurrently, and are they already listed on `/promo`?** This gates #22 entirely (ops/CMS question).
- **Pull Chat-tab usage** before shipping the nav change.

### ⚠️ Owed by Tech
- **Is the method minimum evaluated against the remainder after poin, or against the service price?** The minimum itself is clearly already exposed — production renders `Minimal Rp 10.000` per VA and Convenience Store row and dims them below threshold. The open part is poin-specific: once `coins_to_use` is applied, is availability recomputed against `balance_after_redeem`? The owner has ruled it should be (§1). Confirm the API does it server-side rather than leaving the client to compare, since the client must not hardcode minimums. See §1 API and §5.
- Does `/preview` expose **why** a number changed (event ended vs balance changed vs expired), or must the client diff to infer it? See §6.
- **When does the client re-fetch `/preview`** — on entering the payment step, at submit, or while the payment screen is visible? The third case needs a UI string; the first two need nothing. See §6.
- Does the order system **time out a stalled order**, and does it land on `Dibatalkan` or `Terkendala`? The poin ledger mirrors that answer; it does not set it. (A poin lot must **not** carry its own staleness clock — poin are earned on delivery, so a lot is pending purely because its order is in progress. A parallel 30-day poin timeout would let the ledger and the order list disagree.)

### ⚠️ Owed by the owner
- Resize Figma node `870:20218` from 368 → **445**.
- Export an updated `Token.png` (asked 3×, still byte-identical).
- ~~Strict vs standard on `murah`~~ — ✅ **DECIDED 2026-08-03: STANDARD.** Keep `murah` as a plain adjective, kill `termurah` and every superlative. The 11 audit fixes in §2 already assume this, so that table is unchanged and `murah` stays in titles.
- `TERMURAH` → `HEMAT` tag re-export (CMS image). **Still owed** — and no lint or grep will ever catch this one.
- 🚨 **The new-member multiplier: what number goes on the strip?** `2x` is drawn as a placeholder (§5a). Three ways to settle it, and the copy must not be allowed to decide the economics:
  1. **Keep today's generosity and state it honestly.** Tech confirms the effective multiple per game; the strip says that. In the built drawer it is ~**8×** (11.916 vs 1.489), so `2x` currently undersells by 4×. Needs no backend change *if* the multiple is expressible per game.
  2. **Switch the mechanic to a true ×N multiplier** on the base rate instead of a flat override. Cleanest — the label is then exact for every product-group and the strip needs no selection. Requires a backend change and a decision on N. **If N=2 you are cutting the bonus ~4× from what is mocked today.**
  3. **`minimal 2x poin`.** Ships now, zero engineering, weakest copy — valid only if Tech confirms the override yields ≥2x on *every* product-group.
  Whichever wins, the multiplier is a **CMS display field**, never client-computed — same law as event labels, same reason (member and prime ratios differ).
- **The 5th strip variant** — a member who is logged in but has never purchased needs the `new-member` copy with the Login pill swapped for the member variant's `❯` chevron. One clone; not built.
- **`Game Page Upsell - M` `1176:23490` has a header contradiction.** `Header / M` `1176:23797` carries `Show PRIME = true`, but the frame now shows a guest `Login` CTA — and the locked nav matrix says a guest header is logo · search · sandwich with **no PRIME button**. The frame is genuinely ambiguous: the pill reads `+35 poin` (member-style) and step 5 shows a masked `08***********` (logged-in). Resolve one way: **guest** → flip `Show PRIME` to false and blank the phone input; **member-never-purchased** → keep PRIME true and use the 5th variant above instead of the Login pill.
- ~~Close C1~~ — ✅ **DECIDED 2026-08-03: the balance lives in the sandwich menu's poin block.** No header chip, no tab badge. The 132px header gap stays empty.
- Resize `870:20218` 368 → 445, and the `Token.png` re-export (both still open, listed above).
- ⏸ **Parked: a purple Active state on the Rewards nav tab.** The recommendation is **keep gold** — active colour is one signal shared across all five tabs, and tinting one means "active" loses its single meaning; the purple token already carries Rewards identity in every state. The real counter-argument is that gold is *already* overloaded (PRIME + primary CTA + selected). **If it does go purple: use lilac `#D9A6FC`, not `#AB5CEB`** — the 11px label at `#AB5CEB` on `#191716` is ~5:1, barely AA; lilac is ~2× that, gold ~8:1. And **desktop cannot follow** (Active/Inactive live in the read-only library set `99:9986`), so you would ship purple-active mobile against gold-active desktop until the library owner acts.
- ❗ **Nobody on this project has read the actual RRQ reseller agreements.** [publisher-compliance-copy-rule.md](publisher-compliance-copy-rule.md) is **copy discipline, not a compliance sign-off**, until someone holding the contracts confirms §1. It is strictly safer than what ships today either way.

### Your calls, front-end
- **Target framework was never established** — no application code exists in this directory. Tell us what you're integrating into so the hero CSS/JS and the nav-tab hook can be shaped to it.
- **Does the compliance lint actually land in CI?** The rule only survives the next copywriter if it does. Spec is in the compliance doc §5.
- **C3 earn-preview element — audit before building.** It may already exist inside the built drawer and redeem toggle.

---

## 11. Build order

```
Phase 0 rules  ✅ CLOSED (all of them)
  ↓
Phase 1 core earn/redeem  (#1–#9)          ← compliance i18n sweep starts here
  ↓
account / wallet  (#13–#15)
  ↓
PRIME  (#16–#17)
  ↓
support states + emails  (#19–#21, E1/E4/E6/E7)
```

**If you want the shortest path to something shippable:** the E2 email needs no Figma and no application code. After that, the compliance i18n sweep (11 keys, priority F1–F3 because they are indexed) is a locale-file edit with no design dependency. Then the hero, which is finished code waiting on a host.

---

## 12. Top risks to hold in your head

1. **Guest bounce.** Guests can't earn. The homepage and checkout must convert without feeling like a paywall — which is why the strip carries the condition and the pill carries the lure, and why guest `/preview` returns boosted potential undimmed.
2. **Delivered-to-earn gap.** Poin are not instant. Success and pending surfaces must set that expectation every time, and the tense must be correct ("akan dapat" vs "dapat").
3. **A method that cannot take the remainder.** When redeeming drops the remainder under a method's minimum, that method needs an explicit disabled state **with a reason**, never a silently missing control — the toggle sits *above* the method list, so otherwise the total jumps back up after an action the user just took. (This risk used to be written as a DANA carve-out; that rule never existed.)
4. **Expiry clarity.** Extend-on-purchase is subtle. If the wallet doesn't surface it, the entire retention mechanic is invisible.

## 13. Glossary

| ID term | Meaning |
|---|---|
| poin | the currency unit. 1 poin = Rp 1 |
| hemat | to save (money) — allowed when the saving is named as ours |
| murah | cheap — allowed as a plain adjective; `termurah` is banned |
| kedaluwarsa | expired (KBBI-correct; `kadaluwarsa` is the misspelling to grep for) |
| hangus | forfeited / burnt — used for expiring poin |
| Pakai Poin | "use poin" — the redeem toggle |
| Panduan Poin | the poin guide (drawer on mobile, modal on desktop) |
| Riwayat | history |
| Saldo | balance |
| Nominal | a top-up denomination / SKU |
| Kupon / Voucher | coupon / voucher |
| Terkirim | delivered — the status that triggers earning |
| Diproses / Terkendala / Dibatalkan | processing / problem / cancelled |
| Berhenti berlangganan | unsubscribe |
| Nanti Lagi | "later" — the back-out option |
| WIB | Western Indonesia Time — the timezone all expiry timestamps use |
