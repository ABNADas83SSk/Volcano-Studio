# Volcano Studio

**An interactive, browser-based tool for volcano plot visualization and exploratory analysis of differential abundance and expression data.**

Version 1.2 · English / 日本語 ([Japanese documentation](README.ja.md))

## Overview

Volcano Studio is designed to simplify the visualization, exploration, and presentation of differential analysis results. It combines an editable spreadsheet-like interface with an interactive volcano plot, enabling researchers to move directly from tabulated statistical results to customized figures without programming.

The tool is suitable for transcriptomics, quantitative proteomics, metabolomics, proximity-labeling experiments, and other studies that report fold changes and p-values. Identifiers are arbitrary strings, including Arabidopsis AGI codes, gene symbols, protein accessions, and metabolite identifiers. The tool displays the supplied identifiers; it does not automatically retrieve gene annotations.

## Getting started

Open `index.html` in a modern desktop browser, or access the GitHub Pages deployment once published. `ja.html` starts in Japanese. The language selector switches between English and Japanese. No installation or account is required. Select Demo to explore synthetic example data; these are not experimental measurements.

## Input data

Provide three columns in this order:

| ID | Fold change (Sample 1 / Sample 2) | p-value |
| --- | ---: | ---: |
| AT1G30440.1 | 2.0 | 0.001 |
| AT2G40980.1 | -1.5 | 0.01 |
| AT3G23160.1 | 0.2 | 0.50 |

The example uses log2 fold changes and illustrative p-values, not measured biological results. Copy cells from Excel and paste into the sheet, or import an XLSX, CSV, TSV or TXT file. For XLSX, the first worksheet is used. Select the correct fold-change mode before interpreting the plot. Linear fold changes must be strictly positive. Log2 fold changes may be negative, zero or positive. Changing the mode reinterprets the input numbers rather than converting the stored values.

Supply **unadjusted p-values** from an appropriate upstream statistical analysis. Do not supply already adjusted p-values if you intend to use the built-in BH correction. Each row is treated as one hypothesis; duplicate identifiers are not merged. Rows with missing identifiers, missing numeric values, invalid fold changes or p-values outside [0,1] are excluded from calculation and plotting.

## Features and workflow

- Edit cells, paste multiple rows, add rows, or import files. Clear all data with confirmation; undo/redo retains up to 80 history states. Plot dimensions are not included in undo history.
- Sort by ID, fold change, p-value, BH-adjusted p-value or significance status. Filter significant, increased, decreased or selected rows, and search identifiers.
- Choose raw or BH-adjusted p-values for classification and for the vertical axis. Set a p-value threshold, minimum absolute log2 fold change and optional exploratory S0 curve parameter.
- Hover over points to inspect values, click to select, and right-click to open the label/selection menu. Drag labels to reposition them. Customize color, marker shape and size for selected points or significance groups.
- Change plot dimensions by dragging the lower-right corner or entering width and height. Customize the plot title and label font size.
- Export the full sheet or visible rows as XLSX, CSV or tab-delimited TXT; export plots as SVG, PDF or PNG (1–4× raster scaling). To export in the displayed sort order, select Visible rows only; full export follows the underlying row order.
- Save and reload a JSON project containing data, point styles, label positions and core plot settings. Keep the original statistical output and project file for reproducibility.

## Statistical methods

### Volcano plot coordinates

For a positive linear fold change FC = Sample 1 / Sample 2, the horizontal coordinate is x = log2(FC). If log2 FC is supplied, it is used directly. Thus FC = 2 corresponds to x = 1, FC = 0.5 to x = -1, and FC = 1 to x = 0. The vertical coordinate is y = -log10(u), where u is either the raw p-value or the BH-adjusted p-value. Positive x indicates higher abundance in Sample 1 relative to Sample 2, not necessarily biological activation.

For log transformations, probabilities below 10^-300 (including zero) are floored at 10^-300; the original numeric p-values are retained for BH adjustment. The same floor is used in the classification calculation. A reported zero should be investigated as a possible numerical underflow or rounding artifact, not interpreted as certainty.

### Multiple-testing correction

Volcano Studio implements the Benjamini–Hochberg (BH) step-up adjustment [1]. For m valid hypotheses, sort the raw p-values as p_(1) <= ... <= p_(m). The adjusted p-value for rank i is

$$q_{(i)} = \min\left(1,\min_{j\ge i}\frac{m}{j}p_{(j)}\right).$$

Adjusted values are mapped back to their original rows. The calculation uses all valid input rows, regardless of display filters or sorting. The column labeled FDR contains BH-adjusted p-values, not individual probabilities of being false discoveries or Storey q-values.

Under valid null p-values and independence or appropriate positive dependence (PRDS), BH controls the expected false discovery proportion among its rejection set at the chosen level [1,2]. It does not guarantee that exactly 5% of a particular list are false when alpha = 0.05. Arbitrary dependence is not covered by this guarantee. Import the full intended testing family, not only preselected low-p-value hits. Excluding hypotheses because of their observed results or applying correction separately to selected subsets may invalidate the intended error control.

### Classification and effect-size filtering

With S0 = 0, a row is marked significant if both u <= alpha and |x| >= c, where u is the selected raw or adjusted p-value, alpha is the threshold, and c is the minimum absolute log2 fold change. Defaults: BH-adjusted p-value, alpha = 0.05, c = 1. The latter represents at least a twofold increase or decrease.

The label Significant refers to the current software selection rule. With raw p-values, there is no multiple-testing correction in that rule. With BH-adjusted p-values, the effect-size-filtered subset is not automatically guaranteed to retain the original BH FDR bound. A threshold on an observed fold change is also not a formal test that the true effect exceeds that threshold. For formal minimum-effect inference, use an appropriate upstream method (e.g. TREAT [4]).

### Exploratory S0 curve

The current S0 control defines a custom, symmetric effect-size-dependent selection boundary, not a statistical test. For S0 > 0 and |x| > c:

$$-\log_{10}(u) \ge -\log_{10}(\alpha)+\frac{S_0}{|x|-c}.$$

Equivalently, u <= alpha × 10^(-S0/(|x|-c)). Points at or inside |x| = c are excluded for positive S0 under ordinary parameter settings. The implementation guards the denominator with a small positive floor. A larger S0 makes selection more stringent, especially near the effect-size threshold. S0 = 0 restores the ordinary rectangular selection rule.

**This curve is not the SAM statistic or the permutation-based FDR procedure used in Perseus** [3,5]. SAM uses a variability estimate and a stabilizing constant in a modified test statistic (e.g. d_i = effect_i / (s_i + s0)), together with permutation-based inference. Volcano Studio has no replicate-level measurements or variance estimates and does not perform permutations. Its S0 parameter has no calibrated statistical error-rate interpretation. Do not describe it as a SAM/Perseus FDR threshold or claim that the selected subset has a proven FDR of alpha. Report it as an exploratory curve parameter.

### Threshold visualization

Vertical lines indicate ±c. When the selected classification metric matches the vertical axis, the corresponding horizontal threshold or S0 curve is displayed. With different metrics and S0 = 0, a data-dependent equivalent horizontal cutoff is displayed where available. With different metrics and positive S0, no equivalent curved boundary is drawn. For clarity and reproducibility, use matching classification and axis metrics, particularly when S0 is nonzero. Point styles may be manually overridden and therefore should not be interpreted as significance without checking the sheet.

## Scientific validity and limitations

The log transformations and BH adjustment are established mathematical methods. Their correctness does not establish the validity of the input statistical analysis or of every downstream selection rule. Volcano Studio is a visualization and post-processing tool, not a replacement for differential expression or abundance analysis. It does not estimate variability, normalize samples, handle batch effects, impute missing data, fit statistical models, calculate raw p-values, perform permutation tests, or determine biological relevance.

Use suitable upstream methods and biological replicates; document normalization, statistical tests, the testing family, missing-value handling and the fold-change definition. A small p-value does not necessarily imply a biologically important effect. This software is intended for research and exploratory use, not clinical diagnosis.

### Validation status

The source implements the BH rank-based algorithm above. A useful reference fixture is raw p = [0.001, 0.01, 0.03, 0.04, 0.20], with expected BH-adjusted values [0.005, 0.025, 0.05, 0.05, 0.20]. Compare actual software output with R `p.adjust(p, method = "BH")` before formal release. Further tests should cover ties, zeros, invalid rows, filtering invariance, language switching, spreadsheet paste, and graphic export. Full browser regression testing and independent statistical validation have not been established for v1.2.

## Reproducibility and citation

Record the software version, comparison direction, FC input mode, classification metric, vertical-axis metric, alpha, c, and S0. Preserve the complete input and exported project JSON. When reporting results, distinguish BH adjustment from additional exploratory filtering.

Example methods text (adapt to actual settings):

> Volcano plots were generated using Volcano Studio v1.2. Fold changes were expressed on a log2 scale, and p-values from the upstream statistical analysis were adjusted using the Benjamini–Hochberg method across all valid tested features. Features with BH-adjusted p-values ≤ 0.05 and absolute log2 fold changes ≥ 1 were highlighted. No S0-dependent exploratory filtering was applied.

Suggested software citation: Yoshinari, A. Volcano Studio, version 1.2 [software]. Add the public repository URL and release DOI when available. A DOI has not yet been assigned.

## Privacy and dependencies

Data processing and plot generation occur in the browser; the application does not implement a data-upload backend. XLSX import/export and PDF export load version-pinned external JavaScript libraries on demand from jsDelivr (SheetJS xlsx 0.18.5, jsPDF 2.5.2, svg2pdf.js 2.2.4). Those operations may require internet access. External scripts execute in the page context; the application should not be described as fully offline or independently security-audited. For sensitive unpublished data, consider a reviewed, locally vendored build. Report bugs and feature requests through GitHub Issues after publication.

## License

The project license and institutional approval should be confirmed before public release. Third-party libraries retain their respective licenses. No license grant is implied by this draft.

## References

1. Benjamini Y, Hochberg Y (1995). Controlling the false discovery rate: a practical and powerful approach to multiple testing. JRSS B 57:289–300. https://doi.org/10.1111/j.2517-6161.1995.tb02031.x
2. Benjamini Y, Yekutieli D (2001). The control of the false discovery rate in multiple testing under dependency. Ann Stat 29:1165–1188. https://doi.org/10.1214/aos/1013699998
3. Tusher VG, Tibshirani R, Chu G (2001). Significance analysis of microarrays applied to the ionizing radiation response. PNAS 98:5116–5121. https://doi.org/10.1073/pnas.091062498
4. McCarthy DJ, Smyth GK (2009). Testing significance relative to a fold-change threshold is a TREAT. Bioinformatics 25:765–771. https://doi.org/10.1093/bioinformatics/btp053
5. Cox Labs. Volcano plot analysis documentation. https://cox-labs.github.io/coxdocs/volcanoplotanalysis.html