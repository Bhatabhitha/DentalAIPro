<<<<<<< HEAD
🦷 DentalAI Pro — 3D Orthodontic Analysis System

A full-stack AI-powered web application that performs **automated tooth segmentation and orthodontic diagnosis** on 3D dental scan data using a custom-trained **PointNet deep learning model**.

---

📌 Project Overview

DentalAI Pro allows dental professionals to upload 3D point cloud scans of dental arches (`.npz` format), run AI-based tooth segmentation, visualize results interactively, and receive an automated orthodontic diagnosis — all through a clean, secure web interface.


🏗️ Project Structure

Dental_AI_project/
├── backend/
│   ├── main.py                        # FastAPI server — model, auth, API endpoints
│   └── best_pointnet_model (1).pth    # Trained PointNet model weights
├── frontend/
│   ├── src/
│   │   ├── App.jsx                    # Main React application
│   │   ├── Login.jsx                  # Login page component
│   │   ├── App.css                    # Styling
│   │   └── main.jsx                   # React entry point
│   ├── index.html
│   ├── package.json
│   └── vite.config.js
└── extra testing data/                # Sample .npz scan files for testing



 🤖 AI Model — PointNet Segmentation

 Architecture: `PointNetSeg`
A point cloud segmentation neural network adapted from the original PointNet architecture.

| Layer | Input → Output channels |
|---|---|
| Conv1d + BatchNorm | 3 → 64 |
| Conv1d + BatchNorm | 64 → 128 |
| Conv1d + BatchNorm | 128 → 1024 |
| Global Max Pooling | Extracts global feature vector |
| Conv1d + BatchNorm | 1088 → 512 (local + global concatenated) |
| Conv1d + BatchNorm | 512 → 256 |
| Conv1d (output) | 256 → 50 classes |

**Key design**: Local features (64-dim) are concatenated with the global feature (1024-dim) to give each point both local and global context — this is the hallmark of PointNet segmentation.

 Model Performance

| Metric | Score |
|---|---|
| Overall Accuracy | 80.32% |
| Weighted Precision | 82.49% |
| Weighted Recall | 80.32% |
| Weighted F1 Score | 80.72% |
| Total Points Evaluated | 163,840 |
| Output Classes | 50 (tooth IDs + background) |

---

🔬 How the AI Pipeline Works

1. **Input**: User uploads a `.npz` file containing `points` (Nx3 XYZ coordinates) and optional `labels`
2. **Normalization**: Points are centered and scaled to a unit sphere `[-1, 1]` to remove scale/position bias
3. **Inference**: The PointNet model assigns a class (tooth ID 1–49, or 0 = background/gum) to every point
4. **Centroid computation**: The 3D center of each predicted tooth cluster is calculated in both raw (real-world) and normalized (display) coordinate spaces
5. **Arch curve fitting**: A **2nd-degree polynomial (parabola)** is fitted through all tooth centroids to model the ideal dental arch
6. **Diagnosis**: Rule-based logic compares actual tooth positions vs. the fitted arch

---

🩺 Diagnosis Logic

The `get_diagnosis()` function analyzes three conditions:

1. Crowding (using PCA-based width estimation)
- Each tooth's width is estimated using **PCA on its point cluster** (4× std dev along minor axis ≈ 95% of width)
- Adjacent teeth are compared: if inter-centroid distance < **85% of expected** → crowding flagged

2. Spacing
- If inter-centroid distance > **120% of expected** → spacing flagged

 3. Displacement / Arch Misalignment
- Each tooth centroid's Y-position is compared against the fitted parabolic arch
- If deviation > **2.0 mm** → displacement flagged

 Diagnosis Output Categories
| Condition | Trigger |
|---|---|
| Normal Alignment | No issues detected |
| Mild Crowding | ≥ 1 crowding instance |
| Severe Crowding | ≥ 2 crowding instances |
| Generalized Spacing | ≥ 2 spacing instances |
| Arch Misalignment | ≥ 2 displacement instances |

---

🔐 Authentication System

The app uses **JWT (JSON Web Token)** based authentication.

- Token algorithm: **HS256**
- Token expiry: **8 hours**
- Passwords are hashed with **bcrypt** (compatible with Python 3.13 — passlib was intentionally avoided due to bcrypt>=4 incompatibility)

Default Users

| Username | Password | Role |
|---|---|---|
| `doctor` | `dental123` | doctor |
| `admin` | `admin123` | admin |

> ⚠️ Change the `SECRET_KEY` in `main.py` before deploying to production.

Protected Endpoints
- `POST /predict` — requires valid Bearer token
- `GET /metrics` — requires valid Bearer token
- `GET /health` — public, no auth needed
- `POST /auth/login` — public, returns JWT token

---

🖥️ Frontend Features

Built with **React 19 + Vite + Tailwind CSS + Plotly.js**

Pages & Tabs

**Login Page**
- JWT-based login form
- Session stored in `sessionStorage`
- Auto-restores token on page refresh

**Scan Diagnosis Tab**
- Drag-and-drop `.npz` file upload
- "Run AI Analysis" button triggers backend inference
- **Segmentation chart**: 3D scatter plot with color-coded tooth labels; toggle between *AI Prediction* vs *Ground Truth*
- **Alignment chart**: Shows current tooth centroid positions (red) vs. simulated ideal positions (green) on the fitted arch curve, with dotted correction paths
- Skeleton loading animation during inference
- Downloadable `.txt` diagnosis report

**Model Metrics Tab**
- Displays accuracy, precision, recall, F1 score fetched live from the backend
- Dataset info: architecture name, total points, output classes

Backend Connectivity
- Live server status indicator in sidebar (green dot = connected, red = offline)
- Logged-in user's name and role shown in sidebar and topbar
- Sign out button clears session and resets state

---

⚙️ Tech Stack

| Layer | Technology |
|---|---|
| AI Model | PyTorch (PointNet) |
| Backend | FastAPI (Python) |
| Auth | JWT (`python-jose`) + bcrypt |
| Data processing | NumPy, scikit-learn (PCA) |
| Frontend | React 19, Vite |
| 3D Visualization | Plotly.js / react-plotly.js |
| HTTP Client | Axios |
| Styling | Tailwind CSS, custom CSS |

---

🚀 Setup & Running

Backend

```bash
cd backend

# Install dependencies
pip install fastapi uvicorn torch numpy scikit-learn python-jose[cryptography] bcrypt python-multipart

# Run the server
uvicorn main:app --reload --port 8000
```

The API will be available at `http://127.0.0.1:8000`

Frontend

```bash
cd frontend

# Install dependencies
npm install

# Start development server
npm run dev
```

The app will be available at `http://localhost:5173`

---

📂 Data Format

Input files must be NumPy `.npz` archives with:
- `points` — shape `(N, 3)`, float32 XYZ coordinates of the 3D scan
- `labels` *(optional)* — shape `(N,)`, integer ground-truth tooth IDs for comparison

Sample test files are included in the `extra testing data/` folder (upper and lower arch scans for multiple patients).

---

📡 API Reference

`POST /auth/login`
Authenticate and receive a JWT token.
```json
// Form fields: username, password
// Response:
{ "access_token": "...", "token_type": "bearer", "name": "Dr. Sharma", "role": "doctor" }
```

`POST /predict` *(auth required)*
Upload a `.npz` scan and get AI predictions.
```json
// Response:
{
  "points": [[x,y,z], ...],
  "labels": [tooth_id, ...],
  "ground_truth": [tooth_id, ...],
  "centroids": [[x,y,z], ...],
  "aligned_centroids": [[x,y,z], ...],
  "centroid_ids": [1, 2, ...],
  "curve": { "x": [...], "y": [...], "z": [...] },
  "diagnosis": { "title": "Mild Crowding", "details": ["Overlap: Tooth 3 & 4"] }
}
```

`GET /metrics` *(auth required)*
Returns model evaluation metrics.

`GET /health` *(public)*
Returns `{ "status": "ok" }` — used by frontend for connectivity check.

---
🔮 Future Improvements

- Persistent database (PostgreSQL/SQLite) for user management and patient history
- Support for STL/OBJ 3D mesh formats
- Per-tooth severity scoring
- Report export as PDF
- Model retraining pipeline with new data

---

## 👨‍💻 Development Notes

- The `main.py` contains commented-out versions of earlier iterations (without auth, without alignment simulation) preserved for reference
- Point cloud data is subsampled by factor of 5 (`[::5]`) before sending to frontend to reduce payload size
- The frontend uses `sessionStorage` (not `localStorage`) so the session clears on tab close
=======
# DentalAIPro
>>>>>>> e7ed3b2d28bb33e718afd119e054a52474c0029e
