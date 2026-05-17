# Results

The project was evaluated on synthetic graph datasets where the original values were known. Extracted values were compared against the generated ground truth.

## Summary Table

| Graph Type | RMSE | Accuracy | Correct | Total |
|---|---:|---:|---:|---:|
| Bar | 0.3359 | 90.31% | 175 | 195 |
| Line | 0.2992 | 93.62% | 264 | 282 |
| Scatter Overall | - | 82.85% | 807 | 974 |
| Scatter X | 0.6589 | 80.08% | 390 | 487 |
| Scatter Y | 0.3204 | 85.63% | 417 | 487 |

## Bar Graphs

Bar graph extraction achieved strong accuracy because the top edge of each bar can be detected reliably when the bar color is clearly separated from the background.

Core detection method:

- Axis detection
- OCR for labels
- K-means color clustering
- Prewitt edge detection
- Hough line detection
- Nearest-label mapping

Reported result:

```text
RMSE: 0.3359
Accuracy: 90.31%
Correct: 175
Total: 195
```

## Line Graphs

Line graph extraction achieved the best overall performance among the three supported graph types. The method works well when the plotted line is clean and distinct from the background.

Core detection method:

- Axis detection
- OCR for labels
- Cropping the plotting area
- K-means color isolation
- Binarization
- Line segment detection
- Intersection and endpoint detection
- Nearest-label mapping

Reported result:

```text
RMSE: 0.2992
Accuracy: 93.62%
Correct: 264
Total: 282
```

## Scatter Plots

Scatter plot extraction is more difficult because both x and y values must be recovered from each detected point. Errors in either coordinate affect the final result.

Core detection method:

- Axis detection
- OCR for both axes
- x-axis and y-axis VPP calculation
- Cropping the plotting region
- K-means color isolation
- Binarization
- Blob detection
- Point-center-to-value conversion

Reported results:

```text
Overall Accuracy: 82.85%
Scatter X RMSE: 0.6589
Scatter X Accuracy: 80.08%
Scatter Y RMSE: 0.3204
Scatter Y Accuracy: 85.63%
```

Scatter x-coordinate extraction was less accurate than y-coordinate extraction. This is likely because horizontal tick spacing and x-axis OCR errors have a larger effect on coordinate conversion.

## Interpretation

The results show that the project successfully extracts data from clean 2D graph images under controlled assumptions. Bar and line graphs perform above 90% accuracy, while scatter plots perform lower but remain usable.

The project is especially effective for:

- Synthetic Matplotlib-generated graphs
- Single-series plots
- Solid-colored graph elements
- Clearly separated labels and axes

The project is less reliable for:

- Overlapping scatter points
- Complex legends
- Multi-series graphs
- Low-resolution or noisy scans
- Curved or smoothed lines
- Graphs with gradients or patterned fills

## Example Outputs

Example extracted CSV files are included in:

```text
results/bar/
results/line/
results/scatter/
```

The sample graph image folders are included in:

```text
data/samples/
```

## Future Improvements

Possible improvements include:

- Automatic graph type classification
- Better OCR post-processing
- Support for legends
- Support for multi-series charts
- Support for stacked bar charts
- Support for pie charts
- Robust handling of noisy scanned images
- Refactoring notebook logic into reusable Python modules
- Adding command-line scripts for batch extraction
