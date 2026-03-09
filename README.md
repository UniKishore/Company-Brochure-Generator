Company Brochure Generator
A small end‑to‑end project that turns any company website into a concise, human‑readable brochure for prospective customers, investors, and candidates – using web scraping plus an LLM via OpenRouter.

The notebook takes a company URL, automatically discovers the most relevant pages (About, Strategy, Business Model, Careers, Sustainability, etc.), pulls the key content, and asks a language model to synthesize it into a well‑structured brochure in markdown.

Features
Automated link discovery:
Uses BeautifulSoup to crawl a company’s landing page and extract all outgoing links.

LLM‑driven relevance filtering:
Calls an LLM to classify which links matter for a brochure (e.g. About, Strategy, Careers, Sustainability, Investors) and returns them as structured JSON.

Content aggregation pipeline:
Scrapes and cleans text for the landing page plus all selected sub‑pages, removing scripts, styles, images, and noisy HTML.

Brochure generation:
Feeds the aggregated content into an LLM that writes a short, focused brochure in markdown, suitable for customers, investors, and job seekers.

Configurable tone and prompts:
Uses clear system/user prompts, with an optional humorous brochure mode to show how easily tone can be changed by prompt.

Practical safeguards:
Truncates long prompts to keep them within a safe size and uses user‑agent headers to fetch real‑world websites reliably.

Tech Stack
Language: Python
Environment: Jupyter notebook (brochure.ipynb)
Web scraping: requests, BeautifulSoup
LLM access: openai client with OpenRouter (https://openrouter.ai/api/v1)
Configuration: python-dotenv (.env with OPENROUTER_API_KEY)
Display: IPython.display.Markdown to render the final brochure in‑notebook
High‑Level Architecture & Methodology
1. Web scraping & preprocessing

fetch_website_links(url) (in scraper.py) fetches the HTML, parses it with BeautifulSoup, and extracts all <a> tag hrefs.
fetch_website_contents(url) fetches the page, strips out scripts, styles, images, and inputs, and returns a clean, human‑readable text block (title + body, truncated to ~2,000 chars).
2. LLM‑based link selection

A system prompt (link_system_prompt) instructs the model to:
Look at the list of raw links for a given domain.
Identify only those that are brochure‑relevant (About, Strategy, Business Model, People, Careers, Sustainability, Investors, Locations, etc.).
Return a JSON object of { "links": [ { "type": "...", "url": "..." }, ... ] }.
select_relevant_links(url):
Calls openrouter.chat.completions.create (model: nvidia/nemotron-3-nano-30b-a3b:free).
Enforces response_format={"type": "json_object"} so the result is machine‑readable.
Logs how many relevant links were found and parses them back into Python via json.loads.
3. Content aggregation workflow

fetch_page_and_all_relevant_links(url):
Scrapes the landing page content.
Calls select_relevant_links(url) to get curated sub‑pages.
Iterates through each selected link and scrapes its content as well.
Builds a single, structured markdown‑like text block that looks like:
## Landing Page: ...
### Link: <type> ... content ...
4. Brochure generation via LLM

A system prompt (brochure_system_prompt) instructs the model to:
Act as a brochure writer.
Target multiple audiences: customers, investors, and recruits.
Emphasize company culture, customers, and careers when available.
Respond in pure markdown (no code blocks).
get_brochure_user_prompt(company_name, url):
Combines the company name with the aggregated page content.
Truncates the text to 5,000 characters for safety and efficiency.
create_brochure(company_name, url):
Calls the LLM via OpenRouter with the system + user prompt.
Renders the resulting markdown brochure directly in the notebook using display(Markdown(...)).
This pipeline is essentially a small retrieval‑augmented generation (RAG) system tailored for marketing/corporate communications: it retrieves live content from the web, filters it with an LLM, and then generates a structured summary.

Why This Project Matters
Real‑world applicability:
Companies often struggle to maintain consistent, concise messaging across audiences. This project shows how LLMs can read a company’s current website and automatically generate up‑to‑date brochures for:

Sales decks
Investor overviews
Recruitment pages and employer branding
Demonstrates modern AI patterns:
It showcases how to combine:

Traditional web scraping
Structured LLM prompting and JSON outputs
Retrieval‑augmented content generation into a clean, reproducible workflow.
Extensible foundation:
The architecture is deliberately simple and modular:

Swap in different models or providers.
Add rate limiting, caching, or persistence.
Integrate into a web UI or internal tooling.
Customize prompts for different tones or target segments.
Getting Started
Prerequisites

Python 3.10+ recommended
An OpenRouter API key with access to the chosen model
Jupyter / VS Code / any environment that supports notebooks
1. Clone the repository

git clone <your-repo-url>.git
cd Company_Brochure_Generator
2. Create and activate a virtual environment (example with venv):
python -m venv .venv
.\.venv\Scripts\activate  # Windows
# source .venv/bin/activate  # macOS/Linux
3. Install dependencies
If you are using uv / pyproject.toml, follow your preferred workflow, or simply:

pip install -r requirements.txt
(Alternatively, install: requests, beautifulsoup4, python-dotenv, openai, ipython.)

4. Configure environment variables
Create a .env file in the project root:

OPENROUTER_API_KEY=or_************************
5. Run the notebook
Open brochure.ipynb in Jupyter / VS Code.

Make sure your environment is activated ((llms) or similar in terminal).

Run all cells top‑to‑bottom.

Call, for example:

create_brochure("Standard Chartered", "https://www.sc.com/en/")
The generated brochure will render directly inside the notebook.

Typical Workflow
Step 1 – Pick a company
Decide which company you want to analyze (e.g. "Standard Chartered").

Step 2 – Provide the base URL
Use the main landing page (e.g. https://www.sc.com/en/).

Step 3 – Scrape & curate links
The notebook:

Fetches all links on the landing page.
Asks the LLM to keep only brochure‑relevant pages (About, Strategy, Careers, etc.).
Step 4 – Aggregate content
For each selected link, the project:

Scrapes and cleans page content.
Appends it into a single prompt context.
Step 5 – Generate the brochure
The LLM:

Reads the aggregated context.
Writes a structured, markdown brochure emphasizing:
What the company does
Who it serves
Its strategy and business model
Its culture, values, and career opportunities
Possible Extensions
Multi‑language brochures (e.g. generate English + local language versions).
Web app / API wrapper to expose this as a non‑technical tool.
PDF export of the generated markdown.
More robust crawling (follow internal links to a certain depth, handle sitemaps, etc.).
Model comparison (benchmark different LLMs or prompts for brochure quality).