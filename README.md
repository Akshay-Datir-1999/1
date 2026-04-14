<p align="center">
	<img src="https://capsule-render.vercel.app/api?type=waving&height=210&color=0:0B1220,40:111827,100:1F2937&text=CyberGuardian&fontSize=54&fontColor=22D3EE&fontAlignY=36&desc=AI%20Powered%20Cybersecurity%20Platform&descAlignY=58&descSize=18" alt="CyberGuardian Banner" />
</p>

<p align="center">
	<img src="https://img.shields.io/badge/Build-Passing-10B981?style=for-the-badge" alt="Build Status" />
	<img src="https://img.shields.io/badge/License-Private-111827?style=for-the-badge" alt="License" />
	<img src="https://img.shields.io/badge/Stack-Flask%20%7C%20SQLite%20%7C%20PyTorch%20%7C%20Scikit--learn-22D3EE?style=for-the-badge" alt="Tech Stack" />
</p>

# CyberGuardian - AI Powered Cybersecurity Platform

CyberGuardian is a production-oriented cybersecurity platform that helps detect modern digital threats using AI and machine learning.

It is designed to:
- Identify phishing URLs before users click malicious links.
- Analyze suspicious emails for fraud and phishing patterns.
- Detect deepfake images and videos using vision models.
- Provide dashboard insights for analysts and security teams.

Who can use CyberGuardian:
- Companies building internal threat-monitoring tools.
- Security analysts who need rapid triage and reporting.
- Individuals who want an easy way to validate links and media.

## 🚀 1. Project Title & Description

CyberGuardian combines web APIs, ML models, and a security dashboard into one platform.

Core problem areas solved:
- Phishing and spoofed URL attacks.
- Email-based social engineering and fraud.
- AI-generated deepfake media.

The platform is built for practical use: fast scans, clear confidence scores, role-based access, and extendable model pipelines.

## 🛠 2. Tech Stack (and Why)

### Frontend
- React: ideal for highly dynamic component-driven dashboards, real-time UI updates, and scalable state management in SaaS products.
- Tailwind CSS: speeds up UI development with utility classes, consistent design tokens, and cleaner responsive styling.

Note: Current repository UI is rendered using Flask templates with custom CSS/JS. React + Tailwind can be introduced as the next frontend evolution.

### Backend
- Python Flask: lightweight, flexible REST API framework that is easy to integrate with ML pipelines and rapid experimentation.

### Database
- SQLite: simple, embedded, zero-admin database for local development and small-to-medium deployments.

### AI/ML
- Scikit-learn: reliable for structured features and traditional ML baselines in phishing detection.
- TensorFlow / PyTorch: suited for deep learning workloads such as CNN-based deepfake detection.
- NLP techniques: improve analysis of email subject/body patterns, suspicious language, and semantic intent.

## 🧠 3. Algorithms Used

### Logistic Regression (Phishing Detection)
- What it does: learns a weighted decision boundary to classify malicious vs safe input.
- Why chosen: fast, interpretable, and strong baseline for text/feature classification.

### Random Forest (URL Classification)
- What it does: combines many decision trees and votes on the final class.
- Why chosen: robust against noisy features, captures non-linear behavior, and performs well without heavy tuning.

### CNN (Deepfake Image Detection)
- What it does: extracts visual patterns from images (textures, face artifacts, generation clues).
- Why chosen: state-of-the-art approach for computer vision classification tasks.

### NLP Techniques (Email Analysis)
- What it does: transforms email sender/subject/body into machine-understandable features.
- Why chosen: phishing emails rely heavily on language manipulation, urgency signals, and impersonation patterns.

## 📂 4. Folder Structure

```text
CyberGuardian_fixed/
	frontend/
		static/            # CSS/JS and client assets
		templates/         # HTML templates and dashboard pages

	backend/
		app.py             # backend launcher
		cyberguardian/     # API routes, auth, detectors, DB service layer
			detectors/       # phishing and deepfake detection modules
			training/        # model training utilities
		models/            # trained model artifacts (.joblib, .pth)
		scripts/           # training entry scripts

	database/
		cyberguardian.db   # SQLite database file

	app.py               # root launcher (imports backend)
	requirements.txt
```

Folder intent summary:
- frontend: user-facing dashboard and static UI assets.
- backend: business logic, APIs, auth, model execution.
- models: persisted ML artifacts used at runtime.
- api: implemented under backend/cyberguardian routes and controllers.
- database: isolated persistence layer.

## 🔌 5. API Design

CyberGuardian follows REST principles:
- JSON request/response contracts.
- Endpoint-per-capability design.
- Predictable status codes (200, 400, 401, 403, etc.).

Representative endpoints:
- POST /detect-url (scan-url equivalent)
- POST /detect-email (analyze-email equivalent)
- POST /detect-image and POST /detect-video (detect-deepfake equivalent)
- GET /api/stats, GET /api/reports for analytics

Flask was selected because it provides:
- Fast API development for ML-backed features.
- Easy integration with Python data/ML ecosystem.
- Clean extensibility for authentication and role-based access.

## 🔄 6. Workflow (Step-by-Step)

```text
User Input -> Frontend -> Flask API -> ML Model Inference -> Risk Score/Label -> Dashboard
```

Detailed flow:
1. User submits URL/email/image/video from dashboard.
2. Frontend sends secure request to backend API.
3. Backend validates request and routes it to the right detector.
4. ML model performs inference and produces label + confidence.
5. Result is stored in database for audit/history.
6. Dashboard displays decision with threat context and metrics.

## ⚙️ 7. Features

- Phishing URL Scanner
- Email Analyzer
- Deepfake Detector (image and video)
- Dashboard Analytics
- Real-time style threat detection experience
- Role-based authentication and admin controls
- Reporting and recent scan history

## 💻 8. Installation Guide

### 1. Clone repository
```bash
git clone <your-repo-url>
cd CyberGuardian_fixed
```

### 2. Create virtual environment
```bash
python -m venv .venv
```

### 3. Activate virtual environment
Windows (PowerShell):
```powershell
.\.venv\Scripts\Activate.ps1
```

macOS/Linux:
```bash
source .venv/bin/activate
```

### 4. Install dependencies
```bash
pip install -r requirements.txt
```

### 5. Run backend
```bash
python app.py
```

### 6. Run frontend
Current build uses Flask-rendered templates served by the same backend process. Open:
- http://127.0.0.1:5000

If you later migrate to React, run frontend dev server separately and connect it to Flask APIs.

## 📊 9. Future Improvements

- Real-time monitoring and event streaming
- Browser extension for instant URL reputation checks
- Stronger model training pipelines and evaluation dashboards
- Cloud deployment (Docker, CI/CD, managed DB, observability)
- Threat intelligence feed integration
- Multi-tenant enterprise architecture

## 📌 10. Conclusion

CyberGuardian demonstrates how AI can be applied to practical cybersecurity operations in a clean, extensible platform.

By combining phishing detection, email analysis, and deepfake detection in one unified system, it reduces response time and helps users make safer decisions faster.

If you are building a modern AI-first security product, CyberGuardian is a strong production-style foundation.

---

## Quick Security Note

- Rotate default credentials before production use.
- Set a strong environment secret: CYBERGUARDIAN_SECRET.
- Keep database and model files outside public repositories.
