This is not just a palette swap.

Programmatic diff says the new [`vs code dark.jsonc`](D:/repos/popular-themes/exported/vs code dark.jsonc#L302) keeps the old [`default dark modern.jsonc`](D:/repos/popular-themes/exported/default dark modern.jsonc#L147) as a base and adds an override layer on top:

- `tokenColors` rules: `69 -> 118` (`+49`)
- selector mentions: `173 -> 260` (`+87`)
- unique selectors: `171 -> 228` (`+57`)
- shared selectors whose final style changed: `26 / 171` (`15.2%`)
- removed selectors: `0`
- duplicate selectors: `2 -> 31`
- avg selectors per rule: `2.51 -> 2.20`

Structurally, the first `65` rules are identical, then the new file injects a big extra block starting around [`vs code dark.jsonc`](D:/repos/popular-themes/exported/vs code dark.jsonc#L850), and the old `token.*` diagnostic rules are still there at the end in [`vs code dark.jsonc`](D:/repos/popular-themes/exported/vs code dark.jsonc#L1241). So the model is “old theme + many extra overrides”, not a clean rewrite.

What changed in targeting:
- It is both broader and more precise.
- Broader: new umbrella selectors like `constant`, `entity`, `entity.name`, `support`, `variable` were added.
- More precise: the new file also adds many narrower selectors and exact re-overrides; duplicate selectors jumping to `31` is the giveaway.

Concrete regrouping examples:
- In [`default dark modern.jsonc`](D:/repos/popular-themes/exported/default dark modern.jsonc#L592), one rule grouped `variable`, `support.variable`, `entity.name.variable`, and `constant.other.placeholder` into one light-blue bucket. In the new file, that gets split across red, orange, blue, and neutral buckets around [`vs code dark.jsonc`](D:/repos/popular-themes/exported/vs code dark.jsonc#L860) and [`vs code dark.jsonc`](D:/repos/popular-themes/exported/vs code dark.jsonc#L968).
- In [`default dark modern.jsonc`](D:/repos/popular-themes/exported/default dark modern.jsonc#L675), `constant.character` shared blue with `constant.other.option`. In the new file, `constant.character` moves to red while `constant.other.option` stays blue.
- `markup.deleted` / `inserted` / `changed` were mostly foreground-only before in [`default dark modern.jsonc`](D:/repos/popular-themes/exported/default dark modern.jsonc#L283). The new file adds grouped diff scopes plus background colors in [`vs code dark.jsonc`](D:/repos/popular-themes/exported/vs code dark.jsonc#L1137), [`vs code dark.jsonc`](D:/repos/popular-themes/exported/vs code dark.jsonc#L1156), and [`vs code dark.jsonc`](D:/repos/popular-themes/exported/vs code dark.jsonc#L1167).
- `string` and `string.regexp` used to live in different old buckets; now both converge to the same light blue, with a more specific regex-escape override layered on top around [`vs code dark.jsonc`](D:/repos/popular-themes/exported/vs code dark.jsonc#L1028).

Color-model wise:
- Old `#569CD6` got split into 6 final buckets.
- New `#79C0FF` now merges scopes that previously came from 4 different old colors.
- Comments moved from green `#6A9955` to gray `#8B949E`, and the selector widened to include `punctuation.definition.comment` and `string.comment`.

I added a reusable script at [`compare_token_colors.py`](D:/repos/popular-themes/compare_token_colors.py). Run it with:

```text
python compare_token_colors.py "exported/default dark modern.jsonc" "exported/vs code dark.jsonc"
```

One caveat: these counts compare declared selectors. Actual runtime impact can be wider than `26 changed shared selectors`, because new generic selectors like `constant` and `entity.name` can affect scopes that were not explicitly listed before.