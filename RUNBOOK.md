# Runbook — steps to run on your own machine

I edited `scripts/make_info_card.py`, and wrote `README.md`, `requirements-local.txt`,
and `.github/workflows/update-profile-art.yml` for you (none of these existed in
what you uploaded). Everything else below needs real internet/GitHub access, which
my sandbox doesn't have — run these yourself.

## 0. Get the folder onto your machine
Download the files I produced and drop them into one folder shaped like this:

```
abhijeetmishra2104/
  README.md
  requirements-local.txt
  scripts/
    fetch_contributions.py
    make_ascii_svg.py
    make_info_card.py      <- already edited with your real info
    prep_photo.py
    render_heatmap_svg.py
    requirements.txt
  .github/workflows/update-profile-art.yml
  data/                    <- empty, gets filled by step 3
```

Save your photo (the one on Drive) into this folder as `source-photo.jpg` —
I can't pull it from a Google Drive link myself.

## 1. Install dependencies
```bash
pip install -r requirements-local.txt        # pillow, numpy, opencv-python, rembg, onnxruntime
pip install -r scripts/requirements.txt       # requests, beautifulsoup4
```

## 2. Create the repo (must be named exactly your username)
```bash
gh repo create abhijeetmishra2104 --public --source=. --remote=origin
# if you don't have gh CLI / aren't logged in:
#   create a public repo named "abhijeetmishra2104" on github.com,
#   then: git init && git remote add origin git@github.com:abhijeetmishra2104/abhijeetmishra2104.git
```

## 3. Portrait
```bash
python scripts/prep_photo.py source-photo.jpg source-prepped.png
python scripts/make_ascii_svg.py
```
Preview (macOS):
```bash
qlmanage -t -s 900 -o . avi-ascii.svg
```
Remember: the SVG is blank at t=0 and types itself in on load. To preview the
frozen final frame instead:
```bash
STATIC=1 python scripts/make_ascii_svg.py
qlmanage -t -s 900 -o . avi-ascii.svg
```
If your face reads as a dark blob or washes out, tune in this order:
1. `clipLimit` in `prep_photo.py` (higher = more local contrast)
2. `GAMMA` in `make_ascii_svg.py` (higher = brighter midtones, pushes face into sparser chars)
3. `WHITE_FLOOR` (lower = more background clipped to blank)
4. `CONTRAST` (small nudges only, ±0.05)

Re-run both scripts after any change.

## 4. Info card
Already filled in — just render it:
```bash
python scripts/make_info_card.py
```
If you want to tweak wording later, edit the `ROWS` list at the top of
`scripts/make_info_card.py` and re-run. If it overflows the card, bump `H`
(currently `430`) at the top of the script.

## 5. Contribution graph
```bash
GH_PROFILE_USER=abhijeetmishra2104 python scripts/fetch_contributions.py
python scripts/render_heatmap_svg.py
```

## 6. Push everything
```bash
git add .
git commit -m "Add animated profile README"
git branch -M main
git push -u origin main
```

## 7. Turn on the daily auto-refresh
- GitHub → your repo → **Settings → Actions → General → Workflow permissions** → select **Read and write permissions** → Save.
- Then trigger it once so the graph exists immediately:
```bash
gh workflow run update-profile-art.yml
```
(or: Actions tab → "Update profile art" → Run workflow)

## 8. View it
`https://github.com/abhijeetmishra2104` — the README renders automatically on
your profile.

---

### Later: quick re-tune commands
```bash
# re-tune the portrait after editing GAMMA/WHITE_FLOOR/CONTRAST/clipLimit
python scripts/prep_photo.py source-photo.jpg source-prepped.png && python scripts/make_ascii_svg.py

# edit the info panel (ROWS/HOST in make_info_card.py) then:
python scripts/make_info_card.py

# manually refresh the contribution graph without waiting for the daily cron:
GH_PROFILE_USER=abhijeetmishra2104 python scripts/fetch_contributions.py && python scripts/render_heatmap_svg.py
```
