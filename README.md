# Inbox Shield AI

AI-powered Gmail spam detection with a modern Chrome extension and a Python backend.

**Live Site / API:** https://spam-guard-ai-lac.vercel.app/

## What This Project Does

Inbox Shield AI detects suspicious Gmail messages using a multi-layer spam identification pipeline:
- Personal whitelist lookup
- Known trusted domain verification
- Spam keyword heuristics
- Machine learning classification

The Chrome extension supports manual email analysis and Gmail integration with banner alerts for opened messages.

## Key Features

- Gmail-compatible Chrome extension
- Manual analysis via popup input
- Auto-scan of opened Gmail messages
- Dual backend support: deployed Vercel API + local FastAPI server
- Multi-layer spam detection strategy
- TF-IDF + MultinomialNB machine learning model
- Customizable whitelist and trusted domain lists

## Tech Stack

- Chrome Extension: Manifest V3, popup UI, content script, service worker
- Backend: FastAPI, Uvicorn
- ML: scikit-learn, pandas, NLTK, TF-IDF
- Deployment: Vercel serverless endpoint via api/index.py

## Project Structure

```
Inbox-Shield-AI/
├── api/
│   └── index.py                 # Vercel serverless entrypoint
├── backend/
│   ├── data/
│   │   ├── spam.csv             # Training data
│   │   ├── trusted_domains.csv  # Known safe domains
│   │   └── whitelist.csv        # Personal safe email/domain data
│   ├── model/
│   │   ├── train_model.py       # Model training script
│   │   ├── spam_model.pkl       # Saved ML model
│   │   ├── vectorizer.pkl       # Saved TF-IDF vectorizer
│   │   └── label_map.pkl        # Label mapping file
│   ├── static/
│   │   └── landing.css          # Landing page styles
│   ├── templates/
│   │   └── index.html           # Landing page HTML
│   ├── app.py                   # FastAPI backend
│   ├── requirements.txt         # Backend dependencies
│   ├── README_backend.md        # Backend notes
│   └── verify_model.py          # Model verification helper
├── extension/
│   ├── assets/                  # Extension icons
│   ├── manifest.json            # Extension config
│   ├── popup.html               # Popup UI
│   ├── popup.css                # Styles
│   ├── popup.js                 # Popup logic
│   ├── content.js               # Gmail integration
│   └── background.js            # Service worker
├── requirements.txt             # Root dependency list
├── vercel.json                  # Vercel deployment config
├── LICENSE
└── README.md
```

## How It Works

The backend applies the following detection sequence in backend/app.py:

1. backend/data/whitelist.csv personal whitelist check
2. backend/data/trusted_domains.csv trusted domain check
3. Keyword-based spam/phishing indicator detection
4. ML prediction using TF-IDF features and a saved MultinomialNB model

If the model probability for spam is >= 0.5, the email is labeled as Spam.

## Installation

### Backend Setup

```bash
cd backend
python -m venv venv
.\venv\Scripts\activate     # Windows
# source venv/bin/activate  # Mac/Linux
pip install -r requirements.txt
python model/train_model.py
python app.py
```

The backend will run locally at http://127.0.0.1:8000.

### Chrome Extension Setup

1. Open Chrome and go to chrome://extensions/
2. Enable Developer mode
3. Click Load unpacked
4. Select the extension folder
5. Pin the extension to the toolbar

## Usage

### Manual Analysis

1. Click the extension icon
2. Paste email content into the popup
3. Click Analyze

### Gmail Integration

1. Open an email in Gmail
2. Click Get from Gmail in the popup
3. Click Analyze

### Auto-Scan Behavior

- extension/content.js monitors opened Gmail messages
- If the local backend is reachable, it injects a warning banner for spam detection

## Backend Endpoints

- GET / — serves the landing page
- GET /health — health check status
- POST /predict — email spam prediction

### POST /predict request format

```json
{
  "sender": "sender@example.com",
  "subject": "Email subject",
  "body": "Email body text"
}
```

### POST /predict response example

```json
{
  "label": "Spam",
  "confidence": 0.95,
  "reason": "Contains multiple severe phishing/spam indicators."
}
```

## Training the Model

The training pipeline in backend/model/train_model.py:
- loads backend/data/spam.csv
- normalizes and lemmatizes text
- vectorizes with TF-IDF
- evaluates multiple classifiers (MultinomialNB, LogisticRegression, RandomForest, SVC)
- saves the best model and vectorizer

To retrain with updated dataset:

```bash
cd backend
python model/train_model.py
```

## Customization

- Add safe senders or domains in backend/data/whitelist.csv
- Add trusted domains in backend/data/trusted_domains.csv
- Update training data in backend/data/spam.csv
- Adjust spam heuristics in backend/app.py

## Notes

- extension/popup.js attempts the deployed Vercel API first, then falls back to local http://localhost:8000
- api/index.py exposes the backend as a Vercel serverless endpoint by importing backend.app.app
- The local content script currently requires a reachable local backend for Gmail banner injection

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file.
