# 🤖 AI Data Analyst Chatbot (Nova)

A local n8n-powered chatbot that answers sales questions from your Google Sheets. Works entirely on your machine.

![Chatbot Demo](assets/chat-interface.png)

## ✨ What it does
- Natural-language questions about **Products**, **Customers**, **Orders**
- Validates your Google Sheet and gives friendly error messages
- Uses Groq LLM (gpt-oss-120b) with short-term memory
- Clean, branded chat UI

## 🧱 Architecture
User → **n8n Chat Trigger** → Google Sheets (3 tabs) → **Code validation/normalization** → **AI Agent** → Chat response

![Architecture](assets/architecture-diagram.png)

## ✅ Prereqs
- n8n running **locally** at `http://localhost:5678`
- Google account with **Sheets API** enabled
- Groq API key
- A Google Sheet with 3 tabs (exact names): `Products`, `Customers`, `Orders`

### Required headers
**Products:** `Product ID | Product Name | Category | Unit Price ($) | Stock Quantity`  
**Customers:** `Customer ID | First Name | Last Name | Email | City`  
**Orders:** `Order ID | Customer ID | Product ID | Order Date | Quantity | Total Amount ($)`  
**Dates:** Use `DD/MM/YYYY` (e.g., `15/09/2024`)

A template is in `data/sample-google-sheet-template.xlsx`.

## 🚀 Run locally (no public hosting)
1. **Start n8n** (local): open `http://localhost:5678`
2. **Import the workflow**  
   - n8n → *Import from File* → select `workflows/ai-data-analyst-v1.json`
3. **Set credentials**
   - Open each **Google Sheets** node → create **Google Sheets OAuth2** credential → connect your account
   - Open **Groq Chat Model** node → add **Groq API** credential with your key
4. **Point to your Sheet**
   - In all 3 Sheets nodes, set the `documentId` to your **Google Sheet ID** (from its URL)
5. **Activate the workflow** (toggle “Active”)
6. **Open the chat**
   - Open the **Chat Trigger** node and copy the **Chat URL** it shows (on local it usually looks like  
     `http://localhost:5678/webhook/<id>/chat`)  
   - Paste that URL below so anyone on your machine can click and chat:

**Chat now:** 👉 [Open Nova locally](YOUR_CHAT_URL)

> Notes:
> - The Chat URL is visible inside the Chat Trigger node UI.  
> - Each user message triggers the workflow anew; connect a memory node if you want session recall. :contentReference[oaicite:0]{index=0}

## 🧪 Sample questions
- “Total revenue for September?”
- “Top 5 customers by spend”
- “Products low on stock (< 10)”
- “Revenue by category last 3 months”

## 🛠 Troubleshooting (local)
- **Chat shows but doesn’t reply** → Ensure workflow is *Active* and your credentials are valid.  
- **Validation errors** → Check exact tab names and headers (must match, including `($)` symbols).  
- **Date parsing off** → Use `DD/MM/YYYY`, not formulas.  
- **Chat URL** → Copy it from the Chat Trigger node; local pattern is `/webhook/<id>/chat`. :contentReference[oaicite:1]{index=1}

## 🔐 Security (local)
- Don’t commit credentials or API keys.
- Keep `.n8n/` and `.env` in `.gitignore`.

## 📜 License
MIT — see `LICENSE`.
