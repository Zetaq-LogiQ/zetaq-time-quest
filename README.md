# ZetaQ Time Quest

Chapter-wise memory games for the Maharashtra State Board (e-Balbharati) History & Civics books, Classes 6 to 9.
Each chapter has 40 questions (8 per level, L1 easy to L5 hard). The total is 71 chapters and 2,840 questions.

| Class | Book | Chapters | Questions | App |
|---|---|---|---|---|
| 6 | History and Civics, 2026 edition (603030584) | 10 History + 6 Civics | 640 | `class6/index.html` |
| 7 | History and Civics (703000584) | 13 History + 6 Civics | 760 | `class7/index.html` |
| 8 | History and Civics, 2018 edition (803000584) | 14 History + 6 Civics | 800 | `class8/index.html` |
| 9 | History and Political Science, 2017 edition (2021 reprint) | 10 History + 6 Political Science | 640 | `class9/index.html` |

## Games

Each chapter is a trail of stops. A chapter shows only the games its facts support.
For example, ancient-history chapters with no exact years have no Year Lock.

- **Time Train**: put events in order.
- **Year Lock**: dial the year.
- **Who Am I?**: guess from clues.
- **Fix the Chat**: tap the one wrong fact and correct it.
- **Quick Quiz**
- **Boss Round**

After every question the student sees their answer, the right answer and a memory hook.
The app also has spaced revision ("Revise"), "My Mistakes" and a skills view.

## What is in this repo

```
index.html              landing page that links to each class
logo.png                used by the landing page only (each app has its logo built in)
class6/index.html       one self-contained app per class: questions, icons, logo and code are all inside the file
...
class9/index.html
data/c6/_chapters.json  chapter list (id, number, title, unit)
data/c6/01.json ...     one file per chapter: the fact sheet (book quote + page) and the 40 questions
...
```

The apps are **static files**. They need no server code, database or build step.

## Hosting on the LogiQ server

1. Clone the repo once and copy the folder to the static web root, for example `https://logiq.zetaq.in/apps/time-quest/`:
   ```
   git clone https://github.com/Zetaq-LogiQ/zetaq-time-quest.git
   ```
2. To update later, run `git pull` and redeploy the changed files.
3. Open a class with its folder URL, for example `/apps/time-quest/class8/`.

**Serve from a `zetaq.in` address.** iPad Safari blocks or clears storage for iframes that come from another site.
On `logiq.zetaq.in` or any other `*.zetaq.in` subdomain the app counts as the same site, so progress saves normally.
Do not embed it from github.io or any other domain.

**Caching.** Each app is about 0.8 MB. Use `Cache-Control: no-cache` (revalidate) or add a version query (`?v=2`) after an update.
Gzip or Brotli shrinks the files a lot.

**Fonts.** The apps load Baloo 2, Atkinson Hyperlegible and Caveat from Google Fonts. Offline, system fonts are used and the app still works.

## Saving scores in LogiQ

Progress lives in the browser's `localStorage`, under the key `zq_timequest_c<class>`.
That storage belongs to one device and can be cleared, so **LogiQ should store the attempts on its server.**

After every answer the app reports the attempt in two ways.

**1. When it runs inside an iframe**, it sends a `postMessage` to the parent page:
```js
{ type: "zq-timequest-attempt", class: 8, answer: "...",
  t: 1761300000000, qid: "c8-04-12", ch: "04", type: "lock", lv: 3,
  dims: ["year"], s: 0.5, ms: 8400, hints: 0, misses: 1, box: 1 }
```

| field | meaning |
|---|---|
| t | timestamp (ms) |
| qid / ch / type / lv | question id, chapter id, game type, level 1-5 |
| dims | skill dimensions tested: chrono, year, people, place, event, term |
| s | score 0..1 (part marks for a close year, wagons in place, clues used or wrong taps) |
| ms | response time in ms |
| hints / misses | clues opened / wrong taps |
| box | spaced-revision box after this answer (1-5) |

**Listener on the LogiQ page:**
```js
window.addEventListener("message", (e) => {
  if (!e.origin.endsWith(".zetaq.in")) return;
  const m = e.data;
  if (m && m.type === "zq-timequest-attempt") {
    fetch("/api/history/attempts", {                     // your endpoint
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(m)
    });
  }
});
```

**2. When it is the page itself** (no iframe), it calls `window.ZQ_onAttempt(record)` if that function exists, with the same object minus `type`.

## Skill vector (non-linear, normalised)

For each dimension d over the attempts that test d:
```
w_i    = levelWeight(level_i) * 2^(-ageDays_i / 14)        levelWeight = [0.6, 0.8, 1, 1.25, 1.5] for L1..L5
s'_i   = s_i * (0.92 + 0.08 * e^(-ms_i / 20000))            small, capped speed bonus
raw_d  = sum(w_i * s'_i) / sum(w_i)
conf_d = 1 - e^(-sum(w_i) / 3)                              evidence saturation (one lucky answer != mastery)
m_d    = raw_d * conf_d                                      in [0, 1]
vector = [m_chrono, m_year, m_people, m_place, m_event, m_term]
unit   = vector / ||vector||                                 L2-normalised "shape" for comparing students
```

`window.ZQ.exportScores()` returns `{overall:{mastery,unit,answers}, chapters:{id:{...}}, attempts:[...], formula, ...}`.
Every raw attempt is kept, so the server can recompute any other model later (IRT, Elo, BKT).

**Spaced revision.** A correct answer (score ≥ 0.75) moves the fact one box up, and a miss sends it back to box 1.
The boxes come due after 1, 3, 7, 21 and 45 days.

## How the content was checked

- **Book first.** Every chapter starts from a fact sheet that quotes the book with page numbers.
  Year Lock and Time Train use only years the book states exactly.
- **Independent check.** A second examiner checked every fact and question against the chapter PDF.
  The rules: exactly one correct answer, and every wrong option clearly false by the book.
  A "wrong" word must not be something that is true in real history, or simply missing from a "such as" list.
- **No outdated facts.** For the older books (Class 8: 2018, Class 9: 2017), no question depends on counts or statuses that have changed since.
  Examples: numbers of States, UTs or High Courts, UN membership, and current office-holders.
- **Known book slips are not used as answers.** Examples: the UN Charter signing date, 1965 called the "first" Indo-Pak war, and misprints such as "Eo" for the Ao language.
- **Automated test.** Every app answered all questions right and wrong through the real UI, with 0 failures.
  Layout was checked at 360x640, 390x844, 820x1180 and 1180x820.

---
© ZetaQ LogiQ. Built with Claude Code.
