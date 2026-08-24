# Directories to Data — Fondren Library

**Houston City Directory Digitization Pipeline**
Fondren Fellows Internship, Fondren Library, Rice University. Summer 2026.

This project converts scanned pages of the 1900–1901 Houston city directory into structured, searchable data. 5,621 entries were extracted from 80 pages and delivered as a self-contained interactive HTML dashboard and a companion Excel workbook.

![Directory Dashboard, Browse view](Document/Dashboard - Browse.png)

![Directory Dashboard, Overview view](Document/Dashboard - Overview.png)

## Contents

- [Results](#results)
- [Quick start](#quick-start)
- [Pipeline](#pipeline)
- [Setup](#setup)
- [Running the pipeline](#running-the-pipeline)
- [Deliverables](#deliverables)
- [Data dictionary](#data-dictionary)
- [How it works](#how-it-works)
- [Repository structure](#repository-structure)
- [Data and source material](#data-and-source-material)
- [Limitations](#limitations)
- [Troubleshooting](#troubleshooting)
- [Citing this work](#citing-this-work)
- [Acknowledgments](#acknowledgments)
- [License](#license)

## Results

- **80 pages processed end to end.** 75 contained real entries; 5 were correctly recognized as full-page advertisements or dividers containing zero entries.
- **5,621 structured entries extracted**, each carrying up to 14 fields: name, occupation, employer, address type, racial marker as printed, ownership, and notes.
- **5,308 people and 269 businesses** identified, with **3,104 entries** linked to at least one other entry by shared address, employer, or surname.
- **159 entries (2.8%) flagged for manual review**, divided into a fragment signal (113) and a duplicate signal (46). Both thresholds were tested against the full dataset rather than assumed correct.
- **12 distinct bugs found and fixed**, several of which were silently losing or corrupting real data before they were caught. All are documented in Section 9 of the final report.

On accuracy: Step 3 measured 70.3% field-level accuracy against a hand-typed answer key. That figure is a **diagnostic on a single test page (page 200), taken before the Step 5 hardening**, and its purpose was to identify which of the 14 fields needed work. It is not a measure of the final 5,621-entry dataset, which has not been re-scored against ground truth. See Recommendations in the final report.

Full write-up: [`document/Directories_to_Data_Final_Report.pdf`](document/Directories_to_Data_Final_Report.pdf)

## Quick start

If you only want the data, you do not need to run anything:

- **`code/dashboard_output/Directory_Dashboard.html`** — download and open in any browser.
- **`code/dashboard_output/Directory_Dashboard.xlsx`** — open in Excel.

To rebuild the deliverables from the extracted data, run Step 6 alone. It takes under a minute and needs no API key. To re-extract from source images, see [Setup](#setup) and [Running the pipeline](#running-the-pipeline).

## Pipeline

Six notebooks in `code/`, run in order, each reading the previous step's output. Steps 1 through 4 build and validate a parsing recipe against one hand-verified page. Step 5 applies it at full scale. Step 6 converts the result into the end-user products.

| # | Notebook | Purpose | Output |
|---|----------|---------|--------|
| 1 | `step1_fix_boxes.ipynb` | Hand-correct OpenCV's first-pass bounding boxes on the test page | `corrected_boxes.json` (79 boxes) |
| 2 | `step2_type_ground_truth.ipynb` | Hand-type the answer key for every field of every entry | `ground_truth.json` (79 entries, 14 fields) |
| 3 | `step3_accuracy_comparison.ipynb` | Score pipeline output against ground truth, field by field | per-field accuracy, `mismatches.csv` |
| 4 | `step4_correction_loop.ipynb` | Test few-shot correction on five known error patterns | before/after comparison (negative result) |
| 5 | `step5_process_new_pages.ipynb` | Apply the hardened pipeline to all 80 pages | 5,621 entries (CSV, Excel, overlays) |
| 6 | `step6_build_dashboard.ipynb` | Compute relationships and quality flags, build deliverables | `Directory_Dashboard.html` and `.xlsx` |

Each notebook is self-documenting, with markdown cells explaining what each section does and why. Where the notebooks and the report disagree, the notebooks are the source of truth.

## Setup

Python 3.10 or later is required.

```bash
git clone https://github.com/leolaleo/Directories-to-Data---Fondren-library-.git
cd Directories-to-Data---Fondren-library-

python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

pip install -r requirements.txt
```

### API key

Steps 3 through 5 call the Google Gemini API, using the `gemini-2.5-flash` model. The notebooks look for a key in two places, in this order:

1. A file named `api_key.txt` in the working directory, containing only the key.
2. The `GEMINI_API_KEY` environment variable.

The notebooks resolve all paths relative to the working directory, so **run Jupyter from inside `code/`** and place `api_key.txt` there:

```bash
cd code
echo "your_key_here" > api_key.txt
jupyter notebook
```

`api_key.txt` and `.env` are gitignored. **Do not commit your key.** A key can be obtained from [Google AI Studio](https://aistudio.google.com/app/apikey).

Allow roughly 2 GB of disk space if you regenerate entry crops and page overlays.

## Running the pipeline

**Steps 1 and 2 are manual annotation tools.** They open browser-based editors for correcting boxes and typing ground truth by hand. Their outputs (`corrected_boxes.json`, `ground_truth.json`) are already committed, so these steps do not need to be re-run unless you are extending the project to a new page or a new directory edition.

**Steps 3 through 6 are runnable as-is.** Run them in order. Steps 3, 4, and 5 expect `output_ground_truth/ground_truth.json` to be present, and will stop immediately if it is not.

**Step 5 is a long job.** The full 80-page run makes thousands of API calls and took roughly 8.7 hours unattended. It is checkpointed per page: if it stops, re-running resumes from the last completed page rather than starting over, and nothing already processed is re-sent or re-billed. A legitimately empty page (an advertisement or divider) is recorded as a completed page with zero rows, not as a failure.

**To regenerate only the deliverables**, run Step 6 alone. It reads `step5_new_pages_output/csv/`, is self-contained, and needs no API key.

## Deliverables

Both are written to `code/dashboard_output/` by Step 6.

### `Directory_Dashboard.html`

A single self-contained file. No server, no install, no internet connection required. GitHub will not render it in the browser, so download the file and open it locally.

- **Browse.** Full-text search across name, address, occupation, and employer, with an optional sounds-like match, checkbox multi-select filtering on entry type and marker, sortable columns, and per-page and per-edition filters. Selecting an entry opens a detail panel showing its parsed fields alongside its affiliated business, others at the same address, coworkers at the same employer, and same-surname matches. Results are exportable to CSV.
- **Overview.** A metric explorer for viewing distributions across occupations, entry type, marker, and ownership, with control over ordering and depth, an optional split by marker, and the option to follow the active Browse filters. Clicking any bar filters Browse to those entries. Chart data is exportable.
- **Review.** The 159 flagged entries isolated for checking, labeled by which signal fired.

### `Directory_Dashboard.xlsx`

The same data as an Excel workbook: a summary sheet with collection-level counts, and a full entry sheet with one row per entry, all 14 fields, and separate `Fragment flag` and `Duplicate flag` columns. Values beginning with `=`, `+`, `-`, or `@` are written with a leading apostrophe so Excel keeps them as text instead of reading them as formulas.

## Data dictionary

Every entry carries up to 14 fields. Blank means the printed entry did not supply that information.

| Field | Description | Example |
|---|---|---|
| `entry_type` | One of `person`, `business`, `institution`, `cross_reference` | `person` |
| `last_name` | Surname, printed first in this directory | `Abercrombie` |
| `first_name` | Given name or initials as printed | `Leonard A.` |
| `business_name` | Firm name, for entries carrying no personal name | `Levy Bros., Dry Goods` |
| `occupation` | Occupation as printed, abbreviations preserved | `law student` |
| `employer` | Employer or firm the person works for | `Baker, Botts, Baker & Lovett` |
| `workplace_address` | Address of the workplace, where given separately | `212 Main` |
| `residence` | Home address, from `r.` or `h.` notation | `h. 2017 Main.` |
| `residence_qualifier` | Directional or positional qualifier | `nr`, `cor`, `over`, `es`, `ws`, `bt` |
| `boarding` | Boarding address, from `bds` notation | `bds 2205 Opelousas.` |
| `rooms` | Rooms address, from `rms` notation | `rms 803 Washington` |
| `race` | Racial marker exactly as printed | `(c)` |
| `ownership` | Ownership indicator where printed | `home` |
| `notes` | Widow status, cross-references, and other remarks | `wid Jas` |

**Values are recorded exactly as printed.** If the page reads `(c)`, the field holds `(c)`, not "colored." If it reads `servt`, the field holds `servt`, not "Servant." Abbreviations are expanded at the display layer using lookup tables, never in the stored data. This keeps the archival record unaltered and ensures every accuracy comparison measures the same thing on both sides.

The address notation matters and is easy to conflate. This directory distinguishes three separate concepts: `r.` and `h.` mean residence, `bds` means boarding, and `rms` means rooms. They are stored in three separate fields and are not interchangeable.

## How it works

**Page rendering.** The source PDF is rendered to page images with PyMuPDF at the resolution used for box detection and cropping.

**Box detection.** An OpenCV pass finds the column structure, then detects entry starts by reading the hanging indent directly on each row. An earlier approach that looked for vertical whitespace gaps between lines failed on this directory, whose lines are tightly leaded with no gap between one entry and the next. A minimum-gap merge suppresses false double-starts firing a few pixels apart at a single real line transition.

**OCR.** Each detected box is cropped and sent to Gemini. The call requests a JSON *list* of entries rather than a single transcribed string, so that a box which mistakenly contains two or three entries yields two or three rows instead of silently discarding all but the first.

**Field parsing.** Transcribed text is parsed into a Pydantic `DirectoryEntry` schema with constrained value types on `entry_type` and `ownership_type`. The prompt states this directory's conventions explicitly, beginning with the name-order rule (surname is always printed first), and runs at temperature 0 so identical input always parses identically.

**Reliability.** The OCR call is wrapped in a `tenacity` retry decorator with exponential backoff, five attempts, multiplier 2, waiting 5 to 60 seconds, so a transient API failure does not end a multi-hour run. Results are written per page, so the run resumes rather than restarts.

**Cleanup.** An adjacent-duplicate pass merges the case where a wrapped entry was split across two boxes, keeping the more complete row.

**Relationships and flags (Step 6).** Entries are linked on shared address, employer, and surname, using NaN-aware normalization so that blank fields do not group together into a false match. Two quality signals are computed: a fragment signal for entries too incomplete to stand alone, and a near-duplicate signal comparing full entry text with `difflib.SequenceMatcher` at a 0.92 similarity threshold. That threshold was set by testing against all 5,621 real entries rather than assumed.

## Repository structure

```
.
├── code/
│   ├── step1_fix_boxes.ipynb
│   ├── step2_type_ground_truth.ipynb
│   ├── step3_accuracy_comparison.ipynb
│   ├── step4_correction_loop.ipynb
│   ├── step5_process_new_pages.ipynb
│   ├── step6_build_dashboard.ipynb
│   ├── output_box_fixing/          # corrected_boxes.json, overlay image
│   ├── output_ground_truth/        # ground_truth.json, verified entry crops
│   ├── output_step3_accuracy/      # per-field accuracy, mismatches
│   ├── output_step4_correction/    # few-shot before/after results
│   ├── step5_new_pages_output/     # full-run CSVs, Excel, page stats
│   └── dashboard_output/           # Directory_Dashboard.html and .xlsx
├── document/
│   ├── Directories_to_Data_Final_Report.pdf
│   ├── dashboard_browse.png
│   └── dashboard_overview.png
├── requirements.txt
├── .gitignore
├── LICENSE
└── README.md
```

## Data and source material

The source material is the 1900–1901 Houston city directory, held by Fondren Library, Rice University.

The source scans are not redistributed in this repository. Access to the original page images can be requested from Fondren Library. The derived outputs produced by this pipeline, meaning the structured entry CSVs, the dashboard, and the Excel workbook, are included here.

**A note on historical content.** This directory records people using the conventions of its time, including racial or ethnic identifiers printed beside names. These are preserved exactly as printed rather than removed or modernized, because the printed record is the object of study and altering it would misrepresent the source. Nothing is inferred, standardized, or assigned. These are the 1900 publisher's terms, not descriptions of how any person identified themselves.

## Limitations

- Accuracy has been formally measured on one page only. The full 5,621-entry dataset has not been scored against ground truth.
- The 2.8% review flag is a heuristic. It surfaces likely fragments and likely duplicates; it is not a guarantee that unflagged entries are correct.
- Relationships are computed heuristically from shared field values and are indicative, not authoritative. A shared surname or address is a lead to follow, not a confirmed connection.
- OCR and parsing are model-based and non-deterministic in principle. Temperature is set to 0 for reproducibility, but model updates on the provider side may change output over time.
- The pipeline is tuned to this directory's specific layout and notation. Applying it to a different edition or publisher will require revisiting box detection and the parsing prompt.
- Box detection is not perfect. Merged and split boxes still occur; the quality flags exist precisely because of this.

## Troubleshooting

**`ModuleNotFoundError` on the first cell.** The virtual environment is not active, or `pip install -r requirements.txt` has not been run. Check that Jupyter is using the same environment you installed into.

**`AssertionError: API key required`.** No `api_key.txt` was found in the working directory and `GEMINI_API_KEY` is not set. The notebooks resolve paths relative to the working directory, so confirm you launched Jupyter from inside `code/`.

**`AssertionError: Ground truth not found`.** Steps 3 to 5 expect `output_ground_truth/ground_truth.json` relative to the working directory. Confirm it was cloned and that you are running from inside `code/`.

**Step 5 stopped partway.** Re-run the notebook. It resumes from the last completed page. Nothing already processed is re-sent or re-billed.

**Outputs are not where you expect.** Notebooks write relative to the working directory, not to the notebook file. Run `import os; print(os.getcwd())` in any cell to see where files are being written.

**The HTML dashboard shows raw code in the browser.** GitHub previews HTML as source. Download the file first, then open it locally.

## Citing this work

> Kathirvel, Abhirami. *Directories to Data: Houston City Directory Digitization Pipeline.* Fondren Fellows Program, Fondren Library, Rice University, 2026. https://github.com/leolaleo/Directories-to-Data---Fondren-library-

## Acknowledgments

Developed as part of the Fondren Fellows Program at Fondren Library, Rice University, under the supervision of Sean Smith, with Norie Guthrie.

Questions about the code or data: Abhirami Kathirvel, ak283@rice.edu

## License

Code in this repository is released under the MIT License (see [`LICENSE`](LICENSE)). Derived data outputs are released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Rights to the original source scans are held by Fondren Library, Rice University.
