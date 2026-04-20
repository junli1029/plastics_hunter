# 🌊 Plastics Hunter — Ocean Cleanup Route Optimizer

An interactive Streamlit dashboard that visualises optimal vessel routes for
collecting marine plastic debris in Southeast Asia, based on HYCOM ocean
current forecasts and a particle-drift simulation.

**Live demo:** https://plastic-hunter-dashboard.streamlit.app

---

## 📦 Project layout

```
plastics_hunter/
├── app.py              ← Dashboard UI (Streamlit)
├── backend.py          ← Thin API wrapper around the model bundle
├── requirements.txt    ← Python dependencies
├── .streamlit/
│   └── config.toml     ← Theme + server config
├── data/               ← (auto-created) Bundle cache — gitignored
└── README.md
```

The optimization model itself is a large cloudpickle (~190 MB) hosted on
Google Drive and downloaded on first launch.

---

## 🚀 Quick start (local development)

```bash
# 1. Clone the repo
git clone https://github.com/<YOUR_ORG>/plastics_hunter.git
cd plastics_hunter

# 2. Install dependencies
pip install -r requirements.txt

# 3. Run the dashboard
streamlit run app.py
```

On first run, `backend.py` automatically downloads the optimization bundle
from Google Drive into `./data/`. Subsequent runs use the cached file.

> **First run takes a few minutes** while the 190 MB bundle downloads.

---

## 👥 Collaboration guide

### Workflow at a glance

```
Local edit → git push → GitHub → Streamlit Cloud auto-redeploys (~1-2 min)
```

Every push to `main` triggers an automatic redeploy on the public URL.

### Teammate A — backend model updates

Your job: keep the optimization model fresh.

1. Continue developing in your research notebook.
2. When ready, export a new bundle as `optimization_dashboard_bundle.pkl`
   with this **exact contract**:

   ```python
   bundle = {
       "meta":      {BBOX, PORT_LON, PORT_LAT, n_prod_snapshots, ...},
       "query":     callable(departure_day, port, trip_days, n_vessels)
                    -> (routes, plastic, fuel, forecast),
       "find_best": callable(port, trip_days, candidate_days, n_vessels)
                    -> list of dicts (see backend.py for keys),
   }
   import cloudpickle
   with open("optimization_dashboard_bundle.pkl", "wb") as f:
       cloudpickle.dump(bundle, f)
   ```

3. Upload to Google Drive:
   - **Preferred** (no code change needed): right-click the existing bundle
     file in Drive → *Manage versions* → *Upload new version*. The file ID
     stays the same; Streamlit Cloud will pick it up on the next reload.
   - **Alternative**: upload as a new file, copy its ID (the bit between
     `/d/` and `/view` in the share URL), update `BUNDLE_GDRIVE_ID` in
     `backend.py`, commit, push.

4. On Streamlit Cloud, click *Manage app → Clear cache* (or "Reboot app")
   to force the dashboard to re-download the new bundle. Otherwise the
   old cached bundle stays in memory.

### Teammate B — dashboard UI updates

Your job: UI/UX, new charts, layout changes.

1. Edit `app.py` (and helpers inside it).
2. Test locally:
   ```bash
   streamlit run app.py
   ```
3. Commit and push:
   ```bash
   git add app.py
   git commit -m "Improve KPI card layout"
   git push
   ```
4. Wait ~1–2 minutes, then refresh the public URL. Done.

You do **not** need to touch `backend.py` unless you want to change the API
surface (add a new query function, for example).

### Handling merge conflicts

If you and another teammate edit the same file:

```bash
git pull --rebase          # get latest main
# resolve conflicts in the flagged files
git add <file>
git rebase --continue
git push
```

To minimise conflicts, split work by file: B owns `app.py`, A owns
`backend.py` (or at least `BUNDLE_GDRIVE_ID` and the model contract).

---

## 🔐 Secrets

The bundle's Google Drive file ID is **not** a secret (anyone with the
public link can download it), so we hard-code it in `backend.py` for
simplicity. If you ever need real secrets (API keys, etc.), use Streamlit
Cloud's *Settings → Secrets* panel and read them via `st.secrets[...]`.

---

## ❓ FAQ

**Q: The app hangs on "Loading optimization model..."**
A: First run downloads ~190 MB. Give it 1–2 minutes. If it still fails,
   check that `BUNDLE_GDRIVE_ID` is set correctly in `backend.py` and the
   file is shared as "Anyone with the link".

**Q: I updated the model but the dashboard shows old results.**
A: Streamlit caches the bundle in memory. Go to *Manage app → Reboot*
   on Streamlit Cloud, or clear cache locally by deleting `data/*.pkl`.

**Q: Streamlit Cloud says "out of memory".**
A: The free tier has a 1 GB limit. The bundle itself is ~190 MB loaded,
   so there's headroom, but large simulation runs may push over. Consider
   shrinking the bundle (subsample particles, lower grid resolution).

---

## 📄 License

Coursework project for Columbia University IEOR E4550 Analytics Lab.
