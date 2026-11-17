# AskHAW - AI Chatbot for HAW Hamburg

**Live demo: [Open AskHAW](PASTE-YOUR-LIVE-LINK-HERE)**

Click the link above to try the chatbot directly in your browser. Ask questions in English or German. (You will first be shown a short privacy notice to accept.)

## Team

AskHAW was developed as a team project at the Hamburg University of Applied Sciences (HAW Hamburg).

| Role | Name |
|---|---|
| Team member | Sajjadur Rahman ([GitHub](https://github.com/sajjadurrahman1)) |
| Team member | Mainuddin Monsur Robin |
| Team member | Monzurul Islam Zhimon |
| Supervising professor |Pawel Buczek |

---

AskHAW is a bilingual (English / German) chatbot for students and applicants of the Hamburg University of Applied Sciences (HAW Hamburg). It answers questions about programs, applications, deadlines, scholarships, housing, library, counselling, campus life and contacts, using only information scraped from official HAW Hamburg web pages.

It is built as a Retrieval-Augmented Generation (RAG) pipeline: relevant text chunks are retrieved from a local vector index and passed to an OpenAI model, which is instructed not to state anything that is not in the retrieved context.

Note: AskHAW is a prototype and not an official HAW Hamburg service.

---

## Features

- Bilingual: detects whether a question is in English or German and searches the matching knowledge base.
- Grounded answers: the model is told to answer only from the retrieved context and to say so when information is missing, instead of guessing.
- Streaming responses in a Streamlit chat interface with HAW branding.
- Conversation memory within a session, so follow-up questions work.
- Suggested starter questions on the welcome screen.
- GDPR privacy notice that must be accepted before chatting.
- Thumbs up / thumbs down feedback on every answer, saved to `data/feedback.csv`.
- Fallback message with real contact details (Student Counselling, Admissions Office) when nothing relevant is found.

---

## How It Works

```
data/HAW_Hamburg_Online_Services.xlsx     list of official HAW URLs (Label, URL, Category)
                |
                v
scripts/build_kb.py                       scrape pages, extract e-mail/phone links,
                                          split into ~500-word chunks, translate to EN and DE
                |
                v
data/haw_kb.json, haw_kb_en.json, haw_kb_de.json
                |
                v
scripts/embed.py                          OpenAI embeddings -> FAISS indexes
                |
                v
data/haw_kb_en.index, haw_kb_de.index + haw_kb_meta_en.json, haw_kb_meta_de.json
                |
                v
main.py (Streamlit app)
  1. detect the language of the question (langid)
  2. embed the question with text-embedding-3-small
  3. retrieve the 5 nearest chunks from the matching FAISS index
  4. if the best match is too far away (L2 distance above 3.5), show the fallback message
  5. otherwise send the chunks plus strict answering rules and the chat history to gpt-4o-mini
  6. stream the answer to the user
```

Models used:

| Purpose | Model |
|---|---|
| Embeddings | `text-embedding-3-small` (1536 dimensions) |
| Chat completion | `gpt-4o-mini` |

Current knowledge base: about 85 successfully scraped HAW pages, split into 199 text chunks per language. The source list covers 16 categories, including faculties and programs, department pages, contact pages, scholarships and finance, housing, library, counselling, international students and campus facilities.

---

## Tech Stack

- Python 3.9 or newer
- Streamlit (web interface)
- OpenAI API (embeddings and chat)
- FAISS (vector similarity search)
- langid (language detection)
- requests and BeautifulSoup (web scraping)
- pandas and openpyxl (reading the Excel URL list)
- deep-translator (creating the English and German knowledge bases)

---

## Project Structure

```
AskHAW/AskHAW/
|-- main.py                               Streamlit chat application
|-- requirements.txt                      Python dependencies
|-- assets/                               Logos and favicon
|-- data/
|   |-- HAW_Hamburg_Online_Services.xlsx  Source list of HAW URLs
|   |-- haw_kb.json                       Combined knowledge base
|   |-- haw_kb_en.json, haw_kb_de.json    Per-language knowledge bases
|   |-- haw_kb_en.index, haw_kb_de.index  FAISS vector indexes
|   |-- haw_kb_meta_en.json, haw_kb_meta_de.json   Chunk texts used for retrieval
|   `-- feedback.csv                      Collected user feedback
|-- scripts/
|   |-- build_kb.py                       Scrape, chunk and translate pages
|   `-- embed.py                          Build FAISS indexes
|-- docs/                                 SRS report, presentation, sample output
`-- archive/                              Earlier versions of the project
```

---

## Getting Started

### Prerequisites

- Python 3.9 or newer
- An OpenAI API key: https://platform.openai.com/account/api-keys

### 1. Clone the repository

```bash
git clone https://github.com/sajjadurrahman1/Ask-Haw.git
cd Ask-Haw/AskHAW/AskHAW
```

### 2. Create a virtual environment

```bash
python -m venv .venv
source .venv/bin/activate        # macOS / Linux
.venv\Scripts\activate           # Windows
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
pip install openai faiss-cpu numpy
```

`main.py` also imports `openai`, `faiss` and `numpy`, which are not listed in `requirements.txt`, so they are installed in the second command.

### 4. Add your OpenAI API key

Create the file `.streamlit/secrets.toml` in the project folder:

```toml
OPENAI_API_KEY = "sk-..."
```

Never commit this file to a public repository.

### 5. Run the app

The repository already contains a pre-built knowledge base and FAISS indexes, so the app can be started directly:

```bash
streamlit run main.py
```

Streamlit opens the app in your browser, by default at http://localhost:8501.

---

## Updating the Knowledge Base

1. Edit `data/HAW_Hamburg_Online_Services.xlsx` (columns: `Label`, `URL`, `Category`).
2. Rebuild the knowledge base. This scrapes and translates every page and can take several minutes:
   ```bash
   python scripts/build_kb.py
   ```
3. Rebuild the vector indexes (requires the API key in `.streamlit/secrets.toml`):
   ```bash
   python scripts/embed.py
   ```

Run both scripts from the project folder (`AskHAW/AskHAW`) so the `data/` paths resolve correctly.

---

## Usage

- Ask a question in English or German; the language is detected automatically.
- Click a suggested question on the welcome screen to get started.
- Ask follow-up questions in the same session.
- Rate answers with the thumbs up / thumbs down buttons.
- Use "Clear Chat" in the sidebar to start a new conversation, or "Privacy Notice" to review the data policy.

Example questions:

- What programs does HAW offer?
- How do I apply for a Bachelor?
- Are there scholarships?
- Wie kontaktiere ich die Studienberatung?

---

## Privacy

- Questions are sent to the OpenAI API to generate answers.
- AskHAW does not collect names, student IDs or e-mail addresses.
- Chat history exists only in the current browser session.
- Feedback entries (timestamp, rating, question, answer) are stored locally in `data/feedback.csv`.

---

## Limitations

- The bot only knows the pages listed in the Excel file. Questions outside that scope receive a fallback answer rather than a guess.
- The German and English knowledge bases are machine-translated (Google Translate via `deep-translator`) and may contain translation errors.
- Retrieval uses only the latest question, so follow-up questions that depend heavily on earlier messages may retrieve weaker results.
- Broad overview questions (for example "What programs does HAW offer?") were rated unhelpful in early feedback, which indicates that the knowledge base needs more overview pages.

---

## Possible Improvements

- Use the chat history when building the retrieval query (the function `build_retrieval_query` in `main.py` is a placeholder for this).
- Add the missing dependencies to `requirements.txt`.
- Show source links with each answer.
- Add more program overview pages to the knowledge base.
- Add automated tests and a `.gitignore` for `__pycache__/`, `.streamlit/secrets.toml` and feedback data.

---

## Documentation

The `docs/` folder contains the Software Requirements Specification (SRS) for AskHAW, a project presentation and a sample question-and-answer output.

