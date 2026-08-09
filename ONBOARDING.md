# Sparkle — New User Journey: Project Onboarding

This document exists so a new person — designer, PM, or engineer — can pick up this
project cold and understand what it is, how it's built, and how work actually gets
done here. It's written from the real history of this repo (commits, PRs, reverts),
not from a plan — so it teaches the *actual* working method, warts and all.

## 1. What this project is

**Sparkle** is a clickable, front-end-only prototype of a jewellery-store retail
assistant app — built for **Indriya** (a gold/jewellery retail brand). It's the tool a
Jewellery Consultant ("JC") uses on a tablet/kiosk while walking a customer through a
store visit, end to end:

1. **Capture / recognise the customer** — phone number lookup; returning customers get
   a "Customer 360" profile (loyalty tier, points, active savings scheme, celebrations,
   recently viewed); new customers or walk-in guests can skip and browse anonymously.
2. **Browse / find product** — a catalogue with filters, serial-code search, barcode
   scan, and "search by image."
3. **Product Detail Page (PDP)** — price breakdown, stone information, like/dislike/wishlist.
4. **Estimate / session tray** — running total across multiple pieces (sub-total, GST,
   grand total).
5. **Billing** — customer/billing details, PAN capture (mandated above ₹2,00,000 per
   Indian tax rules), and **tenders** (loyalty points, savings schemes, advance payments).
6. **Verify & Redeem** — OTP + consent gate, then an animated "Done" confirmation.
7. **Home** — a log of the day's sessions (ongoing / completed, purchased or not).

There is **no backend, no database, no real auth** — this is a design/product
prototype, not production software. All "data" (customers, loyalty points, catalogue)
is hard-coded JavaScript inside the HTML file itself.

## 2. How the codebase actually works

The entire app is a handful of **single, self-contained HTML files**. Each file
contains its own `<style>` and `<script>` — everything needed to run it, with no
build step, no `npm install`, no framework. Open the file in a browser and it works.

Why this shape:
- It's a prototype meant to be shared instantly (email a file, or open a raw GitHub
  URL) — no deploy pipeline needed for someone to click through it.
- It made **parallel iteration** trivial: a whole alternate version of the app is
  just another `.html` file, safe to diverge freely without touching the version
  everyone else is looking at.

The cost of that shape, which you should expect:
- No shared component library — a button/chip/color fixed in one file has to be
  **manually ported** to every other file that needs the same fix (you'll see many
  commits literally titled "...(both index.html and sparklequicksession_3.html)").
- Images are embedded as base64 `data:` URIs (fonts are the only external
  dependency, loaded from Google Fonts) — so files are large (1–1.3MB each) and a
  binary/image asset can't be diffed meaningfully in a PR.
- No automated tests. Verification is manual: drive the flow in a headless browser
  and confirm there are no JS console errors and the right screens/states appear
  (see §7).

## 3. File map

| File | Role |
|---|---|
| `index.html` | The **main / canonical app** — the reference implementation other versions get aligned back to. Has the Home page (session history). |
| `sparklequicksession_2.html` | **"V1"** — the original quick-session flow (no Home page, no product-search-first screen). Kept untouched as a baseline once newer iterations branched off. |
| `sparklequicksession_3.html` | **"V2"** — adds a dedicated Product Search screen (with a Product View / Catalogue View toggle), the full billing→verify→done flow, and richer tenders. Iterated on heavily and repeatedly aligned to `index.html`'s chrome. |
| `sparklequicksession_4.html` | **"V3"** — a copy of V2 with a **lighter onboarding**: drops the upfront preference-picking step in favour of a single number-capture screen. |
| `sparklequicksession_5.html` | **"V4"** — a copy of V3 with a **catalogue-first** shopping experience: a single catalogue page with search/scan/image built into the top bar and a pop-up session tray, instead of a separate search screen. |

Read this table as a timeline, not a ranking: each new numbered file is a **fork of
the previous one**, created to test a different idea for the same step of the
journey, while the "losing" ideas stay available for comparison rather than being
deleted.

## 4. The versioning / iteration model ("multiple branches, multiple iterations")

Two different, easily-confused mechanisms are both called "versions" here — keep
them straight:

**a) Parallel concept files** (the real "iterations" in the product sense). When the
team wanted to try a materially different idea for a screen or flow — "what if
onboarding was one page instead of two?", "what if you shop from a single catalogue
instead of search-then-catalogue?" — the move was: **duplicate the current best
file under a new name**, then diverge. This is why `sparklequicksession_5.html`
exists instead of a `search-first` boolean flag inside one file. It keeps every
concept clickable and comparable side by side, at the cost of manual duplication
(see §2).

**b) Git branches, in the normal engineering sense.** Almost all day-to-day work
happened on one long-lived branch, `claude/sparkle-new-journey-changes-dom04e`,
merged into `main` through **small, single-purpose pull requests** — one PR per
fix/feature, usually with a single commit, e.g.:

- PR #2 — "Customer 360 chip: unified stroke + stronger hierarchy"
- PR #6 — "v2 journey polish: money formatting, copy, autofocus, back-nav"
- PR #9 — "Billing: recognize known number; remove signature step"

Later work (the V3/V4 forks, background-motif experiments) happened as direct
commits to `main` in tight loops — small change → merge commit → next small change
— rather than long-lived PRs. Either way, the pattern held: **keep each change
small and reviewable, merge often, don't let branches drift far from `main`.**

## 5. Reverts: the other half of "iterating"

Not every idea survived. When a design direction was tried and rejected on review,
the fix was a plain `git revert`, not a manual undo:

- *"V3 customer-360: replace scattered motifs with soft Indriya-style floral
  damask"* → reviewed → **reverted** in the next commit, back to the scattered
  jewellery-motif background.
- *"V3 app bar: drop Indriya Circle from chip, tighten clock spacing..."* → **reverted**.

Lesson: land the experiment as its own commit (don't fold it into something else),
so that if it's rejected, undoing it is a one-line `git revert <sha>` instead of a
manual, error-prone unpick.

## 6. Design-to-code: "copying the screens"

Screens were not eyeballed from a screenshot — they were built against **Figma as
the source of truth**, referenced directly in commit messages by node IDs (e.g.
*"Add a blue gradient 'Search' button ... per Figma 15097-8186"*). The practical
workflow:

1. Pull the exact layout, spacing, and colour tokens from the Figma frame.
2. For real artwork (the hero background image, the gazelle line-art), **export
   the asset from Figma and embed it directly in the HTML** as a `data:` URI —
   e.g. a 15MB source PNG was downscaled and recompressed to a ~240KB WebP data
   URI so the single-file constraint held (see the commit *"embed hero background
   image; remove raw asset"*).
3. Reuse the same small set of design tokens everywhere rather than
   reinventing them per screen: navy `#0A2A66`, gold `#B89149`, cream
   `#FBF8F3`/`#E9E4DB`, hairline borders `#ECE5DA`, `Playfair Display` for
   headings + `DM Sans` for body text.
4. When two files needed to look identical (e.g. the app bar, the customer chip),
   the newer file was explicitly **aligned back to the older/canonical one** rather
   than the other way round — `index.html` and "V1" acted as the style anchor.

If you're picking up new screens: find the Figma frame first, match its exact
spacing/type/colour, then decide whether the asset can be recreated in CSS/SVG or
needs to be embedded as an image.

## 7. Flow optimisation — how steps got cut, not just polished

A recurring pattern across this project's history is *removing* friction, not
just prettifying it. Concrete examples worth learning from:

- **Removed a whole onboarding step.** V3 dropped the manual "What brings them
  in?" preference-picking screen entirely — the assistant now goes straight to a
  single number-capture screen; preferences are meant to be inferred passively
  from browsing instead of asked upfront.
- **Removed the signature-capture step** from Verify & Redeem — redemption now
  gates on OTP + consent only, because the signature added friction without
  adding real verification value.
- **Collapsed two screens into one.** New-customer detail fields (name/email/
  birthday/anniversary) moved from a separate screen to inline fields on the same
  "save the visit" page.
- **Ergonomics for the person actually using it**, not just the customer:
  auto-focusing the serial-search field when the screen opens, so a JC on a
  tablet doesn't need an extra tap.
- **Fixing navigation state, not just adding buttons.** The PDP's back button
  target (search vs. catalogue) is *contextual* — it remembers where you came
  from and resets on re-open, so it never sends you somewhere stale.

The underlying skill: when a step exists, ask whether it can be removed or merged
before you polish it. Several of these changes shipped explicitly as "per client/
JC feedback" — i.e. real usage friction reported back, not internal guesses.

## 8. Domain research baked into the product

Because this is an India-market gold-jewellery product, a fair amount of domain
research shows up directly in the code and copy:

- **GST at 3%** on jewellery, shown as its own line in the estimate (sub-total /
  GST (3%) / grand total).
- **PAN card capture required above ₹2,00,000** per Indian KYC rules for large
  cash/jewellery transactions — modelled with a simulated government-API failure
  and a manual-scan fallback.
- **Indian digit grouping** ("lakh" formatting, e.g. `40,98,545` not `4,098,545`)
  — this was actually broken once (an escaped regex bug, `\\B`/`\\d` instead of
  `\B`/`\d`) and had to be fixed; worth remembering if money ever renders wrong
  again.
- **Savings-scheme codes** — `GGP` (Gold Growth Plan) / `GQP` (Gold Quick Plan) —
  modelled as a code→name lookup, only shown when the customer has an active
  scheme.

If you're extending this product, treat these as real constraints, not flavour —
they came from how Indian jewellery retail actually operates.

## 9. How changes were verified (no test suite)

There's no Jest/Playwright test suite in this repo. Verification was manual and
consistent, and every PR description states it explicitly:

> *"Driven end-to-end in a headless browser (no JS errors): search → PDP →
> estimate → billing → tenders → OTP/signature → redeem → done."*

The recipe, if you're making a change:
1. Walk the **full path(s)** your change touches, end to end, not just the one
   screen you edited — regressions liked to hide in navigation state (back
   buttons, reset-on-open flags).
2. Check **both the returning-customer and walk-in/guest paths** — they diverge
   early (recognition card vs. inline new-customer fields vs. no-tenders empty
   state) and a fix for one can silently break the other.
3. Confirm **no JS console errors** — with no framework and no tests, a silent
   `TypeError` is the only signal something broke.
4. If a screen exists in more than one file, check it in **every file** it's
   duplicated into.

## 10. Deployment

A GitHub Pages workflow was added to publish `index.html` (V1) and
`sparklequicksession_3.html` (V2) to a stable, auto-updating URL for easy
sharing/review — then **removed** shortly after, because the `github-pages`
environment is locked to the default branch and couldn't serve a working branch.
If you need a shareable preview URL again, use GitHub Pages' own "Deploy from a
branch" setting rather than a custom Actions workflow, or just share the raw file
link / open it locally.

## 11. Skills & lessons this project actually taught

- **Duplicate-to-fork is a legitimate iteration strategy** for a no-build,
  single-file prototype — it trades DRY-ness for the ability to keep every
  concept alive and clickable at once. Know when you're in that mode vs. when
  you're accumulating unmanaged drift.
- **Small, single-purpose commits/PRs merge cleanly and revert cleanly.** Nearly
  every PR in this repo's history does exactly one thing; that's why the revert
  commits worked as a one-line undo instead of a manual unpick.
- **Alignment passes need a stated "source of truth."** Whenever two files had to
  look the same, the commits are explicit about which file is canonical and which
  is being brought into line — never "let's average the two."
- **Design fidelity means pulling from Figma by node ID, not eyeballing a
  screenshot** — and for real artwork, embedding the actual exported asset
  (compressed appropriately) rather than approximating it in CSS.
- **Cutting a step is often the highest-leverage UX fix** — several of the
  biggest usability wins here were deletions (a screen, a field, a signature
  step), not additions.
- **Manual, scripted verification can substitute for a test suite** *if it's
  actually run the same way every time* — end to end, both customer paths, every
  duplicated file, checked for console errors. The discipline is in doing it
  every time, not the tooling.
- **Domain research has to show up in the code, precisely** — a wrong GST rate,
  a wrong PAN threshold, or a broken digit-grouping regex isn't a cosmetic bug in
  this domain; get the specifics from the client/domain expert, not assumptions.

## 12. Glossary

| Term | Meaning |
|---|---|
| JC | Jewellery Consultant — the staff member using this app with a customer |
| PDP | Product Detail/Description Page |
| Customer 360 | The returning-customer profile view: tier, points, scheme, celebrations, recently viewed |
| Tender | A way a customer can pay/redeem value: loyalty points, a savings scheme, or an advance payment |
| GGP / GQP | Gold Growth Plan / Gold Quick Plan — named jewellery savings-scheme types |
| Walk-in / Guest | A customer who declines to share a phone number; browses without a profile |
| Estimate / Tray | The running selection of pieces and their running total for the current session |
| "V1" / "V2" / "V3" / "V4" | Informal names for `sparklequicksession_2/3/4/5.html` respectively — see §3 |

## 13. Getting started checklist for a new joiner

1. Open `index.html` in a browser and click through the full flow once, as a
   returning customer (use a number the code recognises) and once as a guest.
2. Skim `git log --oneline main` — the commit messages are unusually descriptive
   and are the fastest way to absorb "why" a given piece of UI looks the way it does.
3. Before touching a screen, check whether it exists in more than one file
   (§3) — plan to port your fix to all of them, or explicitly note why you didn't.
4. Find the Figma frame for whatever you're changing before writing CSS by eye.
5. Land your change as its own small commit/PR so it's revertable on its own if
   it doesn't land well.
6. Verify with the checklist in §9 before calling it done — there is no CI to
   catch you.
