# Nova — Detailed Setup & Developer Guide

This document covers the **data contract**, **local setup**, **configuration**, **how it works**, **error handling**, **charting**, **customization**, and **troubleshooting**.

---

## 📄 Data Contract

Create a **Google Sheet** with **three tabs named exactly**:

### 1) `Products`
| Product ID | Product Name | Category | Unit Price ($) | Stock Quantity |

### 2) `Customers`
| Customer ID | First Name | Last Name | Email | City |

### 3) `Orders`
| Order ID | Customer ID | Product ID | Order Date | Quantity | Total Amount ($) |

- **Dates:** `DD/MM/YYYY` (e.g., `15/09/2024`)
- **Totals rule:** Prefer recorded **“Total Amount ($)”**. If missing, fallback to **Unit Price × Quantity** from `Products`.

Template: `data/sample-google-sheet-template.xlsx`

---

## ⚡ Quick Start (Local)

**Prerequisites**
- n8n running locally at `http://localhost:5678`
- Google account with Sheets API access
- Groq API key

**1) Import workflows**
- n8n → **Import from File**
  - `workflows/ai-data-analyst-v1.json` (Main)
  - `workflows/generate-chart.json` (Sub-workflow)

**2) Set credentials**
- **Google Sheets OAuth2** → connect your account
- **Groq** → add API credential (your key)

**3) Point to your Sheet**
- In all **3 Google Sheets nodes** (Main workflow), set `documentId` to your Sheet ID (from its URL)  
- Tabs must be named exactly: **Products**, **Customers**, **Orders**

**4) Activate**
- Toggle the **Main** workflow to **Active**  
- (Optional) Sub-workflow can remain inactive; it’s invoked by the tool node

**5) Open the chat**
- Open the **Chat Trigger** node
- Copy the **Chat URL** (`/webhook/<id>/chat`) and open it in your browser

---

## ⚙️ Configuration

The **agent system message** enforces:
- Answer **only sales questions** using the **JSON** built from Sheets
- **No fabrication** → ask for missing data
- **Date parsing:** day-first `DD/MM/YYYY` → ISO for grouping
- **Computation:** programmatic sums/aggregations only (use calculator/tool)
- **Formatting:** currency/percent with two decimals
- **Transparency:** list included **Order IDs** per group, show subtotals before totals
- **No meta:** don’t mention prompts or node internals
- **Chart tool policy:** only when asked or when it clearly finalizes the answer (one chart/turn)

The **Chat Trigger** includes custom CSS for the UI branding.

---

## 🧠 How It Works (Nodes Overview)

### 1) Data ingestion (3 Google Sheets nodes)
- Reads the `Products`, `Customers`, `Orders` tabs
- Aggregators wrap rows into `products`, `customers`, `orders`

### 2) Validation & Normalization (Code node)
Key behaviors:
- Detects **missing tabs** (including “error arrays”)
- Flags **empty tabs**
- Enforces **required headers** (exact match, including symbols like `($)`)
- **Coerces numbers**
- Parses **day-first dates** to ISO (`YYYY-MM-DD`)
- Fills **missing totals** with `quantity * unit_price` when safe
- Emits a structured **health** object with counts

### 3) Error guard (If node)
- If errors/missing/empty: returns **friendly messages** telling the user what to fix
- Otherwise → proceed to the agent

### 4) AI Agent (Groq)
- Model: `openai/gpt-oss-120b` (low temperature)
- Memory: last 3 turns
- Tools:
  - **Calculator** — programmatic math  
  - **Generate a chart** — sub-workflow tool

### 5) Chart generation (Sub-workflow)
- Model: `llama-3.3-70b-versatile` generates **Chart.js v2** config (JSON only)
- JSON sanitized & validated → QuickChart URL built and returned to the main workflow

---

## 🧯 Error Handling & Validation

Examples:
- **Missing tabs:**  
  “I couldn’t find the tabs “Products”, “Orders”… Please check tab names.”
- **Empty tabs:**  
  “Your sheet is connected, but these tabs are empty: Customers…”
- **Header mismatch:**  
  “In the “Products” tab, I couldn’t find: Unit Price ($). I see: Price…”

Defensive notes:
- Detects common “error array” shapes from Sheets
- Exact header matches (including symbols like `($)`)
- Fallback totals **only** if explicit totals are missing
- Day-first date parsing spot-check (e.g., `03/01/YYYY` → Jan 3)

---

## 📊 Charts

- **One** chart per turn (enforced)
- Charting contract:
  - Chart.js **v2** JSON with top-level `type`, `data`, `options`
  - `options.scales.yAxes[0].ticks.beginAtZero = true`
  - Single-series style fixed for brand consistency
  - Labels and data **must align**; invalid points dropped to preserve parity
- Rendering: **QuickChart** URL returned to the agent

---

## 🎛️ Customization

- **Branding/UI:** edit CSS inside the Chat Trigger node  
- **Models:** switch Groq models, or wire different providers supported by your n8n  
- **Memory:** adjust window size or memory strategies  
- **Validation:** add business rules (e.g., city allowlist, min order quantity)  
- **Chart policy:** change when charts auto-trigger

---

## 🧩 Troubleshooting

**Chat shows but no reply**
- Ensure the **Main workflow is Active**
- Verify **Google Sheets** and **Groq** credentials

**Validation errors**
- Check tab names & headers **exactly**
- Ensure at least one data row exists under each header

**Date parsing looks wrong**
- Use `DD/MM/YYYY` literals (avoid formulas where possible)

**Chart not appearing**
- Confirm the **Generate Chart** sub-workflow file is imported
- Verify the tool connection in the main workflow
- Ensure numeric arrays are valid and aligned with labels
