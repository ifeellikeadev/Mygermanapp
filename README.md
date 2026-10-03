# Deutsch: personal German learning app

A phone-installable web app (PWA) to learn German with spaced repetition. It has no server, no account and no build step. Everything runs in the browser, your progress is stored on your phone, and the whole thing is hosted for free on GitHub Pages.

This README explains every file, every data format and every part of the code, so you can change the app yourself.

---

## 1. Contents

1. [How it fits together](#2-how-it-fits-together)
2. [Folder structure](#3-folder-structure)
3. [Deploying and updating](#4-deploying-and-updating)
4. [The data files](#5-the-data-files)
5. [Adding content](#6-adding-content)
6. [How the app code works](#7-how-the-app-code-works)
7. [Where your progress is stored](#8-where-your-progress-is-stored)
8. [The learning algorithm](#9-the-learning-algorithm)
9. [Recipes: common changes](#10-recipes-common-changes)
10. [The PWA files (manifest, service worker, icons)](#11-the-pwa-files)
11. [Backup and restore](#12-backup-and-restore)
12. [Troubleshooting](#13-troubleshooting)
13. [Conversation mode (not built yet): design notes](#14-conversation-mode-design-notes)
14. [Known limitations and ideas](#15-known-limitations-and-ideas)
15. [Where the content comes from](#16-where-the-content-comes-from)

---

## 2. How it fits together

```
 phone browser
   └── opens  https://<user>.github.io/<repo>/        (GitHub Pages serves static files)
         ├── index.html           the whole app (HTML + CSS + JavaScript in one file)
         ├── data/words.json      vocabulary
         ├── data/grammar.json    grammar rules + quiz questions
         ├── data/verbs.json      irregular verbs
         ├── sw.js                service worker: makes the app work offline
         └── manifest.webmanifest + icons   make "Add to Home screen" work

 on the phone only (localStorage, key "dl1"):  your progress, settings, streak
```

Key idea: the **content** (words, grammar, verbs) lives in three JSON files, and the **behavior** lives in `index.html`. You can change content without touching code, and the other way round.

---

## 3. Folder structure

```
Mygermanapp/                    ← your GitHub repository (main branch, root folder)
├── index.html                  the app
├── manifest.webmanifest        app name, colors, icons (for installing)
├── sw.js                       offline cache
├── icon-192.png                app icon (small)
├── icon-512.png                app icon (large)
├── README.md                   this file
├── data/
│   ├── words.json              2,674 words
│   ├── grammar.json            47 grammar rules, 94 quiz questions
│   └── verbs.json              191 irregular verbs
└── tools/
    └── merge_words.py          optional helper to rebuild words.json from batch files (see section 6)
```

Important: `data/` must sit **next to** `index.html`, and its name and the three file names must be exactly as above (lowercase). The app loads them with the relative addresses `data/words.json`, `data/grammar.json` and `data/verbs.json`.

---

## 4. Deploying and updating

### First deployment
1. Create a **public** repository on github.com.
2. Upload the files and folders from section 3 to the **top level** of the repository (**Add file → Upload files**; you can drag whole folders in Chrome on a computer).
3. **Settings → Pages → Build and deployment**: Source = *Deploy from a branch*, Branch = `main`, Folder = `/(root)`, then **Save**.
4. After 1 to 2 minutes the site is live at `https://<your-username>.github.io/<repository-name>/` (for you: `https://ifeellikeadev.github.io/Mygermanapp/`).
   Note it's `github.io`, not `github.com`, and the name is case-sensitive in the path.
5. On the phone, open that link in Chrome → **⋮ menu → Add to Home screen**.

### Updating later
- **Small text edits** (a word, a typo): on github.com open the file, click the **✏️ pencil**, edit, click **Commit changes**.
- **A proper editor in the browser (recommended):** on the repository page press the **`.`** key (a full stop). GitHub opens *github.dev*, a free VS Code editor in the browser. Edit several files, then use the **Source Control** icon on the left, type a message and click **Commit & Push**.
- **Big updates:** upload files again with **Add file → Upload files**. Files with the same name are replaced.
- **After an update**, GitHub Pages needs about a minute to publish. On the phone, close the app completely and reopen it. If you still see the old version, see [Troubleshooting](#13-troubleshooting).

### Rules to avoid trouble
- Always edit the **JSON** files with a proper editor and keep the quotes, commas and brackets valid. One missing comma breaks the whole file and the app will show "Could not load data". Check a file at https://jsonlint.com if unsure.
- Files must be saved as **UTF-8** (the umlauts ä ö ü ß must stay readable).

---

## 5. The data files

### 5.1 `data/words.json`

```json
{
  "f": ["de","gender","en","pos","theme","cefr","rank","example","plural"],
  "w": [
    ["Mutter","f","mother","noun","family","A1",470,"Meine Mutter kocht gern.","Mütter"],
    ["machen","","to do, to make","verb","everyday","A1",40,"Was machst du heute?",""]
  ]
}
```

`"f"` is only a description of the columns. The app reads the rows in `"w"` **by position**:

| # | Field | Meaning | Allowed / notes |
|---|---|---|---|
| 0 | `de` | German word. For reflexive verbs write `sich waschen`. Do **not** put the article here. | text |
| 1 | `gender` | Colors the word: `m` blue, `f` red, `n` green | `m`, `f`, `n` or `""` (empty for non-nouns and plural-only nouns, shown in the default text color) |
| 2 | `en` | English translation | text |
| 3 | `pos` | Part of speech | `noun`, `verb`, `adjective`, `adverb`, `preposition`, `phrase`, `numeral`, `pronoun`, `proper noun`, `conjunction`, `question word`, `interjection`, `determiner`, `article` |
| 4 | `theme` | Category used by the Browse filter | any text; currently 33 themes (see below). The filter list is built automatically from whatever you use. |
| 5 | `cefr` | Level | `A1`, `A2`, `B1` or `B2`. **The placement test only looks at A1, A2 and B1.** |
| 6 | `rank` | Frequency rank (1 = most common). Decides **the order in which new words are introduced** in Learn. | number |
| 7 | `example` | Example sentence shown on the back of the card | text |
| 8 | `plural` | Plural form (nouns only), shown as "Plural: die …" | text or `""` |

Current themes: abstract, animals, body, business, city, clothing, colors, communication, connectors, education, emotions, everyday, family, food, grammar, health, home, kitchen, leisure, media, nature, numbers, objects, phrases, prepositions, pronouns, questions, school, shopping, social, tech, time, travel.

Current content: 2,674 words (1,476 nouns, 458 verbs, 331 adjectives, 207 adverbs and others), levels A1/A2/B1 plus 75 words marked B2.

### 5.2 `data/grammar.json`

```json
{ "rules": [
  {
    "id": "g02",
    "title": "Definite article (der, die, das) in all cases",
    "level": "A1",
    "cat": "cases",
    "explain": "The definite article changes with gender, number and case ...",
    "table": { "head": ["Case","masc.","fem.","neut.","plural"],
               "rows": [["Nominativ","der","die","das","die"], ["Akkusativ","den","die","das","die"]] },
    "examples": [["Ich gebe dem Kind den Ball.", "I give the child the ball."]],
    "quiz": [ { "q": "Ich helfe ___ Frau.", "opts": ["die","der","den","dem"], "a": 1 } ]
  }
]}
```

| Field | Meaning |
|---|---|
| `id` | Unique, never change it once you use the app (your progress is saved under this id). Pattern `g01`, `g02`, … |
| `title`, `level`, `cat` | Display texts. `cat` is only informational right now. |
| `explain` | Explanation (plain text; simple HTML like `<b>` also works) |
| `table` | Optional. `head` = column headings, `rows` = rows. Use `null` for no table. |
| `examples` | List of `[German, English]` pairs |
| `quiz` | List of questions used by the daily quiz. `opts` = answer options, `a` = **index** of the correct option, **starting at 0** (`"a": 1` means the second option). |

The 47 rules cover cases, article declension (definite, indefinite, possessive, personal pronouns, relative pronouns), prepositions, adjective endings, tenses, modal verbs, separable verbs, word order, subordinate clauses and more.

### 5.3 `data/verbs.json`

```json
{ "fields": "present = ich, du, er/sie/es, wir, ihr, sie/Sie; preterite = same order; perfect = aux + participle",
  "verbs": [
    { "inf": "fahren", "en": "to drive, to go (by vehicle)", "sep": "",
      "present":   ["fahre","fährst","fährt","fahren","fahrt","fahren"],
      "preterite": ["fuhr","fuhrst","fuhr","fuhren","fuhrt","fuhren"],
      "aux": "sein", "participle": "gefahren", "level": "A1" }
  ] }
```

| Field | Meaning |
|---|---|
| `inf` | Infinitive |
| `en` | Translation |
| `sep` | Separable prefix (`"auf"` for aufstehen) or `""`. Informational. |
| `present`, `preterite` | Exactly **6** forms each, in the order ich, du, er/sie/es, wir, ihr, sie/Sie. For separable verbs the prefix is already attached at the end (`"stehe auf"`). |
| `aux` | `haben`, `sein` or `haben/sein` |
| `participle` | Partizip II (`gefahren`). The app shows the Perfekt as "aux: participle". |
| `level` | A1/A2/B1 (informational) |

### 5.4 IMPORTANT: IDs are positions, so only add at the END

The app saves your progress under IDs built from **positions**:

- word = `w` + position in the `"w"` list of `words.json` (`w0` = first row)
- verb = `v` + position in the `"verbs"` list of `verbs.json`
- grammar rule = its `id` text (`g07`)

Therefore:

- ✅ **Add new words, verbs and rules at the END of their list.** Existing progress stays correct.
- ❌ **Do not reorder, sort, delete or insert in the middle** of `words.json` or `verbs.json`. Your saved progress would then point to the wrong words.
- If you must fix a word, **edit it in place** (fix the translation or example, keep the row where it is).
- If you want to remove a word, leave the row and change its content instead, or accept that progress will shift (see "reset" in section 10).

(How new words are introduced is independent of file position: Learn orders them by `rank`. The position is only the ID.)

---

## 6. Adding content

### 6.1 Add a few words by hand
1. Open `data/words.json` (github.dev is easiest).
2. Go to the **end of the `"w"` list**, just before the last `]` and `}`.
3. Add a comma after the last row, then your new rows in the 9-column format:
   ```json
   ,["Fernbedienung","f","remote control","noun","tech","B1",150000,"Wo ist die Fernbedienung?","Fernbedienungen"]
   ```
   Use a high `rank` for rare words (they come later in Learn). A high number like 150000 means "very rare".
4. Commit. Reopen the app.

### 6.2 Add many words with the helper script (optional)
`tools/merge_words.py` rebuilds `words.json` from batch files and **fills in real frequency ranks and plurals automatically**. You need Python on a computer:

```
pip install wordfreq german-nouns
python tools/merge_words.py  ./batches  ./data/words.json
```

`./batches` is a folder with files named `words-01.json`, `words-02.json`, … in this format (8 columns, no plural, the script adds rank and plural):

```json
{"fields":["de","gender","en","pos","theme","cefr","rank","example"],
 "words":[["Haus","n","house","noun","home","A1",180,"Das Haus ist groß."]]}
```

Rules: read in alphabetical order (that order becomes the IDs), so **never edit or remove old batch files, only add new, later-named ones** (`words-17.json`, …). The original 16 batch files I produced during the build contain exactly the current 2,674 words. If you no longer have them, edit `words.json` directly instead (6.1).

### 6.3 Add a grammar rule
Append a new object at the end of `"rules"` in `grammar.json` with a **new unique id** (`g48`, `g49`, …) and the fields from section 5.2. Remember: `"a"` is the index of the right answer starting at 0. Rules appear in Learn in file order (one new rule per day if "Grammar rules in Learn" is on).

### 6.4 Add an irregular verb
Append an object at the end of `"verbs"` in `verbs.json` with exactly 6 `present` and 6 `preterite` forms. Check the stem-changing forms (du/er) by hand.

---

## 7. How the app code works

Everything is in `index.html`, in three blocks: **CSS** (in `<style>`), **HTML** (just an empty `<main id="v">` and `<nav id="n">`), and **JavaScript** (in `<script>`). The page is drawn by JavaScript: each tab function builds an HTML string and puts it into `<main>` with `H(...)`.

### 7.1 Global variables

| Name | What it is |
|---|---|
| `W`, `G`, `V` | words list, grammar rules list, verbs list (loaded from the JSON files) |
| `O` | word indexes sorted by frequency rank (the order in which new words are introduced) |
| `S` | all saved state (progress, settings, activity), see section 8 |
| `tab` | name of the open tab |
| `Q` | current quiz (a list of questions with a few extra properties) |
| `L` | current Learn session (`L.q` queue, `L.i` position, `L.f` flipped or not, `L.day` day it was built) |
| `BF` | Browse filter (`t` theme, `q` search text, `n` how many rows are shown) |
| `IV` | the SRS intervals in days: `[1,3,7,21]` |
| `COL` | gender colors, as CSS variables: `{m:'var(--m1)', f:'var(--f1)', n:'var(--n1)'}` (the actual colors are defined in the CSS, see section 10) |
| `AR` | articles: `{m:'der', f:'die', n:'das'}` |
| `LV` | the level order `['A1','A2','B1','B2']` used for the adaptive level |
| `GC` | gender → CSS class (`gm`, `gf`, `gn`) that tints the flashcard and Browse rows |

### 7.2 Function reference

| Function | What it does |
|---|---|
| `today()` | Today's day number (days since 1970, in your local time). Used for due dates and streaks. |
| `streak()` | Number of days in a row with at least one review (shown in the header and in Stats). |
| `sv()` | Saves `S` to localStorage. Called after every change. |
| `sh(a)`, `pick(a,n)` | Shuffle a list; pick n random items. |
| `wd(w)` | Returns a word as colored HTML with its article (`der Mann` in blue). |
| `tabs()` | Draws the bottom navigation (with icons) and the header (title and streak). |
| `go(name)` | Switches tab and calls the tab's function. |
| `mark(id,g)` | **Core SRS update** for a card: g = 0 Again, 1 Hard, 2 Good, 3 Easy, 4 Already known. |
| `ivt(id,g)` | Text of the next interval ("<1d", "3d" ...) shown under each grade button. |
| `act()` | Counts one review for today (feeds the streak and the 7-day chart). |
| `vtab(x)`, `gbody(r)` | Build the verb table and the grammar rule body (explanation, table, examples). |
| `learn()` | Builds today's queue (first call of the day) and draws the current card. |
| `gr(g)` | Called by the grade buttons: marks the card, re-queues "Again" cards, moves on. |
| `lvl()` | Returns your **working level** (see section 7.5). |
| `rec(type,level,ok)` | Records a quiz answer in the accuracy statistics (`S.qt` per area, `S.ql` per level). |
| `wpick(items,n)` | Weighted random pick **without replacement** (this is why a quiz never repeats a question). |
| `mkQ()` | Creates the daily quiz (10 questions) from the cards you have started. |
| `quiz()`, `qa(answer, button)` | Draw a question; check an answer and update the statistics and the card. |
| `grammar()` | Grammar tab: list of rules (tap to expand). |
| `browse()` | Browse tab: theme filter, search, rows, verb tables. |
| `stats()` | Stats tab: streak, chart, progress, "Your level", settings, export/import. |
| `place()`, `pq()`, `pa(a)` | Placement test: build it, draw a question, answer. |
| *boot block (last lines)* | Loads the three JSON files, builds `O`, then decides what to open first (see 7.3). |

---

### 7.3 What happens when you open the app

1. The three JSON files are fetched. If one fails you see "Could not load data: …".
2. If you never did the placement test (`S.p` missing), the test starts.
3. Else, if the quiz of today is not done yet (`S.q != today()`) and you have started more than 3 cards, the **daily quiz opens automatically**.
4. Otherwise the Learn tab opens.

### 7.4 How the Learn queue is built (`learn()`)

On the first call each day (`L.day != today()`):

1. **Due reviews**: every saved card whose due day is today or earlier, shuffled, **at most 40**. Cards you answered wrongly in a quiz are due today, so they appear here.
2. **New words**: N words you have never seen (N = "New words per day" in Stats, default 4), chosen **adaptively**: words of your *working level* first (see 7.5), then lower levels, then higher ones. Inside each group the order is by frequency `rank` (most common first).
3. **One new grammar rule** (if enabled in settings): the first rule you have not seen yet.

The queue is `[reviews..., new words..., new grammar rule]`. A card marked *Again* is put back 4 positions later. Each grade button shows the next review interval (`ivt()`).

### 7.5 Daily quiz and the adaptive level

**What the quiz contains (`mkQ()`)**: up to **10 questions** from the cards you have started:

| Part | How many | Question |
|---|---|---|
| Verbs | up to 2 (if you started verbs) | Präteritum (er), Perfekt or du-form, multiple choice |
| Grammar | up to 3 (if you started rules) | a quiz question from `grammar.json` |
| Words (rest, so 5 to 10) | the remaining slots | about **40% typed** (English → German **with the article**), the rest German → English multiple choice (3 wrong options of the same part of speech) |

If you have started fewer cards the quiz is shorter. Questions are shuffled.

**Typed answers**: for nouns the article is **required** (`der Hund`, not `Hund`). Case and extra spaces are ignored, but umlauts and ß must be exact. Words without a gender (verbs, adjectives, adverbs) need just the word.

**No repeated questions**:
- Items are picked **without replacement** inside a quiz (`wpick`), so nothing appears twice in the same quiz.
- The last **90 questions** are remembered (`S.rq`) and get only 15% of their normal chance in the following quizzes.

**What gets asked more often**: each item has a weight, `(1 + 2·x + 3 if due today + 2 if box 0) × 0.3 if marked "already known" × 0.15 if asked recently`, where `x` is how many times you got it wrong recently. Grammar questions use `(1 + 2·x of the rule)`.

**After each answer (`qa()`)**:
- the answer is recorded in the statistics: by **area** (words, verbs, grammar) in `S.qt` and by **level** (A1 to B2) in `S.ql`. When a counter passes 80 (areas) or 60 (levels) both numbers are halved, so recent results count more;
- a **wrong** answer sends the card to box 0, due today, and raises its `x`; a **correct** answer lowers `x` by 1. This works for words, verbs and grammar rules;
- the schedule of a correctly answered card is not changed (only Learn grades move cards up).

**Your working level (`lvl()`)**: the first level in A1, A2, B1, B2 where you have fewer than **6 recorded answers** or an accuracy **below 80%**. If all four are above 80% it returns B2. It controls which new words Learn introduces (7.4) and is shown in Stats under "Your level", together with accuracy per level and per area. If one area is below 75% Stats adds "Focus on …".

### 7.6 Placement test

`place()` picks **8 random words per level** (A1, A2, B1) from the most frequent 60% of that level and asks German → English multiple choice (24 questions in total). A level with **6 or more correct** (`Q.res[l]>=6` in `pq()`) is marked as known: every unseen word of that level gets `mark(id,4)` (box 3, due in 60 days). The 8 answers per level also **seed the level statistics** (`S.ql`), so your working level is set immediately. It runs at first launch and any time from Stats.

---

## 8. Where your progress is stored

Everything is in the phone's browser storage (`localStorage`, key `dl1`), as one JSON object `S`:

```json
{
  "c":  { "w12": {"b":2,"d":19994,"x":1}, "g03": {"b":1,"d":19990}, "v7": {"b":3,"d":20050,"k":1} },
  "s":  { "n": 4, "g": 1 },
  "d":  { "19990": 12, "19991": 30 },
  "q":  19991,
  "p":  1,
  "ql": { "A1": {"c":14,"w":2}, "A2": {"c":9,"w":4} },
  "qt": { "w": {"c":30,"w":6}, "v": {"c":5,"w":3}, "g": {"c":8,"w":4} },
  "rq": ["w12","w40","g03.0","v7pret"]
}
```

| Key | Meaning |
|---|---|
| `c` | Cards you have started. Key = card id (`w…` word, `v…` verb, `g…` grammar). `b` = box 0 to 3, `d` = day number when it is next due, `k:1` = marked "already known", `x` = recent wrong quiz answers (makes the card come back more often). |
| `s` | Settings: `n` new words per day, `g` grammar in Learn (1 or 0) |
| `d` | Reviews per day, by day number (feeds the streak and the chart) |
| `q` | Day number when the daily quiz was last completed |
| `p` | 1 if the placement test was done |
| `ql` | Quiz accuracy per level: `c` correct, `w` wrong (decides your working level) |
| `qt` | Quiz accuracy per area: `w` words, `v` verbs, `g` grammar |
| `rq` | The last 90 question keys asked (to avoid repeats) |

Day numbers are "days since 1970-01-01": 20,000 is in October 2024, and 20,363 is in October 2025 (use `today()` in the browser console to see today's).

Notes:
- The data lives in **this browser on this phone** only. Clearing Chrome's site data, uninstalling Chrome or switching phones deletes it. Use Export (section 12).
- If you open the site on a computer, it has its own, separate progress.
- Older saves without `ql`, `qt` or `rq` still work: the app creates them when needed.

---

## 9. The learning algorithm

An SM-2-style system simplified to **4 boxes**: 0 (new or failed), 1, 2, 3 (mastered). The intervals are `IV=[1,3,7,21]` days. Implemented in `mark(id,g)`:

| Grade | Effect |
|---|---|
| **Again** (0) | box 0, due today, and the card comes back again in this session |
| **Hard** (1) | same box, due again in `max(1, round(IV[box]/2))` days |
| **Good** (2) | box +1 (max 3), due in `IV[new box]` days |
| **Easy** (3) | box +2 (max 3), due in `IV[new box]` days |
| **Already known** (4) | box 3, due in **60 days**, flagged `k:1` |

Quiz answers also feed the schedule: a wrong answer in the quiz sends the card back to box 0, due today (section 7.5). A card is "due" when its due day number is less than or equal to today. Cards in box 3 keep coming back every 21 days (or 60 for "known"), so nothing is ever completely forgotten.

---

## 10. Recipes: common changes

All changes are in `index.html`. Search (Ctrl+F) for the quoted text to find the place.

| I want to … | Do this |
|---|---|
| **Change the review intervals** | Change `IV=[1,3,7,21]`, for example `[1,2,5,12]` for a faster cycle. Keep 4 numbers (or add more, and change the `Math.min(c.b+1,3)` limits to the last index). |
| **Change the default number of new words** | `S.s=S.s\|\|{n:4,g:1}` → change `n:4`. (Only affects new users or after a reset. Otherwise use the Settings in Stats.) |
| **Make the daily quiz longer or shorter** | In `mkQ()`: the number of slots is `10-nV-nG` (words), `nV=Math.min(2,…)` (verbs) and `nG=Math.min(3,…)` (grammar). Change the `10` for the total, the `2` and `3` for verbs and grammar. |
| **Change the share of typed questions** | In `mkQ()`: `Math.round(nW*.4)` means 40% of the word questions are typed. For example `.7` gives 70% typed. |
| **Stop typed answers needing the article** | In `qa()`: replace `ok=n(a)==n(x.a)` by `ok=n(a).replace(/^(der\|die\|das) /,'')==n(x.a).replace(/^(der\|die\|das) /,'')`. |
| **Change when you move up a level** | In `lvl()`: `q.c/n<.8` is the 80% accuracy needed and `n<6` the minimum number of answers per level. |
| **Change how strongly repeated questions are avoided** | In `mkQ()`: `(rq.has(k)?.15:1)` is the weight of recently asked items (smaller = less repeats), and `.slice(-90)` is how many are remembered. |
| **Stop the quiz opening automatically** | In the last lines: delete the part `if(S.q!=today()&&Object.keys(S.c).length>3){...return quiz()}`. |
| **Change gender colors** | In the CSS at the top: `--m1` (der, masculine), `--f1` (die, feminine), `--n1` (das, neuter) are the strong colors and `--m2`, `--f2`, `--n2` the soft background tints of the flashcard. There is one set for light mode (`:root`) and one for dark mode (inside `@media(prefers-color-scheme:dark)`). |
| **Change the app colors / dark mode** | The CSS variables at the top: `--bg` page background, `--c` card background, `--t` text, `--m` muted text, `--l` lines, `--p` button color, `--hd` the blue header band. Light values are in `:root`, dark values in the `@media(prefers-color-scheme:dark)` block. The header diamond pattern is the `background-image` of `header`; remove it for a flat header. |
| **Cap daily reviews** | In `learn()`: `due.slice(0,40)` → change 40. |
| **Change placement test size or pass mark** | In `place()`: `pick(p,8)` is the number per level (also change the `/8` texts in `pq()`); in `pq()`: `Q.res[l]>=6` is the pass mark. |
| **Remove grammar from Learn by default** | `S.s=S.s\|\|{n:4,g:1}` → `g:0`. |
| **Show 300 rows in Browse instead of 150** | `BF={t:'all',q:'',n:150}` and the `BF.n+=150` in the "More" button. |
| **Change the app name** | `<title>` in `index.html` and `name`/`short_name` in `manifest.webmanifest`. Reinstall the app on the phone to see it. |
| **Reset all progress** | In Chrome on the phone: ⋮ → Settings → Site settings → find the site → **Clear & reset**. Or open the browser console and run `localStorage.removeItem('dl1')`. |
| **Add a new tab** | 1) add `['mytab','My tab']` to the list in `tabs()`; 2) add `mytab` to the `({learn,quiz,grammar,browse,stats})` object in `go()`; 3) write `function mytab(){ H('<div class="card">…</div>') }`. |
| **Test on a computer** | Do not open `index.html` by double-clicking (the browser blocks loading the JSON files that way). Run `python -m http.server` in the folder and open `http://localhost:8000`. |

---

## 11. The PWA files

- **`manifest.webmanifest`**: name, colors, start address and icons. Chrome uses it for "Add to Home screen". `display:"standalone"` removes the browser bar.
- **`icon-192.png`, `icon-512.png`**: the app icon. Replace them with any PNG of the same sizes (keep the file names, or change them in the manifest).
- **`sw.js`** (service worker): it caches the app and the data files so the app **works offline**. It is *network first*: when you are online it always fetches the newest files and refreshes the cache; when offline it serves the cache. The cache name is `dl-v1`.
  - If an update does not show up, change `const C='dl-v1'` to `'dl-v2'` (and so on) in `sw.js`, commit, and reopen the app twice.
  - The first visit must happen online so that the files get cached.

---

## 12. Backup and restore

In the **Stats** tab:
- **Export** copies all your progress as text to the clipboard. Paste it into a note or an email to yourself.
- **Import** asks you to paste that text and restores it.

Do an export from time to time, and **always before** you clear Chrome data, change phones or edit the code in a way that touches `S`.

---

## 13. Troubleshooting

| Problem | Cause and fix |
|---|---|
| **"Could not load data: SyntaxError: Unexpected token '<'"** | The app asked for a data file and GitHub answered with its HTML "404" page. One of `data/words.json`, `data/grammar.json`, `data/verbs.json` is missing or in the wrong place. Open each address in the browser, e.g. `https://<user>.github.io/<repo>/data/words.json`. A 404 shows you which one is missing. Check that the `data` folder is **next to** `index.html` and the names are exactly lowercase with no `(1)`. |
| "Could not load data: SyntaxError: Unexpected token … in JSON" (a letter or symbol other than `<`) | A JSON file has a typo (missing comma, quote or bracket). Paste the file into https://jsonlint.com and fix the line it points to. |
| Page is a **404** | GitHub Pages is not enabled yet, wrong address (github.com instead of github.io), or the repo is private. Settings → Pages. Wait 2 minutes. |
| **Blank white page** | Open the Chrome menu → *Desktop site* on a computer and press F12 to see the console error. Usually a typo from editing `index.html`. |
| I updated a file but the app is **unchanged** | Close the app fully and reopen. Still old? Bump the cache name in `sw.js` (section 11). Last resort: Chrome → Site settings → *Clear & reset*, but **export your progress first**. |
| **My progress disappeared** | Browser data was cleared, or you opened a different address (for example `…/deutsch/` vs `…/`), or a different browser. Progress is per address and per browser. Import your last export. |
| **Words are wrong after I edited `words.json`** | You reordered, deleted or inserted rows in the middle (section 5.4). Restore the old file. Only append at the end. |
| **Colors missing on a word** | Its `gender` field is empty. Nouns need `m`, `f` or `n`. |
| **A word never appears in Learn** | Words are introduced in `rank` order. A rank like 150000 means "far down the list". Lower the rank to bring it forward. |
| "Add to Home screen" gives only a shortcut | Chrome decides this. Open the site, wait a few seconds, and use the ⋮ menu → *Install app* if it is offered. Both work the same for you. |
| **Typed answer marked wrong** | For nouns the **article is required** (`der Hund`). Case and extra spaces are ignored, but umlauts and ß must be exact. |
| **Learn keeps showing words of a low level** | New words come from your *working level* (section 7.5). Retake the placement test in Stats to reset your starting level, or answer more quizzes. |

---

## 14. Conversation mode: design notes

Not built yet. This is the plan so you (or I, later) can add it. It is an extra tab with a chat where you write or speak, and an AI answers in German at your level and corrects you.

### Parts
1. **Speaking in**: the browser's `webkitSpeechRecognition` with `lang='de-DE'` (works in Chrome on Android; needs microphone permission and an internet connection).
2. **Speaking out**: `speechSynthesis.speak(new SpeechSynthesisUtterance(text))` with `utterance.lang='de-DE'`.
3. **The "brain"**: a call to the Anthropic API. It needs an **API key** (pay per use; short chats cost cents).
4. **Level**: your level can be estimated from the app's own progress. For example, use the highest level where you have many mastered cards (`S.c` entries with `b==3`, joined with `W[i][5]`), and put it into the system prompt.

### API call sketch (inside the app)
```js
const key = localStorage.getItem('dl-key');          // typed once into a settings field
const r = await fetch('https://api.anthropic.com/v1/messages', {
  method: 'POST',
  headers: {
    'content-type': 'application/json',
    'x-api-key': key,
    'anthropic-version': '2023-06-01',
    'anthropic-dangerous-direct-browser-access': 'true'   // needed to call from a browser
  },
  body: JSON.stringify({
    model: 'claude-haiku-4-5-20251001',                 // fast and cheap; or claude-sonnet-5-5
    max_tokens: 400,
    system: 'You are a friendly German teacher. The learner is level B1. Reply in simple German, '
          + 'then on a new line starting with "Korrektur:" correct any mistakes in their last message '
          + 'with a short English explanation.',
    messages: history   // [{role:'user',content:'...'},{role:'assistant',content:'...'}, ...]
  })
});
const reply = (await r.json()).content[0].text;
```

### Security
- The key is stored **only in that phone's `localStorage`**. **Never put the key in any file in the repository**, because the repo is public and bots scan GitHub for keys.
- Set a spending limit for the key in the Anthropic console.

---

## 15. Known limitations and ideas

**Not built:**
- Conversation mode (section 14).
- The SVG line icons for concrete words (dropped on request).
- Audio pronunciation of words in the cards (easy to add with `speechSynthesis`, see section 14).

**Behavior to know about:**
- A **wrong** quiz answer (word, verb or grammar rule) sends the card back to box 0; a correct quiz answer does not move it up. Learn grades are the main driver of the schedule.
- Your working level only starts to reflect reality after about 6 quiz answers per level, or right after a placement test.
- The placement test only covers A1, A2 and B1. About 75 words are labeled B2 and are not covered by it.
- Plural-only nouns (*Eltern, Leute, Möbel* …) have no gender color on purpose. 81 nouns have no plural entry because the source dictionary does not list one; you can fill the 9th column by hand.
- Country names (*Deutschland, Italien*) are tagged "proper noun" without gender because they normally take no article.
- Verbs that can take both auxiliaries show the most common one.
- `rank` for words from the first 10 batches was recomputed from real frequency data; very specific compound words get the lowest priority.

**Ideas for later:**
- Fill the 2,790 remaining words from the source lists (more B2 and specialist words).
- Add the audio button, a "weak words" quiz (only box 0 and 1), a words-per-theme study mode, or a dark/light toggle.
- Move progress to a cloud sync (needs a backend, so it breaks the "no server" simplicity).

---

## 16. Where the content comes from

- The word list was assembled by me (Claude) in batches. Many words were **selected using two lists you provided** as pick-lists: the DTZ word list (exam vocabulary, A2 to B1) and the LanGeek A1 to B2 lists. **Translations, themes and example sentences are my own writing**, not copied. Some LanGeek source translations were wrong, so they were not used.
- Real frequency ranks come from the open `wordfreq` data; genders and plurals were checked against the open `german-nouns` dictionary (derived from Wiktionary).
- Verb forms (Präteritum and Perfekt) follow the DTZ list; the other forms are generated by rule and were checked by reading through them.
- Because some of the source lists are copyrighted, **keep the app private to yourself** (personal use on your own phone) and do not publish or redistribute it.
- I can make mistakes in German, especially in rare words and example sentences. If you spot an error, fix it directly in the JSON (edit in place, section 5.4).
