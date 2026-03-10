⚙️ Setup Instructions
-------
📌 1. Prerequisites
-------
  Before running the project, ensure the following are installed on your machine:
  
    • Python 3.10+
    
    • Node.js 16+ or later
    
    • npm (bundled with Node.js)
    
    • Tavily API Key (for web research)
    
    • Google Gemini API Key (for AI generation)
  

🖥️ 2. Backend Setup (FastAPI)
-------
  Step 1 — Navigate to the backend folder
  
    cd company-research-assistant
  
  Step 2 — Install Python dependencies
  
    pip install -r requirements.txt
  
  Step 3 — Set environment variables
  
    Windows PowerShell
    
    setx TAVILY_API_KEY "your_tavily_key"
    
    setx GEMINI_API_KEY "your_gemini_key"
  
  
  Step 4 — Start the backend server
  
    python server.py


  Backend will start at:
  
    • http://localhost:8000

3. Frontend Setup (React + Vite + Tailwind)
-------
  Step 1 — Navigate to the frontend
  
    cd frontend
  
  Step 2 — Install dependencies
  
    npm install
  
  Step 3 — Start the development server
  
    npm run dev
  
  
  Frontend default URL:
  
    • http://localhost:5173

4. Connecting Frontend & Backend
-------

  | Functionality                | HTTP Endpoint          |
  | ---------------------------- | ---------------------- |
  | Run company research         | `POST /api/research`   |
  | Edit specific section        | `POST /api/edit`       |
  | Ask questions about the plan | `POST /api/chat`       |
  | Export professional PDF      | `POST /api/export_pdf` |

📁 5. Project Structure
-------
    company-research-assistant/
    │── agent/
    │   ├── research_agent.py        # Search + synthesis agent
    │   ├── chat_agent.py            # Q/A agent
    │   ├── plan_editor.py           # Section editing agent
    │   └── voice.py                 # Transcription utilities
    │
    │── static/
    │   └── logo.png                 # Optional PDF logo
    │
    │── server.py                    # FastAPI backend
    │── requirements.txt
    │
    frontend/
    │── src/
    │   ├── App.jsx                  # Main UI logic
    │   ├── main.jsx                 # Entry point
    │   └── index.css                # Custom styles
    │
    │── package.json



🏗️ Architecture Notes
-------
The Company Research Assistant is designed using a layered, modular architecture optimized for agentic workflows and clean separation of responsibilities. Each component — research, synthesis, editing, and presentation — is isolated, making the system easy to maintain, debug, and extend.

The architecture is centered around three core technologies:

    •LangChain – orchestration and agent logic
    
    •Tavily – real-time web research
    
    •Google Gemini – large language model for reasoning, synthesis, and editing


🔶 1. Architectural Overview
-------
          Frontend (React) 
             ↓
          FastAPI Backend
             ↓
          Agent Layer (LangChain)
          ⎧ ResearchAgent  → Tavily Search → Gemini Synthesis
          ⎪ PlanEditor     → Gemini Editing
          ⎩ ChatAgent      → Gemini Q/A


Each layer has a distinct specialization:

    •Frontend handles UI and user interactions
    •Backend exposes REST APIs
    •Agents Layer performs all intelligent reasoning
    •External Services (Tavily + Gemini) enable research & AI output

🔍 2. LangChain-Centered Agent Architecture
-------
  LangChain is used as the brain of every workflow.
    It handles:
    
    •Tool invocation
    •Prompt templates
    •Input/output normalization
    •Chaining of multiple LLM calls
    •Data routing between agents

Why LangChain?
✔ Tooling Support

  LangChain allows each capability (Tavily search, Gemini generation, editing, Q&A) to be wrapped as isolated tools that can be combined or extended easily.

✔ Consistent Prompting

  Prompts are strictly controlled and standardized across all agents.

✔ Encapsulation

  Each agent's logic lives in its own class:

  •ResearchAgent
  
  •PlanEditor
  
  •ChatAgent

This separation reduces prompt leakage, hallucinations, and incorrect cross-use of context.

✔ Extensibility

  If future features are needed (e.g., CRM integration, document ingestion), they can be added directly as new tools without modifying existing logic.

🌐 3. Tavily Search Integration (Automated Company Research)
-------
    Tavily serves as the primary research engine for real-time company insights.
  
  🔹 How it Works:
  
   1. User enters a query (e.g., "Research Tesla opportunities").
    
   2.ResearchAgent uses LangChain’s Tavily tool to run:
  
      tavily.search(query, max_results=5)
  
  
   3.Tavily returns:
  
      •summaries
      •raw texts
      •links
      •structured metadata
  
   4.LangChain aggregates the multi-source research output.
  
  🔹 Why Tavily?
  
      Purpose-built for AI agents
      Clean JSON output (LLMs understand it easily)
      Summaries + sources included
      No scraping required
      Fast, reliable, and high-quality web intelligence
      Tavily reduces hallucinations by grounding LLM responses in real data.

🤖 4. Gemini LLM Integration (Synthesis, Editing, & Q/A)
-------
  The system uses Google Gemini across all reasoning tasks because of its:
  
      •High-quality summarization
      •Strong structured output handling
      •Fast inference
      •Excellent transformation abilities
  
  Gemini is used in three primary modes:

  4.1 Synthesis Mode → Generate Account Plan
    
  After Tavily collects data, LangChain sends a structured prompt to Gemini to produce a multi-section plan:
    
    •Company Overview
    •Key Findings
    •Pain Points
    •Opportunities
    •Competitors
    •Recommended Strategy
    •Confidence Estimate
    
  The plan is always generated as a structured JSON object, ensuring consistency and easy rendering in the frontend.
    
   4.2 Editing Mode 
    
  The PlanEditor agent uses Gemini for precise edits.
    
  ✨ Constraints Applied:
    
    No content drift
    No rewriting of untouched sections
    Format consistency guaranteed
    Plan remains properly structured
    
  This creates a reliable “LLM-in-the-loop editor” experience.
    
  4.3 Q/A Mode → Follow-Up Questions
    
  The ChatAgent uses Gemini in retrieval-free mode:
    
    Only the generated plan is used as context
    No external search is called
    Ensures deterministic and grounded answers

This prevents hallucinations and keeps conversation tightly scoped to the account plan.

🧠 5. Multi-Agent Design
-------
  | Agent Name        | Responsibility                         | Powered By                  |
  | ----------------- | -------------------------------------- | --------------------------- |
  | **ResearchAgent** | Search + summarization + LLM synthesis | LangChain + Tavily + Gemini |
  | **PlanEditor**    | Update only one plan section at a time | LangChain + Gemini          |
  | **ChatAgent**     | Answer user questions about the plan   | LangChain + Gemini          |
  | **Voice Module**  | (Optional) speech recognition          | Gemini (Transcription)      |


📄 6. PDF Generation Architecture
-------
  •The backend uses ReportLab in single-column layout i.e stable formatting.
  
  •The PDF pipeline:

    Plan JSON → Formatting Engine → ReportLab → Downloadable PDF

•  Automatic paragraph splitting + line wrapping.
  
• Sections ordered consistently.

This architecture ensures consistent, professional PDF output regardless of text length.

🖼️ 7. Frontend Architecture (React + Vite + Tailwind)
-------
  The frontend is a clean, component-based architecture:

  UI Components:

    •Research Chat Panel
    •Narration Controls
    •Plan Viewer (two-column responsive)
    •Actions & Q/A
    •Session Panel
    •Header + Footer

Key UX Decisions:

    •Dark mode support
    •Smooth layout using Tailwind Grid
    •Clear separation between research, editing, Q/A
    •TTS narration using browser Speech API

Each component communicates with backend via dedicated endpoints.

🔁 8. End-to-End Flow
-------
    User Query
       ↓
    Frontend → /api/research
       ↓
    ResearchAgent
       → Tavily Search
       → Summaries
       → Gemini Synthesis
       ↓
    Backend returns JSON plan
       ↓
    Frontend displays plan
    
    User Edits
       ↓
    Frontend → /api/edit
       ↓
    PlanEditor → Gemini Editing
       ↓
    Updated plan
    
    User Asks Question
       ↓
    Frontend → /api/chat
       ↓
    ChatAgent → Gemini Q/A
       ↓
    Answer
    
    User Exports PDF
       ↓
    Frontend → /api/export_pdf
       ↓
    ReportLab renders professional PDF

🎨 Design Decisions (UI)
-------
  •Card-based layout for a clean, professional dashboard feel.
  
  •Two-column responsive design (chat on left, plan on right) for clarity and easy navigation.
  
  •Dark mode support with Tailwind for modern, comfortable viewing.
  
  •Dedicated narration panel placed centrally for visibility and quick access.
  
  •Clear typographic hierarchy (titles, subtitles, body) to improve readability of long plans.
  
  •Inline section editing using modals to keep the interface minimal and focused.
  
  •Separated Q/A panel to prevent confusion between insights and the main account plan.
  
  •Prominent PDF export button placed for quick access during demos.
