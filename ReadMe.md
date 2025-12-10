# 🕵️‍♂️ AI-Powered Market Research Assistant

**A lightweight, full-stack tool that automates market research using n8n, Perplexity AI, and Google Sheets.**

I built this project to automate the tedious process of gathering market data. Instead of manually Googling and summarizing, this tool takes a query, performs a grounded search via Perplexity's `sonar-pro`, cleans the data, and logs structured results directly into Google Sheets.

---

## 🛠 Tech Stack

- **Frontend:** Vanilla HTML5, CSS, & JavaScript (Fetch API).
- **Orchestration:** [n8n](https://n8n.io/) (Workflow automation).
- **Intelligence:** Perplexity API (`sonar-pro` model) for real-time, internet-grounded data.
- **Database:** Google Sheets (via Google Cloud Service Account).

---

## ⚙️ How It Works Under the Hood

This isn't just a wrapper for ChatGPT. I had to solve a few specific engineering problems to make this reliable. Here is the flow:

### 1. The Frontend (`html form.html`)
The interface is a single-file application. I avoided heavy frameworks like React to keep it deployable anywhere (even locally).
- **State Management:** When you click submit, the button enters a `Loading...` state. This is crucial because API calls to Perplexity can take 10-30 seconds; this prevents users from rage-clicking and double-charging the API.
- **Error Handling:** It catches network errors specifically to handle scenarios where the n8n self-hosted instance might be asleep or unreachable.

### 2. The Backend Logic (n8n)
The n8n workflow acts as the API gateway. It receives the webhook from the frontend and orchestrates the logic:
- **The Model Choice:** I specifically chose `sonar-pro`. Unlike generic GPT-4, this model is "grounded," meaning it searches the live internet before answering. This significantly reduces hallucinations when asking about current market trends.
- **The "JSON Problem":** LLMs are notoriously bad at outputting strict JSON. They often wrap the response in Markdown code blocks (e.g., ` ```json {data} ``` `).
- **The Regex Fix:** To solve this, I implemented a robust Regex extractor in n8n. It strips away the Markdown formatting to ensure the data passed to Google Sheets is valid, parseable JSON every time.

### 3. Data Persistence (Google Sheets)
- **Session Tracking:** The JavaScript logic includes dynamic "Header Rows." Instead of just appending rows blindly, the system logs query history intelligently, allowing me to see which research session produced which insights.

---

## 🚀 Installation & Setup

If you want to spin this up yourself, here is what you need.

### Prerequisites
1. A self-hosted or cloud instance of **n8n**.
2. A **Perplexity API Key** (needs credits).
3. A **Google Cloud Service Account** with read/write access to Sheets.

### Step-by-Step Guide

**1. The Backend (n8n)**
- Download the workflow file `Market_Research_Tool.json` from this repository.
- Import it into your n8n instance.
- Add your Perplexity and Google Cloud credentials to n8n's credential manager.

**2. The Frontend**
- Open `html form.html` in any text editor (VS Code, Notepad, etc.).
- Locate the constant at the top of the script:
  ```javascript
  const WEBHOOK_URL = "YOUR_N8N_PRODUCTION_URL_HERE";
  ```
- Replace it with your specific n8n Webhook URL.

**3. Run It**
- Simply open `html form.html` in your browser. No `npm install` or build server required.

---

## 🧠 Challenges & Solutions (Dev Log)

Building this wasn't entirely smooth. Here are the specific hurdles I hit and how I fixed them:

| Challenge | The Fix |
| :--- | :--- |
| **Hallucinations** | Generic models made up stats. Switching to **`sonar-pro`** (search-grounded) fixed this by forcing the AI to cite live sources. |
| **Broken JSON** | The AI kept returning Markdown text. I wrote a **Regex extractor** node in n8n to forcibly clean the output before parsing. |
| **Rate Limiting** | Users could spam the submit button. I added **frontend state management** to disable the button until the `fetch` request resolves. |
| **Context Loss** | It was hard to match rows to queries. I added **Session Tracking** logic to log the original query alongside the results in the Sheet. |

---

## 🔮 Future Improvements

- **Authentication:** Currently, the frontend is open. I plan to add basic API Key auth or a login screen.
- **Export to PDF:** Add a feature to generate a formatted PDF report from the JSON data.
- **Multi-Model Support:** Allow the user to toggle between `sonar-pro` and `gpt-4o` via the frontend UI.

---
*Created as a personal utility tool. Feel free to fork and adapt!*
