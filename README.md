## Company Brochure Generator

An end‑to‑end AI pipeline that turns any company website into a concise, human‑readable brochure for prospective customers, investors, and candidates – using web scraping plus a Large Language Model (LLM) via OpenRouter.

The project starts from a single company URL, automatically discovers the most relevant pages (About, Strategy, Business Model, Careers, Sustainability, Investors, etc.), pulls the key content, and asks an LLM to synthesize it into a well‑structured brochure in markdown.

---

## Features

- **Automated link discovery**  
  Extracts all outgoing links from a company’s landing page using `BeautifulSoup`, providing raw navigation data for the LLM to reason about.

- **LLM‑driven relevance filtering**  
  Uses an LLM to classify which links are brochure‑relevant (About, Strategy, Business Model, People, Careers, Sustainability, Investors, Locations, etc.) and returns them as structured JSON.

- **Content aggregation pipeline**  
  Scrapes and cleans text for the landing page plus all selected sub‑pages, removing scripts, styles, images, and noisy HTML, then aggregates them into a single context.

- **Brochure generation**  
  Feeds the aggregated content into an LLM that writes a short, focused brochure in markdown, suitable for customers, investors, and job seekers.

- **Configurable tone and prompts**  
  Uses clear system/user prompts, with an optional humorous brochure mode to show how easily tone and style can be changed by prompt engineering.

- **Practical safeguards**  
  Truncates long prompts to keep them within a safe size and uses realistic user‑agent headers to fetch real‑world websites reliably.

---

## Tech Stack

- **Language**: Python  
- **Environment**: Jupyter Notebook (`brochure.ipynb`)  
- **Web scraping**: `requests`, `beautifulsoup4`  
- **LLM access**: `openai` client configured to use OpenRouter (`https://openrouter.ai/api/v1`)  
- **Configuration**: `python-dotenv` for `.env` management (`OPENROUTER_API_KEY`)  
- **Display**: `IPython.display.Markdown` to render the final brochure directly in the notebook

---

## Project Structure

- `brochure.ipynb` – main notebook implementing the full workflow (scraping, link selection, aggregation, brochure generation).  
- `scraper.py` – helper module with functions to fetch raw HTML, extract links, and clean page contents.  
- `.env` – environment file holding the `OPENROUTER_API_KEY` (not committed).  
- `pyproject.toml` / `uv.lock` – dependency and environment configuration (if using `uv`/PEP 621 tooling).

---

## Methodology & Workflow

### 1. Web Scraping & Preprocessing

- `fetch_website_links(url)` (in `scraper.py`)  
  - Fetches the HTML of the given URL using `requests` with a standard browser user‑agent.  
  - Parses the page with `BeautifulSoup` and extracts all `<a>` tag `href` attributes.

- `fetch_website_contents(url)` (in `scraper.py`)  
  - Fetches the page HTML and parses it with `BeautifulSoup`.  
  - Removes non‑content elements (`<script>`, `<style>`, images, inputs).  
  - Returns a clean string containing the page title and visible text, truncated to ~2,000 characters.

### 2. LLM‑Based Relevant Link Selection

- `link_system_prompt`  
  - A system prompt that instructs the model to look at the raw list of links and pick only those that are brochure‑relevant.  
  - The model is asked to respond with a JSON object of the form:
    - `{ "links": [ { "type": "...", "url": "https://..." }, ... ] }`

- `select_relevant_links(url)`  
  - Calls `openrouter.chat.completions.create` (model: `nvidia/nemotron-3-nano-30b-a3b:free`).  
  - Provides the system prompt plus a user prompt containing all discovered links for the given URL.  
  - Requests `response_format={"type": "json_object"}` so the output is machine‑readable.  
  - Parses the JSON back into Python and logs how many links were selected.

This step uses the LLM as an intelligent filter over the navigation graph of the website.

### 3. Content Aggregation

- `fetch_page_and_all_relevant_links(url)`  
  - Scrapes and cleans the landing page content.  
  - Calls `select_relevant_links(url)` to get curated sub‑pages (About, Strategy, Careers, etc.).  
  - Iterates through each selected link, scrapes its content, and appends it.  
  - Builds a single text block with sections such as:
    - `## Landing Page: ...`  
    - `### Link: <type> ...`

This forms the “retrieval” step in a retrieval‑augmented generation (RAG) pattern, gathering all the raw material the LLM needs.

### 4. Brochure Generation via LLM

- `brochure_system_prompt`  
  - Positions the model as a brochure writer for multiple audiences: customers, investors, and recruits.  
  - Asks it to emphasize company culture, customers, and career opportunities when available.  
  - Requires the response to be in plain markdown (no code blocks).

- `get_brochure_user_prompt(company_name, url)`  
  - Constructs a user prompt that introduces the company name and embeds the aggregated page content.  
  - Truncates the text to 5,000 characters to keep the prompt efficient and robust.

- `create_brochure(company_name, url)`  
  - Calls the LLM via OpenRouter with the system and user prompts.  
  - Extracts the generated markdown brochure.  
  - Renders it inline in the notebook using `display(Markdown(...))`.

---

## Why This Project Matters

- **Real‑world applicability**  
  Many organisations need consistent, concise messaging for customers, investors, and candidates. This project shows how to automatically generate such messaging from a company’s own live website, making it easier to keep brochures and one‑pagers up to date.

- **Demonstrates modern AI patterns**  
  The project combines:  
  - Traditional web scraping and HTML cleaning.  
  - Structuring LLM outputs in JSON for downstream use.  
  - Retrieval‑augmented generation of marketing copy from real content.

- **Extensible foundation**  
  The code is deliberately simple and modular, making it straightforward to:  
  - Swap in different models or providers.  
  - Wrap the notebook in a web UI or API.  
  - Localise brochures into multiple languages.  
  - Tune the prompts for different tones and audiences.

---

## Getting Started

### Prerequisites

- Python 3.10+  
- An OpenRouter API key with access to at least one compatible chat model  
- Jupyter Notebook / JupyterLab / VS Code with notebook support

### 1. Clone the Repository

```bash
git clone <your-repo-url>.git
cd Company_Brochure_Generator
```

### 2. Create and Activate a Virtual Environment

Example with `venv` on Windows:

```bash
python -m venv .venv
.\.venv\Scripts\activate
```

(On macOS/Linux, use `source .venv/bin/activate`.)

### 3. Install Dependencies

If you use `pyproject.toml` with `uv` or another tool, follow that workflow.  
Otherwise, install the core libraries manually:

```bash
pip install requests beautifulsoup4 python-dotenv openai ipython
```

### 4. Configure Environment Variables

Create a `.env` file in the project root:

```bash
OPENROUTER_API_KEY=or_************************
```

Make sure `.env` is **not** committed to version control.

### 5. Run the Notebook

1. Open `brochure.ipynb` in Jupyter or VS Code.  
2. Ensure your virtual environment is selected as the notebook kernel.  
3. Run all cells in order.  
4. Call a function such as:
   - `create_brochure("Standard Chartered", "https://www.sc.com/en/")`

The generated brochure will appear rendered as markdown inside the notebook.

---

## Example Use Case

For a company such as **Standard Chartered**:

1. Provide the base URL: `https://www.sc.com/en/`.  
2. The notebook fetches all links from the landing page.  
3. The LLM filters down to relevant links like About, Strategy, Business Model, Sustainability, Investors, and Careers.  
4. The scraper pulls content from those pages.  
5. The LLM writes a concise brochure describing:
   - What the company does.  
   - Who its main customers are.  
   - Its strategic positioning and business model.  
   - Its culture, purpose, and career opportunities.

---

## Possible Extensions

- Multi‑language brochures (e.g., generate English + local language versions).  
- A small web front‑end where users can input a URL and download a brochure.  
- PDF export or automated integration into slide decks.  
- Deeper crawling (e.g., following internal links to a defined depth or using sitemaps).  
- Model comparison, prompt tuning, or evaluation for brochure quality.

---

## Disclaimer

Please respect each website’s **robots.txt**, terms of use, and any applicable legal and ethical constraints when scraping and generating content. This project is intended for educational and prototyping purposes and should be used responsibly.

