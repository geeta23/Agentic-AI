# Agentic-AI
An intelligent batch-processing pipeline designed to categorize, archive, and extract data from legal PDF documents (specifically Cease and Desist orders) using a multi-tiered LLM approach and OCR.

⚖️ Intelligent Legal Document Pipeline
The Problem
Processing high volumes of legal PDFs (Cease and Desist orders, Settlements, POAs) is traditionally slow and prone to human error. Digital text is often missing in scanned documents, and using high-reasoning LLMs for bulk processing often leads to "Rate Limit" bottlenecks and high costs.

Our Approach: The Three-Pillar Strategy
To build a production-ready solution, we approached the problem through three specific engineering lenses:

Robust Ingestion (Hybrid OCR): We didn't rely on simple text extraction. We implemented a "fail-fast" check that detects empty or scanned PDFs and automatically triggers a Tesseract OCR engine powered by Poppler.

Tiered Intelligence (Cost & Speed Optimization): We implemented a "Smart-to-Fast" fallback. We use a high-reasoning model (Llama 3.3 70B) for accuracy, but if the provider throttles the connection, the system instantly fails over to an "Instant" model (Llama 3.1 8B) to keep the batch moving.

Defensive Programming (Self-Healing Data): AI models can be unpredictable. We built a middleware layer to "heal" inconsistent JSON structures (like converting unexpected lists into strings) before they reach our data validation layer.

Technical Implementation
1. The Processing Core
Orchestration: Built with LiteLLM to manage cross-model communication and retry logic.

OCR Integration: Explicit path-mapping for Windows binaries (Poppler and Tesseract) to ensure the pipeline is portable across local dev environments.

2. The Decision Engine
We implemented a Confidence Gate (Threshold: 0.80):

High Confidence: Data is automatically extracted (Names, Dates, Account IDs) and saved to the DB.

Low Confidence / Uncertain: The system isolates these files into a dedicated human_review_list.csv, creating a clean "Human-in-the-Loop" workflow.

Irrelevant: Documents like Settlements are logged in an archive and skipped to save API tokens.

3. Persistence & State
State Tracking: Using a SQLAlchemy + SQLite backend, the system maintains an audit log of every filename. If the script is restarted, it remembers exactly where it left off, preventing double-billing on API usage.

Automated Exports: At the end of every run, the system regenerates a Master CSV from the SQL database to ensure the user always has the most up-to-date reference file.

How to Use This Project
Place all legal PDFs into the /data folder.

Configure your .env with a Groq API Key.

Run the batch script; the system will handle the rest, producing a clean database and a specific "To-Do" list for any documents it couldn't confidently classify.
