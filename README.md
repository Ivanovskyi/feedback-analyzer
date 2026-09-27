# Customer Feedback Analyzer (Gen AI)

**The problem:** A restaurant owner has dozens of customer reviews and no time to read them all. They want a simple tool: paste the reviews, click a button, and instantly see how customers feel and what they keep complaining about.


---

## Step 1: Set up the project

```bash
uv init feedback_analyzer
cd feedback_analyzer
uv add "fastapi[standard]" google-genai python-dotenv pydantic streamlit requests
```

Put your Gemini key in a `.env` file (see `sample.env`):

```
GOOGLE_API_KEY=your_key_here
```

---

## Step 2: The backend (`api.py`)

The full code is in `api.py`. The important part is the answer shape:

```python
class Analysis(BaseModel):
    label: str   # "positive", "negative", or "neutral"
    score: int   # 1 (very bad) to 5 (very good)
    theme: str   # one word, e.g. "delivery"
```

Run the backend in its **own terminal** and leave it running:

```bash
uv run fastapi dev api.py
```

Quick test: open **http://127.0.0.1:8000/docs**, try `/analyze` with a review, and confirm you get back a label, score, and theme.

---

## Step 3: The frontend (`app.py`)

Open a **second terminal** (keep the backend running in the first one) and run:

```bash
uv run streamlit run app.py
```

Paste a few reviews (one per line) and click **Analyze**. You can copy test reviews from `sample_reviews.txt`.

The key idea in the frontend is this loop. For each review, we call our own backend and collect the answer:

```python
for review in reviews:
    try:
        response = requests.post(API_URL, json={"text": review})
        data = response.json()
        results.append({
            "review": review,
            "label": data["label"],
            "score": data["score"],
            "theme": data["theme"],
        })
    except Exception:
        # one bad review should not stop the whole batch
        results.append({"review": review, "label": "error", "score": 0, "theme": "error"})
```

---

## How to run the whole thing (two terminals)

| Terminal 1 (backend) | Terminal 2 (frontend) |
|---|---|
| `uv run fastapi dev api.py` | `uv run streamlit run app.py` |
| stays running | opens in your browser |

If the frontend shows "error" rows, the most common reason is that the backend is not running. Start it first.
