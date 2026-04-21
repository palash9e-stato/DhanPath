# 💰 DhanPath — My Journey Building an AI-Powered Personal Finance Advisor from Scratch

---

## 🧭 Where It All Started

I looked around and saw the same story everywhere — people earning decent salaries, yet struggling to grow their money. Not because they were careless, but because personal finance in India is genuinely confusing. Between NIFTY, SENSEX, SIPs, EMIs, FDs, mutual funds, and tax-saving instruments, there's no single place that makes sense of it all and actually helps you make decisions.

Most finance apps either give you raw data with no advice, or generic advice with no data. None of them talked *to* you.

That's when I started building **DhanPath** — a platform that combines real market data, AI intelligence, and a personal financial dashboard — all in one place, built specifically for Indian users.

---

## ❓ The Core Problem I Was Solving

**Problem 1 — No personalized guidance**
Generic investment articles and YouTube videos don't know your income, your EMIs, your goals, or your risk appetite. A 22-year-old engineering fresher and a 45-year-old salaried professional should never get the same advice — but almost every platform treats them identically.

**Problem 2 — Market data is scattered**
Stock prices are on one site, mutual fund data on another, financial news on a third, and portfolio tracking in a spreadsheet. There's no unified dashboard.

**Problem 3 — AI tools don't understand finance context**
Generic chatbots like standard ChatGPT don't have real-time market data. They can't tell you Tata Motors' last 3-day stock price, search for news about a company, or reason about whether to invest right now.

**Problem 4 — Financial planning has no visual clarity**
When you say "I have ₹5 lakh, where do I invest?", the answer should be a *visual map* — showing how money flows across index funds, gold, mid-caps, FDs — not a wall of text.

---

## 🏗️ How I Decided to Build It

### Choosing the Tech Stack

**Frontend → React + TypeScript + Vite + Tailwind CSS**

I chose React because the app is heavily interactive — charts, chat interfaces, flow diagrams, real-time news. TypeScript was a must because with this many components and data interfaces (portfolio metrics, news articles, investment nodes), type safety prevents a whole class of silent bugs. Vite over Create React App because the dev server startup is nearly instant and hot module replacement is seamless.

Tailwind CSS made sense because I needed a dark mode toggle, responsive layouts across sidebar/dashboard/full-page views, and consistent design — all without writing a separate CSS file for every component.

**Backend → Python Flask**

Flask over Django because this is a lean API server. I don't need an ORM, an admin panel, or a full web framework. Flask gives me routes and nothing more — perfect for wrapping AI model calls and data processing logic into clean endpoints.

**AI → Google Gemini 1.5 Flash**

I evaluated multiple models. Gemini 1.5 Flash is free-tier, fast, multilingual (important for Indian users), and handles long context well — crucial when the AI Agent passes research results back alongside the user query. The system prompt engineering for the Financial Path module (which returns structured JSON for graph rendering) worked cleanly with Gemini.

**Auth → Clerk**

Building auth from scratch (JWT, refresh tokens, Google OAuth, password reset flows) is a week of work minimum. Clerk handles all of it with two lines of React code. More importantly, it stores all user data securely in their cloud with GDPR compliance — I don't have to worry about password storage, session management, or breaches.

---

## 🔨 Building the Features — One at a Time

### 1. The AI Chatbot

The first thing I built was the chatbot because it was the heart of the idea. The challenge here was that a plain Gemini API call would give generic answers — it needed *real data*.

I built a **ReAct Agent** (Reasoning + Acting loop) using LangChain. The agent can:
- Search the web in real-time
- Fetch current and historical stock prices
- Get company fundamentals
- Run Python calculations
- Check system time for market hours

The architecture: when a user asks something like *"Should I invest in Cipla right now?"*, the agent doesn't just answer. It **acts** — searches for recent news, fetches the current price, checks historical trends, then reasons over all of it before responding.

The output is streamed through a `subprocess.Popen` call in Flask, which reads the agent's stdout line by line. The final answer is extracted using a `<Response>...</Response>` tag pattern. If that tag isn't found (agent timed out or got confused), it falls back to a direct Gemini call with the partial research as context.

**Challenge:** Getting the agent to produce consistently tagged output was tricky. The ReAct loop sometimes ended without a clean final answer. The fallback to `jgaad_chat_with_gemini()` with the partial research was the solution — the secondary Gemini call sees everything the agent thought about and produces a clean answer from it.

---

### 2. Financial Path — Visual Investment Planning

This was the most technically interesting feature to build. The goal: user types or speaks a financial goal ("I have ₹10 lakh, I want to retire in 20 years"), and the app generates a **visual flow graph** showing exactly how to allocate the money.

On the backend, `gemini_fin_path.py` sends the query to Gemini with a precise system prompt that forces the output to be a JSON structure matching the ReactFlow node/edge format — with positions, labels, colors, and allocation percentages. The model returns a graph blueprint, not prose.

On the frontend, `FinancialPathFlow.tsx` uses `@xyflow/react` to render the blueprint as a draggable, interactive graph. Each node is a colored box representing an asset class (Index Funds, Gold, Mid-Caps, FD, etc.) with the rupee allocation. Edges show percentage flows between them.

I also added **voice input** using the Web Speech API — users can speak their financial situation instead of typing it. The transcript feeds directly into the path generator.

**Challenge:** Gemini occasionally wrapped the JSON in markdown code blocks (` ```json ``` `). The solution was a regex extraction: `re.search(r'```json\s*(.*?)\s*```', response, re.DOTALL)` with a fallback to direct JSON parse if no wrapper is present.

---

### 3. Stock Analyzer

The original plan was to embed a Streamlit-based stock analysis app. I built the integration, tested it — and discovered the Streamlit app required users to authenticate with Streamlit's own login before loading. In an embedded iframe, this was completely broken. Users saw a login wall inside the widget.

**Solution:** Replaced with **TradingView's Advanced Chart widget** — a production-grade, embeddable, no-auth-required charting library used by professional traders. It loads NSE:NIFTY by default (Indian market context), supports searching any NSE/BSE symbol, shows candlestick charts, volume, technical indicators (RSI, MACD, Bollinger Bands), and adapts to dark/light theme automatically by reading the app's theme context.

The switch took the Stock Analyzer from broken to one of the strongest features in the app.

---

### 4. Portfolio Dashboard

The dashboard shows a full financial picture: net worth, asset allocation pie chart, monthly performance bar chart, investment goals tracker, risk metrics (Sharpe Ratio, Beta, Alpha, Max Drawdown), liabilities overview, and recent activity feed — all animated with Framer Motion.

Charts are built on Recharts — chosen over Chart.js because it's React-native (component-based, not canvas-based), works naturally with TypeScript, and handles responsive containers with a single wrapper.

---

### 5. MoneyPulse — Real-Time Financial News

Finance decisions should never happen in an information vacuum. MoneyPulse pulls live Indian financial news via the **GNews API** — filtered by categories (Markets, Economy, Corporate, Policy, Stocks, Cryptocurrency) with a search bar. Every article shows source, publish time, thumbnail, and a direct link.

The API key is stored in `.env` as `VITE_GNEWS_API_KEY` — never hardcoded, never exposed in commits.

---

### 6. Money Calculator

The MoneyCalc module lets users project their financial future. Users add assets (name, current value, expected annual return, quantity), set a projection horizon, and get a line chart showing wealth growth over time with compound interest calculated correctly in INR.

---

### 7. Learning Center (Money Matters)

I didn't want to build a fake course platform. The Learning Center surfaces real, vetted YouTube videos (categorized into Beginner / Intermediate / Advanced) alongside structured course outlines covering Introduction to Investing, Technical Analysis, Portfolio Management, Risk Management, and AI in Trading. Videos play inline with an embedded player. The content is genuine — each video link points to a real, relevant finance resource.

---

### 8. MyData — Know Yourself Before You Invest

Before any AI can give personalized advice, it needs to know the user. MyData has six tabs: Goals, Risk Tolerance, Income, Expenses, Assets, and Liabilities. Users fill in their actual financial situation. This data feeds the recommendation engine and AI chatbot context.

---

### 9. Profile & Security

Profile is wired directly to Clerk's user management API. Users can update their first/last name, and the changes reflect in real-time via `user.update()`. Sign-out uses Clerk's `useClerk().signOut()`. Toast notifications (via Sonner) confirm success or error for every action.

---

## 🪲 The Real Challenges — And How I Solved Them

### Challenge 1 — Python venv Path on Windows PowerShell
Running `.\venv\Scripts\pip` in PowerShell from a different working directory caused a "not recognized" error. Windows path resolution doesn't inherit the correct scope when chaining `cd` and `.\` in a single command.
**Fix:** Used absolute paths (`& "C:\...\venv\Scripts\python.exe"`) and the PowerShell call operator `&` to invoke executables correctly.

### Challenge 2 — Pip Dependency Backtracking (45+ minutes)
Installing `langchain_community` + `google-generativeai` + `grpcio` together triggered pip's backtracking resolver — it tried hundreds of version combinations for `googleapis-common-protos` and `grpcio-status` before finding a compatible set.
**Fix:** Ran the install as a background process and waited. No pinning needed — pip eventually resolved it. Added a note to pin versions for future `requirements.txt` updates.

### Challenge 3 — Clerk Not Redirecting After Login
After sign-in, users stayed on the auth page instead of going to `/portfolio`. Three bugs were stacked:
- `ClerkProvider` was **outside** `<Router>`, so Clerk had no access to React Router's navigate function
- `redirectUrl` and `afterSignInUrl` props were **deprecated** in newer Clerk versions
- `/sign-in/sso-callback` route was **missing** (only sign-up SSO was handled, breaking Google login)

**Fix:** Moved `ClerkProvider` inside the router, passed `navigate={(to) => navigate(to)}` to give Clerk routing access, replaced props with `forceRedirectUrl`/`fallbackRedirectUrl`, and added the missing SSO callback route.

### Challenge 4 — Gemini API Deprecation Warning
`google.generativeai` package showed a `FutureWarning` on every startup: *"All support has ended"*. The app still works, but it's a flag.
**Status:** Noted. The migration path is to switch to `google.genai` (the new SDK). Not blocking for now.

### Challenge 5 — Financial Path JSON Parsing Failures
Gemini sometimes returned the JSON wrapped in markdown code fences, sometimes bare, and occasionally with trailing text. A plain `json.loads(response.text)` would crash on the wrapped versions.
**Fix:** A two-stage parser — regex extracts content between ` ```json ``` ` tags first; if that fails, tries direct JSON parse of the entire response.

---

## 🔐 Security Decisions

- All API keys (Gemini, GNews, Clerk) live in `.env` files — never hardcoded
- Clerk handles all password storage, sessions, and OAuth — no plaintext credentials anywhere
- Every dashboard route is wrapped in `<ProtectedRoute>` — unauthenticated users are redirected to sign-in instantly
- CORS is enabled on Flask with `Flask-Cors` — locked to the frontend origin in production

---

## 📐 Architecture Overview

```
User Browser
    │
    ├── React Frontend (Vite, port 5173)
    │       ├── Clerk Auth (cloud)
    │       ├── TradingView Widget (CDN)
    │       ├── GNews API (direct)
    │       └── Flask Backend (port 5000)
    │               ├── /agent          → ReAct Agent (LangChain + Groq/Gemini)
    │               ├── /ai-financial-path → Gemini 1.5 Flash (structured JSON)
    │               ├── /auto-bank-data → Static bank data
    │               └── /auto-mf-data   → Static mutual fund data
    │
    └── Clerk Dashboard (user management)
```

---

## 🚀 What DhanPath Solves — In Plain English

| Problem | How DhanPath Solves It |
|---------|------------------------|
| "I don't know where to invest" | AI Agent researches live market data and gives a personalized answer |
| "I can't visualize my financial plan" | Financial Path generates an interactive investment flow map |
| "Stock charts are on another site" | TradingView chart is embedded — search any NSE/BSE stock |
| "I forget to track my expenses" | MyData tabs for income, expenses, assets, liabilities |
| "Finance news is scattered" | MoneyPulse aggregates Indian financial news by category |
| "I don't know how my money will grow" | Money Calculator projects wealth with compound returns |
| "I want to learn but don't know where to start" | Learning Center with curated videos + courses by level |
| "I don't trust apps with my login data" | Clerk-secured auth — enterprise-grade, not a custom DB |

---

## 🛠️ Running the Project

### Backend
```powershell
cd backend
.\venv\Scripts\Activate.ps1
python app.py
# Runs on http://localhost:5000
```

### Frontend
```powershell
cd frontend
npm run dev
# Runs on http://localhost:5173
```

### Environment Variables

**backend/.env**
```
GEMINI_API_KEY=your_gemini_key
```

**frontend/.env**
```
VITE_CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key
VITE_CLERK_SECRET_KEY=your_clerk_secret_key
VITE_GNEWS_API_KEY=your_gnews_key
```

---

## 📦 Tech Stack Summary

| Layer | Technology | Why |
|-------|-----------|-----|
| Frontend Framework | React 18 + TypeScript | Component model, type safety |
| Build Tool | Vite 7 | Fast HMR, optimized builds |
| Styling | Tailwind CSS | Utility-first, dark mode built-in |
| Charts | Recharts | React-native, TypeScript-friendly |
| Animations | Framer Motion | Smooth, declarative animations |
| Flow Graphs | @xyflow/react | Interactive node-edge diagrams |
| Auth | Clerk | Enterprise auth, zero backend needed |
| Backend | Python Flask | Lightweight API server |
| AI Model | Google Gemini 1.5 Flash | Fast, free-tier, structured output |
| Agent Framework | LangChain ReAct | Tool-using reasoning agent |
| News API | GNews | Indian finance news, filterable |
| Market Charts | TradingView Widget | Professional charts, no auth required |
| Notifications | Sonner | Toast notifications |
| HTTP Client | Axios | Promise-based, interceptors |

---

*Built with curiosity, a lot of debugging, and the belief that financial clarity should be accessible to everyone — not just those who can afford a personal advisor.*

