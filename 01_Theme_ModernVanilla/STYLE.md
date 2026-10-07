# Homestead PDA: vanilla skin

A reskin of Homestead's Settlements tab so it reads like the rest of the PDA. It starts from
dynz's vanilla reskin (Homestead_GAMMA_ui.zip, 2026-10-03) and keeps every element name,
function and string of Homestead 1.1.8, so new Homestead features drop in unchanged.

## Files

| file | what it holds |
|---|---|
| `scripts/ui_stalker_camp_builder_theme.script` | **new.** Every color and widget texture by role, and the helpers below. Change the look here. |
| `scripts/ui_stalker_camp_builder_pda.script` | Homestead's tab, asking the theme for colors and textures instead of literals. |
| `configs/ui/ui_stalker_camp_builder_pda.xml` | the layout; its colors are the theme's palette values. |
| `configs/ui/textures_descr/ui_pda_stalker_camp_builder.xml` | adds `ui_homestead_btn_on_*` (the vanilla button, pressed look) and `ui_homestead_rule`. |
| `textures/ui/homestead_pda/ui_rule.dds` | a 4x4 gray square for the 1 px rules. |
| `textures/ui/app_settlement.dds` | MAC's Settlements tile, redrawn: Homestead's tent, house and fire as line art on a transparent ground (light gray, amber when highlighted, white when touched, dim when disabled), like the tiles beside it; same atlas layout as `ui_app_settlement.xml`. The old states had opaque backgrounds (olive on white, neon green on black). |

## The palette

| role | color | for |
|---|---|---|
| `heading` | 238,196,112 | section headings, names, what is selected |
| `body` | 200,200,200 | body text |
| `bright` | 255,255,255 | a value or a term that matters |
| `dim` | 160,160,160 | what is off, secondary labels |
| `good` | 56,209,115 | friendly, secured, it went well (the PDA's `pda_green`) |
| `warn` | 238,155,23 | needs attention (the PDA's `ui_7`) |
| `danger` | 238,28,36 | hostile, destructive (the PDA's `pda_red`) |

Amber headings and gray body are the Anomaly PDA convention; Seasons of the Zone's pages use
the same. Faction colors for rival camps live in the theme's `factions` table.

## Layout conventions

- A section is a heading (letterica18, `heading`) with a 1 px `ui_homestead_rule` under it.
  No card or bar behind it.
- Body text is letterica16 in `body`; emphasis is `bright`, not a new color.
- Every button is the base game's `ui_button_ordinary`. A selected tab, filter or job shows
  the pressed look (`hs_theme.set_active(btn, true)`).
- Dialogs use the base game's message box (`ui_inGame2_message_box`), edit box
  (`ui_inGame2_edit_box_2`) and Cancel-then-OK, like the stash-naming dialog
  (`ui_items_backpack_16.xml`). A dialog shown with `ShowDialog` is placed on the
  1024x768 screen, not inside the PDA: center it there.
- Text strings say what they mean in sentence case; color comes from the theme, not from
  inline codes. Inline codes already in strings are mapped by `hs_theme.recolor`.

## Engine facts that shaped this

- A `CUI3tButton` sets its label color from the XML `<text_color>` (`e`, `h`, `t`, `d`,
  white by default) every frame. `btn:TextControl():SetTextColor(...)` never shows, and a
  plain `<text r g b>` on a button is overwritten too. Give a button its label color in
  `<text_color>` (see the danger buttons); show state with the texture.
- An inline color code needs four numbers, `%c[a,r,g,b]`, or a name from `color_defs.xml`
  (`%c[pda_green]`). With three numbers the engine silently keeps the element's default color
  (`CUILines::GetColorFromText`), so Homestead's three-number codes (the "Ambushed",
  "Defeated", "Recovered", "Deposited" highlights and the resets after faction names) never
  showed. `hs_theme.markup` writes four, and `hs_theme.recolor` rewrites three-number codes.
- A texture id must be declared in a `textures_descr` file. `ui_inGame2_pda_line_*` are not
  declared in GAMMA (the log says `Can't find texture`), which is why the rule is shipped.
- `ui_button_ordinary` comes from `ui\ui_common`; GAMMA's UI pack restyles that file, and
  `ui_homestead_btn_on_*` point into it, so the selected look follows the pack.
- A text with `complex_mode="1"` wraps. With `vert_align="c"` the wrapped lines grow up and
  down from the middle of the box, over whatever sits above it: the overview's region line
  covered the camp name that way. Top-align anything that can wrap, measure it
  (`AdjustHeightToText`, then `GetHeight`), and place what comes under it from that.
- A button's label never wraps; what does not fit runs past both edges. Keep labels short
  (`hs_theme.short` drops a trailing "(...)").
- A line break inside a text is the two characters `\n` (`"\\n"` in Lua source), as the
  guide's own text already does.
- A scroll view measures its content when a window is added or removed (and on Clear),
  never when one changes size, and Lua cannot force it. The survivor detail is one window
  resized per layout, so the tab's Update re-adds it after a change (`job_resized`); doing
  it inside a click handler would edit a child list the engine is walking.
- A text rewritten after the layout ran (the survivor refresh rewrites telemetry and status
  every tick) needs room made again: see `RefitSurvivorDetail`.

## Theme API (`ui_stalker_camp_builder_theme`)

    hs_theme.argb(role[, alpha])      -- color for SetTextColor / SetTextureColor
    hs_theme.rgb(role)                -- r, g, b
    hs_theme.markup(role)             -- "%c[r,g,b]" for SetText
    hs_theme.recolor(text)            -- older inline colors -> theme roles
    hs_theme.faction(key)             -- a faction's ARGB, or nil
    hs_theme.faction_codes()          -- { key = "255,r,g,b" } for inline markup
    hs_theme.set_active(btn, on)      -- selected look, by texture
    hs_theme.banner(static, kind)     -- a status line's backing ("secure"/"neutral"/"threat"); hidden in this theme
    hs_theme.texture(role)            -- "button", "button_on", "rule", "banner"
    hs_theme.short(text)              -- a label without its trailing "(...)", for a narrow button

## Adding to the tab

- **A section:** in the XML, a `<name_lbl>` (letterica18, 238,196,112, height 20) and a
  `<name_rule>` 1 px tall right under it with `<texture>ui_homestead_rule</texture>`; create
  both in the script (`xml:InitTextWnd`, `xml:InitStatic`). The rival panel's three headings
  were in the XML but never created; they are now.
- **A button:** `<texture>ui_button_ordinary</texture>`, `stretch="1"`, no color on `<text>`.
  If it destroys something, add the danger `<text_color>` block.
- **A selectable button:** call `hs_theme.set_active(btn, is_selected)` wherever the
  selection changes.
- **A new color:** add a role to `colors` in the theme, and use `hs_theme.argb("role")`.

## Previewing the tab without a camp

The separate dev mod **Homestead UI preview (dev, sample data)** fills the Settlements tab
with two sample camps (marked "(sample)") and twelve settlers covering every job state:
guard, an expedition, a paused crafter, a medic waiting on supplies, idle, a raid in
progress. Enable it, open the tab, work on the look; disable it to play. It never touches
Homestead's data or the save: the tab reads Homestead through the global
`stalker_camp_builder`, and the preview puts a field of that name into the tab's own
namespace (`ui_stalker_camp_builder_pda`), which shadows the global for the tab's code only.
Getters answer from the sample; jobs, renames and specializations change the sample; every
other Homestead function the tab can call does nothing and answers `false, "preview"`. The
status line reads PREVIEW: SAMPLE DATA. The same trick works for testing any tab against
made-up data.

## Checking the layout

`check_layout.py` (kept with the build tools, not in the mod) measures every text in the
tab against its box with the game's own glyph widths, in English and Russian, and fills in
the worst cases the script can produce: every level name with every specialization, every
status line Homestead can show, every order label. Run it after any change to the layout or
the strings. It reports, per language, what can overflow, and which texts wrap where the
layout makes room for them. The Russian lines it still reports are translation lengths:

- "Отозвать всех поселенцев (Со всех локаций)" on a 166-wide button needs 218
- "Специализация лагеря (нажмите для смены)", the spec button's first label, needs 217
  (the script replaces it with "Spec: ..." once a camp is shown)
- "ОБЯЗАННОСТИ И ТЕЛЕМЕТРИЯ" needs 150 of 148
- "[ Боезапас турелей поселения ]" needs 154 of 152

## Changes from Homestead 1.1.8 beyond dynz's

- Theme module; literal colors in the tab script replaced by roles (dynz's palette
  238,153,26 orange moved back to the PDA's amber).
- Card and banner backings removed; header bars became rules.
- Selected tab/filter/job marked by texture (the label colors never showed: see above).
- Rival panel: its three section headings and rules are created.
- "Refresh" uses `st_pda_btn_refresh_survivors` everywhere (was a literal, and
  `ui_st_refresh` in capitals on the rival panel).
- Rename dialogs: the vanilla stash-naming box, centered (dynz's sat at 240,240 of the
  screen, and its backing was a stretched 60x16 slice, hence see-through).
- dynz's survivor rename handle is a local, like the camp one.
- Overview: region and specialization on two lines of their own; on one line they
  overflowed 350 wide for most specializations (Cordon with a farm already did) and the wrap
  covered the camp name. The defense line goes under them.
- Field station: the buttons go below every line of the status. A paused job explains itself
  in a sentence of up to three lines ("Camp storage has no batteries for electrician
  maintenance"), and the buttons used to cover all but the first.
- The survivor refresh makes room again when the telemetry or status text changes its line
  count (`RefitSurvivorDetail`; positions only).
- Dossier: XP on one line, its gauge on the next (it wrapped and left the percentage alone on a
  line).
- Job buttons: no brackets around the selected one (the texture shows it).
- Crafting and woodcutting order buttons show the order without its "(...)" detail; five
  of them ran past the button ("[ Handgun Calibers (9x18, 9x19, .45 ACP) ]" needs 191 of
  152). The medic orders keep their vodka cost; they fit.
- How to Play: its title and rule were in the XML but never created; they are now.
- Survivor detail: the section rules sit under their headings. The script places the former
  header bars at each section's top, which as 1 px rules put them above the heading.
- Rename dialogs: an opaque backing under the message box (the rule texture tinted to the
  PDA's dark gray); the message box alone is see-through, and the tab's text showed
  through it.
- Survivor detail scrolls to its end: content past its first height (720) was cut and
  could not be reached once a crafter's order button and a two-line status pushed it there.
- Transfer list: the item's text field gets a size. It kept the zero height it was made
  with, so its centered text sat half above the list, and the list's edge cut it.
