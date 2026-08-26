# Filling Out `Medica-Wiki-Template-SIMPLE.html`

This guide tells an AI agent how to add new entries to the Materia Medica template without breaking the page. Only two JavaScript objects need to be edited: `RECEPTORS` and `HERBS`. Everything else (CSS, HTML skeleton, render/modal logic) is generic and must not be touched.

## Where to edit

Open the file and find the `<script>` tag. Inside it:

1. `const RECEPTORS = { ... }` — a glossary of receptor/target entries, keyed by ID.
2. `const HERBS = [ ... ]` — the array of item cards. Each item references `RECEPTORS` keys.

Add new entries to both. Never rename or remove the functions below `HERBS.sort(...)` (`gaugeTicks`, `receptorTable`, `cardHTML`, modal logic).

## Step-by-step process

**1. Gather the raw facts for the item first.** Name, Latin/scientific name, whether it's psychoactive, a potency estimate 0–10, primary effects, key active compounds, and — for each compound — which receptors/targets it hits and what's known about affinity (Ki, IC50, or a hedge like "not quantified").

**2. For each receptor/target hit, decide: does a matching `RECEPTORS` entry already exist?**
- Match on the *combination* of target + activity type (e.g. "GABA-A" + "Positive allosteric modulator" is a different entry than "GABA-A" + "Antagonist").
- If it exists, reuse its key — don't duplicate.
- If it doesn't exist, add a new entry to `RECEPTORS` before you reference it.

**3. Write the `RECEPTORS` entry** (skip if reusing an existing key):

```js
"unique_key_here": {
  title: "Human-readable receptor/target name",
  activity: "Short label, e.g. Agonist / Antagonist / Inhibitor / Positive allosteric modulator / Modulator",
  subtitle: "Endogenous ligand(s) — receptor/molecule class (e.g. 'Gq/11-coupled GPCR')",
  body: "Paragraph 1: where it's located and what it normally does.\n\nParagraph 2: what happens functionally with this specific activity — effects and risks."
}
```

- Key format: `TargetName_activityType` (e.g. `GABAA_PAM`, `5HT2A_agonist`, `MAO_inhibitor`). Must be unique and must exactly match what you reference from `HERBS`.
- `body` uses a literal `\n\n` between the two paragraphs (not a real line break) — keep it as one JS string.
- Keep tone clinical/neutral, matching the existing entries. Don't include dosing or acquisition info.

**4. Write the `HERBS` item entry:**

```js
{ name:"Common Name", latin:"Scientific name", psychoactive:true|false, potency:0.0,
  effects:"Comma-separated effects/traditional uses, one sentence.",
  compounds:"Comma-separated list of key active compounds.",
  substances:[
    { name:"Compound Name", hits:[
      { key:"receptors_key", receptor:"Name as shown in table row", activity:"Short activity phrase", affinity:"Ki/IC50 or hedge like 'Not quantified'" }
      // one object per receptor this compound hits
    ]}
    // one object per named active compound
  ] }
```

Rules:
- `potency` is a number 0–10 (10 = most potent/psychoactive). The list auto-sorts high → low; you don't need to place it manually.
- `receptor` and `activity` inside a `hits` object can be worded slightly differently from the matching `RECEPTORS[key].title`/`.activity` (e.g. adding a parenthetical), since the table row and the modal are allowed to say slightly different things.
- If the mechanism is completely unknown, use `key: null` and set `receptor`/`activity`/`affinity` to hedge text, e.g.:
  ```js
  { key:null, receptor:"Unknown / unconfirmed", activity:"—", affinity:"No data located" }
  ```
- If an item has literally no compound/receptor data at all, set `substances: []` — the page will automatically render an "unconfirmed mechanism" placeholder.
- Every `key` you reference **must** exist in `RECEPTORS`, or that row will silently render as non-clickable.

**5. Insert both additions:**
- Add the new `RECEPTORS` entry inside the existing `{ ... }`, comma-separated from the entries already there.
- Add the new `HERBS` entry inside the existing `[ ... ]`, comma-separated from the entries already there.
- Don't touch `HERBS.sort(...)` or anything after it.

## Validation checklist before finishing

- [ ] Every `key:` value used in any `hits` array exists in `RECEPTORS` (or is `null`).
- [ ] No duplicate `RECEPTORS` keys.
- [ ] `potency` is a plain number between 0 and 10.
- [ ] `psychoactive` is `true` or `false` (unquoted boolean).
- [ ] JSON-like syntax is valid JS: commas between array/object items, no trailing comma after the last item in `RECEPTORS` or the last item in `HERBS`.
- [ ] All strings use straight double quotes and don't contain unescaped double quotes.
- [ ] `body` strings use `\n\n` (escaped) for the paragraph break, not an actual newline that would break the JS string.

## What not to do

- Don't add new fields to the item or receptor objects — the renderer only reads the fields listed above.
- Don't change card/table/modal markup or CSS to "customize" a single entry; keep all entries structurally identical so the page stays consistent.
- Don't include dosing, sourcing, or acquisition information — stick to effects, compounds, and receptor pharmacology, matching the tone of the existing examples.
