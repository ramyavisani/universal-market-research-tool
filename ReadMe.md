# 📄 Technical Project Report: Universal Market Research Tool

## 1. Project Overview
The **Universal Market Research Tool** is an automated OSINT (Open Source Intelligence) application designed to streamline B2B lead generation and company scouting. By combining a user-friendly frontend with a powerful low-code backend, the tool allows users to input natural language research queries (e.g., *"Top 10 AI startups in Berlin"*) and receive structured, verifiable data in a Google Spreadsheet.

### Key Objectives
* **Automation:** Replace manual web searching and data entry with autonomous AI agents.
* **Structured Data:** Convert unstructured web data into standardized formats (JSON/CSV).
* **Scalability:** Decoupled architecture allowing for high-volume queries without browser crashes.

---

## 2. System Architecture
The application follows a **Headless Architecture** pattern, separating the frontend (User Interface) from the logic layer (n8n).

### Technology Stack
* **Frontend:** HTML5, CSS3 (Dark Mode), Vanilla JavaScript.
* **Orchestrator (Backend):** n8n (Self-hosted on Docker/MacBook M2).
* **Intelligence Engine:** Perplexity API (Model: `sonar-pro`).
* **Database/Storage:** Google Sheets (via API).

---

## 3. Workflow Logic (The "Backend")
The core logic is hosted on an n8n workflow consisting of four sequential stages:

### Stage 1: The Trigger (Webhook)
* **Type:** `POST` Webhook.
* **Function:** Acts as the API gateway. It receives the JSON payload (`{ "user_prompt": "..." }`) from the HTML frontend.
* **Protocol:** Configured to accept CORS requests from the local client and respond immediately to prevent UI freezing.

### Stage 2: The Intelligence Agent (Perplexity API)
* **Model:** `sonar-pro` (Online search enabled).
* **Prompt Engineering:** A strict system prompt is used to enforce JSON-only output, stripping away conversational filler.
    * *System Prompt:* "You are a B2B Market Intelligence Analyst. Return valid JSON."
* **Data Extraction:** The agent extracts 12 key data points per entity, including Revenue Estimates, Tech Stack, and Decision Maker contact info.

### Stage 3: The Normalizer (JavaScript Code Node)
* **Problem:** LLMs often output "dirty" JSON (wrapped in Markdown or containing trailing text).
* **Solution:** A custom Regex parser `response.match(/\[[\s\S]*\]/)` isolates the JSON array.
* **Session Management:** The code dynamically generates a **"Header Row"** containing the search topic and timestamp. This separates distinct search sessions in the Google Sheet, preventing data confusion.

### Stage 4: Persistence (Google Sheets)
* **Action:** `Append` operation.
* **Mapping:** Data is auto-mapped to columns (`Entity Name`, `Industry`, `Email`, etc.). The specific "Header Rows" created in Stage 3 are also written here to visually segment the data history.

---

## 4. Technical Implementation Details

### A. The JSON Parser (JavaScript)
To ensure system stability, the Code Node implements error handling and session tagging.

```javascript
// Logic to handle session separation
let userTopic = $('Webhook').item.json.body.user_prompt || "General Search";

// Create visual separation in the database
const headerRow = {
  'Entity Name': `━━━ 🔍 TOPIC: ${userTopic} (${new Date().toLocaleDateString()}) ━━━`,
  'Industry Niche': '---'
};

// Combine empty spacing, header, and actual data
const finalOutput = [emptyRow, headerRow, ...dataRows];

## B. The Frontend Interface

The dashboard is a single-file HTML application (`html form.html`). It handles the POST request using the browser's **fetch API**.

- **State Management:** The button enters a “Loading” state to prevent double submissions.  
- **Error Handling:** Catches network errors if the n8n instance is offline.

---

## 🚀 5. Installation & Setup

### **Prerequisites**
- An instance of **n8n**  
- A **Perplexity API Key**  
- A **Google Cloud Service Account** (for Sheets access)

### **Step-by-Step**

1. **Backend:**  
   Download `Market_Research_Tool.json` from this repository and import it into n8n.

2. **Credentials:**  
   Set up your Perplexity and Google Sheets credentials in n8n.

3. **Frontend:**  
   Open `html form.html` in a text editor and replace  
   ```js
