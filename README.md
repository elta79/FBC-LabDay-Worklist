# Daily Lab & Handout Worklist

Takes the day's appointment schedule as a PDF and produces a printable
three-column draw sheet: patient and time, lab checkboxes, handout checkboxes.

## How it works

One static HTML file. No server, no database, no build step.

The schedule PDF is read **entirely inside the browser** by pdf.js. Nothing is
uploaded, nothing is stored, nothing is logged. Close the tab and it's gone.
The hosted site itself contains no patient information at any point.

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole application — markup, styles, parsing rules, logic |
| `pdf.min.js` | pdf.js 3.11.174, served locally so clinic network filters can't break it |
| `pdf.worker.min.js` | pdf.js background worker |
| `netlify.toml` | Publish settings, noindex headers, cache rules |
| `robots.txt` | Keeps the site out of search results |

## Changing the clinical rules

Everything clinical lives in one place: the `RULES` array near the top of the
`<script>` block in `index.html`. Each entry looks like:

```js
{ id:'36wk', label:'36 Week Labs',
  match:/\b3[5-7]\s*(wk|week)/i,
  labs:['CBC','TIBC','Gen Cult','GBS Cult'],
  handouts:['Birth Pref','Doula Q'] }
```

- `match` — the pattern tested against the appointment reason line
- `labs` / `handouts` — the checkbox items that print in each column
- `{t:'Other', write:true}` renders a checkbox with a blank write-in line
- `{t:'Collect $120', flag:true}` renders it in amber as an action item

Two gates sit above the rules:

- `NEEDS_LAB` — a line must contain "lab" or "labs" to produce a row
- `LAB_EXEMPT` — rule IDs allowed to bypass that. Currently `sneak`, since
  Sneak Peek never carries the word but still needs a row.

**Order matters.** `RULES` is matched top to bottom, first hit wins. The
`generic` rule at the bottom is the catch-all for any line saying "labs" that
matches nothing more specific, so nobody gets silently dropped. Keep it last.

## Deploying

Netlify, connected to this repo:

- Build command: *(empty)*
- Publish directory: `.`

Commit to `main` and Netlify redeploys in under a minute. Staff get the change
on their next page load.

## Notes

- The site is public but unindexed. It holds no patient data — only the lab
  panels and the Sneak Peek price. If that changes, put it behind auth.
- If patients go missing from the sheet, open "What the reader pulled out of
  the PDF" in the app. It lists every line read, plus anything that matched a
  visit type but was skipped for not saying "labs".
