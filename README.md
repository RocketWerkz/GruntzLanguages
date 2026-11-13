# Gruntz Languages

Community translations for Gruntz. One folder per language, one `.resx` file per locale inside it.

```
German/de-DE.resx
French/fr-FR.resx
Russian/ru-RU.resx
Spanish/es-ES.resx
```

The file name is the locale code and becomes the language id in the game. The folder name is for people reading this repository. The four starter files were exported from the game on 2026-09-24 and contain the English text for every string, ready to translate.

The English file, `en-US.resx`, is generated from the game's code and content and is not translated by hand. It is not kept in this repository.

## The file format

A file is a standard .NET resx. Each string is one `data` element:

```xml
<data name="Modifier.NoRespawns.Label" xml:space="preserve">
  <value>NO RESPAWNS</value>
  <comment>NO RESPAWNS</comment>
</data>
```

- `name` is the string id. Do not change it. Ids are case sensitive.
- `value` is the text the game shows. This is what you translate.
- `comment` is the English text at the time the file was generated. The game ignores it. It is there so you can see when the English changed under an id you translated earlier.

Three kinds of entry are reserved and are not strings:

- `Language` is the name shown in the language picker. The standard for this repository is the language's own name for itself, in its own script, with no country: `Deutsch`, `Français`, `Русский`, `Español`. When the entry is empty the game falls back to the .NET culture name for the locale code, such as `German (Germany)`, which is not what we want shipped, so always set it.
- `FontMain` and `FontSecondary` name font files. Leave both empty to keep the game's own fonts. See the next section.
- `FontScale` multiplies the game's interface scale while the language is active, for scripts whose glyphs need more room than the Latin faces: `1.3` draws text and the panels around it 30% larger. Empty means 1. The game accepts values from 0.5 to 3 and treats anything else as 1.

Any text editor works. A resx editor works too, as long as it writes plain resx and keeps the `name` values.

## Fonts

The game draws with two typefaces. `FontMain` is the pixel face used across the HUD, the CRT screens and the menus (PressStart2P, an 8 pixel grid font). `FontSecondary` is the text face used for tooltips, settings and prose (JetBrains Mono, with an italic variant). Both cover Latin including the extended ranges, Greek and Cyrillic, so German, French, Spanish, Russian and Ukrainian need no font entries. Neither covers Arabic, Hebrew, Thai, Chinese, Japanese or Korean.

A language whose script the shipped faces do not cover names its own font files in these two entries:

```xml
<data name="FontMain" xml:space="preserve">
  <value>MyPixelFont.ttf</value>
</data>
<data name="FontSecondary" xml:space="preserve">
  <value>MyTextFont.ttf</value>
</data>
```

- The value is a file name. The game looks for it next to the language file first, then in its own `Content/Fonts` folder. A name found in neither place is logged once and the default face stays.
- An empty value keeps the default face for that slot. Set only the slot you need. A Japanese translation, for example, needs both, since neither shipped face has kana or kanji.
- `FontSecondary` replaces both the regular and the italic text face. There is no separate italic entry.
- The swap happens when the language is selected, live, no restart.
- A font file committed here goes in the language's folder beside its resx and must be under a licence that allows redistribution with the game. State the licence in the pull request. The shipped faces are under the SIL Open Font License.
- A pixel face for `FontMain` should sit on the same 8 pixel grid as PressStart2P, or the HUD text will not align to the pixel grid the game snaps sizes to.
- Every language's fonts are also merged behind the game's own faces as glyph fallbacks, so a language's name renders in the picker whatever language is active, provided the font files sit beside the resx.
- If the script reads small at the game's sizes, set `FontScale` (see the reserved entries above) rather than editing the font. Korean ships `1.3`.

## How the game finds language files

- Shipped languages are listed in a mod's `mod.toml` under `languages`. The English file ships this way.
- Every `*.resx` in `Documents\My Games\Gruntz\Languages` also loads. A file there whose name matches a shipped language id replaces the shipped one. Files ending in `.missing.resx` are skipped.
- Languages are listed by display name. Pick one under Settings, Interface, Language, or with the console command `languages set <id>`. The choice is saved in the game settings.
- The console opens with the backslash key by default. The binding can be changed in the game's key bindings settings.

A translation only reaches other players once it is shipped with the game. Until then a file in the user folder is how you test your own work.

## Who maintains which file

- Any file with at least one translated row is maintained by its translators only, whether or not the language ships with the game yet. RocketWerkz never regenerates such a file and never adds English rows to it; after a game update the new strings are simply absent from it, show in English in the game, and `languages missing` lists them. For a shipped language (Danish, German, Korean, Russian, Spanish and Ukrainian at the time of writing) the game's copy is replaced by the file in this repository whenever a pull request is merged.
- A starter file that nobody has translated yet (every value still equal to its comment) is regenerated by RocketWerkz after game updates so it always holds the current English.
- A row whose `value` still equals its `comment` is untranslated English; it renders exactly as a missing row would. Leaving such rows out of a shipped language is fine and keeps `languages missing` accurate.

## Rules a translation must follow

These are enforced by the game. A string that breaks one shows in English and writes a warning to the log. Nothing crashes, but the translation is silently not used.

1. **Placeholders must be preserved.** A placeholder is a name in braces, such as `{NAME}`, `{COUNT}`, `{WEATHER}` or `{ExtractionMinutes}`. The translation must contain exactly the same set of placeholder names, each the same number of times, spelled the same way. They may be moved anywhere in the string. A string may carry at most ten placeholders.
2. **Fractions are two placeholders.** A value such as `37.5` arrives as `{WHOLE}.{FRAC}`. Keep both. You may change the separator, for example `{WHOLE},{FRAC}`.
3. **256 characters at most.** A longer value is refused whole.
4. **Keep the ids.** A `name` the game does not know is ignored and counted as EXTRA in the `languages` table. A missing id shows in English.

## Critical: ids that depend on row order in the game content

Most ids are stable names: `Menu.Quit`, `Def.Obj.Item.Weapon.Gun.Rifle.M41A.DisplayName`, `Emote.Warcry.GetSome`, `Intro.CAPE.Advisory`, `Modifier.NoRespawns.Tooltip`. Those survive any reordering of the content.

Some ids are derived from a row's position in the game's XML and re-key when the rows are reordered or a row is inserted above them:

- `Def.<Id>.Upgrade.<n>.Label` and `Def.<Id>.Upgrade.<n>.Summary`, the upgrade rows on an entity.
- `Emote.<Id>.<n>`, `Intro.<Id>.Line.<n>` and `Mode.<Id>.Modifier.<n>.Label` or `.Tooltip`. These only appear for a row the content authored without a key. Every shipped row currently carries a key, so these ids only show up for modded content.

If the content reorders such rows, a translation stays attached to the old index and appears under the wrong row. After every game update, regenerate a file for your locale (see below), and compare the `comment` column against your previous file for any id whose English changed. `languages missing` does not report a changed English string that already has a translation, so this comparison is the only way to catch a re-keyed row or reworded English.

Also critical for anyone packaging a mod: a language file goes in the mod's `languages` list, never in `assets`. The `assets` list is folded into the join checksum, and a client-only language file there makes the client refuse every server whose copy differs.

## Console commands

All of these write only into `Documents\My Games\Gruntz\Languages`. The game never writes into its own content folder. Before overwriting a file, `generate` and `merge` move the existing one to `<locale>.resx.bak`.

| Command | What it does |
|---|---|
| `languages` | Lists every loaded language: id, display name, string count, how many game strings it is MISSING (absent or rejected), how many EXTRA ids it carries that the game does not know, and the total number of game strings. The active language is marked with `*`. |
| `languages set <id>` | Switches to that language and saves the choice. Refused while the settings window is open, because that window holds a copy of the settings that would revert the change. |
| `languages reload` | Re-reads every language file from disk and re-applies the active one. If the active file is gone, every string resets to English. |
| `languages generate <locale>` | Writes `<locale>.resx` with every string the game has. Where that locale is already loaded and has a valid translation, the value is the translation; otherwise the value is the English text. `generate en-US` always writes the English text. The locale must be a .NET culture name, such as `it-IT` or `pt-BR`. |
| `languages missing [locale]` | Writes `<locale>.missing.resx` containing only the strings that locale lacks or that were rejected, with English values. Without an argument it uses the active language. Writes nothing when nothing is missing. The `.missing.resx` suffix keeps the file out of the loader. |
| `languages merge <locale> <file>` | Reads `<file>` (usually a translated `.missing.resx`) and writes a complete `<locale>.resx`: the file's values first, then the locale's existing valid translations, then English for anything left. `Language` and `Font` come from the file when present, otherwise from the existing language. |
| `languages mark` | Toggles marking. While on, every string the active language lacks shows as `** English text **` in the game, so gaps are visible in place. Toggling reloads. |

## Editing in the game

The fastest way to translate is in place. Set your language, open the console and run `languages edit on`. While the mode is on, hold CTRL and SHIFT and left click any string on screen. A popup opens with the string's id, the English text and a field holding the current translation. Change it and press APPLY. The screen updates at once, and the game writes your whole language file to `Documents\My Games\Gruntz\Languages\<locale>.resx`, backing the previous copy up as `.bak`. RESET drops the string back to English. When several strings share the same words the popup lists them by id first; when nothing is under the cursor it offers a search field over ids and English.

- The mode refuses to start while English is selected, since English is generated from the game's code.
- A translation must keep the placeholders the English has; APPLY refuses one that does not.
- The field takes any character in the Basic Multilingual Plane, typed or pasted, up to 256 characters.
- Turn the mode off with `languages edit off` or `languages edit`. The file it wrote is the file to commit here.

## Making a new language

1. Start the game and open the console.
2. Run `languages generate <locale>`, for example `languages generate it-IT`. The file lands in `Documents\My Games\Gruntz\Languages\it-IT.resx`.
3. Translate the `value` of each string. Leave `name` alone. Set `Language` to the language's own name for itself, with no country, for example `Italiano`.
4. Back in the game, run `languages reload`, then pick the language in Settings or run `languages set it-IT`. Run `languages mark` to see what is still untranslated while you play.
5. In this repository, create a folder named after the language, for example `Italian/`, put `it-IT.resx` in it, and open a pull request.

The file has about 18,700 strings. A partial file is fine: anything untranslated shows in English.

## Updating a translation after a game update

1. Put your current file in the user folder, start the game and run `languages set <locale>`.
2. Run `languages missing`. Translate the `value` entries in `<locale>.missing.resx`.
3. Run `languages merge <locale> <full path to the missing file>`. The result is a complete file in the user folder.
4. Run `languages generate <locale>` once more and compare its `comment` entries against your previous file. Any id whose comment changed had its English reworded or re-keyed; re-check those translations.
5. Replace the file in this repository and open a pull request.

## Contributing

- One pull request per language.
- Keep the file name as the locale code and keep the folder name as the language name in English. The `Language` entry inside the file is the language's own name for itself, no country.
- Do not reformat the file. Keep the `name` values and the placeholders exactly as generated.
- Describe in the pull request which game build you generated or merged against.
