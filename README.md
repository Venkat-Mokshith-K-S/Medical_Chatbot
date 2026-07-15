# Meddy — Medical Chatbot

A Flask app that uses a small PyTorch intent-classification model plus a
scikit-learn model to guess a likely condition from symptoms you describe in
plain English.

## Project structure

```
medical-chatbot/
├── app.py              # Flask app + inference logic
├── nnet.py             # PyTorch model definition
├── nltk_utils.py        # tokenization / bag-of-words helper
├── requirements.txt
├── Procfile             # tells the host how to start the app
├── runtime.txt           # pinned Python version
├── models/               # trained model weights (.pth, .pickle)
├── data/                 # symptom/disease reference CSVs + pickles
├── templates/            # index.html
└── static/               # css/js/images
```

## Run locally

```bash
python -m venv venv
# Windows: venv\Scripts\activate
# Linux/Mac: source venv/bin/activate

pip install -r requirements.txt
python app.py
```

Visit `http://127.0.0.1:5000`.

## Deploy

This app loads PyTorch and scikit-learn models into memory, which makes it a
poor fit for pure serverless platforms like Vercel (250MB function size
limit, no persistent memory between invocations). It deploys cleanly as-is on
any platform that runs a persistent container, e.g. **Render**, **Railway**,
or **Fly.io**.

### Render (recommended, has a free tier)

1. Push this repo to GitHub.
2. In the Render dashboard: **New → Web Service** → connect the repo.
3. Render auto-detects `requirements.txt` and `Procfile` — no extra config
   needed. Just confirm:
   - **Build Command:** `pip install -r requirements.txt`
   - **Start Command:** `gunicorn app:app --workers 1 --threads 4 --timeout 120`
4. Deploy. First boot is slower (~1-2 min) since it has to download the
   PyTorch CPU wheel.

### Railway

1. Push to GitHub, then **New Project → Deploy from GitHub repo**.
2. Railway reads the `Procfile` automatically. No further config needed.

## Known limitations

- `user_symptoms` is stored as an in-memory global `set()`, so all visitors
  currently share the same session state. Fine for a personal demo; if this
  ever gets real traffic, move that into a per-session store (Flask
  `session`, or a small Redis instance) so users don't see each other's
  symptoms.
- The model files in `models/` are loaded once at process startup, so a
  gunicorn restart or platform cold start takes a few seconds to warm up.
