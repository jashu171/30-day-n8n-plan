# AI Agent Data Analyst — n8n Workflow (All-in-README Edition)

This package contains a complete, **copy‑pasteable** guide to build the workflow:
- Download CSV from Google Drive
- Clean & impute nulls (numeric columns → fill with mean)
- Summarize with an **AI Agent**
- Render a **line chart** with QuickChart
- Email an HTML report with Gmail


---

## ✅ Prerequisites
- n8n server
- Credentials:
  - Google Drive (OAuth2)
  - Gmail (OAuth2) or SMTP
  - LLM provider: Google AI (Gemini) or OpenAI

---

## 🔁 Node order
1. **Manual Trigger**
2. **Google Drive → Download file**
3. **Extract from File** (CSV)
4. **Code (JavaScript)** — *Data Preparation*
5. **AI Agent** (with **Chat Model** wired to it)
6. **Merge** (append)
7. **Code (JavaScript)** — *Email body + Line Chart*
8. **Gmail** — *Send a message*

---
## Canvas WorkFlow
![Data prep output](images/Screenshot%202025-09-18%20125756.png)
---

## 🧩 Node setup

### 1) Google Drive → Download file
- **Operation:** `Download`
- **File ID:** your CSV file id (e.g. `1wy6...`)
- **Binary property:** `data` (default)
---
## Drive CSV file 
![Email result](images/Screenshot%202025-09-18%20125812.png)


### 2) Extract from File
- **Binary property:** `data`
- **File format:** `CSV`
- **Header row:** ON (if present)

### 3) Code (JavaScript) — Data Preparation
Mode: **Run Once for All Items**. Paste the code below:

```javascript
// INPUT: items = one item per CSV row (from Extract from File)
// OUTPUT: a single consolidated item with cleaned data, stats, and a compact summary for AI

function isNullish(v) {
  if (v === null || v === undefined) return true;
  if (typeof v === 'string') {
    const s = v.trim().toLowerCase();
    return s === '' || s === 'na' || s === 'null' || s === 'none' || s === 'nan';
  }
  return false;
}

const rows = $input.all().map(it => it.json);
if (!rows.length) {
  return [{ json: { rowCount: 0, columns: [], cleanedRows: [], numericColumns: [], means: {}, missingCounts: {}, summaryText: 'No data rows found.' } }];
}

const columns = Object.keys(rows[0] || {});

// detect numeric columns (at least ~60% parseable)
const numericColumns = columns.filter(col => {
  let total = 0, parseable = 0;
  for (const r of rows) {
    const v = r[col];
    if (!isNullish(v)) {
      total++;
      if (!Number.isNaN(parseFloat(String(v).replace(/,/g, '')))) parseable++;
    }
  }
  if (total === 0) return false;
  return (parseable / total) >= 0.6;
});

// compute means ignoring nullish
const sums = {}, counts = {}, means = {};
numericColumns.forEach(c => { sums[c] = 0; counts[c] = 0; });

for (const r of rows) {
  for (const c of numericColumns) {
    const v = r[c];
    if (!isNullish(v)) {
      const num = parseFloat(String(v).replace(/,/g, ''));
      if (!Number.isNaN(num)) {
        sums[c] += num;
        counts[c] += 1;
      }
    }
  }
}

for (const c of numericColumns) {
  means[c] = counts[c] ? (sums[c] / counts[c]) : 0;
}

// replace nulls in numeric columns with mean
const cleanedRows = rows.map(r => {
  const out = { ...r };
  for (const c of numericColumns) {
    const v = r[c];
    if (isNullish(v)) {
      out[c] = means[c];
    } else {
      const num = parseFloat(String(v).replace(/,/g, ''));
      out[c] = Number.isNaN(num) ? means[c] : num;
    }
  }
  return out;
});

// missing counts per column (pre-clean)
const missingCounts = {};
for (const c of columns) {
  let m = 0;
  for (const r of rows) if (isNullish(r[c])) m++;
  missingCounts[c] = m;
}

const rowCount = cleanedRows.length;
const maxRowsForAI = Number($json.maxRowsForAI ?? 200);
const previewRows = cleanedRows.slice(0, Math.min(rowCount, maxRowsForAI));

// build a compact, AI-friendly summary
const lines = [];
lines.push(`Rows: ${rowCount}`);
lines.push(`Columns: ${columns.join(', ')}`);
lines.push(`Numeric Columns: ${numericColumns.join(', ') || '(none)'}`);
lines.push(`Means: ${JSON.stringify(means)}`);
lines.push(`Missing (before clean): ${JSON.stringify(missingCounts)}`);
lines.push(`Sample (first ${previewRows.length} rows):`);
lines.push(JSON.stringify(previewRows, null, 2));

const summaryText = lines.join('\n');

return [{
  json: {
    rowCount,
    columns,
    numericColumns,
    means,
    missingCounts,
    cleanedRows,
    summaryText
  }
}];

```

**Outputs:** `columns`, `numericColumns`, `means`, `missingCounts`, `cleanedRows`, `summaryText`

### 4) AI Agent
- **System Instruction** (paste exactly as below)
- **User Message**:
```
Here is a compact summary of a cleaned CSV:

{ $json.summaryText }

Please produce the JSON object as specified in the system instruction.
```

#### System Instruction
```
You are a data summarization and reasoning assistant.
You always output valid, minified JSON with the following schema:

{
  "subject": "string",
  "summary_html": "string (HTML body; short but information-dense; must be valid HTML)",
  "reasoning": "string (plain text with concise insights)"
}

Rules:
- Do not include markdown in summary_html; use clean HTML tags only.
- Base your summary and reasoning strictly on the user-provided CSV summary.
- Keep the language precise and neutral. No fluff.

```

> The Agent returns a single string field named `output` that contains:
> ```
> {"subject":"...","summary_html":"...","reasoning":"..."}
> ```

### 5) Merge
- **Mode:** `append`  
This produces two items into the next node:
- Item A: ``{ output: "<AI JSON string>" }``
- Item B: ``{ columns, numericColumns, means, cleanedRows, ... }``

### 6) Code (JavaScript) — Email body + Line Chart
Mode: **Run Once for All Items**. Paste the code below:

```javascript
// n8n Code (JavaScript) — works with Merge output (two items)
// Input items:
//  - AI item: { output: "<stringified JSON: {subject, summary_html, reasoning}>" }
//  - Data item: { columns, numericColumns, means, cleanedRows, summaryText, ... }
//
// Output: { email_subject, email_html, chart_url, columns, numericColumns }

function robustParseJson(text) {
  const t = String(text || '').trim();
  if (!t) return {};
  try { return JSON.parse(t); } catch {}
  const s = t.indexOf('{'), e = t.lastIndexOf('}');
  if (s !== -1 && e !== -1 && e > s) {
    try { return JSON.parse(t.slice(s, e + 1)); } catch {}
  }
  return {};
}

function isNullish(v) {
  if (v === null || v === undefined) return true;
  if (typeof v === 'string') {
    const s = v.trim().toLowerCase();
    return s === '' || s === 'na' || s === 'null' || s === 'none' || s === 'nan';
  }
  return false;
}

function detectNumericColumns(rows) {
  if (!rows?.length) return [];
  const cols = Object.keys(rows[0] || {});
  const out = [];
  for (const c of cols) {
    let seen = 0, ok = 0;
    for (const r of rows) {
      const v = r[c];
      if (isNullish(v)) continue;
      seen++;
      const num = typeof v === 'number' ? v : parseFloat(String(v).replace(/,/g, ''));
      if (Number.isFinite(num)) ok++;
    }
    if (seen > 0 && ok / seen >= 0.6) out.push(c);
  }
  return out;
}

function pickSeries(rows, columns, numericColumns, hint) {
  const dateCol = columns?.find(c => /date|day|timestamp|time/i.test(c));
  const xCol = dateCol || 'index';

  let yCol = '';
  if (hint && numericColumns?.includes(hint)) yCol = hint;
  if (!yCol && numericColumns?.length) yCol = numericColumns[0];

  const labels = [];
  const values = [];
  for (let i = 0; i < (rows?.length || 0); i++) {
    const r = rows[i] || {};
    let label;
    if (dateCol) {
      const v = r[dateCol];
      const d = new Date(v);
      label = isNaN(d.getTime()) ? String(v) : d.toISOString().slice(0,10);
    } else {
      label = String(i + 1);
    }
    labels.push(label);
    const raw = r[yCol];
    const num = typeof raw === 'number' ? raw : parseFloat(String(raw).replace(/,/g, ''));
    values.push(Number.isFinite(num) ? num : NaN);
  }
  return { xCol, yCol, labels, values };
}

function downsample(labels, values, maxPoints = 200) {
  const n = labels.length;
  if (n <= maxPoints) return { labels, values };
  const step = Math.ceil(n / maxPoints);
  const L = [], V = [];
  for (let i = 0; i < n; i += step) { L.push(labels[i]); V.push(values[i]); }
  return { labels: L, values: V };
}

function buildQuickChartURL(cfg) {
  const base = 'https://quickchart.io/chart?width=900&height=420&devicePixelRatio=2&c=';
  return base + encodeURIComponent(JSON.stringify(cfg));
}

const escapeHTML = s => String(s || '').replace(/[<>&]/g, m => ({'<':'&lt;','>':'&gt;','&':'&amp;'}[m]));

// ---- Gather BOTH items from Merge ----
const all = $input.all();
let aiStr = '';
let data = {};
for (const it of all) {
  const j = it.json || {};
  if (!aiStr && typeof j.output === 'string') aiStr = j.output;
  if (j.columns || j.numericColumns || j.cleanedRows || j.means || j.summaryText) {
    data = { ...data, ...j };
  }
}

// ---- AI ----
const ai = robustParseJson(aiStr);
const subject = ai.subject || 'Summary of Dataset';
const summaryHtml = ai.summary_html || '<p>No summary_html from AI.</p>';
const reasoning = ai.reasoning || '';

// ---- Data ----
let cleanedRows = Array.isArray(data.cleanedRows) ? data.cleanedRows : [];
let columns = Array.isArray(data.columns) ? data.columns : (cleanedRows[0] ? Object.keys(cleanedRows[0]) : []);
let numericColumns = Array.isArray(data.numericColumns) ? data.numericColumns : [];
const means = data.means || {};
const chartHint = data.chartMetricHint || '';

if (!numericColumns.length && cleanedRows.length) {
  numericColumns = detectNumericColumns(cleanedRows);
}

// ---- Chart URL ----
let chartUrl = '';
let chartTitle = '';

if (numericColumns.length && cleanedRows.length) {
  const { xCol, yCol, labels, values } = pickSeries(cleanedRows, columns, numericColumns, chartHint);
  if (values.some(v => Number.isFinite(v))) {
    const ds = downsample(labels, values, 200);
    const cfg = {
      type: 'line',
      data: { labels: ds.labels, datasets: [{ label: yCol, data: ds.values, fill: false, tension: 0.3 }] },
      options: {
        plugins: { legend: { display: false }, title: { display: true, text: `Trend of ${yCol}` } },
        scales: { x: { title: { display: true, text: xCol } }, y: { title: { display: true, text: yCol } } }
      }
    };
    chartUrl = buildQuickChartURL(cfg);
    chartTitle = `Line Chart — ${yCol}`;
  }
}

// Fallback: means per numeric column
if (!chartUrl) {
  const keys = numericColumns.length ? numericColumns : Object.keys(means || {});
  const vals = keys.map(k => {
    const v = means[k];
    const n = typeof v === 'number' ? v : parseFloat(String(v));
    return Number.isFinite(n) ? n : NaN;
  }).filter(v => Number.isFinite(v));

  if (keys.length && vals.length) {
    const cfg = {
      type: 'line',
      data: { labels: keys, datasets: [{ label: 'Mean', data: vals, fill: false, tension: 0.3 }] },
      options: {
        plugins: { legend: { display: false }, title: { display: true, text: 'Means per Numeric Column' } },
        scales: { x: { title: { display: true, text: 'Metric' } }, y: { title: { display: true, text: 'Mean' } } }
      }
    };
    chartUrl = buildQuickChartURL(cfg);
    chartTitle = 'Line Chart — Means';
  }
}

// ---- Email HTML ----
const emailHtml = `
  <div style="font-family:Arial, Helvetica, sans-serif; line-height:1.5;">
    ${summaryHtml}
    <hr style="margin:24px 0; border:none; border-top:1px solid #ddd;" />
    ${chartUrl ? `
      <h3 style="margin:0 0 8px;">${chartTitle}</h3>
      <img src="${chartUrl}" alt="Line chart" style="max-width:100%; height:auto; display:block;" />
    ` : `
      <p style="color:#777; margin:0;">No numeric series available for charting.</p>
    `}
    ${reasoning ? `
      <hr style="margin:24px 0; border:none; border-top:1px solid #eee;" />
      <details>
        <summary style="cursor:pointer; color:#444;">Model reasoning (compact)</summary>
        <pre style="white-space:pre-wrap; background:#fafafa; padding:12px; border:1px solid #eee; border-radius:6px;">${escapeHTML(reasoning)}</pre>
      </details>
    ` : '' }
  </div>
`;

return [{
  json: {
    columns: columns || [],
    numericColumns: numericColumns || [],
    email_subject: subject,
    email_html: emailHtml,
    chart_url: chartUrl
  }
}];

```

**Outputs:** `email_subject`, `email_html`, `chart_url`

### 7) Gmail — Send a message
- **To:** your email
- **Subject:** `={ $json.email_subject }`
- **Message (HTML):** `={ $json.email_html }`
---
## output Gmail
![Workflow overview](images/Screenshot%202025-09-18%20125543.png)
---

## 🩺 Troubleshooting
- **No chart URL?** Ensure `numericColumns` has at least one column and `cleanedRows` contains parseable numbers. The final Code node will fall back to a **means** chart if row-series fails.
- **Two inputs at Merge:** Keep `append` so the final node sees both AI and data items. Script reads from `$input.all()`.
- **Email blocked images:** QuickChart returns a PNG via HTTPS; most clients (incl. Gmail) display it inline.

---

## 🧪 Optional tweaks
- Add a **Set** node up front for `fileId`, `emailTo`, and `chartMetricHint`.
- Attach the cleaned CSV to the email.
- Swap Chat Model to Gemini 1.5 for faster summaries.

Enjoy! 🚀
