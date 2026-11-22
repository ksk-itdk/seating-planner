# Bordplan (seating planner) – notes for AI agents

Single static page, no server/API. All data lives in the browser's localStorage key `seatplan`.

## JavaScript API (run in the page)
- `bordplan.get()` – full state object
- `bordplan.set(state)` – replace state (then re-render)
- `bordplan.arrange()` – auto-seat everyone; returns `{ok, issues, seats, guests}` or `{error}`
- `bordplan.check()` – `{issues:[...], seats, guests}`
- `bordplan.seating()` – `[{guest, table, seat}]`
- `bordplan.go('tables'|'guests'|'rules'|'plan')` – switch step

## State format
```json
{"event":"Name",
 "tables":[{"id":1,"name":"Table 1","shape":"rect|round","side":4,"endL":1,"endR":1,"x":0,"y":0,"rot":0}],
   // optional "side2": seats on the bottom long side of a rect table (omit = same as "side")
 "guests":[{"id":2,"name":"Anna","type":"adult|kid","parents":[ids],"pmode":"next|table"}],
 "rules":[{"id":3,"a":2,"b":4,"or":[5],"type":"yes|across|table|no|nottable","kids":true}],
 "assign":{"1:T0":2}, "locked":{"1:T0":2},
 "settings":{"across":false,"floor":false}, "nextId":6}
```
`side` = seats per long side (rect) or total seats (round). Seat keys: `<tableId>:T|B|L|E|R<index>`.
`or` = alternatives (rule satisfied by b OR any in or). `kids` on a `yes` rule lets the pair's own children sit between them.
`settings.floor` = room layout: touching long tables are joined.

## Share
`#d=<base64url(deflate-raw(JSON state))>` opens a plan from a link. UI languages: Danish (default) and English (menu).

## Guided setup (event mode)
- Pitfall: the guest sheet's event branch is nested inside `${CM?`...`:`...`}` — never escape its inner template literals (an escaped version shipped as raw text until v45).
- Code in part `4b_guide.html` (built between 4_ui and 5_plan). View `guide` (`vGuide`), state `S.guide={step:names|kids|groups|style|conflicts|sum|done,i,groups,cur}`.
- New/empty event plan (no `S.guide`, no guests) opens in the guide; plans with guests and no `S.guide` open as before. Never in class mode.
- Steps: names (one per line/comma, `*` = child) -> children pick 1–2 parents -> one card per adult: tap who they don't get along with, then choose "Not next to each other" (`no`) or "Not at the same table" (`nottable`) -> summary -> Tables.
- Social layer (v42+): guests may carry `group` (who-knows-whom circle) and `style` (`talk|quiet`; undefined = easy-going). Couples typed as `A & B` / `A og B` / `A + B` become a `yes` rule with `couple:true`. `social(where,seats,TB,hit,assign)` in 3_engine is called (as a statement — it adds via hit(); never `cost+=social()`) only outside class mode when someone has group/style: lonely guest (has group, nobody known beside/across) 12, quiet between two talkative non-acquaintances 8, talkative spread over tables. Its issues start with `Tip:` and are shown as yellow "suggestions", not counted as problems in badge/toast. Rule type `soft` = "preferably not next to" (weight 15).
- Everything is stored as normal guests/rules, so the regular screens edit it afterwards. "Advanced" skips to the regular screens; restart via the "Guided setup" button on the Guests screen or in the Menu. Start over / New event use the in-app `askOk()` dialog (native confirm() can be blocked in embedded browsers); Start over resets to blank()/blankClass() and opens the guide.

## School class mode
- Menu → Plan type: Event (default) / School class. Stored in localStorage `bp_mode`; each mode has its own plan (`seatplan` / `seatplan_class`); switching reloads.
- Global `CM` (bool), `MODE`. Students: `type:'adult'`, `level`: `strong|mid|support` (`LV` map has labels + colours green/amber/blue).
- Scoring (evaluate): support student without strong neighbour = 35 (20 if a middle neighbour); two support neighbours = 15; two strong neighbours = 6 (spread helpers). Social ties use the normal rules (next to / not next to).
- New class plan starts with 6 groups of 4 (two facing pairs). Bulk paste: `Name+` strong, `Name-` needs support.

## Kids table
- Table field `"kids": true` marks a kids table (any shape). Adults there are penalised; so are children not set to kids table.
- Child field `"pmode"`: `"next"` (next to a parent, default), `"table"` (same table as a parent), `"kids"` (sits at a kids table; falls back to "table" if no kids table exists). Use "next"/"table" for children too small to sit away from their parents.

## Design rules
Theme: dark is the default; light via menu (localStorage bp_theme, html[data-theme=light]). Inspired by the Philips Hue app: neutral surfaces (light #f2f2f4/white cards; dark pure black/#1c1c1e), system sans-serif (system-ui), bold tight headings, 18px card radius, pill buttons, one warm amber accent (#ff9f2e, dark text on it), tables drawn as warm light-colour gradients like Hue room cards.
Icons: Lucide (ISC license), inlined as `IC` map + `ic(name)` in the engine part. No emoji as icons. Add new icons from lucide-static, never hand-drawn or emoji.

## Class mode desks (v48+)
- Desk = rect table with `desk:true, dir:0-3, side:N` -> N seats on ONE side only (key 'B'). dir 0 = seats below desk, students face the board at the top; each +1 turns 90° clockwise. Geometry in tableGeom's `t.desk` branch.
- New class plans default to `deskLayout(3,4)` (3 rows x 4 two-person desks, floor mode on). Tables page has a "Standard classroom" card (rows/desks per row -> buildClass, askOk, clears seating).
- Move mode: drag = move one desk; dropping within 26px of another table snaps flush (snapTable) so they join; tap (<6px) = rotGroup(): turns the desk and everything joined to it 90° around the group centre, then slides the group to the nearest free spot if it overlaps. A green "Board/Tavle" bar is drawn at the top in class floor mode (fixed).
- Pitfall: code run while `S` initialises (blankClass/deskLayout) must not use `U`/`D` (const declared later -> TDZ, page dies with "S is not defined").
- Old class plans (two-sided group tables) are converted once by migrate() to one-sided desks (S.settings.desks=1; seating cleared). Class mode always uses the floor view, so the board shows on the plan overview and in print.
- Class mode plan view: desks are always draggable (no move mode needed; drawing(...,moveMode||CM)) and each has a .trot turn button (data-rot -> rotGroup, handled first in pointerdown). Tap on a desk only turns it in move mode (event mode). .vert puts the button at the bottom of turned desks.
- Class wording: L() runs every UI string through CLS regex swaps when window.__CM (guest->student, event->class, gæst->elev, begivenhed->klasse, bordkort->navneskilte). Strings that must stay literal in class mode (the Event/Begivenhed option in the Plan type switch) bypass L(). Class icons: tabs layout-grid / graduation-cap / handshake / school, board label 'presentation'.
- Direct links: ?klasse / ?class / ?mode=class opens class mode; ?fest / ?event / ?begivenhed / ?mode=event opens event mode (saved to bp_mode before MODE is read). setMode() navigates to ?klasse or ?fest so the URL always matches the mode.
- Link language: ?class/?event (or lang=en) force English, ?klasse/?fest/?begivenhed (or lang=da) force Danish. setLang/setMode navigate to the matching link so the URL never fights the menu.

## Class guide & levels (v67)
- Class mode has its own guided setup (vCGuide in 4b_guide.html): names → level → seat needs → relations → rotation → summary.
- Levels are High/Middle/Low (keys strong/mid/support). Pairing follows the Danish SYKL rule: neighbouring levels; high next to low costs 30, same-level pairs cost a little.
- g.front = "near the board" (front third of seats by y), cost 40.
- S.history (last 8 saved neighbour pairs) + S.settings.fresh: repeats from the last 4 cost 6 (Tip). S.settings.rot = weeks; the next change is shown as a Wednesday.

## Credit & privacy (v68)
- Footer + Menu → Privacy & cookies (openPrivacy). Credit: KSKIT https://github.com/ksk-itdk. No cookies, no third-party requests, data only in localStorage, share data in URL hash.

## License
MIT, © 2026 KSKIT. LICENSE file + notice at top of index.html + footer link. Keep it when editing.
