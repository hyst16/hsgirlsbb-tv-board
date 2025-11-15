# Nebraska HS Girls Basketball TV Board

This repo powers a simple TV board that shows local Nebraska high school **girls basketball** schedules for a specific set of schools. It’s designed to run on a TV in an office, lobby, or shop using a full-screen browser.

Data is scraped daily from the **NSAA girls basketball class pages** and saved into a JSON file that the front end reads.

---

## What the board shows

For each configured team, the board displays:

- School name and NSAA class
- Current season record (wins–losses)
- Last game:
  - Opponent
  - Date
  - Result / score (if completed)
- Next game:
  - Opponent
  - Date / time
  - Home / away / neutral
  - Tournament info (if applicable)

Cards are shown in a rotating grid so multiple teams can share one screen.

---

## How the data works

- Scraper: `scraper/scrape_nsaa_girls_basketball.py`
  - Pulls from the NSAA girls basketball class pages (A, B, C1, C2, D1, D2), e.g.:
    - `https://nsaa-static.s3.amazonaws.com/calculate/showclassbbgA.html`
    - `https://nsaa-static.s3.amazonaws.com/calculate/showclassbbgB.html`
    - `https://nsaa-static.s3.amazonaws.com/calculate/showclassbbgC1.html`
    - `https://nsaa-static.s3.amazonaws.com/calculate/showclassbbgC2.html`
    - `https://nsaa-static.s3.amazonaws.com/calculate/showclassbbgD1.html`
    - `https://nsaa-static.s3.amazonaws.com/calculate/showclassbbgD2.html`
  - Normalizes the tables and groups games by team.
  - Writes `data/girls_basketball.json`.

- GitHub Actions workflow: `.github/workflows/scrape_girls_basketball.yml`
  - Runs on a schedule (and can be triggered manually).
  - Installs `requirements.txt`.
  - Runs the scraper.
  - Commits updated `data/girls_basketball.json` back to the repo if anything changed.

The front end (`index.html`) then fetches `./data/girls_basketball.json` and renders the cards.

---

## Configuring which teams show up (teams.json)

The file `teams.json` defines groups of teams (usually by office or TV location).

Example:

```json
{
  "yanka-office": [
    "Aquinas Catholic",
    "David City",
    "East Butler"
  ],
  "northbend-office": [
    "North Bend Central",
    "Arlington",
    "Logan View/Scribner-Snyder",
    "Schuyler"
  ],
  "tarnov-office": [
    "Columbus",
    "Columbus Lakeview",
    "Scotus Central Catholic",
    "Humphrey-Lindsay Holy Family",
    "Twin River"
  ]
}
```

Notes:

- The keys (`"yanka-office"`, `"northbend-office"`, etc.) are your `office` IDs.
- Each array lists the **exact team names** as they appear in the NSAA schedules.
- The URL’s `office` parameter must match one of these keys.

You can edit this file to match whatever teams you want on each screen.

---

## Hosting / Viewing the board

You can host this:

- As a **GitHub Pages** site (point it at the repo’s root), or
- On any static host that serves `index.html` and the `data` folder.

Once hosted, point a TV or browser to the site URL with the appropriate query parameters (see below).

Example base URL (GitHub Pages):

```text
https://your-username.github.io/hsgirlsbb-tv-board/
```

---

## URL parameters (how to control the board)

The board is controlled by query parameters added to the URL.

Example:

```text
https://your-username.github.io/hsgirlsbb-tv-board/?office=northbend-office&per=2&rotate=10
```

### Supported parameters

| Param      | Required? | Default        | Example                        | What it does                                                                                       |
|-----------|-----------|----------------|--------------------------------|----------------------------------------------------------------------------------------------------|
| `office`  | Yes*      | `mead-office`  | `office=northbend-office`     | Selects which group of teams to show. Must match a key in `teams.json`.                           |
| `per`     | No        | `2`            | `per=3`                        | Number of teams per screen (per “page”). Must be 1 or higher.                                     |
| `rotate`  | No        | `10` seconds   | `rotate=15`                    | Number of seconds each page is shown before rotating to the next. Minimum of about 5 seconds.     |
| `mode`    | No        | `compact`      | `mode=full` or `mode=compact` | Card layout. `compact` = summary cards. `full` = more detailed cards with extra fields.           |
| `cardmin` | No        | CSS default    | `cardmin=260`                  | Minimum card width in pixels. Increase to make larger cards (fewer per row), decrease to fit more.|
| `anim`    | No        | `on`           | `anim=off`                     | Turn fade animations and the progress bar on or off. `anim=off` disables both (instant page swap).|

\*There is an internal default of `office=mead-office`, but unless you have a `mead-office` entry in `teams.json`, you will see a “no teams configured” message. In practice, you should always pass an `office` parameter.

---

## URL examples

### 1. Simple setup – 2 teams per screen, quick rotation

```text
?office=yanka-office&per=2&rotate=10
```

- Uses the teams listed under `yanka-office`.
- Shows 2 teams at a time.
- Rotates every 10 seconds.

---

### 2. More teams per screen, slower rotation

```text
?office=northbend-office&per=3&rotate=20
```

- Shows 3 teams at a time.
- Rotates every 20 seconds.
- Good for larger TVs where more cards fit comfortably.

---

### 3. Detailed “full” mode

```text
?office=tarnov-office&per=2&rotate=15&mode=full
```

- Uses the full card layout with more line-by-line details.
- Shows 2 teams per page.
- Rotates every 15 seconds.

---

### 4. Tighter cards (more columns)

```text
?office=northbend-office&per=4&rotate=15&cardmin=220
```

- Minimum card width is reduced to 220px.
- Allows more cards per row on wide screens (if the screen resolution supports it).

---

### 5. No animations (instant switching)

```text
?office=yanka-office&per=2&rotate=12&anim=off
```

- Disables fade animations and the progress bar.
- Pages swap instantly at the rotation interval.

---

## Running locally (optional)

If you want to run this locally for testing:

1. Make sure you have Python installed.
2. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

3. Run the scraper to generate `data/girls_basketball.json`:

   ```bash
   python scraper/scrape_nsaa_girls_basketball.py
   ```

4. Start a simple HTTP server from the repo root:

   ```bash
   python -m http.server 8000
   ```

5. Visit in your browser:

   ```text
   http://localhost:8000/?office=yanka-office&per=2&rotate=10
   ```

Then, once everything looks good, push to GitHub and set up GitHub Pages plus the scheduled Action to keep the data fresh.
