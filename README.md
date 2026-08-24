Directories to Data — Fondren Library
Houston City Directory Digitization Pipeline. Fondren Fellows Internship, Summer 2026.

This project converts scanned pages of the 1900–1901 Houston city directory into structured, searchable data: 5,621 entries extracted from 80 pages, delivered as a self-contained interactive HTML dashboard and a companion Excel workbook.
Results
80 pages processed end to end; 75 contained real entries, 5 were correctly recognized as full-page advertisements or dividers.
5,621 structured directory entries extracted, each carrying up to 14 fields (name, occupation, employer, address type, racial marker as printed, ownership, notes).
159 entries (2.8%) flagged for manual review, split into a fragment signal and a duplicate signal, both validated against the real data rather than assumed correct.
Field-level accuracy on the hand-verified test page: 70.3% overall (152 mismatches across 14 fields).
12 distinct bugs found and fixed over the course of the project, documented in full in docs/Directories_to_Data_Final_Report.pdf.
Pipeline
Six notebooks, run in order, each reading the previous step's output.

#
Notebook
Purpose
Output
1
step1_fix_boxes.ipynb
Hand-correct OpenCV's first-pass bounding boxes on a test page
corrected_boxes.json
2
step2_type_ground_truth_v3.ipynb
Hand-type the answer key for every field of every entry on that page
ground_truth.json
3
step3_accuracy_comparison.ipynb
Score the pipeline's output against ground truth, field by field
per-field accuracy tables, mismatches.csv
4
step4_correction_loop.ipynb
Test few-shot correction on known error patterns (mixed/negative result)
before/after accuracy comparison
5
step5_process_new_pages_14.ipynb
Apply the hardened pipeline to all 80 pages of the source PDF
5,621 structured entries (CSV, Excel)
6
step6_build_dashboard.ipynb
Compute relationships and quality flags, build the deliverables
Directory_Dashboard.html, Directory_Dashboard.xlsx


Each notebook is self-documenting, with markdown cells explaining what each section does and why. The full write-up, including all 12 debugged issues and the reasoning behind each design choice, is in docs/Directories_to_Data_Final_Report.pdf.
Deliverables
Directory_Dashboard.html — self-contained interactive dashboard (browse, search, filter, and review flagged entries) with no server or install required.
Directory_Dashboard.xlsx — companion Excel workbook of the same data.
Requirements
Python 3.10+
Google Gemini API key (for OCR and field parsing in Steps 3–5)
Jupyter (pip install notebook)
Data
Source material is the 1900–1901 Houston city directory, a public-domain historical document. Transcribed values (including racial markers) are recorded exactly as printed for historical accuracy; see the final report for the scope note on this.
License
See LICENSE.
