# Stress-Strain Analyzer

A web app that automatically extracts mechanical properties from stress-strain curve data. Upload a CSV file (or pick from the built-in material library) and get Young's Modulus, Yield Strength, UTS, Toughness, and more — instantly.

Built for the MLL253 course at IIT Delhi.

---

## What It Does

- Reads stress-strain data from a CSV file
- Smooths noisy data using a Savitzky-Golay filter
- Detects the elastic region and calculates **Young's Modulus**
- Finds the **0.2% offset Yield Point** (ASTM E8 standard)
- Identifies **Ultimate Tensile Strength (UTS)** and necking
- Calculates **Resilience** and **Toughness** by numerical integration
- Fits the Hollomon power law for **Strain Hardening** (σ = Kεⁿ)
- Generates interactive Plotly charts + a static Matplotlib graph
- Supports **up to 10 materials** simultaneously for comparison
- Exports results as PDF report, Excel, JSON, or CSV

---

## Setup & Running

You need **Python 3.8+** and **Node.js** installed.

### Step 1 — Start the Backend

```bash
cd backend
pip install -r requirements.txt
python app.py
```

Runs on `http://localhost:5000`

### Step 2 — Start the Frontend

```bash
cd frontend
npm install
npm start
```

Opens automatically at `http://localhost:3000`

> Run both at the same time in two separate terminals.

---

## How to Use

### Option A — Upload your own CSV

1. Go to the **Upload CSV** tab
2. Drag & drop (or click to browse) your CSV file
3. File must have at least 2 columns: `Strain` and `Stress`, with 20+ rows
4. Adjust smoothing settings if needed (defaults work for most data)
5. Click **Run Analysis**

**CSV format example:**
```
Strain,Stress
0.0001,20
0.0002,40
0.001,200
0.002,400
0.01,591
0.11,920
...
```

A sample file `sample_stress_strain.csv` is included in the project root.

### Option B — Use the Material Library

1. Go to the **Material Library** tab
2. Browse 7 real experimental datasets:
   - DP780 Dual-Phase Steel
   - Aluminum AA6016A and AA5086
   - Natural Rubber NR60
   - EPDM Rubber
   - Neoprene Rubber
   - Flax Fiber Bundle
3. Click any material to analyze it instantly

### Comparing Multiple Materials

- Add up to 10 materials (mix of uploaded files and library materials)
- An interactive overlay chart appears automatically once you add 2+
- Use the **Cluster** button to run K-means clustering on their properties
- Use the **Export** button to download results

---

## Results Explained

| Property | Unit | What it means |
|---|---|---|
| Young's Modulus (E) | GPa | Stiffness — resistance to elastic deformation |
| Yield Strength | MPa | Stress at which permanent deformation begins (0.2% offset) |
| Yield Strain | — | Strain at yield point |
| UTS | MPa | Maximum stress the material can withstand |
| % Elongation | % | Total stretch before fracture — measures ductility |
| Resilience | MPa | Energy absorbed elastically (area under elastic curve) |
| Toughness | MPa | Total energy absorbed before fracture (area under full curve) |
| Strain Hardening (n) | — | How much the material strengthens as it deforms plastically |
| Strength Coefficient (K) | MPa | Scale factor in Hollomon equation σ = Kεⁿ |

---

## Analysis Methods

### Mathematical (default)
Uses pure numerical algorithms — no machine learning:
- Deviation-based elastic region detection
- ASTM E8 standard 0.2% offset yield method
- Trapezoidal numerical integration for resilience/toughness
- Log-log linear regression for strain hardening

Handles both **metals** (distinct elastic-plastic regions) and **elastomers** (rubbers with >50% elongation) automatically.

### Machine Learning (optional)
Uses scikit-learn linear regression:
- Fits σ = Eε in the elastic region
- Fits σ = Kεⁿ in log-space for the plastic region
- Derives properties analytically from the fitted models

Switch between methods in the Parameters panel.

---

## Project Structure

```
MLL253-Project/
├── backend/
│   ├── app.py                  # Flask API — all analysis logic
│   ├── material_library.py     # 7 real experimental datasets
│   └── requirements.txt        # Python dependencies
│
├── frontend/
│   └── src/
│       ├── App.js              # Main React app
│       └── components/
│           ├── FileUpload.js       # CSV drag-and-drop upload
│           ├── ParametersForm.js   # Smoothing & method settings
│           ├── Results.js          # Single material results display
│           ├── ComparisonChart.js  # Interactive multi-material Plotly chart
│           ├── ComparisonResults.js# Side-by-side property comparison
│           ├── MaterialSelector.js # Library browser
│           ├── InteractiveCurve.js # Plotly stress-strain curve
│           ├── ClusteringPanel.js  # K-means clustering + PCA
│           ├── ExportPanel.js      # PDF / Excel / JSON / CSV export
│           ├── SessionManager.js   # Save and load sessions
│           ├── ThemeToggle.js      # Light / dark mode
│           └── Tutorial.js         # First-time user guide
│
├── sample_stress_strain.csv    # Sample data to test with
├── stress_strain.ipynb         # Original course assignment notebook
├── QUICKSTART.md               # One-page quick reference
└── README.md                   # This file
```

---

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/health` | Check if backend is running |
| POST | `/api/analyze` | Analyze uploaded CSV file |
| POST | `/api/analyze-material` | Analyze a library material |
| GET | `/api/materials` | List all library materials |
| GET | `/api/materials/<id>` | Get raw data for one material |
| POST | `/api/generate-comparison` | Generate comparison bar charts |
| POST | `/api/generate-report` | Generate PDF report |
| POST | `/api/export` | Export as Excel / JSON / CSV |
| POST | `/api/cluster-materials` | K-means clustering + PCA |
| POST | `/api/save-session` | Save current session to file |
| POST | `/api/load-session` | Load a saved session |
| POST | `/api/preview-csv` | Preview and detect CSV format |

---

## Troubleshooting

**"An error occurred during analysis"**
- Make sure the backend is running (`python app.py` in the `backend/` folder)
- Check that your CSV has at least 20 data rows
- Open browser DevTools (F12) → Network tab to see the actual error from the server

**"No module named flask"**
- Run `pip install -r requirements.txt` inside the `backend/` folder

**CORS error in browser**
- Backend must be running on port 5000 before you start the frontend

**CSV not recognized**
- Make sure the first row is the header: `Strain,Stress`
- Strain values should be dimensionless (e.g. 0.002, not 0.2%)
- Stress values in MPa

---

## References

- ASTM E8 — Standard Test Methods for Tension Testing of Metallic Materials
- Savitzky, A. & Golay, M.J.E. (1964) — Smoothing and Differentiation of Data
- Hollomon, J.H. (1945) — Tensile Deformation, *Trans. AIME*
- Material datasets: Zenodo (CC-BY 4.0) and Dryad (CC0 1.0) — see citations in `material_library.py`

---

## Authors

MLL253 Project — IIT Delhi
