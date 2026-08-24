
## Data

The source material is the 1900–1901 Houston city directory, held by Fondren Library, Rice University.

The source scans are not redistributed in this repository. Access to the original page images can be requested from Fondren Library, Rice University. The derived outputs produced by this pipeline (structured entry CSVs, the dashboard, and the Excel workbook) are included here.

Transcribed values are recorded exactly as printed, including racial markers such as `(c)` and abbreviations such as `servt`. These are not normalized, expanded, or modernized in the stored data. This is deliberate: the ground truth records what is on the page so that every downstream comparison measures the same thing on both sides, and so that the historical record is not silently altered. Abbreviation expansion happens at the display layer using lookup tables.

## Limitations

- Accuracy has been formally measured on one page only. The full 5,621-entry dataset has not been scored against ground truth.
- The 2.8% review flag is a heuristic. It surfaces likely fragments and likely duplicates; it is not a guarantee that unflagged entries are correct.
- OCR and parsing are model-based and non-deterministic in principle. Temperature is set to 0 for reproducibility, but model updates on the provider side may change output over time.
- The pipeline is tuned to this directory's specific layout and notation. Applying it to a different edition or publisher will require revisiting box detection and the parsing prompt.

## Citing this work

If you use this data or code, please cite:

> Kathirvel, Abhirami. *Directories to Data: Houston City Directory Digitization Pipeline.* Fondren Fellows Program, Fondren Library, Rice University, 2026. https://github.com/[your-username]/directories-to-data-fondren-library

## Acknowledgments

Developed as part of the Fondren Fellows Program at Fondren Library, Rice University, under the supervision of Sean Smith, with Norie Guthrie.

## License

Code in this repository is released under the MIT License (see `LICENSE`). Derived data outputs are released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Rights to the original source scans are held by Fondren Library, Rice University.
