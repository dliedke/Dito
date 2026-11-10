# Dito

🇧🇷 [Leia em português](README.pt-BR.md)

A word guessing game in **English and Portuguese**, in the style of Wordle — 100% front-end, no build step, no server dependencies.

🔗 **Play** https://dliedke.github.io/Dito/

## How to play

Find the secret word before you run out of tries. Every guess must have the chosen number of letters.

- 🟩 **Green** — the letter is in the word and in the right spot.
- 🟦 **Blue** — the letter is in the word, but in another spot.
- ⬛ **Gray** — the letter is not in the word.

In Portuguese, accents are filled in automatically: typing `avo` counts as `avô`, and `ç` is typed as `C`.

## Languages

- The language is **detected from the browser**: Portuguese browsers get Portuguese, every other language gets English.
- It can be changed at any time in **Settings → Language** (the choice is remembered in the browser). `?lang=en` / `?lang=pt` in the URL also works.
- Each language has its **own word list, theme groups and word of the day**, and the daily history is kept separately per language. XP, achievements, stats and records are shared.
- Challenge links carry the language they were created in, so a friend always gets a word from the right dictionary.

## Features

- **Configurable word length** — 3 to 8 letters.
- **Configurable number of tries** — 3 to 8.
- **Built-in dictionaries** in English and Portuguese, grouped by theme (animals, food, emotions, places...), with no external API.
- **Modes** — 📅 Daily (one word a day, the same for everyone, with an archive of past days), ♾️ Unlimited, ⚡ Blitz (against the clock) and 🎲 Variants (Duet, Quartet, Cheater, Chain, Sudden death, Pass & play, Liar, Count, Memory, Ladder and Anagram).
- **Themes** — limit the draw to one category; the 💡 group hint reveals the theme of the secret word.
- **Hard mode** — every revealed clue must be reused in the next guess.
- **Hints at the end** — on the last two tries the game reveals a letter, then a letter's position (optional).
- **Challenge by link** — 🔗 copies a link with the current word for a friend to try.
- **XP, levels, daily missions and achievements**, with optional sync between devices through your own Google Drive.
- **End-of-game analysis** — how many words were still possible after each guess, plus a dictionary link for the answer.
- **Share** your result as text or as an image (colors only, no spoilers).
- **Light / dark / 3D looks**, synthesized sound effects (Web Audio) and vibration on supported phones.
- **Accessible virtual keyboard**, working alongside the physical keyboard.
- **Responsive and mobile-friendly** — respects safe areas (notch/home indicator) on iOS, larger touch targets, landscape tweaks.

## Project structure

```
Dito.slnx            → Visual Studio solution
Dito.csproj          → ASP.NET Core project (only hosts the static files)
Program.cs           → minimal host, serves wwwroot/ with UseStaticFiles
Properties/
  launchSettings.json → run profiles (F5 opens the browser)
wwwroot/
  index.html          → page structure (HTML), with English text in data-en attributes
  style.css           → look, themes and responsiveness
  script.js           → game logic, dictionaries (pt + en) and state
```

The game itself (`wwwroot/`) is 100% front-end — plain HTML/CSS/JS, no build and no
server calls. The .NET project only exists as a convenient way to open the repository
as a solution in Visual Studio and run it locally with F5; it has no game logic, API
or server-side state.

No external dependencies other than Google Fonts (`Bricolage Grotesque` and `Space Grotesk`).

## Running locally

**With Visual Studio:** open `Dito.slnx` and press F5 (or Ctrl+F5). The browser opens
at `http://localhost:5080/`.

**With the .NET CLI** (SDK 8 or newer):

```bash
dotnet run
```

**Without .NET**, since it is a fully static site, you can open `wwwroot/index.html`
directly in the browser, or serve the folder with any simple server:

```bash
npx serve wwwroot
```

```bash
python -m http.server --directory wwwroot
```

Then open `http://localhost:<port>/`. Add `?lang=en` or `?lang=pt` to force a language.

## Publishing to GitHub Pages

Deployment is automatic through GitHub Actions (`.github/workflows/pages.yml`): on every
push to `main`, the workflow publishes the contents of `wwwroot/` to GitHub Pages (no
build, just a copy of the static files).

1. Push the project to [github.com/dliedke/Dito](https://github.com/dliedke/Dito) (branch `main`).
2. On GitHub, go to **Settings → Pages**.
3. Under **Source**, select **GitHub Actions** (instead of "Deploy from a branch").
4. Push to `main` (or run the workflow manually under **Actions**) and wait for the deploy (about 1 minute).
5. The game will be available at `https://dliedke.github.io/Dito/`.

## Data persistence

Settings (look, word length, tries, modes, language) and progress are saved in the
browser's `localStorage`, so they persist between visits on the same device/browser.
If `localStorage` is unavailable (e.g. restricted private mode), the game detects it
and keeps working, just without saving. Signing in with Google (under 🎯) optionally
syncs progress through a hidden file in your own Google Drive.

## License

Define the project's license here (for example, MIT) if you want to make it open.
