# Data Retrieval from Graph Images

This project extracts numerical data from 2D graph images, focusing on three graph types:

- Bar graphs
- Line graphs
- Scatter plots

The implementation uses traditional computer vision and OCR techniques instead of a full custom deep-learning pipeline. The goal is to identify graph axes, read axis labels/titles, detect plotted visual elements, convert pixel measurements into numerical values, and validate the extracted data against known synthetic ground truth.

## Supported Graph Types

### Bar Graphs

For bar graphs, the pipeline detects axes and OCR labels, identifies the top edge of each bar, maps each detected bar to the closest x-axis label, and converts the bar height from pixels into actual values.

### Line Graphs

For line graphs, the pipeline isolates the graphical plotting region, detects the plotted line using color clustering and edge/line detection, finds data-point intersections or segment endpoints, maps them to x-axis labels, and converts pixel positions into y-values.

### Scatter Plots

For scatter plots, the pipeline calculates value-per-pixel for both x and y axes, isolates the scatter point color, detects blobs/points, and converts each detected point center into numerical x-y coordinates.

## Core Techniques

- Keras OCR for text detection
- OpenCV preprocessing
- Grayscale conversion
- Image cropping around axis regions
- Prewitt edge detection
- Probabilistic Hough line detection
- K-means color clustering
- Blob detection for scatter plots
- Pixel-to-value conversion using Value Per Pixel
- RMSE and accuracy-based validation

## Repository Layout

```text
.
├── README.md
├── requirements.txt
├── .gitignore
├── notebooks/
├── data/
├── results/
└── docs/
```

See `docs/repository_structure.md` for a detailed explanation of every folder.

The key implementation challenges and solved edge cases are documented in:

```text
docs/challenges_and_edge_cases.md
```

## Quick Start

1. Create a Python environment.
2. Install dependencies:

```bash
pip install -r requirements.txt
```

3. Open the notebooks in Jupyter Notebook, JupyterLab, or VS Code.
4. Start with:

```text
notebooks/bar_axis_value_extraction.ipynb
notebooks/line_axis_value_extraction.ipynb
notebooks/scatter_axis_value_extraction.ipynb
```

## Data Included

This repository intentionally includes only a small sample dataset so that the GitHub repository remains lightweight.

Included:

- Synthetic CSV ground truth for bar, line, and scatter plots
- Three sample image folders per graph type
- Example extracted output CSV files for bar and line graphs

Not included:

- All generated images from the original working directory
- Large unrelated PDFs and annual reports
- Screenshots and temporary experiment files
- Duplicate testing folders

## Current Status

This is a cleaned academic/demo version of the project. The main implementation is still notebook-based. The `src/` folder is prepared for future refactoring, where reusable Python functions can be extracted from the notebooks.

## Reported Results

From the project report:

| Graph Type | RMSE | Accuracy |
|---|---:|---:|
| Bar | 0.3359 | 90.31% |
| Line | 0.2992 | 93.62% |
| Scatter Overall | - | 82.85% |
| Scatter X | 0.6589 | 80.08% |
| Scatter Y | 0.3204 | 85.63% |

More details are available in `docs/results.md`.

## Limitations

The current project assumes:

- 2D graphs
- Solid colors
- No overlapping scatter points
- No legends inside the plotted region
- Mostly clean graph images
- Single-series charts
- Axis labels readable by OCR

Future work can extend this to handle multi-series charts, legends, stacked bars, pie charts, noisy scanned images, rotated graphs, and fully automatic graph-type classification.
